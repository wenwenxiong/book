参考网址： http://xiaqunfeng.cc/2018/02/28/ceph-ansible%E4%BD%BF%E7%94%A8%E6%89%8B%E5%86%8C/

###ceph roles

名称|作用
---|----
ceph-defaults|
ceph-validate|
ceph-infra|
ceph-handler|
ceph-common|
ceph-config|
ceph-mon|

每个角色```host```定义不同的```roles```进行安装。
角色```mons```上要装的```roles```为```ceph-defaults，ceph-handler，ceph-common，ceph-config，ceph-mon```。

角色```osds```上要装的```roles```为```ceph-defaults，ceph-handler，ceph-common，ceph-config，ceph-osd```。
###ansible部署ceph集群

使用iso安装ubuntu16.04操作系统后，进行如下操作
关闭防火墙
```
systemctl stop firewalld && systemctl disable firewalld
```
用```ansible```对所有节点同步ntp时间服务器。
1、安装notario
```
[root@node1 ~]# apt-get install -y python-pip
[root@node1 ~]# pip install notario
```
离线安装
```
pip install --no-index --find-links=./packages notario
```

2、下载ceph-ansible的压缩包到本地
```
wget -c https://github.com/ceph/ceph-ansible/archive/v3.1.12.tar.gz
tar xf v3.1.12.tar.gz
cd ceph-ansible-3.1.12
```

3、修改inventory，添加主机信息
```
[root@node1 ceph-ansible-3.1.12]# cat hosts.ini 
[all]
node1 ansible_host=192.168.122.11 ansible_ssh_user=root ansible_ssh_private_key_file=/root/ceph-ansible/deploy.key
node2 ansible_host=192.168.122.12 ansible_ssh_user=root ansible_ssh_private_key_file=/root/ceph-ansible/deploy.key
node3 ansible_host=192.168.122.13 ansible_ssh_user=root ansible_ssh_private_key_file=/root/ceph-ansible/deploy.key

[mons]
node1
node2
node3

[osds]
node1
node2
node3

[mgrs]
node1
node2
node3

[mdss]
node1
node2
node3

[clients]
node1
node2
node3
[root@node1 ceph-ansible-3.1.12]# 
```
4、修改all.yml写入如下内容
```
cp group_vars/all.yml.sample group_vars/all.yml

vim group_vars/all.yml

[root@node1 ceph-ansible-3.1.12]# cat group_vars/all.yml
---
ceph_origin: repository
#community
#ceph_repository: community
#ceph_stable_release: luminous
#ceph_mirror: http://mirrors.aliyun.com/ceph
#ceph_stable_key: http://mirrors.aliyun.com/ceph/keys/release.asc
#ceph_stable_repo: "{{ ceph_mirror }}/rpm-{{ ceph_stable_release }}"
#fsid: 2345bb34-2704-47ac-8975-9a17e4f9924e
#generate_fsid: false

#community-ubuntu
#ceph_repository: community
#ceph_stable_release: luminous
#ceph_mirror: http://mirrors.aliyun.com/ceph
#ceph_stable_key: http://mirrors.aliyun.com/ceph/keys/release.asc
#ceph_stable_repo: "{{ ceph_mirror }}/debian-{{ ceph_stable_release }}"
#fsid: 2345bb34-2704-47ac-8975-9a17e4f9924e
#generate_fsid: false

#Custom
ceph_repository: custom
ceph_custom_repo: http://192.168.122.1/ceph_stable
ceph_stable_release: luminous

cephx: true

public_network: "192.168.122.0/24"
cluster_network: "192.168.122.0/24"
mon_host: 192.168.122.11,192.168.122.12,192.168.122.13

#monitor_interface: eth0
monitor_interface: ens3
devices:
  - '/dev/vdb'
  - '/dev/vdc'
osd_scenario: collocated

ceph_conf_overrides:
    global:
      rbd_default_features: 7
      auth cluster required: cephx
      auth service required: cephx
      auth client required: cephx
      osd journal size: 2048
      osd pool default size: 3
      osd pool default min size: 1
      mon_pg_warn_max_per_osd: 1024
      osd pool default pg num: 64
      osd pool default pgp num: 64
      max open files: 131072
      osd_deep_scrub_randomize_ratio: 0.01

    mgr:
      mgr modules: dashboard

    mon:
      mon_allow_pool_delete: true
      mon_clock_drift_allowed: 0.5
      mon clock drift warn backoff: 50

    client:
      rbd_cache: true
      rbd_cache_size: 335544320
      rbd_cache_max_dirty: 134217728
      rbd_cache_max_dirty_age: 10

    osd:
      osd mkfs type: xfs

      ms_bind_port_max: 7100
      osd_client_message_size_cap: 2147483648
      osd_crush_update_on_start: true
      osd_deep_scrub_stride: 131072
      osd_disk_threads: 4
      osd_map_cache_bl_size: 128
      osd_max_object_name_len: 256
      osd_max_object_namespace_len: 64
      osd_max_write_size: 1024
      osd_op_threads: 8

      osd_recovery_op_priority: 1
      osd_recovery_max_active: 1
      osd_recovery_max_single_start: 1
      osd_recovery_max_chunk: 1048576
      osd_recovery_threads: 1
      osd_max_backfills: 4
      osd_scrub_begin_hour: 23
      osd_scrub_end_hour: 7
[root@node1 ceph-ansible-3.1.12]#
```
注意，ceph的网络地址，repo源，osd使用磁盘设备的配置。

5、生成site.yml
```
cp site.yml.sample site.yml

# 注释不需要的组件
vim site.yml
---
# Defines deployment design and assigns role to server groups

- hosts:
  - mons
#  - agents
  - osds
  - mdss
#  - rgws
#  - nfss
#  - restapis
#  - rbdmirrors
  - clients
  - mgrs
#  - iscsigws
#  - iscsi-gws # for backward compatibility only!
```
6、安装
```
ansible-playbook -i hosts site.yml -e 'ansible_python_interpreter=/usr/bin/python3'
```
安装完成后，运行命令检测
```
ceph -s
```
7、清理
如果部署过程中出现报错，建议先清空集群 再进行部署操作
```
cp infrastructure-playbooks/purge-cluster.yml purge-cluster.yml # 必须copy到项目根目录下
ansible-playbook -i hosts purge-cluster.yml
```

改进点： 1）离线deb包配置，2）离线ntp时间同步配置

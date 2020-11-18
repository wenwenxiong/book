###修改ansbible脚本
1、基于```cluster.yml```修改生成新的```sync.yml```文件。
```
cp cluster.yml sync.yml
```
在```sync.yml```只保留以下内容
```
---
- hosts: k8s-cluster:etcd:calico-rr
  any_errors_fatal: "{{ any_errors_fatal | default(true) }}"
  roles:
    - { role: kubespray-defaults}
#    - { role: kubernetes/preinstall, tags: preinstall }
#    - { role: "container-engine", tags: "container-engine", when: deploy_container_engine|default(true) }
    - { role: sync, tags: download, when: "not skip_downloads" }
  environment: "{{proxy_env}}"
```
2、基于```download```创建```sync```的新的```ansible role```。
```
cp -r roles/download roles/sync
```
修改```roles/sync//tasks/main.yml```内容，删除```Sync container```内容，保留以下内容。
```
---
- include_tasks: download_prep.yml
  when:
    - not skip_downloads|default(false)

- name: "Download items"
  include_tasks: "download_{% if download.container %}container{% else %}file{% endif %}.yml"
  vars:
    download: "{{ download_defaults | combine(item.value) }}"
  with_dict: "{{ downloads }}"
  when:
    - not skip_downloads|default(false)
    - item.value.enabled
    - (not (item.value.container|default(False))) or (item.value.container and download_container)
```
3、基于```local```创建```sync```的新的```ansible inventory```。
```
cp -r inventory/local inventory/sync
```
修改```inventory/sync/hosts.ini```为下载镜像的节点。
```
[all]
nyvps ansible_ssh_host=104.156.251.73 ansible_user=root ansible_ssh_private_key_file="/root/.ssh/id_rsa"

[kube-master]
nyvps

[etcd]
nyvps

[kube-node]
nyvps

[k8s-cluster:children]
kube-master
kube-node
```
4、修改文件```roles/sync/defaults/main.yml```，使之下载所有的镜像。
```
vim roles/sync/defaults/main.yml
%s#enabled\:\ \".*#enabled\:\ true#g
```

###执行ansible脚本下载镜像
1、上传修改后的```ansible```脚本到下载镜像的节点。
```
tar -zcvf kubespray-2.8.0.tar.gz kubespray-2.8.0
scp -r kubespray-2.8.0.tar.gz  root@104.156.251.73:/root/Code/
```
2、在下载镜像的节点运行脚本下载镜像
```
tar -zxvf kubespray-2.8.0.tar.gz
cd kubespray-2.8.0
```
编辑脚本文件```pullkubesprayimages.sh```，内容为
```
#!/bin/bash

set -x
#pull images
ansible-playbook  -i inventory/sync/hosts.ini sync.yml -e download_run_once=True -e download_localhost=True
docker save $(docker images -q) | xz>./kubespray_images.tar.xz
# tobedone
docker images | sed -n '1!p'| grep -v "<none>" | awk {'print "docker tag "$3" "$1":"$2'}|tee load.sh
# static website
mv kubespray_images.tar.xz load.sh /root/server
# remove images
docker rmi $(docker images -q)
set +x
```
执行脚本文件```pullkubesprayimages.sh```进行镜像下载。

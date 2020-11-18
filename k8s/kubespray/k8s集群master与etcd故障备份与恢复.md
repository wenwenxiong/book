参考文档： https://yq.aliyun.com/articles/561894

###```Kubernetes Master```节点灾备恢复操作指南

本文档简述了```Kubernetes```主节点灾备恢复的相关步骤，供在发生```k8s master```崩溃时操作。

就算是在```k8s```里部署了```etcd```群集, 主节点控制组件的高可用节点，灾备恢复也是必须要实现的操作，才能形成完备的企业级服务方案。

严格来讲，通过```kubeadm```安装的```k8s```主节点包括两大类的灾备恢复，```etcd```数据存储恢复和主节点控制组件恢复(包括但不限于```kube-apiserver，kube-controller-manager，kube-scheduler，flannel，coreDns，dashboard```)。

所以本文档也会相应的分成两个章节来进行描述。

一，```Etcd```数据备份及恢复
组件```etcd```的数据默认会存放在我们的命令工作目录中，我们发现数据所在的目录，会被分为两个文件夹中：

```snap```: 存放快照数据,```etcd```防止WAL文件过多而设置的快照，存储```etcd```数据状态。
```wal```: 存放预写式日志,最大的作用是记录了整个数据变化的全部历程。在```etcd```中，所有数据的修改在提交前，都要先写入到```WAL```中。
A,单节点```etcd```数据备份和恢复
这种方式的备份和恢复，用基于文件的备份即可。```Kubeadm```的默认安装时，将```etcd```的存储数据落地到了宿主机的```/var/lib/etcd/```目录，将此目录下的文件定期备份起来，如果以后```etcd```的数据出现问题，需要恢复时，直接将文件还原到此目录下，就实现了单节点的```etcd```数据恢复。

(tips:如果```etcd```容器正在启动，是不能覆盖的，这时只需要将```etcd```的```manifest```文件[```/etc/kubernetes/manifests/etcd.yaml```]里的etcd版本号更改一下，然后，用docker stop命令停止etcd容器，就不会自动重启了。数据还原后，将etcd版本再回到正确的版本，kubelet服务就会自动将etcd容器重启起来)

B，```etcd```集群数据的备份和恢复
如果在线上跑的是etcd集群，那么这个集群数据的备份恢复就不能基于单个容器的文件，而需要在集群的容器内用etcdctl命令进行备份和还原数据的操作了。操作思路同单点。相关命令如下：
版本```V2 api```:

备份数据：

```
# etcdctl backup --data-dir /home/etcd/ --backup-dir /home/etcd_backup
```
恢复：
```
# etcdctl -data-dir=/home/etcd_backup/  -force-new-cluster
```

版本```V3 api```：

在使用 ```API 3``` 时需要使用环境变量 ```ETCDCTL_API``` 明确指定。

在命令行设置：
```
# export ETCDCTL_API=3
```
备份数据：
```
# etcdctl --endpoints localhost:2379 snapshot save snapshot.db
```
恢复：
```
# etcdctl snapshot restore snapshot.db --name m3 --data-dir=/home/etcd_data
```
二，```Master```节点控制组件的备份及恢复
一般来说，如果```master```节点需要备份恢复，那除了误操作和删除，很可能就是整个机器已出现了故障，故而可能需要同时进行```etcd```数据的恢复。

而在恢复时，有个前提条件，就是在待恢复的机器上，机器名称和```ip```地址需要与崩溃前的主节点配置完成一样，因为这个配置是写进了```etcd```数据存储当中的。

A，主节点数据备份
主节点数据的备份包括三个部分：

1，```/etc/kubernetes/```目录下的所有文件(证书，```manifest```文件)

2，用户主目录下```.kube/config```文件(```kubectl```连接认证)

3，```/var/lib/kubelet/```目录下所有文件(```plugins```容器连接认证)

B，主节点组件恢复
主节点组件的恢复可按以下步骤进行：

1，按之前的安装脚本进行全新安装(```kubeadm reset，iptables –X…```)
2，停止系统服务```systemctl stop kubelet.service```。
3，删除相关插件容器(```coredns,flannel,dashboard```)。
4，恢复```etcd```数据(参见第一章节操作)。
5，将之前备份的三个目录依次还原。
6，重启系统服务```systemctl start kubelet.service```。
7，稍等片刻，待所有组件启动成功后进行验证。
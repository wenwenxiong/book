参考文档： https://github.com/kubernetes-sigs/kubespray/blob/master/docs/vars.md
https://github.com/kubernetes-sigs/kubespray/blob/master/docs/upgrades.md


###下载镜像，二进制，```cni```配置文件
下载开源项目```kubespray```的```release```包，例如```kubespray-2.8.2```。
执行以下步骤下载镜像，二进制，```cni```配置文件。
1、基于```local```创建```sync```的新的```ansible inventory```
```
cp -r  inventory/sample inventory/sync
```
修改文件```inventory/sync/hosts.ini```配置下载的目的主机，一般为该运行机器。
修改文件```inventory/sync/group_vars/k8s-cluster/k8s-cluster.yml```中变量```local_release_dir```的值为```/root/server/kubespray-2.8.2```，这个目录路径会保存下载的镜像，二进制和```cni```配置文件。

2、基于```download```创建```sync```的新的```ansible role```。
```
cp -r roles/download roles/sync
```
修改文件```roles/sync/defaults/main.yml```，使之下载所有的镜像。
```
vim roles/sync/defaults/main.yml
%s#enabled\:\ \".*#enabled\:\ true#g
```
3、基于```cluster.yml```修改生成新的```sync.yml```文件
```
cp cluster.yml sync.yml
```
修改```sync.yml```中```download```的```role```名称为```ysnc```。

4、执行以下命令下载
```
ansible-playbook  -i inventory/sync/hosts.ini sync.yml -e download_run_once=True -e download_localhost=True --tags=download
```
下载的镜像，二进制和```cni```配置文件在```/root/server/kubespray-2.8.2```目录下。

如果机器内存或者存储不足。
修改```roles/sync//tasks/main.yml```内容，删除```Sync container```内容

5、下载```kubeadm```所需的```kubernetes```集群镜像
查询```kubeadm```所需的镜像
```
./kubeadm config images list --kubernetes-version v1.12.5
```
下载```kubeadm```所需的镜像
```
./kubeadm config images pull --kubernetes-version v1.12.5
```

###导入镜像到镜像仓库
在待升级的集群上任意节点把新的镜像(包括```kubeadm```下载的镜像)```push```到镜像仓库
把新的二进制包和```cni```包放到```repo```服务器目录上

###升级集群
把新的```kubespray```的```release```包拷贝到待升级集群的```deploy```节点上。按以下步骤进行升级
1）升级```kubernetes```集群版本
```
ansible-playbook  -i inventory/rong/hosts.ini upgrade-cluster.yml -e kube_version=v1.12.5
```
升级完成后，执行命令
```
kubectl version
```
验证版本。
2）升级其他组件
所有组件版本变量如下
```
docker_version
kube_version
etcd_version
calico_version
calico_cni_version
weave_version
flannel_version
kubedns_version
```
组件升级顺序
```
Docker
etcd
kubelet and kube-proxy
network_plugin (such as Calico or Weave)
kube-apiserver, kube-scheduler, and kube-controller-manager
Add-ons (such as KubeDNS)
```
升级```docker```命令
```
ansible-playbook -b -i inventory/sample/hosts.ini cluster.yml --tags=docker
```
升级```etcd```集群
```
ansible-playbook -b -i inventory/sample/hosts.ini cluster.yml --tags=etcd
```
升级```vault```集群
```
ansible-playbook -b -i inventory/sample/hosts.ini cluster.yml --tags=vault
```
升级```kubelet```集群
```
ansible-playbook -b -i inventory/sample/hosts.ini cluster.yml --tags=node --skip-tags=k8s-gen-certs,k8s-gen-tokens
```
升级```Kubernetes master components```集群
```
ansible-playbook -b -i inventory/sample/hosts.ini cluster.yml --tags=master
```
升级``` network plugins```
```
ansible-playbook -b -i inventory/sample/hosts.ini cluster.yml --tags=network
```
升级```add-ons```
```
ansible-playbook -b -i inventory/sample/hosts.ini cluster.yml --tags=apps
```
升级插件中的```helm```
```
ansible-playbook -b -i inventory/sample/hosts.ini cluster.yml --tags=helm
```
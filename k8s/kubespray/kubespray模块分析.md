###入口文件
kubespray安装kubernetes的命令如下
```
ansible-playbook -i inventory/mycluster/hosts.ini  cluster.yml
```
入口文件为```cluster.yml```。

在```cluster.yml```文件中定义了各个不同组的```host```执行不同的```ansible roles```。

重要的```ansbile roles```。

名称| 功能| 说明
----|----|----
kubespray-defaults| 加载ansible中定义的变量和默认值|在每个其他roles执行前，都会执行该role
bootstrap-os|针对不同os执行优化配置和工具安装，hostname配置|主要是安装python类工具
kubernetes/preinstall|安装前的一些准备工作，例如创建kubernetes目录，安装源配置|
docker|不同os下的docker安装与配置|
download|下载与同步docker镜像|有直接从仓库下载和安装节点下载一次然后发送给目标kubernetes节点
vault|证书管理器vault安装|
etcd|etcd集群安装与配置|有docker形式安装和直接安装到主机两种方式
kubernetes/node|kubernetes 工作节点安装与配置| kubelet， kube-proxy，IPVS配置，kubeadm方式（docker，主机安装方式）等
kubernetes/master|kubernetes master节点安装与配置|kubectl安装，master上组件的安装（有kubeadm安装和static pod安装方式）
kubernetes/client|kubernetes client节点配置文件生成|
kubernetes-apps/cluster_roles|不同环境下kubernetes的RBAC配置|
kubernetes/kubeadm|kubeadm配置|
netwok_plugin|不同网络插件的roles|
kubernetes-apps/rotate_tokens|kubernetes secrets操纵|
kubernetes-apps/network_plugin|不同网络插件的启动|
kubernetes-apps/policy_controller|calico的policy controller安装|
kubernetes-apps/ingress_controller|ingress controller和cert manager安装与配置|
kubernetes-apps/external_provisioner|cephfs和localvolume的配置|
network_plugin/calico/rr|calico网络rr模式配置|
dnsmasq|kubernetes dns安装与配置|
kubernetes-apps|非kubernetes系统组件，一些常用的组件安装与配置|cloud_controller,dns,dashboard,helm,efk,registry等

在实践场景遇到相关问题，就定位到对应的模块进一步细看。
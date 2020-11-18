###kubespray启用cilium网络插件
修改配置文件```inventory/rong/group_vars/k8s-cluster/k8s-cluster.yml```的配置```kube_network_plugin: cilium```。

###kubespray测试cilium的clustermesh
创建第二个云平台```Rong```，网络插件为```cilium```，并且配置与第一个```Rong```不同的```pod cidr```。
配置```pod cidr```为配置文件```inventory/rong/group_vars/k8s-cluster/k8s-cluster.yml```的配置项```kube_pods_subnet: 10.233.64.0/18```。
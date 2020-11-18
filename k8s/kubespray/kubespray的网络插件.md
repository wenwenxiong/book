### kubespray支持的网络插件
kubespray部署kubernetes集群时支持多种网络插件的集成，如下表所示

网络插件名 | 特征作用
---------|---------
calico   | pod网络连通、网络策略隔离（namespace，pod）
flannel  | pod网络连通
canal    | calico与flannel集合
multus   | pod多网卡支持，与其他cni插件一起部署，其他cni插件提供pod网络连通
weave    | pod网络连通、轻量级，不需要额外的K/V数据库集群(etcd集群)
macvlan  | pod网络连通
kube-router| pod网络连通
contiv   | pod网络连通，网络隔离策略（防火墙，流量限制），多租户子网隔离
kube-ovn | pod网络连通，固定IP，网络隔离策略（防火墙，流量限制），多子网支持（-namespace-子网，子网安全组策略）
cilium | pod网络连通，负载均衡，多集群网络联通

### calico网络插件
参考网址：https://blog.csdn.net/ccy19910925/article/details/82424275
1、pod联通性原理
网络插件```calico```主要通过iptables规则和```veth```接口对实现```pod```网络连通（采用的是路由策略）。```k8s```创建容器时调用```cni```插件```calico```创建一个```veth```接口对，一个在容器中为```eth0```,一个在容器所在的节点上名为```calxxxx```。```veth```接口对的特性是流量访问接口对的一端会从另一端出来，在容器内部默认路由是走```eth0```,因此会流入到容器所在节点的```calxxxx```接口。
容器内部接口
![01](./calicol/01.png "01")
通过```calicoctl```运行命令查看关联性
![02](./calicol/02.png "02")
在节点上查看接口
![03](./calicol/03.png "03")
使用命令```ip route```在节点的路由表中可以看到```pod```目标IP的下一跳地址是配置在```pod```所在节点```ip```上。
到达目标```pod```所在的节点后。在节点上使用命令```ip route```会看到目标```pod```的```IP```对应的网卡是```caliXXX```，写入caliXX的报文会通过容器内的eth0流出，从而进入到容器的网络空间中。
![04](./calicol/04.png "04")

### flannel网络插件
参考网址： http://dockone.io/article/618
Flannel是 CoreOS 团队针对 Kubernetes 设计的一个覆盖网络（Overlay Network）工具，其目的在于帮助每一个使用 Kuberentes 的 CoreOS 主机拥有一个完整的子网。
1、pod连通性
Flannel实质上是一种“覆盖网络(overlay network)”，也就是将TCP数据包装在另一种网络包里面进行路由转发和通信，目前已经支持UDP、VxLAN、AWS VPC和GCE路由等数据转发方式。
默认的节点间数据通信方式是UDP转发。
1）为什么每个节点上的Docker会使用不同的IP地址段？

这个事情看起来很诡异，但真相十分简单。其实只是单纯的因为Flannel通过Etcd分配了每个节点可用的IP地址段后，偷偷的修改了Docker的启动参数。
2）为什么在发送节点上的数据会从docker0路由到flannel0虚拟网卡，在目的节点会从flannel0路由到docker0虚拟网卡？
因为配置了路由表大范围子网通过```flannel0```，小范围子网通过```docker0```。

### canal网络
网络插件```calico```和```flannal```组合，已不再更新，新特点将单独在组件```calico```和```flannal```中开发。




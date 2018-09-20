###service的proxy mode
  kube-proxy是实现 service 功能的关键。kube-proxy 运行在每个节点上，监听 API Server 中服务对象的变化，通过管理 iptables 来实现网络的转发。
kube-proxy 有几种实现 service 的方案：userspace ， iptables 和ipvs（kubernetes 1.9）

userspace 是在用户空间监听一个端口，所有的 service 都转发到这个端口，然后 kube-proxy 在内部应用层对其进行转发。因为是在用户空间进行转发，所以效率也不高。
iptables 完全实现 iptables 来实现 service，是目前默认的方式，也是推荐的方式，效率很高（只有内核中 netfilter 一些损耗）。
userspace 只是旧版本支持的模式，以后可能会放弃维护和支持。

iptables的模式中如果转发的 pod 不能正常提供服务，它不会自动尝试另一个 pod，当然这个可以通过 readiness probes 来解决。每个 pod 都有一个健康检查的机制，当有 pod 健康状况有问题时，kube-proxy 会删除对应的转发规则。

ipvs模式 调用netlink接口创建ipvs规则（ipvs规则与kubernetes的services和endpoint周期性保持同步）。ipvs的重定向服务比uptables快速，并且同步代理规则也比iptables性能优越。还提供以下的负载均衡算法：
```
rr: round-robin
lc: least connection
dh: destination hashing
sh: source hashing
sed: shortest expected delay
nq: never queue
```
注：ipvs 模式需要每个节点都安装了IPVS内核模块（在装kube-proxy之前），如果在装kube-proxy验证节点没有安装IPVS内核模块，会配置为iptables模式。
###节点配置IPVS模块
 配置内核参数：/etc/sysctl.d/k8s.conf
```
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
vm.swappiness=0
```
执行命令
```
sysctl -p /etc/sysctl.d/k8s.conf
```
载入ipvs内核模块：
```
modprobe ip_vs
modprobe ip_vs_rr
```
###centos节点配置IPVS模块
第一步，在内核中加载ip_vs模块：
```
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
ipvs_modules="ip_vs ip_vs_lc ip_vs_wlc ip_vs_rr ip_vs_wrr ip_vs_lblc ip_vs_lblcr ip_vs_dh ip_vs_sh ip_vs_nq ip_vs_sed ip_vs_ftp nf_conntrack_ipv4"
for kernel_module in \${ipvs_modules}; do
    /sbin/modinfo -F filename \${kernel_module} > /dev/null 2>&1
    if [ $? -eq 0 ]; then
        /sbin/modprobe \${kernel_module}
    fi
done
EOF
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep ip_vs
```
输出结果应该为：
```
[root@node01 ~]# lsmod | grep ip_vs
ip_vs_ftp              13079  0 
ip_vs_sed              12519  0 
ip_vs_nq               12516  0 
ip_vs_sh               12688  0 
ip_vs_dh               12688  0 
ip_vs_lblcr            12922  0 
ip_vs_lblc             12819  0 
ip_vs_wrr              12697  0 
ip_vs_rr               12600  3 
ip_vs_wlc              12519  0 
ip_vs_lc               12516  0 
nf_nat                 26787  3 ip_vs_ftp,nf_nat_ipv4,nf_nat_masquerade_ipv4
ip_vs                 141092  27 ip_vs_dh,ip_vs_lc,ip_vs_nq,ip_vs_rr,ip_vs_sh,ip_vs_ftp,ip_vs_sed,ip_vs_wlc,ip_vs_wrr,ip_vs_lblcr,ip_vs_lblc
nf_conntrack          133387  7 ip_vs,nf_nat,nf_nat_ipv4,xt_conntrack,nf_nat_masquerade_ipv4,nf_conntrack_netlink,nf_conntrack_ipv4
libcrc32c              12644  4 xfs,ip_vs,nf_nat,nf_conntrack
```

第二步，安装ipvs管理工具ipvsadm
```
yum install -y ipvsadm
```
第三步，修改集群配置文件

在使用kubeadm init --config config.yaml初始化集群前，修改集群配置文件
```
Kubernetes v1.8 v1.9
kind: MasterConfiguration
apiVersion: kubeadm.k8s.io/v1alpha1
...
kubeProxy:
  config:
    featureGates: SupportIPVSProxyMode=true
    mode: ipvs
...

Kubernetes v1.10
kind: MasterConfiguration
apiVersion: kubeadm.k8s.io/v1alpha1
...
kubeProxy:
  config:
    featureGates: 
      SupportIPVSProxyMode: true
    mode: ipvs
...
```

第四步，结果验证

```
[root@node01 ~]# ipvsadm -L -n
```
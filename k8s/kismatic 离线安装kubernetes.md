###kismatic 离线安装kubernetes
构建kvm环境
内网网络：192.168.123.0/24
节点 | IP
----|----
aptly| 192.168.123.110
registry| 192.168.123.111
k8s1| 192.168.123.112
k8s2| 192.168.123.113
k8s3| 192.168.123.114

###外部访问k8s的dashboard
1、生成dashboard的认证配置文件
运行命令
```
./kismatic dashboard
```
会在文件夹generated中产生dashboard-admin-kubeconfig配置文件，该文件用于登录k8s的dashboard界面认证文件。
此外，它还会在本机调用浏览器访问dashboard（只能在本机上访问）。
按Ctrl+c结束命令。
2、更改运行中dashboard的service
执行命令
```
./kubectl --kubeconfig generated/kubeconfig edit svc kubernetes-dashboard -n kube-system
```
把type: ClusterIP 改为 type: NodePort。
3、外部访问dashboard
使用命令
```
./kubectl --kubeconfig generated/kubeconfig get svc kubernetes-dashboard -n kube-system
```
得到kubernetes-dashboard 的 NodePort端口
访问https://$(masterip):$(NodePort),认证配置文件使用第一步生成的dashboard-admin-kubeconfig。
###kismatic的calico组成
1、calico.yaml和rbac.yaml文件
位于
```
/etc/calico/calico.yaml
/etc/calico/rabc.yaml
```
2、
位于policy-controller.yaml
```
/etc/kubernetes/specs/policy-controller.yaml
```
3、etcd_networking服务
```
systemctl status etcd_networking

```
###销毁kismatic安装的calico网络
使用以下命令销毁由kismatic安装的calico网络服务
```
kubectl delete -f /etc/calico/calico.yaml -f /etc/calico/rabc.yaml -f /etc/kubernetes/specs/policy-controller.yaml
```
停止etcd_networking服务
```
systemctl stop etcd_networking
systemctl disable etcd_networking
```

###重新安装calico
https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/kubeadm/
下载calico镜像
下载calico的yaml文件（https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/kubeadm/1.7/calico.yaml）
修改calico.yaml文件使pod网络，service ip网络与实际的kubernetes环境保持一致
```
kubectl create -f calico.yaml
```


###处理重装calico后的异常服务
重新安装calico后，kubernetes-dashboard和kubernetes-dns,tiller服务出现异常，处理方式就是重新启动这些服务
重新启动命令如下
```
kubectl delete -f /etc/kubernetes/specs/kubernetes-dns.yaml -f /etc/kubernetes/specs/kubernetes-dashboard.yaml
kubectl create -f /etc/kubernetes/specs/kubernetes-dns.yaml -f /etc/kubernetes/specs/kubernetes-dashboard.yaml

kubectl delete deployment tiller-deploy -n kube-system
kubectl delete svc tiller-deploy -n kube-system
helm init --service-account=tiller -i=mirror.teligen.com/gcr.io/kubernetes-helm/tiller:v2.7.2 --upgrade --skip-refresh
```

重新安装的calico支持kubernetes的hostport模式，但是只能通过非本地IP（locahost，127.0.0.1）访问才有效果。
例如 curl 192.168.122.112
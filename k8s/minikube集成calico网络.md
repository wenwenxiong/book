###启动minikube
启动一个minikube实例，并且配置以下参数
```
minikube start --vm-driver kvm2 -p minikubeingresstest --network-plugin cni --cpus 4 --memory 8196
```
 --vm-driver kvm2:配置VM的驱动为kvm
 -p minikubeingresstest：指定minikube的虚拟机名称，不同名称可以同时运行（但是kvm2驱动不支持）
 --network-plugin cni：网络模型为cni

###安装calico
https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/kubeadm/
下载calico镜像
下载calico的yaml文件（https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/kubeadm/1.7/calico.yaml）
修改calico.yaml文件使pod网络，service ip网络与实际的kubernetes环境保持一致
```
kubectl create -f calico.yaml
```

###安装custom nginx-ingress

nginx ingress controller的ingress规则配置体现在nginx-controller pod中的/etc/nginx/nginx.conf
文件中，每次kubernetes创建新的ingress时，便会刷新/etc/nginx/nginx.conf配置文件。
对每一个ingress规则的nginx自定义配置，是在ingress的annotations中配置的
要配置所有的nginx自定义配置，是在 ingress controller pod的命令行指定的configmap文件中配置，我们可以自行修改该configmap文件。
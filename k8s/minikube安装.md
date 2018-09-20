###linux下安装minikube
下载minikube
```
sudo curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && sudo chmod +x minikube && sudo mv minikube /usr/local/bin/
```
下载kubectl
```
sudo curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && sudo chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```
下载minikube的KVM2驱动（本环境使用kvm创建的VM）
```
sudo curl -LO https://storage.googleapis.com/minikube/releases/latest/docker-machine-driver-kvm2 && sudo chmod +x docker-machine-driver-kvm2 && sudo mv docker-machine-driver-kvm2 /usr/bin/
```
###启动minikube
本次启动的minikube使用cni网络
```
minikube start --vm-driver kvm2 --network-plugin cni
```
配置docker命令操作minikube环境
```
eval $(minikube docker-env)
```
下载minikube所需的镜像
```
#!/bin/bash

set -x
dockerimages=(
k8s.gcr.io/k8s-dns-kube-dns-amd64:1.14.5
k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64:1.14.5
k8s.gcr.io/k8s-dns-sidecar-amd64:1.14.5
k8s.gcr.io/kubernetes-dashboard-amd64:v1.8.1
gcr.io/google_containers/pause-amd64:3.0
gcr.io/google-containers/kube-addon-manager:v6.5
)

j=1
for i in ${dockerimages[@]}
do
    echo $i
    echo $j

    docker pull $i && docker save $i -o $j.tar && xz $j.tar
    docker rmi $i
    let j+=1
done
set +x
```
###minikube插件启用
使用minikube命令可以查看，启用，禁用插件。启用插件是即时生效，不需要重启。
```
xww@xww-HP-EliteBook-8460p:~$ minikube addons list
- addon-manager: enabled
- coredns: disabled
- dashboard: enabled
- default-storageclass: enabled
- efk: enabled
- freshpod: disabled
- heapster: enabled
- ingress: enabled
- kube-dns: enabled
- registry: enabled
- registry-creds: disabled
- storage-provisioner: enabled

```

###kubespray使用kubeadm模块

部署框架```kubespray```在```2.8```版本配置默认以```kubeadm```作为部署```kubernetes```方法（之前版本使用```hyperkube```），```kubeadm```部署时默认从Internet下载镜像，需作以下配置从私有镜像仓库下载镜像。

修改配置文件```k8s-cluster.yaml```,文件位于```Rong/kubespray/inventory/rong/group_vars/k8s-cluster```文件夹下。修改配置项
```
...
# kubernetes image repo define
kube_image_repo: "portus.teligen.com:5000/kubesprayns/gcr.io/google-containers"
...
```
部署时，```kubeadm```则会从私有仓库```portus.teligen.com:5000```的```kubesprayns```名字空间下载镜像。
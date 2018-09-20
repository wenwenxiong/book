###安装kubeapps
执行以下命令安装
```
curl -s https://api.github.com/repos/kubeapps/kubeapps/releases/latest | grep -i $(uname -s) | grep browser_download_url | cut -d '"' -f 4 | wget -i -
sudo mv kubeapps-$(uname -s| tr '[:upper:]' '[:lower:]')-amd64 /usr/local/bin/kubeapps
sudo chmod +x /usr/local/bin/kubeapps
```
部署kubeapps
如果能够访问外部的网络，执行下面命令
```
kubeapps up
```
离线环境，首先下载所需的镜像
```
kubeapps up --dry-run -o yaml > kubeapps.yaml
cat kubeapps.yaml | grep image:
```
下载完镜像后，替换镜像拉取策略
```
sed -i 's#Always#IfNotPresent#g' kubeapps.yaml
```
建立存储块给kubeapps使用，实际存储依赖k8s的使用环境，我是使用ceph rbd存储。
rbd-default-storageclass.yaml
```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: default-sc
  annotations:
    storageclass.beta.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/rbd
parameters:
  monitors: 192.168.122.120:6789,192.168.122.121:6789,192.168.122.122:6789
  adminId: admin
  adminSecretName: ceph-secret
  adminSecretNamespace: kubeapps
  pool: ssd-pool
  userId: admin
  userSecretName: ceph-secret-user
  fsType: ext4
  imageFormat: "2"
  imageFeatures: "layering"
```
rbd-pv-8g.yaml
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: rbdpv8g
spec:
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: default-sc
  rbd:
      monitors:
        - '192.168.122.120:6789,192.168.122.121:6789,192.168.122.122:6789'
      pool: ssd-pool
      image: rbdpd8
      fsType: ext4
      readOnly: false
      user: admin
      keyring: /etc/ceph/ceph.client.admin.keyring
```
需要自己预先在ceph的ssd-pool池上创建名为rbdpd8的image。
命令执行如下
```
kubectl create -f kubeapps-ns.yaml
kubectl create -f rbd-default-storageclass.yaml
kubectl create -f rbd-pv-8g.yaml
kubectl create -f rbd-pv-1g.yaml
kubectl create -f rbd-pv-1g2.yaml
kubectl create -f rbd-pv-1g3.yaml
```
更改kubeapps.yaml，替换里面的镜像为自己registry上的镜像，执行以下命令安装
```
kubectl create -f kubeapps.yaml
```
###删除kubeapps
从集群中删除kubeapps
```
kubeapps down
```
或者使用kubectl逆序删除前面创建的对象。
###启动kubeapps ui
启动kubeapps ui
```
kubeapps dashboard
```
这个命令只能启用本地127.0.0.1的端口访问UI，只能本机器的浏览器访问。
如果想要被外部访问UI
编辑kubeapps名字空间下kubeapps服务，把服务类型ClusterIP改成LoadBalancer。
```
kubectl edit svc kubeapps -n kubeapps
```
![dashboard-login](./kubeapps/dashboard-login.png "dashboard-login")

执行以下命令生成访问token
```
kubectl create serviceaccount kubeapps-operator
kubectl create clusterrolebinding kubeapps-operator --clusterrole=cluster-admin --serviceaccount=default:kubeapps-operator
kubectl get secret $(kubectl get serviceaccount kubeapps-operator -o jsonpath='{.secrets[].name}') -o jsonpath='{.data.token}' | base64 --decode
```

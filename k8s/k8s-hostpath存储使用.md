参考网址: https://github.com/nailgun/k8s-hostpath-provisioner
###运行k8s-hostpath-provisioner
下载github项目
```
git clone https://github.com/nailgun/k8s-hostpath-provisioner.git
```
下载k8s-hostpath镜像
```
docker pull nailgun/k8s-hostpath-provisioner:latest
```
自己可以把镜像push到私有仓库，修改yaml中镜像的路径。
进行项目的example目录，运行k8s-hostpath-provisioner
```
$ cd example
$ kubectl create -f sa.yaml -f cluster-roles.yaml -f cluster-role-bindings.yaml -f ds.yaml
```
标记所有k8s节点都可以提供hostpath卷
```
$ kubectl label node --all nailgun.name/hostpath=enabled
```
###k8s hostpath存储使用
定义storageclass
例如sc.yaml指明将要使用的目录为k8s-hostpath-provisioner的/ssd目录。
```
[root@k8s-master example]# cat sc.yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: local-ssd
provisioner: nailgun.name/hostpath
parameters:
  hostPathName: ssd
```
标注节点（自主选择）将k8s-hostpath-provisioner的/ssd目录映射到节点/tmp/ssd目录
```
$ kubectl annotate node NODE_NAME hostpath.nailgun.name/ssd=/tmp/ssd
```
构建pvc.yaml使用存储local-ssd。
```
[root@k8s-master example]# cat pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: local-ssd
  annotations:
    volume.beta.kubernetes.io/storage-class: local-ssd
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Mi
```
构建Pod使用名为local-ssd的pvc。
```
[root@k8s-master example]# cat test-pod.yaml 
kind: Pod
apiVersion: v1
metadata:
  name: test-pod
spec:
  containers:
  - name: test-pod
    image: gcr.io/google_containers/busybox:1.24
    command:
    - "/bin/sh"
    args:
    - "-ec"
    - "echo hello >> /mnt/test && cat /mnt/test"
    volumeMounts:
    - name: hostpath-pvc
      mountPath: "/mnt"
  restartPolicy: "Never"
  volumes:
  - name: hostpath-pvc
    persistentVolumeClaim:
      claimName: local-ssd
```
这样该test-pod的/mnt目录就会映射到选定节点的/tmp/ssd目录中的一个以PVC开头的目录。
运行例子
```
$ kubectl create -f sc.yaml -f pvc.yaml -f test-pod.yaml
```
使用命令查看/tmp/ssd目录中那个PVC目录绑定
```
kubectl get pvc
```
可以查看该PVC目录是否有test文件。
###kubespray中cephfs_provisioner启用
修改配置文件```Rong/kubespray/inventory/rong/group_vars/k8s-cluster/addons.yml```，加入以下配置项
```
...
# CephFS provisioner deployment
cephfs_provisioner_enabled: true
cephfs_provisioner_namespace: "cephfs-provisioner"
cephfs_provisioner_cluster: ceph
cephfs_provisioner_monitors: "192.168.122.11:6789,192.168.122.12:6789,192.168.122.13:6789"
cephfs_provisioner_admin_id: admin
#cephfs_provisioner_secret: secret
cephfs_provisioner_secret: "AQAlhQdcybvfFxAA5YAgeakHm3NGVpDWIqMCKg=="
cephfs_provisioner_storage_class: cephfs
cephfs_provisioner_reclaim_policy: Delete
cephfs_provisioner_claim_root: /volumes
cephfs_provisioner_deterministic_names: true
...
```
注：
1,```cephfs_provisioner_monitors```配置ceph集群存储的```monitor```节点。
2, ```cephfs_provisioner_secret```配置ceph集群存储admin用户的证书，使用命令```ceph auth get-key client.admin```获取。
3,```ceph```集群的部署参考文档《ansible部署ceph》。

###测试cephfs_provisioner的使用
配置完后，使用kubespray搭建kubernetes集群，会发现```cephfs-provisioner```容器实例。
```
[root@node1 kubespray]# kubectl get pod --all-namespaces
NAMESPACE            NAME                                       READY   STATUS    RESTARTS   AGE
cephfs-provisioner   cephfs-provisioner-bfbd95c76-x64dx         1/1     Running   2          12h
...
```
并且存在名为```cephfs```的```storageclass```。
```
[root@node1 ~]# kubectl get sc
NAME     PROVISIONER       AGE
cephfs   ceph.com/cephfs   12h
[root@node1 ~]#
```
测试示例文件如下```cephfs-pvc.yaml，cephfs-pod.yaml```。内容分别为
```
[root@node1 ~]# cat cephfs-pvc.yaml 
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim1
spec:
  storageClassName: cephfs
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
[root@node1 ~]#
```
文件```cephfs-pod.yaml```内容为
```
[root@node1 ~]# cat cephfs-pod2.yaml 
kind: Pod
apiVersion: v1
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
       claimName: claim1
  containers:
    - name: task-pv-container
      image: portus.teligen.com:5000/kubesprayns/nginx:1.13
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
[root@node1 ~]#
```
使用命令```kubectl create -f cephfs-pvc.yaml -f cephfs-pod.yaml```执行后，测试```pvc```能成功挂载```pv```，并且```pod```能成功运行和使用挂载的存储。
但是经过测试发现，```cephfs-provisioner```无法限制存储大小，```pvc```配置```1G```，却可以拷入```1.9G```大小的文件。
```
[root@node1 ~]# docker cp Rong.tar.gz k8s_task-pv-container_task-pv-pod_default_3be7b5e8-f88c-11e8-9500-52540020d495_1:/usr/share/nginx/html
[root@node1 ~]#
[root@node1 ~]# kubectl get pvc
NAME     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
claim1   Bound    pvc-e5741403-f88a-11e8-9500-52540020d495   1Gi        RWX            cephfs         12h
[root@node1 ~]# 
[root@node1 ~]# kubectl get pod
NAME          READY   STATUS    RESTARTS   AGE
task-pv-pod   1/1     Running   1          12h
[root@node1 ~]# 
[root@node1 ~]# kubectl exec -it task-pv-pod /bin/bash
root@task-pv-pod:/# 
root@task-pv-pod:/# cd /usr/share/nginx/html/
root@task-pv-pod:/usr/share/nginx/html# ls
Rong.tar.gz  SUCCESS
root@task-pv-pod:/usr/share/nginx/html# du -sh ./*
1.9G    ./Rong.tar.gz
512     ./SUCCESS
root@task-pv-pod:/usr/share/nginx/html#
```
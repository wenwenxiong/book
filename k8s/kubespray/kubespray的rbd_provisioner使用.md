参考网址： https://github.com/kubernetes-sigs/kubespray/pull/3668/commits
https://github.com/kubernetes-incubator/external-storage/issues/538

###kubespray中rbd_provisioner启用
修改配置文件```Rong/kubespray/inventory/rong/group_vars/k8s-cluster/addons.yml```，加入以下配置项
```
...
# RBD provisioner deployment
rbd_provisioner_enabled: true
rbd_provisioner_namespace: "rbd-provisioner"
rbd_provisioner_replicas: 2
rbd_provisioner_monitors: "192.168.122.21:6789,192.168.122.22:6789,192.168.122.23:6789"
rbd_provisioner_pool: kube
rbd_provisioner_admin_id: admin
rbd_provisioner_secret_name: ceph-secret-admin
rbd_provisioner_secret_token: AQBz2AlcHt1gHRAAZiYRbFCsHwrXnCGVO2fbVw==
rbd_provisioner_user_id: kube
rbd_provisioner_user_secret_name: ceph-secret-user
rbd_provisioner_user_secret_token: AQAK3QlcICmBNhAA8PDulQs2mjFpUkcKaDKtyQ==
rbd_provisioner_user_secret_namespace: rbd-provisioner
rbd_provisioner_fs_type: ext4
rbd_provisioner_image_format: "2"
rbd_provisioner_image_features: layering
rbd_provisioner_storage_class: rbd
rbd_provisioner_reclaim_policy: Delete
...
```
注：
1,```rbd_provisioner_monitors```配置ceph集群存储的```monitor```节点。
2, ```rbd_provisioner_secret_token```配置ceph集群存储admin用户的证书，使用命令```ceph auth get-key client.admin```获取。
3,配置项```rbd_provisioner_pool: kube、rbd_provisioner_user_id: kube、rbd_provisioner_user_secret_token: ceph-key-user```需要预先在```ceph```集群上执行以下操作
```
ceph osd pool create kube 8 8
ceph auth add client.kube mon 'allow r' osd 'allow rwx pool=kube'
```
使用命令```ceph auth get-key client.kube```获取用户```kube```的证书填入配置项```rbd_provisioner_user_secret_token```。
4,```ceph```集群的部署参考文档《ansible部署ceph》。

###测试rbd_provisioner的使用
配置完后，使用kubespray搭建kubernetes集群，会发现```rbd-provisioner```容器实例。
```
[root@node1 kubespray]# kubectl get pod --all-namespaces
NAMESPACE            NAME                                       READY   STATUS    RESTARTS   AGE
rbd-provisioner   rbd-provisioner-75f99c8b4b-fxpwt           1/1     Running   0          4m54s
rbd-provisioner   rbd-provisioner-75f99c8b4b-nmmjq           1/1     Running   0          4m54s
...
```
并且存在名为```rbd```的```storageclass```。
```
[root@node1 ~]# kubectl get sc
NAME     PROVISIONER       AGE
rbd   ceph.com/rbd   12h
[root@node1 ~]#
```
测试示例文件如下```rbd-pvc.yaml，rbd-pod.yaml```。内容分别为
```
[root@node1 ~]# cat rbd-pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim1
spec:
  storageClassName: rbd
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
[root@node1 ~]#
```
文件```rbd-pod.yaml```内容为
```
[root@node1 ~]# cat rbd-pod.yaml
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
使用命令```kubectl create -f rbd-pvc.yaml -f rbd-pod.yaml```执行后，测试```pvc```能成功挂载```pv```，并且```pod```能成功运行和使用挂载的存储。
经过测试发现，```rbd-provisioner```可以限制存储大小，```pvc```配置```1G```，不可以拷入```1.9G```大小的文件。
```
[root@node1 ~]# docker ps | grep task-pv-pod
b6fe8d8b035d        ae513a47849c                                                                   "nginx -g 'daemon of…"   57 seconds ago       Up 54 seconds                                                                                      k8s_task-pv-container_task-pv-pod_default_0447737f-f9ef-11e8-8d69-525400331106_0
c11c6fd20db4        portus.teligen.com:5000/kubesprayns/gcr.io/google_containers/pause-amd64:3.1   "/pause"                 About a minute ago   Up 58 seconds                                                                                      k8s_POD_task-pv-pod_default_0447737f-f9ef-11e8-8d69-525400331106_0
[root@node1 ~]# 
[root@node1 ~]# du -sh Rong.tar.gz 
1.9G    Rong.tar.gz
[root@node1 ~]# 
[root@node1 ~]# docker cp Rong.tar.gz k8s_task-pv-container_task-pv-pod_default_0447737f-f9ef-11e8-8d69-525400331106_0:/usr/share/nginx/html/

Error response from daemon: Error processing tar file(exit status 1): write /Rong.tar.gz: no space left on device
[root@node1 ~]# 
```
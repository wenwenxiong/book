###kubespray启用localvolume存储模块
修改配置文件```addons.yaml```,文件位于```Rong/kubespray/inventory/rong/group_vars/k8s-cluster```文件夹下。修改配置项```local_volume_provisioner_*```。
```
# Local volume provisioner deployment
local_volume_provisioner_enabled: true
local_volume_provisioner_namespace: kube-system
local_volume_provisioner_storage_classes:
  - name: "{{ local_volume_provisioner_storage_class | default('local-storage') }}"
    host_dir: "{{ local_volume_provisioner_base_dir | default ('/mnt/disks') }}"
    mount_dir: "{{ local_volume_provisioner_mount_dir | default('/mnt/disks') }}"
#   - name: "local-ssd"
#     host_dir: "/mnt/local-storage/ssd"
#     mount_dir: "/mnt/local-storage/ssd"
#   - name: "local-hdd"
#     host_dir: "/mnt/local-storage/hdd"
#     mount_dir: "/mnt/local-storage/hdd"
#   - name: "local-shared"
#     host_dir: "/mnt/local-storage/shared"
#     mount_dir: "/mnt/local-storage/shared"
```
存储```localvolume```支持配置多个```mount```的目录。

###kubespray的localvolume存储模块使用
存储模块```localvolume```启动后，会自动检查配置中指定多个```mount```的目录下所有具有```mount```挂载的子目录，每个具有```mount```挂载的子目录会生成一个```kubernetes```中```PersistentVolume```对象。如果没有一个具有```mount```挂载的子目录，则创建```PersistentVolumeClaim```对象时会找不到对应可以绑定到它的```PersistentVolume```对象。也就无法使用该存储。

在生产环境推荐使用```Mount physical disks```创建```mount```挂载的子目录。
```
mkdir /mnt/disks/ssd1
mount /dev/vdb1 /mnt/disks/ssd1
```
存储大小的使用限制与实际的```PersistentVolume```对象有关，不受```PersistentVolumeClaim```对象控制。即使定义了一个```5G```的```PersistentVolumeClaim```对象，如果它选择了一个```20G```的```PersistentVolume```对象绑定，则存储的使用最多可以到```20G```。
测试示例如下，文件有```nginx-deplyment.yaml，nginx-pvc.yaml```。
内容如下
```
[root@node2 nginx]# cat nginx-deplyment.yaml 
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 5
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: portus.teligen.com:5000/kubesprayns/nginx:1.7.9
        imagePullPolicy: Always
        name: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
      volumes:
        - name: task-pv-storage
          persistentVolumeClaim:
            claimName: task-pv-claim
[root@node2 nginx]# 
[root@node2 nginx]# cat nginx-pvc.yaml 
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: task-pv-claim
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
[root@node2 nginx]#
```
环境中有在配置目录```/mnt/disks```下创建并挂载子目录```vol1```，```/dev/vdb1```具有20G大小。
```
[root@node2 nginx]# mount | grep vol1
/dev/vdb1 on /mnt/disks/vol1 type ext4 (rw,relatime,seclabel,data=ordered)
```
运行示例
```
[root@node2 nginx]# kubectl create -f nginx-deplyment.yaml -f nginx-pvc.yaml 
deployment.extensions/nginx created
persistentvolumeclaim/task-pv-claim created
```
查看```pvc，pv```，会发现都是```20G```的大小限制。
```
[root@node2 nginx]# kubectl get pvc
NAME             STATUS    VOLUME              CAPACITY   ACCESS MODES   STORAGECLASS    AGE
task-pv-claim    Bound     local-pv-d44e40d8   19Gi       RWO            local-storage   9m29s
task-pv-claim2   Pending                                                 local-storage   15s
[root@node2 nginx]# kubectl get pv
NAME                CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS    REASON   AGE
local-pv-d44e40d8   19Gi       RWO            Delete           Bound    default/task-pv-claim   local-storage            10m
[root@node2 nginx]#
```
上传一个超过```1G```大小的文件并不会报错。
```
[root@node2 ~]# kubectl exec -it nginx-564d49d77d-2fscc /bin/bash
root@nginx-564d49d77d-2fscc:/# 
root@nginx-564d49d77d-2fscc:/# cd /usr/share/nginx/html/
root@nginx-564d49d77d-2fscc:/usr/share/nginx/html# ls
CentOS-x64-kubespray-1708-JP.iso  lost+found
root@nginx-564d49d77d-2fscc:/usr/share/nginx/html# du -sh ./*
1.7G    ./CentOS-x64-kubespray-1708-JP.iso
16K     ./lost+found
root@nginx-564d49d77d-2fscc:/usr/share/nginx/html# exit
exit
```

总结：
1、该存储使用需要预先创建子目录，并挂在磁盘分区，并且磁盘分区大小需要仔细规划，因为一个磁盘分区会最终创建一个```pv```对象被使用，并且大小的限制跟磁盘分区大小相关。
2、挂载的子目录与节点相关，如果只有一个节点配置目录下有磁盘分区挂载子目录存在，则依赖它所创建的```pod```容器则只能运行在该节点上。
3、部署时```kubespray```没有为```localvolume```存储配置```default sc```，可以自己去修改```ansible template```文件。
###前提
OpenEBS依赖与iSCSI做存储管理，因此需要先确保您的集群上已有安装openiscsi。

安装iSCSI服务十分简单，不需要额外的配置，只要安装后启动服务即可。

在每个node节点上执行下面的命令：
```
yum -y install iscsi-initiator-utils
systemctl enable iscsid
systemctl start iscsid
```
###快速安装openebs
使用Operator运行OpenEBS服务：
```
wget https://raw.githubusercontent.com/openebs/openebs/master/k8s/openebs-operator.yaml
kubectl apply -f openebs-operator.yaml
```
使用默认或自定义的storageclass：
```
wget https://raw.githubusercontent.com/openebs/openebs/master/k8s/openebs-storageclasses.yaml
kubectl apply -f openebs-storageclasses.yaml
```
用到的镜像有：
```
openebs/m-apiserver:0.5.4
openebs/openebs-k8s-provisioner:0.5.4
openebs/jiva:0.5.4
openebs/m-exporter:0.5.4
```
测试
下面使用OpenEBS官方文档中的示例，安装Jenkins测试
```
wget https://raw.githubusercontent.com/openebs/openebs/master/k8s/demo/jenkins/jenkins.yml
kubectl apply -f jenkins.yml
```
查看pod
```
[root@node1 openebs]# kubectl get pod
NAME                                                             READY     STATUS    RESTARTS   AGE
jenkins-7bd46c4b67-t5lgn                                         1/1       Running   0          7m
maya-apiserver-9679b678-txvxt                                    1/1       Running   0          8m
openebs-provisioner-55ff5cd67f-qk56v                             1/1       Running   0          8m
pvc-4ba359b2-5984-11e8-b9da-525400764cc3-ctrl-6fb6d78d59-jk7nl   2/2       Running   0          7m
pvc-4ba359b2-5984-11e8-b9da-525400764cc3-rep-8677b4ccd4-4qct5    0/1       Pending   0          7m
pvc-4ba359b2-5984-11e8-b9da-525400764cc3-rep-8677b4ccd4-5ndnd    0/1       Pending   0          7m
pvc-4ba359b2-5984-11e8-b9da-525400764cc3-rep-8677b4ccd4-gsz4n    1/1       Running   0          7m
```
查看PV和PVC
```
[root@node1 openebs]# kubectl get pvc
NAME            STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       AGE
jenkins-claim   Bound     pvc-4ba359b2-5984-11e8-b9da-525400764cc3   5G         RWO            openebs-standard   7m
[root@node1 openebs]# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                   STORAGECLASS       REASON    AGE
pvc-4ba359b2-5984-11e8-b9da-525400764cc3   5G         RWO            Delete           Bound     default/jenkins-claim   openebs-standard             7m
```
可以看出，jenkins使用的pv为```pvc-4ba359b2-5984-11e8-b9da-525400764cc3```，它以pod的形式存在，其中```pvc-4ba359b2-5984-11e8-b9da-525400764cc3-ctrl-6fb6d78d59-jk7nl```为主存储节点，```pvc-4ba359b2-5984-11e8-b9da-525400764cc3-rep-8677b4ccd4-gsz4n```等为三个副存储节点（备份节点）。这里因为k8s环境只有一个节点，因此，有两个备份节点没有起来（备份节点之间有```affinity/anti-affinity```约束，不能两个在同一个节点上）。
查看Jenkins svc：
```
[root@node1 openebs]# kubectl get svc
NAME                                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
jenkins-svc                                         NodePort    172.20.237.139   <none>        80:32037/TCP        12m
kubernetes                                          ClusterIP   172.20.0.1       <none>        443/TCP             19h
maya-apiserver-service                              ClusterIP   172.20.57.26     <none>        5656/TCP            13m
pvc-4ba359b2-5984-11e8-b9da-525400764cc3-ctrl-svc   ClusterIP   172.20.129.254   <none>        3260/TCP,9501/TCP   12m
```
启动成功。Jenkins配置使用的是NodePort方式访问，现在访问集群中任何一个节点的Jenkins service的NodePort即可。
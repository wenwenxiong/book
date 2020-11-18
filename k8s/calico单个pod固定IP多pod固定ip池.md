## 原理

主要利用```calico```组件的两个```kubernetes```注解:

1)```cni.projectcalico.org/ipAddrs```；

2)```cni.projectcalico.org/ipv4pools```。

## 单个pod固定IP

利用注解```cni.projectcalico.org/ipAddrs```。

示例```yaml```配置如下

```
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1 # tells deployment to run 1 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
      annotations:
        "cni.projectcalico.org/ipAddrs": "[\"10.20.126.1\"]"
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```



## 多pod固定IP池

需要创建额外IP池（除了默认IP池）。利用注解```cni.projectcalico.org/ipv4pools```。

示例```yaml```配置如下

```
[root@k8s-master3 ~]# cat ippool1.yaml 
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: new-pool1
spec:
  blockSize: 31
  cidr: 10.21.0.0/31
  ipipMode: Never
  natOutgoing: true
[root@k8s-master3 ~]# calicoctl create -f ippool1.yaml 
Successfully created 1 'IPPool' resource(s)
[root@k8s-master3 ~]# cat ippool2.yaml 
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: new-pool2
spec:
  blockSize: 31
  cidr: 10.21.0.2/31
  ipipMode: Never
  natOutgoing: true
[root@k8s-master3 ~]# calicoctl create -f ippool2.yaml 
Successfully created 1 'IPPool' resource(s)
[root@k8s-master3 ~]# 
[root@k8s-master3 ~]# 
[root@k8s-master3 ~]# calicoctl get ippool
NAME                  CIDR           SELECTOR   
default-ipv4-ippool   10.20.0.0/16   all()      
new-pool1             10.21.0.0/31   all()      
new-pool2             10.21.0.2/31   all() 

root@k8s-master3 ~]# cat nginx.yaml 
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 4 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
      annotations:
        "cni.projectcalico.org/ipv4pools": "[\"new-pool1\",\"new-pool2\"]"
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
[root@k8s-master3 ~]# kubectl create -f nginx.yaml 
deployment.apps/nginx-deployment created
[root@k8s-master3 ~]# kubectl get pods -o wide
NAME                               READY   STATUS    RESTARTS   AGE   IP          NODE          NOMINATED NODE   READINESS GATES
nginx-deployment-f49447c5d-4k4px   1/1     Running   0          11s   10.21.0.0   k8s-master4   <none>           <none>
nginx-deployment-f49447c5d-5sbrx   1/1     Running   0          11s   10.21.0.2   k8s-master4   <none>           <none>
nginx-deployment-f49447c5d-flfb8   1/1     Running   0          11s   10.21.0.3   k8s-master4   <none>           <none>
nginx-deployment-f49447c5d-q4945   1/1     Running   0          11s   10.21.0.1   k8s-master4   <none>           <none>
```


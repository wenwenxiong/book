###kubespray启用IPVS与kube-router模块
修改配置文件```kube-cluster.yaml```,文件位于```Rong/kubespray/inventory/rong/group_vars/k8s-cluster```文件夹下。修改配置项```kube_network_plugin```和```kube_proxy_mode```。
```
kube_network_plugin: kube-router
kube_proxy_mode: ipvs
```
###kube-router配置负载均衡算法
网络插件```kube-router```使用```LVS```来实现负载均衡算法，使用注解```annotation```来标识。
默认使用轮询策略```round-robin```。
```
For least connection scheduling use:
kubectl annotate service my-service "kube-router.io/service.scheduler=lc"

For round-robin scheduling use:
kubectl annotate service my-service "kube-router.io/service.scheduler=rr"

For source hashing scheduling use:
kubectl annotate service my-service "kube-router.io/service.scheduler=sh"

For destination hashing scheduling use:
kubectl annotate service my-service "kube-router.io/service.scheduler=dh"
```
测试示例```echoserver```是一个显示自己主机名的TCP套接字应用，文件如下```echoserver-deployment.yaml，echoserver-svc.yaml```。
文件```echoserver-deployment.yaml```内容如下
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: echoservertcp
spec:
  replicas: 5
  template:
    metadata:
      labels:
        app: echoservertcp
    spec:
      containers:
      - image: portus.teligen.com:5000/kubesprayns/echoservertcppy:latest
        imagePullPolicy: Always
        name: echoserver
        ports:
        - containerPort: 31114
```
文件```echoserver-svc.yaml```内容如下
```
apiVersion: v1
kind: Service
metadata:
  name: echoservertcp
  annotations:
    kube-router.io/service.scheduler: ”rr“
spec:
  type: LoadBalancer
  ports:
  - port: 31114
    targetPort: 31114
    protocol: TCP
  selector:
    app: echoservertcp
```
运行示例应用
```
kubectl create -f echoserver-deployment.yaml -f echoserver-svc.yaml
```
查看```service```暴露的端口
```
[root@node2 httpechoserver]# kubectl get svc
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
echoservertcp   LoadBalancer   10.233.28.202   <pending>     31114:31202/TCP   17s
kubernetes      ClusterIP      10.233.0.1      <none>        443/TCP           11h
```
使用客户端脚本```echoclient.py```内容如下，配置了目标服务```IP```和端口。
```
import socket
import sys

# Create a TCP/IP socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Connect the socket to the port where the server is listening
server_address = ('portus.teligen.com', 31202)
print >>sys.stderr, 'connecting to %s port %s' % server_address
sock.connect(server_address)
try:
    
    # Send data
    message = 'M'
    print >>sys.stderr, 'sending "%s"' % message
    sock.sendall(message)

    # Look for the response
    amount_received = 0
    amount_expected = len(message)
    
    while amount_received < amount_expected:
        data = sock.recv(1600)
        amount_received += len(data)
        print >>sys.stderr, 'received "%s"' % data

finally:
    print >>sys.stderr, 'closing socket'
    sock.close()
```
运行命令如下
```
while true; do python echoclient.py; sleep 1; echo "********************"; done
```
可以发现每次响应的```pod```容器实例不同，以轮询方式提供。但是只能成功访问同一个节点的pod。
使用```ipvsadm```命令查看，发现以```node ip```作为负载均衡节点。
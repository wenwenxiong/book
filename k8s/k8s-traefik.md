###为什么要引入Ingress Controller
目前，kubernetes中的负载均衡大致可以分为以下几种机制，每种机制都有其特定的应用场景：

Service：直接用Service提供cluster内部的负载均衡，并借助cloud provider提供的LB提供外部访问
Ingress Controller：还是用Service提供cluster内部的负载均衡，但是通过自定义LB提供外部访问
Service Load Balancer：把load balancer直接跑在容器中，实现Bare Metal的Service Load Balancer
Custom Load Balancer：自定义负载均衡，并替代kube-proxy，一般在物理部署Kubernetes时使用，方便接入公司已有的外部服务

####service
Service是对一组提供相同功能的Pods的抽象，并为它们提供一个统一的入口。借助Service，应用可以方便的实现服务发现与负载均衡，并实现应用的零宕机升级。Service通过标签来选取服务后端，一般配合Replication Controller或者Deployment来保证后端容器的正常运行。

Service有三种类型：

ClusterIP：默认类型，自动分配一个仅cluster内部可以访问的虚拟IP
NodePort：在ClusterIP基础上为Service在每台机器上绑定一个端口，这样就可以通过<NodeIP>:NodePort来访问该服务
LoadBalancer：在NodePort的基础上，借助cloud provider创建一个外部的负载均衡器，并将请求转发到<NodeIP>:NodePort
另外，也可以将已有的服务以Service的形式加入到Kubernetes集群中来，只需要在创建Service的时候不指定Label selector，而是在Service创建好后手动为其添加endpoint。

####Ingress Controller
Service虽然解决了服务发现和负载均衡的问题，但它在使用上还是有一些限制，比如

对外访问的时候，NodePort类型需要在外部搭建额外的负载均衡，而LoadBalancer要求kubernetes必须跑在支持的cloud provider上面
Ingress就是为了解决这些限制而引入的新资源，主要用来将服务暴露到cluster外面，并且可以自定义服务的访问策略。

###安装traefik作为Ingress Controller

treafix可以创建基于Path based routing和Name based routing两种类型，不能混用，其中Path based routing类型的Ingress需要配置以下annotations。
```
annotations:
    kubernetes.io/ingress.class: traefik
    traefik.frontend.rule.type: PathPrefixStrip
```
以daemon set运行traefik。yaml文件可以在https://github.com/containous/traefik/tree/master/examples/k8s 找到。
```
kubectl create -f traefik-rbac.yaml
kubectl create -f traefik-ds.yaml
kubectl create -f traefik-ui.yaml
```
在机器上
```
echo "$(minikube ip) traefik-ui.minikube" | sudo tee -a /etc/hosts
```
或者在powerDns上配置一条记录。访问http://traefik-ui.minikube 就可以访问traefik的ui了。
###powerDNS与treafik结合使用

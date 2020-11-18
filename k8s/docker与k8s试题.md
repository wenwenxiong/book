一、填空题（每题2分，共40分）
1、docker是C/S架构，包括 ```docker服务端/docker守护进程``` 和 ```docker客户端``` 。

2、正在运行的Linux shell的容器中，输入 ```exit```命令退出shell和容器。

3、使用命令 ```docker container commit``` ，可以把提交容器并从中创建镜像。

4、创建docker镜像有两种方式，分别是 使用文件```dockerfile``` 和 命令```docker contianer commit``` 。

5、Docker映像的registry是 ```Docker Store```。

6、使用命令```docker image tag```，可以把标记（tag）docker镜像。

7、在docker swarm模式中，使用命令 ```docker node ls``` 显示集群中的节点数量。

8、在docker swarm模式中，有两种节点角色，分别为 ```manage``` 和 ```worker```。

9、构建docker镜像时，使用```FROM```指令指定一个基本镜像。

10、构建docker镜像时，使用```COPY```指令从我们的工作目录复制文件到镜像中。

11、构建docker镜像时，使用```WORKDIR```指令指定启动容器时的使用的目录。

12、Docker镜像由一系列```层```构建而成，每个```层```代表镜像的Dockerfile中的指令。

13、Docker在线培训教程中，演示多容器应用程序的示例是```voting投票应用程序```。

14、kubernetes集群可以部署在```物理机器```或者```虚拟机器```上。

15、kubernetes集群中，使用命令```kubectl cluster-info```查看集群的详细信息。

16、在kubernetes集群中，有两中节点角色，分别是 ```master``` 和 ```node```。

17、kubernets集群中的服务service有4中类型的type，分别是 ```clusterIP nodePort loadbalancer externalname ```。

18、部署kubernetes集群时，使用命令 ```kubeadm init``` 初始化集群，使用命令``` kubeadm join ```把节点加入集群。

19、kubernetes集群的命令行工具是 ```kubectl``` ，官方提供部署kubernetes集群的工具是 ```kubeadm```。

20、在教程中，使用命令```kubectl version```查看kubernetes集群客户端版本以及服务器版本。

二、选择题（每题2分，共30分）
1、使用什么命令列出docker容器？``` a```
a，docker container ls  b, docker container start c, docker container stop, d docker container run 
2、使用什么命令拉取docker镜像? ```a```
a，docker image pull b， docker image ls c， docker image push， d docker image rm
3、使用什么命令列出docker镜像? ```b```
a，docker image pull b， docker image ls c， docker image push， d docker image rm
4、使用什么命令运行docker容器? ```d```
a，docker container ls  b, docker container start c, docker container stop, d docker container run
5、使用什么命令删除docker容器? ```b```
a，docker container ls  b, docker container rm c, docker container stop, d docker container run
6、使用什么命令停止docker容器? ```c```
a，docker container ls  b, docker container start c, docker container stop, d docker container run
7、使用什么命令启动docker容器?``` b```
a，docker container ls  b, docker container start c, docker container stop, d docker container run
8、使用什么命令进入docker容器中? ```a```
a，docker container exec  b, docker container start c, docker container stop, d docker container run
9、容器镜像默认从哪里拉取？ ```d```
a，docker registry   b， something you have set up on your machine  c，there is no default d， docker store
10、使用什么命令可以检查docker 镜像的层数？```b```
a，docker image pull b， docker image inspect c， docker image push， d docker image rm
11、 docker swarm中stack是什么？```a```
a, a multi-service app running on a swarm b, a way of creating multiple nodes c,a method of using multiple compose files to run an app
12、使用什么命令查看kubernetes集群的节点信息？```a```
a，kubectl get node  b， kubectl get pod  c， kubectl get svc  d， kubectl get ing
13、使用什么命令查看kubernetes集群中的名字空间信息？```a```
a，kubectl get namespace  b， kubectl get pod  c， kubectl get svc  d， kubectl get ing
14、 使用什么命令查看kubernetes集群中的所有容器？```b```
a，kubectl get node --all-namespaces b， kubectl get pod --all-namespaces c， kubectl get svc --all-namespaces d， kubectl get pod
15、使用什么命令进入kubernetes中的容器中？ ```d```
a，kubectl get node  b， kubectl logs  c， kubectl run  d， kubectl exec

三、判断题（每题2分，共30分）

1、pod是kubernetes平台上的原子单元。    ```正确```

2、kubernets集群只能拥有一个master节点和一个或多个node节点。 ```错误```

3、kubernetes集群中service通常使用来向一组pod路由流量。 ```正确```

4、在kubernets集群中使用命令 kubectl scale 扩展缩放容器应用。 ```正确```

5、在kubernets集群中，滚动升级容器应用时，使用命令 kubectl rollout status来确认当前所有的pod是否都运行最新版本。 ```正确```



6、kubernetes集群api只能通过master节点上的kubectl工具访问。 ```错误```

7、kubernetes集群只能有一个namespace。 ```错误```

8、kubernetes集群中service使用Label和selectors匹配一组pod。  ```正确```

9、kubernetes集群提供service使得pod在kubernetes中重启、销毁、迁移时，不会影响到应用程序对外提供服务的功能。 ```正确```

10、kubernetes的工作节点宕机时，节点上的pod会消失，通过deployment部署的pod会在其他工作节点上重启，并且pod ip保持不变 ```错误```

11、kubernetes每个工作节点至少运行kubelet和docker服务 ```正确```

12、kubernetes中pod是一组一个或多个应用程序容器，包括共享存储卷，IP地址以及有关如何运行它们的信息。```正确```

13、kubernetes协调并构建出一个高可用的计算机集群，并将它们连接起来作为一个单元工作。 ```正确```

14、kubernetes以更高效的方式，主导了自动化跨群集的应用程序容器的分发和调度  ```正确```

15、kubernetes是google团队发起的开源项目，主要实现语言为go语言 ```正确```
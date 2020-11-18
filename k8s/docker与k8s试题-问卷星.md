
一、选择题（每题4分，共60分）
1、使用什么命令列出docker容器？``` a```
a，docker container ls
b，docker container start
c， docker container stop
d， docker container run

2、使用什么命令拉取docker镜像? ```a```
a，docker image pull
b， docker image ls
c， docker image push
d， docker image rm

3、使用什么命令列出docker镜像? ```b```
a，docker image pull
b， docker image ls
c， docker image push
d， docker image rm

4、使用什么命令运行docker容器? ```d```
a，docker container ls
b，docker container start
c，docker container stop
d， docker container run

5、使用什么命令删除docker容器? ```b```
a，docker container ls
b，docker container rm
c，docker container stop
d， docker container run

6、使用什么命令停止docker容器? ```c```
a，docker container ls
b，docker container start
c，docker container stop
d， docker container run

7、使用什么命令启动docker容器?``` b```
a，docker container ls
b，docker container start
c，docker container stop
d， docker container run

8、使用什么命令进入docker容器中? ```a```
a，docker container exec
b，docker container start
c，docker container stop
d， docker container run

9、容器镜像默认从哪里拉取？ ```d```
a，docker registry
b， something you have set up on your machine
c，there is no default
d， docker store

10、使用什么命令可以检查docker 镜像的层数？```b```
a，docker image pull
b， docker image inspect
c， docker image push
d， docker image rm

11、 docker swarm中stack是什么？```a```
a，a multi-service app running on a swarm
b，a way of creating multiple nodes
c，a method of using multiple compose files to run an app
d，a container

12、使用什么命令查看kubernetes集群的节点信息？```a```
a，kubectl get node
b， kubectl get pod
c， kubectl get svc
d， kubectl get ing

13、使用什么命令查看kubernetes集群中的名字空间信息？```a```
a，kubectl get namespace
b， kubectl get pod
c， kubectl get svc
d， kubectl get ing

14、 使用什么命令查看kubernetes集群中的所有容器？```b```
a，kubectl get node --all-namespaces
b， kubectl get pod --all-namespaces
c， kubectl get svc --all-namespaces
d， kubectl get pod

15、使用什么命令进入kubernetes中的容器中？ ```d```
a，kubectl get node
b， kubectl logs
c， kubectl run
d， kubectl exec

二、判断题（每题2分，共40分）
1、正在运行的Linux shell的容器中，输入 ```exit```命令退出shell和容器。 正确

2、创建docker镜像有两种方式，分别是 使用文件```dockerfile``` 和 命令```docker container commit``` 。 ```正确```

3、构建docker镜像时，使用```FROM```指令指定一个基本镜像。```正确```

4、构建docker镜像时，使用```COPY```指令从我们的工作目录复制文件到镜像中。```正确```

5、构建docker镜像时，使用```WORKDIR```指令指定启动容器时的使用的目录。```正确```

6、```pod```是```kubernetes```平台上的原子单元。    ```正确```

7、```kubernetes```集群只能拥有一个```master```节点和一个或多个```node```节点。 ```错误```

8、```kubernetes```集群中```service```通常使用来向一组```pod```路由流量。 ```正确```

9、在```kubernetes```集群中使用命令``` kubectl scale``` 扩展缩放容器应用。 ```正确```

10、在```kubernetes```集群中，滚动升级容器应用时，使用命令 ```kubectl rollout backup```来确认当前所有的```pod```是否都运行最新版本。 ```错误```

11、```kubernetes```集群可以部署在```物理机器```或者```虚拟机器```上。```正确```

12、```kubernetes```集群中，使用命令```kubectl cluster-info```查看集群的详细信息。 ```正确```

13、```kubernetes```集群中的服务```service```有4中类型的```type```，分别是 ```clusterIP nodePort loadbalancer externalname ```。 ```正确```

14、部署```kubernetes```集群时，使用命令 ```kubeadm init``` 初始化集群，使用命令``` kubeadm join ```把节点加入集群。 ```正确```

15、```kubernetes```集群的命令行工具是 ```kubectl``` ，官方提供部署kubernetes集群的工具是 ```kubeadm```。 ```正确```

16、```kubernetes```中```service```是一组一个或多个应用程序容器，包括共享存储卷，```IP```地址以及有关如何运行它们的信息。```错误```

17、```kubernetes```集群只能有一个```namespace```。 ```错误```

18、在教程中，使用命令```kubectl version```查看```kubernetes```集群客户端版本以及服务器版本。 ```正确```

19、```kubernetes```每个工作节点至少运行```kubelet```和```docker```服务。 ```正确```

20、```kubernetes```集群提供```service```使得```pod```在```kubernetes```中重启、销毁、迁移时，不会影响到应用程序对外提供服务的功能。 ```正确```

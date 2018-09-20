###k8s deployment service默认配置

应用部署在k8s上，首先想到的是应用k8s的默认service模式配置。
![matrixgatenet](./matrixgateimages/matrixgatenet.png "matrixgatenet")
应用通过service向集群内部（ClusterIP）和集群外部（NodePort）暴露服务。k8s中的其他应用通过kube-dns提供的dns解析功能，访问servicename:port即可访问service后面的pod的服务。这需要两个应用服务之间的交互不需要记录对方的hostname或者IP，例如，假如应用服务通过socket连接到另一个应用服务，并通过记录他的IP或者Hostname，以便下次连接时作为验证对方。这样的话会出现servicename:port与podname:port或者podIP:port不符。

###k8s hostnetwork配置

解决servicename:port与podname:port或者podIP:port不符的问题，可以把被验证的应用使用hostnetwork。
应用与宿主机共享网络空间，也就是k8s节点的IP，端口占用与宿主机一样。这样应用的IP就是宿主机的IP，与另一个应用的连接也不经过service IP这一层。另一个应用仍然采用kubernetes的pod网络模型，为使使用hostnetwork的应用能对其他应用的service name进行DNS解析，需要设置参数hostNetwork: true和dnsPolicy: ClusterFirstWithHostNet。

该配置的缺点：应用的网络与宿主机一样，需保证应用需要监听的网络端口在宿主机上没有被占用。并且无法使用容器漂移，动态伸缩特性。

###k8s headless service配置

解决servicename:port与podname:port或者podIP:port不符的问题，还有一种方法，就是把被验证的应用配置使用headless service。headless service即是把service type为ClusterIP的service配置spec.clusterIP为None，这样的话就不会为其分配serviceIP，kube-dns解析servicename:port就会变成podIP:port。在把servicename配置和pod的hostname一样，这样的kube-dns解析servicename:port就与podhostname:port一样。达到了获得实际podIP的目的。相比hostnetwork配置，这个配置没有它所具有的缺点。

###k8s statefulset headless service配置

上面提到headless service可以解决servicename:port与podname:port或者podIP:port不符的问题。这是针对一个应用pod在service后面，另一个应用pod在service的前面。现在假设同一个service后面的pod之间需要获悉对方的hostname或者podIP进行通信，怎么办？例如，storm的worker组件之间需要通过hostname通信，我们想要能快速伸缩worker组件，因此把所有的worker放到同一个service后面。
一种解决办法就是使用statefulset加headless service配置，这种配置kube-dns可以解析
podname.servicename.namespacename.svc.cluster.local对应到podIP。在同一个statefulset的pod的podname都是podname-0,podname-1,podname-2等数字递增的命名。对于podname-0可以解析
podname-1.servicename.namespacename.svc.cluster.local到podname-1的podIP。
在podname-0中使用命令
```
nslookup podname-1
```
会解析podname-1.namespacename.svc.cluster.local。因此需要我们修改/etc/resolv.conf
文件
在search 后面加上 servicename.namespacename.svc.cluster.local，这样就达到了解析到
podname-1.servicename.namespacename.svc.cluster.local的目的。
具体做法就是在statefulset的pod配置中加入lifecycle的postStart配置
```
        lifecycle:
          postStart:
            exec:
              command:
              - /bin/sh
              - -c
              - >
                if [ -z "$(grep servicename /etc/resolv.conf)" ]; then
                  sed "s/^search \([^ ]\+\)/search servicename.\1 \1/" /etc/resolv.conf > /etc/resolv.conf.new;
                  cat /etc/resolv.conf.new > /etc/resolv.conf;
                  rm /etc/resolv.conf.new;
                fi;
```
至此，headless service后面的statefulset的pod之间可以相互通信了。

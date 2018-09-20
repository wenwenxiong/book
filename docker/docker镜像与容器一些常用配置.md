###配置locale
如果docker镜像内的os是debain/ubuntu系列，则在dockerfile配置如下
```
FROM ubuntu:13.10

# Set the locale
RUN locale-gen en_US.UTF-8  
ENV LANG en_US.UTF-8  
ENV LANGUAGE en_US:en  
ENV LC_ALL en_US.UTF-8  
```
docker镜像内的os是foreda/centos系列，则在dockerfile配置如下
```
FROM centos:6.9

# Set the locale
RUN localedef -c -i en_US -f UTF-8 en_US.UTF-8 --quiet
ENV LANG en_US.UTF-8
ENV LANG_ALL en_US.UTF-8
```
###配置/etc/hosts
有以下方法
####开启时加参数
开启容器时候添加参数—add-host machine:ip可以实现hosts修改，在容器中可以识别machine主机。缺点是很多个节点的话命令会很长，有点不舒服（当然，你可以写一个脚本了）。

####进入容器修改
进入容器后可以用各种编辑器编辑修改hosts文件，这样是最简单粗暴的方式，缺点就是每次启动容器都要修改一次。

####dnsmasq
在每个容器中开启dnsmasq开负责局域网内的容器节点主机名解析，但是这种方式在跨物理机的时候，好像不是很方便（具体情况还没有实践过，挖坑日后再填）。可以参考一下kiwenlau的工作

####修改容器hosts查找目录
我们可以想办法让容器开启时候，不去找/etc/hosts文件，而是去找我们自己定义的hosts文件，下面是一个Dockerfile实例
```
FROM ubuntu:14.04
RUN cp /etc/hosts /tmp/hosts #路径长度最好保持一致
RUN mkdir -p -- /lib-override && cp /lib/x86_64-linux-gnu/libnss_files.so.2 /lib-override
RUN sed -i 's:/etc/hosts:/tmp/hosts:g' /lib-override/libnss_files.so.2
ENV LD_LIBRARY_PATH /lib-override
RUN echo "192.168.0.1 node1" >> /tmp/hosts #可以随意修改/tmp/hosts了
...
```
这样修改后，容器每次启动时就不去找/etc/hosts了，而是我们自己定义的/tmp/hosts，可以随心所欲在其中修改了

####dockerfile中配置
在dockerfile中RUN命令把内容加入到/etc/hosts。
```
...
ADD hosts /usr/hosts
RUN cat /usr/hosts >> /etc/hosts
```
###配置ulimit
对全局的容器配置ulimit
修改/etc/sysconfig/docker（不同os查找不同的配置文件）加入OPTIONS配置参数。
```
OPTIONS="--ulimit nofile=1280:2560 --ulimit nproc=256:512"
```
对单个容器配置ulimit
在运行容器加上--ulimit参数，例如
```
docker run -i -t --rm myapp:2.0 --ulimit nofile=128:256 --ulimit nproc=32:64
```
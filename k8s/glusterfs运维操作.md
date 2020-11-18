### glusterfs术语
1*集群(Cluster) : 它是相互连接的一组主机，这些主机协同工作共同完成某一个功能，对外界来说就像一台主机一样。
2*可信的存储池(Trusted Storage Pool)：它是存储服务器所组成的可信网络。
3*服务器(Server)：实际存储数据的服务器。
4*卷(Volume)：Brick的逻辑集合。
5*分卷(SubVolume)：由多个Brick逻辑构成的卷，它是其它卷的子卷。比如在分布复制卷中每一组复制的Brick就构成了一个复制的分卷，而这些分卷又组成了分布卷。
6*块(Brick)：存储的基本单元，表现为服务器导出的一个目录。
7*客户端(Client)：挂载Volume的主机。

### 常用运维命令

cli工具gluster可以对```node、volume```操作。

###heketi-cli查看

在k8s集群中部署的```glusterfs```使用```heketi-cli```查看```volume```信息。
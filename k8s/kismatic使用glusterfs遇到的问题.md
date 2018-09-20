1、kismatic已经不在支持nfs的安装，需要用户自己安装nfs服务，然后把ip，path信息填入kismatci-cluster.yaml文件中。
2、kismatic中配置nfs和glusterfs是个两中存储的使用，它们之间没有交互使用。
3、在kismatic安装glusterfs后，需要手动创建和启动glusterfs volume。注意在操作过程中注意防火墙的配置
ubuntu16.04下关闭防火墙的方法为
```
systemctl stop ufw
systemctl disable ufw
```
4、在kismatic安装glusterfs后，gluster-healthz pod会报错，查看日志发现除了111端口外，其他端口都未处于监听的状态。解决方法： 在创建和启动glusterfs volume后，使用命令
```
gluster volume info xxx
```
查看volume信息，发现volume的nfs.disable处于on状态，使用命令
```
gluster volume set xxx nfs.disable off
```
即使开启其他相关端口
5、创建pv pvc pod使用glusterfs的volume，需要在k8s的节点上安装glusterfs client，否则会出现glusterfs类型无法找到的错误。ubuntu安装glusterfs的命令为
```
apt-get update
apt-get install -y glusterfs-client
```
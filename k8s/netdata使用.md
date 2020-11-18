###netdata使用


###netdata启动与停止
组件```netdata```通过```systemd```管理。
启动与停止```netdata```。
```
systemctl start netdata
systemctl stop netdata
```
重启```netdata```。
```
systemctl restart netdata
```
###netdata配置

1、配置```netdata```数据存储周期

组件```netdata```默认只保留```1```小时数据，新数据覆盖旧数据。
每小时数据，```netdata```需要大概```25MB RAM```内存。
配置文件```/etc/netdata/netdata.conf```中配置项```history```可以设置```netdata```保留数据时长。
```
[global]
    history = SECONDS
```
一小时是```3600```秒，配置值为```HOURS * 3600```。



###netdata日志文件
组件```netdata```日志文件默认存放在```/var/log/netdata/```文件夹下，包含三个文件```error.log，access.log，debug.log```。通过配置文件```/etc/netdata/netdata.conf```可以开启和关闭三个文件日志。默认开启```error.log，access.log```文件，```debug.log```文件当```netdata```的```debugging/tracing```开启时则开启。

###netdata显示docker容器名称
组件```netdata```没有显示```docker```容器名称，显示```docker```容器```id```。
首先，通过命令```grep docker /var/log/netdata/error.log```找出错误日志。如果错误日志类似以下信息。
```
docker: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post http://%2Fvar%2Frun%2Fdocker.sock/v1.26/containers/create: dial unix /var/run/docker.sock: connect: permission denied.
```
则是运行```netdata```组件的用户```netdata```没有访问```docker```的权限，执行以下命令为```netdata```用户赋予权限。
```
sudo usermod -a -G docker netdata
```
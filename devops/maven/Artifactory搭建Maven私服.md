###使用docker部署Artifactory

1、获取 artifactory-oss 镜像
```
$ docker pull docker.bintray.io/jfrog/artifactory-oss
```
2、创建数据卷

例如在 ```~/docker/volume/artifactory ```路径下执行
```
$ docker volume create data_artifactory
```
3、启动容器
```
$ docker run --name any-artifactory -d \
-v data_artifactory:/var/opt/jfrog/artifactory \
-p 8081:8081 docker.bintray.io/jfrog/artifactory-pro
```
###Artifactory配置
登录到artifactory的web ui界面8081端口。

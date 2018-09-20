参考网址：https://kubernetes.io/docs/tools/kompose/user-guide/
Kompose是一个转换工具，可以将docker-compose编排docker-compose.yaml文件转换为kubernetes或者OpenShift编排文件。
###Kompose安装
直接下载二进制文件安装
```
# Linux
curl -L https://github.com/kubernetes/kompose/releases/download/v1.1.0/kompose-linux-amd64 -o kompose

# macOS
curl -L https://github.com/kubernetes/kompose/releases/download/v1.1.0/kompose-darwin-amd64 -o kompose

# Windows
curl -L https://github.com/kubernetes/kompose/releases/download/v1.1.0/kompose-windows-amd64.exe -o kompose.exe

chmod +x kompose
sudo mv ./kompose /usr/local/bin/kompose
```
###Kompose使用
####转换Docker-compose文件到kubernetes
docker-compose.yml和docker-guestbook.yml都是docker-compose的编排文件。
```
kompose -f docker-compose.yml -f docker-guestbook.yml convert
```
####转换Docker-compose文件到openshift
```
kompose --provider openshift --file buildconfig/docker-compose.yml convert
```
####转换Docker-compose文件后并且直接运行yml文件
```
kompose --file ./examples/docker-guestbook.yml up
```
转换为OpenShift
```
kompose --file ./examples/docker-guestbook.yml --provider openshift up
```
可以使用命令Kompose down删除运行的k8s服务
```
kompose --file docker-guestbook.yml down
```
####build和push docker镜像
当docker-compose文件包含build关键字，kompose会构建镜像并且推送镜像
```
version: "2"

services:
    foo:
        build: "./build"
        image: docker.io/foo/bar
```
```
$ kompose up
# Disable building/pushing Docker images
$ kompose up --build none

# Generate Build Config artifacts for OpenShift
$ kompose up --provider openshift --build build-config
```
###转换支持
转换成json格式文件
```
kompose convert -j
```
把*-deployment.yaml生成为*-replicationcontroller.yaml
```
$ kompose convert --replication-controller
$ kompose convert --replication-controller --replicas 3
```
把*-deployment.yaml生成为*-daemonset.yaml
```
$ kompose convert --daemon-set
```
转换后生成Chart（Helm使用）
```
$ kompose convert -c
$ tree docker-compose/
docker-compose
├── Chart.yaml
├── README.md
└── templates
    ├── redis-deployment.yaml
    ├── redis-svc.yaml
    ├── web-deployment.yaml
    └── web-svc.yaml
```
Kompose支持在docker-compose.yml增加label支持自定义的配置
kompose可以配置转换后的service的type为NodePort
```
version: "2"
services:
  nginx:
    image: nginx
    dockerfile: foobar
    build: ./foobar
    cap_add:
      - ALL
    container_name: foobar
    labels:
      kompose.service.type: nodeport
```
kompose可以配置暴露给Ingress Controller的hostname。
![kompose-expose](./images/kompose-expose.png "kompose-expose")
kompose可以镜像的restart取值决定是否是pod.yml。
![kompose-restart](./images/kompose-restart.png "kompose-restart")
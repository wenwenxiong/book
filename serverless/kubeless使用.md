###kubeless 安装
安装kubeless分为三个步骤
1、安装kubeless cli工具
```
wget https://github.com/kubeless/kubeless/releases/download/v1.0.0-alpha.1/kubeless_linux-amd64.zip
```
2、创建kubeless 名字空间
```
kubectl create namespace kubeless
```
3、在kubenetes集群中部署kubeless
```
export RELEASE=v1.0.0-alpha.1
kubectl create -f https://github.com/kubeless/kubeless/releases/download/$RELEASE/kubeless-$RELEASE.yaml
```
###kubeless serverless插件工具安装
kubeless serverless插件名为serverless，通过以下命令安装
```
npm install serverless -g
```
###kubeless ui安装
在kubernetes集群安装kubeless的ui
```
kubectl create -f https://raw.githubusercontent.com/kubeless/kubeless-ui/master/k8s.yaml
```

###kubeless使用
使用kubeless cli工具可以创建三种类型的function
* http triggered (function will expose an HTTP endpoint)
* pubsub triggered (function will consume event on a specific topic; a running kafka cluster on your k8s is required)
* schedule triggered (function will be called on a cron schedule)
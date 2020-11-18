###devops开源工具集-cicd
* gitlab-ce (gitlab-ci gitlab-runner)
* jenkins (pipeline， kubernetes插件)
* Travis (只能与github配合使用)
* spinnaker (kubernetes cd, jenkins ci)
* fabric8 (针对对微服务，尤其是java程序，mamageiq，redhat出品)
* openshift
* contianerum( 内部集成Travis)

###应用市场（行业解决方案）工具集
* helm chart, monicular
* helm chart, kubeapps
* helm chart, containerum

###开源paas解决方案
* openshift（基于kubernetes，感觉最优）
* dokku
* deis
* Heroku
* flynn
* Cloud Foundry2.0(Pivotal Cloud Foundry, garden(warden), buildpack vs dockerfile)
* cfcr( Cloud Foundry Container Runtime,2018.9.27 新东西，官方文档资料还不完善)

All this means that with PCF, your build artifact is your native deployment artifact, while in Kubernetes your build artifact is a docker image. With Kubernetes, you need to define the template for this docker image yourself in a Dockerfile, while in PCF you get this template automatically from a buildpack.



###fabric8 安装到kubernetes（目前只有minikube 联网环境使用成功）
1,在能访问到目标kubernetes的机器上执行
```
helm repo add fabric8 https://fabric8.io/helm
helm install --name fabric8 fabric8/fabric8-platform
```
###fabric8 helm安装到kubernetes
1,下载gofabric8
```
curl -sS https://get.fabric8.io/download.txt | bash
export PATH=$PATH:$HOME/.fabric8/bin
```
2,在能访问到目标kubernetes的机器上执行
```
gofabric8 deploy -y
gofabric8 secrets -y
gofabric8 validate
gofabric8 pull cd-pipeline
```
3,删除fabric8
```
kubectl delete deployment -l provider=fabric8
kubectl delete rc -l provider=fabric8
kubectl delete rs -l provider=fabric8
kubectl delete service -l provider=fabric8
kubectl delete secret -l provider=fabric8
kubectl delete ingress -l provider=fabric8
kubectl delete configmap -l provider=fabric8
kubectl delete configmap -l provider=fabric8.io
kubectl delete sa -l provider=fabric8
kubectl delete ns -l provider=fabric8
```
###fabric8 安装到minikube
1,下载gofabric8
```
curl -sS https://get.fabric8.io/download.txt | bash
export PATH=$PATH:$HOME/.fabric8/bin
```
2,启动minikube

3,执行以下命令安装fabric8到minikube
```
gofabric8 start --memory=8192 --cpus=4
```
###fabric8 安装到minishift
1,下载gofabric8
```
curl -sS https://get.fabric8.io/download.txt | bash
export PATH=$PATH:$HOME/.fabric8/bin
```
2,启动minishift

3,执行以下命令安装fabric8到minishift
```
gofabric8 start --minishift --memory=8192 --cpus=4
```
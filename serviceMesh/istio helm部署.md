###kismatic部署kubernetes
kismatic在部署kubernetes时，把```package_manager```为```helm```不启用。配置如下
```
package_manager:
    disable: true

    # Options: 'helm'.
    provider: helm
    options:
      helm:
        namespace: kube-system
```
部署kubernetes完成后，部署helm
```
kubectl create -f install/kubernetes/helm/helm-service-account.yaml
```
下载```tiller```的docker image。
```
#!/bin/bash

set -x
dockerimages=(
gcr.io/kubernetes-helm/tiller:v2.8.2
)

j=1
for i in ${dockerimages[@]}
do
    echo $i
    echo $j

    docker pull $i && docker save $i -o $j.tar && xz $j.tar
    docker rmi $i
    let j+=1
done
set +x
```
初始化```helm```
```
helm init --service-account tiller
```
###helm安装istio
默认的helm安装istio命令为
```
helm install install/kubernetes/helm/istio --name istio
```
###helm删除istio
卸载istio
```
helm delete --purge istio
```
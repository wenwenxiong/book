###gitlab 安装helm插件源码所在
源码位置位于
```
/opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/kubernetes/helm
```
###gitlab通过helm安装的其他插件源码所在
源码位置位于
```
/opt/gitlab/embedded/service/gitlab-rails/app/models/clusters/applications
```
###gitlab通过helm安装其他插件的value配置文件源码所在
源码位置位于
```
/opt/gitlab/embedded/service/gitlab-rails/vendor
```
###修改

####修改安装helm的pod中到外网下载helm二进制包
在helm插件源码下base_command.rb文件中，通过
```
wget -q -O - https://kubernetes-helm.storage.googleapis.com/helm-v...
```
去墙外下载helm二进制包
修改为从内网下载
```
...
def generate_script
          <<~HEREDOC
            set -eo pipefail
            ALPINE_VERSION=$(cat /etc/alpine-release | cut -d '.' -f 1,2)
            echo http://mirror.clarkson.edu/alpine/v$ALPINE_VERSION/main >> /etc/apk/repositories
            echo http://mirror1.hs-esslingen.de/pub/Mirrors/alpine/v$ALPINE_VERSION/main >> /etc/apk/repositories
            #apk add -U ca-certificates openssl >/dev/null
            wget -q -O - http://portus.teligen.com:8888/helm-v#{Gitlab::Kubernetes::Helm::HELM_VERSION}-linux-amd64.tar.gz | tar zxC /tmp >/dev/null
            mv /tmp/linux-amd64/helm /usr/bin/
          HEREDOC
        end
...
```
###修改helm init语句从外网下载tiller
在helm插件源码下```init_command.rb```，```install_command.rb```把源码中
```
helm init >/dev/null
```
改为
```
helm init --stable-repo-url http://portus.teligen.com:8988 --tiller-image portus.teligen.com:5000/kubesprayns/gcr.io/kubernetes-helm/tiller:v2.7.0 >/dev/null
```
###更改安装helm的pod容器镜像为私有仓库中镜像
在helm插件源码下```pod.rb```中，把源码中
```
alpine:3.7
```
改为
```
portus.teligen.com:5000/kubesprayns/alpine:3.7
```
###修改k8s下一键安装gitlab-runner
在helm安装的其他插件源码所在的```runner.rb```文件中改为
```
def repository
  'http://portus.teligen.com:8988'
end
```
然后手动下载chart文件放到自己私有的chart repo中
```
helm repo add runner https://charts.gitlab.io
helm fetch runner/gitlab-runner
curl --data-binary "@gitlab-runner-0.1.33.tgz" http://portus.teligen.com:8988/api/charts
```
修改gitlab-runner chart中image的名称为私有镜像仓库镜像名称。

在通过helm安装其他插件的value配置文件源码所在的```runner/values.yaml```文件中把rbac打开
```
...
rbac:
  create: true
  clusterWideAccess: true
...
```
###修改k8s下一键安装ingressController
手动下载chart文件放到自己私有的chart repo中
```
helm fetch stable/nginx-ingress
curl --data-binary "@nginx-ingress-0.28.1.tgz" http://portus.teligen.com:8988/api/charts
```
修改nginx-ingress chart中image的名称为私有镜像仓库镜像名称。

在通过helm安装其他插件的value配置文件源码所在的```ingress/values.yaml```文件中把rbac打开
```
...
rbac:
  create: true
  createRole: true
  createClusterRole: true
...
```

###修改k8s下一键安装prometheus
手动下载chart文件放到自己私有的chart repo中
```
helm fetch stable/prometheus
curl --data-binary "@prometheus-6.7.3.tgz" http://portus.teligen.com:8988/api/charts
```
修改prometheus chart中image的名称为私有镜像仓库镜像名称。

在通过helm安装其他插件的value配置文件源码所在的```prometheus/values.yaml```文件中把rbac和其他插件打开
```
alertmanager:
  enabled: true

kubeStateMetrics:
  enabled: true

nodeExporter:
  enabled: true

pushgateway:
  enabled: true

rbac:
  create: true
...
```
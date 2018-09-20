istio是一个servicemesh服务网格的开源工具
###minishift配置istio profile
考虑istio组件规模，配置minishift的profile如下
```
minishift profile set servicemesh
minishift config set memory 8GB
minishift config set cpus 3
minishift config set openshift-version v3.7.0
minishift config set network-device virbr11
minishift config set network-ipaddress 192.192.189.10
minishift config set network-netmask 24
minishift config set network-gateway 192.192.189.1
minishift addon enable admin-user
minishift addon enable anyuid
minishift start
```
3Core8G的资源配置，使用OpenShift 版本为3.7.x及以后，启用了admin-user和anyuid插件。
###配置```ISTIO_HOME```环境变量
下载最新的istio
```
curl -L https://git.io/getLatestIstio | sh -
```
添加```ISTIO_HOME```环境变量
###安装istio插件
运行命令
```
git clone https://github.com/minishift/minishift-addons
minishift addon install ./minishift-addons/add-ons/istio
```
###minishift部署istio
运行命令
```
minishift addon apply istio --addon-env ISTIO_HOME=$ISTIO_HOME
```
ISTIO_HOME变量的配置至此可以体现。
###minishift删除istio组件
运行命令
```
minishift addon remove istio
```
###minishift卸载istio插件
运行命令
```
minishift addon uninstall istio
```

遇到问题
minishift部署istio时报错```...exists```。
修改```add-ons/istio/istio.addon```文件把
```
oc create -f #{ISTIO_HOME}/install/kubernetes/istio.yaml
```
修改为
```
oc apply -f #{ISTIO_HOME}/install/kubernetes/istio.yaml
```
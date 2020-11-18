###简介
组件```keel```是一个自动发现```kubernetes```集群中```deployment,statefulset,daemonset```中镜像在仓库中有更新并自动更新已部署的镜像。还可以应用到```helm```，自动发现```chart```仓库中版本有更新则自动更新已部署的```release```。

###自动化安装keel到k8s集群
安装包```keel.tar.gz```通过```ansible```自动化安装。
安装前提条件：云平台套件```rong```已经部署
安装步骤：
1）配置目标机器
配置文件为```keel/hosts.ini ```。

2）运行```ansible```命令执行安装
```
tar -zxvf keel.tar.gz
cd keel
ansible-playbook -i hosts.ini keel.yaml
```

###测试```poll```触发更新
####配置kubernetes原生对象
触发方法为```poll```需要加上目标（```deployment,statefulset,daemonset```）的```annotations```如下配置。
```
annotations:
keel.sh/trigger: poll     # 触发策略指定为poll
keel.sh/pollSchedule: "@every 1m"   #触发调度频率
```
此外，还要加入触发策略的配置。
```
keel.sh/policy: minor
```
策略值可以取值如下：
镜像```tag```严格遵循```semver的x.x.x```规则，主版本号，次版本号，补丁版本。
```
all: update whenever there is a version bump or a new prerelease created (ie: 1.0.0 -> 1.0.1-rc1)
major: update major & minor & patch versions
minor: update only minor & patch versions (ignores major)
patch: update only patch versions (ignores minor and major versions)
force: force update even if tag is not semver, ie: latest, optional label: keel.sh/match-tag=true which will enforce that only the same tag will trigger force update.
```
```glob: use wildcards to match versions```

例如：
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata: 
  name: wd
  namespace: default
  labels: 
      name: "wd"
  annotations:
      keel.sh/policy: "glob:build-*"  # <- build-1, build-2, build-foo will match this. 
```

```regexp: use regular expressions to match versions```
例如：
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata: 
  name: wd
  namespace: default
  labels: 
      name: "wd"
  annotations:
      keel.sh/policy: "regexp:^([a-zA-Z]+)$"
```
####配置helm chart
如果目标是```helm```的```chart```，则在```values.yaml```文件中配置```keel```相关的配置项，例如：
```
replicaCount: 1
image:
  repository: karolisr/webhook-demo
  tag: "0.0.8"
  pullPolicy: IfNotPresent
service:
  name: webhookdemo
  type: ClusterIP
  externalPort: 8090
  internalPort: 8090

keel:
  # keel policy (all/major/minor/patch/force)
  policy: all
  # trigger type, defaults to events such as pubsub, webhooks
  trigger: poll
  # polling schedule
  pollSchedule: "@every 2m"
  # images to track and update
  images:
    - repository: image.repository
      tag: image.tag
      imagePullSecret: my-secret-name
```

注1、```imagePullSecret```，如果镜像仓库配置了```用户名密码```认证，则需要在目标加上该配置获取仓库镜像信息。
注2、```hostAliases```，如果镜像仓库的```domian```名无法解析，则需要在目标加上该配置设置域名到```IP```的解析。
注3、镜像```tag```严格遵循```semver的x.x.x```规则，主版本号，次版本号，补丁版本。
注4、检测到镜像更新，会重建容器以更新镜像（服务有一段时间不可用）。

###测试```webhook```触发更新
组件```keel```的默认触发方法为```webhook```，内网环境下使用的是```docker registry```和```harbor```镜像仓库。它们的```webhook```配置是类似的（```harbor```后端使用的是```docker-registry```存储镜像，参考```https://github.com/goharbor/harbor/issues/2448```）。
都是修改```docker-registry```的配置文件```/etc/registry/config.yaml```文件。
在文件中的```notifications```下增加名为```keel```的```endpoints```条目配置。
```
...
...
...
notifications:
  endpoints:
  - name: harbor
    disabled: false
    url: http://ui:8080/service/notifications
    timeout: 3000ms
    threshold: 5
    backoff: 1s
  - name: keel
    disabled: false
    url: http://192.168.124.61:30162/v1/webhooks/registry
    timeout: 3000ms
    threshold: 5
    backoff: 1s
```
名为```keel```的```endpoints```条目配中的```url```取值则是```keel pod 9300```对外暴露端口服务加上针对```docker-registry```的特定后缀```/v1/webhooks/registry```（参考```https://keel.sh/docs/#harbor-webhooks```）。
```
kubectl get svc -n kube-system
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
...
keel                   LoadBalancer   10.233.3.181    <pending>     9300:30162/TCP           36m
...
```
####配置kubernetes原生对象
只需在目标（```deployment,statefulset,daemonset```）的```annotations```加入触发策略的配置。
```
keel.sh/policy: minor
```
策略取值参考上一章节。
####配置helm chart
如果目标是```helm```的```chart```，则在```values.yaml```文件中配置```keel```相关的配置项，例如：
```
eplicaCount: 1
image:
  repository: karolisr/webhook-demo
  tag: "0.0.8"
  pullPolicy: IfNotPresent
service:
  name: webhookdemo
  type: ClusterIP
  externalPort: 8090
  internalPort: 8090

keel:
  # keel policy (all/major/minor/patch/force)
  policy: all
  # images to track and update
  images:
    - repository: image.repository # [1]
      tag: image.tag  # [2]
```
注意事项参考上一章节。
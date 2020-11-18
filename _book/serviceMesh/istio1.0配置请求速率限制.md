###istio配置服务访问速率限制
1、创建名为requestcount的quota实例（instance），quota的dimensions定义速率限制的维度标识。
名为requestcount的quota定义了source，sourceVersion，destination，destinationVersion四个维度标识。开发人员还可以配置多个自定义的标识。例如，可以多加一个
```
path： request.path | "unknow"
```
可以将速率配额限定粒度到url级别。
```
apiVersion: config.istio.io/v1alpha2
kind: quota
metadata:
  name: requestcount
  namespace: istio-system
spec:
  dimensions:
    source: source.labels["app"] | source.service | "unknown"
    sourceVersion: source.labels["version"] | "unknown"
    destination: destination.labels["app"] | destination.service | "unknown"
    destinationVersion: destination.labels["version"] | "unknown"
```

2、配置速率限制的类型为memquota的适配器
memquota配置定义了quota作用的服务的速率限制为1天5000次请求，并且标志了由v2版本的reviews服务到ratings服务的访问速率限制为1天100次请求。
在overrides下有个dimensions，里面的子项便是根据requestcount.quota.istio-system（即名为requestcount的quota）中dimensions声明的。
```
apiVersion: config.istio.io/v1alpha2
kind: memquota
metadata:
  name: handler
  namespace: istio-system
spec:
  quotas:
  - name: requestcount.quota.istio-system
    # default rate limit is 5000qps
    maxAmount: 5000
    validDuration: 86400s
    # The first matching override is applied.
    # A requestcount instance is checked against override dimensions.
    overrides:
    # The following override applies to traffic from 'rewiews' version v2,
    # destined for the ratings service. The destinationVersion dimension is ignored.
    - dimensions:
        destination: ratings
        source: reviews
        sourceVersion: v2
      maxAmount: 100
      validDuration: 86400s
```
3、创建rule把quota实例应用到memquota的适配器。
```
---
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
  name: quota
  namespace: istio-system
spec:
  actions:
  - handler: handler.memquota
    instances:
    - requestcount.quota
```
也可以配置带条件的服务访问限制，例如
创建如下的rule，会配置不同namespace之间的服务的访问速率进行限制。
```
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
 name: quota
 namespace: istio-system
spec:
 match: source.namespace != destination.namespace
 actions:
 - handler: handler.memquota
   instances:
   - requestcount.quota
```
4、指定配额作用到服务
配置名为RequestCount的quota每次计数加1。
```
apiVersion: config.istio.io/v1alpha2
kind: QuotaSpec
metadata:
  creationTimestamp: null
  name: request-count
  namespace: istio-system
spec:
  rules:
  - quotas:
    - charge: 1
      quota: RequestCount
```
配置名为RequestCount的quota作用到namespace为default下的ratings，reviews，details，productpage服务上。如果没有配置这一项，速率限制是不会在服务上生效的。
```
apiVersion: config.istio.io/v1alpha2
kind: QuotaSpecBinding
metadata:
  creationTimestamp: null
  name: request-count
  namespace: istio-system
spec:
  quotaSpecs:
  - name: request-count
    namespace: istio-system
  services:
  - name: ratings
    namespace: default
  - name: reviews
    namespace: default
  - name: details
    namespace: default
  - name: productpage
    namespace: default
```
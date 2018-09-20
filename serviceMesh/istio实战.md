###istio手动注入sidecar
1 直接注入，不开放pod访问外部网络的权限
```
kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml)
```
2 直接注入，开放pod访问外部网络的权限
```
kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml --includeIPRanges=172.20.0.0/16)
```
172.20.0.0/16是kubernetes集群的service ip cidr。可以配置多个IP网络范围。

###开启限流（请求次数配额）
可以配置memquota(测试环境)，redisquota（生产环境）的mixer handler方式来作为解决方案。
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
    validDuration: 1s
    # The first matching override is applied.
    # A requestcount instance is checked against override dimensions.
    overrides:
    # The following override applies to traffic from 'rewiews' version v2,
    # destined for the ratings service. The destinationVersion dimension is ignored.
    - dimensions:
        destination: ratings
        path: /ratings
        source: reviews
        sourceVersion: v2
      maxAmount: 1
      validDuration: 1s
---
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
    path: request.path | "unknown"
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
---
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
---
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
上面可以配置有效时间validDuration内最多请求次数为maxAmount。它对v2版本reviews服务到ratings服务的url路径/ratings在1s内最多一次请求,一旦超过请求时报http 429 Too Many Requests或者 http 503 Service Unavailable。
### istio ingress作为服务入口
创建服务的Ingress作为入口
例如，访问nginx-autoindex-svc服务的80的/nginx路径可以访问服务的主页，则进行如下的入口规则配置
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    kubernetes.io/ingress.class: istio
spec:
  rules:
  - http:
      paths:
      - path: /nginx/.*
        backend:
          serviceName: nginx-autoindex-svc
          servicePort: 80
```
这里需要注意一点，即要求所有需要接入入口流量的服务拥有一个主页的/path。例如上例中通过http://$IP/nginx访问nginx的主页。
### istio ingress作为外部服务入口
1、首先在kubernetes中创建一个外部的service
例如，创建192。192.189.127的外部服务
```
apiVersion: v1
kind: Service
metadata:
  name: nginx127
spec:
  type: ExternalName
  externalName: 192.192.189.127
  ports:
  ports:
    - port: 80
      name: http
```
2、创建一个到service的ingress。
例如，创建一个到service名为nginx127的ingress。
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx127-ingress
  annotations:
    kubernetes.io/ingress.class: istio
spec:
  rules:
  - http:
      paths:
      - path: /nginx127/.*
        backend:
          serviceName: nginx127
          servicePort: 80
```
3、根据实际情况定义routerule，把路由到外部服务/nginx127的url重写到主页/
```
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  name: nginx127
spec:
  destination:
    name: nginx127
  match:
    request:
      headers:
        uri:
          prefix: /nginx127
  rewrite:
    uri: /
    authority: 192.192.189.127
```

### istio内部使用egressrule与routerule控制外部服务流量
使用istio egressrule可以配置访问外部IP的服务，istio envoy默认阻止所有去外部的流量，通过配置egressrule可以访问外部IP的服务。
例如
```
apiVersion: config.istio.io/v1alpha2
kind: EgressRule
metadata:
  name: nginx127
spec:
  destination:
    service: 192.192.189.127
  ports:
    - port: 80
      protocol: http
```
可以是istio管理的pod访问http://192.192.189.127的nginx服务。
也可以用routerule配置destination为外部服务的流量管理，例如配置访问http://192.192.189.127的nginx服务的超时时间为3s。
```
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  name: nginx127-timeout-rule
spec:
  destination:
    service: 192.192.189.127
  http_req_timeout:
    simple_timeout:
      timeout: 3s
```
####istio内部配合DNS服务器访问外部域名服务
首先需要一个DNS服务器，例如192.192.189.114。如果DNS服务器部署在集群外面，则配置一条访问外部DNS服务IP的egressrule
```
apiVersion: config.istio.io/v1alpha2
kind: EgressRule
metadata:
  name: dns114
spec:
  destination:
    service: 192.192.189.114
  ports:
    - port: 53
      protocol: tcp
```
然后在pod里面的/etc/resolv.conf加一行
```
nameserver 192.192.189.114
```
这个操作也可以在pod yaml文件里修改。
然后创建到外部基于域名的服务访问egressrule。
```
apiVersion: config.istio.io/v1alpha2
kind: EgressRule
metadata:
  name: nginx127
spec:
  destination:
    service: nginx.teligen.com
  ports:
    - port: 80
      protocol: http
```
DNS服务器192.192.189.114配置了nginx.teligen.com的IP为192.192.189.127。
###istio appcode与token鉴权
配置HTTP headers，例如要求incoming的流量cookie头包含字串“user=jason”
```
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  name: ratings-jason
spec:
  destination:
    name: reviews
  match:
    request:
      headers:
        cookie:
          regex: "^(.*?;)?(user=jason)(;.*)?$"
  ...
```
也可以同时配置source和http header
```
apiVersion: config.istio.io/v1alpha2
kind: RouteRule
metadata:
  name: ratings-reviews-jason
spec:
  destination:
    name: ratings
  match:
    source:
      name: reviews
      labels:
        version: v2
    request:
      headers:
        cookie:
          regex: "^(.*?;)?(user=jason)(;.*)?$"
  ...
```

###监控

###弹性伸缩

###istio后续版本
#### istio gateway作为服务入口
#### istio externalservice配置外部服务



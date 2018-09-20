###istio1.0流量管理关键概念
* VirtualService： 定义了服务请求方来源（hosts和gateways）和服务请求匹配规则及符合规则后对应目的地服务（match，route）。
* DestinationRule：定义了经过流量路由后的目的地服务配置，包括负载均衡策略，服务版本定义。
* Gateway：定义了指定pod的入口或出口边界代理，一般用于配置入口服务的代理，绑定到流量路由到定义的VirtualServic上，也可以用于配置内部服务的代理（它通过kubernetes里label来指定）。
* ServiceEntry：所有通过sidecar注入的pod对应的service会在istio内部生成一个ServiceEntry，在VirtualService，DestinationRule，Gateway内部的host定义就是从所有的ServiceEntry中查找的，一般用于添加额外的服务到ServiceEntry库里，从而给其他定义中host可以查找到，额外的服务包括kubernetes外部的服务，属于kubernetes内部的服务但未被istio管控的服务。

###负载均衡规则配置
在DestinationRule里可以配置指定服务的负载均衡策略，使用trafficPolicy字段指定，例如配置服务```ratings.prod.svc.cluster.local```被访问的负责均衡策略为最少连接算法。
```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: bookinfo-ratings
spec:
  host: ratings.prod.svc.cluster.local
  trafficPolicy:
    loadBalancer:
      simple: LEAST_CONN
```
可以指定对服务的某个版本指定另外的负载均衡策略。
```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: bookinfo-ratings
spec:
  host: ratings.prod.svc.cluster.local
  trafficPolicy:
    loadBalancer:
      simple: LEAST_CONN
  subsets:
  - name: testversion
    labels:
      version: v3
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
```
可以对服务的不同端口访问指定不同的负载均衡策略。
```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: bookinfo-ratings-port
spec:
  host: ratings.prod.svc.cluster.local
  trafficPolicy: # Apply to all ports
    portLevelSettings:
    - port:
        number: 80
      loadBalancer:
        simple: LEAST_CONN
    - port:
        number: 9080
      loadBalancer:
        simple: ROUND_ROBIN
```
可配置的负载均衡策略包含simple和consistentHash。simple类型的有ROUND_ROBIN（循环，默认算法）、LEAST_CONN（最少连接数）、RANDOM（随机）、PASSTHROUGH（直接路由，源IP到固定连接）。consistentHash可以配置为基于源IP、httpCookie、httpHeader的hash负载均衡算法。
###配置请求路由规则与流量转移分发
配置请求规则在微服务中可以动态的把请求路由到服务的不同版本上，例如配置应用中的所有微服务组成都访问V1版本。
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: details
  ...
spec:
  hosts:
  - details
  http:
  - route:
    - destination:
        host: details
        subset: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: productpage
  ...
spec:
  gateways:
  - bookinfo-gateway
  - mesh
  hosts:
  - productpage
  http:
  - route:
    - destination:
        host: productpage
        subset: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
  ...
spec:
  hosts:
  - ratings
  http:
  - route:
    - destination:
        host: ratings
        subset: v1
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
  ...
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
---
```
此外，可以配置某个用户访问到指定服务的V2版本。例如，配置reviews服务在用户jason访问时，路由到V2版本，其他用户访问时，路由到V1版本。
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
  ...
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
```
在版本更新时，可以分流一部分用于新版本服务测试，另外部分到旧版本服务。在route下定义多条规则便可以实现，例如，配置reviews服务50%流量路由到V1版本，50%流量路由到V3版本。
```
piVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
  ...
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 50
    - destination:
        host: reviews
        subset: v3
      weight: 50
```
待测试通过后，在把所有流量路由到新版本上。
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v3
```
请求路由规则和流量分发都是在VirtualService上配置的。
###控制ingress服务入口流量
配置入口服务流量原理是首先在istio ingress入口配置一个gateway代理，使用它来充当istio管理入口服务的访问方，例如，创建一个gateway代理http 80端口，允许任意hostname来源。
```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: httpbin-gateway
spec:
  selector:
    istio: ingressgateway # use Istio default gateway implementation
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
```
注：上面的select配置了它为```istio: ingressgateway```istio ingress入口的代理，如果设为其他服务的label，则是为其他服务的gateway代理。
配置入口服务的访问来源放为gateway代理。（这里配置访问 http://$(gateway)/headers 到目标服务httpbin的8000端口）
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin
spec:
  hosts:
  - "*"
  gateways:
  - httpbin-gateway
  http:
  - match:
    - uri:
        prefix: /headers
    route:
    - destination:
        port:
          number: 8000
        host: httpbin
```
###控制egress服务访问外部服务流量
外部服务不在istio的serviceentry库里面，因此，需要创建一个serviceentry来标识外部服务，然后创建VirtualService把流量路由到查找到指定的serviceentry。例如，创建ServiceEntry代表外部服务http://httpbin.org。
```
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: httpbin-ext
spec:
  hosts:
  - httpbin.org
  ports:
  - number: 80
    name: http
    protocol: HTTP
  resolution: DNS
```
创建VirtualService把流量路由到ServiceEntry。
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin-ext
spec:
  hosts:
    - httpbin.org
  http:
  - route:
      - destination:
          host: httpbin.org
        weight: 100
```
有了VirtualService还可以配置超时，故障注入，熔断等特性到外部服务。
此外，还可以配置访问https的外部服务，例如，访问https://www.google.com。
创建ServiceEntry。
```
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: google
spec:
  hosts:
  - www.google.com
  ports:
  - number: 443
    name: https
    protocol: HTTPS
  resolution: DNS
```
创建VirtualService。
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: google
spec:
  hosts:
  - www.google.com
  tls:
  - match:
    - port: 443
      sniHosts:
      - www.google.com
    route:
    - destination:
        host: www.google.com
        port:
          number: 443
      weight: 100
```
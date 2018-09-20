###istio RBAC(基于角色访问控制)
istio RBAC支持namespace-level，service-level,method-level的服务访问控制。
Role-Base语义，支持服务到服务，用户到服务的认证
可以灵活的定义roles和role-bindings的properties
mixter的认证相关instance为authorization
```
apiVersion: "config.istio.io/v1alpha2"
kind: authorization
metadata:
  name: requestcontext
  namespace: istio-system
spec:
  subject:
    user: source.user | ""
    groups: ""
    properties:
      service: source.service | ""
      namespace: source.namespace | ""
  action:
    namespace: destination.namespace | ""
    service: destination.service | ""
    method: request.method | ""
    path: request.path | ""
    properties:
      version: request.headers["version"] | ""
```
subject：定义一系列properties来identify调用者(caller)。
action：定义服务哪些可以被访问。
mixter的认证相关handler为rbac
```
apiVersion: "config.istio.io/v1alpha2"
kind: rbac
metadata:
  name: handler
  namespace: istio-system
spec:
  config_store_url: "k8s://"
  cache_duration: "30s"

---
apiVersion: "config.istio.io/v1alpha2"
kind: rule
metadata:
  name: rbaccheck
  namespace: istio-system
spec:
  match: destination.namespace == "default"
  actions:
  # handler and instance names default to the rule's namespace.
  - handler: handler.rbac
    instances:
    - requestcontext.authorization
```
config_store_url：定义RBAC engine获取RBAC policies的来源地，默认为"k8s://"，即从kubernetes API server获取。
cache_duration： 定义缓存authorization 结果的持续时间。
上面创建了rule将定义缓存authorization的instance requestcontext.authorization与handler handler.rbac作用在一起。
####istio RBAC Policy
istio RBAC Policy包括ServiceRole和ServiceRoleBinding。
ServiceRole：定义访问服务的角色。
ServiceRoleBinding：赋予用户（一个用户，一个用户组，一个服务）某个角色。
```
apiVersion: "config.istio.io/v1alpha2"
kind: ServiceRole
metadata:
  name: products-viewer
  namespace: default
spec:
  rules:
  - services: ["products.default.svc.cluster.local"]
    methods: ["GET", "HEAD"]
---
apiVersion: "config.istio.io/v1alpha2"
kind: ServiceRoleBinding
metadata:
  name: test-binding-products
  namespace: default
spec:
  subjects:
  - user: "alice@yahoo.com"
  - properties:
      service: "reviews.abc.svc.cluster.local"
      namespace: "abc"
  roleRef:
    kind: ServiceRole
    name: "products-viewer"
```
ServiceRole中的service，methods，paths属于requestcontext.authorization定义action的维度。
ServiceRoleBinding中的subjetcs取值的user，group，properties属于requestcontext.authorization定义subject的维度。
###istio服务配置黑名单和白名单
1、使用黑名单来隔离服务访问
配置黑名单的模型为：
instance： checknothing
handler：denier
rule：配置前提条件，把denier和checknothing组合在一起
例如，配置v3版本服务reviews到服务ratings的黑名单中
```
apiVersion: "config.istio.io/v1alpha2"
kind: denier
metadata:
  name: denyreviewsv3handler
spec:
  status:
    code: 7
    message: Not allowed
---
apiVersion: "config.istio.io/v1alpha2"
kind: checknothing
metadata:
  name: denyreviewsv3request
spec:
---
apiVersion: "config.istio.io/v1alpha2"
kind: rule
metadata:
  name: denyreviewsv3
spec:
  match: destination.labels["app"] == "ratings" && source.labels["app"]=="reviews" && source.labels["version"] == "v3"
  actions:
  - handler: denyreviewsv3handler.denier
    instances: [ denyreviewsv3request.checknothing ]
```
配置serviceaccount的bookinfo-productpage为服务details的黑名单
```
apiVersion: "config.istio.io/v1alpha2"
kind: denier
metadata:
  name: denyproductpagehandler
spec:
  status:
    code: 7
    message: Not allowed
---
apiVersion: "config.istio.io/v1alpha2"
kind: checknothing
metadata:
  name: denyproductpagerequest
spec:
---
apiVersion: "config.istio.io/v1alpha2"
kind: rule
metadata:
  name: denyproductpage
spec:
  match: destination.labels["app"] == "details" && source.user == "cluster.local/ns/default/sa/bookinfo-productpage"
  actions:
  - handler: denyproductpagehandler.denier
    instances: [ denyproductpagerequest.checknothing ]
```
2、使用白名单来隔离服务访问
配置黑名单的模型为：
instance： listentry
handler：listchecker
rule：配置前提条件，把listentry和listchecker组合在一起
例如，配置v3版本服务reviews到服务ratings的黑名单中
```
apiVersion: config.istio.io/v1alpha2
kind: listentry
metadata:
  name: appversion
spec:
  value: source.labels["version"]
---
apiVersion: config.istio.io/v1alpha2
kind: listchecker
metadata:
  name: whitelist
spec:
  # providerUrl: ordinarily black and white lists are maintained
  # externally and fetched asynchronously using the providerUrl.
  overrides: ["v1", "v2"]  # overrides provide a static list
  blacklist: false
---
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
  name: checkversion
spec:
  match: destination.labels["app"] == "ratings"
  actions:
  - handler: whitelist.listchecker
    instances:
    - appversion.listentry
```
###istio配置服务间TLS认证
在安装istio的时候，如果配置了TLS认证，则会对所有归istio管理的服务（注入了sidecar的服务）都会启动相互TLS认证。
以下展示功能如下：
1、在服务的Kubernetes annotate上启用或禁用服务相互TLS认证访问功能。
2、修改istio的配置来对指定的一些control服务禁用相互TLS认证访问功能。

1、在服务的Kubernetes annotate上启用或禁用服务相互TLS认证访问功能
在istio管理服务的service上添加annotation如下
```
annotations:
  auth.istio.io/8000: NONE
```
可以禁用对该服务的8000端口TLS认证功能。
2、修改istio的配置来对指定的一些服务禁用相互TLS认证访问功能
在istio的配置有mtlsExcludedServices参数，在其中加入需要禁用相互TLS认证访问功能的服务
例如，对与kubernetes ApiServer的服务访问禁用相互TLS认证。
```
kubectl get configmap -n istio-system istio -o yaml | grep mtlsExcludedServices
mtlsExcludedServices: ["kubernetes.default.svc.cluster.local"]
```
访问kubernetes apiserver
```
kubectl exec $(kubectl get pod -l app=sleep -o jsonpath={.items..metadata.name}) -c sleep -- curl https://kubernetes.default:443/api/ -k -s
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "104.199.122.14"
    }
  ]
}
```
删除配置中的mtlsExcludedServices，再次访问
```
kubectl get pod $(kubectl get pod -l istio=pilot -n istio-system -o jsonpath={.items..metadata.name}) -n istio-system -o yaml | kubectl replace --force -f -
```
```
kubectl exec $(kubectl get pod -l app=sleep -o jsonpath={.items..metadata.name}) -c sleep -- curl https://kubernetes.default:443/api/ -k -s
command terminated with exit code 35
```
###istio结合jwt服务器进行认证
istio结合常用的jwt服务器keycloak对服务的```url```进行jwt认证。
在istio0.7.1上，首先对服务进行jwt认证配置。
例如下面的配置文件
```
[root@node1 istio]# cat mixer-rule-only-authorized.yaml
---
apiVersion: "config.istio.io/v1alpha2"
kind: denier
metadata:
  name: carsapi-handler
  namespace: myproject
spec:
  status:
    code: 16
    message: You are not authorized to access the service
---
apiVersion: "config.istio.io/v1alpha2"
kind: checknothing
metadata:
  name: carsapi-denyrequest
  namespace: myproject
spec:
---
apiVersion: "config.istio.io/v1alpha2"
kind: rule
metadata:
  name: carspi-deny
  namespace: myproject
spec:
  match: destination.labels["app"] == "cars-api" && destination.namespace == "myproject" && request.path == "/cars/list" && (request.headers["authorization"]|"unauthorized") == "unauthorized"
  actions:
  - handler: carsapi-handler.denier.myproject
    instances: [carsapi-denyrequest.checknothing.myproject]
```
istio定义了```denier```，```checknothing```，```rule```来对服务```cars-api```的```/cars/list```来进行认证。
具体的认证方法定义在```EndUserAuthenticationPolicySpec```中，通过```EndUserAuthenticationPolicySpecBinding```绑定到具体的服务，例如，下面配置文件
```
[root@node1 istio]# cat car-api-auth_config.yaml 
--- 
apiVersion: config.istio.io/v1alpha2
kind: EndUserAuthenticationPolicySpec
metadata: 
  name: cars-api-auth-policy
  namespace: myproject
spec: 
  jwts: 
    - issuer: http://keycloak.myproject:8080/auth/realms/istio
      jwks_uri: http://keycloak.myproject:8080/auth/realms/istio/protocol/openid-connect/certs
      audiences: 
      - cars-web
      forward_jwt: true
--- 
apiVersion: config.istio.io/v1alpha2
kind: EndUserAuthenticationPolicySpecBinding
metadata:
  name: cars-api-auth-policy-binding
  namespace: myproject
spec:
  policies:
    - name: cars-api-auth-policy
      namespace: myproject
  services:
    - name: cars-api
      namespace: myproject
```
```cars-api-auth-policy```配置了认证方式jwt的路径为```jwks_uri: http://keycloak.myproject:8080/auth/realms/istio/protocol/openid-connect/certs```，```cars-api-auth-policy-binding```把认证方式```cars-api-auth-policy```绑定到服务```cars-api```。
不过，在istio0.8版本的预览中
jwt认证方式已更改为
通过```policy```进行配置，配置如下
```
$ export JWKS=https://www.googleapis.com/service_accounts/v1/jwk/<YOUR-SVC-ACCOUNT>
$ export TOKEN=<YOUR-TOKEN>

cat <<EOF | istioctl replace -n foo -f -
apiVersion: "authentication.istio.io/v1alpha1"
kind: "Policy"
metadata:
  name: "httpbin"
spec:
  targets:
  - name: httpbin
  peers:
  - mtls:
  origins:
  - jwt:
      issuer: "YOUR_SERVICE_ACCOUNT_EMAIL"
      jwksUri: $JWKS
  principalBinding: USE_ORIGIN
EOF
```
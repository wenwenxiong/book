参考网址：https://github.com/kameshsampath/istio-keycloak-demo
###部署keycloak与istio结合的demo
下载源码
```
git clone https://github.com/kameshsampath/istio-keycloak-demo
```
部署 keycloak
```
oc apply -f istio-keycloak-demo/openshift-files/keycloak.yaml
```
打开keycloak的webconsole
```
minishift openshift service keycloak --in-browser
```
###部署cars api
1、编译生成car api的docker image镜像
在cars-api目录下执行命令
```
./mvnw -Distio.home=[your istio home folder] clean package fabric8:build
```
2、部署cars-api到openshift平台上
```
oc apply -f cars-api/src/istio/istio-cars-api-0.0.1-all.yml
```
配置istio认证
```
oc apply -f cars-api/src/istio/car-api-auth_config.yaml
```
```
oc apply -f $DEMO_HOME/cars-api/src/istio/mixer-rule-only-authorized.yaml
```
3、测试
没有token访问
```
curl -vvv $(minishift openshift service cars-api)/cars/list
```
token访问
生成token
```
---
kubectl run -i --rm --restart=Never tokenizer --image=tutum/curl \
--command \
-- curl -X POST 'http://keycloak.istio-system:8080/auth/realms/istio/protocol/openid-connect/token' \
-H "Content-Type: application/x-www-form-urlencoded" \
-d 'username={demo-user}&password={demo-user}&grant_type=password&client_id=cars-web' | jq .access_token
---
```
上面命令执行后会生成一串字符串，把它存入```$token```。
```
curl -vvv -H "Authorization: Bearer $token" $(minishift openshift service cars-api)/cars/list
```
4、查询istio的配置
查询LDS和CDS
```
istioctl proxy-config <pod-name>
```

5、遇到问题
```
UNAUTHENTICATED:carsapi-handler.denier.myproject:You are not authorized to access the service
```
在```istio/car-api-auth_config.yaml```文件中加入```forward_jwt: true```内容。
```
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
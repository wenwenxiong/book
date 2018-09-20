###k8s部署kong
参考网址：https://github.com/Kong/kong-dist-kubernetes/tree/master/minikube
本次实验环境为```minikube```。
kong可以使用cassandra或postgres作为数据库，这里使用cassandra作为数据库。
```
kubectl create -f cassandra.yaml
```
数据库准备
```
kubectl create -f kong_migration_cassandra.yaml
```
部署kong
```
kubectl create -f kong_cassandra.yaml
```

###kong配置服务转发
创建名为```example-service```的转发地址为```http://mockbin.org```的服务。
```
curl -i -X POST http://192.168.39.32:30374/services/ --data 'name=example-service' --data 'url=http://mockbin.org'
```
####通过```http```子url转发
创建路由```route```通过子url```/auth-sample```路由到服务```example-service```。
```
curl -i -X POST \
  --url http://localhost:8001/services/example-service/routes \
  --data 'paths[]=/auth-sample'
```
通过```http://localhost:8000/auth-sample```可以访问服务的地址```http://mockbin.org```。

####通过```http```头部转发
创建路由```route```通过头部```Hosts: example.com```路由到服务```example-service```。
```
curl -i -X POST http://192.168.39.32:30374/services/example-service/routes --data 'hosts[]=example.com'
```
通过以下命令行测试访问
```
curl -i -X GET http://192.168.39.32:30556/ --header 'Host: example.com'
```
以下部署通过firefox浏览器测试访问。
####firefox中查看http request headers
参考网站：https://o7planning.org/en/11637/how-to-view-http-headers-in-firefox

####firefox中修改http request headers
firefox使用插件```modefy header```
![firefoxaddon](./kong/firefoxaddon.png  "firefoxaddon")
配置```modify header```，增加一条记录，为```url```为```http://192.168.39.32:30556/```添加头部```Host```和头部值为```example.com```，如下
![modifyheader](./kong/modifyheader.png  "modifyheader")
再次访问```http://192.168.39.32:30556/```，显示的页面如下
![servicepage](./kong/servicepage.png  "servicepage")

显示的```http request hearder```如下,可以看到在请求头下有```Host: example.com```头部。
![requestheader](./kong/requestheader.png  "requestheader")

###kong配置认证

####启用```key-auth```插件
```
curl -i -X POST --url http://localhost:8001/services/example-service/plugins/ --data 'name=key-auth'
```
测试插件是否启动成功
```
curl -i -X GET --url http://localhost:8000/ --header 'Host: example.com'
```
没有```apikey```头部，返回```401 Unauthorized```。
```
HTTP/1.1 401 Unauthorized
Date: Mon, 04 Jun 2018 04:20:28 GMT
Content-Type: application/json; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
WWW-Authenticate: Key realm="kong"
Server: kong/0.13.1

{"message":"No API key found in request"}
```
为服务添加用户```Jason```。
```
curl -i -X POST --url http://localhost:8001/consumers/ --data "username=Jason"
```
为用户```Jason```配置```key```。
```
curl -i -X POST --url http://localhost:8001/consumers/Jason/key-auth/ --data 'key=cef1fe6937e444a6b18a26965d619718'
```
使用用户与```key```访问服务。
```
curl -i -X GET --url http://localhost:8000 --header "Host: example.com" --header "apikey: cef1fe6937e444a6b18a26965d619718"
```
####启用```jwt```插件
参考网址：https://getkong.org/plugins/jwt/?_ga=2.54302632.1414723783.1528097448-1093226140.1527845787
```
curl -i -X POST --url http://localhost:8001/services/example-service/plugins/ --data "name=jwt"
```
为服务添加用户```bill```。
```
curl -i -X POST --url http://localhost:8001/consumers/ --data "username=bill"
```
为用户```bill```创建```jwt```证书
```
 curl -i -X POST --url http://localhost:8001/consumers/bill/jwt -H "Content-Type: application/x-www-form-urlencoded"
```
获得```response```。
```
HTTP/1.1 201 Created
Date: Tue, 05 Jun 2018 01:39:25 GMT
Content-Type: application/json; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Access-Control-Allow-Origin: *
Server: kong/0.13.1

{"created_at":1528162765119,"id":"a0d929d2-51be-47e9-967b-a177897fc911","algorithm":"HS256","secret":"NIt4p3uIUBepybyNc7Ypqse3F9WbS70d","key":"kK5XS2PaAXvUAGAmZtTgIyLPeCqUbMdO","consumer_id":"1a87badc-364e-4b6d-a051-3cc9c9323d8b"}
```
在网址```https://jwt.io/```上生成JWT。

![jwttoken](./kong/jwttoken.png  "jwttoken")

```
Issuer (key from above): kK5XS2PaAXvUAGAmZtTgIyLPeCqUbMdO
Key (secret from above): NIt4p3uIUBepybyNc7Ypqse3F9WbS70d
```
产生的JWT为
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJrSzVYUzJQYUFYdlVBR0FtWnRUZ0l5TFBlQ3FVYk1kTyJ9.Guko4BRzLkRyAtYnu2T9T-M11HdcP5FbBnI_gr-b6BM
```
使用JWT 访问服务
```
curl http://localhost:8000 \
    -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJrSzVYUzJQYUFYdlVBR0FtWnRUZ0l5TFBlQ3FVYk1kTyJ9.Guko4BRzLkRyAtYnu2T9T-M11HdcP5FbBnI_gr-b6BM'
```
###kong速率限制
参考网址：https://konghq.com/plugins/
https://getkong.org/plugins/rate-limiting/?_ga=2.54302632.1414723783.1528097448-1093226140.1527845787
https://getkong.org/plugins/request-size-limiting/?_ga=2.18338489.1414723783.1528097448-1093226140.1527845787

####http请求次数限制
kong可以启用插件```rate-limiting```对一段周期内（秒，分钟，小时，天，月，年）的请求次做限制，如果service/Route没有认证层，则基于```client IP```为基准配置请求速率限制，如果配置认证层，则基于Consumer为基准配置请求速率限制。
例如，为服务配置请求速率限制为每分钟内5次,没有指定consumer，对所有访问计数限制。
```
curl -X POST http://kong:8001/services/{service}/plugins \
    --data "name=rate-limiting"  \
    --data "config.minute=5"
```
配置对指定consumer访问服务配置请求速率限制为每分钟内5次。
```
curl -X POST http://kong:8001/services/{service}/plugins \
    --data "name=rate-limiting"  \
    --data "config.minute=5" \
    --data "consumer_id={consumer_id}"
```

####http请求体大小限制
请求体大小以M为单位。
例如，配置服务的请求体最大为128M。
```
curl -X POST http://kong:8001/services/{service}/plugins \
    --data "name=request-size-limiting"  \
    --data "config.allowed_payload_size=128"
```
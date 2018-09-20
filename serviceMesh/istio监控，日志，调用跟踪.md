###分布式跟踪
使用zipkin作为istio的分布式跟踪软件套件。
在kubernetes中运行zipkin
```
kubectl apply -f install/kubernetes/addons/zipkin.yaml
```
访问zipkin的dashboard。

服务能别zipkin收集分布式跟踪信息，必须在程序中收集和传播以下http头
```
x-request-id
x-b3-traceid
x-b3-spanid
x-b3-parentspanid
x-b3-sampled
x-b3-flags
x-ot-span-context
```
例如，收集http头如下
```
def getForwardHeaders(request):
    headers = {}

    user_cookie = request.cookies.get("user")
    if user_cookie:
        headers['Cookie'] = 'user=' + user_cookie

    incoming_headers = [ 'x-request-id',
                         'x-b3-traceid',
                         'x-b3-spanid',
                         'x-b3-parentspanid',
                         'x-b3-sampled',
                         'x-b3-flags',
                         'x-ot-span-context'
    ]

    for ihdr in incoming_headers:
        val = request.headers.get(ihdr)
        if val is not None:
            headers[ihdr] = val
            #print "incoming: "+ihdr+":"+val

    return headers
```
和
```
@GET
@Path("/reviews")
public Response bookReviews(@CookieParam("user") Cookie user,
                            @HeaderParam("x-request-id") String xreq,
                            @HeaderParam("x-b3-traceid") String xtraceid,
                            @HeaderParam("x-b3-spanid") String xspanid,
                            @HeaderParam("x-b3-parentspanid") String xparentspanid,
                            @HeaderParam("x-b3-sampled") String xsampled,
                            @HeaderParam("x-b3-flags") String xflags,
                            @HeaderParam("x-ot-span-context") String xotspan) {
  String r1 = "";
  String r2 = "";

  if(ratings_enabled){
    JsonObject ratings = getRatings(user, xreq, xtraceid, xspanid, xparentspanid, xsampled, xflags, xotspan);
```
###监控
使用prometheus和grafana来收集和展示监控信息。
在kubernetes中运行prometheus和grafana
```
kubectl apply -f install/kubernetes/addons/prometheus.yaml
kubectl apply -f install/kubernetes/addons/grafana.yaml
```
istio配置监控指标信息
istio对资源的配置有三种类型instance，handler，rule。
instance：定义指标实例。instance的类型有metric，logentry，quota，listentry等等。
handler：定义指标实例的处理方式。handler的类型有Fluentd，Datadog，Denier，prometheus，stdio等等。
rule：以限定条件的方式把instance与hanler配对。


###生成图展示
使用Servicegraph来展示服务的生成图信息。
在kubernetes中运行Servicegraph。
```
kubectl apply -f install/kubernetes/addons/servicegraph.yaml
```
访问Servicegraph的dashboard。

###日志收集与查询
运行EFK日志收集套件
```
kubectl apply -f install/kubernetes/addons/logging-stack.yaml
```

istio配置fluentd日志类型信息
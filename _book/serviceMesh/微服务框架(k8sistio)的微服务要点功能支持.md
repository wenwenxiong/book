###kismatic部署kubernetes1.10
创建kubernetes部署节点机
准备待部署kubernetes的物理机或者虚拟机，配置安装包源和镜像仓库，配置ntp服务器，禁用防火墙，禁用swap分区。
进入kubernetes部署节点机，修改kismatic-cluster.yaml文件
执行命令部署kubernetes集群
```
./kismatic install apply
```
等待十几分钟便完成kubernetes部署。
###istio0.7.1部署
安装istio后可以配置制定namespace下的pod自动注入sidecar。
###istio功能支撑
应用微服务化对底层支持平台初步的功能点需求
功能|需求点|实现方|操作方法|应用程序代码是否需要修改|api url
----|----|------|------|--------
流量控制|流量入口规则|istio pilot|istio ingress作为服务入口|应用程序需要有子url作为首页path|$APISERVER/apis/extensions/v1beta1/ingresses
流量控制|配置外部服务集成|istio pilot|istio egressrule与routerule控制外部服务流量|否|$APISERVER/apis/config.istio.io/v1alpha2/egressrules $APISERVER/apis/config.istio.io/v1alpha2/routerules
执行策略|api请求次数配额|istio mixer|istio 开启限流（请求次数配额）|否|$APISERVER/apis/config.istio.io/v1alpha2/memquotas $APISERVER/apis/config.istio.io/v1alpha2/quotas $APISERVER/apis/config.istio.io/v1alpha2/rules $APISERVER/apis/config.istio.io/v1alpha2/quotaspecs $APISERVER/apis/config.istio.io/v1alpha2/quotaspecbindings
执行策略|api鉴权|istio mixer| | |
监控|调用链监控|istio zipkin|部署zipkin|跟踪的路径代码方法中需加上特定的http头部|
监控|生成图展示|istio servicegraph|部署servicegraph|否|
监控|资源监控| | | |
监控|请求次数监控| | | |
弹性伸缩|自动弹性伸缩| | | |
镜像仓库管理|| | | |
istio组件高可用自动化部署|| | | |
###Kubernetes中存储的应用场景
在Kubernetes中部署和运行的服务大致分为：

1. 无状态服务

Kubernetes使用ReplicaSet来保证一个服务的实例数量，如果说某个Pod实例由于某种原因挂掉或崩溃，ReplicaSet会立刻用这个Pod的模版新启一个Pod来替代它。由于是无状态的服务，新Pod与旧Pod一模一样。此外Kubernetes通过Service（一个Service后面可以挂多个Pod）对外提供一个稳定的访问接口，实现服务的高可用。

2. 普通有状态服务

和无状态服务相比，它多了状态保存的需求。Kubernetes提供了以Volume和Persistent Volume为基础的存储系统，可以实现服务的状态保存。

3. 有状态集群服务

和普通有状态服务相比，它多了集群管理的需求。要运行有状态集群服务要解决的问题有两个，一个是状态保存，另一个是集群管理。Kubernetes为此开发了StatefulSet（以前叫做PetSet），方便有状态集群服务在Kubernetes上部署和管理。

简单来说是通过Init Container来做集群的初始化工作，用Headless Service来维持集群成员的稳定关系，用动态存储供给来方便集群扩容，最后用StatefulSet来综合管理整个集群。

分析以上的服务类型，Kubernetes中对于存储的使用主要集中在以下几个方面：

* 服务的基本配置文件读取、密码密钥管理等；
* 服务的存储状态、数据存取等；
* 不同服务或应用程序间共享数据；


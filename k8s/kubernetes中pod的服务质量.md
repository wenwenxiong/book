###kubernetes的QoS类型
kubernetes包含三种QoS类型
* Guaranteed
* Burstable
* BestEffort

Guaranteed: 配置成Guaranteed的QoS类型pod需要满足1）Pod中的每个Container需要配置memory request与memory limit相同，2）Pod中的每个Container需要配置cpu request与cpu limit相同。

Burstable: 配置成Burstable的QoS类型pod需要满足1）Pod中的资源配置不满足Guaranteed的QoS类型，2）Pod中的至少一个Container配置了cpu request或者memory request。

BestEffort: 配置成BestEffort的QoS类型pod需要满足1) Pod中的Container必须不配置memory or cpu limits or requests。

###实践参考
在提高资源利用率、降低成本的同时，需要在服务的QoS与优化资源利用率之间有个平衡。我们的原则是在保证服务质量的同时，尽量提高资源的利用率。

根据Kubernetes的资源模型，在Pod level的QoS分为三个等级：Guarantee、Burstable、BestEffort，我们也是依照这三个级别对应我们应用的优先级来制定资源超卖的标准。

我们对应用设置的QoS标准：

Kubernetes自带的组件使用Guarantee。
重要的组件和应用，比如ZooKeeper、Redis，用户服务等使用Guarantee。
普通的应用（Burstable）按照重要性分级，按重要程度CPU分为2，5，10三个超卖标准，10倍超卖适合boss后台类的应用，大多数适合访问量不高。内存使用固定的1.5倍超卖标准。
有一点需要特别注意，在生产环境中，不要使用BestEffort的方式，它会引发不确定的行为。
###kubernetes配置pod的资源限制
pod中的container可以配置cpu/memory的request和limit。
pod的cpu/memory request和limit为pod中所有container的cpu/memory的request和limit之和。
如果pod中container使用的cpu/memory超出了定义cpu/memory的limit，则会killed它。
如果pod cpu/memory的request超出了node的剩余cpu/memory，则不能调度分配到该节点，

###kubernetes配置namespace的默认资源限制
在指定namespace下创建LimitRange对象，便可以为namespace设置默认资源限制
####memory
例如，配置namespace默认资源momery限制为memory request为256Mi，memory limit为512Mi。
```
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```
针对在拥有默认memory配置的namespace中创建pod时
1）如果pod中container配置了memory limits，则创建后pod的memory limits和memory request的值都为pod配置的memory limits的值。
2）如果pod中container配置了memory request，则创建后pod的memory limits为namespace的默认memory limits的值，memory request的值都为pod配置的memory request的值。
3） 如果pod中container没有配置memory request和memory limits，container则使用namespace中定义的default memory request和memory limits。
配置了默认memory的namespace保证
1）namespace下每个container会有memory limit。
2）namespace下所有container使用的memory 总数不能超过namespace指定的memory limit。
####cpu
例如，配置namespace默认资源momery限制为cpu request为0.5核，memory limit为1核。
```
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 0.5
    type: Container
```
针对在拥有默认cpu配置的namespace中创建pod时
1）如果pod中container配置了cpu limits，则创建后pod的cpu limits和cpu request的值都为pod配置的memory limits的值。
2）如果pod中container配置了cpu request，则创建后pod的cpu limits为namespace的默认cpu limits的值，cpu request的值都为pod配置的cpu request的值。
3） 如果pod中container没有配置cpu request和cpu limits，container则使用namespace中定义的default cpu request和cpu limits。
配置了默认cpu的namespace保证
1）namespace下每个container会有cpu limit。
2）namespace下所有container使用的cpu 总数不能超过namespace指定的cpu limit。

###配置namespace下container的最大最小cpu与内存
namespace通过LimitRange来设置内部container的最大最小cpu/memory数值。
配置namespace下容器内存最小值500M，最大值1G。
```
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-min-max-demo-lr
spec:
  limits:
  - max:
      memory: 1Gi
    min:
      memory: 500Mi
    type: Container
```
如果在namespace下创建的pod中容器memory limits大于namespace配置的最大内存值、pod中容器memory requests小于namespace配置的最小内存值，则会报错，阻止容器的创建。
```
Error from server (Forbidden): error ......
```
配置namespace下容器cpu最小值200M，最大值800M。
```
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-min-max-demo-lr
spec:
  limits:
  - max:
      cpu: "800m"
    min:
      cpu: "200m"
    type: Container
```
如果在namespace下创建的pod中容器cpu limits大于namespace配置的最大cpu值、pod中容器cpu requests小于namespace配置的最小cpu值，则会报错，阻止容器的创建。
```
Error from server (Forbidden): error ......
```
###配置namesapce的memory和cpu配额
kubernetes通过ResourceQuota来配置namespace的配额。
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-demo
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```
配置了mem-cpu-demo的ResourceQuota的namespace要求
1. 每一个namespace下的container必须配置memory request, memory limit, cpu request, and cpu limit。
2. 所有容器的memory request之和不能超过1G
3. 所有容器的memory limiti之和不能超过2G
4. 所有容器的cpu request之和不能超过1G
5. 所有容器的cpu limit之和不能超过2G

###配置namespace的pod运行数量配额
kubernetes通过ResourceQuota来配置namespace的最大pod运行数
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: pod-demo
spec:
  hard:
    pods: "2"
```
###配置namespace的API Objects配额
kubernetes通过ResourceQuota来配置namespace的API Objects最大数量
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-quota-demo
spec:
  hard:
    persistentvolumeclaims: "1"
    services.loadbalancers: "2"
    services.nodeports: "0"
```
可配置的API Objects如下
String|API Object
------|---------
"pods"|Pod
"services | Service
"replicationcontrollers" | ReplicationController
"resourcequotas" | ResourceQuota
"secrets" | Secret
"configmaps" | ConfigMap
"persistentvolumeclaims" | PersistentVolumeClaim
"services.nodeports" | Service of type NodePort
"services.loadbalancers" | Service of type LoadBalancer

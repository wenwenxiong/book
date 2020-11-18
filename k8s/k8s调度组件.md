kubernetes除了```kube-controller-manager```分配```node```给```pod```，```kube-scheduler```进行```pod```调度并且保证```pod```高可用。社区还存在一些优化```kubernetes```集群的调度插件。
###自动弹性伸缩
集群```kubernetes```的```hpa```，以及可以基于自定义监控指标的弹性伸缩插件（https://github.com/stefanprodan/k8s-prom-hpa）

###社区组件
名称|作用|参考网址
---|----|-----
rescheduler（二次调度）|配置保证关键组件高可用，在资源不足情况下，通过杀死非关键组件|https://kubernetes.io/docs/tasks/administer-cluster/guaranteed-scheduling-critical-addon-pods/，https://github.com/apprenda/kismatic/blob/master/docs/add_ons.md#rescheduler
descheduler|针对整个kubernetes集群，调整各节点的pod，优化整个集群的资源利用率|https://github.com/kubernetes-incubator/descheduler
addon-resizer| 针对特定组件，根据集群规模大小，调整特定组件的实例数量|https://github.com/kubernetes/autoscaler/tree/master/addon-resizer

还有一些调度组件
组件```cluster-autoscaler```：伸缩```kubernetes```集群的规模，只能在云上的```kubernetes```使用。
组件```vertical-pod-autoscaler```: 伸缩```pod```的资源```requests```量，使每个```pod```实例都能运行成功。
上述两个组件在```github```项目```autoscaler```中可以找到（https://github.com/kubernetes/autoscaler）


还有一个组件```k8s-spot-rescheduler```，把节点分为固定组和按需启动组，该组件会不断尝试降低按需启动组节点的负载，在资源允许情况下把```pod```驱逐到固定组节点上。
参考网址：https://github.com/pusher/k8s-spot-rescheduler


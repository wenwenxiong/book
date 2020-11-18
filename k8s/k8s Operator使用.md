###Operator
```Stateless is Easy, Stateful is Hard```
在kubernetes上，管理和伸缩无状态服务（web apps， mobile backends, and API services）容易，因为恢复和扩展它们不需要额外的状态数据同步等考虑。
管理和扩展```stateful applications```（数据库，缓存数据库，监控系统）面临挑战，这些程序在失败恢复和扩展，升级，重新配置过程中需要保证数据不丢失和可用性。

Operator是由CoreOS开发的，用来扩展Kubernetes API，特定的应用程序控制器，它用来创建、配置和管理复杂的有状态应用，如数据库、缓存和监控系统。Operator基于Kubernetes的资源和控制器概念之上构建，但同时又包含了应用程序特定的领域知识。创建Operator的关键是kubernetes中CRD（自定义资源）的设计。

operator可以处理有状态应用的 有状态应用程序集群的安全平滑升级、备份数据到异地存储、通过kubernetes native api进行服务发现、应用程序TLS证书配置、灾难恢复等场景。

####工作原理
Operator是将运维人员对软件操作的知识给代码化，同时利用Kubernetes强大的抽象来管理大规模的软件应用。

Operator使用了Kubernetes的自定义资源扩展API机制，如使用CRD（CustomResourceDefinition）来创建。Operator通过这种机制来创建、配置和管理应用程序。

当前CoreOS提供的以下四种Operator：

etcd：创建etcd集群
Rook：云原生环境下的文件、块、对象存储服务
Prometheus：创建Prometheus监控实例
Tectonic：部署Kubernetes集群
以上为CoreOS官方提供的Operator，另外还有很多很多其他开源的Operator实现。

Operator基于Kubernetes的以下两个概念构建：

资源：对象的状态定义
控制器：观测、分析和行动，以调节资源的分布

###Operator Framework

Operator Framework是一个开源项目为开发者快速开发Operator提供工具。
它包含以下组件

* Operator SDK: 允许开发人员在不需要了解Kubernetes API复杂性的情况下，基于他们的专业知识构建 Operators.
* Operator Lifecycle Management: 监视运行在Kubernetes集群上的所有Operators(及其相关服务)的生命周期的安装、更新和管理。
* Operator Metering (开发中): 为指定专门服务的Operators启用使用报告。

###operator-sdk使用
1、 operator-sdk cli命令行工具安装
```
$ mkdir -p $GOPATH/src/github.com/operator-framework
$ cd $GOPATH/src/github.com/operator-framework
$ git clone https://github.com/operator-framework/operator-sdk
$ cd operator-sdk
$ git checkout master
$ make dep
$ make install
```
2、使用operator sdk cli命令行工具创建和部署app-operator
```
# Create an app-operator project that defines the App CR.
$ mkdir -p $GOPATH/src/github.com/example-inc/
# Create a new app-operator project
$ cd $GOPATH/src/github.com/example-inc/
$ operator-sdk new app-operator
$ cd app-operator

# Add a new API for the custom resource AppService
$ operator-sdk add api --api-version=app.example.com/v1alpha1 --kind=AppService

# Add a new controller that watches for AppService
$ operator-sdk add controller --api-version=app.example.com/v1alpha1 --kind=AppService

# Build and push the app-operator image to a public registry such as quay.io
$ operator-sdk build quay.io/example/app-operator
$ docker push quay.io/example/app-operator

# Update the operator manifest to use the built image name
$ sed -i 's|REPLACE_IMAGE|quay.io/example/app-operator|g' deploy/operator.yaml

# Setup Service Account
$ kubectl create -f deploy/service_account.yaml
# Setup RBAC
$ kubectl create -f deploy/role.yaml
$ kubectl create -f deploy/role_binding.yaml
# Setup the CRD
$ kubectl create -f deploy/crds/app_v1alpha1_appservice_crd.yaml
# Deploy the app-operator
$ kubectl create -f deploy/operator.yaml

# Create an AppService CR
# The default controller will watch for AppService objects and create a pod for each CR
$ kubectl create -f deploy/crds/app_v1alpha1_appservice_cr.yaml

# Verify that a pod is created
$ kubectl get pod -l app=example-appservice
NAME                     READY     STATUS    RESTARTS   AGE
example-appservice-pod   1/1       Running   0          1m

# Cleanup
$ kubectl delete -f deploy/app_v1alpha1_appservice_cr.yaml
$ kubectl delete -f deploy/operator.yaml
$ kubectl delete -f deploy/role.yaml
$ kubectl delete -f deploy/role_binding.yaml
$ kubectl delete -f deploy/service_account.yaml
$ kubectl delete -f deploy/crds/app_v1alpha1_appservice_crd.yaml
```
参考网址： https://github.com/operator-framework/operator-sdk/blob/master/doc/user-guide.md

###operator sdk 安装

前提：安装以下软件
* dep version v0.5.0+.
* git
* go version v1.10+.
* docker version 17.03+.
* kubectl version v1.10.0+.
* Access to a kubernetes v.1.10.0+ cluster.

安装operator sdk和命其令行工具如下
```
$ mkdir -p $GOPATH/src/github.com/operator-framework
$ cd $GOPATH/src/github.com/operator-framework
$ git clone https://github.com/operator-framework/operator-sdk
$ cd operator-sdk
$ git checkout master
$ make dep
$ make install
```

###operator sdk示例使用

使用operator-sdk 命令行工具创建示例项目memcached-operator
```
$ mkdir -p $GOPATH/src/github.com/example-inc/
$ cd $GOPATH/src/github.com/example-inc/
$ operator-sdk new memcached-operator
$ cd memcached-operator
```

创建后的项目结构如下

File/Folders|	Purpose
------------|-----------
cmd|	包含的 ```manager/main.go ```是自定义operator的运行程序入口. 它实例化``` a new manager``` 注册 ```pkg/apis/...```下所有的自定义资源和 ```pkg/controllers/...```下所有的控制器。
pkg/apis|	包含the APIs of the Custom Resource Definitions(CRD). 用户可以修改``` pkg/apis/<group>/<version>/<kind>_types.go``` 文件来定义``` the API for each resource type``` 和引入 控制器里的包来观察这些```Resource```。
pkg/controller|	这里包含``` controller```的实现 ，用户可以编辑``` pkg/controller/<kind>/<kind>_controller.go ```定义控制器的处理特定```kind```资源类型的``` reconcile logic```。
build|	包含```Dockerfile``` 和```build scripts``` 用来构建自定义``` operator```的镜像。
deploy|	包含多种``` YAML manifests```用于注册 ```CRDs```, 配置 ```RBAC```, 和部署``` operator as a Deployment```。
Gopkg.toml Gopkg.lock|	文件```Go Dep manifests```用于描述自定义```operator```的外部依赖。
vendor|	包管理器```vendor```目录，包含外部依赖的本地拷贝。

使用命令行工具添加自定义的资源类型（CRD）
```
$ operator-sdk add api --api-version=cache.example.com/v1alpha1 --kind=Memcached
```
用户可以修改```pkg/apis/cache/v1alpha1/...```下的代码，增加自定义的属性值等。
之后通过以下命令生成代码
```
$ operator-sdk generate k8s
```

使用命令行工具添加自定义的```Controller```。
```
$ operator-sdk add controller --api-version=cache.example.com/v1alpha1 --kind=Memcached
```


用户一般通过编辑```pkg```目录下的文件来实现自定义```operator```所需要的行为。

修改完代码后，就进入构建和运行自定义```operator```阶段。
执行以下命令构建定义```operator```镜像。
```
$ operator-sdk build quay.io/example/memcached-operator:v0.0.1
$ sed -i 's|REPLACE_IMAGE|quay.io/example/memcached-operator:v0.0.1|g' deploy/operator.yaml
$ docker push quay.io/example/memcached-operator:v0.0.1
```

创建定义```operator```的```rbac```及运行```operator```。
```
$ kubectl create -f deploy/service_account.yaml
$ kubectl create -f deploy/role.yaml
$ kubectl create -f deploy/role_binding.yaml
$ kubectl create -f deploy/operator.yaml
```
验证是否成功运行
```
$ kubectl get deployment
```

创建自定义的```CR```。
```
$ kubectl apply -f deploy/crds/cache_v1alpha1_memcached_cr.yaml
```
可以更新```deploy/crds/cache_v1alpha1_memcached_cr.yaml```的属性值再次运行

清理环境
```
$ kubectl delete -f deploy/crds/cache_v1alpha1_memcached_cr.yaml
$ kubectl delete -f deploy/operator.yaml
$ kubectl delete -f deploy/role_binding.yaml
$ kubectl delete -f deploy/role.yaml
$ kubectl delete -f deploy/service_account.yaml
```
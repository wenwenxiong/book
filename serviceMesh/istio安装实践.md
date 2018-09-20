###启动minikube
istio的实验环境部署在minikube上，首先启动minikube（minikube安装参考上一博客）
```
  minikube start --vm-driver kvm2 \
	--extra-config=controller-manager.ClusterSigningCertFile="/var/lib/localkube/certs/ca.crt" \
	--extra-config=controller-manager.ClusterSigningKeyFile="/var/lib/localkube/certs/ca.key" \
	--extra-config=apiserver.Admission.PluginNames=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,DefaultTolerationSeconds,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota \
    --cpus 4 --memory 8196
```

###安装istio
下载最新的istio
```
curl -L https://git.io/getLatestIstio | sh -
cd istio-0.7
export PATH=$PWD/bin:$PATH
```
下载istio镜像
```
#!/bin/bash

set -x
dockerimages=(
prom/statsd-exporter:v0.5.0
docker.io/istio/mixer:0.7.1
docker.io/istio/proxy:0.7.1
docker.io/istio/pilot:0.7.1
docker.io/istio/proxy_init:0.7.1
docker.io/istio/istio-ca:0.7.1
docker.io/istio/sidecar_injector:0.7.1
)

j=1
for i in ${dockerimages[@]}
do
    echo $i
    echo $j

    docker pull $i && docker save $i -o $j.tar && xz $j.tar
    docker rmi $i
    let j+=1
done
set +x

```
在k8s中运行istio
```
kubectl apply -f install/kubernetes/istio.yaml
```
在k8s中运行服务之间带auth功能的istio
```
kubectl apply -f install/kubernetes/istio-auth.yaml
```
这里有个小问题，镜像以docker.io开头导入下载与本地将头部docker.io去掉了，例如docker.io/istio/mixer:0.7.1，在下载和导入到本地时，镜像的名称变为istio/mixer:0.7.1。
解决办法是手动的修改yaml文件的镜像名称。
###配置istio注入sidecar
前提
1）一个pod只能属于一个service，属于多个service的pod不支持。
2）service的ports必须按照一定的命名规范
```
<protocol>[-<suffix>]
```
```
<protocol> 可以取值 http, http2, grpc, mongo, or redis，这样可以获得istio的动态路由特性，否则都当作tcp流量。
```
3）推荐pod通过deployment部署，每个deployment都应显式的通过label app来标识。
4）同一个pod内的容器不能通过sidecar来控制流量。

注入sidecar是在deployment的所有pod sec下加入创建injected sidecar的yaml配置。

手动注入sidecar和自动注入sidecar加载的配置都是从namespace istio-system下的ConfigMap istio-inject。不同的是手动注入sidecar可以选择从本地文件读取配置。

手动注入sidecar
```
kubectl apply -f <(istioctl kube-inject -f samples/sleep/sleep.yaml)
```
读者可以比较
```
istioctl kube-inject -f samples/sleep/sleep.yaml > sleep2.yaml
```
sleep2.yaml和sleep.yaml文件便可以看出对yaml配置的修改内容。

从本地文件获取配置注入sidecar
```
kubectl create -f install/kubernetes/istio-sidecar-injector-configmap-release.yaml \
    --dry-run \
    -o=jsonpath='{.data.config}' > inject-config.yaml

kubectl -n istio-system get configmap istio -o=jsonpath='{.data.mesh}' > mesh-config.yaml

istioctl kube-inject \
    --injectConfigFile inject-config.yaml \
    --meshConfigFile mesh-config.yaml \
    --filename samples/sleep/sleep.yaml \
    --output sleep-injected.yaml

kubectl apply -f sleep-injected.yaml
```
自动注入sidecar
前提
1）kubernetes版本要1.9或以上，并且支持以下api版本
```
kubectl api-versions | grep admissionregistration
admissionregistration.k8s.io/v1beta1
```
安装webhook
```
./install/kubernetes/webhook-create-signed-cert.sh \
    --service istio-sidecar-injector \
    --namespace istio-system \
    --secret sidecar-injector-certs
```
如遇到错误
```
ERROR: After approving csr istio-sidecar-injector.istio-system, the signed certificate did not appear on the resource. Giving up after 10 attempts.
```
解决办法：配置kube-controller-manager的两个参数
```
cluster-signing-cert-file=/etc/kubernetes/pki/ca.pem
cluster-signing-key-file=/etc/kubernetes/pki/ca-key.pem
```
kubernetes1.10版本遇到错误
```
error: error validating "STDIN": error validating data: [apiVersion not set, kind not set]; if you choose to ignore these errors, turn validation off with --validate=false
```
修改./install/kubernetes/webhook-create-signed-cert.sh脚本的最后几行为
```
# create the secret with CA cert and server cert/key
kubectl create secret generic ${secret} \
        --from-file=key.pem=${tmpdir}/server-key.pem \
        --from-file=cert.pem=${tmpdir}/server-cert.pem \
        -n ${namespace}
#        --dry-run -o yaml |
#    kubectl --validate=false -n ${namespace} apply -f -

```
安装sidecar注入configmap
```
kubectl apply -f install/kubernetes/istio-sidecar-injector-configmap-release.yaml
```
配置caBundle，kubernetes api-server使用caBundle调用webhook
```
cat install/kubernetes/istio-sidecar-injector.yaml | \
     ./install/kubernetes/webhook-patch-ca-bundle.sh > \
     install/kubernetes/istio-sidecar-injector-with-ca-bundle.yaml
```
安装sidecar injector webhook
```
kubectl apply -f install/kubernetes/istio-sidecar-injector-with-ca-bundle.yaml
```
使用方法
首先，配置指定的namespaces的label istio-injection为enabled。
```
kubectl label namespace default istio-injection=enabled
```
然后，在配置了标签istio-injection为enabled的namespace创建pod，便会自动注入sidecar容器。
```
kubectl apply -f samples/sleep/sleep.yaml  -n default
```
###验证istio相互TLS认证
验证集群中istio CA已部署
```
kubectl get deploy -l istio=istio-ca -n istio-system
```
验证istio的AuthPolicy为MUTUAL_TLS。
```
kubectl get configmap istio -o yaml -n istio-system | grep authPolicy | head -1
```
进入productpage容器中，查看证书，使用curl测试带证书与不带证书访问的结果区别
```
NAME                              READY     STATUS    RESTARTS   AGE
productpage-v1-4184313719-5mxjc   2/2       Running   0          23h

kubectl exec -it productpage-v1-4184313719-5mxjc -c istio-proxy /bin/bash
ls /etc/certs/
cert-chain.pem   key.pem   root-cert.pem

curl https://details:9080/details/0 -v --key /etc/certs/key.pem --cert /etc/certs/cert-chain.pem --cacert /etc/certs/root-cert.pem -k

curl https://details:9080/details/0 -v
```
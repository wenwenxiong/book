参考网址
https://docs.openebs.io/docs/next/installation.html#helm

###helm方式安装openebs存储
分为三个步骤，首先在每个k8s的node节点安装和配置open-iscsi工具，其次配置helm的RBAC，最后helm安装openebs。

1、假设每个k8s的node节点为centos操作系统。
在centos机器上
```
yum install iscsi-initiator-utils -y
systemctl enable iscsid
systemctl start iscsid
```
2、配置helm的RBAC。
```
kubectl -n kube-system create sa tiller
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
kubectl -n kube-system patch deploy/tiller-deploy -p '{"spec": {"template": {"spec": {"serviceAccountName": "tiller"}}}}'
```
3、helm安装openebs
```
helm install  --namespace openebs --name openebs  -f https://openebs.github.io/charts/helm-values-0.6.0.yaml stable/openebs
kubectl apply -f https://raw.githubusercontent.com/openebs/openebs/v0.6/k8s/openebs-storageclasses.yaml
```
实际部署中，会把```helm-values-0.6.0.yaml```的值直接配置在openebs的chart包内的```values.yaml```文件中，并且在```stable/openebs```已经定义```storageclass```的创建，并且可以修改chart里的template文件，把openebs-standard配置为default storageclass。这样，在ansible部署时，只需一条命令。
```
---
  - name: helm install openebs storage plugin
    command: helm install --name openebs stable/openebs
```
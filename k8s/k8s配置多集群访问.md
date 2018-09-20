###k8s配置多集群访问
kubernetes的配置文件描述了集群、用户名和上下文的关系。
通过命令
```
kubectl config view
```
可以查看k8s所有的配置文件信息。
命令
```
kubectl config  view --minify --kubeconfig=config-demo
```
查看k8s当前的配置。指定配置文件为当前的```config-demo```文件。

kubernetes可以命令配置集群，用户，上下文的详细信息。
配置集群信息，假设存在两个集群development和scratch。配置它们的服务地址和认证方式添加到```config-demo```文件中。
```
kubectl config --kubeconfig=config-demo set-cluster development --server=https://1.2.3.4 --certificate-authority=fake-ca-file
kubectl config --kubeconfig=config-demo set-cluster scratch --server=https://5.6.7.8 --insecure-skip-tls-verify
```
配置用户信息。假设存在两个用户developer和experimenter。配置它们的认证信息添加到```config-demo```文件中。
```
kubectl config --kubeconfig=config-demo set-credentials developer --client-certificate=fake-cert-file --client-key=fake-key-seefile
kubectl config --kubeconfig=config-demo set-credentials experimenter --username=exp --password=some-password
```
配置上下文信息，上下文信息关联用户和集群，并且可选的名字空间。配置三个上下文dev-frontend，dev-storage，exp-scratch。
```
kubectl config --kubeconfig=config-demo set-context dev-frontend --cluster=development --namespace=frontend --user=developer
kubectl config --kubeconfig=config-demo set-context dev-storage --cluster=development --namespace=storage --user=developer
kubectl config --kubeconfig=config-demo set-context exp-scratch --cluster=scratch --namespace=default --user=experimenter
```

切换使用另一个上下文
```
kubectl config --kubeconfig=config-demo use-context exp-scratch
```
####实战：配置kismatic部署的k8s访问信息到外部默认的配置文件```$HOME/.kube/config```中。
查看k8s master节点上的```$HOME/.kube/config```。
```
[root@node1 vagrant]# cat ~/.kube/config 
apiVersion: v1
kind: Config
clusters:
- name: kubernetes
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.pem
    server: "https://127.0.0.1:6443" 
users:
- name: admin
  user:
    client-certificate: "/etc/kubernetes/pki/admin.pem"
    client-key: "/etc/kubernetes/pki/admin-key.pem"
contexts:
- name: kubernetes
  context:
    cluster: kubernetes
    user: admin
current-context: kubernetes
```
1）拷贝配置文件```/etc/kubernetes/pki/ca.pem```，```/etc/kubernetes/pki/admin.pem```，```/etc/kubernetes/pki/admin-key.pem```到远程机器自定义的目录。例如```$HOME/.kismatic```
```
scp -r /etc/kubernetes/pki/ca.pem /etc/kubernetes/pki/admin.pem /etc/kubernetes/pki/admin-key.pem xww@192.192.189.1:~/.kismatic/
```
2)配置集群，用户，上下文。
编辑```$HOME/.kube/config```，增加名为kismatic用户，集群，上下文。
```
cat .kube/config
...
- cluster:
  name: kismatic
...
- context:
  name: kismatic
...
- name: kismatic
...
```
配置kismatic集群信息
```
kubectl config set-cluster kismatic --server=https://192.192.189.31:6443 --certificate-authority=/home/xww/.kismatic/ca.pem
```
配置kismatic用户信息
```
kubectl config set-credentials kismatic --client-certificate=/home/xww/.kismatic/admin.pem --client-key=/home/xww/.kismatic/admin-key.pem
```
配置kismatic上下文
```
kubectl config set-context kismatic --cluster=kismatic --user=kismatic
```
3)当前切换到kismatic上下文
```
kubectl config use-context kismatic
```
至此，配置完毕。
测试执行
```
kubectl get pod --all-namespaces
```
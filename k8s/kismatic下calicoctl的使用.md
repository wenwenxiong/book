### 安装calicotl
有两种方式安装calicoctl，以二进制方式安装和以pod的形式运行calicoctl。

以pod的形式运行calicoctl，只要配置了kubectl的节点就可以使用calicoctl，很方便，但是有些命令可能因为权限问题执行不成功。
以二进制方式安装calicoctl，则只能在安装的节点上使用calicoctl。
###二进制方式安装配置calicoctl
在以下网址选择下载二进制文件```calicoctl```
```
https://github.com/projectcalico/calicoctl/releases
```
配置```calicoctl```到环境变量```path```中。
程序```calicoctl```的配置文件内容为（根据自己环境修改）
```
[root@node1 vagrant]# cat /etc/calico/calicoctl.cfg
apiVersion: projectcalico.org/v3
kind: CalicoAPIConfig
metadata:
spec:
  etcdEndpoints: https://node1:6666
  etcdKeyFile: /etc/kubernetes/pki/etcd-client-key.pem
  etcdCertFile: /etc/kubernetes/pki/etcd-client.pem
  etcdCACertFile: /etc/kubernetes/pki/ca.pem
```
运行命令
```
[root@node1 vagrant]# calicoctl get node
NAME    
node1 
```
###以pod的形式运行calicoctl
下载```calicoctl```的```yaml```文件
```
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calicoctl.yaml
```
运行命令
```
[root@node1 vagrant]# kubectl exec -ti -n kube-system calicoctl -- /calicoctl get node -o wide
NAME    ASN         IPV4   IPV6   
node1   (unknown)
```
其中有些命令会报错
```
[root@node1 vagrant]# kubectl exec -ti -n kube-system calicoctl -- /calicoctl get networkPolicy -o wide
Failed to get resources: connection is unauthorized
command terminated with exit code 1
```
###calico的资源类型
calicoctl可以操作的
有效的资源类型
```
Valid resource types are:


    * bgpConfiguration
    * bgpPeer
    * felixConfiguration
    * globalNetworkPolicy
    * hostEndpoint
    * ipPool
    * networkPolicy
    * node
    * profile
    * workloadEndpoint
```
输出格式可以指定
```
    ps                    Display the results in ps-style output.
    wide                  As per the ps option, but includes more headings.
    custom-columns        As per the ps option, but only display the columns
                          that are requested in the comma-separated list.
    golang-template       Display the results using the specified golang
                          template.  This can be used to filter results, for
                          example to return a specific value.
    golang-template-file  Display the results using the golang template that is
                          contained in the specified file.
    yaml                  Display the results in YAML output format.
    json                  Display the results in JSON output format.
```
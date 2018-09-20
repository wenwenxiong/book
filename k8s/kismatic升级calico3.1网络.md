###kismatic的calico组成
1、calico.yaml和rbac.yaml文件
位于
```
ansible/playbooks/roles/calico/templates/calico.yaml
ansible/playbooks/roles/calico/templates/rabc.yaml
```
2、
位于policy-controller.yaml
```
ansible/playbooks/roles/calico-network-policy/templates/policy-controller.yaml
```
3、etcd_networking服务
```
systemctl status etcd_networking
```
4、kismatic中声明的calico镜像
```
ansible/playbooks/group_vars/container_images.yaml
```
###下载并修改3.1版calico的yaml文件
下载calico3.1网络，选择与已存在的etcd datastore集成安装
下载rbac.yaml文件
```
wget https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/rbac.yaml
```
下载calico.yaml文件
```
wget https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/calico.yaml
```
在名为```calico-config```的```ConfigMap```，修改```etcd_endpoints```为kismatic运行的```etcd_networking```服务。
参照kismatic的calico template文件修改```rbac.yaml```和```calico.yaml```文件，修改后的文件如下
1、calico.yaml和rbac.yaml文件
calico.yaml
```
ansible/playbooks/roles/calico/templates/calico.yaml
ansible/playbooks/roles/calico/templates/rabc.yaml
```
2、
位于policy-controller.yaml
```
ansible/playbooks/roles/calico-network-policy/templates/policy-controller.yaml
```
3、kismatic中声明的calico镜像
```
  calico_node:
    name: calico/node
    version: v3.1.1
  calico_ctl:
    name: calico/ctl
    version: v1.6.3
  calico_cni:
    name: calico/cni
    version: v3.1.1
  calico_kube_controller:
    name: calico/kube-controllers
    version: v3.1.1

```

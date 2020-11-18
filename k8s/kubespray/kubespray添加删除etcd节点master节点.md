###kubespray删除etcd节点
首先通过```etcdctl```工具删除```etcd```集群中的待删除节点，例如```node01```
```
etcdctl --endpoints=https://192.168.124.171:2379,https://192.168.124.172:2379,https://192.168.124.173:2379 --ca-file=/etc/ssl/etcd/ssl/ca.pem --key-file=/etc/ssl/etcd/ssl/member-node01-key.pem --cert-file=/etc/ssl/etcd/ssl/member-node01.pem member list
```
找出```node01```的```id```
删除```etcd member node01```
```
etcdctl --endpoints=https://192.168.124.171:2379,https://192.168.124.172:2379,https://192.168.124.173:2379 --ca-file=/etc/ssl/etcd/ssl/ca.pem --key-file=/etc/ssl/etcd/ssl/member-node01-key.pem --cert-file=/etc/ssl/etcd/ssl/member-node01.pem member remove node01id
```

###kubespray添加etcd节点
在```hosts.ini```中添加新的机器，执行```cluster.yml```脚本（加上```-l etcd 所有的 node```）
例如，原```hosts.ini```为
```
[all]
node01 ansible_host=192.168.124.171 ansible_ssh_user=root ansible_ssh_private_key_file=./deploy.key
node02 ansible_host=192.168.124.172 ansible_ssh_user=root ansible_ssh_private_key_file=./deploy.key
node03 ansible_host=192.168.124.173 ansible_ssh_user=root ansible_ssh_private_key_file=./deploy.key

[kube-deploy]
node01

[etcd]
node01

[kube-master]
node01
node02

[kube-node]
node01
node02
node03

[k8s-cluster:children]
kube-master
kube-node
```
添加新的```etcd```节点后的```hosts.ini```为
```
[all]
node01 ansible_host=192.168.124.171 ansible_ssh_user=root ansible_ssh_private_key_file=./deploy.key
node02 ansible_host=192.168.124.172 ansible_ssh_user=root ansible_ssh_private_key_file=./deploy.key
node03 ansible_host=192.168.124.173 ansible_ssh_user=root ansible_ssh_private_key_file=./deploy.key

[kube-deploy]
node01

[etcd]
node01
node02
node03

[kube-master]
node01
node02

[kube-node]
node01
node02
node03

[k8s-cluster:children]
kube-master
kube-node
```
执行命令，添加```etcd```节点
```
ansible-playbook -i inventory/rong/hosts.ini cluster.yml -l node01,node02,node03 -v
```
###kubespray添加master节点
例如，原```hosts.ini```为
```
[all]
node01 ansible_host=192.168.124.171 ansible_ssh_user=root ansible_ssh_private_key_file=./deploy.key

[kube-deploy]
node01

[etcd]
node01

[kube-master]
node01


[kube-node]
node01

[k8s-cluster:children]
kube-master
kube-node
```
添加新的```master```节点后的```hosts.ini```为
```
[all]
node01 ansible_host=192.168.124.171 ansible_ssh_user=root ansible_ssh_private_key_file=./deploy.key
node02 ansible_host=192.168.124.172 ansible_ssh_user=root ansible_ssh_private_key_file=./deploy.key

[kube-deploy]
node01

[etcd]
node01


[kube-master]
node01
node02

[kube-node]
node01


[k8s-cluster:children]
kube-master
kube-node
```
执行命令，添加```master```节点
```
ansible-playbook -i inventory/rong/hosts.ini cluster.yml  -v
```

###kubespray删除master节点

例如，原```hosts.ini```为
```
[all]
node01 ansible_host=192.168.124.171 ansible_ssh_user=root ansible_ssh_private_key_file=./deploy.key
node02 ansible_host=192.168.124.172 ansible_ssh_user=root ansible_ssh_private_key_file=./deploy.key

[kube-deploy]
node01

[etcd]
node01

[kube-master]
node01
node02


[kube-node]
node01

[k8s-cluster:children]
kube-master
kube-node
```
需要删除```master```节点```node02```。
执行命令，删除```master```节点```node02```。
```
ansible-playbook -i inventory/rong/hosts.ini remove-node.yml  -v --extra-vars "node=node02“
```

总结：
如果一个节点是```etcd```，```master```，```worker```节点角色，添加时。
1）当作```etcd```节点添加；
2）当作```master```节点添加；
3）当作```worker```节点添加。

如果一个节点是```etcd```，```master```，```worker```节点角色，删除时。
1）当作```etcd```节点删除；
2）当作```worker```节点删除；（此时，已经从k8s集群中删除，删除```master```和删除```worker```节点是一样的操作）

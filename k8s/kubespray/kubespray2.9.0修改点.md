###```kube-deply```的```role```修改
大幅调整代码模块结构
1. 增加以```tag```指示各模块的运行的功能
2. 增加变量```kube_deploy_uograde```控制```kube-deploy```升级
3. 增加了新的```role```为```post-upgrade```解决升级后```calico```网络插件权限问题。

###```bootstrap-os```的```role```修改
增加了升级```docker-ce```的```task```
###文件```upgrade-cluster.yml```修改点

修改文件```upgrade-cluster.yml```如下
1、注释了带有```container-engine```的```role```那一行
2、在执行```bootstrap-os```的```role```机器中剔除```kube-deploy```
3、加上新的```kube-deploy```的```role```运行，如下所示
```
- hosts: kube-deploy
  gather_facts: true
  roles:
    - { role: kube-deploy, tags: ["dns", "secureregistryserver", "mynginx"]}
```
4、增加了解决升级后```calico```网络插件权限问题```kube-deploy/post-upgrade```的```role```运行。
```
- hosts: kube-master[0]
  gather_facts: true
  roles:
    - { role: kube-deploy/post-upgrade}
```
###修改点
https://github.com/kubernetes-sigs/kubespray/issues/3980
由于在版本```1.12.×```到```1.13.×```升级中，网络插件```calico```升级到了```3.4.0```,在最新的```3.4.0```版本```calico```安装中，是通过```initcontainer```来初始化```cni```配置到节点上，而在以前```calico```版本中，是通过外部节点生成```cni```配置文件。因此升级过程中会产生节点获取```namespace```权限问题。需要手动修改。
可修改如下
```
kubectl create clusterrolebinding nodeX-cluster-rule --clusterrole=cluster-admin --user=system:node:nodeX
```
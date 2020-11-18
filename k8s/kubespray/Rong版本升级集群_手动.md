
准备文件：最新版的Rong发布包
###更新DNS
把新增的```dns```配置文件拷贝到```bind9```的配置目录中，重启```bind9```.

###更新系统镜像仓库服务```secureregistryserver```
解压最新版的Rong发布包获取Rong部署脚本，例如```rong1904.tar.gz```。
在待升级集群的部署节点上执行以下命令
```
systemctl stop secureregistryserver
```
把Rong部署脚本目录下的```secureregistryserver```服务安装包拷入待升级集群的部署节点上
```
scp -r roles/kube-deploy/files/secureregistryserver.service  roles/kube-deploy/files/secureregistryserver.tar test@x.x.x.x:~/
```
待升级集群的部署节点上替换新的```secureregistryserver```服务安装包
```
cp -r secureregistryserver.service /etc/systemd/system/secureregistryserver.service
rm -rf /usr/local/secureregistryserver
mv secureregistryserver.tar /usr/local/ && cd /usr/local/ && tar xvf secureregistryserver.tar
```
启动```secureregistryserver```服务
```
systemctl daemon-reload && systemctl start secureregistryserver
```

###更新```repo```源服务```mynginx```

解压最新版的Rong发布包获取Rong部署脚本，例如```rong1904.tar.gz```。
在待升级集群的部署节点上执行以下命令
```
systemctl stop mynginx
```
把Rong部署脚本目录下的```mynginx```服务安装包拷入待升级集群的部署节点上
```
scp -r roles/kube-deploy/files/mynginx.service  roles/kube-deploy/files/fileserver.tar.gz test@x.x.x.x:~/
```
把Rong部署脚本目录下的新版本```deb```包，新版本二进制文件```kubeadm```、```hyperkube```，新版本网络插件配置包```cni-plugins-×```拷贝到待升级集群的部署节点上
```
scp -r roles/kube-deploy/files/1804debs.tar.xz roles/kube-deploy/files/kubeadm roles/kube-deploy/files/hyperkube roles/kube-deploy/files/cni-plugins-amd64-v0* test@x.x.x.x:~/
```
待升级集群的部署节点上替换新的```mynginx```服务安装包
```
cp -r mynginx.service /etc/systemd/system/
rm -rf /usr/local/fileserver
mv fileserver.tar.gz /usr/local/ && cd /usr/local/ && tar -zxvf fileserver.tar.gz
```
待升级集群的部署节点上```repo```目录加入新的```deb```包，新版本二进制文件```kubeadm```、```hyperkube```，新版本网络插件配置包```cni-plugins-×```
```
mv 1604debs.tar.xz /usr/local/ && cd /usr/local/ && tar -Jxvf 1604debs.tar.xz
mv kubeadm hyperkube cni-plugins-× /usr/local/static
```

启动```mynginx```服务
```
systemctl daemon-reload && systemctl start mynginx
```
###升级docker
执行以下命令升级```docker-ce```。
```
apt-get update && apt-get install docker-ce
```
检查服务```docker-infra```，```mynginx```，```secureregistryserver```是否状态良好
```
systemctl status docker-infra
systemctl status mynginx
systemctl status secureregistryserver
```
###升级集群
把新的```Rong```发布包的Rong部署脚本，例如```rong1904.tar.gz```。拷贝到待升级集群的```deploy```节点上。按以下步骤进行升级
1）升级```kubernetes```集群版本
```
ansible-playbook  -i inventory/rong/hosts.ini upgrade-cluster.yml
```
2）升级完成后，执行命令
```
kubectl version
docker version
helm version
kubectl get pod --all-namespaces
```
验证版本是否升级。
3）查看原有运行的用户应用是否运行

注意Dns配置 db.elastic.co
###```kube-deply```的```role```修改
大幅调整代码模块结构，增加以```tag```指示各模块的运行的功能、增加变量```kube_deploy_uograde```控制```kube-deploy```升级、增加了新的```role```为```post-upgrade```解决升级后```calico```网络插件权限问题。
###```bootstrap-os```的```role```修改
增加了升级```docker-ce```的```task```
###文件```upgrade-cluster.yml```修改点
注意```roles/kubernetes/master/tasks/kubeadm-setup.yml```的```196```行加上```--force```。（因为```kubeadm```是我们自编译的，需要加```--force```解决```kubeadm```执行命令时版本兼容报错）
修改文件```upgrade-cluster.yml```如下
1、注释了带有```container-engine```的```role```那一行
2、在执行```bootstrap-os```的```role```机器中剔除```kube-deploy```
3、加上新的```kube-deploy```的```role```运行，如下所示
```
- hosts: kube-deploy
  gather_facts: true
  roles:
    - { role: kube-deploy, kube_deploy_uograde:true}
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
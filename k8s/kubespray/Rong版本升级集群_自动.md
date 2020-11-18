准备文件：最新版的Rong发布包
###升级云平台套件k8s集群版本
1、解压最新的```Rong```平台部署包到待升级的集群部署节点上

2、修改```Rong```部署包中的集群平台配置文件```inventory/rong/hosts.ini```，把原来集群的```hosts.ini```文件拷贝到```inventory/rong/hosts.ini```。

3、执行升级脚本命令
```
ansible-playbook -i inventory/rong/hosts.ini upgrade-cluster.yml
```

参考网址： https://github.com/kubernetes-incubator/kubespray/blob/master/docs/downloads.md

###kubespray download_run_once安装
  在kubespray的官方文档中有介绍，kubespray可以通过只在安装节点上下载所需的docker镜像一次，然后会在```ansible```脚本中通过```docker save```，```scp```，```docker load```命令把所需的```docker```镜像发送并加载到目标主机上。
  采用这种方式，目标```kubernetes```集群机器不需要同```docker```私有仓库交互，kubespray升级docker镜像则无需在```docker```私有仓库为```kubernetes```集群机器管理多版本的```docker```镜像，只需kubespray安装节点上能获取多版本的```docker```镜像。
  启动这种安装方式，只需配置两个变量```download_run_once```和```download_localhost```。
  创建文件```var.yaml```,内容如下
```
[root@localhost kubespray_centos_offline]# cat var.yaml
download_run_once: true
download_localhost: true
```
执行以下命令```kubespray```安装```kubernetes```。
```
ansible-playbook -i inventory/test/hosts.ini cluster.yml  --extra-vars "@var.yaml" -vvv
```

###kubespray download_run_once安装原理

kubespray download_run_once安装通过在安装节点使用```docker pull```命令拉取所需的镜像，并存放在```/tmp/release/containers/*```目录下。然后执行```ansible```脚本```roles/download/tasks/sync_container.yml```把镜像推送并加载到目标```kubernetes```集群。
镜像传送到目标```kubernetes```集群的文件夹路径也是```/tmp/release/containers/*```目录下。
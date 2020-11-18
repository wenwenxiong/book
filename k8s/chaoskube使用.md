###k8s测试工具
 ```chaoskube```: ```chaoskube```周期性的随机杀死```k8s```集群中的```pod```容器，以达到检测```k8s```集群稳定性的目的。

###chaoskube使用
```github```地址： ```https://github.com/linki/chaoskube.git```

准备文件安装包```chaoskube.tar.gz```通过```ansible```自动化安装。
安装前提条件：云平台套件```rong```已经部署
安装步骤：
1）配置目标机器
配置文件为```chaoskube/hosts.ini ```。

2）运行```ansible```命令执行安装
```
tar -zxvf chaoskube.tar.gz
cd chaoskube
ansible-playbook -i hosts.ini chaoskube.yaml
```
安装使用测试工具。
注意点：
1、```chaoskube```默认每隔```10```分钟随机杀死一个```pod```容器，可以通过文件```chaoskube/roles/chaoskube/templates/chaoskube_values.yaml.j2```配置项
```
...
# interval between pod terminations
interval: 10m
...
```
在安装前修改配置。
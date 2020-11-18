参考网址： https://github.com/kubernetes-incubator/kubespray/blob/master/docs/getting-started.md

###kubespray安装k8s
环境： 两台centos76 虚拟机 2核4G
IP为 192.168.122.61 192.168.122.62

1、下载kubespray git项目
```
git clone https://github.com/kubernetes-incubator/kubespray.git
```
2、配置待安装机器
```
cp -r inventory/sample inventory/mycluster
declare -a IPS=(192.168.122.61 192.168.122.62)
CONFIG_FILE=inventory/mycluster/hosts.ini python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```
kubernetes变量配置在```inventory/mycluster/groups_vars/*.yaml```。
3、执行脚本安装k8s
```
pip install -r requirements.txt

ansible-playbook --module-path=/usr/local/lib/python2.7/dist-packages/ansible/modules/hashivault -i inventory/mycluster/hosts.ini cluster.yml -b -v   --private-key=~/.ssh/id_rsa
```
###部署机配置kubesray
环境，centos75-jp 节点
1、安装ansible
```
easy_install pip
pip install ansible
```
2、安装python依赖包
```
cd kubespray
pip install -r requirements.txt
```
3、配置private registry
参考网址: https://github.com/kubernetes-incubator/kubespray/commit/1081f620d259540fc0294c502345bdc0338d2dac

kubespray的image配置文件在
```
./roles/download/defaults/main.yml
./roles/kubernetes/node/defaults/main.yml
./roles/kubernetes-apps/ansible/defaults/main.yml
./roles/dnsmasq/defaults/main.yml
```
使用命令```grep -rn "_image_repo" ./*```可以查询到。

###安装minishift的kvm driver
使用以下命令安装minishift的kvm driver。
```
curl -L https://github.com/dhiltgen/docker-machine-kvm/releases/download/v0.7.0/docker-machine-driver-kvm -o /usr/local/bin/docker-machine-driver-kvm

chmod +x /usr/local/bin/docker-machine-driver-kvm
```
###下载并安装minishift工具
下载minishift的工具包，解压后，并添加```minishift```到```PTAH```。
```
wget https://github.com/minishift/minishift/releases/download/v1.17.0/minishift-1.17.0-linux-amd64.tgz
tar -zxvf minishift-1.17.0-linux-amd64.tgz
```
编辑```$HOME/.bashrc```文件，把```minishift```二进制执行文件加入```PATH```中。
###启动minishift
minishift默认```--vm-driver```为```KVM```。
启动命令
```
minishift start --cpus 2 --memory 4GB
```
使用命令
```
 minishift oc-env
```
把```oc```客户端添加到```PATH```。
关闭minishift
```
minishift stop
```
更新minishift
```
 minishift update
```
删除minishift
```
 minishift delete
```
###部署简单的程序
使用命令来部署一个``` Node.js ```例子程序。
```
oc new-app https://github.com/openshift/nodejs-ex -l name=myapp

oc logs -f bc/nodejs-ex

oc expose svc/nodejs-ex

minishift openshift service nodejs-ex --in-browser
```
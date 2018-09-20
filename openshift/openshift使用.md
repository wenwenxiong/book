###oepnshift  origin安装

角色 | IP | 备注
----|-----|----
openshift | 192.168.122.132| 12G内存

openshift单节点安装与启动（https://blog.frognew.com/2017/11/openshift-manual-installation.html）
https://docs.openshift.org/latest/getting_started/administrators.html#getting-started-administrators

下载tar包到/opt目录下
```
cd /opt
wget https://github.com/openshift/origin/releases/download/v3.6.1/openshift-origin-server-v3.6.1-008f2d5-linux-64bit.tar.gz
tar -zxvf openshift-origin-server-v3.6.1-008f2d5-linux-64bit.tar.gz
mv openshift-origin-server-v3.6.1-008f2d5-linux-64bit openshift
```
配置环境变量
编辑/etc/profile文件，增添以下内容
```
#for openshift
export OPENSHIFT_HOME=/opt/openshift
export PATH=$OPENSHIFT_HOME:$PATH
export KUBECONFIG=$OPENSHIFT_HOME/openshift.local.config/master/admin.kubeconfig
export CURL_CA_BUNDLE=$OPENSHIFT_HOME/openshift.local.config/master/ca.crt
```
启动openshift集群
注意在/opt目录下执行命令启动(openshift会在当前文件夹下产生配置文件)
```
./openshift start
```

###openshift使用

使用系统管理员用户system:admin登录
```
$ oc login -u system:admin
$ oc project default
```
添加Router
先下载Router的docker镜像
```
docker pull openshift/origin-haproxy-router:v3.7.1
```
创建Router
```
oadm policy add-scc-to-user privileged system:serviceaccount:default:router
oadm router router --replicas=1 --service-account=router
```
删除并重建Router
```
oc delete pods $(oc get pods | grep router | awk '{print $1}')
oc delete svc router
oc delete serviceaccount router
oc delete dc router
oadm router router --service-account=router
```


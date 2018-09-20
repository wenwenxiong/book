###chart构成
创建一个名为mychart的cahrt，查看文件结构
```
helm create mychart
[root@k8s-master ~]# tree mychart/
mychart/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   └── service.yaml
└── values.yaml
```
所有kubenetes要执行的yaml模板都存放在templates文件夹下，例如上个例子中存放了deployment，service，ingress三个kubenetes资源对象。
values.yaml文件存放yaml模板中定义的默认值。
Chart.yaml存放cahrt的版本信息。
NOTES.txt显示cahrt的release运行后的帮助信息。
###helm转移chart
首先，编辑和配置好本地的chart文件，然后使用helm打包成tar文件。
```
helm package ./mychart
```
把tar包拷入到另一个环境，通过helm install命令指定tar名称导入。
```
helm install --name example3 mychart-0.1.0.tgz --set service.type=NodePort
```
####helm本地repo
helm可以启动一个本地HTTP服务器，作为一个为本地chart的repo服务。
```
helm serve
```
另开一个终端，可以搜索和安装本地repository中的chart。
```
helm search local
```
```
helm install --name example4 local/mychart --set service.type=NodePort
```
###chart中定义依赖
可以在chart目录中创建一个requirements.yaml文件定义该chart的依赖。
```
$ cat > ./mychart/requirements.yaml <<EOF
dependencies:
- name: mariadb
  version: 0.6.0
  repository: https://kubernetes-charts.storage.googleapis.com
EOF
```
通过helm命令更新和下载cahrt的依赖
```
helm dep update ./mychart
```
在次安装运行chart时会把依赖中定义的chart运行起来。

###自定义chart repository
首先，把每个chart打包的tar文件集中存放到charts目录，使用以下命令生成index.yaml文件。
```
mv mychart-0.1.0.tgz charts/
$ helm serve --repo-path ./charts
```
命令执行完后
charts目录结构如下
```
[root@k8s-master ~]# tree ./charts
./charts
├── index.yaml
└── mychart-0.1.0.tgz

0 directories, 2 files
```
把charts目录在远端web服务器上复制一份，保持连个文件里面tar包文件一样。执行以下命令在本地和远端生成新的index.yaml文件，该文件的url为远端web服务器的url。
```
helm repo index charts --url http://192.168.122.1:81/charts
[root@k8s-master ~]# cat charts/index.yaml
apiVersion: v1
entries:
  mychart:
  - apiVersion: v1
    created: 2018-01-16T11:53:46.922200367+08:00
    description: A Helm chart for Kubernetes
    digest: 7471a2a8496517b4ce1014b2787d3dc745b981fb69c9e53a257ccd7ac390d036
    name: mychart
    urls:
    - http://192.168.122.1:81/charts/mychart-0.1.0.tgz
    version: 0.1.0
generated: 2018-01-16T11:53:46.921691858+08:00
```
####增加chart
在本地charts目录和远端web服务器目录增加新chart的tar文件，然后执行以下命令重建index.yaml。
```
helm repo index charts --url http://192.168.122.1:81/charts
```
或者，在index.yaml中之增加新cahrt的元数据信息。
```
helm repo index charts --url http://192.168.122.1:81/charts --merge
```
把新生成的index.yaml拷贝到远端web服务器上。
####其他人使用该repo
通过以下命令增加repo
```
helm repo add charts http://192.168.122.1:81/charts
[root@k8s-master ~]# helm repo list
NAME     	URL
local    	http://127.0.0.1:8879/charts
stable   	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
monocular	https://kubernetes-helm.github.io/monocular
charts   	http://192.168.122.1:81/charts
```
如果repo有更新，执行repo update命令会更新所以已增加的repo
```
helm repo update
```
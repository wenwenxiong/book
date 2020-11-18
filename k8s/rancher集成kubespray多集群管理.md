###rancher作用
云平台套件是一个部署，升级，支持节点增删功能的```k8s```集群部署维护的工具；并且在第三方插件目录中提供监控，日志，共享存储，备份功能模块；提供对应用市场，持续集成与持续部署模块的开源的组件集成支撑。它缺乏统一界面管理所部署的集群，不提供多租户功能。
云平台套件与```rancher```集成，可以提供以下优点。
使用```rancher```可以导入使用云平台套件部署的多个集群，进行统一界面的多集群编排容器管理。并且集群升级，节点增删功能仍然由云平台套件管理。
使用```rancher```可以与云平台套件部署的```harbor```集成，配置```harbor```作为默认的镜像仓库，支持离线化管理```rancher```所有的系统组件镜像。
使用```rancher```可以与```gitlab```集成，配置默认的```helm charts```仓库地址，支持离线化使用应用市场功能和简单的基于```jinkens```的```pipeline```功能。
使用```rancher```可以对多个集群提供多租户管理功能。
使用```rancher```可以配置自带的监控模块，监控数据深度与界面```	UI```集成，查看监控数据简单，美观，并且提供据直达```grafana```该指标数界面的链接。
使用```rancher```可以配置自带的日志模块，把收集的容器日志存放到用户指定外部```elasticsearch```。
使用```rancher```提供中文界面支持。

###rancher安装

安装包```rancher.tar.gz```通过```ansible```自动化安装。
安装前提条件：云平台套件```rong```已经部署```harbor```仓库和```bind9```服务。
1、配置使用任一安装了```docker```并且可以域名访问```harbor```仓库的机器作为目标部署节点（默认使用```8081：80,8043：443```端口映射，如遇端口冲突，可以修改配置）。
![00](./rancher/00.png "00")

2、运行```ansible```命令执行安装
```
tar -zxvf rancher.tar.gz
cd rancher
ansible-playbook -i hosts.ini rancher.yaml
```
###rancher卸载
卸载```rancher```只要删除运行的```rancher-server```容器和它对应的持久化卷目录
在目标节点上执行以下命令卸载```rancher```
```
docker stop rancher-server && docker rm rancher-server
rm -rf /mnt/rancher-etcd /root/var
```
###rancher配置gitlab（默认系统charts仓库）
默认登录界面为```https://hostip:8043```,登录后首先配置```admin```用户密码和```server url```。
![01](./rancher/01.png "01")
配置完成后，进入```rancher```主界面。
![02](./rancher/02.png "02")
点击菜单```tools```选择```catalog```，配置系统```charts```仓库。
![04](./rancher/04.png "04")
离线环境下，默认的```catalog url```无法访问，使用内网```gitlab```来保存外网导入的```catalog url```项目。
![03](./rancher/03.png "03")
配置系统```charts```仓库完成后，可以发现警告信息解除。
![05](./rancher/05.png "05")
系统```charts```仓库保存```rancher```自带的监控，日志等```chart```。
###rancher导入已存在的kubespray部署的k8s集群
参考网址： https://www.cnrancher.com/docs/rancher/v2.x/en/cluster-provisioning/imported-clusters/
导入集群步骤：
1、点击按钮```Add cluster```选择```Import```，并命名导入的集群。
![06](./rancher/06.png "06")
2、点击```Create```后，到添加集群帮助面板，按照指示，首先在待加入的集群创建一个```clusterrolebinding```。

```
kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user kubernetes-admin
```
在目标集群导入```rancher```的插件。
```
curl --insecure -sfL https://192.168.124.161:8043/v3/import/kgvnjf8hkvz8kjhm8g6p6wbbf2hwqv55bjdhsxzvqt6j627wkkkrdm.yaml | kubectl apply -f -
```
3、点击```Done```完成集群导入。
![07](./rancher/07.png "07")
当待导入集群上```rancher```插件运行起来时，面板上的新导入集群则处于```actived```状态。
![08](./rancher/08.png "08")
![09](./rancher/09.png "09")
进入导入的集群首页。
![10](./rancher/10.png "10")
###rancher监控与日志导入到es
进入导入的集群首页后，选择菜单```Tools```下的```Monitor```，启用集群级别监控。
![11](./rancher/11.png "11")
![12](./rancher/12.png "12")
使用默认的配置，或者调整参数后，点击```Enable Monitoring```。
![13](./rancher/13.png "13")
启动监控后，会在目标集群运行```rancher```监控的组件。
![14](./rancher/14.png "14")
启动监控后，进入集群的首页，会发现集群的指标信息更详细，并且每个指标有直达```grafana```页面的链接。
![15](./rancher/15.png "15")
进入导入的集群首页后，选择菜单```Tools```下的```Logging```，可以把集群的日志导入到外部的```es```集群中。
![16](./rancher/16.png "16")
###rancher试用应用市场
进入目标集群的```default```项目下，首页是导入集群中已运行的容器，```rancher```称之为```workload```。
![17](./rancher/17.png "17")
点击```Apps```面板，进入应用市场。
![18](./rancher/18.png "18")
首先进入```Manage Catalogs```配置```helm chart```的```url```。
![19](./rancher/19.png "19")
把外网的```helm chart```项目移动到内网```gitlab```项目。
![20](./rancher/20.png "20")
配置内网可以访问的```helm chart```的```url```。
![21](./rancher/21.png "21")
回到```Apps```面板，点击```Launch```按钮，可以选择已同步的```chart```模板部署应用。
![22](./rancher/22.png "22")
###rancher结合gitlab试用pipeline
进入目标集群的```default```项目下，选择```Tools```下的```Pipeline```菜单，进入配置源码管理工具的配置。
![23](./rancher/23.png "23")
按照指示，配置关联的```gitlab```。
![24](./rancher/24.png "24")
![25](./rancher/25.png "25")
把信息填入```rancher```。
![26](./rancher/26.png "26")
![27](./rancher/27.png "27")
配置完成，会看到所有```gitlab```项目的显示。
![28](./rancher/28.png "28")
启用一个项目的```pipeline```。
![29](./rancher/29.png "29")
在```workload```面板下的```Popelines```可以看到该项目的持续集成与持续部署过程。
![30](./rancher/30.png "30")
在```rancher```运行的```pipeline```项目，必须存在```.rancher-pipeline.yml或者.rancher-pipeline.yaml```文件，可选的```dockerfile，deployment.yaml```文件满足构建镜像和在```k8s```集群部署```yaml```文件格式。
![31](./rancher/31.png "31")
###rancher试用容器编排功能
有两种创建```workload```的方式，通过```UI```配置和通过```yaml```文件创建。
![32](./rancher/32.png "32")
通过```yaml```文件创建，是上传文件，选择```namespace```进行创建。
![33](./rancher/33.png "33")
通过```UI```配置各项参数部署，例如部署一个名为```testnginx```的```nginx```应用。
![34](./rancher/34.png "34")
部署完成后，可以在```workloads```面板查看。
![35](./rancher/35.png "35")
###rancher多集群应用创建
在```rancher```主页的多集群应用面板，以应用市场的方式部署应用到一个或多个集群。
![37](./rancher/37.png "37")
![38](./rancher/38.png "38")
例如，在三个目标项目部署同一个应用。
![3601](./rancher/3601.png "3601")
![3602](./rancher/3602.png "3602")
###rancher多租户管理
在```rancher```主页的用户面板，创建和管理用户。
![36](./rancher/36.png "36")
用户的权限配置有4种。
![36003](./rancher/36003.png "36003")
新建的用户登录后可以创建自己的项目，创建后，项目则与用户绑定。资源的限制与具体项目相关，与用户无关。
![36004](./rancher/36004.png "36004")
新建项目的主页如下,开始时没有任何工作负载。
![36002](./rancher/36002.png "36002")
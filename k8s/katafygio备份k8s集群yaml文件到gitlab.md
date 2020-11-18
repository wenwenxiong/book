###配置并安装katafygio到k8s集群
安装包```katafygio.tar.gz```通过```ansible```自动化安装。
安装前提条件：云平台套件```rong```已经部署
安装步骤：
1）配置持久化或者```gitlab```集成
配置文件为```katafygio/roles/katafygio/defaults/main.yml```。

2）运行```ansible```命令执行安装
```
tar -zxvf katafygio.tar.gz
cd katafygio
ansible-playbook -i hosts.ini katafygio.yaml
```
####配置持久化（可选）
配置文件的内容为
![01](./katafygio/01.png "01")
配置```persistence.enabled```值为```true```，在创建持久化存储时，```k8s```集群会使用默认的共享存储构建```pvc```给```katafygio```使用。
####配置```gitlab```集成（可选）
配置文件中的配置项```gitUrl```用于配置与外部的```gitlab```服务器集成。
例如，值配置为```http://root:NPsd2zxjhChsyi8znocu@192.168.124.200/root/yamlbak.git```则表示与外部```gitlab```服务器```192.168.124.200```上```root```用户的访问```personal_access_tokens```为```NPsd2zxjhChsyi8znocu```，存储文件的项目名为```yamlbak```。

###与gitlab集成管理yaml文件版本
前提：在外部拥有```gitlab```服务器
步骤：
1） 在```gitlab```服务器上创建用户，项目。
2）配置```gitlab```服务器用户的```personal_access_tokens```。
3）配置配置项```gitUrl```。
在```gitlab```服务器上创建用户，项目是比较简单的，配置配置项```gitUrl```在上面已经提及，下面说明一下配置```gitlab```服务器用户的```personal_access_tokens```。

####配置```personal_access_tokens```
参考网址：https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html

首先，登录```gitlab```，点击右上角```Settings```。
![02](./katafygio/02.png "02")

进入```Settings```后，选择左侧面板的```Access Tokens```。
![03](./katafygio/03.png "03")

配置```token```名称和过期时间，点击```Create personal access token```创建```token```。
![04](./katafygio/04.png "04")

创建完毕，记下生成的```access token```。

注： ```access token```只能生成时查看，务必记下，以后将无法查看，只能创建新的```access token```来替换。
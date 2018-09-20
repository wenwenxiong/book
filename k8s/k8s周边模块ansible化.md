###基本模型
```
➜  kong #ansible-playbook -i inventory ansible_install.yml
➜  kong # 1. tag images, push to portus.teligen.com:xxxxxxxx/xsososg/jgowugoeu
➜  kong # 2. upload yml to /etc/kubernetes/manifest/gowoguwoe.yml, kubectl create -f gowguowuogewgo.yml
➜  kong tree
.
├── ansible_install.yml
├── images
│   ├── kong-ingress.tar
│   └── kong.tar
├── inventory
└── yml
    └── kong.yml
```
###kong
文件目录结构如下
```
[root@portus kong]# tree .
.
├── kong.retry
├── kong.yaml
├── kong.yaml~
├── local.ini
└── roles
    ├── kong
    │   ├── defaults
    │   │   └── all.yaml
    │   ├── tasks
    │   │   ├── main.yaml
    │   │   └── main.yaml~
    │   └── templates
    │       ├── kongadb_postgres.yaml
    │       ├── konga.yaml
    │       ├── kong-dashboard.yaml
    │       ├── kong_migration_postgres.yaml
    │       ├── kong_postgres.yaml
    │       └── postgres.yaml
    └── loadkongimages
        ├── files
        │   ├── kong
        │   │   ├── 1.tar.xz
        │   │   ├── 2.tar.xz
        │   │   ├── 3.tar.xz
        │   │   └── 4.tar.xz
        │   ├── kongtagandpush.sh
        │   └── kongtagandpush.sh~
        └── tasks
            ├── main.yaml
            └── main.yaml~

9 directories, 21 files
```
执行以下命令进行安装
```
ansible-playbook -i local.ini kong.yaml --private-key=~/.ssh/id_rsa --extra-vars "@roles/kong/defaults/all.yaml"
```
###nfs-client
文件目录结构如下
```
[root@portus nfsclient]# tree .
.
├── local.ini
├── local.ini~
├── nfsclient.retry
├── nfsclient.yaml
├── nfsclient.yaml~
└── roles
    ├── loadnfsclientimages
    │   ├── files
    │   │   ├── nfs
    │   │   │   └── 1.tar.xz
    │   │   └── nfsclienttagandpush.sh
    │   └── tasks
    │       ├── main.yaml
    │       └── main.yaml~
    └── nfs-client
        ├── defaults
        │   └── all.yaml
        ├── tasks
        │   └── main.yaml
        └── templates
            ├── nfs-client-clusterrolebinding.yaml
            ├── nfs-client-clusterrole.yaml
            ├── nfs-client-deployment.yaml
            ├── nfs-client-sc.yaml
            └── nfs-client-serviceaccount.yaml

9 directories, 16 files
```
执行以下命令进行安装
```
ansible-playbook -i local.ini nfsclient.yaml --private-key=/root/.ssh/id_rsa --extra-vars "@roles/nfs-client/defaults/all.yaml"
```
###helmcli
文件目录结构如下
```
[root@portus helmcli]# tree .
.
├── helmcli.yaml
├── helmcli.yaml~
├── local.ini
├── local.ini~
└── roles
    └── helmcli
        ├── files
        │   └── helm
        └── tasks
            ├── main.yaml
            └── main.yaml~

4 directories, 7 files
```
执行以下命令安装
```
ansible-playbook -i local.ini helmcli.yaml --private-key=/root/.ssh/id_rsa
```
###monitor
```
[root@portus monitor]# tree .
.
├── local.ini
├── local.ini~
├── monitor.retry
├── monitor.yaml
└── roles
    ├── loadmonitorimages
    │   ├── files
    │   │   ├── monitor
    │   │   │   ├── 10.tar.xz
    │   │   │   ├── 11.tar.xz
    │   │   │   ├── 1.tar.xz
    │   │   │   ├── 2.tar.xz
    │   │   │   ├── 3.tar.xz
    │   │   │   ├── 4.tar.xz
    │   │   │   ├── 5.tar.xz
    │   │   │   ├── 6.tar.xz
    │   │   │   ├── 7.tar.xz
    │   │   │   ├── 8.tar.xz
    │   │   │   ├── 9.tar.xz
    │   │   │   ├── pullimages.sh
    │   │   │   └── tagandpush.sh
    │   │   ├── monitortagandpush.sh
    │   │   └── tagandpush.sh~
    │   └── tasks
    │       ├── main.yaml
    │       └── main.yaml~
    └── monitor
        ├── defaults
        │   └── all.yaml
        ├── tasks
        │   └── main.yaml
        └── templates
            └── kube-prometheus-custom-values.yaml

9 directories, 24 files
```
执行以下命令安装
```
ansible-playbook -i local.ini monitor.yaml --private-key=/root/.ssh/id_rsa --extra-vars "@roles/monitor/defaults/all.yaml"
```
###logging
文件目录如下
```
[root@portus logging]# tree .
.
├── local.ini
├── local.ini~
├── logging.retry
├── logging.yaml
├── logging.yaml~
└── roles
    ├── loadloggingimages
    │   ├── files
    │   │   ├── logging
    │   │   │   ├── 1.tar.xz
    │   │   │   ├── 2.tar.xz
    │   │   │   ├── 3.tar.xz
    │   │   │   ├── 4.tar.xz
    │   │   │   └── 5.tar.xz
    │   │   └── loggingtagandpush.sh
    │   └── tasks
    │       ├── main.yaml
    │       └── main.yaml~
    └── logging
        └── tasks
            └── main.yaml

7 directories, 14 files
```
执行以下命令安装
```
ansible-playbook -i local.ini logging.yaml --private-key=/root/.ssh/id_rsa
```
###openebs
文件目录如下
```
```
执行以下命令安装
```
```
###istio
文件目录如下
```
```
执行以下命令安装
```
```
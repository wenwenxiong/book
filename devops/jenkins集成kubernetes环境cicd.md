参考网址： https://zhangchenchen.github.io/2017/12/17/achieve-cicd-in-kubernetes-with-jenkins/
   https://www.jianshu.com/p/2b7288e77ff8
   
###前提条件
已存在的kubernetes集群
已存在的docker镜像私有仓库
已存在的gitlab代码管理仓库

jenkins在kubernetes环境的部署文件
```
xww@xww-HP-EliteBook-8460p:~/jenkinsk8s$ tree .
.
├── 1.tar.xz
├── 2.tar.xz
├── 3.tar.xz
├── jenkins-deployment.yaml
├── jenkins-pvc.yaml
├── jenkins-service.yaml
└── pullimages.sh

0 directories, 7 files

```
1、```pullimage.sh```和```.xz```文件为下载jenkins相关docker镜像的脚本和镜像文件。
2、```.yaml```文件是jenkins在kubernetes的部署yaml文件。

###部署
1, 创建名为```cicd```的kubernetes namespace 。
2, 把jenkins镜像上传到docker私有镜像仓库，并且修改yaml文件中镜像名称和镜像拉取策略。
3, 执行以下命令创建jenkins
```
kuberctl create -f jenkins-pvc.yaml -f jenkins-deployment.yaml -f jenkins-service.yaml
```

###ansible自动部署
1、编写ansible playbook role loadijenkinsmages脚本将jenkins相关的镜像导入docker私有仓库
2、编写ansible playbook role jenkis脚本在目标kubernetes环境部署jenkins
文件如下
```
[root@portus jenkins]# tree .
.
├── jenkins.yaml
├── jenkins.yaml~
├── local.ini
├── local.ini~
└── roles
    ├── jenkins
    │   ├── defaults
    │   │   └── all.yaml
    │   ├── tasks
    │   │   ├── main.yaml
    │   │   └── main.yaml~
    │   └── templates
    │       ├── jenkins-deployment.yaml
    │       ├── jenkins-pvc.yaml
    │       └── jenkins-service.yaml
    └── loadjenkinsimages
        ├── files
        │   ├── jenkins
        │   │   ├── 1.tar.xz
        │   │   ├── 2.tar.xz
        │   │   └── 3.tar.xz
        │   ├── jenkinstagandpush.sh
        │   └── jenkinstagandpush.sh~
        └── tasks
            ├── main.yaml
            └── main.yaml~

9 directories, 17 files
```
执行以下命令安装
```
ansible-playbook -i local.ini jenkins.yaml --extra-vars "@roles/jenkins/defaults/all.yaml"
```
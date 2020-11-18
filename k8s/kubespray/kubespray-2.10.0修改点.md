###修改点
1、在```roles/kubernetes-apps/```文件夹下增加了```nfs```和```velero```两个文件夹，文件夹下保存了```nfs```和```velero```的```ansible role```。

2、修改```cluster.yml```文件。
在文件```cluster.yml```文件最后增加了以下代码。
```
- hosts: kube-deploy
  roles:
    - { role: kubernetes-apps/nfs/nfs-server, tags: nfs }

- hosts: k8s-cluster
  roles:
    - { role: kubernetes-apps/nfs/nfs-common, tags: nfs }

- hosts: kube-master[0]
  roles:
    - { role: kubernetes-apps/nfs/loadnfsimages, tags: nfs }
    - { role: kubernetes-apps/nfs/nfs-client, tags: nfs }
    - { role: kubernetes-apps/velero/loadveleroimages, tags: velero }
    - { role: kubernetes-apps/velero/velero, tags: velero }
```
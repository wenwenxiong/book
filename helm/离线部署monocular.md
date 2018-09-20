在使用kismatic部署完kubernetes集群后，只安装了helm的服务端tiller，helm客户端没有安装到集群环境中，因此在部署monocular时，首先安装和配置helm客户端。
因为是离线安装，需要配置一个helm chart repo源，这里采用chartmuseum来启动helm chart repo源。
###安装和配置chartmuseum
首先下载或拷贝chartmuseum二进制文件从部署节点到kubernetes集群master节点机上。
执行命令
```
chmod +x ./chartmuseum
mv ./chartmuseum /usr/local/bin
sudo mkdir chartstorage
sudo chartmuseum --debug --port=8988 --storage="local" --storage-local-rootdir="./chartstorage"
```
上传已经进行过修改的monocular相关的chart包到chartmuseum上
```
curl -L --data-binary "@nginx-ingress-0.20.3.tgz" http://portus.teligen.com:8988/api/charts

curl -L --data-binary "@mongodb-0.4.17.tgz" http://portus.teligen.com:8988/api/charts

curl -L --data-binary "@monocular-0.6.1.tgz" http://portus.teligen.com:8988/api/charts
```
主要的修改点是image的名称，image拉取策略，cpu内存资源配置。
###安装和配置helm客户端
首先下载或拷贝helm二进制文件从部署节点到kubernetes集群master节点机上。
执行命令配置helm的客户端
```
helm init --client-only --stable-repo-url http://portus.teligen.com:8988
```
因为是离线环境，所以配置stable chart repo为私有的chartmuseum管理helm chart repo源。
###离线安装monocular
首先编写自定义chart repo源的配置文件custom-repos.yaml
```
[root@rhel74 monocular]# cat custom-repos.yaml
api:
  config:
    repos:
      - name: chartmuseum
        url: http://192.192.189.37:8988
        source: http://192.192.189.37:8988/charts
```
注意，这里把portus.teligen.com换成主机的IP，防止monocular api pod容器不能解析主机名。
执行命令安装monocular
```
helm install  chartmuseum/nginx-ingress
helm install  chartmuseum/monocular -f custom-repos.yaml
```
使用命令查询molocular的ingress
```
[root@rhel74 monocular]# kubectl get ingress
NAME                    HOSTS     ADDRESS   PORTS     AGE
wishing-rat-monocular   *                   80        28m
```
找到ingress controller的访问入口，访问molocular的ui
```
[root@rhel74 monocular]# kubectl get svc
NAME                                        TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                              AGE
kilted-joey-nginx-ingress-controller        LoadBalancer   172.20.220.213   <pending>     80:30117/TCP,443:30216/TCP           18h
kilted-joey-nginx-ingress-default-backend   ClusterIP      172.20.2.220     <none>        80/TCP                               18h
kubernetes                                  ClusterIP      172.20.0.1       <none>        443/TCP                              17d
nfs-provisioner                             ClusterIP      172.20.238.224   <none>        2049/TCP,20048/TCP,111/TCP,111/UDP   14d
wishing-rat-mongodb                         ClusterIP      172.20.191.58    <none>        27017/TCP                            29m
wishing-rat-monocular-api                   NodePort       172.20.57.232    <none>        80:32262/TCP                         29m
wishing-rat-monocular-prerender             NodePort       172.20.182.184   <none>        80:31750/TCP                         29m
wishing-rat-monocular-ui                    NodePort       172.20.114.195   <none>        80:32024/TCP                         29m
```
访问https://192.192.189.31:30216即可访问monocular。
![monocular](./monocular/monocular2.png "monocular")
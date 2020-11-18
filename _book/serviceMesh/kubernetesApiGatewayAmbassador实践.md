参考网址： https://www.getambassador.io/user-guide/getting-started
###kubernetes部署ambassador
运行ambassador-service.yaml
```
kubectl apply -f ambassador-service.yaml
```
下载ambassador的deployment文件
```
wget https://getambassador.io/yaml/ambassador/ambassador-rbac.yaml
kubectl apply -f ambassador-rbac.yaml
```
运行例子
```
kubectl apply -f qotm.yaml
```
访问qotm
```
kubectl get svc
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
ambassador         NodePort    172.20.205.235   <none>        80:31986/TCP     23m
ambassador-admin   NodePort    172.20.162.212   <none>        8877:31264/TCP   19m
kubernetes         ClusterIP   172.20.0.1       <none>        443/TCP          6h

curl -v http://192.192.189.121:31986/qotm/
```
访问ambassador的ui界面
```
http://192.192.189.121:31264/ambassador/v0/diag/
```
###ambassador添加认证功能


###ambassador添加访问速率限制功能

###ambassador添加动态路由

###结合istio使用

###ambassador的统计，监控，告警
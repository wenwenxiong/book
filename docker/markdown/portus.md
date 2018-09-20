### 用户名/密码
https://192.192.189.144
baolh/123456aB
工作区间: telecom

### 步骤
以centos 7.3为例，需要运行以下配置， 
1\. 手动添加`portus.teligen.com`至`/etc/hosts`   

```
# vim /etc/hosts
......
192.192.189.144 portus.teligen.com
```

2\. 拷贝portus.crt至系统目录后，docker login：    

```
# cp portus.crt /etc/pki/ca-trust/source/anchors/
# update-ca-trust 
# docker login portus.teligen.com:5000
Username: baolh
Password: 
Login Succeeded
```

3\. tag / push 镜像至镜像仓库:    

```
# docker tag sonatype/nexus:3 portus.teligen.com:5000/telecom/sonatype/nexus:3
# docker push portus.teligen.com:5000/telecom/sonatype/nexus:3
The push refers to a repository [portus.teligen.com:5000/telecom/sonatype/nexus]
cd8a60667200: Pushed 
59289d78b8c4: Pushed 
32e104ecc8b6: Pushed 
b35d14f4a6a9: Pushed 
fa4ac6535770: Pushed 
b362758f4793: Pushed 
3: digest: sha256:3cdaf4b23857b7b4a30aac562df55e241f50e5e1d3e13052f8b1d9a1b0dd3733 size: 1579
```

刷新`https://192.192.189.144`的dashboard，可以看到nexus镜像已经上传成功:    

![./images/portus.jpg](./images/portus.jpg)

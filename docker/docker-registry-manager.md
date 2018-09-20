参考网址：https://github.com/snagles/docker-registry-manager
###下载镜像
镜像名称为```snagles/docker-registry-manager```。
```
#!/bin/bash

set -x
dockerimages=(
snagles/docker-registry-manager
)

j=1
for i in ${dockerimages[@]}
do
    echo $i
    echo $j

    docker pull $i && docker save $i -o $j.tar && xz $j.tar
    docker rmi $i
    let j+=1
done
set +x

```

配置文件registry.yml
```
registries:
  exampleRegistry:
    url: http://192.168.122.1
    port: 6000 # Example: 443, 8080, 5000
    username: admin
    password: admin123
    refresh-rate: "5m" # Example: 60s, 5m, 1h
    skip-tls-validation: true # REQUIRED for self signed certificates
    dockerhub-integration: true # Optional - compares to dockerhub to determine if image up to date
```
docker-compose配置文件
```
version: '2'

services:
  docker-registry-manager:
    container_name: docker-registry-manager
    image: snagles/docker-registry-manager
    ports:
      - "8080:8080"
    volumes:
      - ./registries.yml:/app/registries.yml
      #- ./ssl.crt:/app/ssl.crt # https certfile location
      #- ./ssl.key:/app/ssl.key # https keyfile location

    environment:
      - MANAGER_PORT=8080
      - MANAGER_REGISTRIES=/app/registries.yml
      - MANAGER_LOG_LEVEL=warn
      #- MANAGER_ENABLE_HTTPS=true
      #- MANAGER_KEY=/app/ssl.crt
      #- MANAGER_CERTIFICATE=/app/ssl.key
```
###构建及运行docker-registry-manager服务
使用docker-compose命令运行服务
```
docker-compose up -d
```
使用docker-compose命令停止删除服务
```
docker-compose down
```
优点，界面美观，可以查看，搜索镜像。
问题： 极不稳定，查看详细信息卡，无法在界面增加删除镜像。
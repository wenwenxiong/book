参考网址：https://github.com/big-data-europe/docker-hive

###下载相关镜像

```
docker pull bde2020/hadoop-namenode:1.1.0-hadoop2.8-java8
docker pull bde2020/hadoop-datanode:1.1.0-hadoop2.8-java8
docker pull bde2020/hive-metastore-postgresql:2.1.0
docker pull shawnzhu/prestodb:0.181
```

克隆项目到本地
```
git clone https://github.com/big-data-europe/docker-hive.git
```
###构建及运行hive服务

```
docker-compose build
docker-compose up -d namenode hive-metastore-postgresql
docker-compose up -d datanode hive-metastore
docker-compose up -d hive-server
docker-compose up -d presto-coordinator
```
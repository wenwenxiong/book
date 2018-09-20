kafka概念

broke
kafka集群中包含一个或多个服务器，这种服务器被称为broker
topic
每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。（物理上不同Topic的消息分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）
Partition
Parition是物理上的概念，每个Topic包含一个或多个Partition.
producter
负责发布消息到kafka broker
consumer
消息消费者，向kafka broker读取消息的客户端
consumer group
每个Consumer属于一个特定的Consumer Group（可为每个Consumer指定group name，若不指定group name则属于默认的group）

####kafka docker镜像
参考网址：https://github.com/big-data-europe/docker-kafka
https://github.com/big-data-europe/pilot-sc6-cycle2/tree/master/sc6-kafka
https://github.com/big-data-europe/docker-zookeeper
https://github.com/big-data-europe/pilot-sc6-cycle2/tree/master/sc6-zookeeper

kafka集群构建基于zookeeper集群。zookeeper base镜像中安装了zookeeper,并且初始命令中启动zookeeper集群，但是配置文件还是默认的配置（https://github.com/big-data-europe/docker-zookeeper）。在zookeeper base镜像基础上，增加特定的配置文件，就可以构建自己需要的zookeeper镜像。
#####zookeeper镜像
```
FROM bde2020/zookeeper:latest

MAINTAINER Juergen Jakobitsch <jakobitschj@semantic-web.at>

ADD zoo.cfg /config/
ADD zoo_replicated1.cfg.dynamic /config/
```
zoo.cfg
```
standaloneEnabled=false
dataDir=/tmp/zookeeper
syncLimit=2
initLimit=5
tickTime=2000
dynamicConfigFile=/config/zoo_replicated1.cfg.dynamic
```
zoo_replicated1.cfg.dynamic
```
server.1=sc6_zoo_1:31200:31201:participant;31202
server.2=sc6_zoo_2:31200:31201:participant;31202
server.3=sc6_zoo_3:31200:31201:participant;31202
```
#####kafka镜像
kafka base镜像中安装了kafka，并且在镜像中定义了基于kafka-startup.json配置文件来启动kafka，基于kafka-init.json来创建topic（实验中没有成功，还是需要手动执行脚本）。
```
FROM bde2020/kafka
ADD kafka-startup.json /config/
ADD kafka-init.json /config/
```
kafka-startup.json
```
[
  {
    "sh":"/app/bin/kafka-server-start.sh",
    "./config/server.properties":"",
    "--override":"-Djava.net.preferIPv4Stack=true",
    "--override":"delete.topic.enable=true",
    "--override":"advertised.host.name=$HOSTNAME",
    "--override":"advertised.port=9092",
    "--override":"zookeeper.connect=sc6_zoo_1:31202,sc6_zoo_2:31202,sc6_zoo_3:31202/kafka"
  }
]
```
kafka-init.json
```
[
  {
    "sh":"/app/bin/kafka-topics.sh",
    "--zookeeper":"sc6_zoo_1:31202,sc6_zoo_2:31202,sc6_zoo_3:31202/kafka",
    "--create":"",
    "--topic":"sampleTopic",
    "--partitions":"3",
    "--replication-factor":"1"
  }
]
```
###docker-compose编排kafka集群
```
version: "2"

services:
  sc6_zoo_1:
    image: "bde2020/sc6-zookeeper"
    ports:
      - 31200:31200
      - 31201:31201
      - 31202:31202
    container_name: sc6_zoo_1
    command: "bash -c /startup"
    hostname: "sc6_zoo_1"
  sc6_zoo_2:
    image: "bde2020/sc6-zookeeper"
    ports:
      - 32200:31200
      - 32201:31201
      - 32202:31202
    container_name: sc6_zoo_2
    command: "bash -c /startup"
    hostname: "sc6_zoo_2"
  sc6_zoo_3:
    image: "bde2020/sc6-zookeeper"
    ports:
      - 33200:31200
      - 33201:31201
      - 33202:31202
    container_name: sc6_zoo_3
    command: "bash -c /startup"
    hostname: "sc6_zoo_3"
  sc6-kafka-1:
    image: "xiongwen/kafka:v2"
    depends_on:
      - sc6_zoo_1
      - sc6_zoo_2
      - sc6_zoo_3
    ports:
      - 9092:9092
    container_name: sc6-kafka-1
    command: "bash -c /app/bin/kafka-init"
    hostname: "sc6-kafka-1"
```
进入sc6-kafka-1容器中，手动创建topic，运行生产者生产消息，运行消费者进行测试。
创建sampleTopic
```
/usr/bin/python3 /app/bin/kafka-bin.py /config/kafka-init.json
```
生产与消费消息
```
/usr/local/apache-kafka/current/bin/kafka-console-producer.sh --topic sampleTopic --broker-list sc6-kafka-1:9092
```
输入消息。
```
/usr/local/apache-kafka/current/bin/kafka-console-consumer.sh --topic sampleTopic --zookeeper sc6_zoo_1:31202/kafka --bootstrap-server sc6-kafka-1:9092 --from-beginning
```
运行后显示消息。
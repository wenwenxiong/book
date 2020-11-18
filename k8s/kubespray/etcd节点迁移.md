步骤如下：
1、停止需要被迁移的```etcd```进程
2、拷贝被迁移的```etcd```进程数据目录到新增的节点上
3、更新etcd集群中需要迁移节点的```peer URLs```等元数据信息为新的```etcd```节点
4、以相同的配置和数据目录启动新节点上的```etcd```进程

1、停止需要被迁移的```etcd```进程
列出```ectd```成员
```
etcdctl --endpoints=https://192.168.124.171:2379,https://192.168.124.172:2379,https://192.168.124.173:2379 --ca-file=/etc/ssl/etcd/ssl/ca.pem --key-file=/etc/ssl/etcd/ssl/member-node01-key.pem --cert-file=/etc/ssl/etcd/ssl/member-node01.pem member list
```
停止```192.168.124.171```节点上的```etcd```服务
```
kill `pgrep etcd`
```
2、拷贝被迁移的```etcd```进程数据目录到新增的节点上
从配置文件```/etc/etcd.env```查找到数据目录为```ETCD_DATA_DIR=/var/lib/etcd```。
```
tar -cvzf infra1.etcd.tar.gz /var/lib/etcd
$ scp infra1.etcd.tar.gz 192.168.124.174:~/
```
3、更新etcd集群中需要迁移节点的```peer URLs```等元数据信息为新的```etcd```节点
```
$ curl http://10.0.1.10:2379/v2/members/b4db3bf5e495e255 -XPUT \
-H "Content-Type: application/json" -d '{"peerURLs":["http://10.0.1.13:2380"]}'
```
或者使用命令```etcdctl member update command```。
```
$ etcdctl --endpoints=https://192.168.124.171:2379,https://192.168.124.172:2379,https://192.168.124.173:2379 --ca-file=/etc/ssl/etcd/ssl/ca.pem --key-file=/etc/ssl/etcd/ssl/member-node01-key.pem --cert-file=/etc/ssl/etcd/ssl/member-node01.pem member update b4db3bf5e495e255 http://10.0.1.13:2380
```
4、以相同的配置和数据目录启动新节点上的```etcd```进程
```
$ ssh 10.0.1.13
$ tar -xzvf infra1.etcd.tar.gz -C %data_dir%
etcd -name infra1 \
-listen-peer-urls http://10.0.1.13:2380 \
-listen-client-urls http://10.0.1.13:2379,http://127.0.0.1:2379 \
-advertise-client-urls http://10.0.1.13:2379,http://127.0.0.1:2379
```
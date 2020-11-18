### 1) 修改配置项

在部署节点上，修改`Rong`目录下的`rong-vars.yml`文件的配置项

```
fluentd_enable: true
fluentd_elasticsearch_host: "" #自己准备的es集群IP，例如 "192.192.190.149"
fluentd_elasticsearch_port: "9200"
kibana_enable: true
kibana_elasticsearch_host: "" #自己准备的es集群IP，例如 "192.192.190.149"
kibana_elasticsearch_port: "9200"
```



### 2）安装

在部署节点上，在`Rong`目录下

拷贝`rong-vars.yml`到`rong/4_addons/rong-vars.yml`。

```
# cp -f rong-vars.yml rong/4_addons/rong-vars.yml
```

运行以下命令安装`fluentd`。

```
# ansible-playbook  -i hosts.ini rong/4_addons/addons.yml --extra-vars \ @rong/4_addons/rong-vars.yml --tags efk_fluentd
```

运行以下命令安装`kibana`。

```
# ansible-playbook  -i hosts.ini rong/4_addons/addons.yml --extra-vars \ @rong/4_addons/rong-vars.yml --tags efk_kibana
```


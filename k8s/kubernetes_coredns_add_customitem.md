### 修改coredns的configmap

在`coredns`的`configmap`加入以下内容满足两个作用

1）、配置`corefile`的`hosts`插件使用文件`/etc/coredns/hosts`

2）、`configmap`中写入`hosts`文件内容，最终文件`hosts`会挂载到`pod`的`/etc/coredns/hosts`

```
Corefile: |
...
  hosts /etc/coredns/hosts {
          fallthrough
        }
...
hosts: |
  192.192.189.237 xww.example.com
```



### 修改coredns的deployment

修改`coredns`的`deployment`实现挂载`configmap`中的`hosts`文件到`pod`中

```
 volumes:
        - name: config-volume
          configMap:
            name: coredns
            items:
            - key: Corefile
              path: Corefile
            - key: hosts  #这是新增的
              path: hosts  #这是新增的

```


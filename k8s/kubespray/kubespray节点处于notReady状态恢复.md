###问题描述

在修改节点时间后，节点与```apiserver```的认证应为时间不一致导致证书有误，认证失败。因此，```apiserver```会判断节点的状态处于```notReady```。并且会伴随着故障节点```nginx-proxy pod```删除掉。此时会一直报错，```kubelet```无法与```127.0.0.1：6443```连接。（容器```nginx-proxy pod```监听```127.0.0.1：6443```端口，后端配置的是```apiserver```的端口）

###解决办法

首先，把节点之间的时间进行同步。

其次，故障节点```nginx-proxy pod```已被删除，因此无法通过```127.0.0.1：6443```端口访问```apiserver```。需要修改文件```/etc/kubernetes/kubelet.conf```把配置```server```设置为```apiserver```的实际端口。重启```kubelet```。

最后，```kubelet```重启后，```nginx-proxy pod```会自动启动，在次修改文件```/etc/kubernetes/kubelet.conf```把配置```server```设置为```127.0.0.1：6443```端口。再次重启```kubelet```，至此，问题解决。配置恢复到故障前环境配置一致。
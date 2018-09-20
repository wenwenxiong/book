参考网址： http://cilium.readthedocs.io/en/latest/kubernetes/quickinstall/

在kubeadm安装kubernetes1.11版本上安装成功
config.yaml
```
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.0
kubeProxy:
  config:
    featureGates: 
      SupportIPVSProxyMode: true
    mode: ipvs
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
```
国内kubeadm的kubernetes源配置（ubuntu）。
```
#kubernetes
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg |apt-key add -
cat<< EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
```

###前提
linux内核4.10以上（实验环境采用的是ubuntu-18.04.1-live-server-amd64.iso）
在各个节点执行命令挂载BPF 文件系统
```
mount bpffs /sys/fs/bpf -t bpf
```

###安装
首先 安徽cilium的键值存储etcd（也可以使用现存的etcd）。
```
wget https://raw.githubusercontent.com/cilium/cilium/HEAD/examples/kubernetes/addons/etcd/standalone-etcd.yaml

kubectl create -n kube-system -f standalone-etcd.yaml
```
安装cilium
```
wget https://raw.githubusercontent.com/cilium/cilium/HEAD/examples/kubernetes/1.11/cilium.yaml

kubectl create -n kube-system -f cilium.yaml
```
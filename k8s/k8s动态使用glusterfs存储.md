###k8s中运行Heketi服务

在使用kismatic安装k8s和glusterfs后，需要配置k8s默认动态管理（provisioning）使用glusterfs，以下内容是实现这样的目的。
Hekeli：glusterfs的restful端点，k8s向它发送命令来使用glusterfs。

####Hekeli cli安装
下载Hekeli cli工具，安装在$PATH路径下
下载地址 https://github.com/heketi/heketi/releases/

####Hekeli 服务端安装在k8s环境
在所有的glusterfs节点，创建hekeli的数据库存储目录、ssh免密码登录文件目录。
```
mkdir -p /data/heketi/{db,.ssh} && chmod 700 /data/heketi/.ssh
```
生成ssh密钥
```
ssh-keygen -t rsa -b 2048 -f /data/heketi/.ssh/id_rsa

for NODE in node1 node2 node3; do scp -r /data/heketi/.ssh root@${NODE}:/data/heketi; done

for NODE in node1 node2 node3; do ssh-copy-id -i /data/heketi/.ssh/id_rsa.pub root@${NODE} ; done
```
k8s运行heketi的服务
```
kubectl apply -f heketi-secret.yaml -f heketi-deployment.json
```
heketi-secret.yaml文件内容
```
apiVersion: v1
kind: Secret
metadata:
  name: heketi-secret
  namespace: default
data:
  # base64 encoded password. E.g.: echo -n "password" | base64
  key: cGFzc3dvcmQ=
type: kubernetes.io/glusterfs
```
heketi-deployment.json文件内容
```
{
  "kind": "List",
  "apiVersion": "v1",
  "items": [
    {
      "kind": "Service",
      "apiVersion": "v1",
      "metadata": {
        "name": "heketi",
        "labels": {
          "glusterfs": "heketi-service",
          "deploy-heketi": "support"
        },
        "annotations": {
          "description": "Exposes Heketi Service",
          "tags": "kubernetes,k8s,heketi",
          "traefik.backend.loadbalancer": "wrr",
          "traefik.backend.weight": "10",
          "traefik.enable": "true",
          "traefik.frontend.entryPoints": "http,https",
          "traefik.frontend.rule": "Host:heketi-api.example.com",
          "traefik.tags": "kubernetes"
        }
      },
      "spec": {
        "type": "NodePort",
        "selector": {
          "name": "heketi"
        },
        "ports": [
          {
            "name": "heketi",
            "port": 8080,
            "targetPort": 8080,
            “nodePort”: 30944
          }
        ]
      }
    },
    {
      "kind": "Deployment",
      "apiVersion": "apps/v1beta1",
      "metadata": {
        "name": "heketi",
        "labels": {
          "glusterfs": "heketi-deployment"
        },
        "annotations": {
          "description": "Defines how to deploy Heketi"
        }
      },
      "spec": {
        "replicas": 1,
        "strategy": {
          "rollingUpdate": {
            "maxSurge": 0,
            "maxUnavailable": 1
          },
          "type": "RollingUpdate"
        },
        "template": {
          "metadata": {
            "name": "heketi",
            "labels": {
              "name": "heketi",
              "glusterfs": "heketi-pod"
            }
          },
          "spec": {
            "terminationGracePeriodSeconds": 0,
            "nodeSelector": {
              "storagenode": "glusterfs"
            },
            "containers": [
              {
                "image": "heketi/heketi:5",
                "imagePullPolicy": "Always",
                "name": "heketi",
                "env": [
                  {
                    "name": "HEKETI_EXECUTOR",
                    "value": "ssh"
                  },
                  {
                    "name": "HEKETI_SSH_USER",
                    "value": "root"
                  },
                  {
                    "name": "HEKETI_SSH_PORT",
                    "value": "22"
                  },
                  {
                    "name": "HEKETI_SSH_KEYFILE",
                    "value": "/root/.ssh/id_rsa"
                  },
                  {
                    "name": "HEKETI_ADMIN_KEY",
                    "valueFrom": {
                      "secretKeyRef": {
                        "name": "heketi-secret",
                        "key": "key"
                      }
                    }
                  }
                ],
                "ports": [
                  {
                    "containerPort": 8080
                  }
                ],
                "volumeMounts": [
                  {
                    "name": "heketi-ssh-key",
                    "mountPath": "/root/.ssh"
                  },
                  {
                    "name": "heketi-db",
                    "mountPath": "/var/lib/heketi"
                  }
                ],
                "readinessProbe": {
                  "timeoutSeconds": 3,
                  "initialDelaySeconds": 3,
                  "httpGet": {
                    "path": "/hello",
                    "port": 8080
                  }
                },
                "livenessProbe": {
                  "timeoutSeconds": 3,
                  "initialDelaySeconds": 15,
                  "httpGet": {
                    "path": "/hello",
                    "port": 8080
                  }
                }
              }
            ],
            "volumes": [
              {
                "name": "heketi-ssh-key",
                "hostPath": {
                  "path": "/data/heketi/.ssh"
                }
              },
              {
                "name": "heketi-db",
                "hostPath": {
                  "path": "/data/heketi/db"
                }
              }
            ]
          }
        }
      }
    }
  ]
}

```
这里配置Heketi的service在NodeIP：30944上
通过命令检查heketi服务
```
curl -s http://nodeIP:30944/hello
```
通过cli连接
```
heketi-cli --user admin --secret password --server http://node1:30944 cluster list
```
导入glusterfs集群拓扑（topology）信息
```
heketi-cli --user admin --secret password --server http://node1:30944 topology load --json heketi-topology.json
```
heketi-topology.json的文件内容为
```
{
  "clusters": [
    {
      "nodes": [
        {
          "node": {
            "hostnames": {
              "manage": [
                "192.168.122.112"
              ],
              "storage": [
                "192.168.122.112"
              ]
            },
            "zone": 1
          },
          "devices": [
            "/dev/vdb"
          ]
        }
      ]
    }
  ]
}

```
有关glusterfs集群的topology的配置参考
https://github.com/heketi/heketi/blob/master/docs/admin/topology.md
使用以下命令获得cluster id
```
heketi-cli --user admin --secret password --server http://192.168.122.112:30944 cluster list
```
###k8s使用glusterfs
k8s上定义glusterds的default storageclass，然后通过pvc来动态使用glusterfs存储
```
kubectl apply -f heketi-storageclass.yaml
```
heketi-storageclass.yaml的内容如下
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: "http://192.168.122.112:30944"
  clusterid: "e4b83af0a6260975ed1001ea3e89a0ad"
  restuser: "admin"
  secretNamespace: "default"
  secretName: "heketi-secret"
  volumetype: "none"
```
```
kubectl apply -f heketi-pvc.yaml
kubectl get pvc test-claim
kubectl get pv
```
heketi-pvc.yaml的文件内容
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-claim
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```
使用命令查看glusterfs中的volume
```
heketi-cli --user admin --secret password --server http://192.168.122.112:30944 volume list
```
创建pod使用pvc
```
kubectl apply -f nginx-pod.yaml
```
nginx-pod.yaml的内容
```
kind: Pod
apiVersion: v1
metadata:
  name: nginx-with-pv
spec:
  containers:
    - name: frontend
      image: nginx:stable-alpine
      volumeMounts:
      - mountPath: "/usr/share/nginx/html"
        name: pv
  nodeSelector:
    storagenode: "glusterfs"
  volumes:
    - name: pv
      persistentVolumeClaim:
        claimName: test-claim
```

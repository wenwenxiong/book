###k8s中部署nfs-provisioner存储
参考网址：https://github.com/kubernetes-incubator/external-storage/tree/master/nfs
https://github.com/kubernetes-incubator/external-storage/blob/master/nfs/docs/authorization.md
创建nfs-provisioner的auth。
创建nfs-provisioner的```serviceaccount.yaml```。
serviceaccount.yaml
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-provisioner
```
创建nfs-provisioner的```PodSecurityPolicy```。
psp.yaml
```
apiVersion: extensions/v1beta1
kind: PodSecurityPolicy
metadata:
  name: nfs-provisioner
spec:
  fsGroup:
    rule: RunAsAny
  allowedCapabilities:
  - DAC_READ_SEARCH
  - SYS_RESOURCE
  runAsUser:
    rule: RunAsAny
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  volumes:
  - configMap
  - downwardAPI
  - emptyDir
  - persistentVolumeClaim
  - secret
  - hostPath
```
创建nfs-provisioner的```ClusterRole```。
clusterrole.yaml
```
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["list", "watch", "create", "update", "patch"]
  - apiGroups: [""]
    resources: ["services", "endpoints"]
    verbs: ["get"]
  - apiGroups: ["extensions"]
    resources: ["podsecuritypolicies"]
    resourceNames: ["nfs-provisioner"]
    verbs: ["use"]
```
创建nfs-provisioner的```ClusterRoleBinding```把名为```nfs-provisioner-runner```的```ClusterRole```绑定到名为```run-nfs-provisioner```的```ServiceAccount```。
clusterrolebinding.yaml
```
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-provisioner
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
```
nfs-provisioner的deployment文件为deployment-sa.yaml。
```
kind: Service
apiVersion: v1
metadata:
  name: nfs-provisioner
  labels:
    app: nfs-provisioner
spec:
  ports:
    - name: nfs
      port: 2049
    - name: mountd
      port: 20048
    - name: rpcbind
      port: 111
    - name: rpcbind-udp
      port: 111
      protocol: UDP
  selector:
    app: nfs-provisioner
---
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: nfs-provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate 
  template:
    metadata:
      labels:
        app: nfs-provisioner
    spec:
      serviceAccount: nfs-provisioner
      containers:
        - name: nfs-provisioner
          image: quay.io/kubernetes_incubator/nfs-provisioner:v1.0.8
          ports:
            - name: nfs
              containerPort: 2049
            - name: mountd
              containerPort: 20048
            - name: rpcbind
              containerPort: 111
            - name: rpcbind-udp
              containerPort: 111
              protocol: UDP
          securityContext:
            capabilities:
              add:
                - DAC_READ_SEARCH
                - SYS_RESOURCE
          args:
            - "-provisioner=example.com/nfs"
          env:
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
            - name: SERVICE_NAME
              value: nfs-provisioner
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          imagePullPolicy: "IfNotPresent"
          volumeMounts:
            - name: export-volume
              mountPath: /export
      volumes:
        - name: export-volume
          hostPath:
            path: /srv
```
使用节点上的```/srv```目录作为nfs-provisioner的存储。配置的provisioner名称为
```
args:
  - "-provisioner=example.com/nfs"
```
###kong使用nfs-provisioner
修改kong使用的数据库cassandra或者postgres，这里使用的是cassandra。
修改cassandra.yaml使用的volume。
```
...
...
...
  volumeClaimTemplates:
  - metadata:
      name: cassandra-data
      annotations:
        volume.beta.kubernetes.io/storage-class: example-nfs
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
---
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: example-nfs
provisioner: example.com/nfs
#parameters:
#  type: pd-ssd

```
使用下面命令部署kong
```
kubectl create -f cassandra.yaml

kubectl create -f kong_migration_cassandra.yaml

kubectl create -f kong_cassandra.yaml
```

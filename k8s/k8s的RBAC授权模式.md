###RBAC
RBAC声明4种类型：Role，ClusterRole，RoleBinding，ClusterRoleBinding。

Role： 对单个namespace下的resource资源赋予操作权限。
例如，创建一个可以read名字空间“default”下pods的Role。
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```
ClusterRole：与Role一样，可以创建资源的访问权限，但是，它的范围更广，可以定义的范围有
* cluster-scope资源（nodes）
* non-resource endpoints (例如 “/healthz”)
* 名字空间 resources (例如 pods) ，所有名字空间
例如，创建一个读访问指定namespace或所有namspace下的secrets的ClusterRole。

```
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

RoleBinding与ClusterRoleBinding
RoleBinding 把定义资源访问权限的Role赋予给一个或多个subjects(users，groups，service accounts)。RoleBinding限定用户在namespace范围内。
例如，名字空间"default"下user为"jane"绑定Role为名字空间“default”下"pod-reader"。
```
# This role binding allows "jane" to read pods in the "default" namespace.
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```
RoleBinding也可以绑定ClusterRole，但是会限定在RoleBinding指定的namespace下。

ClusterRoleBinding可以把范围扩展到整个集群和所有名字空间
例如，赋予group组“manager”下任意用户读取所有namespaces下secrets。
```
# This cluster role binding allows anyone in the "manager" group to read secrets in any namespace.
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: manager # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```
特殊的Role 例子
使用resourceNames指定特定名称的资源
```
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  resourceNames: ["my-config"]
  verbs: ["get"]
```
使用nonResourceURLs指定url
```
rules:
- nonResourceURLs: ["/healthz", "/healthz/*"] # '*' in a nonResourceURL is a suffix glob match
  verbs: ["get", "post"]
```
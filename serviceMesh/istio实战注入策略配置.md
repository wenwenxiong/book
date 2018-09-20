默认istio的自动注入sidecar功能在具有标签istio-injection=enabled的namespace中实现。它是通过配置kubernetes的```admissionregistration.k8s.io/v1beta1#MutatingWebhookConfiguration ```配置的。
用户可以在```install/kubernetes/istio-sidecar-injector-with-ca-bundle.yaml```文件中修改```MutatingWebhookConfiguration```的配置。
```
---
apiVersion: admissionregistration.k8s.io/v1beta1
kind: MutatingWebhookConfiguration
metadata:
  name: istio-sidecar-injector
webhooks:
  - name: sidecar-injector.istio.io
    clientConfig:
      service:
        name: istio-sidecar-injector
        namespace: istio-system
        path: "/inject"
      caBundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQVRBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwdGFXNXAKYTNWaVpVTkJNQjRYRFRFNE1ETXlOakEyTWpNME0xb1hEVEk0TURNeU16QTJNak0wTTFvd0ZURVRNQkVHQTFVRQpBeE1LYldsdWFXdDFZbVZEUVRDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBS21KCm83UGVsR1RhUkxHMVIxWkRxYUhlNVlJK2FwbTloTVl3Z1ZrUStoeW1XTmU2QzRjRmRSN0ZuQmxUWjAvNXlHa0UKMWl4dkdqRlU4N0ROblhpc1ZxZFBaMnh4RjBHTWlkTVpMT3lXNytqc3NhVUhDQUFyMEV6NE5OSkQ2QzdEMHQ4Ygp6MEszK2hmRzBxSnEraE1kQ293WFRjQXhKL1BRTFBKemVEZjRxSUJ2NGltWU1SU3hMWVA0cmp5cTM0WTk2cXhWCk9IUXFiaDBxaXhxcTdqcnVoZUs0a3hCSVF2SkxIOG9TZ2draWJJZ2ZnZFpTY1E2OXg2RFZvRDlRQlVUM2JjSEUKQzRnM2ZVaEZYWCtpODl3ajdUS2pvcSt3UGQvZ0c5aytveDF1K3NSWm9uM2F5ZkVGbDRaOG02UnBwVlAvb2xqegpiOHpmWFd1UnFVSHBvazJ3MWVrQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUIwR0ExVWRKUVFXCk1CUUdDQ3NHQVFVRkJ3TUNCZ2dyQmdFRkJRY0RBVEFQQmdOVkhSTUJBZjhFQlRBREFRSC9NQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFCMGdIcDZ3ViswM2RubWl1cVE1TEVnUkk1UlorNStCaFJGbkZaM2NNbmE1dVFKNk1CVApHMWhGcUxoNVd5OUU3ejk3bVRmVVpxL0tGUTYwN2NjMlZQUXJTa3BwTEEzaXI2YWFZeWU5RFh2VEUzMzJmaitpCmpIUWY4SHpJTExleldTRHB1U1U0amgrQm5nYWgydWpGMUpmN2Nmc2lDTkhyWmVPY0J1cnU5RlpGUXk4TStHRUMKa0txemRjMm1qenRHcVdqTHZ1YzR6MnlzQUVWdlZiNm02bWhZN2R5TytZL1lMQk9SVGk3WXZiLzROdVNKRHVGQgp5K1I1RWFqTFVhRDR3RjNpL1hTRGxUdWd3Q0Jxck9vekxvVDNPbUtNNGlpSXJPbmQxai9kS3NZRTFiQTZNcjJRCitQaVNJRkhwM3hEQ2l2Y2RualR1K3czWXRrZE5oZWxQQ2FJSwotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    rules:
      - operations: [ "CREATE" ]
        apiGroups: [""]
        apiVersions: ["v1"]
        resources: ["pods"]
    failurePolicy: Fail
    namespaceSelector:
      matchLabels:
        istio-injection: enabled
---
```
此外，在名为istio-inject的configmap中有配置默认的injection policy 和 sidecar injection template。
```
apiVersion: v1
data:
  config: |
    policy: enabled
    template: |-
      initContainers:
      - name: istio-init
        image: docker.io/istio/proxy_init:0.7.1
        args:
        - "-p"
        - {{ .MeshConfig.ProxyListenPort }}
        - "-u"
        - 1337
        imagePullPolicy: IfNotPresent
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
        restartPolicy: Always
      containers:
      - name: istio-proxy
        image: docker.io/istio/proxy:0.7.1
        args:
        - proxy
        - sidecar
...
```
* policy的取值有两种enabled和disabled。默认为enabled。
  1. enabled的策略为：在具有标签istio-injection=enabled的namespace中，创建pod时则注入sidecar，可以在pod的annotation添加```sidecar.istio.io/inject: "false"```来阻止注入sidecar。
  2. disabled的策略为：在具有标签istio-injection=enabled的namespace中，创建pod时不注入注入sidecar，可以在pod的annotation添加```sidecar.istio.io/inject: "true"```来注入sidecar。
一般选着enabled策略。
* sidecar injection template
可以在template里面修改镜像的名字，以达到使用私有仓库中镜像的目的，还有其他的功能待挖掘。
* istio-inject的configmap中有定义的变量值来源于istio-system名字空间下的istio configmap的变量取值。可以使用命令查看对比取值
```
kubectl get configmap istio-inject -n istio-system -o yaml
kubectl get configmap istio -n istio-system -o yaml
```
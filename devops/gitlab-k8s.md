####gitlab install in docker
下载镜像
```
docker pull gitlab/gitlab-ce
docker pull gitlab/gitlab-runner
```
运行容器服务
```
docker run --detach \
    --hostname gitlab.example.com \
    --publish 8929:80 --publish 2289:22 \
    --name gitlab \
    --restart always \
    --volume /media/xww/sda1/myproject/gitlab-ce/gitlab/config:/etc/gitlab \
    --volume /media/xww/sda1/myproject/gitlab-ce/gitlab/logs:/var/log/gitlab \
    --volume /media/xww/sda1/myproject/gitlab-ce/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
```
运行gitlab-runner 容器服务
```
docker run -d --name gitlab-runner --restart always \
  -v /media/xww/sda1/myproject/gitlab-ce/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```
编辑docker-compose.yml文件（/media/xww/sda1是我机器的一块盘）
```
gitlab-ce:
  image: 'gitlab/gitlab-ce:latest'
  restart: always
  hostname: 'gitlab.xiongwen.com'
  ports:
    - '8082:80'
    - '2224:22'
  volumes:
    - '/media/xww/sda1/myproject/gitlab-ce/gitlab/config:/etc/gitlab'
    - '/media/xww/sda1/myproject/gitlab-ce/gitlab/logs:/var/log/gitlab'
    - '/media/xww/sda1/myproject/gitlab-ce/gitlab/data:/var/opt/gitlab'
gitlab-runner:
  image: 'gitlab/gitlab-runner:latest'
  restart: always
  volumes:
    - '/var/run/docker.sock:/var/run/docker.sock'
    - '/media/xww/sda1/myproject/gitlab-runner/config:/etc/gitlab-runner'

```
在k8s中运行gitlab-runner
configmap.yml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: gitlab-runner
  namespace: gitlab
data:
  config.toml: |
    concurrent = 4

    [[runners]]
      name = "Kubernetes Runner"
      url = "https://gitlab.com/ci"
      token = "...."
      executor = "kubernetes"
      [runners.kubernetes]
        namespace = "gitlab"
        image = "busybox"
```
gitlab-runner.yml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: gitlab-runner
  namespace: gitlab
spec:
  replicas: 1
  selector:
    matchLabels:
      name: gitlab-runner
  template:
    metadata:
      labels:
        name: gitlab-runner
    spec:
      containers:
      - args:
        - run
        image: gitlab/gitlab-runner:latest
        imagePullPolicy: Always
        name: gitlab-runner
        volumeMounts:
        - mountPath: /etc/gitlab-runner
          name: config
        - mountPath: /etc/ssl/certs
          name: cacerts
          readOnly: true
      restartPolicy: Always
      volumes:
      - configMap:
          name: gitlab-runner
        name: config
      - hostPath:
          path: /usr/share/ca-certificates/mozilla
        name: cacerts
```

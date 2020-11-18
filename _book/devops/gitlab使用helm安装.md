参考网址: https://docs.gitlab.com/ee/install/kubernetes/gitlab_chart.html

###gitlab-ce chart
配置helm chart镜像使用镜像拉取策略为```IfNotPresent```.
配置默认的持久化存储
```
 helm upgrade --install gitlab ./gitlab --timeout 600 --set global.hosts.domain=example.com --set global.hosts.externalIP=10.10.10.10 --set certmanager-issuer.email=email@example.com --set gitlab.migrations.image.repository=registry.gitlab.com/gitlab-org/build/cng/gitlab-rails-ce --set gitlab.sidekiq.image.repository=registry.gitlab.com/gitlab-org/build/cng/gitlab-sidekiq-ce --set gitlab.unicorn.image.repository=registry.gitlab.com/gitlab-org/build/cng/gitlab-unicorn-ce
```
###gitlab auto devops
参考网址： https://docs.gitlab.com/ee/topics/autodevops/index.html#auto-sast
gitlab包含了auto devops全过程，具体如下
* auto build
* auto test
* auto code quality
* auto Static Application Security Testing (SAST)
* Auto Dependency Scanning
* Auto License Management
* Auto Container Scanning
* Auto Review Apps
* Auto Dynamic Application Security Testing (DAST)
* Auto Browser Performance Testing
* Auto Deploy
* Auto Monitoring

此外，用户还可以在代码文件自定义一些 buildpacks, to Dockerfiles, Helm charts等配置，也可以配置Environment variables实现自主的功能。

###gitlab集成kubernetes
在新版（11.0）gitlab中可以配置集成已存在的kubernetes集群和GCE kubernetes集群。

####gitlab docker runner config

#####1 配置优先使用local docker image
######在/etc/gitlab/config.toml中加入pull_policy = "if-not-present"
#####2 job运行中无法git clone 代码并报错
fatal: unable to access 'http://gitlab-ci-token:xxxxxxxxxxxxxxxxxxxx@gitlab/root/Buildlocal.git/': Failed to connect to gitlab port 80: Host is unreachable
######在配置文件/etc/gitlab/config.toml中加入extra_hosts = ["gitlab:10.64.3.13"]

````
concurrent = 1
check_interval = 0

[[runners]]
  name = "docker"
  url = "http://10.64.3.13/ci"
  token = "44d7d1c859e9f0c72698d14be2c005"
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "ruby:2.1"
    privileged = true
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
    pull_policy = "if-not-present"
    extra_hosts = ["gitlab:10.64.3.13"]
  [runners.cache]
````
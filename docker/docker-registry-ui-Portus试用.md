###portus部署
portus可以为docker registry提供两大功能，提供认证服务，提供易用，强大的UI。
portus可以与LDAP服务器集成。
下载portus的github项目
```
git clone https://github.com/SUSE/Portus.git
```
下载portus相关的docker 镜像
使用```docker-compose```启动portus
```
cd Portus
docker-compose up
```
遇到问题，国内go安装命令失败
```
go get -u github.com/openSUSE/portusctl
```
到portus的example下使用docker-compose启动portus服务
```
cd Portus/examples/compose/
docker-compose -f docker-compose.insecure.yml up
```
采用helm安装portus
首先下载portus的charts
```
git clone https://github.com/kubic-project/caasp-services.git
```
下载子chart```mariadb```。
```
helm dependency build contrib/helm-charts/portus
```
运行portus。
```
helm install --name portus contrib/helm-charts/portus
```
删除portus
```
helm delete portus
```
遇到问题
内置的```docker-registry```无法```push```和```pull```镜像。

问题解决
文件```docker-compose.yaml```内容如下：

```
version: "2"

services:
  portus:
    image: opensuse/portus:head
    environment:
      - PORTUS_MACHINE_FQDN_VALUE=${MACHINE_FQDN}

      # DB. The password for the database should definitely not be here. You are
      # probably better off with Docker Swarm secrets.
      - PORTUS_DB_HOST=db
      - PORTUS_DB_DATABASE=portus_production
      - PORTUS_DB_PASSWORD=${DATABASE_PASSWORD}
      - PORTUS_DB_POOL=5

      # Secrets. It can possibly be handled better with Swarm's secrets.
      - PORTUS_SECRET_KEY_BASE=${SECRET_KEY_BASE}
      - PORTUS_KEY_PATH=/certificates/portus.key
      - PORTUS_PASSWORD=${PORTUS_PASSWORD}

      # SSL
      - PORTUS_PUMA_TLS_KEY=/certificates/portus.key
      - PORTUS_PUMA_TLS_CERT=/certificates/portus.crt

      # NGinx is serving the assets instead of Puma. If you want to change this,
      # uncomment this line.
      #- RAILS_SERVE_STATIC_FILES='true'
    ports:
      - 3000:3000
    links:
      - db
    volumes:
      - ./secrets:/certificates:ro
      - static:/srv/Portus/public
    extra_hosts:
      - "xxxx.test-portus.com:192.192.189.1"

  background:
    image: opensuse/portus:head
    depends_on:
      - portus
      - db
    environment:
      # Theoretically not needed, but cconfig's been buggy on this...
      - CCONFIG_PREFIX=PORTUS
      - PORTUS_MACHINE_FQDN_VALUE=${MACHINE_FQDN}

      # DB. The password for the database should definitely not be here. You are
      # probably better off with Docker Swarm secrets.
      - PORTUS_DB_HOST=db
      - PORTUS_DB_DATABASE=portus_production
      - PORTUS_DB_PASSWORD=${DATABASE_PASSWORD}
      - PORTUS_DB_POOL=5

      # Secrets. It can possibly be handled better with Swarm's secrets.
      - PORTUS_SECRET_KEY_BASE=${SECRET_KEY_BASE}
      - PORTUS_KEY_PATH=/certificates/portus.key
      - PORTUS_PASSWORD=${PORTUS_PASSWORD}

      - PORTUS_BACKGROUND=true
    links:
      - db
    volumes:
      - ./secrets:/certificates:ro
    extra_hosts:
      - "xxxx.test-portus.com:192.192.189.1"

  db:
    image: library/mariadb:10.0.23
    command: mysqld --character-set-server=utf8 --collation-server=utf8_unicode_ci --init-connect='SET NAMES UTF8;' --innodb-flush-log-at-trx-commit=0
    environment:
      - MYSQL_DATABASE=portus_production

      # Again, the password shouldn't be handled like this.
      - MYSQL_ROOT_PASSWORD=${DATABASE_PASSWORD}
    volumes:
      - /var/lib/portus/mariadb:/var/lib/mysql
    extra_hosts:
      - "xxxx.test-portus.com:192.192.189.1"

  registry:
    image: library/registry:2.6
    command: ["/bin/sh", "/etc/docker/registry/init"]
    environment:
      # Authentication
      REGISTRY_AUTH_TOKEN_REALM: https://${MACHINE_FQDN}:3000/v2/token
      REGISTRY_AUTH_TOKEN_SERVICE: ${MACHINE_FQDN}:5000
      REGISTRY_AUTH_TOKEN_ISSUER: ${MACHINE_FQDN}
      #REGISTRY_AUTH_TOKEN_ISSUER: portus.test.lan
      REGISTRY_AUTH_TOKEN_ROOTCERTBUNDLE: /secrets/portus.crt

      # SSL
      REGISTRY_HTTP_TLS_CERTIFICATE: /secrets/portus.crt
      REGISTRY_HTTP_TLS_KEY: /secrets/portus.key

      # Portus endpoint
      REGISTRY_NOTIFICATIONS_ENDPOINTS: >
        - name: portus
          url: https://${MACHINE_FQDN}:3000/v2/webhooks/events
          #url: https://192.192.189.1:3000/v2/webhooks/events
          timeout: 2000ms
          threshold: 5
          backoff: 1s
    volumes:
      - /var/lib/portus/registry:/var/lib/registry
      - ./secrets:/secrets:ro
      - ./registry/config.yml:/etc/docker/registry/config.yml:ro
      - ./registry/init:/etc/docker/registry/init:ro
    ports:
      - 5000:5000
      - 5001:5001 # required to access debug service
    links:
      - portus:portus
    extra_hosts:
      - "xxxx.test-portus.com:192.192.189.1"

  nginx:
    image: library/nginx:alpine
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./secrets:/secrets:ro
      - static:/srv/Portus/public:ro
    ports:
      - 80:80
      - 443:443
    links:
      - registry:registry
      - portus:portus
    extra_hosts:
      - "xxxx.test-portus.com:192.192.189.1"
volumes:
  static:
    driver: local
```
配置注意点
* 自签名证书是在secret目录下执行： openssl req -newkey rsa:4096 -nodes -sha256 -keyout portus.key -x509 -days 3650 -out portus.crt。证书产生时common name配置为xxxx.test-portus.com。
* 如遇到portus不能跟它的registry正常认证，需要在portus web上配置registry时，第一行registry应填写xxxx.test-portus.com:5000,externalname那一行填写xxxx.test-portus.com。这里注意如果把registry的5000端口映射到主机的端口如果不为5000,也会产生错误。
* 不能login和push镜像到portus的registry，需要配置主机的docker的"insecure-registries" : ["xxxx.test-portus.com", "xxxx.test-portus.com:5000"]就可以了。
* 试了下外部访问xxxx.test-portus.com和xxxx.test-portus.com:5000的区别，xxxx.test-portus.com:5000是没有用户权限细分的，test01用户可以访问admin的namespace。xxxx.test-portus.com是严格区分的，test01用户不可以访问admin的namespace。
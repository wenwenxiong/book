```
docker run -d -p 80:8080 -it -e APP_CONTEXT=ui -e REG1=http://192.192.189.1:5000/v2/ atcol/docker-registry-ui
```
存在issue
无法列出v2版本registry的镜像。

```
docker run -d -p 5000:5000 --name registry-srv registry:2
docker run -it -p 80:8080 --name registry-web --link registry-srv -e REGISTRY_URL=http://registry-srv:5000/v2 -e REGISTRY_NAME=localhost:5000 hyper/docker-registry-web
```
功能有限，只能查看

```
docker run -v $(pwd)/registry-web.yml:/conf/config.yml:ro \
           -v $(pwd)/devdockerCA.crt:/conf/auth.key -v $(pwd)/db:/data \
           -it -p 8080:8080 --link dockerregistry_registry_1 --name registry-web hyper/docker-registry-web
```
功能有限，只能查看
```
sudo docker run \
  -d \
  -e ENV_DOCKER_REGISTRY_HOST=192.192.189.1 \
  -e ENV_DOCKER_REGISTRY_PORT=5000 \
  -p 8080:80 \
  konradkleine/docker-registry-frontend:v2
```
```
sudo docker run \
  -d \
  -e ENV_DOCKER_REGISTRY_HOST=192.192.189.105 \
  -e ENV_DOCKER_REGISTRY_PORT=5000 \
  -e ENV_DOCKER_REGISTRY_USE_SSL=1 \
  -p 8080:80 \
  konradkleine/docker-registry-frontend:v2
```
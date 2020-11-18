###Fission简介
Fission是一个fast，high performance的基于kubernetes的serverless框架。
Fission包含以下概念
function： 实现fission function interface的code
environment：运行function的环境（镜像）
trigger：触发器，Fission支持http触发，事件触发

fission自带的environment有

Environment|	Image
-----------|---------
Binary (for executables or scripts)|	fission/binary-env
Go	|fission/go-env
.NET	|fission/dotnet-env
.NET 2.0	|fission/dotnet20-env
NodeJS (Alpine)	|fission/node-env
NodeJS (Debian)|	fission/node-env-debian
Perl	|fission/perl-env
PHP 7	|fission/php-env
Python 3|	fission/python-env
Ruby	|fission/ruby-env
用户可以自定义Fission的environment。
###Fission安装
前提安装了kubectl，helm，kubernetes集群。
安装minimal的Fission
```
helm install --namespace fission https://github.com/fission/fission/releases/download/0.7.1/fission-core-0.7.1.tgz
```
安装Fission，包含NATS消息队列，日志收集influxDB等
```
helm install --namespace fission --set serviceType=NodePort https://github.com/fission/fission/releases/download/0.7.1/fission-all-0.7.1.tgz
```
安装Fission cli工具
```
curl -Lo fission https://github.com/fission/fission/releases/download/0.7.1/fission-cli-linux && chmod +x fission && sudo mv fission /usr/local/bin/
```
###Fission使用
创建一个hello world例子
```
 # Add the stock NodeJS env to your Fission deployment
  $ fission env create --name nodejs --image fission/node-env

  # A javascript one-liner that prints "hello world"
  $ curl https://raw.githubusercontent.com/fission/fission/master/examples/nodejs/hello.js > hello.js

  # Upload your function code to fission
  $ fission function create --name hello --env nodejs --code hello.js

  # Map GET /hello to your new function
  $ fission route create --method GET --url /hello --function hello

  # Run the function.  This takes about 100msec the first time.
  $ fission function test --name hello
  Hello, world!
```
参考网址： https://github.com/kubernetes/ingress-nginx/blob/master/docs/how-it-works.md

通过```kubernetes informers```监控到```Ingresses, Services, Endpoints, Secrets, and Configmaps```的变化，从而产生新的```nginx conf```配置。

配置是通过```Go template```模板文件，输入变量值而产生的。

为避免频繁修改和加载```nginx```配置，对于```Endpoints```只发生改变的情况，使用```openresty lua-nginx-module```组件来动态修改```nginx```配置并不需重新加载。

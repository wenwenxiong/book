k8s测试工具
* k8s-testsuite: 由两个Helm图表组合而成，适用于网络带宽测试与单个Kubernetes集群的负载测试。负载测试模拟了带有loadbots的简单网页服务器，这些服务器可在Vegeta基础上以Kubernetes微服务的形式运行。网络测试则是在内部连续对iperf3与netperf-2.7.0运行三次。这两项测试都会生成涵盖全部结果与指标的综合日志信息。
* sonobuoy: Sonobuoy允许用户以易于访问与非破坏性的方式运行一组测试，从而对当前Kubernetes集群状态进行评估。Sonobuoy可生成有关集群性能详细信息的信息性报告，并能够支持Kubernetes1.8及更高版本。SonobuoyScanner是一款基于浏览器的工具。在该工具的帮助下，用户只需点击数下即可完成对Kubernetes集群的测试。当然，其CLI版本能够应对规模更大的测试集群。
* PowerfulSeal: python编写,遵循混沌工程原理。因此，PowerfulSeal不仅可终止pod，还能够在集群中添加或删除虚拟机。PowerfulSeal具有交互模式，从而允许用户以手动方式中断特定的集群组件。另外，除了SSH以外，PowerfulSeal无需其它外部依赖。
* Test-infra：是一套用于Kubernetes测试与结果验证的工具集合。Test-infra包括多种仪表板，分别用于显示历史记录、汇总故障以及当前正在测试的内容。用户可通过创建自定义测试作业以增强Test-infra套件。此外，Test-infra可在使用Kubetest的不同供应商平台上，通过模拟完整的Kubernetes生命周期实现端到端Kubernetes测试。

###k8s-testsuite使用
github地址： https://github.com/mrahbar/k8s-testsuite

首先下载helm chart文件中所需的docker images。然后执行命令
```
> git clone https://github.com/mrahbar/k8s-testsuite.git
> cd k8s-testsuite
> helm install --namespace load-test ./load-test
> helm install --namespace network-test ./network-test
```
安装使用测试工具。
注意点：
1、load-test采用分别扩展pod实例数进行测试，两个pod（loadbots，webserver）分别由1扩展到100,而pod的默认cpurequest为100m，以此需要保证kubernetes集群环境拥有闲置的20core以上cpu资源运行测试，同样内存也应保证充足，建议32以上。
2、网络测试采用iperf和netperf两个测试工具进行测试，对于tcp，mtu配置从96以64步长直到1460,测试时间有点长。此外测试场景包括pod到另一个node上pod的网络性能，因此，kubernetes测试需要至少两个节点。
3、日志的查看和chart运行时变量的配置在github上给除了详细的说明。

###sonobuoy使用
github地址：https://github.com/heptio/sonobuoy

sonobuoy是采用go语言编写的，安装时使用go get命令（解决墙问题，可以使用Shadowsocks+cow组合方式，安装和使用可以搜索google）
```
$ go get -u -v github.com/heptio/sonobuoy
```
使用```sonobuoy --help```可以查看使用帮助，下面列出常用的命令
运行sonobuoy（自行提前下载好镜像，除了sonobuoy自身运行的镜像，它还调用了kubernetes endtoendtest的镜像，镜像比较多具体参考 https://github.com/kubernetes/kubernetes/blob/master/test/utils/image/manifest.go#L25:54）
```
sonobuoy run --image-pull-policy IfNotPresent
```
查看测试状态
```
sonobuoy status
```
查看测试日志
```
sonobuoy logs
```
打包获取结果输出
```
sonobuoy retrieve .
mkdir ./results; tar xzf *.tar.gz -C ./results
```
目前还没有图形化工具可以离线分析查看sonobuoy输出结果。
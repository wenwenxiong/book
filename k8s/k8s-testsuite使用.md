###k8s测试工具
* k8s-testsuite: 由两个Helm图表组合而成，适用于网络带宽测试与单个Kubernetes集群的负载测试。负载测试模拟了带有loadbots的简单网页服务器，这些服务器可在Vegeta基础上以Kubernetes微服务的形式运行。网络测试则是在内部连续对iperf3与netperf-2.7.0运行三次。这两项测试都会生成涵盖全部结果与指标的综合日志信息。

###k8s-testsuite使用
github地址： https://github.com/mrahbar/k8s-testsuite

准备文件安装包```k8s-testsuite.tar.gz```通过```ansible```自动化安装。
安装前提条件：云平台套件```rong```已经部署
安装步骤：
1）配置目标机器
配置文件为```k8s-testsuite/hosts.ini ```。

2）运行```ansible```命令执行安装
```
tar -zxvf k8s-testsuite.tar.gz
cd k8s-testsuite
ansible-playbook -i hosts.ini k8s-testsuite.yaml
```
安装使用测试工具。
注意点：
1、load-test采用分别扩展pod实例数进行测试，两个pod（loadbots，webserver）分别由1扩展到100,而pod的默认cpurequest为100m，以此需要保证kubernetes集群环境拥有闲置的20core以上cpu资源运行测试，同样内存也应保证充足，建议32以上。

2、网络测试采用iperf和netperf两个测试工具进行测试，对于tcp，mtu配置从96以64步长直到1460,测试时间有点长。此外测试场景包括pod到另一个node上pod的网络性能，因此，kubernetes测试需要至少两个节点。

3、测试结果通过安装运行helm命令后显示的帮助信息操作查看。

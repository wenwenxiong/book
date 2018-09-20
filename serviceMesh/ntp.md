###linux修改时间和时区
1、设置时区
查看时区
```
date
```
看时区是UTC或CST。

修改配置文件来修改时区
1）、修改/etc/sysconfig/clock
ZONE=Asia/Shanghai

2）、rm /etc/localtime
链接到上海时区文件
```
 ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
执行完上述过程后，重启机器，即可看到时区已经更改。

2、 修改系统时间
修改日期
```
date -s 04/16/18
```
修改时间
```
date 04160910.30
```
硬件时钟与系统时钟同步：
```
# hwclock --hctosys 或者 # clock --hctosys  hc代表硬件时间，sys代表系统时间，即用硬件时钟同步系统时钟
```
系统时钟和硬件时钟同步：
```
# hwclock --systohc或者# clock --systohc  即用系统时钟同步硬件时钟
```


### Server
redhat74上已经没有ntpd rpm包，可以只下载ntp的rpm包，配置文件也变更为ntp.conf
server side configuration:    

```
# yum install -y ntp
# vim /etc/ntp.conf
```
The configuration file is listed as following:   

```
driftfile /var/lib/ntp/drift
restrict default nomodify notrap nopeer noquery
restrict 127.0.0.1 
restrict ::1
restrict 192.168.0.0 mask 255.255.0.0 nomodify notrap
server 127.127.1.0  # local clock
fudge 127.127.1.0  stratum 10
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor
```
Disable the chronyd service thus the ntpd could acts properly:   

```
# systemctl disable chronyd
# systemctl enable ntpd
# systemctl start ntpd
# systemctl disable firewalld
```
### Client
Install via:    

```
# yum install -y ntp
```
Configuration file:    

```
driftfile /var/lib/ntp/drift
restrict default nomodify notrap nopeer noquery
restrict 127.0.0.1 
restrict ::1
server 192.168.122.200
# 配置允许上游时间服务器主动修改本机的时间
restrict 192.168.122.200 nomodify notrap noquery
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor
```
Also disable the chronyd service and enable the ntpd service. The client will
automatically sync with the server `192.168.122.200`.   

遇到错误
运行
```
ntpq -p
```
查询时间同步信息。
结果系统提示： no association ID`s returned
解决办法：手动修改该机器的时间与目标服务器的时间基本相同，然后重启本机的ntpd服务，再次查询便正确了。
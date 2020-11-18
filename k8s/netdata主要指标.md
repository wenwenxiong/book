
###CPUs
1、```utilization - total and per core CPU usage.```
CPU利用率-总```CPU```使用量和每个核心```CPU```使用量。
```
1. guest		访客
2. softirq	软中断
3. user		用户
4. system	系统
5. iowait	io请求导致等待
```
2、```interrupts - total and per core CPU interrupts.```
中断-总中断和每个核心CPU中断。
```
softirqs - total and per core SoftIRQs.
软中断-总软中断和每个核心软中断。
softnet - total and per core SoftIRQs related to network activity.
与网络活动相关的总软中断和每个核心软中断。
throttling - collects per core CPU throttling.
cpu温度过高降频节流-收集每个核心CPU节流。
cpufreq - collects the current CPU frequency
CPU频率-收集当前CPU频率。
cpuidle - collects the time spent per processor state.
cpu空闲-收集每个处理器状态花费的时间。
```
3、```IdleJitter - measures CPU latency.```
测量CPU延迟。
```
Entropy - random numbers pool, using in cryptography.
随机数池，用于密码学。
Interprocess Communication - IPC - such as semaphores and semaphores arrays.
工控机-如信号量和信号量数组。
```
###Memory
system   系统
ram - collects info about RAM usage.
主存 -收集有关主存使用的信息。
swap - collects info about swap memory usage.
交换——收集关于交换内存使用情况的信息。
available memory - collects the amount of RAM available for userspace processes.
可用内存——收集用户空间进程可用的RAM数量。
committed memory - collects the amount of RAM committed to userspace processes.
提交内存——收集提交给用户空间进程的RAM数量。
Page Faults - collects the system page faults (major and minor).
页面错误-收集系统页面错误(主要和次要)。
Kenerl   内核
writeback memory - collects the system dirty memory and writeback activity.
回写内存——收集系统等待写入磁盘的内存和回写活动。
slab - collects info about the Linux kernel memory usage.
内存片——收集关于Linux内核内存使用情况的信息。
huge pages - collects the amount of RAM used for huge pages.
大页——收集用于大页的RAM数量。
numa – (Non Uniform Memory Access Architecture)collects Numa info on systems that support it.
非统一内存访问体系结构——收集支持它的系统的Numa信息。
Ecc – ECC Memory Corectatble Errors
错误检查、纠错 ——ecc 内存矫正错误
###Disks
dm-0 、sda（第一个磁盘,sdb第二个磁盘，依次类推）  
block devices - per disk: I/O, operations, backlog, utilization, space, etc.
块设备——每个磁盘:I/O、操作、积压、利用率、空间等。
BCACHE - detailed performance of SSD caching devices.
SSD缓存设备的详细性能。
DiskSpace - monitors disk space usage.
磁盘空间—监视磁盘空间使用情况。
mdstat - software RAID.
软件RAID
hddtemp - disk temperatures.
磁盘温度
smartd - disk S.M.A.R.T. values.
磁盘自动状态检测值
device mapper - naming disks.
设备映射器——命名磁盘
Veritas Volume Manager - naming disks.
命名磁盘。
megacli - adapter, physical drives and battery stats.
适配器、物理驱动器和电池状态。
adaptec_raid - logical and physical devices health metrics.
逻辑和物理设备健康指标。
ioping - to measure disk read/write latency.
ioping—测量磁盘读写延迟。
###Networking
tcp – tcp connection aborts,tcp reordered packets by detection method,tcp out-of-order queue
TCP连接中止，TCP按检测方法重新排序数据包，TCP无序队列
Network Stack - everything about the networking stack (both IPv4 and IPv6 for all protocols: TCP, UDP, SCTP, UDPLite, ICMP, Multicast, Broadcast, etc), and all network interfaces (per interface: bandwidth, packets, errors, drops).
网络堆栈-关于网络堆栈的所有信息（所有协议的IPv4和IPv6：TCP、UDP、SCTP、Udplite、ICMP、多播、广播等）以及所有网络接口（每个接口：带宽、数据包、错误、丢弃）。
Netfilter - everything about the netfilter connection tracker.
有关Netfilter连接跟踪器的所有信息。
SynProxy - collects performance data about the linux SYNPROXY (DDoS).
收集有关Linux synproxy（ddos）的性能数据。
NFacct - collects accounting data from iptables.
从iptables收集统计数据。
Network QoS - the only tool that visualizes network tc classes in real-time
唯一一个实时可视化网络TC类的工具
FPing - to measure latency and packet loss between any number of hosts.
测量任何数量的主机之间的延迟和数据包丢失。
ISC dhcpd - pools utilization, leases, etc.
池利用、租赁等。
AP - collects Linux access point performance data (hostapd).
收集Linux接入点性能数据
SNMP - SNMP devices can be monitored too (although you will need to configure these).
也可以监控SNMP设备
port_check - checks TCP ports for availability and response time.
检查TCP端口的可用性和响应时间
Virtual Private Networks
OpenVPN - collects status per tunnel.
收集每个隧道的状态。
LibreSwan - collects metrics per IPSEC tunnel.
收集每个ipsec隧道的度量。
Tor - collects Tor traffic statistics.
收集Tor流量统计数据。
###Processes
System Processes - running, blocked, forks, active.
系统进程-运行、阻塞、分叉、激活。
Applications - by grouping the process tree and reporting CPU, memory, disk reads, disk writes, swap, threads, pipes, sockets - per process group.
应用程序-通过分组进程树并报告每个进程组的CPU、内存、磁盘读取、磁盘写入、交换、线程、管道、套接字。
systemd - monitors systemd services using CGROUPS.
使用cgroups监控systemd服务。
###Users
Users and User Groups resource usage - by summarizing the process tree per user and group, reporting: CPU, memory, disk reads, disk writes, swap, threads, pipes, sockets
用户和用户组资源使用情况-通过汇总每个用户和组的进程树，报告：CPU、内存、磁盘读取、磁盘写入、交换、线程、管道、套接字
logind - collects sessions, users and seats connected.
收集连接的会话、用户和座位。
###Containers and VMs
cpu、mem
Containers - collects resource usage for all kinds of containers, using CGROUPS (systemd-nspawn, lxc, lxd, docker, kubernetes, etc).
容器-使用cgroups收集各种容器的资源使用情况。
libvirt VMs - collects resource usage for all kinds of VMs, using CGROUPS.
使用cgroups收集各种虚拟机的资源使用情况。
dockerd - collects docker health metrics.
收集Docker健康指标。

###Web Servers
Apache and lighttpd - mod-status (v2.2, v2.4) and cache log statistics, for multiple servers.
多台服务器的mod状态（v2.2、v2.4）和缓存日志统计信息。
IPFS - bandwidth, peers.
带宽、对等端。
LiteSpeed - reads the litespeed rtreport files to collect metrics.
读取LiteSpeed RTReport文件以收集度量。
Nginx - stub-status, for multiple servers.
多个服务器的存根状态。
Nginx+ - connects to multiple nginx_plus servers (local or remote) to collect real-time performance metrics.
连接到多个nginx+服务器（本地或远程），以收集实时性能指标。
PHP-FPM - multiple instances, each reporting connections, requests, performance, etc.
多个实例、每个报告连接、请求、性能等。
Tomcat - accesses, threads, free memory, volume, etc.
访问、线程、可用内存、卷等。
web server access.log files - extracting in real-time, web server and proxy performance metrics and applying several health checks, etc.
实时提取、Web服务器和代理性能指标，并应用多个运行状况检查等。
HTTP check - checks one or more web servers for HTTP status code and returned content.
HTTP检查-检查一个或多个Web服务器的HTTP状态代码和返回的内容。

###Proxies, Balancers, Accelerators
HAproxy - bandwidth, sessions, backends, etc.
带宽、会话、后端等。 
Squid - multiple servers, each showing: clients bandwidth and requests, servers bandwidth and requests.
多个服务器，每个服务器显示：客户端带宽和请求、服务器带宽和请求。
Traefik - connects to multiple traefik instances (local or remote) to collect API metrics (response status code, response time, average response time and server uptime).
连接到多个traefik实例（本地或远程）以收集API度量（响应状态代码、响应时间、平均响应时间和服务器正常运行时间）。
Varnish - threads, sessions, hits, objects, backends, etc.
线程、会话、点击、对象、后端等。
IPVS - collects metrics from the Linux IPVS load balancer.
从Linux IPV负载均衡器收集度量。

###Database Servers
CouchDB - reads/writes, request methods, status codes, tasks, replication, per-db, etc.
读/写、请求方法、状态代码、任务、复制、每个数据库等。
MemCached - multiple servers, each showing: bandwidth, connections, items, etc.
多个服务器，每个服务器显示：带宽、连接、项目等。
MongoDB - operations, clients, transactions, cursors, connections, asserts, locks, etc.
操作、客户机、事务、光标、连接、断言、锁等。
MySQL and mariadb - multiple servers, each showing: bandwidth, queries/s, handlers, locks, issues, tmp operations, connections, binlog metrics, threads, innodb metrics, and more.
多个服务器，每个服务器都显示：带宽、查询/秒、处理程序、锁、问题、tmp操作、连接、binlog度量、线程、innodb度量等。
PostgreSQL - multiple servers, each showing: per database statistics (connections, tuples read - written - returned, transactions, locks), backend processes, indexes, tables, write ahead, background writer and more.
多个服务器，每个服务器显示：每个数据库的统计信息（连接、读写-返回的元组、事务、锁）、后端进程、索引、表、提前写入、后台编写器等。
Proxy SQL - collects Proxy SQL backend and frontend performance metrics.
收集代理SQL后端和前端性能指标。
Redis - multiple servers, each showing: operations, hit rate, memory, keys, clients, slaves.
多个服务器，每个服务器都显示：操作、命中率、内存、密钥、客户机、从机。
RethinkDB - connects to multiple rethinkdb servers (local or remote) to collect real-time metrics.
连接到多个Rethinkdb服务器（本地或远程）以收集实时度量。
###Message Brokers
beanstalkd - global and per tube monitoring.
全局和每个管道监控。
RabbitMQ - performance and health metrics.
性能和健康指标。

###Search and Indexing
ElasticSearch - search and index performance, latency, timings, cluster statistics, threads statistics, etc.
搜索和索引性能、延迟、计时、集群统计、线程统计等。

###DNS Servers
bind_rndc - parses named.stats dump file to collect real-time performance metrics. All versions of bind after 9.6 are supported.
绑定命名的.stats转储文件以收集实时性能指标。支持9.6之后的所有bind版本。
dnsdist - performance and health metrics.
性能和健康指标。
ISC Bind (named) - multiple servers, each showing: clients, requests, queries, updates, failures and several per view metrics. All versions of bind after 9.9.10 are supported.
多个服务器，每个服务器显示：客户机、请求、查询、更新、失败和多个每视图度量。支持9.9.10之后的所有bind版本。
NSD - queries, zones, protocols, query types, transfers, etc.
查询、区域、协议、查询类型、传输等。
PowerDNS - queries, answers, cache, latency, etc.
查询、答案、缓存、延迟等。
unbound - performance and resource usage metrics.
性能和资源使用指标。
dns_query_time - DNS query time statistics.
DNS查询时间-DNS查询时间统计。
###Time Servers
chrony - uses the chronyc command to collect chrony statistics (Frequency, Last offset, RMS offset, Residual freq, Root delay, Root dispersion, Skew, System time).
使用chronic命令收集chrony统计信息
ntpd - connects to multiple ntpd servers (local or remote) to provide statistics of system variables and optional also peer variables.
连接到多个ntpd服务器（本地或远程），以提供系统变量和可选的对等变量的统计信息。
###Mail Servers
Dovecot - POP3/IMAP servers.
POP3/IMAP服务器。
Exim - message queue (emails queued).
消息队列
Postfix - message queue (entries, size).
消息队列
###Hardware Sensors
IPMI - enterprise hardware sensors and events.
企业硬件传感器和事件。
lm-sensors - temperature, voltage, fans, power, humidity, etc.
温度、电压、风扇、电源、湿度等。
Nvidia - collects information for Nvidia GPUs.
收集Nvidia GPU的信息。
RPi - Raspberry Pi temperature sensors.
树莓PI温度传感器。
w1sensor - collects data from connected 1-Wire sensors.
从连接的单线传感器收集数据。
###UPSes
apcupsd - load, charge, battery voltage, temperature, utility metrics, output metrics
负载、充电、电池电压、温度、效用指标、输出指标
NUT - load, charge, battery voltage, temperature, utility metrics, output metrics
负载、充电、电池电压、温度、效用指标、输出指标
Linux Power Supply - collects metrics reported by power supply drivers on Linux.
收集Linux上电源驱动程序报告的指标。
###Social Sharing Servers
RetroShare - connects to multiple retroshare servers (local or remote) to collect real-time performance metrics.
连接到多个追溯共享服务器（本地或远程）以收集实时性能指标。
###Security
Fail2Ban - monitors the fail2ban log file to check all bans for all active jails.
监控fail2ban日志文件，检查所有活动监狱的所有禁令。
###Authentication, Authorization, Accounting (AAA, RADIUS, LDAP) Servers
FreeRadius - uses the radclient command to provide freeradius statistics (authentication, accounting, proxy-authentication, proxy-accounting).
使用radclient命令提供freeradius统计信息（身份验证、记帐、代理身份验证、代理记帐）。
###Telephony Servers
opensips - connects to an opensips server (localhost only) to collect real-time performance metrics.
连接到OpenSips服务器（仅限本地主机）以收集实时性能指标。
###Household Appliances
SMA webbox - connects to multiple remote SMA webboxes to collect real-time performance metrics of the photovoltaic (solar) power generation.
连接到多个远程SMA Webbox，以收集光伏（太阳能）发电的实时性能指标。
Fronius - connects to multiple remote Fronius Symo servers to collect real-time performance metrics of the photovoltaic (solar) power generation.
连接到多个远程fronius symo服务器，以收集光伏（太阳能）发电的实时性能指标。
StiebelEltron - collects the temperatures and other metrics from your Stiebel Eltron heating system using their Internet Service Gateway (ISG web).
使用其互联网服务网关（ISG Web）从您的Stiebel Eltron供暖系统收集温度和其他指标。
###Game Servers
SpigotMC - monitors Spigot Minecraft server ticks per second and number of online players using the Minecraft remote console.
使用Minecraft远程控制台监控Spigot Minecraft服务器每秒滴答数和在线玩家数。
###Distributed Computing
BOINC - monitors task states for local and remote BOINC client software using the remote GUI RPC interface. Also provides alarms for a handful of error conditions.
使用远程GUI RPC接口监控本地和远程BOINC客户端软件的任务状态。还为一些错误情况提供警报。
###Media Streaming Servers
IceCast - collects the number of listeners for active sources.
收集活动源的侦听器数量。
###Monitoring Systems
Monit - collects metrics about monit targets (filesystems, applications, networks).
收集有关monit目标
###Provisioning Systems
Puppet - connects to multiple Puppet Server and Puppet DB instances (local or remote) to collect real-time status metrics.
连接到多个Puppet服务器和Puppet DB实例（本地或远程），以收集实时状态度量。



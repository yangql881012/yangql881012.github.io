---
title: Elasticsearch 集群安装
date: 2017-06-29 08:54:59
tags: Elasticsearch
toc: true
categories: 大数据技术
---
## 1.下载解压 ##
```
tar xvf elasticsearch-5.4.3.tar.gz
```

## 2.配置 /config/elasticsearch.yml ##
```
cluster.name: es-cluster
node.name: es01
path.data: /home/yangql/es/data
path.logs: /home/yangql/es/logs
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
network.host: hadoop01
http.port: 9200
discovery.zen.ping.unicast.hosts: ["hadoop01", "hadoop02","hadoop03"]
```
查看集群  
http://192.168.1.231:9200/_cluster/stats?pretty
<!-- more -->
## 3.错误解决 ##
### 3.1 unable to install syscall filter ###
```
[2017-06-29T08:27:42,584][WARN ][o.e.b.JNANatives         ] unable to install syscall filter:
java.lang.UnsupportedOperationException: seccomp unavailable: requires kernel 3.5+ with CONFIG_SECCOMP and CONFIG_SECCOMP_FILTER compiled in
	at org.elasticsearch.bootstrap.SystemCallFilter.linuxImpl(SystemCallFilter.java:350) ~[elasticsearch-5.4.3.jar:5.4.3]
	at org.elasticsearch.bootstrap.SystemCallFilter.init(SystemCallFilter.java:638) ~[elasticsearch-5.4.3.jar:5.4.3]
	at org.elasticsearch.bootstrap.JNANatives.tryInstallSystemCallFilter(JNANatives.java:215) [elasticsearch-5.4.3.jar:5.4.3]
	at org.elasticsearch.bootstrap.Natives.tryInstallSystemCallFilter(Natives.java:99) [elasticsearch-5.4.3.jar:5.4.3]
	at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:111) [elasticsearch-5.4.3.jar:5.4.3]
	at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:194) [elasticsearch-5.4.3.jar:5.4.3]
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:350) [elasticsearch-5.4.3.jar:5.4.3]
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:123) [elasticsearch-5.4.3.jar:5.4.3]
	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:114) [elasticsearch-5.4.3.jar:5.4.3]
	at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:67) [elasticsearch-5.4.3.jar:5.4.3]
	at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:122) [elasticsearch-5.4.3.jar:5.4.3]
	at org.elasticsearch.cli.Command.main(Command.java:88) [elasticsearch-5.4.3.jar:5.4.3]
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:91) [elasticsearch-5.4.3.jar:5.4.3]
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:84) [elasticsearch-5.4.3.jar:5.4.3]
```
原因：报了一大串错误，大家不必惊慌，其实只是一个警告，主要是因为你Linux版本过低造成的。
解决方案：  
1、重新安装新版本的Linux系统
2、警告不影响使用，可以忽略
### 3.2 启动 elasticsearch 如出现异常  can not run elasticsearch as root ###
解决方法：创建ES 账户，修改文件夹 文件 所属用户 组
### 3.3 启动异常：ERROR: bootstrap checks failed ###
```
system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk
```

问题原因：因为Centos6不支持SecComp，而ES5.2.1默认bootstrap.system_call_filter为true进行检测，所以导致检测失败，失败后直接导致ES不能启动。详
解决方法：在elasticsearch.yml中配置bootstrap.system_call_filter为false，注意要在Memory下面:
```
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
```
### 3.4 启动后，如果只有本地可以访问，尝试修改配置文件 elasticsearch.yml ###
中network.host(注意配置文件格式不是以 # 开头的要空一格， ： 后要空一格)
为 network.host: 0.0.0.0
默认端口是 9200
注意：关闭防火墙 或者开放9200端口
### 3.5 ERROR: bootstrap checks failed ###
```
max file descriptors [4096] for elasticsearch process likely too low, increase to at least [65536]
max number of threads [1024] for user [lishang] likely too low, increase to at least [2048]
```
解决方法：切换到root用户，编辑limits.conf 添加类似如下内容
vi /etc/security/limits.conf
添加如下内容:
```
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096
```
### 3.6max number of threads [1024] for user [lish] likely too low, increase to at least [2048] ###
解决：切换到root用户，进入limits.d目录下修改配置文件。
vi /etc/security/limits.d/90-nproc.conf
修改如下内容：
```
* soft nproc 1024
```
#修改为
```
* soft nproc 2048
```
### 3.7 max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144] ###

解决：切换到root用户修改配置sysctl.conf
vi /etc/sysctl.conf

添加下面配置：
```
vm.max_map_count=655360
```
并执行命令：
```
sysctl -p
```

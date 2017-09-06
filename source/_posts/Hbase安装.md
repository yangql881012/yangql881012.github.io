---
title: Hbase安装
date: 2017-04-19 12:54:59
tags: Hbase
toc: true
categories: 大数据技术
---
## 1. 解压缩hbase的软件包 ##
```
[yangql@hadoop01 app]$ tar -xvf hbase-1.2.5-bin.tar.gz
```
## 2. 进入hbase的配置目录，在hbase-env.sh文件里面加入java环境变量.##
```
export JAVA_HOME=/opt/jdk1.8.0_91
```
## 3. 关闭HBase自带的Zookeeper,使用Zookeeper集群 ##
```
export HBASE_MANAGES_ZK=false
```
<!-- more -->
## 4. 编辑hbase-site.xml ，添加配置文件##
```
<configuration>
<property>
 <name>hbase.rootdir</name>
 <value>hdfs://hadoop01:8020/hbase</value>
</property>
<property>
 <name>hbase.cluster.distributed</name>
 <value>true</value>
</property>
<property>
 <name>hbase.zookeeper.quorum</name>
 <value>hadoop01,hadoop02,hadoop03</value>
</property>
<property>
 <name>hbase.zookeeper.property.dataDir</name>
 <value>/home/yangql/app/hbase-1.2.5/tmp/zk/data</value>
</property>
</configuration>
```
## 5. 编辑配置目录下面的文件regionservers ##
```
hadoop02
hadoop03
```
## 6.配置环境变量 ##
```
# config for hbase
export HBASE_HOME=/home/yangql/app/hbase-1.2.5
export PATH=$PATH:$PATH/bin
```
## 7.把Hbase复制到其他机器 ##
```
[yangql@hadoop01 app]$ scp -r hbase-1.2.5/ yangql@hadoop02:/home/yangql/app/
[yangql@hadoop01 app]$ scp -r hbase-1.2.5/ yangql@hadoop03:/home/yangql/app/
scp  .bash_profile yangql@hadoop02:/home/yangql
scp  .bash_profile yangql@hadoop03:/home/yangql
source .bash_profile
```
## 8.启动 hbase##
- hadoop
```
sh /home/yangql/app/hadoop-2.7.2/sbin/start-all.sh
7843 Master
7892 Jps
7756 ResourceManager
7406 NameNode
7599 SecondaryNameNode
```
- 启动zookeeper
```
sh /home/yangql/app/zookeeper-3.4.6/bin/zkServer.sh start
7843 Master
9541 Jps
7756 ResourceManager
7406 NameNode
8126 QuorumPeerMain
7599 SecondaryNameNode
```
- 启动HBase

```
[yangql@hadoop01 bin]$ sh  start-hbase.sh
starting master, logging to /home/yangql/app/hbase-1.2.5/logs/hbase-yangql-master-hadoop01.out
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option PermSize=128m; support was removed in 8.0
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0
hadoop02: starting regionserver, logging to /home/yangql/app/hbase-1.2.5/bin/../logs/hbase-yangql-regionserver-hadoop02.out
hadoop03: starting regionserver, logging to /home/yangql/app/hbase-1.2.5/bin/../logs/hbase-yangql-regionserver-hadoop03.out
hadoop03: Java HotSpot(TM) 64-Bit Server VM warning: ignoring option PermSize=128m; support was removed in 8.0
hadoop03: Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0
hadoop02: Java HotSpot(TM) 64-Bit Server VM warning: ignoring option PermSize=128m; support was removed in 8.0
hadoop02: Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0
7843 Master
9541 Jps
8806 HMaster
7756 ResourceManager
7406 NameNode
8126 QuorumPeerMain
7599 SecondaryNameNode
[yangql@hadoop01 bin]$
```
## 9. 验证 ##
```
[yangql@hadoop01 bin]$ sh hbase shell
2017-04-19 00:47:18,635 WARN  [main] util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.2.5, rd7b05f79dee10e0ada614765bb354b93d615a157, Wed Mar  1 00:34:48 CST 2017

hbase(main):001:0> status
1 active master, 0 backup masters, 2 servers, 0 dead, 1.0000 average load

hbase(main):002:0> version
1.2.5, rd7b05f79dee10e0ada614765bb354b93d615a157, Wed Mar  1 00:34:48 CST 2017

hbase(main):003:0> list
TABLE                                                                                                                                 
0 row(s) in 0.0990 seconds

=> []
hbase(main):004:0>
```
## 10. web查看集群状态 ##
```
http://192.168.1.231:16010/master-status
http://192.168.1.232:16030/rs-status
```

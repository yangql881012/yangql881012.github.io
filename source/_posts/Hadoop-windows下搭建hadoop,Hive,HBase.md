---
title: Hadoop-windows下搭建hadoop,Hive,HBase
date: 2016-11-13 09:09:59
tags: Hadoop
toc: true
categories: 大数据技术
---
## 1.搭建Hadoop ##
- 安装JDK1.8并设置环境变量 JAVA_HOME。
- 下载hadoop2.7.2 ,解压到D盘，路径为D:\winbigdata\hadoop2.7.2(注：如何不是根目录，不要带空格)
- 添加环境变量HADOOP_HOME=D:\winbigdata\hadoop2.7.2\ ,将D:\winbigdata\hadoop2.7.2\bin和D:\winbigdata\hadoop2.7.2\sbin添加到path中。
- 下载hadooponwindows，下载地址`https://github.com/sardetushar/hadooponwindows `
- 删除hadoop下的etc和bin。将hadooponwindows里的etc和bin拷贝到D:\winbigdata\hadoop2.7.2\下。
- 修改etc/hadoop/core-site.xml
<!-- more -->
```
<configuration>
   <property>
       <name>fs.defaultFS</name>
       <value>hdfs://localhost:9000</value>
   </property>
</configuration>
```
- 修改etc/hadoop/mapred-site.xml
```
<configuration>
   <property>
       <name>mapreduce.framework.name</name>
       <value>yarn</value>
   </property>
</configuration>
```
- 修改etc/hadoop/hdfs-site.xml,data/namenode,data/datanode自己创建的目录
```
<configuration>
   <property>
       <name>dfs.replication</name>
       <value>1</value>
   </property>
   <property>
       <name>dfs.namenode.name.dir</name>
       <value>file:/winbigdata/hadoop-2.7.2/data/namenode</value>
   </property>
   <property>
       <name>dfs.datanode.data.dir</name>
     <value>file:/winbigdata/hadoop-2.7.2/data/datanode</value>
   </property>
</configuration>
```
- 修改etc\hadoop\yarn-site.xml
```
<configuration>
    <property>
       <name>yarn.nodemanager.aux-services</name>
       <value>mapreduce_shuffle</value>
    </property>
    <property>
       <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
       <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
</configuration>
```
- 修改etc/hadoop/hadoop-env.cmd,修改JAVA_HOME(这里使用`PROGRA~1`,不要用`program files`,有空格会报错)
```
@rem set JAVA_HOME=%JAVA_HOME%
set JAVA_HOME=C:\PROGRA~1\Java\jdk1.8.0_05
```
- 格式化namenode
```
hdfs namenode -format
```
-  启动Hadoop,sbin目录下执行
```
start-all
```
启动了4个窗口，namenode,datanode,yarn resourcemanager,yarn nodemanager
```
- 停止Hadoop,sbin下执行
```
stop-all
```
- web页面
```
Resourcemanager address:http://localhost:8088/
Namenode address:http://localhost:50070/
```
## 2.安装MYSQL ##
- 下载mysql-5.7.17-winx64.zip，解压到D:\winbigdata，将文件夹重命名为mysql
- 配置环境变量MYSQL_HOME=D:\winbigdata\mysql并将 %MYSQL_HOME%\bin 加到Path中
- 创建my.ini文件，内容如下：
```
[mysqld]
basedir = D:\\winbigdata\\mysql
datadir = D:\\winbigdata\\mysql\\data
port = 3306
server_id = MYSQL
```
- 以管理员身份运行cmd，进入mysql的bin目录。
```
mysqld  --initialize
```
初始化成功后，会在datadir目录下生成一些文件，其中，xxx.err文件里说明了root账户的临时密码。那行大概长这样：
2017-06-15T01:20:27.235211Z 1 [Note] A temporary password is generated for root@localhost: (r=ol.mQK7kD
即密码是：(r=ol.mQK7kD
- 进入到bin目录注册mysql服务(一定要进入到bin目录，否则启动时会报错，无法找到指定文件)
```
mysqld -install MySQL
```
- 启动mysql服务
```
net start MySQL
```
- 修改密码
```
mysql -uroot -p
SET PASSWORD = PASSWORD('mysql123');
```
## 2.安装HIVE ##
- 下载Hive，我安装的版本是`apache-hive-2.1.1-bin`。安装路径是`D:\winbigdata\hive-2.1.1`
- 设置环境变量 `HIVE_HOME=D:\winbigdata\hive-2.1.1`，并将`D:\winbigdata\hive-2.1.1\bin`加入到Path中
- 修改hive-site.xm（复制一个hive-default.xml.template，重命名为hive-site.xml）
```
   <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true</value>
        <description>JDBC connect string for a JDBC metastore</description>
    </property>

    <property>
      <name>javax.jdo.option.ConnectionDriverName</name>
      <value>com.mysql.jdbc.Driver</value>
      <description>Driver class name for a JDBC metastore</description>
    </property>

    <property>
      <name>javax.jdo.option.ConnectionUserName</name>
      <value>root</value>
      <description>username to use against metastore database</description>
    </property>

    <property>
      <name>javax.jdo.option.ConnectionPassword</name>
      <value>mysql123</value>
      <description>password to use against metastore database</description>
    </property>
```
修改类似于 “${system:” 的配置了。在hive-site.xml中有很多地方引用了这种形式的变量，但是在实际运行时，在windows环境下这个变量存在问题。因此统一修改成
 相对路径 `/winbigdata/hive-2.1.1/hive_home`.使相关的文件尽量保存在同一个目录下。
```
 <property>
    <name>hive.exec.local.scratchdir</name>    
    <value>/winbigdata/hive-2.1.1/hive_home/scratch_dir</value>
    <description>Local scratch space for Hive jobs</description>
  </property>

  <property>
    <name>hive.downloaded.resources.dir</name>    
    <value>/winbigdata/hive-2.1.1/hive_home/resources_dir/${hive.session.id}_resources</value>    
    <description>Temporary local directory for added resources in the remote file system.</description>
  </property>

   <property>
    <name>hive.querylog.location</name>
    <value>/winbigdata/hive-2.1.1/hive_home/querylog_dir</value>
    <description>Location of Hive run time structured log file</description>
  </property>

   <property>
    <name>hive.server2.logging.operation.log.location</name>
    <value>/winbigdata/hive-2.1.1/hive_home/operation_dir</value>
    <description>Top level directory where operation logs are stored if logging functionality is enabled</description>
  </property>
```
- 修改hive-log4j2.properties  
将hive-log4j2.properties.template这个文件复制，重命名为hive-log4j2.properties.

```
status = INFO
name = HiveLog4j2
packages = org.apache.hadoop.hive.ql.log
# list of properties
property.hive.log.level = INFO
property.hive.root.logger = DRFA
property.hive.log.dir = hive_log
property.hive.log.file = hive.log
property.hive.perflogger.log.level = INFO

# list of all appenders
appenders = console, DRFA

# console appender
appender.console.type = Console
appender.console.name = console
appender.console.target = SYSTEM_ERR
appender.console.layout.type = PatternLayout
appender.console.layout.pattern = %d{ISO8601} %5p [%t] %c{2}: %m%n

# daily rolling file appender
appender.DRFA.type = RollingRandomAccessFile
appender.DRFA.name = DRFA
appender.DRFA.fileName = ${hive.log.dir}/${hive.log.file}
# Use %pid in the filePattern to append <process-id>@<host-name> to the filename if you want separate log files for different CLI session
appender.DRFA.filePattern = ${hive.log.dir}/${hive.log.file}.%d{yyyy-MM-dd}
appender.DRFA.layout.type = PatternLayout
appender.DRFA.layout.pattern = %d{ISO8601} %5p [%t] %c{2}: %m%n
appender.DRFA.policies.type = Policies
appender.DRFA.policies.time.type = TimeBasedTriggeringPolicy
appender.DRFA.policies.time.interval = 1
appender.DRFA.policies.time.modulate = true
appender.DRFA.strategy.type = DefaultRolloverStrategy
appender.DRFA.strategy.max = 30

# list of all loggers
loggers = NIOServerCnxn, ClientCnxnSocketNIO, DataNucleus, Datastore, JPOX, PerfLogger

logger.NIOServerCnxn.name = org.apache.zookeeper.server.NIOServerCnxn
logger.NIOServerCnxn.level = WARN

logger.ClientCnxnSocketNIO.name = org.apache.zookeeper.ClientCnxnSocketNIO
logger.ClientCnxnSocketNIO.level = WARN

logger.DataNucleus.name = DataNucleus
logger.DataNucleus.level = ERROR

logger.Datastore.name = Datastore
logger.Datastore.level = ERROR

logger.JPOX.name = JPOX
logger.JPOX.level = ERROR

logger.PerfLogger.name = org.apache.hadoop.hive.ql.log.PerfLogger
logger.PerfLogger.level = ${hive.perflogger.log.level}

# root logger
rootLogger.level = ${hive.log.level}
rootLogger.appenderRefs = root
rootLogger.appenderRef.root.ref = ${hive.root.logger}
```
- 将元数据导入到mysql
切换到目录D:\winbigdata\hive-2.1.1\scripts\metastore\upgrade\mysql
```
mysql -uroot -p
mysql>SOURCE D:\winbigdata\hive-2.1.1\scripts\metastore\upgrade\mysql\hive-schema-2.1.0.mysql.sql
```
- 启动Hadoop
```
start-all
```
- 启动hive-启动metastore
```
hive --service metastore -hiveconf hive.root.logger=DEBUG
```
- 启动hiveserver
```
hive --service hiveserver2
```
- 启动客户端
```
hive --service cli
```
- 验证
```
create table test_table(id INT);
show tables;
```
## 3.搭建HBase ##
- 下载HBase1.2.5,解压到目录D:\winbigdata\hbase-1.2.5
- 修改配置hbase-site.xml
```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
/**
 *
 * Licensed to the Apache Software Foundation (ASF) under one
 * or more contributor license agreements.  See the NOTICE file
 * distributed with this work for additional information
 * regarding copyright ownership.  The ASF licenses this file
 * to you under the Apache License, Version 2.0 (the
 * "License"); you may not use this file except in compliance
 * with the License.  You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
-->
<configuration>
    <property>
       <name>hbase.master</name>
       <value>localhost:6000</value>
   </property>
   <property>
           <name>hbase.master.maxclockskew</name>
           <value>180000</value>
   </property>
   <property>
           <name>hbase.rootdir</name>
           <value>hdfs://localhost:9000/hbase</value>
   </property>
   <property>
           <name>hbase.cluster.distributed</name>
           <value>false</value>
   </property>
   <property>
           <name>hbase.zookeeper.quorum</name>
           <value>localhost</value>
   </property>
   <property>
           <name>hbase.zookeeper.property.dataDir</name>
           <value>/hbase</value>
   </property>
   <property>
           <name>dfs.replication</name>
           <value>1</value>
   </property>
</configuration>
```
- 修改配置hbase-env.cmd
set JAVA_HOME=C:\PROGRA~1\Java\jdk1.8.0_05
- 停止hadoop
```
stop-all.cmd
```
- 格式化Hadoop命名节点
```
hdfs namenode -format
```
- 启动hadoop
```
start-all.cmd
```
- 启动hbase
```
start-hbase.cmd
```
- 启动 HBase的rest服务
```
hbase rest start -p 6000
```
`http://localhost:8085/`验证是否启动成功
- 启动HBase Shell
```
hbase shell
```
- 测试验证
```

```

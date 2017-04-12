---
title: hadoop-RedHat5.4-64bit编译安装hadoop2.5.1.md
date: 2017-01-09 00:54:59
tags: Hadoop,源码编译
toc: true
categories: Hadoop
---
## 1 环境 ##
- 系统：redhat 5.4
- 软件: jdk 1.7,hadoop- 2.5.1- src.tar.gz,Maven 3.1.1,protobuf2.5.0
- IP:192.168.1.231
- 主机名：hadoop01
- <font color="red">需要连接互联网(重要)</font>
<!-- more -->
## 2 安装包 ##
```
[root@hadoop01 yangql]# yum install gcc
[root@hadoop01 yangql]# yum intall gcc-c++  
[root@hadoop01 yangql]# yum install make
[root@hadoop01 yangql]# yum install cmake
[root@hadoop01 yangql]# yum install openssl-devel
[root@hadoop01 yangql]# yum install ncurses-devel
```
## 3 安装protoc ##
hadoop2.5.1编译需要protoc2.5.0的支持，所以还要安装下载protoc2.5.0
- 官方网址：https://code.google.com/p/protobuf/downloads/list
- 百度云盘：http://pan.baidu.com/s/1slEnwT3
- 配置命令：
```
[root@hadoop01 yangql]# tar -xvf protobuf-2.5.0.tar.gz
[root@hadoop01 yangql]# cd protobuf-2.5.0
[root@hadoop01 yangql]# ./configure --prefix=/usr/local/protobuf
[root@hadoop01 yangql]# make
[root@hadoop01 yangql]# make install
```
- <font color="red" size=5>/usr/local/protobuf</font>:自定义安装目录，可根据自己情况设定
- 修改.bash_profile,protobuf加入到环境变量中
```
export PATH=/usr/local/protobuf/bin:$PATH
```
- 执行protoc --version命令，显示如下信息时，安装成功
```
[root@hadoop01 etc]# protoc --version
libprotoc 2.5.0
```
## 4 maven 安装 ##
准备maven作为编译hadoop的工具
- 下载mavn
```
[root@hadoop01 yangql]# wget http://mirror.bit.edu.cn/apache/maven/maven-3/3.1.1/binaries/apache-maven-3.1.1-bin.tar.gz
[root@hadoop01 yangql]#  tar -zxvf apache-maven-3.1.1-bin.tar.gz
```
- 配置maven的环境变量,修改.bash_profile文件，加入：
```
export MAVEN_HOME=/home/yangql/apache-maven-3.1.1
export PATH=$PATH:${MAVEN_HOME}/bin
```
执行命令source .bash_profile,使环境变量生效
- 执行命令 mvn -version，显示如下信息时，maven安装成功
```
root@hadoop01 yangql]# mvn -version
Apache Maven 3.1.1 (0728685237757ffbf44136acec0402957f723d9a; 2013-09-17 08:22:22-0700)
Maven home: /home/yangql/maven
Java version: 1.7.0_79, vendor: Oracle Corporation
Java home: /usr/jdk1.7.0_79/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "2.6.18-164.el5", arch: "amd64", family: "unix"
```
- 由于maven国外服务器可能连不上，先给maven配置一下国内镜像
在maven目录下，conf/settings.xml,在<mirrors></mirros>里添加如下内容
```
<mirror>
  <id>nexus-osc</id>  
  <mirrorOf>*</mirrorOf>  
  <name>Nexusosc</name>  
  <url>http://maven.oschina.net/content/groups/public/</url>  
</mirror>
```
```
<profile>  
  <id>jdk-1.7</id>  
  <activation>  
  <jdk>1.7</jdk>  
  </activation>  
  <repositories>  
  <repository>  
   <id>nexus</id>  
   <name>local private nexus</name>  
   <url>http://maven.oschina.net/content/groups/public/</url>  
   <releases>  
    <enabled>true</enabled>  
   </releases>  
   <snapshots>  
    <enabled>false</enabled>  
   </snapshots>  
  </repository>  
  </repositories>  
  <pluginRepositories>  
   <pluginRepository>  
    <id>nexus</id>  
    <name>local private nexus</name>  
    <url>http://maven.oschina.net/content/groups/public/</url>  
    <releases>  
      <enabled>true</enabled>  
    </releases>  
    <snapshots>  
      <enabled>false</enabled>  
    </snapshots>  
   </pluginRepository>  
  </pluginRepositories>  
</profile>
```
## 5 编译hadoop源码 ##
- 解压hadoop源文件，并执行编译命令
```
[root@hadoop01 yangql]#  tar -zxvf hadoop-2.5.1-src.tar.gz
[root@hadoop01 yangql]# cd /home/yangql/hadoop-2.5.1-src
[root@hadoop01 hadoop-2.5.1-src]# mvn package -Pdist,native -DskipTests -Dtar
```
- 编译耗时比较长，成功后结果类似于：
```
main:
     [exec] $ tar cf hadoop-2.5.1.tar hadoop-2.5.1
     [exec] $ gzip -f hadoop-2.5.1.tar
     [exec]
     [exec] Hadoop dist tar available at: /home/yangql/hadoop-2.5.1-src/hadoop-dist/target/hadoop-2.5.1.tar.gz
     [exec]
[INFO] Executed tasks
[INFO]
[INFO] --- maven-javadoc-plugin:2.8.1:jar (module-javadocs) @ hadoop-dist ---
[INFO] Building jar: /home/yangql/hadoop-2.5.1-src/hadoop-dist/target/hadoop-dist-2.5.1-javadoc.jar
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO]
[INFO] Apache Hadoop Main ................................ SUCCESS [15.164s]
[INFO] Apache Hadoop Project POM ......................... SUCCESS [8.006s]
[INFO] Apache Hadoop Annotations ......................... SUCCESS [14.515s]
[INFO] Apache Hadoop Assemblies .......................... SUCCESS [1.927s]
[INFO] Apache Hadoop Project Dist POM .................... SUCCESS [7.819s]
[INFO] Apache Hadoop Maven Plugins ....................... SUCCESS [18.135s]
[INFO] Apache Hadoop MiniKDC ............................. SUCCESS [12.993s]
[INFO] Apache Hadoop Auth ................................ SUCCESS [17.715s]
[INFO] Apache Hadoop Auth Examples ....................... SUCCESS [11.649s]
[INFO] Apache Hadoop Common .............................. SUCCESS [6:07.558s]
[INFO] Apache Hadoop NFS ................................. SUCCESS [34.173s]
[INFO] Apache Hadoop Common Project ...................... SUCCESS [0.259s]
[INFO] Apache Hadoop HDFS ................................ SUCCESS [12:47.239s]
[INFO] Apache Hadoop HttpFS .............................. SUCCESS [1:37.363s]
[INFO] Apache Hadoop HDFS BookKeeper Journal ............. SUCCESS [37.636s]
[INFO] Apache Hadoop HDFS-NFS ............................ SUCCESS [20.800s]
[INFO] Apache Hadoop HDFS Project ........................ SUCCESS [0.432s]
[INFO] hadoop-yarn ....................................... SUCCESS [0.207s]
[INFO] hadoop-yarn-api ................................... SUCCESS [4:55.064s]
[INFO] hadoop-yarn-common ................................ SUCCESS [2:25.453s]
[INFO] hadoop-yarn-server ................................ SUCCESS [0.342s]
[INFO] hadoop-yarn-server-common ......................... SUCCESS [1:02.345s]
[INFO] hadoop-yarn-server-nodemanager .................... SUCCESS [1:16.102s]
[INFO] hadoop-yarn-server-web-proxy ...................... SUCCESS [14.198s]
[INFO] hadoop-yarn-server-applicationhistoryservice ...... SUCCESS [35.040s]
[INFO] hadoop-yarn-server-resourcemanager ................ SUCCESS [1:00.546s]
[INFO] hadoop-yarn-server-tests .......................... SUCCESS [3.670s]
[INFO] hadoop-yarn-client ................................ SUCCESS [28.510s]
[INFO] hadoop-yarn-applications .......................... SUCCESS [0.156s]
[INFO] hadoop-yarn-applications-distributedshell ......... SUCCESS [11.962s]
[INFO] hadoop-yarn-applications-unmanaged-am-launcher .... SUCCESS [8.555s]
[INFO] hadoop-yarn-site .................................. SUCCESS [0.356s]
[INFO] hadoop-yarn-project ............................... SUCCESS [15.177s]
[INFO] hadoop-mapreduce-client ........................... SUCCESS [0.347s]
[INFO] hadoop-mapreduce-client-core ...................... SUCCESS [1:35.106s]
[INFO] hadoop-mapreduce-client-common .................... SUCCESS [1:18.736s]
[INFO] hadoop-mapreduce-client-shuffle ................... SUCCESS [23.857s]
[INFO] hadoop-mapreduce-client-app ....................... SUCCESS [55.494s]
[INFO] hadoop-mapreduce-client-hs ........................ SUCCESS [48.159s]
[INFO] hadoop-mapreduce-client-jobclient ................. SUCCESS [1:43.626s]
[INFO] hadoop-mapreduce-client-hs-plugins ................ SUCCESS [9.165s]
[INFO] Apache Hadoop MapReduce Examples .................. SUCCESS [37.976s]
[INFO] hadoop-mapreduce .................................. SUCCESS [15.514s]
[INFO] Apache Hadoop MapReduce Streaming ................. SUCCESS [1:31.967s]
[INFO] Apache Hadoop Distributed Copy .................... SUCCESS [1:17.597s]
[INFO] Apache Hadoop Archives ............................ SUCCESS [10.781s]
[INFO] Apache Hadoop Rumen ............................... SUCCESS [31.381s]
[INFO] Apache Hadoop Gridmix ............................. SUCCESS [22.927s]
[INFO] Apache Hadoop Data Join ........................... SUCCESS [13.934s]
[INFO] Apache Hadoop Extras .............................. SUCCESS [15.672s]
[INFO] Apache Hadoop Pipes ............................... SUCCESS [31.610s]
[INFO] Apache Hadoop OpenStack support ................... SUCCESS [30.244s]
[INFO] Apache Hadoop Client .............................. SUCCESS [37.309s]
[INFO] Apache Hadoop Mini-Cluster ........................ SUCCESS [0.690s]
[INFO] Apache Hadoop Scheduler Load Simulator ............ SUCCESS [1:38.066s]
[INFO] Apache Hadoop Tools Dist .......................... SUCCESS [17.296s]
[INFO] Apache Hadoop Tools ............................... SUCCESS [0.204s]
[INFO] Apache Hadoop Distribution ........................ SUCCESS [1:46.145s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 54:21.938s
[INFO] Finished at: Sun Jan 08 21:52:17 PST 2017
[INFO] Final Memory: 83M/243M
[INFO] ------------------------------------------------------------------------
```
- 编译后路径在译后的路径在:hadoop-2.5.1-src/hadoop-dist/target/hadoop-2.5.1，进入hadoop-2.5.1目录，测试安装是否成功
```
[root@hadoop01 bin]# ./hadoop version
Hadoop 2.5.1
Subversion Unknown -r Unknown
Compiled by root on 2017-01-09T05:00Z
Compiled with protoc 2.5.0
From source with checksum 6424fcab95bfff8337780a181ad7c78
This command was run using /home/yangql/hadoop-2.5.1-src/hadoop-dist/target/hadoop-2.5.1/share/hadoop/common/hadoop-common-2.5.1.jar
```
## 6 编译中遇到的问题 ##
- 问题描述1：Could not resolve dependencies(Could not resolve dependencies for project org.apache.hadoop:hadoop-minikdc:jar:2.7.0:)
- 问题解决1：解决办法：
这种情况很常见，而且很多都碰到了，他们也是完全按照文档来配置的，但是就不成功，这就是因为插件没有下载完毕造成的。所以尽量多执行几次下面命令
```
mvn package -Pdist,native -DskipTests -Dtar
```

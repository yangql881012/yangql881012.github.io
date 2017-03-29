---
title: flume安装
date: 2017-03-09 20:54:59
tags: flume
categories: flume
---
1.解压重命名  
```
tar -xvf apache-flume-1.7.0-bin.tar.gz
mv apache-flume-1.7.0-bin flume-1.7.0
```
2.配置环境变量
```
#config flume
export FLUME_HOME=/home/yangql/app/flume-1.7.0
export FLUME_HOME_CONF=/home/yangql/app/flume-1.7.0/conf
export PATH=$PATH:$FLUME_HOME/bin
```
2.配置flume.conf
```
cp flume-conf.properties.template flume.conf
```
加入以下内容  
 ```
 # Define a memory channel called ch1 on agent1
 agent1.channels.ch1.type = memory

 # Define an Avro source called avro-source1 on agent1 and tell it
 # to bind to 0.0.0.0:41414. Connect it to channel ch1.
 agent1.sources.avro-source1.channels = ch1
 agent1.sources.avro-source1.type = avro
 agent1.sources.avro-source1.bind = 0.0.0.0
 agent1.sources.avro-source1.port = 41414

 # Define a logger sink that simply logs all events it receives
 # and connect it to the other end of the same channel.
 agent1.sinks.log-sink1.channel = ch1
 agent1.sinks.log-sink1.type = logger

 # Finally, now that we've defined all of our components, tell
 # agent1 which ones we want to activate.
 agent1.channels = ch1
 agent1.sources = avro-source1
 agent1.sinks = log-sink1
 ```

 4.启动
 ```
 bin/flume-ng agent --conf ./conf/ -f conf/flume.conf -Dflume.root.logger=DEBUG,console -n agent1
 ```

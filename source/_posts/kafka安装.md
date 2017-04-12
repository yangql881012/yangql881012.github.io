---
title: kafka安装
date: 2017-03-09 00:54:59
tags: kafka
toc: true
categories: kafka
---
1.解压重命名
```
tar -xvf kafka_2.12-0.10.2.0.tgz
mv kafka_2.12-0.10.2.0 kafka-2.12
```
2.配置环境变量
```
#config kafka 2.12
export KAFKA_HOME=/home/yangql/app/kafka-2.12
export PATH=$PATH:$KAFKA_HOME/bin
```
<!-- more-->
3.修改server.properties

broker.id=0
listeners = PLAINTEXT://hadoop01:9092
advertised.listeners=PLAINTEXT://hadoop01:9092
log.dirs=/home/yangql/app/kafka-2.12/logs
zookeeper.connect=hadoop01:2181,hadoop02:2181,hadoop03:2181

4.同步
```
scp -r kafka-2.12/ yangql@hadoop02:/home/yangql/app/
scp -r kafka-2.12/ yangql@hadoop03:/home/yangql/app/
```
5.修改node1
```
broker.id=1
listeners = PLAINTEXT://hadoop02:9092
advertised.listeners=PLAINTEXT://hadoop02:9092
```
6.修改node2
```
broker.id=2
listeners = PLAINTEXT://hadoop03:9092
advertised.listeners=PLAINTEXT://hadoop03:9092
```
因为Kafka集群需要保证各个Broker的id在整个集群中必须唯一，需要调整这个配置项的值。

7.启动
```
./kafka-server-start.sh -daemon /home/yangql/app/kafka-2.12/config/server.properties &
```
8.测试  
在namenode上创建mytest主题（kafka有几个，replication-factor就填几个）
```
./kafka-topics.sh --create --topic mytest --replication-factor 3 --partitions 2 --zookeeper hadoop01:2181
```
在namenode上查看刚才创建的mytest主题
```
 ./kafka-topics.sh --list --zookeeper hadoop01:2181
```
在datanode1上发送消息至kafka，发送消息“this is for test”
```
./kafka-console-producer.sh --broker-list hadoop01:9092 --sync --topic mytest
```
在datanode2上开启一个消费者，模拟consumer，可以看到刚才发送的消息
```
./kafka-console-consumer.sh --zookeeper hadoop01:2181 --topic mytest --from-beginning
```

---
title: Spark两种提交方式Yarn-client and Yarn-cluster
date: 2017-04-12 10:54:59
tags: Spark
toc: true
categories: 大数据技术
---
Spark支持三种集群部署方式(Standalone,Mesos,Yarn),其中Master服务(Spark Standalone,Mesos Master,Yarn ResourceManager)决定哪些应用可以运行，在那个节点上运行，以及什么时候运行。Slave服务（Yarn NodeManager）运行在每个节点上，节点控制着Executor进程，同时监控作业的运行状态以及资源的消耗。Spark运行在Yarn上，有两种模式，Yarn-Client和Yarn-Cluster。通常情况下，Yarn-Cluster用于生产环境，Yarn-Client用于交互、调试。
<!-- more -->
## 1.Appliaction Master ##
在Yarn中，每个application都有一个Application Master进程，它是Appliaction启动的第一个容器，它负责从ResourceManager中申请资源，分配资源，同时通知NodeManager来为Application启动container，Application Master避免了需要一个活动的client来维持，启动Applicatin的client可以随时退出，而由Yarn管理的进程继续在集群中运行。  
当在Yarn下运行Spark作业时，每个Spark Executor作为一个Yarn 容器(container)在运行，同时支持多个任务在同一个容器中运行，节省了任务的启动时间。
## 2.Yarn-client ##
在Yarn-client模式下，AM仅仅从Yarn中申请资源分配给Executor，之后client会跟容器(Container)通信进行作业调度。Client不能离开.如下图所示：
![](Spark两种提交方式Yarn-client and Yarn-cluster/spark-yarn-client.png)
执行流程：
- 客户端提交作业给ResourceManager(RM)
- RM在本地NodeManager(NM)启动container并将AM分配给该NM
- NM接收到RM的分配，启动Application Master并初始化作业，此时这个NM就称为Driver
- Application向RM申请资源，分配资源同时通知其他NodeManager启动相应的Executor
- Executor向本地启动的AM注册汇报并完成相应的任务
## 3.Yarn-Cluster ##
在Yarn-cluster模式下，driver运行在AM上，AM进程同时负责驱动Application和从Yarn中申请资源，该进程运行在Yarn container内，所以启动AM的client可以立即关闭而不必持续到Application的生命周期，如下图所示：
![](Spark两种提交方式Yarn-client and Yarn-cluster/spark-yarn-culster.png)
执行流程：
- 客户端生成作业信息提交给ResourceManager(RM)
- RM在某一个NodeManager(由Yarn决定)启动container并将Application Master(AM)分配给该NodeManager(NM)
- NM接收到RM的分配，启动Application Master并初始化作业，此时这个NM就称为Driver
- Application向RM申请资源，分配资源同时通知其他NodeManager启动相应的Executor
- Executor向NM上的Application Master注册汇报并完成相应的任务

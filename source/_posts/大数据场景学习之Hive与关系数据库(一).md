---
title: 大数据场景学习之Hive与关系数据库(一)
date: 2017-07-15 15:54:59
tags: 大数据场景学习
toc: true
categories: 大数据技术
---
## 一.业务场景 ##
1. 从关系数据库中获取任务信息
2. 根据从数据库中的任务信息，从Hive中过滤满足条件的数据
3. 将数据加载到回关系数据库中

## 二.开发环境 ##
1. MySQL
2. hive-2.1.1
3. hadoop-2.7.2
4. spark-2.1.0-bin-hadoop2.7
<!-- more -->

## 三.MySQL 数据库相关表 ##
- crm_brch_topN_info
```
CREATE TABLE `crm_brch_topN_info` (
  `statt_dt` date DEFAULT NULL,
  `2` int(11) NOT NULL,
  `cust_no` text,
  `org_no` text,
  `bal` double DEFAULT NULL,
  `rn` int(11) DEFAULT NULL
) ENGINE=MyISAM DEFAULT CHARSET=utf8
```
- task
```
CREATE TABLE `task` (
  `task_id` int(11) NOT NULL AUTO_INCREMENT,
  `task_name` varchar(255) DEFAULT NULL,
  `create_time` varchar(255) DEFAULT NULL,
  `start_time` varchar(255) DEFAULT NULL,
  `finish_time` varchar(255) DEFAULT NULL,
  `task_type` varchar(255) DEFAULT NULL,
  `task_status` varchar(255) DEFAULT NULL,
  `task_param` text,
  PRIMARY KEY (`task_id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8
task_id	task_name	create_time	start_time	finish_time	task_type	task_status	task_param
2	      取排名前十客户	1	       22	         333	      33	       333	       {"brchId":"88088,85018"}
```

## 四.配置文件application.conf ##
```
jdbc.driver:"com.mysql.jdbc.Driver"
jdbc.url:"jdbc:mysql://hadoop01:3306/sparkpro"
jdbc.user:"root"
jdbc.password:"root123"
taskTableName:"task"
custBrchBalTopN:"crm_brch_topN_info"
updateHbaseCustBalOutPutTbl:"crm:bcst_t_ep_bal"
kafka-broker:"hadoop01:9092,hadoop02:9092,hadoop03:9092"
checkpoint-path:"/user/kafka/checkpoint"
phoenix-url:"jdbc:phoenix:hadoop01,hadoop02,hadoop03"
kafkaTopics:"custbal,custbal_nbc"
hbaseTableName:""""crm:bcst_t_ep_bal""""
tblKeys:""""key","cust_no","bal""""
```

## 五.pom.xml信息 ##
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>
  <groupId>com.yangql</groupId>
  <artifactId>spark</artifactId>
  <version>1.0</version>
  <inceptionYear>2008</inceptionYear>
  <properties>
    <scala.version>2.11.8</scala.version>
  </properties>

  <repositories>
    <repository>
      <id>scala-tools.org</id>
      <name>Scala-Tools Maven2 Repository</name>
      <url>http://scala-tools.org/repo-releases</url>
    </repository>
  </repositories>

  <pluginRepositories>
    <pluginRepository>
      <id>scala-tools.org</id>
      <name>Scala-Tools Maven2 Repository</name>
      <url>http://scala-tools.org/repo-releases</url>
    </pluginRepository>
  </pluginRepositories>

  <dependencies>
    <dependency>
      <groupId>org.scala-lang</groupId>
      <artifactId>scala-library</artifactId>
      <version>${scala.version}</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.4</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.specs</groupId>
      <artifactId>specs</artifactId>
      <version>1.2.5</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.17</version>
    </dependency>
      <dependency>
          <groupId>org.apache.spark</groupId>
          <artifactId>spark-core_2.11</artifactId>
          <version>2.1.0</version>
      </dependency>
      <dependency>
          <groupId>org.apache.spark</groupId>
          <artifactId>spark-sql_2.11</artifactId>
          <version>2.1.0</version>
      </dependency>
      <dependency>
          <groupId>org.apache.spark</groupId>
          <artifactId>spark-hive_2.11</artifactId>
          <version>2.1.0</version>
      </dependency>
      <dependency>
          <groupId>org.apache.spark</groupId>
          <artifactId>spark-streaming_2.11</artifactId>
          <version>2.1.0</version>
      </dependency>
      <dependency>
          <groupId>org.apache.hadoop</groupId>
          <artifactId>hadoop-client</artifactId>
          <version>2.7.2</version>
      </dependency>
      <dependency>
          <groupId>org.apache.spark</groupId>
          <artifactId>spark-streaming-kafka-0-8_2.11</artifactId>
          <version>2.1.0</version>
      </dependency>
      <dependency>
          <groupId>org.apache.spark</groupId>
          <artifactId>spark-streaming-flume_2.11</artifactId>
          <version>2.1.0</version>
      </dependency>
      <dependency>
          <groupId>org.apache.hive</groupId>
          <artifactId>hive-jdbc</artifactId>
          <version>0.13.0</version>
      </dependency>
      <dependency>
          <groupId>org.apache.httpcomponents</groupId>
          <artifactId>httpclient</artifactId>
          <version>4.4.1</version>
      </dependency>
      <dependency>
          <groupId>org.apache.httpcomponents</groupId>
          <artifactId>httpcore</artifactId>
          <version>4.4.1</version>
      </dependency>
     <dependency>
	    <groupId>com.typesafe.play</groupId>
	    <artifactId>play-json_2.11</artifactId>
	    <version>2.5.12</version>
     </dependency>
     <dependency>
		  <groupId>org.joda</groupId>
		  <artifactId>joda-convert</artifactId>
		  <version>1.8</version>
     </dependency>
     <!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase -->
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase</artifactId>
    <version>1.2.5</version>
    <type>pom</type>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase-common -->
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-common</artifactId>
    <version>1.2.5</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase-server -->
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-server</artifactId>
    <version>1.2.5</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.hbase/hbase-client -->
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-client</artifactId>
    <version>1.2.5</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.hive/hive-common -->
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-common</artifactId>
    <version>2.1.1</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.phoenix/phoenix-core -->
<dependency>
    <groupId>org.apache.phoenix</groupId>
    <artifactId>phoenix-core</artifactId>
    <version>4.10.0-HBase-1.2</version>
</dependency>


  </dependencies>
  <build>
    <sourceDirectory>src/main/scala</sourceDirectory>
    <testSourceDirectory>src/test/scala</testSourceDirectory>
    <plugins>
        <plugin>
        <artifactId>maven-assembly-plugin</artifactId>
        <configuration>
          <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
          </descriptorRefs>
          <archive>
            <manifest>
              <mainClass></mainClass>
            </manifest>
          </archive>
        </configuration>
        <executions>
          <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
              <goal>single</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
      <plugin>
        <groupId>org.scala-tools</groupId>
        <artifactId>maven-scala-plugin</artifactId>
        <executions>
          <execution>
            <goals>
              <goal>compile</goal>
              <goal>testCompile</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <scalaVersion>${scala.version}</scalaVersion>
          <args>
            <arg>-target:jvm-1.8</arg>
          </args>
        </configuration>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-eclipse-plugin</artifactId>
        <configuration>
          <downloadSources>true</downloadSources>
          <buildcommands>
            <buildcommand>ch.epfl.lamp.sdt.core.scalabuilder</buildcommand>
          </buildcommands>
          <additionalProjectnatures>
            <projectnature>ch.epfl.lamp.sdt.core.scalanature</projectnature>
          </additionalProjectnatures>
          <classpathContainers>
            <classpathContainer>org.eclipse.jdt.launching.JRE_CONTAINER</classpathContainer>
            <classpathContainer>ch.epfl.lamp.sdt.launching.SCALA_CONTAINER</classpathContainer>
          </classpathContainers>
        </configuration>
      </plugin>
    </plugins>
  </build>
  <reporting>
    <plugins>
      <plugin>
        <groupId>org.scala-tools</groupId>
        <artifactId>maven-scala-plugin</artifactId>
        <configuration>
          <scalaVersion>${scala.version}</scalaVersion>
        </configuration>
      </plugin>
    </plugins>
  </reporting>
</project>
```

## 六.工具类 ##
```
package com.crm.utils

import play.api.libs.json.Json

object HandleParasUtils {

  /**
   * 从Json中获取提取指定值
   */
  def getParam(param:String,field:String):Option[String]={
    val json=Json.parse(param)
    val result=(json \ field).asOpt[String]
    result
  }

  /**
   * 机构拼接
   * 源     xxxx,yyyy
   * 目标 'xxxx','yyyy'
   */
  def brchIdsConcat(brchIds:String):String={
    val splitedBrchIds=brchIds.split(",")
    var result=""
    for(i <- (0 until splitedBrchIds.length)){
      result += "'"+splitedBrchIds(i) +"',"
    }
    result="("+result.substring(0,result.length()-1)+")"
    result
  }
  def  main(args: Array[String]): Unit = {
    println(brchIdsConcat("xxxx,yyyy,zzzzzz"))
  }

}
```

## 七.实现类 ##
```
package com.crm.query

import org.apache.spark.sql.SparkSession
import java.sql.DriverManager
import org.apache.spark.SparkConf
import org.apache.spark.sql.SQLContext
import org.apache.spark.SparkContext
import org.apache.spark.sql.hive.HiveContext
import org.apache.spark.sql.types.StructType
import org.apache.spark.sql.types.StructField
import org.apache.spark.sql.types.StringType
import org.apache.spark.sql.types.DoubleType
import org.apache.spark.sql.Row
import com.project.bean.Task
import com.project.dao.impl.DaoFactory
import com.typesafe.config.ConfigFactory
import com.crm.utils.HandleParasUtils
import scala.util.Properties
import org.apache.spark.sql.SaveMode

object QueryBrchTopNCustomer {
  def main(args: Array[String]): Unit = {
    //通过参数设置程序启动参数
    val Array(
       taskId,  //任务ID  
       statt_dt //统计日期
    )=args

    if(taskId==null){
      println("请输入任务编号")
      sys.exit()
    }
    //加载作业QueryBrchTopNCustomer得配置文件
    val config = ConfigFactory.load("application.conf")

    val conf = new SparkConf()
      .setMaster("local")
      .setAppName("JDBCDataSrc")
    val sc = new SparkContext(conf)
    sc.setLogLevel("ERROR")
    val sqlContext=new SQLContext(sc)
    val hiveContext=new HiveContext(sc)

    //导入隐式转换
    import sqlContext.implicits._

    val url=config.getString("jdbc.url")
    val user=config.getString("jdbc.user")
    val password=config.getString("jdbc.password")
    val targetTblName=config.getString("custBrchBalTopN")

    //根据任务编号将任务找出来并获得任务得参数
    val taskDao= DaoFactory.getTaskDao()
    val task=taskDao.findByKey(taskId.toLong)
    val taskParam=task.task_param
    val brchIds=HandleParasUtils.getParam(taskParam, "brchId").getOrElse("000000")
    /**
     * 源     xxxx,yyyy
     * 目标 'xxxx','yyyy'
     */
    val whereCond=HandleParasUtils.brchIdsConcat(brchIds)
    hiveContext.sql("use crm")
    /**
     * 数据分组排序
     */
    //println(statt_dt)
    val sql="""select to_date('"""+statt_dt+"""') as statt_dt,"""+
                                                  taskId+""",
                                                   cust_no,
                                                   org_no,
                                                   sum(COALESCE(bal,0)) as bal ,
                                                   row_number() over(partition by org_no order by sum(nvl(bal,0))) rn
                                              from crm.bacc_t_ep_deps
                                              where org_no in
                                              """ +
                                            whereCond +
                                             """
                                              group by cust_no,org_no
                                              """
    //println(sql)
    val bacc_t_ep_depsDF=hiveContext.sql(sql)

    bacc_t_ep_depsDF.show(20,true)
    //设置mysql相关参数
    val props=new java.util.Properties
    props.setProperty("user", user)
    props.setProperty("password", password)

    //将DF数据写入到mysql
    bacc_t_ep_depsDF.write.mode(SaveMode.Overwrite).jdbc(url, targetTblName, props)

    sc.stop()
  }
}
```

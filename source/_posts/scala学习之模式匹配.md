---
title: scala学习之模式匹配
date: 2017-02-10 00:54:59
tags: 模式匹配
categories: scala
---
1.样本类：添加了case的类便是样本类。这种修饰符可以让Scala编译器自动为这个类添加一些语法上的便捷设定。如下：  
2.添加与类名一致的工厂方法。也就是说，可以写成Var("x")来构造Var对象。  
3.样本类参数列表中的所有参数隐式获得了val前缀，因此它被当作字段维护。  
4.编译器为这个类添加了方法toString,hashCode和equals等方法。    
5..match对应Java里的switch，但是写在选择器表达式之后。即： 选择器 match {备选项}。  
6.一个模式匹配包含了一系列备选项，每个都开始于关键字case。每个备选项都包含了一个模式及一到多个表达式。箭头符号 => 隔开了模式和表达式。  
7.match表达式通过以代码编写的先后次序尝试每个模式来完成计算。类似于UnOp("-" , UnOp("-" , e))这种形式的，是构造器模式匹配。  
8.Scala的备选项表达式永远不会“掉到”下一个case；  
9.如果没有模式匹配，MatchError异常会被抛出。这意味着必须始终确信所有的情况都考虑到了，或者至少添加一个默认情况什么都不做。如 case _ =>  
10.通配模式：case _ => 。表示默认的全匹配备选项。通配模式还可以用来忽略对象中不关心的部分。如：case BinOp(_,_,_) => XXX，则表示不关心二元操作符的元素是什么，只是检查是否为二元操作符  
11.常量模式 ：仅匹配自身。任何字面量都可以用作常量。包括String类型。另外，任何的val或单例对象也可以被用作常量。如，单例对象Nil是只匹配空列表的模式。  
12.变量模式 ：变量模式类似于通配符，可以匹配任何对象。不同点在于，Scala把变量绑定在匹配的对象上。之后就可以使用这个变量操作对象。
<!-- more -->
```
class DataFramwork
//class class must have arguments
case class ComputationFramework(name:String,popular:Boolean) extends DataFramwork
case class SparkFramework(name:String,popular:Boolean) extends DataFramwork

object HelloPatternMatch {
  def main(args: Array[String]): Unit = {
    //getSary("Hadoop")
    getSary("Test", 6)
    getMatchType(Array(3))
    getCollectionsType(Array("Spark"))  
    getCollectionsType(Array("Scala"))
    getCollectionsType(Array("Scala","Java"))

    getBigDataType(ComputationFramework("Mapreduce",true))
    getBigDataType(SparkFramework("Spark",true))

    getValue("Java",Map("Java"->"The best","Scala"->"The second","Spark"->"The most"))

  }
  //模式匹配
  def getSary(name:String,age:Int){
    name match{
      case "Spark" => println("Spark")
      case "Scala" => println("Scala")
      case _ if name == "Hadoop"  => println("Hadoop")
      //将传入参数name值传给变量_name
      case _name if age>5 => println(_name + " age is " + 5 )
      case _ => println("others")
    }
  }

  //模式匹配 匹配数据类型
  def getMatchType(msg:Any){
    msg match{
      case i:Int => println("Integer")
      case s:String => println("String")
      case d:Double => println("Double")
      //只能匹配Array不能具体匹配内容
      case array:Array[Int] => println("Array")
      case  _ => println("Unknow")
    }
  }
  //匹配集合中具体的值
  def getCollectionsType(msg:Array[String]){
    msg match{
      case Array("Scala") => println("one element")
      case Array("Scala","Java") => println("two elements")
      case Array("Spark",_*) => println("many elements")
      case _ => println("unknow")
    }
  }
  //匹配模式类
  def getBigDataType(data:DataFramwork){
    data match{
      case ComputationFramework(name,popular) => println("ComputationFramwork name: " + name + " popular :" + popular)
      case SparkFramework(name,popular) => println("SparkFramework name: " + name + " popular :" + popular)
      case _ => println("Unkown class")
    }
  }

  //值匹配
  def getValue(key:String,context: Map[String,String]){
    context.get(key) match{
      case Some(value) => println(value)
      case None => println("Not Found")
    }
  }
}
```  

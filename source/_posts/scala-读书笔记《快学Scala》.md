---
title: 快学Scala
date: 2017-03-28 10:54:59
tags: scala
categories: 读书笔记
---
### 1.可变参数列表 ###
```
//定义一个函数,可变长度参数
  def sum(args:Int*):Int={
    var result = 0
    for(arg <- args){
      result += arg
    }
    result
  }
```
<!-- more -->
`val s = sum(1 to 5)`//错误  
如果sum函数被调用时传入的是单个参数，那么该参数就应该是单个整数，而不是一个整数区间。解决这个问题的办法就是告诉编译器你希望这个参数被当作参数序列处理
正确的做法  
```
val s =sum(1 to 5:_*)
```  
在递归调用中，用`_*`将其转化为参数序列
```
  //递归调用
  def recursiveSum(args:Int*):Int={
    if(args.length==0) 0
    else args.head + recursiveSum(args.tail:_*)
  }
```

### 2.过程 ###
scala对于不返回值得函数有特殊的表示方法，如果函数提包含在括号中，但是前面没有=号，那么返回值就是Unit类型，这样的函数被称为过程。  
打印------------
```
println("-" * 10)
```
### 3.懒值 ###
```
lazy val wordCount=Source.fromFile("D:/test.txt").mkString
```
如果程序从不访问变量wordCount,那么文件永远不会被打开，懒值对于开销较大的初始化语句十分有用。
### 4.异常 ###
throw 表达式有特殊的类型Nothing，在if/else 表达式中，如果一个分支的类型是Nothing，if/else表达式的类型就是另一个分支的类型。

## 2.数组 ##
不可变数组使用Array  
可变数组使用ArrayBuffer  
```
 val arrayBuffer =ArrayBuffer[Int](1,2)
  arrayBuffer += 3
  val array=Array(4,5,6)
  arrayBuffer ++= array
  arrayBuffer.trimEnd(3)
  println(arrayBuffer.mkString(","))

```
+=:在尾端添加元素  
++=：追加集合  
trimEnd(n):移除最后的n个元素  
toArray:转化为数组  
toBuffer():转化为数组缓冲  
0 to n:[0,n]  
0 until n:[0,n)  
0 until(n,2):每两个元素一跳  
(0 until n).reverse

### 1.数组转化 ###
数组转化并不会修改原始数组，而是会产生一个全新的数组  
使用for 推导式  
```
  val a = Array(1,2,3,4)
  val result = for(item <- a) yield item*2
  println(result.mkString(","))
```
### 2.数组守卫 ###
```
for(elem <- a if elem % 2 ==0 ) yield elem * 2
```
array.sum  
array.min  
array.max  
println(a.sorted)  //升序  
println(a.sortWith(_<_))//升序  
println(a.sortWith(_>_))//降序  

### 3.多维数组 ###
```
 val matrix = Array.ofDim[Double](3,4)
 matrix(2)(1)=42
 println(matrix(2)(1))
```

## 映射 ##
```
val scores=Map("key1"->"values","key2"->"value2")
val scores =Map((key1,value1),(key2,value2))
//获取值
scores("key1")
contains//判断是否含有某个元素
scores.getOrElse("key1","0")//如果没有，就取默认值
//如果可变
score("bog")="bog"//如果存在就更新，不存在就新增
+=：添加多个关系
-=：移除某个键和对应的值
迭代映射
for((k,v) <- map)
map.keySet:获取key
map.values:获取值
反转键和值:
for((k,v)<-map) yield(v,k)
```
### 元组 ###
val tuple=（1，2,3）
元组访问:tuple._1 tuple._2 tuple._3  
使用模式匹配来获取元组的组员，  
val (first,second,third) = tuple
如果不需要某个值，可以在不需要的部件上使用`_` val (first,second,_)=tuple  
println("New YourK".partition(_.isUpper)):  
partition():返回元组（满足条件，不满足条件）

使用元组的好处就是可以将多个值绑在一起进行处理，可以使用zip来处理。
```
  val array=Array("key1","key2","key3")
  val count=Array(1,2,3)
  val paris =array.zip(count)
  for((k,v) <- paris) println(k + " " + v)
```
## 类 ##
1.定义一个类时，定义一个公有变量是，默认生成JVM类时，变量为私有，getter setter方法是公有的。定义私有变量时，变量，getter setter方法都是私有的。getter
setter 分别叫做age age_=
```
class Person {
  private var privateage=0
  def age=privateage
  def age_=(newAge:Int){
    privateage=newAge
  }
}
```
2.如果字段是私有的，则getter and setter 方法也是私有的  
3.如果字段是val的，则只有getter方法生成  
4.如果你不需要任何getter setter方法，可以将字段声明为`private[this]`  
5.如果定义方法没有(),则在调用时，也不能加()  
6.方法可以访问该类所有对象的私有字段  
```
class Counter{
  private var value =0
  def isLess(other:Counter) = value < other.value//other同样也是Counter对像
}
```
7.pirvate[this] 限制只能是当前对象才能访问
```
class Counter1{
  private[this] var value =0
  def isLess(other:Counter1) = value < other.value//错误，只能访问当前对象
}
```
8.private[类名]修饰符可以定义仅有指定的类的方法可以访问给定的字段，这里的类名必须是当前定义的类，或者是包含该类的外部类。
```
class Counter1{
  private[Counter1] var value =0
  def isLess(other:Counter1) = value < other.value//错误，只能访问当前对象
}
```
9.Bean属性，生成getXXX,setXX形式的方法，在属性前面加上`@BeanPropery`
```
import scala.beans.BeanProperty
class Persons{
  @BeanProperty var name:String="default"
}
//生成四个方法
1.name:String
2.name_=(newValue:String):Unit
3.getName():String
4.setName(newValue:String):Unit
```
![image](E:\03_学习资料\06_IT学习\02_编程语言\07_Scala\scala字段生成方法.png)
10.辅助构造器,
- 辅助构造器的名称为this
- 每一个辅助构造器都必须以一个对先前已经定义的其它辅助构造器或者主构造器开始。
 ```
 class Person {
   var age=0
   var name =""

  def this(name:String){//一个辅助构造器
    this()//调用柱构造器
    this.name=name
  }
  def this(name:String,age:Int){//另外一个辅助构造器
    this()//也可以是 this(name)
    this.name=name
    this.age=age
  }
}
//==============================================
package com.scala.base

object TestClass extends App{
  val p1 = new Person()//主辅助器
  val p2 = new Person("test",1)//第一个辅助器
  val p3 = new Person("test")//第二个辅助器
  println(p1.name + " " + p1.age)
  println(p2.name + " " + p2.age)
  println(p3.name + " " + p3.age)
}
 ```   
 11.主构造器
 - 每个类都有主构造器，主构造器并不以this方法定义。而是: class Person(val name:String,val age:Int)
 - 针对主构造器参数生成的字段和方法

 |柱构造器参数|生成的字段和方法|
|---|---|
name:Stirng|对象私有字段，如果没有方法使用name，那么就没有这个字段
private val/var name:String|私有字段，私有的getter setter
val/var name:String|私有字段，公有的getter setter
@BeanPropery val/var name:String|私有字段，JavaBean版本的setter getter

- class peroson private(val id:Int):主构造器变为私有

## 对象 ##
1.对象的作用(object)
- 作为存放工具函数或者常量的地方
- 高效地共享单个不可变实例
- 需要用单个实例来协调某个服务
2.伴生对象，类的伴生对象可以被访问，但并不在作用域中，
3.枚举
```
object TrafficLightColor extends Enumeration{
  //val red,yello,green = Value
  val red = Value(0,"Red")
  val yellow = Value(1,"yellow")
  val green = Value(2,"green")
}
```
```
object TestClass extends App{
  println(TrafficLightColor.red)
  println(TrafficLightColor.yellow)
  for(c <- TrafficLightColor.values){
    println(c.id + " " + c)
  }
  println(TrafficLightColor(0))
  println(TrafficLightColor.withName("green"))
}
```
## 类 ##
- extends关键字，继承类。
- final 修饰类时，类不能被扩展
1.类型检查和转化
- isInstanceOf:测试某个对象是否属于某个特定的类
- asInstanceOf:将引用转化为子类的引用
- p.getClass=classOf(类名) ：测试P指向的是某个对象但不是其子类。
- 抽象类:abstract,在子类中重写抽象方法时，不需要使用override关键字
- val 定义完了 引用对象的地址就不会变了
- def 定义后 每调用一次就会被重新执行一次
```
abstract class Person0322(val name:String){
  def id:Int //没有方法体，这是一个抽象方法
}
class Employee0322(name:String) extends Person0322(name){
  def id = name.hashCode()
}
```
- 抽象字段：没有初始值的字段
- 匿名类
```
    val alien = new Person("Fred"){
    def greeting="dddddd" + name
  }
```
- 提前定义：
```
class Ant extends {
    override val range=2
} with Animal
```
## 文件及正则表达式 ##
1.scala.io.Source.fromFile("文件名","字符编码")   
Source.fromURL
source.fromString
- 读取二进制文件：scala没有提供，需要使用java的类库
```
val file = new File(fileNmae)
val in = new FileInputStream(file)
val bytes = new Array[Byte](file.length.toInt)
in.read(bytes)
in.close()
```
- 写入文件使用java PrintWriter
- 序列化
```
@SerialVersionUID(42L) class Counter extends Serializable{
  private var value =0
  def isLess(other:Counter) = value < other.value//other同样也是Counter对像
}
```
- sys.process提供用于与shell交互。
```
import sys.process._
"ls -al .." ! //返回上层目录的所有文件 !!两个感叹号是字符串形式返回
"ls -al .." #| "grep xxx" ! //#|管道
#>重定向
```
- 正则表达式类：scala.util.matching.Regex
```
 import scala.util.matching.Regex
  val numPattern="[0-9]+".r
  for(matching <- numPattern.findAllIn("dsafdsa sa33 333")){//匹配所有
    println(matching)
  }
  //转成数组
  val array=numPattern.findAllIn("dd321 ").toArray
  //匹配收个
  val matchfirst=numPattern.findFirstIn("dasfda asf3")
  //检查开始时候匹配
  val firstmatch=numPattern.findPrefixOf("")
  //替换
  numPattern.replaceAllIn("", "")
  numPattern.replaceFirstIn("", "")
  //正则表达式组
  val numGroupPatter="([0-9]+) ([a-z]+)"
}
```
## 特质 ##
- 特质可以同时拥有抽象方法和具体方法，不需要将方法声明为abstract,未被实现的方法默认就是抽象的
- 继承多个特质时，使用with语句
- scala只能有一个超类，但是可以有多个特质
- 子类实现特质的方法时，不需要写override
```
package com.scala.base

trait Logger {
  def log(msg:String) //这是一个抽象方法。没有实现
}
package com.scala.base

class ConsoleLogger extends Logger{
  def log(msg:String){ //不需要写override
    println(msg)
  }
}
```
- 特质可以带部分实现，带有特质的对象
```
  val test=new Account with ConsoleLogger
  test.log("test")
```
- 继承多个特质时，特质从最后一个开始处理
```
  val acct1 = new Account  with TimestampLogger with shorterLogger
  val acct2 = new Account  with shorterLogger with TimestampLogger
  acct1.log("acct1")
  acct1.log("acct2")
```
- 在特质中重新抽象方法
```
trait Logger {
  def log(msg:String) //这是一个抽象方法。没有实现
}

trait TimestampLogger extends Logger{
  println("TimestampLogger")
  abstract override def log(msg:String){//scala 认为TimestampLogger是抽象的，必须要混入一个具体log方法，所以必须要加上abstrat 和 override关键字
    super.log(new java.util.Date() + " " +msg)
  }
}
```
- 当作富接口使用的特质
```
trait Logger {
  def log(msg:String) //这是一个抽象方法。没有实现
  def info(msg:String){log("info "+ msg)}
  def warn(msg:String){log("warn" + msg)}
  def error(msg:String){log("error" + msg)}
}
```
- 特质中的字段可以是具体的，也可以是抽象的，如果你给出了初始值，那么字段就是具体的。任何通过这种方式混入的字段都会自动成为该类自己的字段，特质中未被初始化的字段在子类中必须重写
- 特质不能有构造器参数，每个特质都有一个无参数的构造器（缺少构造器参数是特质与类之间唯一的技术差别）
```
//提前定义
val acc = new {
val filenmae ="file.txt"

}with SavingAccount with FileLogger
```
- 自身类型：当特质以如下代码开始定义时， this:类型=>,它便只能被混入指定类型的子类
```
  def mulOne(x:Int)=(y:Int)=>{
    x*y
  }
  println(mulOne(7)(8))
```
## 高阶函数 ##
val fun = ceil _ //( _ 将fun方法转化为函数 )
- 柯里化：将原来接受两个参数的函数变成新的接受一个参数的函数过程，新的函数返回一个以原有第二个参数作为参数的类型
- unioin【|/++】 intersect【&】 diff【&~/--】
- - 一般而言，+用于将元素添加到无先后次序的集合，+:和:+将元素添加到有先后次序的集合的开头和末尾。++ 一次添加多个元素

## 模式匹配 ##
```
ch match{
case '+' => sign=1
case '-' => sign=-1
case _ if ch==1 => kk //守卫是以任何boolean条件
csse x:Int=>Integer.ParseInt(x) //类型匹配，在匹配类型的时候，必须给出一个变量名，否则，你将会拿对象本身来进行匹配
case _ => sign=0
}

arry match{
case Array(0) => "0" //匹配包含0的数组
case Array(x,y) => x + y //匹配任何包含两个元素的数组
case Array(0,_*) => "0 ...."//匹配包含任何以0开始的数组
case _ => "something"
}
```
- 样例类时一种特殊的类，经过优化被用于模式匹配
- copy方法和带名参数
```
val amt=Account("test")
val price=amt.copy("teset1")
```
- sealed 密封类

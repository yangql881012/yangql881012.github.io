---
title: scala集合
date: 2017-03-03 00:54:59
tags: 集合
toc: true
categories: scala
---
数组是一种可变的、可索引的数据集合。在Scala中用Array[T]的形式来表示Java中的数组形式 T[]。以下介绍各种集合操作方法。
## 1. ++ ##
合并集合，并返回一个新的数组，新数组包含左右两个集合对象的内容。
```
def ++[B](that: GenTraversableOnce[B]): Array[B]
```
```
val a = Array(1,2)
val b = Array(3,4)
val c = a ++ b
println(c.mkString(","))
//output
1,2,3,4
```
## 2. ++: ##
合并集合，并返回一个新的数组，新数组包含左右两个集合对象的内容。但是不同的是右边操纵数的类型决定着返回结果的类型
```
def ++:[B >: A, That](that: collection.Traversable[B])(implicit bf: CanBuildFrom[Array[T], B, That]): That
```
```
val a = List(1,2)
val b = scala.collection.mutable.LinkedList(3,4)
val c = a ++: b
println(c.mkString(","))
println(c.getClass().getName())
//output
1,2,3,4
scala.collection.mutable.LinkedList
```
## 3. +: ##
在数组前面添加一个元素，并返回新的对象
```
def +:(elem: A): Array[A]
```
```
val a = List(1,2)
val c = 0 +: a
println(c.mkString(","))
```
## 4. :+ ##
在数组末尾添加一个元素，并返回新对象
```
def :+(elem: A): Array[A]
```
```
val a = List(1,2)
val c =  a :+ 0
println(c.mkString(","))
//output
1,2,0

```
## 5. fold ##
fold函数是将一种格式的输入数据转换成另外一种格式，fold函数需要输入两个参数，初始值，以及一个函数，输入的函数也需要两个参数，累加值和当前item的索引。
```
val numbers = Array(1, 2, 3, 4)
   val result =numbers.fold(0)({
      (z,i)=>{
        z-i
      }
    })
   println(result)
//output
-10
//
```
- 代码开始运行时，初始值0作为第一个参数传入到fold函数中，array中的第一个item作为第二个参数传入到fold函数中
- fold函数开始对传进的参数进行计算，此处是加法计算，然后返回值
- fold函数将上一步返回的值作为第一个参数，并把array中的第二个item作为参数传进去继续计算，同样返回值。
- 重复上一步计算，知道array中所有元素都遍历完。
应用例子:

```
package com.scala.base

class Foo(val name:String,val age:Int,val sex:Symbol) {

}

object Foo{
  def apply(name:String,age:Int,sex:Symbol)=new Foo(name,age,sex)
}

var fooList=Foo("name1",10,'male)::
                 Foo("name2",10,'female)::
                 Foo("name3",10,'male)::Nil
   var stringList=fooList.foldLeft(List[String]())(
   (z,f)=>{
     var title=f.sex match {
       case 'male => "Mr."
       case 'female => "Ms."
     }
     z :+ s"$title  ${f.name}   ${f.age}"
   }    
   )
   println(stringList)
   //初始值 空list
   //output
   //List(Mr.  name1   10, Ms.  name2   10, Mr.  name3   10)
```
## 6. /:  foldLeft ##
对数组中所有的元素进行相同的操作 ，foldLeft 的简写
```
def /:[B](z: B)(op: (B, T) ? B): B
```
```
val a = List(1,2,3,4)
val c = (10 /: a)(_+_)   
val d = (10 /: a)(_*_)   
println("c:"+c)   
println("d:"+d)   
//output
c:20
d:240
```
## 7. :\ foldRight ##
对数组中所有的元素进行相同的操作 ,foldRight 的简写
## 8.fold foldLeft foldRight 区别 ##
- foldLeft从左开始计算，然后往右遍历
- foldRight从右开始计算，然后往左遍历
- fold遍历的顺序没有特殊的次序，但初始化参数和返回值都有限制，一：初始值的类型必须是list元素中元素类型的超类，二：初始值必须是中立的，也就是它不能改变结果，比如加法，中立值为0，乘法中立值为1，数组来说是Nil
- 三个函数中，初始化和返回值得参数类型必须相同
<=未完待续=>

---
title: Scala学习之偏函数-异常-lazy
date: 2017-02-11 16:54:59
tags: Scala
toc: true
categories: 大数据技术
---
偏函数：当函数有多个参数，而在使用该函数时不想提供所有参数（比如函数有3个参数），只提供0~2个参数，此时得到的函数便是偏函数。  

Scala中使用关键字lazy来定义惰性变量，实现延迟加载(懒加载)。 惰性变量只能是不可变变量，并且只有在调用惰性变量时，才会去实例化这个变量。  
<!-- more -->
1.偏函数  
```
package com.spark
/**
 * 偏函数：当函数有多个参数，而在使用该函数时不想提供所有参数（比如函数有3个参数），只提供0~2个参数，此时得到的函数便是偏函数。
 * Scala中使用关键字lazy来定义惰性变量，实现延迟加载(懒加载)。 惰性变量只能是不可变变量，并且只有在调用惰性变量时，才会去实例化这个变量。
 */
object HelloPartitionFunc {
  def main(args: Array[String]): Unit = {
    val sample = 1 to 8
    val isEven:PartialFunction[Int,Unit]={
      case x if x % 2 ==0 => println(x)
    }
     val isEven1:PartialFunction[Int,String]={
      case x if x % 2 ==0 => x + " is even"
    }
    isEven(4)
    //val evenNumber = sample collect isEven1
    val evenNumber = sample.collect(isEven1)
    evenNumber.foreach(x=>println(x))

    val isOdd:PartialFunction[Int,String]={
      case x if x % 2 != 0 => x + " is odd"
    }

    val numbers = sample map (isEven1 orElse isOdd)
    numbers.foreach(println)
  }
}
```
2.异常,lazy
```
package com.spark

import java.io.IOException

object HelloExceptionAndLazy {
  def main(args: Array[String]): Unit = {
    try{
     lazy  val score=100
      println(score)
     1/0
    }catch{
      case ioexception:IOException => println("IOException " + ioexception.toString())
      case illegalArgument:IllegalArgumentException => println("IllegalArgumentException "+ illegalArgument.toString())
      case arithmeticException:ArithmeticException => println("ArithmeticException "+arithmeticException.toString())
    }
  }
}
```

---
title: 柯里化
date: 2017-03-30 00:54:59
tags: Scala
toc: true
categories: 大数据技术
---
## 概念 ##
柯里化(Currying)指的是将原来接受两个参数的函数变成新的接受一个参数的函数的过程。新的函数返回一个以原有第二个参数为参数的函数。
定义一个函数
```
def add(x:Int,y:Int)=x+y
```
调用时：add(1,2)
将函数变形为：
```
def add(x:Int)(y:Int) = x + y
```
调用时：add(1)(2)
以上的过程就是柯里化
<!-- more -->
## 实现过程 ##
add(1)(2) 实际上是依次调用两个普通函数（非柯里化函数），第一次调用使用一个参数 x，返回一个函数类型的值，第二次使用参数y调用这个函数类型的值。
实质上最先演变成这样一个方法：
```
def add(x:Int)=(y:Int)=>x+y
// 接收一个x为参数，返回一个匿名函数，该匿名函数的定义是：接收一个Int型参数y，函数体为x+y
```
调用
```
val result = add(1)
//返回一个result，那result的值一个匿名函数：(y:Int)=>1+y
```
继续调用result
```
val sum = result(2)
```
## 实例代码 ##
```
package com.scala.base
/**
 * 柯里化
 */
object CurringTest extends App{
  def add(x:Int,y:Int)=x+y
  def add1(x:Int)(y:Int) = x + y
  println(add(1,2))
  println(add1(1)(2))
}
```

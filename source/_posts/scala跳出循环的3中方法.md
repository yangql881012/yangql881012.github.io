---
title: scala跳出循环的3中方法
date: 2017-03-29 11:54:59
tags: 跳出循环
categories: scala
---
Java里经常会用到continue和break，Scala里并没有这俩个语法。介绍三种跳出循环的方式,使用boolean变量、使用return、使用Breaks对象的break方法
<!-- more -->
## 使用boolean变量 ##
while代码:
```
//while代码
 var result:Int=0
 var flag:Boolean=true
 while(flag){
   println("output: " + result)
   result += 1
   if(result==5){
     flag=false
   }
 }
//output: 0
//output: 1
//output: 2
//output: 3
//output: 4
```
for代码：
```
//for 代码
var result:Int=0
var flag:Boolean=true
for(i<-0 until 10 if flag){
  println("output: " + i)
  if(i==5){
    flag=false
  }
}
//output: 0
//output: 1
//output: 2
//output: 3
//output: 4
//output: 5
```
## 使用return ##
while代码：
```
var x:Int=10
while(x >= 0){
  println("output:"+x)
  x-=1
  if(x == 5)
    return
}
/*output:10
output:9
output:8
output:7
output:6*/
```
for代码:
```
for(x <- 0 until 10){
  println("output : "+ x)
  if(x==5) return
}
/*output : 0
output : 1
output : 2
output : 3
output : 4
output : 5*/
```
##  使用Breaks对象的break方法 ##
while代码
```
import scala.util.control.Breaks._
var x=10
breakable(
while(x>0){
  println("output: " + x)
  x-=1
  if(x==5) break()
}    
)
/*output: 10
output: 9
output: 8
output: 7
output: 6*/
```
for代码：
```
import scala.util.control.Breaks._
breakable(
for(x <- 0 until 10){
  println("output:"+x)
  if(x==5) break()
}
)
/*   output:0
output:1
output:2
output:3
output:4
output:5*/
```

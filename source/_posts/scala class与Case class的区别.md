---
title: class与Case class的区别
date: 2017-03-03 17:54:59
tags: case class
categories: scala
---
在Scala中存在case class，它其实就是一个普通的class。但是它又和普通的class略有区别，如下：
- 初始化的时候可以不用new，当然你也可以加上，普通类一定需要加new
- toString的实现更漂亮
- 默认实现了equals 和 hashCode
- 默认是可以序列化的，也就是实现了Serializable
- 自动从scala.Product中继承一些函数
- case class构造函数的参数是public级别的，我们可以直接访问
- 支持模式匹配

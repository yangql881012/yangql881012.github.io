---
title: Scala的Symbol类型
date: 2017-03-29 20:54:59
tags: Scala
toc: true
categories: 大数据技术
---
相比较于String类型，Symbol类型有两个比较明显的特点:节省内存和快速比较
## String的intern ##
String类内部维护一个字符串池(strings pool)，当调用String的intern()方法时，如果字符串池中已经存在该字符串，则直接返回池中字符串引用，如果不存在，则将该字符串添加到池中，并返回该字符串对象的引用。执行过intern()方法的字符串，我们就说这个字符串被拘禁了(interned)。默认情况下，代码中的字符串字面量和字符串常量值都是被拘禁的，例如：
```
String s1 = "abc";
String s2 =new String("abc");
//返回true
System.out.println(s1 == s2.intern());
```
同值字符串的intern()方法返回的引用都相同，例如：
```
String s2 = new String("abc");
String s3 = new String("abc");

//返回true
System.out.println(s2.intern() == s3.intern());
//返回false
System.out.println(s2 == s3);
```
<!-- more -->
## 节省内存 ##
在Scala中，Symbol类型的对象是被拘禁的(interned)，任意的同名symbols都指向同一个Symbol对象，避免了因冗余而造成的内存开销。而对于String类型，只有编译时确定的字符串是被拘禁的(interned)。
```
val s='test
println(s=='test)
//output true
println(s==Symbol("test"))
//output true
```
## 快速比较 ##
由于Symbol类型的对象是被拘禁的(interned)，任意的同名symbols都指向同一个Symbol对象，而不同名的symbols一定指向不同的Symbol对象，所以symbols对象之间可以使用操作符==快速地进行相等性比较，常数时间内便可以完成，而字符串的equals方法需要逐个字符比较两个字符串，执行时间取决于两个字符串的长度，速度很慢。(实际上，String.equals方法会先比较引用是否相同，但是在运行时产生的字符串对象，引用一般是不同的)
## Symbol 应用 ##
Symbol类型一般用于快速比较，例如用于Map类型：Map<Symbol, Data>,根据一个Symbol对象，可以快速查询相应的Data, 而Map<String, Data>的查询效率则低很多。
实例代码
```
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
```
s"$title  ${f.name}   ${f.age}"：其中s是一个函数
```
val name="Tim"
println(s"$name")
//output Tim
println(s"1+1=${1+1}")
//output 1+1=2
```

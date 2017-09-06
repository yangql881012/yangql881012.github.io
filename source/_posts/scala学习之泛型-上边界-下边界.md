---
title: Scala学习之泛型-上边界-下边界
date: 2017-02-10 16:54:59
tags: Scala
toc: true
categories: 大数据技术
---
1.Scala的类，函数，方法都可以是泛型  
2.上边界 "<:"表达了泛型的类型必须是某种类型或者其子类，是对类型的限定   
3.下边界 ">:"表达了泛型的类型必须是某种类型或者其父类，是对类型的限定   
4.View bound:把类型转换为目标类型，是上边界和下边界的加强版本 T <% ,这个代码所表达的T必须是writable类型的，
但是T没有直接继承Writable接口，此时需要通过“implicit”的方式来实现这个功能。implicit def dog2Person(dog:Dog)=new Person(dog.name)  
5.T：ClassTag：这也是一种类型转换系统，只是在编译时类型信息不够，需要借助JVM的runtime来通过运行时获得完整的类型信息。  
6.逆变协变：-T +T  
7.covariant 协变:使你能够使用比原始指定的类型的子类,当我们定义一个协变类型 List[A+] 时，List[Child]可以是List[Parent]的子类型。  
8.Contravariance 逆变:使你能够使用比原始指定的类型的父类。当我们定义一个逆变类型 List[-A] 时，List[Child]可以是List[Parent]的父类型。  
9.context bound,T:Ordering这种语法必须能够变成Ordering T格式  
10.Invariance 不变。你只能使用原始指定的类型，不能协变和逆变  
<!-- more -->
```
class Animal[T](val species:T){
  def getAnimal(specie:T):T=species
}

//定义一个Person类
class Person(val name:String){
  def talk(person:Person){
    println(this.name + " " + person.name)
  }
}

//Worker类
class Worker(name:String) extends Person(name)

//Club类 上边界限定只能是Person类
class Club[T <: Person](p1:T,p2:T){
  def communication = p1.talk(p2)
}

//Club类 view bound <%
class Club1[T <% Person](p1:T,p2:T){
  def communication = p1.talk(p2)
}

class Dog(val name:String) extends Animal

class Enginer

class Expert extends Enginer

//逆变
class Meeting[-T]
//协变
class Meeting1[+T]
//context bound
class Maximum[T:Ordering](val x:T,val y:T){
  def bigger(implicit ord:Ordering[T])={
    if(ord.compare(x, y)  >  0) x else y
  }
}

object HelloTypeSystem {
  def main(args: Array[String]): Unit = {
    //将Dog类型转换为Person类型
    implicit def dog2Person(dog:Dog)=new Person(dog.name)
    val p = new Person("Tom")
    val w=new Worker("Mary")
    val dog=new Dog("daHuang")
    new Club(p,w).communication
    //view bound
    new Club1[Person](w,dog).communication
    val e = new Meeting[Enginer]
    val export=new Meeting[Expert]
    participateMeet(e)
    participateMeet(export)
    println(new Maximum(3,5).bigger)
  }

  def participateMeet(meeting:Meeting[Expert]){
    println("welcome")
  }

}
```

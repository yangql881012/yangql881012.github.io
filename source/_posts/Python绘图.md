---
title: python绘图
date: 2017/06/04 14:52:31
tags: Python
toc: true
categories: Python
---
python matplotlib,numpy,生成随机数，绘制折线图，散点图，直方图，子图
<!-- more -->

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# python data import
"""
@author:yangql
@create:2017-06-01 16:55
折线图，散点图[plot]，
"""
import  matplotlib.pylab as pyl
import numpy as npy
#x轴数据
x=[1,2,3,4,8]
#y轴数据
y=[5,7,6,8,9]
#折线图
#pyl.plot(x,y)#plot(X轴，Y轴，形式)
#display
#pyl.show()

#散点图
#pyl.plot(x,y,'o')#plot(X轴，Y轴，形式)
#pyl.show()
#改变颜色
'''
c-cyan:青色
r-red
m-magente
g-green
b-blue
y-yellow
b-black
w-white
'''
#pyl.plot(x,y,'oy')#plot(X轴，Y轴，形式)
#pyl.show()
#线条样式
'''
-:直线
--：虚线
-. -.形式
: 细小虚线
'''
#pyl.plot(x,y,':')#plot(X轴，Y轴，形式)
#pyl.show()
'''
点的样式
s：方形
h:六角形
H:六角形
*：星型
+：加号形式
x:x形式
d/D:菱形
p：五角行形
'''
#pyl.plot(x,y)#plot(X轴，Y轴，形式)
#同一区域，绘制多条线段
#x2=[1,3,8,9,10]
#y2=[4,7,8,9,1]
#pyl.plot(x2,y2)
#加标题,x轴，Y轴标题
#pyl.title("show")
#pyl.xlabel("age")
#pyl.ylabel("weight")
#设置X轴Y轴范围
#pyl.xlim(0,20)
#pyl.ylim(0,30)
#pyl.show()

#随机数生成
data=npy.random.random_integers(1,100,1000)
print(data)#random_integers 最小值，最大值，个数
#具有正态分布的随机数
normal_data=npy.random.normal(10,1,1000)#均数，西格玛，个数
print(normal_data)
#直方图[hist]
#pyl.hist(normal_data)
#pyl.show()
#设置宽度
#取消轮廓histtype='stepfilled'
#sty=npy.arange(2,19,4)
#pyl.hist(normal_data,sty,histtype='stepfilled')
#pyl.show()

#绘制子图
#pyl.subplot(2,2,3)#行,列，当前区域
#pyl.show()
pyl.subplot(2,2,1)
x=[11,2,3,4,8]
y=[5,47,6,8,9]
pyl.plot(x,y)
pyl.subplot(2,2,2)
x=[1,2,43,4,8]
y=[5,7,63,8,9]
pyl.plot(x,y)
pyl.subplot(2,1,2)
x=[1,42,3,4,8]
y=[5,7,46,8,9]
pyl.plot(x,y)
pyl.show()

```

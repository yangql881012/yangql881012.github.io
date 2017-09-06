---
title: python数据导入
date: 2017/06/01 14:52:31
tags: Python
toc: true
categories: Python
---
python数据导入，主要用pandas package实现，可以导入以下不同类型的数据：
- pda.read_csv ：CSV格式
- pda.read_excel：Excel格式
- pda.read_sql：mysql格式
- =pda.read_html：html格式
- read_table：文本格式
<!-- more -->

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# python data import
"""
@author:yangql
@create:2017-06-01 16:55
import data by pandas
"""
import  pandas as pda

# import data by csv
csv_data=pda.read_csv("F:/yangql/pythonworkplace/college-man.csv",encoding="gbk")
#print(csv_data.describe())

#print(csv_data.sort_values(by="INCOME_FAMILY",ascending=False))

#import data by xls
xls_data=pda.read_excel(u"F:/yangql/pythonworkplace/大学-男.xlsx",encoding='gbk')

#import data by mysql
import pymysql
conn=pymysql.connect(host='192.168.1.231',port=3306,user='root',password='root123',database='hive')
sql='select * from TBLS'
mysql_data=pda.read_sql(sql,conn)
#print(mysql_data)

#import data by html
#pip install beautifulsoup4
#pip install html5lib
#just read table
html_data=pda.read_html("https://book.douban.com/")
#print(html_data)

#import data by text
text_data=pda.read_table("F:/yangql/pythonworkplace/manifest.json")
print(text_data)
```

---
title: Python操作Hive数据
date: 2017/05/30 6:52:31
tags: Hive
toc: true
categories: 大数据技术
---
python操作hive，此处选用pyhive package。悲催的是Windows上不能安装后不能连接，一直报错（暂时没有解决办法）
```
File "D:\python\lib\site-packages\thrift_sasl-0.2.1-py3.6.egg\thrift_sasl\__init__.py", line 79, in open
thrift.transport.TTransport.TTransportException: Could not start SASL: b'Error in sasl_client_start (-4) SASL(-4): no mechanism available: Unable to find a callback: 2'
```
以下的操作在CentOS下亲测可以实现：  
Python：3.6.1  
Hive：hive-2.1.1  
1.安装
```
pip install pyhive
http://www.lfd.uci.edu/~gohlke/pythonlibs/vu0h7y4r/sasl-0.2.1-cp36-cp36m-win_amd64.whl
pip install sasl-0.2.1-cp36-cp36m-win_amd64.whl
pip install ply
pip install thrift-sasl
```
<!-- more -->
2.实例代码  
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# hive util with hive server2
"""
@author:yangql
@create:2017-05-08 16:55
"""
from pyhive import hive
from TCLIService.ttypes import TOperationState

class hiveClient:
    def __init__(self, host, port=10000, username=None, database='default', auth='NONE',
                 configuration=None, kerberos_service_name=None, password=None):
        '''
        create connect to hive server2
        '''
        self.conn=hive.connect(host=host,port=port,database=database)

    def hiveExe(self,sql):
        '''
        :param sql:
        :return:
        '''
        cursor=self.conn.cursor()
        cursor.execute(sql)

    def hiveQuery(self,sql):
        cursor=self.conn.cursor()
        cursor.execute(sql)
        return  cursor.fetchall()

    def close(self):
        '''
        close the connection
        :return:
        '''
        self.conn.close()


if __name__ == "__main__":
    #print("this is for hive test")
    hiveClient=hiveClient(host="hadoop01",port=10000,database="crm")
    data=hiveClient.hiveQuery("select * from crm.crm_cus_com limit 10")

```

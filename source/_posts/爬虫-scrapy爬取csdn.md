---
title: 爬虫-scrapy爬取csdn
date: 2017/06/04 14:52:31
tags: Python
toc: true
categories: Python
---
## 1.创建项目 ##
```
cd F:\yangql\pythonworkplace>
scrapy startproject csdn
cd csdn
scrapy genspider csdn_crawler blog.csdn.net -t crawl
```



scrapy crawl csdn_crawler

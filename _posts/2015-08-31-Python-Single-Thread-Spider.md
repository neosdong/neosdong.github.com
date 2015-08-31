---
layout: post
title: Python简单爬虫
category : Python
tags : Python 爬虫

---

## 一、使用Requests网络请求库

### Mac安装

	sudo pip install requests

少用`easy_install`多用`pip`，因为前者只能安装不能卸载。

### 容易使用

一句话获取HTML源码

	#-*-coding:utf8-*-
	import requests
	r = requests.get('http://tieba.baidu.com/f?ie=utf-8&kw=python')
	print r.text
	
`r`是`Response`对象。`Response`包含HTTP协议的响应信息，例如有`status_code`，`headers`等成员变量。

	print(r.status_code)
	print(r.headers['content-type'])


## 二、修改HTTP头获取源码

	#hea是我们自己构造的一个字典，里面保存了user-agent
	hea = {'User-Agent':'Mozilla/5.0 (Windows NT 6.3; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.118 Safari/537.36'}
	# html = requests.get('http://jp.tingroom.com/yuedu/yd300p/')
	html = requests.get('http://jp.tingroom.com/yuedu/yd300p/',headers = hea)
	# html.encoding = 'utf-8' #这一行是将编码转为utf-8否则中文会显示乱码。

`User-Agent`的内容可以在Chrome分析工具HTTP请求的'User-Agent'字段找到。

## 三、Request与正则表达式

使用Request获取网页源码，再使用正则表达式匹配出感兴趣的内容，这是简单爬虫的基本原理。

例如，

	chinese = re.findall('color: #039;">(.*?)</a>',html.text,re.S)
	for each in chinese:
    	print each

打印出符合条件的链接文字。

## 四、实战分析网页源码

步骤：

1. `Requests`获取网页
1. `reb.sub`换页
1. 正则表达式匹配内容

前面已经介绍过，`Requests.get`获取HTML源码。

`reb.sub`用格式化字符串的方式，组合URL。参考代码如下

 
	#changepage用来生产不同页数的链接
    def changepage(self,url,total_page):
        now_page = int(re.search('pageNum=(\d+)',url,re.S).group(1))
        page_group = []
        for i in range(now_page,total_page+1):
            link = re.sub('pageNum=\d+','pageNum=%s'%i,url,re.S)
            page_group.append(link)
        return page_group
        
正则的Python代码示例（将正则匹配到的结果存到一个对象中）

	#getinfo用来从每个课程块中提取出我们需要的信息
    def getinfo(self,eachclass):
        info = {}
        info['title'] = re.search('target="_blank">(.*?)</a>',eachclass,re.S).group(1)
        info['content'] = re.search('</h2><p>(.*?)</p>',eachclass,re.S).group(1)
        timeandlevel = re.findall('<em>(.*?)</em>',eachclass,re.S)
        info['classtime'] = timeandlevel[0]
        info['classlevel'] = timeandlevel[1]
        info['learnnum'] = re.search('"learn-number">(.*?)</em>',eachclass,re.S).group(1)
        return info
        
`re.S`表示匹配模式，通过Python源码`re.py`可知，表示一种匹配模式，不匹配newline。

	S = DOTALL = sre_compile.SRE_FLAG_DOTALL # make dot match newline	

## 总结

要实现实用的爬虫，还需要加入数据库存储，多线程等功能。

## 参考

[实战——极客学院课程爬虫-Python 单线程爬虫-极客学院](http://www.jikexueyuan.com/course/821_4.html?ss=1)
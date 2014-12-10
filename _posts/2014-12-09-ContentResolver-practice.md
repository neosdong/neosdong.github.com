---
layout: post
title: Android ContentResolver实践总结
category: program
tags: Android
keywords: example
description: 

---

ContentResolver

* 内容分解器


## 获取通讯录联系人信息

过程：

1. 示例化一个ContentResolver
2. 通过CONTENTURI获取手机联系人Cursor
3. 读取每一行对应的数值，保存到数组。

使用Adapter将数据显示到ListView

## ResourceCursorAdapter的bindView与BaseAdapter的getView对比
	
	public void bindView(View view, Context context, Cursor cursor);
	
	public View getView(int position, View convertView, ViewGroup parent);

两者都是MVC框架中的Control，关键区别可以从参数得知。
View是单项的View，从这角度看是一致的。
数据方面，**前者是通过Cursor解析数据，后者通过数组的位置获取数据。减少了部分转换工作。**

从源代码分析，
前者的继承自CursorAdapter，getView实现了BaseAdapter中的getView的应用。好吧！CursorAdapter也是继承自BaseAdapter的。所以，跟老大BaseAdapter多混，可以衍生很多的Adapter。



## 筛选数据：最近联系人、陌生人等

`getContentResolver().query`()最终也是转换成SQL操作。
这一点从抛出的异常可以看出来。

![](http://s1.5km.co/201410/2416/48002_z.png)

有空研究一下数据库就知道如何做删选条件了哦。

### 实现
参数`where`是实现筛选条件的参数。

	//定义cursor，可以调用系统的通话
    Cursor cursor = getContentResolver().query(CallLog.Calls.CONTENT_URI,null, where, null, order);
    
where参数的实例：

* `"name is not null"` 名字非空
* `"duration = 1"` 最近联系人
* `"name is null"` 陌生人

完整的DEMO:[Android-Training/DEMO_CallPhoneLogs](https://github.com/neosdong/Android-Training/tree/master/DEMO_CallPhoneLogs)

Log
* 20141031 start
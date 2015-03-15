---
layout: post
title: Android Weekly 04
category: Android
tags: Android Weekly
keywords: Android Weekly
description:
 
---

## 一、教程与实践

### Android学习之RecyclerView

links:[csdn](http://blog.csdn.net/le_go/article/details/36628161)

### 慕课网:Android图像处理

Android图像处理-打造美图秀秀从它开始 : [http://www.imooc.com/view/302](http://www.imooc.com/view/302)

1. 图像处理的基本知识和概念
2. Android中对图像的处理方法
3. 实际演示如何对图像进行各种效果的处理

### Sublime text 3安装smali插件视频教程

links:[极客学院](http://www.jikexueyuan.com/course/131.html)


## 二、开源项目

### Android Asynchronous Networking and Image Loading

一个功能很全面的网络请求库

* Asynchronously download:常用的网络请求内容都涵盖了，相对Volley还多文件下载。（Volley支持小而频繁的网络请求）
  * Images into ImageViews or Bitmaps (animated GIFs supported too)
  * JSON (via Gson)
  * Strings
  * Files
  * Java types using Gson
* Easy to use Fluent API designed for Android
  * Automatically cancels operations when the calling Activity finishes 当Activity结束时，自动结束操作
  * Manages invocation back onto the UI thread 管理进入UI线程的回调
  * All operations return a Future and can be cancelled 所有操作返回一个Future对象，并且可以被取消。Future对象是多重继承于`Cancellable, java.util.concurrent.Future`的接口。用来定义回调对象。
  * 【文档】[Future | Android Developers](http://developer.android.com/reference/java/util/concurrent/Future.html)
  * 【了解接口多重继承】[JAVA 单继承 与 接口 多重继承 - - ITeye技术网站](http://talentluke.iteye.com/blog/1827258)
* HTTP POST/PUT:
  * text/plain
  * application/json - both JsonObject and POJO
  * application/x-www-form-urlencoded
  * multipart/form-data 可以做分片上传文件，也可做进度回调
* Transparent usage of HTTP features and optimizations:
  * SPDY and HTTP/2
  * Caching
  * Gzip/Deflate Compression
  * Connection pooling/reuse via HTTP Connection: keep-alive
  * Uses the best/stablest connection from a server if it has multiple IP addresses
  * Cookies
* View received headers
* Grouping and cancellation of requests
* Download progress callbacks 下载进度回调
* Supports file:/, http(s):/, and content:/ URIs
* Request level logging and profiling
* Support for proxy servers like Charles Proxy to do request analysis
* Based on NIO and AndroidAsync
* Ability to use self signed SSL certificates

[github地址](https://github.com/koush/ion)

### UltimateRecyclerView

一个多功能的RecyclerView，包括了下拉刷新、加载更多，滑动删除，拖拽排序、多种动画、视差拖动、Toolbar渐变、Toolbar和FAB随着滚动出现消失等等效果，都可以放在同一个RecyclerVIew中并自由配置。

[github](https://github.com/cymcsg/UltimateRecyclerView)

### UltimateAndroid rapid development framework
UltimateAndroid is a rapid development framework for developing your apps

[github](http://cymcsg.github.io/UltimateAndroid)

### 一个简单容易使用的对话框——DialogPlus

toast容易有变兼容的情况，Dialog比较容易样式化，这是一个MD风格化的一个开源控件。

![](http://ww3.sinaimg.cn/bmiddle/9484c7d3tw1epz9ajkzunj20lc0zkjtr.jpg)

### PullDownListView仿微信下拉眼睛动画效果

![](http://ww4.sinaimg.cn/bmiddle/005ZJ8j4gw1eq0imaz1xng30aa0gcqv6.gif)

[github](https://github.com/guojunyi/PullDownListView)

<!--

### 扫扫图书

![](http://ww4.sinaimg.cn/thumbnail/8c7a19d3gw1eq0r7av0rgj20u01hc16z.jpg)

links:[github](https://github.com/JayFang1993/ScanBook)

功能

1. 扫码查图书信息  

实现 

1. 使用ZXing扫码条码，返回isbn码，通过intent传给BookViewActivity。BookViewActivity发送get请求豆瓣API取得图书信息。


Bug

1. 搜索图书的结果ListView图片错位闪烁
2. 加载了360的包，但是悬浮推广控件的出现

-->

## 三、资料和文章

### 开源项目—Android 面试题集锦及解答

大家可以在[这里](https://github.com/android-cn/interview-questions/blob/master/README.md)

1. 在 Issues 中分享面试相关资料或遇到的问题及解决方案。Title 以[分享]开头。
2. 在 Issues 中提问问题，并@ 其他人 帮忙解决。Title 以[问答]开头。
3. 逛逛 Issues 了解或解答下其他人的提问，精华可回复手动点赞，Watch接受最新 Issue 通知。



### 每个Android开发者必须知道的内存管理知识

[via](http://www.codeceo.com/article/android-memory-manage.html)

内存泄露导致OOM是大多数Android开发者会遇到的问题。作者整理了一些常用的内存管理知识，值得在项目中尝试与实践。

内存泄漏：对象在内存heap堆中中分配的空间，当不再使用或没有引用指向的情况下，仍不能被GC正常回收的情况。多数出现在不合理的编码情况下，比如在 Activity中注册了一个广播接收器，但是在页面关闭的时候进行unRegister，就会出现内存溢出的现象。通常情况下，大量的内存泄漏会造成 OOM。

OOM：即OutOfMemoery，顾名思义就是指内存溢出了。内存溢出是指APP向系统申请超过最大阀值的内存请求，系统不会再分配多余的空间，就会造成OOM error。在我们Android平台下，多数情况是出现在图片不当处理加载的时候。

### Android Proguard 详解

ProGuard是一个混淆代码的开源项目。它的主要作用就是混淆，当然它还能对字节码进行缩减体积、优化等。

links:[官网](http://proguard.sourceforge.net/)

### 专栏：50个Android开发技巧

Android开发中经常遇到但会被忽视的开发技巧

![](http://avatar.csdn.net/blogpic/20140426020422421.jpg)

links:[csdn](http://blog.csdn.net/column/details/androidhacks.html)

### 收集android上开源的酷炫的交互动画和视觉效果

[Rano1/Interactive-animation](https://github.com/Rano1/Interactive-animation/blob/master/README.md)
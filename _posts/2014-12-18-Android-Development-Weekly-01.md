---
layout: post
title: Android Development Weekly 01
category: Android
tags: Android
keywords: 
description:
 
---


## 一、教程

### 精通Android触摸系统

@Ocean-藏心
精通Android触摸系统(中英字幕) 
[Mastering the Android Touch System](http://t.cn/RzWuVVR) 
历经3周的翻译,1小时18分钟的视频,终于完成了. 

简介:[Tutorial Enhancing Android UI with Custom Views(字幕调整)](http://t.cn/RzxZs6E)的姊妹篇，详细讲解了Android自定义事件处理的方方面面。配合guolin 大神CSDN的博客和上一个视频，让你对开发Android 自定义控件游刃有余。

### Andrid Studio with Gradle

1、[史上最详细的Android Studio系列教程一--下载和安装](http://segmentfault.com/blog/stormzhang/1190000002401964)

[Gradle Plugin User Guide](http://tools.android.com/tech-docs/new-build-system/user-guide) 来自Google的Gradle用户指南。


Android Studio Tips  [StudioDevTips@Mark_Hwang](http://www.weibo.com/p/10080860ab750855d136bb6bbd0c2d348cceab?k=StudioDevTips&from=501&_from_=huati_topic) 

> 第一条 Android Studio 特性介绍：Studio 内置了 .9(点九)图编辑器，可以直接在Studio里面添加或修改.9图。1）如果图片是普通图片，将图片名称直接加“.9”后缀，比如ic_menu_save.9.png 后，导入Studio，双击打开，即可修改。2）如果本身就是.9图片，直接导入双击打开即可。

### UI呈现的优化

[加快activity显示速度，提高用户体验](http://www.imooc.com/wenda/detail/240437) 

* onCreate还没有显示给用户，那么那段黑屏可以确定就是onCreate的时间太长了。
* onResume可以分担部分View的加载。记得加上做标识的Feild变量，避免每次进来都加载。
* 代码框架：见原文。

## 二、开源项目

[litesuits/android-common](https://github.com/litesuits/android-common) 

通用性强，纯洁简单，体积不到50K！其中包括shell命令，静默安装，bitmap处理，文件操作，加密存储器，计数器，均值器，吐司，日志，校验，提示，网络监测等基础功能，以及一些Base64、MD5、Hex、Byte、Number、Dialog、Filed、Class、Package、Telephone、Random等工具类。

[xesam/AndroidInject](http://git.oschina.net/xesam/AndroidInject)

一个简单的 androd 资源注入工具，只有一个类，方便拷贝，用来在平时学习中简化一些操作，减少重复代码量，因此，只适合学习工程

如果需要在工作中使用，可以参考更严谨的视图注入框架 butterknife

## 三、资源

* 快速搭配Material Design主题颜色的并且提供下载的在线网站 [materialpalette](http://www.materialpalette.com/)


## 四、Java

### String,StringBuffer,StringBuilder的区别
String对象是不可变的，在说原因的时候没说清，其实看看String源码就知道了
在new String的时候，String 中的3个成员变量value，count，offset都是final的，当然String类也是final的，所以一旦初始化后不能修改的。

* StringBuffer，与StringBuilder都实现了相同的接口，而且都继承相同的父类。
* 不同的是，StringBuffer的大部分方法都是同步的，所以是线程安全。
* StringBuilder没有同步，所以通常情况下效率上StringBuilder是优于StringBuffer的。

StringBuffer与StringBuilder随着append会扩大value[]的容量，这里具体做法是使用System类中的arraycopy方法拷贝，这个方法是调用底层本地方法来处理的，类似于直接使用C的指针操作，比较快。

[之前](/2014/09/17/String,StringBuffer与StringBuilder对比.html)也了解过，有时间可以看看源码。


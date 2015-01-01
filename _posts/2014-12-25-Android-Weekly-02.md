---
layout: post
title: Android Weekly 02
category: Android
tags: Android Weekly
keywords: Android Weekly
description:
 
---

## 一、教程与实践

### Java语言

[Java 入门第二季](http://www.imooc.com/learn/124) 

讲解Java的面向对象：多态、抽象类、接口等。

#### XML文件读取

[Java 眼中的 XML---文件读取](http://www.imooc.com/view/171?utm_source=jobboleweibo)

* XML格式的用途、优点
* DOM方式解析XML:将XML文件完全读入内存，形成DOM树。通过对象形式访问。PC浏览器使用的就是DOM解析方式。
* SAX方式解析XML:逐条语句解析，Handler根据语句的属性进行响应处理。
* DOM4J以及JDOM方式解析XML
* 四种XML解析方式比较

##### DOM
![](https://raw.githubusercontent.com/neosdong/neosdong.github.com/master/img/20141231-XML-DOM.png)

##### SAX
![](https://raw.githubusercontent.com/neosdong/neosdong.github.com/master/img/20141231-XML-SAX.png)

### UI

#### UI设计

##### Android应用中的导航设计

原则：

1、导航靠左，操作靠右。
2、注意导航层级。例如，Tab导航层级与SlidingMenu的关系。
3、简化应用结构，减少层级。如果层级很多，可以用折叠方式的菜单，甚至单独做个次级导航页。
4、滑动是大屏时代的标配。


Android不是用来导航的，是用来放操作的。Action Overflow少用，因为Android 4.0之后用去掉了默认的菜单键会导致问题([link](http://blog.csdn.net/sunmc1204953974/article/details/40617793))。

See also:
[Android 应用中十大导航设计错误](http://www.geekpark.net/topics/199244) 

#### UI组件

Android的TextView.setError()，可以显示错误信息，效果如图所示。参考：[http://t.cn/RhipiNd](http://t.cn/RhipiNd)

[Best Practices for Android User Interface -RapidValue Solutions](http://www.rapidvaluesolutions.com/tech_blog/best-practices-for-android-user-interface/) 



## 二、开源项目

### UI

#### Tab and ViewPager

[JakeWharton/Android-ViewPagerIndicator](https://github.com/JakeWharton/Android-ViewPagerIndicator) 

可取代ActionBar.Tab等的开源UI控件。一些继承自HorizontalSrollView的自定义View。

与ViewPager公用一个adapter。调用一次`notifyDataSetChanged`即可同时实现Tab和ViewPager的刷新。更多细节问题，参考：android - ViewPager PagerAdapter not updating the View - Stack Overflow : [http://stackoverflow.com/questions/7263291/viewpager-pageradapter-not-updating-the-view](http://stackoverflow.com/questions/7263291/viewpager-pageradapter-not-updating-the-view
)


#### DraggableGridView

[DraggableGridView](https://github.com/thquinn/DraggableGridView)

askerov/DynamicGrid : https://github.com/askerov/DynamicGrid

#### Google Now 风格 Pull To Refresh

[mariotaku/RefreshNow](https://github.com/mariotaku/RefreshNow) 

#### ShowcaseView 

用于APP第一次运行使用的高亮引导效果。

支持Gradle

`compile 'com.github.amlcurran.showcaseview:library:5.0.0'`



#### 高仿【优酷】圆盘旋转菜单的实现

[Android 高仿【优酷】圆盘旋转菜单的实现（附代码）](http://www.apkbus.com/android-65243-1-1.html)

### Network

#### Volley



[Android Volley完全解析(一)，初识Volley的基本用法 - 郭霖的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/guolin_blog/article/details/17482095)

[Comparison of Android Networking Libraries: OkHTTP, Retrofit, Volley - Stack Overflow](http://stackoverflow.com/questions/16902716/comparison-of-android-networking-libraries-okhttp-retrofit-volley/16903205#16903205) 

### 其他UI开源项目

#### Instagram/ig-json-parser

[Instagram/ig-json-parser](https://github.com/Instagram/ig-json-parser)

JSON序列化库ig-json-parser，这是Intagram团队开发的一个快速高效的json库，此库会自动生成解析代码从而达到快速高效的目的

#### gesture-imageview

[jasonpolites/gesture-imageview](https://github.com/jasonpolites/gesture-imageview)

#### 十大Material Design开源项目

直接拿来用！十大Material Design开源项目-CSDN.NET : http://www.csdn.net/article/2014-11-21/2822753-material-design-libs/1

#### 仿QQ 5.0 气泡提示 拖动爆炸消除

Android 仿QQ 5.0 气泡提示 拖动爆炸消除-Android源码下载-eoe 移动开发者论坛 - Powered by Discuz! : http://www.eoeandroid.com/thread-541767-1-1.html






## 三、资源工具

### 架构

Android四款系统架构工具 - 其他 - Android开发论坛 - 安卓开发论坛 - Android开发 - 安卓论坛 - 移动互联网门户 - Powered by Discuz! : [http://www.apkbus.com/android-5994-1.html](http://www.apkbus.com/android-5994-1.html)

### 文档

##### 安卓生命周期

[xxv/android-lifecycle](https://github.com/xxv/android-lifecycle/tree/master/tools/LifecycleLog
) 
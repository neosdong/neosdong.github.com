---
layout: post
title: 说说Android开发的学习
category: program
tags: Android
keywords: example
description: 
---
![20141210-android-hacking](http://neosdong.github.io/img/20141210-android-hacking)

最近专注于学习Android开放，但是学习过程总是零零散散的。不能一下子总结出一套套的经验思路。但是，不写吧，就一段时间都不写了。所以还是坚持，每天一小步，几个月一大跨越吧。

参考如何漫画[『21天学会C++』](http://ww2.sinaimg.cn/bmiddle/795bf814jw1em7a53e009j20yi6qkqm0.jpg)

漫画『21天学会C++』很讽刺，但是，也说明一个现实。没有快速GET的绝技，但是，坚持下去就有无限可能。

## 一、Android学习阶段

参照上图，总结一下Android学习的几个阶段：

1. 【Java语言基础】
2. 【Android基础知识点】练习制作对应[DEMO](https://github.com/neosdong/Android-Training)
3. 【模仿】常见的网上有源码的Android应用：
	1. [新闻客户端](https://github.com/neosdong/DimensionNews)
	2. IM：聊天机器人、仿微信界面等
	3. 普通2D游戏（俄罗斯方块）
	4. 词典
4. 【分析，造轮子】分析优秀应用的功能实现，并模仿实现。例如，[QQ5.0的菜单](https://github.com/neosdong/Android-Training/tree/master/UI_QQ50_SlidingMenu)
5. 【设计自己的应用】

以上阶段都不是一次过的，达到独立开发的水平之后还要经常练习巩固之前的阶段。

## 二、养成良好习惯：

1. **【用纸笔思考】**多用纸笔做草稿，形成思路才开始写代码。一头冲进代码里面很容易迷失。草稿写什么呢？
	* 模块分析：分析APP或者开源组件源码的整体实现，可以画一下类UML的草稿，从功能实现的"过程"出发，不断添加内容。这样就容易读懂别人的代码。包括自己想实现一些功能也可以按此思路构思。
	* 过程分析：画流程图。
	* 以上也是从《程序员的思维修炼》学来的：练习L->R->L的思考过程。
2. **【一段专注时间专注一件事】**
	* 搭框架：写代码是抽象到具体的过程，先搭好框架，需要补充的细节用函数和Log代替。每次解决一个问题。
	* 使用Git：用Git的过程也很好地培养了这一习惯。Git每次commit一批代码，并且这批代码都是为特定问题写的。对以后review也有好处。
3. **【坚持锻炼身体】**身体才是革命的本钱。
 
好吧！本来是想整理一下自定义View的。下次吧。
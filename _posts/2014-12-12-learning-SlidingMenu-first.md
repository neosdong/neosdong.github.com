---
layout: post
title: SlidingMenu简析
category: Android
tags: Android
keywords: 
description: 
---

![](https://raw.githubusercontent.com/neosdong/neosdong.github.com/master/img/20141212-slidingMenu-lib-app.png)

## UML of Sliding's Activity

SlidingMenu的lib.app包定义了一些可被继承的Activity类：
* SlidingActivity
* SlidingListActivity
* SlidingFragmentActivity
* SlidingPreferenceActivity

以上的Activity实现了接口`SlidingActivityBase`，接口的方法实现都是调用SlidingActivityHelper的。

### SlidingActivityBase

* setBehindContentView():设置被隐藏的View，菜单的View。
* getSlidingMenu():获取SlidingMenu。用于做初始化配置，启动。
* toggle()
* showContent()
* showMenu()
* showSecondaryMenu()
* setSlidingActionBarEnabled():滑动的时候，ActionBar是否跟着一起动。

为什么不包含registerAboveContentView?这是Activity的View，重写Activity.setContentView会调用。

## SlidingFragmentActivity实现分析

步骤一：`SlidingFragmentActivity`继承`FragmentActivity`实现了接口`SlidingActivityBase`

	public class SlidingFragmentActivity extends FragmentActivity implements SlidingActivityBase {...}

步骤二：使用`SlidingActivityHelper`的方法协助实现`SlidingActivityBase`接口的函数。

### SlidingFragmentActivity

以接口为框架，重写了FragmentActivity的一些方法：

* onCreate:
	* 调用父类Activity.onCreate 
	* 初始化`SlidingActivityHelper`的实例`mHelper`
	* 调用mHelper.onCreate方法
		* 实例化mSlidingMenu:通过LayoutInflater获取**布局文件**。
* onPostCreate:
	* 调用父类Activity.onPostCreate方法:运行在ononRestoreInstanceState和onStart之后
	* 调用mHelper.onPostCreate
		* 更深度的初始化：
			* 检查mViewBehind和mViewAbove是否null
			* mSlidingMenu.attachToActivity:TODO
			* 读取savedInstanceState，初始化menu的开启关闭状态。注意的是这里也用了handler.post一个Runnable启动动作。
* findViewById
	* 调用父类方法Activity.findViewById
	* 检查find得的view是否为null
	* 如果为null，mHepler则从SlidingMenu的布局文件中找下。（这是基于什么逻辑？）
* onSaveInstanceState
	* SlidingMenu也要保存数据
* setContentView
	* `mHelper.registerAboveContentView`注意，是注册。注册了仅仅用来检查内容View是否为空。
* setBehindContentView
	* 设置菜单View
* getSlidingMenu:
	* 通过`mHelper.getSlingMenu`获取SlidingMenu对象
* toggle
* showContent
* showMenu
* showSecondaryMenu
* setSlidingActionBarEnabled:滑动的时候，ActionBar是否跟着一起动。
* onKeyUp:插入事件处理过程到Activity之前。
	* 调用mHelper.onKeyUp，控制菜单的开关。事件处理成功（菜单打开时press back），返回true，不再执行Activity的onKeyUp
	* 否则调用Activity的onKeyUp
	
## SlidingMenu的添加过程

    //设置content的布局
    //内部运行SlidingMenuHelper的registerAboveContentView方法
    super.setContentView(R.layout.menu_right);

    //菜单布局
    setBehindContentView(R.layout.menu_frame);

    //配置SlidingMenu
    getSlidingMenu().setSlidingEnabled(true);
    getSlidingMenu().setMode(SlidingMenu.LEFT);
    getSlidingMenu().setTouchModeAbove(SlidingMenu.TOUCHMODE_MARGIN);


    //设定Fragment
    if (savedInstanceState == null) {
        //initFragments();
        FragmentTransaction secondFragmentTransaction = getSupportFragmentManager()
                    .beginTransaction();
        secondFragmentTransaction
                    .replace(R.id.menu_frame, getMenuFragment(), LeftMenuFragment.class.getName());
        getSlidingMenu().showContent();
        secondFragmentTransaction.commit();
    }
    

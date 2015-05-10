---
layout: post
title: Android视图绘制流程完全解析
category: Android
tags: Android View
keywords: Android View
description: 

---

每一个视图的绘制过程都必须经历三个最主要的阶段，即onMeasure()、onLayout()和onDraw()，下面我们逐个对这三个阶段展开进行探讨。

## 一、onMeasure()

### onMeasure接收的参数

measure()方法接收两个参数，widthMeasureSpec和heightMeasureSpec，这两个值分别用于确定视图的宽度和高度的规格和大小。

#### widthMeasureSpec和heightMeasureSpec

MeasureSpec是复合数值。

MeasureSpec的值由specSize和specMode共同组成的，其中specSize记录的是大小，specMode记录的是规格。specMode一共有三种类型，如下所示：

##### EXACTLY

表示父视图希望子视图的大小应该是由specSize的值来决定的，系统默认会按照这个规则来设置子视图的大小，开发人员当然也可以按照自己的意愿设置成任意的大小。

XML布局文件中使用固定尺寸或者`match_parent`对应此模式。（在`ViewRoot.performTraversals()`调用的`getRootMeasureSpec`方法的源码中体现）

**specMode在制作流式布局曾用到。用于确定父View的最终尺寸是选择何种测量方式的结果。**

##### AT_MOST

表示子视图最多只能是specSize中指定的大小，开发人员应该尽可能小得去设置这个视图，并且保证不会超过specSize。系统默认会按照这个规则来设置子视图的大小，开发人员当然也可以按照自己的意愿设置成任意的大小。

XML布局文件中使用`wrap_content`对应此模式。

##### UNSPECIFIED

表示开发人员可以将视图按照自己的意愿设置成任意的大小，没有任何限制。这种情况比较少见，不太会用到。



#### widthMeasureSpec和heightMeasureSpec参数的传递



### 重写onMeasure

#### setMeasuredDimension()

>This method must be called by {@link #onMeasure(int, int)} to store the
     * measured width and measured height. Failing to do so will trigger an
     * exception at measurement time.

该方法必须调用，存储当前View的宽和高。否则会抛出异常。

		public class MyView extends View {   
    	......  
    	      
    	@Override  
    	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {  
        	setMeasuredDimension(200, 200);  
    	}  
  	}  
  	
这样的话就把View默认的测量流程覆盖掉了，不管在布局文件中定义MyView这个视图的大小是多少，最终在界面上显示的大小都将会是200*200。

需要注意的是，在setMeasuredDimension()方法调用之后，我们才能使用getMeasuredWidth()和getMeasuredHeight()来获取视图测量出的宽高，以此之前调用这两个方法得到的值都会是0。

#### super.onMeasure

在例如，要定义一个固定高两行的ListView。

	@Override
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
		int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2, MeasureSpec.AT_MOST);
		super.onMeasure(widthMeasureSpec, expandSpec);
	}

这里目的是改变测量模式和测量数值，无须改变ListView的测量算法。
所以，
调用的是父类ListView的onMeasure，直接改变ListView的测量参数。`super.onMeasure`里面会`setMeasuredDimension`。


#### onMeasure何时被调用？

系统调用onMeasure的前后：

1. `ViewRoot.performTraversals()`
	* `getRootMeasureSpec`获取widthMeasureSpec和heightMeasureSpec
2. 通常情况下，这两个值都是由父视图经过计算后传递给子视图的
3. View的`final measure()`调用onMeasure

#### onMeasure被调用次数

简单理解，

1. 【绘制顺序】布局的整体绘制过程是从布局的根节点开始，遍历树，这就意味着这个某视图会在其子视图之前绘制，并且会按照出现在树中的顺序绘制它们的兄弟姐妹。
2. 【绘制结果的反馈链条】注意重绘制过程是由某节点的父节点控制的。也是一个传递过程了，因为子节点太大或太小，父节点的尺寸也会过大或过小。

这个过程大概知道就行，我们**重写onMeasure最重要是`setMeasuredDimension()`的参数正确**。


详见：[How Android Draws Views - Android Developers](http://developer.android.com/guide/topics/ui/how-android-draws.html) 



## 二、onLayout()

### onLayout的作用

onMeasure之后运行onLayout，给视图进行布局。

### onLayout的参数

在layout()方法中，首先会调用setFrame()方法来判断视图的大小是否发生过变化，以确定有没有必要对当前的视图进行重绘，同时还会在这里把传递过来的四个参数分别赋值给mLeft、mTop、mRight和mBottom这几个变量。

    /**
     * Called from layout when this view should
     * assign a size and position to each of its children.
     *
     * Derived classes with children should override
     * this method and call layout on each of
     * their children.
     * @param changed This is a new size or position for this view
     * @param left Left position, relative to parent
     * @param top Top position, relative to parent
     * @param right Right position, relative to parent
     * @param bottom Bottom position, relative to parent
     */
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
    }

changed 来自setFrame的返回结果。在layout()方法中，首先会调用setFrame()方法来判断视图的大小是否发生过变化。


### 重写onLayout

重写onLayout的任务：设置childView的位置。

	childView.layout(0,0,childView.getMeasuredWidth(),childView.getMeasuredHeight());

#### getWidth()方法和getMeasureWidth()方法的区别

在onLayout()过程结束后，我们就可以调用getWidth()方法和getHeight()方法来获取视图的宽高了。说到这里，我相信很多朋友长久以来都会有一个疑问，getWidth()方法和getMeasureWidth()方法到底有什么区别呢？它们的值好像永远都是相同的。其实它们的值之所以会相同基本都是因为布局设计者的编码习惯非常好，实际上它们之间的差别还是挺大的。

首先getMeasureWidth()方法在measure()过程结束后就可以获取到了，而getWidth()方法要在layout()过程结束后才能获取到。另外，getMeasureWidth()方法中的值是通过setMeasuredDimension()方法来进行设置的，而getWidth()方法中的值则是通过视图右边的坐标减去左边的坐标计算出来的。

观察SimpleLayout中onLayout()方法的代码，这里给子视图的layout()方法传入的四个参数分别是0、0、childView.getMeasuredWidth()和childView.getMeasuredHeight()，因此getWidth()方法得到的值就是childView.getMeasuredWidth() - 0 = childView.getMeasuredWidth() ，所以此时getWidth()方法和getMeasuredWidth() 得到的值就是相同的，但如果你将onLayout()方法中的代码进行如下修改：

	@Override  
	protected void onLayout(boolean changed, int l, int t, int r, int b) {  
    	if (getChildCount() > 0) {  
        	View childView = getChildAt(0);  
        	childView.layout(0, 0, 200, 200);  
    	}  
	}  
	

这样getWidth()方法得到的值就是200 - 0 = 200，不会再和getMeasuredWidth()的值相同了。当然这种做法充分不尊重measure()过程计算出的结果，通常情况下是不推荐这么写的。getHeight()与getMeasureHeight()方法之间的关系同上，就不再重复分析了。


## 三、onDraw()



参考：Android视图绘制流程完全解析，带你一步步深入了解View(二) - 郭霖的专栏 - 博客频道 - CSDN.NET : http://blog.csdn.net/guolin_blog/article/details/16330267



<!--
Log
1. 20141224 Create,onMeasure
2. 20141225 onLayout
-->
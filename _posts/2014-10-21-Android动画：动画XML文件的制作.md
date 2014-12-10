---
layout: post
category : program
tags : Android 动画

---

![coordinate](http://s2.5km.co/201410/2416/47876_m.png)

动画就是改变图像显示的属性的过程。

* 图像属性包括：透明度、位置、角度
* 动画属性包括：alpha (透明变化)、translate(位置移动)、scale(缩放) 、rotate(旋转)

## 基本动画类型

根据动画过程中变化的属性可以有：

* translate动画，非常好理解，就是定义一个开始的位置和一个结束位置，定义移动时间，然后就能自动产生移动动画。
* alpha动作，透明度的前后值与时长的组合。

例如，向左滑动屏幕，
从右边进来的动画这样定义。res/anim/left_in.xml:

	
	<?xml version="1.0" encoding="utf-8"?>
	<set xmlns:android="http://schemas.android.com/apk/res/android" >

    	<translate
        	android:duration="500"
        	android:fromXDelta="100%p"
        	android:toXDelta="0" />

    	<alpha
        	android:duration="500"
        	android:fromAlpha="0.1"
        	android:toAlpha="1.0" />

	</set>

translate动作坐标的正负定义与高中数学的直角坐标一致（X轴向右为正，Y轴向上为正）。

## Translate动画的属性：位置、速度、延时

**开发者可以利用加速器、延时可以定义多个动画文件，按一定顺序运行，形成复杂的动作。**

* `android:interpolator`: 加速器，非常有用的属性，可以简单理解为动画的速度，可以是越来越快，也可以是越来越慢，或者是先快后忙，或者是均匀的速度等等。
* `android:duration`: 动画运行时间，定义在多长时间（ms）内完成动画
* `android:startOffset`: 延迟一定时间后运行动画
* `fromXDelta`: X轴方向开始位置，可以是%，也可以是具体的像素 具体见图
* `toXDelta`:   X轴方向结束位置，可以是%，也可以是具体的像素
* `fromYDelta`: Y轴方向开始位置，可以是%，也可以是具体的像素
* `toYDelta`:   Y轴方向结束位置，可以是%，也可以是具体的像素

`android:interpolator`: 加速器属性的可用值如下：

	@android:anim/accelerate_interpolator： 越来越快
	@android:anim/decelerate_interpolator：越来越慢
	@android:anim/accelerate_decelerate_interpolator：先快后慢
	@android:anim/anticipate_interpolator: 先后退一小步然后向前加速
	@android:anim/overshoot_interpolator：快速到达终点超出一小步然后回到终点
	@android:anim/anticipate_overshoot_interpolator：到达终点超出一小步然后回到终点
	@android:anim/bounce_interpolator：到达终点产生弹球效果，弹几下回到终点
	@android:anim/linear_interpolator：均匀速度。

## 参考
* [Android的Activity屏幕切换动画(二)-左右滑动深入与实战 - heavenforgold的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/heavenforgold/article/details/7081484)

## Log
20141021 创建。
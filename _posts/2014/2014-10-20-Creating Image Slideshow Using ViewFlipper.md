---
layout: post
category : program
tags : Android, Android View

---

### ViewFlipper

ViewFlipper is and user interface widget available in android since android API level 1. It can hold two more views, but only one child can be shown at a time. 

ViewFlipper是一个从API 1开始就存在的界面组件。
她可以hold住两个以上的View，但只能同时现实一个child view。

ViewFlipper继承自ViewAnimator。她提供方法设定进出动作：

* `setInAnimation()`
* `setOutAnimation()`

### AnimationListener


### 实现过程

定义

1. ViewFlipper
2. 动作响应，基于[onTouch->onFling->swipe](http://neosdong.github.io/program/2014/10/15/Touch,fling%20and%20swipe/)的模型
3. 动画`R.anim.left_in`,`R.anim.left_out`,`R.anim.right_in`,`R.anim.right_out`

在swipe代码块分别调用ViewFlipper的动画函数，引用以上定义的动画

* `setInAnimation()`
* `setOutAnimation()`

### 参考

* Android ViewFlipper Example - Android Image Slideshow : http://javatechig.com/android/android-viewflipper-example
* 资源文件位置错误会导致`Error inflating class <unknown>`
* onTouch的返回值是true or false 有什么影响？
 
---
layout: post
title: 如何实现一个可拖动控件？
category: Android
tags: Android
keywords: Android
description:
 
---
拖动是移动APP常用动作，能让用户直观感受到顺序、位置改变的结果。

例如，网易新闻客户端的栏目管理：

![网易栏目管理](https://raw.githubusercontent.com/neosdong/neosdong.github.com/master/img/20150107-163-news-channel-manager.png)

1. 点击，跳转到该栏目
2. 长按，进入编辑模式。
3. 编辑模式，可拖动频道以排序。点击左上角X符号移除栏目。

再如，多看阅读管理书架的操作，跟Android图标文件夹的操作类似。长按实现元素的可移动，移动过程中根据悬停位置做相应处理。

## 显示内存用量的悬浮小工具

### 一、悬浮的实现

步骤一：首先，悬浮对象是脱离Activity的存在，它的容器是窗口，通过`WindowManager`接口管理。获取`WindowManager`接口的方法：

	WindowManager wm = null;
	//获取WindowManager
    wm = (WindowManager) getApplicationContext().getSystemService(WINDOW_SERVICE);

步骤二：被悬浮的对象是一个View。添加之前要定义位置相关参数：

Application子类

    private WindowManager.LayoutParams wmParams=new WindowManager.LayoutParams();
    public WindowManager.LayoutParams getMywmParams(){
        return wmParams;
    }

Activity

    	WindowManager.LayoutParams wmParams = null;
    	// 设置LayoutParams(全局变量）相关参数
        wmParams = ((MyApplication) getApplication()).getMywmParams();
        //wmParams的类型跟行为
        wmParams.type = 2002;
        wmParams.flags |= 8;
        wmParams.gravity = Gravity.LEFT | Gravity.TOP; //设置位置
        // 坐标
        wmParams.x = 0;
        wmParams.y = 0;
        // 设置宽高
        wmParams.width = WindowManager.LayoutParams.WRAP_CONTENT;
        wmParams.height = WindowManager.LayoutParams.WRAP_CONTENT;
        wmParams.format = 1;

步骤三：使用addView联结`WindowManager`对象和被悬浮的View。

	wm.addView(memView,wmParams);
	
PS:用完之后记得，`removeView`。



### 二、拖动的实现

重写onTouchEvent，实现触控相关逻辑：

1. 获取位置：通过getRawX获取触控在屏幕的坐标
	    
		int x = (int)ev.getRawX();
        int y = (int)ev.getRawY();

2. 更新位置：改修参数，并做更新。View的大小对触电和View显示的位置有所影响，offset用于做调整，从View的宽高取得。

		wmParams.x = x-offsetX;
       	wmParams.y = y-offsetY;
        wm.updateViewLayout(dragView,wmParams);


## DragGrid的实现

### 一、视图的触控事件处理



### 二、位置判断（初始位置、悬停位置、放下的位置）

GridView继承自AbsListView，可直接使用AbsListView.pointToPosition函数，做坐标与postion的转换。
	  
pointToPosition代码：

    /**
     * Maps a point to a position in the list.
     *
     * @param x X in local coordinate
     * @param y Y in local coordinate
     * @return The position of the item which contains the specified point, or
     *         {@link #INVALID_POSITION} if the point does not intersect an item.
     */
    public int pointToPosition(int x, int y) {
        Rect frame = mTouchFrame;
        if (frame == null) {
            mTouchFrame = new Rect();
            frame = mTouchFrame;
        }

        final int count = getChildCount();
        for (int i = count - 1; i >= 0; i--) {
            final View child = getChildAt(i);
            if (child.getVisibility() == View.VISIBLE) {
                child.getHitRect(frame);
                if (frame.contains(x, y)) {
                    return mFirstPosition + i;
                }
            }
        }
        return INVALID_POSITION;
    }
    
逻辑很简单：

遍历child views。

* 如果包含该坐标，则返回该child view的postion。
* 如果child views，都不包含该坐标，返回INVALID_POSITION。

值得注意的是这个INVALD_POSITION，这个标志在以下情况出现：

1. 不在子视图的显示坐标中：在子试图的边距的空间也属于`INVALD_POSITION`。
2. 子视图被隐藏：`View.GONE`或者`View.INVISIBLE`。

### 三、动画的实现

动画的目标是，实现填充空格的效果。

开始与结束

1. 当一个格子被拖动，离开初始位置，到悬停在另一个位置，动画开始。
2. 动画结束后，格子的初始位置就在悬停位置。
3. 一个动画过程完毕。

动画分解

* 填充格子A，空出B。A:被拖动格子的初始位置，B:被拖动格子的落点。
  * 循环移动B格子，A与B之间的格子。
    * 移动格子
    * 某一次的移动（最好是最后一次）动画结束，通过`Adapter`的对象改变数据层面的排序。不是最后一次也没关系，因为此处只是改变一次数据。

填充格子的函数`fillTheGrid`

    private void fillTheGrid(int positionA,int positionB){

        if (positionA<positionB){
            for (int i=positionA+1;i<=positionB;i++){
                if (i==positionB){
                    moveItemFromTo(i,i-1,true);
                }else {
                    moveItemFromTo(i,i-1,false);
                }
            }
        }
        else if (positionA>positionB){
            for (int i=positionA-1;i>=positionB;i--){
                if (i==positionB){
                    moveItemFromTo(i,i+1,true);
                }else {
                    moveItemFromTo(i,i+1,false);
                }
            }
        }
        return;
    }
    
移动格子的函数：`moveItemFromTo`

#### 获取动画


### 四、数据处理

* 数据处理最好结合监听器实现，并且给监听器的回调函数传递结果。
* 通过Adapter改变数据。

<!--### 五、监听器的实现（未）-->

### 补充：View to ImageView

直接拖动子View会导致原有GridView的混乱。所以，复制一个图像作为被拖动的View是比较简便的做法。

DragGrid的addDragViewToWindow函数

        /*将ViewGroup转化成bitmap*/
        viewGroup.destroyDrawingCache();
        viewGroup.setDrawingCacheEnabled(true);
        Bitmap dragBitmap = Bitmap.createBitmap(viewGroup.getDrawingCache());
        imgView.setImageBitmap(dragBitmap);

### 拓展：Android的窗口管理

其实在android中真正展示给用户的是window和view，activity在android中所其的作用主要是处理一些逻辑问题，比如生命周期的管理、建立窗口等。在android中，窗口的管理还是比较重要的一块，因为他直接负责把内容展示给用户，并和用户进行交互。响应用户的输入等。

WindowManager接口继承ViewManager接口。

#### ViewManager接口

ViewManager这个接口，这个接口主要有以下的实现子接口和实现类，分别是：

* `WindowManager`接口，继承ViewManager接口。
* `ViewGroup`类，继承View，实现ViewManager接口。

里面有三个重要的方法

* `addView`:添加View到窗口。悬浮的实现是最佳实践。
* `updateViewLayout`:更新view的参数：位置等。
* `removeView`:与`addView`对应，用完就移除。

addView在哪使用？

* WindowManager的addView在Activity,ActivityThread,Dialog的显示方法中有使用。
* ViewGroup的addView在继承自ViewGroup的控件HorizontalScrollView,ScrollView,TableLayout使用。
* 更多可以通过Android Studio的Find Usages查得，记得设置查找范围为Project and Libraries。

<!--
Log
20141231 创建。
-->
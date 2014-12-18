---
layout: post
title: A SlidingMenu like QQ5.0's
category: Android
tags: Android 自定义View
keywords: 
description: 

---

## 滑动控件继承于HorizontalScrollView

1. 简便，能左右滑动，只要加上滑动的临界点。
2. HorizontalScrollView本身有解决内部View事件冲突的机制。

## 继承ViewGroup的三个构造方法

    public SlidingMenu(Context context) {
        this(context, null);
        Log.d(TAG,"SlidingMenu(Context context)");
    }

    public SlidingMenu(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
        Log.d(TAG, "SlidingMenu(Context context, AttributeSet attrs)");


    }

    public SlidingMenu(Context context, AttributeSet attrs, int defStyle){
        super(context,attrs,defStyle);
        Log.d(TAG,"SlidingMenu(Context context, AttributeSet attrs, int defStyle)");
        //...
    }

三个构造方法：

1. 一个参数：只有一个context参数的情况，用在动态实例化，`new SlidingMenu(context)`。
2. 两个参数：无自定义参数。AttributeSet用于接收XML布局文件中设定的参数。
3. 三个参数：有自定义参数的情况。(以上代码是使用自定义参数时的三个正确的构造函数写法)

### 自定义参数：菜单右边距rightPadding

自定义参数的情况，除了处理好构造函数的写法，形成正确的调用关系（构造方法二->三），还要做以下设定：

1. `res/values/attrs.xml`文件定义自定义属性
2. 布局文件增加命名空间，使用自定义属性
3. 三个参数的构造方法接收自定义属性的值

#### 自定义参数设定1：attrs.xml文件定义自定义属性

	<?xml version="1.0" encoding="utf-8"?>
	<resources>
    	<attr name="rightPadding" format="dimension">
    	</attr>
    	<declare-styleable name="SlidingMenu">
        	<attr name="rightPadding"/>
    	</declare-styleable>
	</resources>

#### 自定义参数设定2：布局文件增加命名空间，使用自定义属性

Activity布局文件增加命名空间

	xmlns:neosdong="http://schemas.android.com/apk/res-auto"
	
使用自定义属性

	<com.neosdong.ui_qq50_slidingmenu.view.SlidingMenu
		neosdong:rightPadding="80dp"
		>
		...
		
#### 自定义属性设定3：三个参数的构造方法接收自定义属性的值

        /*****************
         * Start of 获取自定义属性
         */
        TypedArray a = context.obtainStyledAttributes(attrs,R.styleable.SlidingMenu,defStyle,0);
        int n = a.getIndexCount();
        for (int i=0;i<n;i++){
            int attr = a.getIndex(i);
            switch (attr){
                case R.styleable.SlidingMenu_rightPadding:
                    Log.d(TAG,"获得SlidingMenu_rightPadding:属性");
                    mMenuRightPadding = a.getDimensionPixelSize(attr,
                            (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, 50, displayMetrics));
                    break;
            }
        }

        a.recycle();
        /*****************
         * End of 获取自定义属性
         */

REF:[Commit: UI_QQ50_SlidingMenu:自定义View的自定义属性 · 0616564 · neosdong/Android-Training](https://github.com/neosdong/Android-Training/commit/061656445be890aa3d910be2b81622e6a72ffc71) 

## onMeasure方法测量子类

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        if (!once){
            mWapper = (LinearLayout)getChildAt(0);
            mMenu = (ViewGroup)mWapper.getChildAt(0);
            mContent = (ViewGroup)mWapper.getChildAt(1);

            mMenuWidth = mMenu.getLayoutParams().width = mScreenWidth - mMenuRightPadding;
            mContent.getLayoutParams().width = mScreenWidth;

            once = true;
        }

        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    }
    
相对简单的onMeasure过程，只需要测量控件的第一个子类，用的是一个唯一的`LinearLayout`，用来包住Menu和Content的布局。这也是来自父类HorizonalScrollView的用法。（ScrollVie too）

![](https://raw.githubusercontent.com/neosdong/neosdong.github.com/8d58147421909c82dc45b8cb31ea03cda829658e/img/20141214-onMesure.jpg)

### 关于onMeasure

Ask all children to measure themselves and compute the measurement of this layout based on the children.

参数widthMeasureSpec和heightMeasureSpec不是具体的尺寸，而是一个指向**测量方法和尺寸**的ID。

1、获取测量方法

	int widthMode = View.MeasureSpec.getMode(widthMeasureSpec);
	
2、测量尺寸

	int widthPixels = View.MeasureSpec.getSize(widthMeasureSpec);

以上参数这次没用上，下回分解。

Read more:
[What are widthMeasureSpec and heightMeasureSpec in Android custom Views? - Stack Overflow](http://stackoverflow.com/questions/14493732/what-are-widthmeasurespec-and-heightmeasurespec-in-android-custom-views)

## onLayout设置子类布局
	
	@Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        Log.d(TAG,"onLayout");
        super.onLayout(changed, l, t, r, b);
        if (changed){
        	//初始，滚动到完全显示content布局的位置
            this.scrollTo(mMenuWidth,0);
        }
    }

## 动画效果

以上完成了一个SlidingMenu的基本功能，还有锦上添花：增加滚动时的动画效果。

`onScrollChanged`是监听ScrollView滚动的响应函数。

参数：
	 
	 * @param l Current horizontal scroll origin.
     * @param t Current vertical scroll origin.
     * @param oldl Previous horizontal scroll origin.
     * @param oldt Previous vertical scroll origin.
     
这里只需要使用`l`参数。根据滚动的位置，调节子View的属性：位移、缩放、透明度。

使用属性动画需要引入：
[NineOldAndroids](http://nineoldandroids.com/) | [GitHub](https://github.com/JakeWharton/NineOldAndroids/) - Android library for using the Honeycomb (Android 3.0) animation API on all versions of the platform back to 1.0!

### Scale参数的使用

动画的实现中，scale(float)参数是一个很重要的统一步伐的工具参数。所有属性动画都是以scale参数为变量，一次元变化。

    @Override
    protected void onScrollChanged(int l, int t, int oldl, int oldt) {
        super.onScrollChanged(l, t, oldl, oldt);

        //scale参数1.0~0.0 ~~~~ l的范围mMenuWidth~0
        float scale = (float)l/mMenuWidth;

        //menu位移
        ViewHelper.setTranslationX(mMenu, mMenuWidth*scale*0.8f);

        //缩放menu 0.7~1.0
        float menuScale = (1.0f-0.3f*scale);
        ViewHelper.setScaleX(mMenu,menuScale);
        ViewHelper.setScaleY(mMenu, menuScale);

        //内容缩放 1.0~0.7
        float contentScale = (0.7f+0.3f*scale);
        ViewHelper.setPivotX(mContent,0);
        ViewHelper.setPivotY(mContent,mContent.getHeight()/2);
        ViewHelper.setScaleX(mContent,contentScale);
        ViewHelper.setScaleY(mContent,contentScale);

        //menu透明度变化 0.6~1.0
        float menuAlpha = 1.0f-0.4f*scale;
        ViewHelper.setAlpha(mMenu,menuAlpha);
    }

## onFling

对比了QQ的菜单滑动，还有左右快速滑动的时候也会触发菜单toggle。
详见之前的[关于onFling文章](/2014/10/15/Touch,Fling%20and%20Swipe.html)。

### 使用GestureDetector与onTouch，而不是onTouchEvent

onTouch的优先级比onTouchEvent高。在这个情况中，如果onTouch消费了onFling过程事件，当时的ACTION_UP，onTouchEvent将不再响应。

所以，**保留原有的onTouchEvent，不修改。SlidingMenu实现View.OnTouchListener，并通过GestureDetector绑定手势响应动作。**

<!--
## 其他

一、关于HorizontalScrollView的使用，scrollTo可以直接被调用就出效果，
而MotionEvent.ACTION_UP下调用smoothScroll却需要一个Runnable（不是新线程）来运行。

    private void openMenu() {
        if (isOpen)return;
        smoothScrollTo(0,0);
        isOpen = true;
    }

    private void closeMenu() {
        if (!isOpen)return;
        smoothScrollTo(mMenuWidth,0);
        isOpen = false;
    }

TODO 调试一下
http://www.eoeandroid.com/forum.php?mod=redirect&goto=findpost&ptid=252860&pid=2778981   
初始化时候 view没有获取焦点  
view.requesFo
view.requestFoscefromWindow
view.getWindowToken
view.smooth....
ok解决

http://stackoverflow.com/a/23001821/698125 这个答案也是这个意思。
-->
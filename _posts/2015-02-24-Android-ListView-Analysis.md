---
layout: post
title: Android ListView源码分析（一）
category : Android
tags : Android XML解析

---

接触Android开发，少不了ListView。大多数教程都是教我们，继承一个BaseAdapter，然后重写那几个方法，再调用相关view的 setAdpater()方法， 接着，你的item 就显示在手机屏幕上了。

比较深入的也不过是说说adapter getview()中的回收情况。

* ListView / GirdView Adpater的getView方法，首项多次调用 | Yet Another Summer Rain : http://www.liaohuqiu.net/cn/posts/first-view-will-be-created-multi-times-in-list-view/
* android listview 异步加载图片并防止错位 - LeslieFang - 博客园 : http://www.cnblogs.com/lesliefang/p/3619223.html

ListView的继承关系：

* AdapterView
  * AbsListView
    * ListView
    
## AdpaterView概览

AdpaterView

>>api手册的说明：An AdapterView is a view whose children are determined by an Adapter.

实际上android里面ListView,GridView,Spinner,Gallery等view都是基于设计模式上的设配器模式实现的，只要熟悉设配器模式的相关知识，就知道如何从源码里面找到相关的实现线索。

https://github.com/android/platform_frameworks_base/blob/master/core/java/android/widget/AdapterView.java

要理解listview等的实现，其父类是不得不看。源码有1200多行。阅读完AdapterView,能搞明白以下问题：

1. 响应数据的更改
2. 知道点击view的时候，获得对应的位置

### 响应数据的更改

#### 观察者模式的应用:BaseAdapter notifyDataChanged响应数据更改背后的逻辑

数据修改之后，调用BaseAdapter.notifyDataChanged是如何更新数据的？

从源码入手，调用过程如下：

* 实际上调用的是DataSetObservable notifyChanged();
  * 【观察者模式核心原理】被观察者DataSetObservable的`notifyChanged`调用观察者DataSetObserver的`onChanged()`方法

可知，数据更新机制使用了设计模式中的观察者模式：数据更新，触发ListView界面的更新。这些更新的响应在AdapterView开始设计。

我的观察者模式DEMO：Java-Training/Pattern_Observer at ce0016901dac9191dc5a6b3e94d2fa26b9623ddb · neosdong/Java-Training : https://github.com/neosdong/Java-Training/tree/ce0016901dac9191dc5a6b3e94d2fa26b9623ddb/Pattern_Observer

![](https://github.com/neosdong/Java-Training/blob/ce0016901dac9191dc5a6b3e94d2fa26b9623ddb/Pattern_Observer/notes/2015-02-24-%E8%A7%82%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F%E8%AE%BE%E8%AE%A1%E6%96%B9%E6%A1%88.png?raw=true)

补充：

notifyDataSetChanged() 与notifyDataSetInvalidated() 具体会到AdapterView 产生什么影响？

我们对比一下onChange() 与 onInvalidated() 方法，就能对比得出，前者会对当前位置的状态进行同步，而后者会重置所有位置的状态。从代码的注释里面还可以获取得到更多的信息。

#### AdapterView的观察者响应

第798行，`class AdapterDataSetObserver extends DataSetObserver {...}`。

定义了一个继承于DataSetObserver的类。

DataSetObserver是观察者模式中的观察者，负责接收事件数据，根据数据并对事件做出响应。

需要继承两个抽象方法：

* onChanged
* onInvalidated

观察者响应的主要任务：
* 获取数据：通过Adapter获取数据
* 更新视图

【扩展】：Adapter使用的是适配器模式，将数据和视图接入到ListView。

### 点击item 怎么能够获取到当前的位置

AdapterView第598行，`public int getPositionForView(View view) {`。

    /**
     * Get the position within the adapter's data set for the view, where view is a an adapter item
     * or a descendant of an adapter item.
     *
     * @param view an adapter item, or a descendant of an adapter item. This must be visible in this
     *        AdapterView at the time of the call.
     * @return the position within the adapter's data set of the view, or {@link #INVALID_POSITION}
     *         if the view does not correspond to a list item (or it is not currently visible).
     */

对于getPositionForView() 这个方法，你肯定没用过，要搞明白为什么我们能够获取到adapterView 里面item view对应的位置，我们需要看 其直接子类：AbsListView.class

AbsListView第2316行，`View obtainView(int position, boolean[] isScrap) {`

这是一个获取itemView的方法。把这块代码看完，以后，会不会有个疑问呢（先不用管回收那块）？ position 到哪里了？我们可以看到这个方法实际上并没有对我们的itemview 设置了任何的监听器，那为什么最后能对我们的itemview的动作进行反应呢？

接下来使用了[委托模式](http://zh.wikipedia.org/wiki/%E5%A7%94%E6%89%98%E6%A8%A1%E5%BC%8F):不必继承B或者C，但通过委托者类A根据情况调用B或者C的方法。

具体根据委托者模式分析：

AbsListView第2398行，`class ListItemAccessibilityDelegate extends AccessibilityDelegate {`

分发事件...

第1142行，performItemClick函数内

	if (dispatchItemClick) {
        handled |= super.performItemClick(view, position, id);
    }
    
父类AdapterView的performItemClick，实现接口函数的回调。

【回顾】为什么不直接在AbsListView实现onClick,onLongClick等动作接口函数的回调？辗转使用委托者模式？

回顾
1. AdapterView负责实现数据更改的响应，动作接口的回调。这些对AbsListView,GridView来说是通用的。子类负责具体的子布局itemView加载与呈现。
2. 绑定动作接口的行为在AdapterView就已经具体实现，不需要再继承。所以用委托者模式，发挥其不需继承，委托调用的特性。

## 认识AbsListView回收机制

长期以来，都有这么一个说法，listview 会自动把不可见的view进行回收，但是长期以来，我都没看到有人对其回收机制进行分析说明。

AbsListView第2316行，`View obtainView(int position, boolean[] isScrap) {`

对，刚才看过，是获取itemView的方法。这个方法有个Feild - `final RecycleBin mRecycler`。

这是一个内部类RecycleBin，我们来看一下这个类的注释，大体的意思这个类是用来帮助复用view的，使用了二级存储。

    /**
     * The RecycleBin facilitates reuse of views across layouts. The RecycleBin has two levels of
     * storage: ActiveViews and ScrapViews. ActiveViews are those views which were onscreen at the
     * start of a layout. By construction, they are displaying current information. At the end of
     * layout, all views in ActiveViews are demoted to ScrapViews. ScrapViews are old views that
     * could potentially be used by the adapter to avoid allocating views unnecessarily.
     *
     * @see android.widget.AbsListView#setRecyclerListener(android.widget.AbsListView.RecyclerListener)
     * @see android.widget.AbsListView.RecyclerListener
     */

#####【回收机制一】二级存储itemView
* ActiveViews ： 一开始显示在屏幕的view
* ScrapViews： 潜在的一些可以让adpater 使用的old views。

然后，注释里面已经说了，ActiveViews 怎么变成 ScrapViews。就注释提供的信息这里我们有两个疑问。

1. 什么时候产生 ActiveViews。
1. 什么时候产生 ScrapViews。

这要把这两点搞清楚了，整个回收体系也就清楚了。

### AbsListView的回收机制的实现

从RecycleBin类的注释里面我们获知，回收机制的第一步就是屏幕的view放在ActiveViews，然后通过对ActiveViews进行降级变成ScrapViews,然后通过scrapViews 进行view 的复用。

#####【回收机制二】数据如果没有发生改变，则使用当前已经存在ActiveViews的view。

AbsListView第6400行，RecycleBin的方法：将所有子布局填充到ActiveViews。

        /**
         * Fill ActiveViews with all of the children of the AbsListView.
         *
         * @param childCount The minimum number of views mActiveViews should hold
         * @param firstActivePosition The position of the first view that will be stored in
         *        mActiveViews
         */
        void fillActiveViews(int childCount, int firstActivePosition) {

值得注意的是，通过在getView log listview.getChildCount()发现，子布局只有界面上显示的那些。通过分析这个函数的实现，可知，这个ActiveViews数组仅仅保存了可视的View。
        
fillActiveViews方法在ListView layoutChildren，第1626开始相关调用

            // Pull all children into the RecycleBin.
            // These views will be reused if possible
            final int firstPosition = mFirstPosition;
            final RecycleBin recycleBin = mRecycler;
            if (dataChanged) {
                for (int i = 0; i < childCount; i++) {
                    recycleBin.addScrapView(getChildAt(i), firstPosition+i);
                }
            } else {
                recycleBin.fillActiveViews(childCount, firstPosition);
            }
            
在给children布局时，我们可以看到一个变量dataChanged，从单词的意思我们就可以，这里的优化规则就是基于数据是否有变化。通过搜索找到这个变量声明自AdapterView (144)，

    /**
     * True if the data has changed since the last layout
     */
    boolean mDataChanged;



回收的调用在ListView，继续搜索ListView代码。
我们通过搜索成员变量mDataChanged在 (1787) 的时候变成了false 接着我们在makeAndAddView（1846）发现了这个变量的使用。

    /**
     * Obtain the view and add it to our list of children. The view can be made
     * fresh, converted from an unused view, or used as is if it was in the
     * recycle bin.
     *
     * @param position Logical position in the list
     * @param y Top or bottom edge of the view to add
     * @param flow If flow is true, align top edge to y. If false, align bottom
     *        edge to y.
     * @param childrenLeft Left edge where children should be positioned
     * @param selected Is this position selected?
     * @return View that was added
     */
    private View makeAndAddView(int position, int y, boolean flow, int childrenLeft,
            boolean selected) {
        View child;


        if (!mDataChanged) {
            // Try to use an existing view for this position
            child = mRecycler.getActiveView(position);
            if (child != null) {
                // Found it -- we're using an existing child
                // This just needs to be positioned
                setupChild(child, position, y, flow, childrenLeft, selected, true);

                return child;
            }
        }

        // Make a new view for this position, or convert an unused view if possible
        child = obtainView(position, mIsScrap);

        // This needs to be positioned and measured
        setupChild(child, position, y, flow, childrenLeft, selected, mIsScrap[0]);

        return child;
    }
    
阅读（1851 - 1861） 我们可以看到回收机制的第一次使用，如果数据没有发生改变，通过判断ActiveViews(这些些view来自（1557 - 1562）) 列表里面有没有当前 活动view，有的话直接复用已经存在的view。这样的好处就是直接复用当前已经存在的view，不需要通过adapter.getview()里面获取子view。

#####【回收机制三】滚动的优化：convertView来实现

convertView是Adapter getView的第二个参数。getView在AbsListView的obtainView(2316)调用。

`final View updatedView = mAdapter.getView(position, transientView, this);`

transientView来自（2323）`final View transientView = mRecycler.getTransientStateView(position);`

从单词的意思里面我们可以得知这是获取一个瞬间状态的view，这里就有个疑问什么是瞬间状态的view？

AbsListView(6445)从RecycleBin getTransientStateView的实现看，"瞬时状态"view来自RecycleBin的两个数组：

1. mTransientStateViewsById
2. mTransientStateViews

这两个数组在何时添加View？

AbsListView(6511)的addScrapView实现中，

注释表示，
如果adapter有稳定的IDs，我们可以复用view for the same data。
AbsListView(6544),`mTransientStateViewsById.put(lp.itemId, scrap);`。如果数据没有变化，我们reuse the views at their old positions
AbsListView(6551),`mTransientStateViews.put(position, scrap);`。

addScapView在何时调用，添加瞬时View到以上两个数组中？



ListView有四个函数调用了addScapView

1. layoutChildren
2. measureHeightOfChildren
3. onMeasure
4. scrollListItemsBy

1、layoutChildren，初始布局和布局有变化时调用。

ListView(1630)

            if (dataChanged) {
                for (int i = 0; i < childCount; i++) {
                    recycleBin.addScrapView(getChildAt(i), firstPosition+i);
                }
            } else {
                recycleBin.fillActiveViews(childCount, firstPosition);
            }
            
只添加了当前可见的itemView，并没有未可见的View。

2、measureHeightOfChildren，同样只添加了当前可见的itemView

        for (i = startPosition; i <= endPosition; ++i) {
            child = obtainView(i, isScrap);

            measureScrapChild(child, i, widthMeasureSpec);

            if (i > 0) {
                // Count the divider for all but one child
                returnedHeight += dividerHeight;
            }

            // Recycle the view before we possibly return from the method
            if (recyle && recycleBin.shouldRecycleViewType(
                    ((LayoutParams) child.getLayoutParams()).viewType)) {
                recycleBin.addScrapView(child, -1);
            }

3、onMeasure，只测量第0个item，然后`mRecycler.addScrapView(child, 0);`


4、scrollListItemBy(int)

当顶部itemView从上方离开，**离开的itemView会被添加到回收器**。
itemView从父布局移除。函数内，**没有生成新的itemView来补充**。

            // top views may be panned off screen
            View first = getChildAt(0);
            while (first.getBottom() < listTop) {
                AbsListView.LayoutParams layoutParams = (LayoutParams) first.getLayoutParams();
                if (recycleBin.shouldRecycleViewType(layoutParams.viewType)) {
                    recycleBin.addScrapView(first, mFirstPosition);
                }
                detachViewFromParent(first);
                first = getChildAt(0);
                mFirstPosition++;
            }

当底部itemView从下方离开，离开的itemView也会被添加到回收器。

这里放进回收类里面的只有当前的显示的view，并没有产生当前屏幕没有的view，但是，实际使用中，当我们进行滚屏的时候，显示下个view的时候，就已经能发现getView 第二个参数已经不为null了，那实际实现在哪里了？

可以推测，getView回调时，第二个参数复用了回收器的itemView。


### REF

* Android AdapterView 源码分析以及其相关回收机制的分析 - youxiachai - 博客园 : http://www.cnblogs.com/youxilua/archive/2013/05/11/3072353.html

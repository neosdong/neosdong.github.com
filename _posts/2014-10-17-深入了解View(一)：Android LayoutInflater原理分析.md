---
layout: post
category : program
tags : Android Layout

---


## 加载布局的类

相信接触Android久一点的朋友对于LayoutInflater一定不会陌生，都会知道它主要是用于加载布局的。

而刚接触Android的朋友可能对LayoutInflater不怎么熟悉，因为加载布局的任务通常都是在Activity中调用setContentView()方法来完成的。其实setContentView()方法的内部也是使用LayoutInflater来加载布局的，只不过这部分源码是internal的，不太容易查看到。

先来看一下LayoutInflater的基本用法吧，它的用法非常简单，首先需要获取到LayoutInflater的实例，有两种方法可以获取到，

第一种写法如下：

    LayoutInflater layoutInflater = LayoutInflater.from(context);  

当然，还有另外一种写法也可以完成同样的效果：

    LayoutInflater layoutInflater = (LayoutInflater) context  
        .getSystemService(Context.LAYOUT_INFLATER_SERVICE);  

其实第一种就是第二种的简单写法，只是Android给我们做了一下封装而已。

得到了LayoutInflater的实例之后就可以调用它的inflate()方法来加载布局了，如下所示：

    layoutInflater.inflate(resourceId, root);

inflate()方法一般接收两个参数，第一个参数就是要加载的布局id，第二个参数是指给该布局的外部再嵌套一层父布局，如果不需要就直接传null。这样就成功成功创建了一个布局的实例，之后再将它添加到指定的位置就可以显示出来了。

## 小例子
下面我们就通过一个非常简单的小例子，来更加直观地看一下LayoutInflater的用法。比如说当前有一个项目，其中MainActivity对应的布局文件叫做activity_main.xml，代码如下所示：

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    	android:id="@+id/main_layout"
    	android:layout_width="match_parent"
    	android:layout_height="match_parent" >
    </LinearLayout>

这个布局文件的内容非常简单，只有一个空的LinearLayout，里面什么控件都没有，因此界面上应该不会显示任何东西。

那么接下来我们再定义一个布局文件，给它取名为button_layout.xml，代码如下所示：

    <Button xmlns:android="http://schemas.android.com/apk/res/android"
    	android:layout_width="wrap_content"
    	android:layout_height="wrap_content"
    	android:text="Button" >
    </Button>

这个布局文件也非常简单，只有一个Button按钮而已。现在我们要想办法，如何通过LayoutInflater来将button_layout这个布局添加到主布局文件的LinearLayout中。根据刚刚介绍的用法，修改MainActivity中的代码，如下所示：

    public class MainActivity extends Activity {

	private LinearLayout mainLayout;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		mainLayout = (LinearLayout)findViewById(R.id.main_layout);
		LayoutInflater layoutInflater = LayoutInflater.from(this);
		View buttonLayout = layoutInflater.inflate(R.layout.button_layout, null);
		mainLayout.addView(buttonLayout);
	    }
    }
    
可以看到，这里先是获取到了LayoutInflater的实例，然后调用它的inflate()方法来加载button_layout这个布局，最后调用LinearLayout的addView()方法将它添加到LinearLayout中。

现在可以运行一下程序。

Button在界面上显示出来了！说明我们确实是借助LayoutInflater成功将button_layout这个布局添加到LinearLayout中了。LayoutInflater技术广泛应用于需要动态添加View的时候，比如在ScrollView和ListView中，经常都可以看到LayoutInflater的身影。

## 深入了解LayoutInflater

从源码的角度上看一看LayoutInflater到底是如何工作的，结论：

1. LayoutInflater其实就是使用Android提供的pull解析方式来解析布局文件的。
2. LinearLayout的父布局确实是一个FrameLayout，而这个FrameLayout就是由系统自动帮我们添加上的。

附上一张Activity窗口的组成图，以便于大家更加直观地理解：

![Mou icon](http://s1.5km.co/201410/2416/47808_z.png)

## 参考

* Android LayoutInflater原理分析，带你一步步深入了解View(一) - 郭霖的专栏 - 博客频道 - CSDN.NET : http://blog.csdn.net/guolin_blog/article/details/12921889
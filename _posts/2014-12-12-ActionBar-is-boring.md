---
layout: post
title: 搞定ActionBar（一）：学习Android Developers API GUIDE
category: Android
tags: 
keywords: ActionBar
description: 
---

ActionBar是个很麻烦的家伙，从外观看来，它也就是自定义ViewGroup标题能实现的效果。偶最怕这种难以抽象归类的知识了。

但是，像**ParentActivity with Up/Home,Action View,Split action bar,Action Provider,**Navigation Tabs,Drop down Navigation这些组件，用起来很有框架的思维。又不得不佩服。

于是，今天照着Google的教程练习了上述组件的，黑体字部分，还留下了Custom Action Provider,Navigation Tabs,Drop down Navigation。练习做的Demo很粗糙，部分知识点在实战中还有待巩固加深印象。

另外，Google的文档果然是传说中最好的教程。

以下是学习的备忘录。

----
## Action Bar | Android Developers

原地址：[Action Bar - Android Developers](http://developer.android.com/guide/topics/ui/actionbar.html)


The ActionBar APIs were 

* first added in Android 3.0 (API level 11) 
* but they are also available in the Support Library for compatibility with Android 2.1 (API level 7) and above.

从3.0开始有ActionBar，用兼容包可以支持到2.1。

This guide focuses on how to use the support library's action bar, but if your app supports only Android 3.0 or higher, you should use the ActionBar APIs in the framework. Most of the APIs are the same—but reside in a different package namespace—with a few exceptions to method names or signatures that are noted in the sections below.

**这个教程专注于兼容包的使用**，API基本一致。使用的时候注意引用不同的包，学一次就够。

Caution: Be certain you import the ActionBar class (and related APIs) from the appropriate package:

* If supporting API levels lower than 11: 
`import android.support.v7.app.ActionBar`
* If supporting only API level 11 and higher: 
`import android.app.ActionBar`

### Adding the Action Bar

API<11

1. Create your activity by extending ActionBarActivity.
2. Use (or extend) one of the Theme.AppCompat themes for your activity. For example:
`<activity android:theme="@style/Theme.AppCompat.Light" ... >`

API>=11

The action bar is included in all activities that use the Theme.Holo theme (or one of its descendants), which is the default theme when either the targetSdkVersion or minSdkVersion attribute is set to "11" or higher. If you don't want the action bar for an activity, set the activity theme to Theme.Holo.NoActionBar.

1. 使用`Theme.Holo`，或其附属主题的Activity都有ActionBar
2. 如果不需要了，将主题设置为`Theme.Holo.NoActionBar`

#### Logo属性比icon高?未测试。

> By default, the system uses your application icon in the action bar, as specified by the icon attribute in the <application> or <activity> element. However, if you also specify the logo attribute, then the action bar uses the logo image instead of the icon.
<br><br>
A logo should usually be wider than the icon, but should not include unnecessary text. You should generally use a logo only when it represents your brand in a traditional format that users recognize. A good example is the YouTube app's logo—the logo represents the expected user brand, whereas the app's icon is a modified version that conforms to the square requirement for the launcher icon.


### Removing the action bar
API<11

	ActionBar actionBar = getSupportActionBar();
	actionBar.hide();

API>=11

Get the ActionBar with the `getActionBar()` method.

如果频繁改变ActionBar的状态，要使用Overlay模式。

### 添加Logo
API<11,Using ActionBarActivity 

动态添加
	        
	ActionBar actionBar = getSupportActionBar();
    actionBar.setDisplayShowHomeEnabled(true);
    actionBar.setLogo(R.drawable.ic_launcher);
    actionBar.setDisplayUseLogoEnabled(true);
    
相当复杂。参考：http://stackoverflow.com/a/26890240/698125

静态添加，在Manifest填写android/app:logo/icon属性都无效。    

API>=11...

### 添加右边按钮

参考原文：http://developer.android.com/guide/topics/ui/actionbar.html#ActionItems

### Using split action bar 

这个应该是很难实现的。

Why android:uiOptions="splitActionBarWhenNarrow" not working with device - Stack Overflow : http://stackoverflow.com/questions/11642840/why-androiduioptions-splitactionbarwhennarrow-not-working-with-device

### Navigating Up with the App Icon

API<11

Activity.java

    ActionBar actionBar = getSupportActionBar();
    actionBar.setDisplayHomeAsUpEnabled(true);//显示返回主页面的按钮
        
Manifest

    <activity
        android:name=".Activity2"
        android:label="@string/title_activity_activity2"
        android:parentActivityName=".MainActivity" >
        <!-- Parent activity meta-data to support API level 7+ -->
        <meta-data
                android:name="android.support.PARENT_ACTIVITY"
                android:value=".MainActivity" />
    </activity> 
    
    
### Adding an Action View

添加Action View。使用v7包的。一句搞定。

	<?xml version="1.0" encoding="utf-8"?>
	<menu xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:yourapp="http://schemas.android.com/apk/res-auto" >
    	<item android:id="@+id/action_search"
          android:title="@string/action_search"
          android:icon="@drawable/ic_action_search"
          yourapp:showAsAction="ifRoom|collapseActionView"
          yourapp:actionViewClass="android.support.v7.widget.SearchView" />
	</menu>

### Handling collapsible action views

v7

安装教程添加代码。注意使用了`collapseActionView`值才生效

`yourapp:showAsAction="ifRoom|collapseActionView"`
 
 
### Adding an Action Provider

Similar to an action view, an action provider replaces an action button with a customized layout. However, unlike an action view, an action provider takes control of all the action's behaviors and an action provider can display a submenu when pressed.

To declare an action provider, supply the actionViewClass attribute in the menu <item> tag with a fully-qualified class name for an ActionProvider.

`actionViewClass`中填写ActionProvider子类的全名。那ActionProvider也是ActionView的一种？

You can build your own action provider by extending the ActionProvider class, but Android provides some pre-built action providers such as ShareActionProvider, which facilitates a "share" action by showing a list of possible apps for sharing directly in the action bar (as shown in figure 6).

你可以继承ActionProvider重写属于自己的action provider。但是Android还预先准备了一些：类如，ShareActionProvider，呈现一些可能的"share" action.

Because each ActionProvider class defines its own action behaviors, you don't need to listen for the action in the onOptionsItemSelected() method. If necessary though, you can still listen for the click event in the onOptionsItemSelected() method in case you need to simultaneously perform another action. But be sure to return false so that the the action provider still receives the onPerformDefaultAction() callback to perform its intended action.

因为ActionProvider已经定义了动作，开发者不需要在`onOptionsItemSelected()`监听。

**如果要在`onOptionsItemSelected()`监听，请返回`false`**


However, if the action provider provides a submenu of actions, then your activity does not receive a call to onOptionsItemSelected() when the user opens the list or selects one of the submenu items.

如果action provider有子菜单，那`onOptionsItemSelected()`是无法监听的。

#### Using the ShareActionProvider

1. 在menu.xml注册：`yourapp:actionProviderClass="android.support.v7.widget.ShareActionProvider"`
2. 在`onCreateOptionsMenu`添加代码，设定shareItem对应的mShareActionProvider。

		// Set up ShareActionProvider's default share intent
    	MenuItem shareItem = menu.findItem(R.id.action_share);
    	mShareActionProvider = (ShareActionProvider)MenuItemCompat.getActionProvider(shareItem);
    	mShareActionProvider.setShareIntent(getDefaultIntent());

无需在`onOptionsItemSelected()`添加事件响应。

#### Creating a custom action provider

Creating your own action provider allows you to re-use and manage dynamic action item behaviors in a self-contained module, rather than handle action item transformations and behaviors in your fragment or activity code. As shown in the previous section, Android already provides an implementation of ActionProvider for share actions: the ShareActionProvider.

To create your own action provider for a different action, simply extend the ActionProvider class and implement its callback methods as appropriate. Most importantly, you should implement the following:

ActionProvider()

This constructor passes you the application Context, which you should save in a member field to use in the other callback methods.

onCreateActionView(MenuItem)

This is where you define the action view for the item. Use the Context acquired from the constructor to instantiate a LayoutInflater and inflate your action view layout from an XML resource, then hook up event listeners. For example:

	public View onCreateActionView(MenuItem forItem) {
    	// Inflate the action view to be shown on the action bar.
    	LayoutInflater layoutInflater = LayoutInflater.from(mContext);
    	View view = layoutInflater.inflate(R.layout.action_provider, null);
    	ImageButton button = (ImageButton)view.findViewById(R.id.button);
    	
    	button.setOnClickListener(new View.OnClickListener() {
        	@Override
        	public void onClick(View v) {
            	// Do something...
        	}
    	});
    	return view;
	}

onPerformDefaultAction()

The system calls this when the menu item is selected from the action overflow and the action provider should perform a default action for the menu item.
However, if your action provider provides a submenu, through the onPrepareSubMenu() callback, then the submenu appears even when the action provider is placed in the action overflow. Thus, onPerformDefaultAction() is never called when there is a submenu.

Note: An activity or a fragment that implements 

onOptionsItemSelected() can override the action provider's default behavior (unless it uses a submenu) by handling the item-selected event (and returning true), in which case, the system does not call onPerformDefaultAction().

For an example extension of ActionProvider, see ActionBarSettingsActionProviderActivity.

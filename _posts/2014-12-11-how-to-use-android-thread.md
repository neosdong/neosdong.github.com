---
layout: post
title: Android线程的应用
category: Android
tags: Android 多线程
keywords: Android process thread 
description: 
---

## Android进程

### 概念

在了解Android线程之前得先了解一下Android的进程。
当一个程序第一次启动的时候，Android会启动一个LINUX进程和一个主线程。默认的情况下，所有该程序的组件都将在该进程和线程中运行。同时，Android会为每个应用程序分配一个单独的LINUX用户。Android会尽量保留一个正在运行进程，只在内存资源出现不足时，Android会尝试停止一些进程从而释放足够的资源给其他新的进程使用， 也能保证用户正在访问的当前进程有足够的资源去及时地响应用户的事件。Android会根据进程中运行的组件类别以及组件的状态来判断该进程的重要性，Android会首先停止那些不重要的进程。按照重要性从高到低一共有五个级别：

### 进程的五个级别

#### 前台进程

前台进程是用户当前正在使用的进程。只有一些前台进程可以在任何时候都存在。他们是最后一个被结束的，当内存低到根本连他们都不能运行的时候。一般来说， 在这种情况下，设备会进行内存调度，中止一些前台进程来保持对用户交互的响应。

#### 可见进程
可见进程不包含前台的组件但是会在屏幕上显示一个可见的进程是的重要程度很高，除非前台进程需要获取它的资源，不然不会被中止。
服务进程
运行着一个通过startService() 方法启动的service，这个service不属于上面提到的2种更高重要性的。service所在的进程虽然对用户不是直接可见的，但是他们执行了用户非常关注的任务（比如播放mp3，从网络下载数据）。只要前台进程和可见进程有足够的内存，系统不会回收他们。

什么时候，我们需要使用service呢？ 

我们知道，service是运行在后台的应用，对于用户来说失去了被关注的焦点。这就跟我们打开了音乐播放之后，便想去看看图片，这时候我们还不想音乐停止，这里就会用到service；又例如，我们打开了一个下载链接之后，我们肯定不想瞪着眼睛等他下载完再去做别的事情，对吧？这时候如果我们想手机一边在后台下载，一边可以让我去看看新闻啥的，就要用到service。 

See also:[Ken Yang 筆記: Android: startService vs bindService](http://blog.kenyang.net/2012/11/android-startservice-vs-bindservice.html) 

> 先去看Google的定義，對於startService，Google定義如下
<br>
Once started, a service can run in the background indefinitely, even if the component that started it is destroyed. Usually, a started service performs a single operation and does not return a result to the caller. For example, it might download or upload a file over the network. When the operation is done, the service should stop itself.
<br>
簡言之用startService啟動，這個service是不會跟隨著啓動他的component消滅！且原則上是不能與UI互動的！
<br><br>
而對於bindService，定義如下：
<br>
A bound service offers a client-server interface that allows components to interact with the service, send requests, get results, and even do so across processes with interprocess communication (IPC). A bound service runs only as long as another application component is bound to it. Multiple components can bind to the service at once, but when all of them unbind, the service is destroyed.
<br>
用bindService起來的話，則此service可以跟該component進行溝通！甚至可以做到 IPC！
<br><br>
小結：
bindService和startService最主要的差異在於其本身的lifecycle！
以及bindService可以用來做IPC！簡單的說就是可以讓你的service和UI溝通！

#### 后台进程
运行着一个对用户不可见的activity（调用过 onStop() 方法).这些进程对用户体验没有直接的影响，可以在服务进程、可见进程、前台进 程需要内存的时候回收。通常，系统中会有很多不可见进程在运行，他们被保存在LRU (least recently used) 列表中，以便内存不足的时候被第一时间回收。如果一个activity正 确的执行了它的生命周期，关闭这个进程对于用户体验没有太大的影响。

#### 空进程
未运行任何程序组件。运行这些进程的唯一原因是作为一个缓存，缩短下次程序需要重新使用的启动时间。系统经常中止这些进程，这样可以调节程序缓存和系统缓存的平衡。

#### 依赖进程的重要性评级

Android 对进程的重要性评级的时候，选取它最高的级别。
另外，当被另外的一个进程依赖的时候，某个进程的级别可能会增高。一个为其他进程服务的进程永远不会比被服务的进程重要级低。因为服务进程比后台activity进程重要级高，因此一个要进行耗时工作的activity最好启动一个service来做这个工作，而不是开启一个子进程――特别是这个操作需要的时间比activity存在的时间还要长的时候。例如，在后台播放音乐，向网上上传摄像头拍到的图片，使用service可以使进程最少获取到“服务进程”级别的重要级，而不用考虑activity目前是什么状态。broadcast receivers做费时的工作的时候，也应该启用一个服务而不是开一个线程。

如何确立服务与被服务关系？<!--TODO 是否只有bindService启动的才是服务与被服务关系？-->

## 单线程模型

当一个程序第一次启动时，Android会同时启动一个对应的主线程（Main Thread），主线程主要负责处理与UI相关的事件，如用户的按键事件，用户接触屏幕的事件以及屏幕绘图事件，并把相关的事件分发到对应的组件进行处理。所以主线程通常又被叫做UI线程。在开发Android应用时必须遵守单线程模型的原则： Android UI操作并不是线程安全的并且这些操作必须在UI线程中执行。
 
### 子线程更新UI

Android的UI是单线程(Single-threaded)的。为了避免拖住GUI，一些较费时的对象应该交给独立的线程去执行。如果幕后的线程来执行UI对象，Android就会发出错误讯息 CalledFromWrongThreadException。以后遇到这样的异常抛出时就要知道怎么回事了！
 
### Message Queue

在单线程模型下，为了解决类似的问题，Android设计了一个Message Queue(消息队列)， 线程间可以通过该Message Queue并结合Handler和Looper组件进行信息交换。下面将对它们进行分别介绍：

#### Message
Message消息，理解为线程间交流的信息，处理数据后台线程需要更新UI，则发送Message内含一些数据给UI线程。
 
#### Handler
Handler处理者，是Message的主要处理者，负责Message的发送，Message内容的执行处理。后台线程就是通过传进来的Handler对象引用来`sendMessage(Message)`。而使用Handler，需要implement 该类的 `handleMessage(Message)`方法，它是处理这些Message的操作内容，例如Update UI。通常需要子类化Handler来实现handleMessage方法。
 
#### Message Queue
Message Queue消息队列，用来存放通过Handler发布的消息，按照先进先出执行。

每个message queue都会有一个对应的Handler。Handler会向message queue通过两种方法发送消息：`sendMessage`或`post`。这两种消息都会插在message queue队尾并按先进先出执行。但通过这两种方法发送的消息执行的方式略有不同：**通过sendMessage发送的是一个message对象,会被Handler的handleMessage()函数处理；而通过post方法发送的是一个runnable对象，则会自己执行。**
 
#### Looper

Looper是每条线程里的Message Queue的管家。**Android没有Global的Message Queue，而Android会自动替主线程(UI线程)建立Message Queue，但在子线程里并没有建立Message Queue。**所以调用`Looper.getMainLooper()`得到的主线程的Looper不为NULL，但调用`Looper.myLooper()`得到当前线程的Looper就有可能为NULL。

对于子线程使用Looper，API Doc提供了正确的使用方法：

	class LooperThread extends Thread {
    	public Handler mHandler;
 	
    	public void run() {
        	Looper.prepare(); //创建本线程的Looper并创建一个MessageQueue
 
        	mHandler = new Handler() {
           	public void handleMessage(Message msg) {
           		// process incoming messages here
           	}
        };
   
        		Looper.loop(); //开始运行Looper,监听Message Queue
    	}
	}

#### Message机制的流程

这个Message机制的大概流程：

1. 在Looper.loop()方法运行开始后，循环地按照接收顺序取出Message Queue里面的非NULL的Message。
2. 一开始Message Queue里面的Message都是NULL的。当`Handler.sendMessage(Message)`到Message Queue，该函数里面设置了那个Message对象的target属性是当前的Handler对象。随后Looper取出了那个Message，则调用该Message的target指向的Hander的`dispatchMessage`函数对Message进行处理。
<br>
在dispatchMessage方法里，如何处理Message则由用户指定，三个判断，优先级从高到低：
	1. Message里面的Callback，一个实现了Runnable接口的对象，其中run函数做处理工作；
	2. Handler里面的mCallback指向的一个实现了Callback接口的对象，由其handleMessage进行处理；
    3. 处理消息Handler对象对应的类继承并实现了其中handleMessage函数，通过这个实现的handleMessage函数处理消息。
    
    由此可见，我们实现的handleMessage方法是优先级最低的！<!--TODO 实际应用场景是？-->
    

3. Handler处理完该Message (update UI) 后，Looper则设置该Message为NULL，以便回收！
 


在网上有很多文章讲述主线程和其他子线程如何交互，传送信息，最终谁来执行处理信息之类的，个人理解是最简单的方法——判断Handler对象里面的Looper对象是属于哪条线程的，则由该线程来执行！

1. 当Handler对象的构造函数的参数为空，则为当前所在线程的Looper；
2. `Looper.getMainLooper()`得到的是主线程的Looper对象，`Looper.myLooper()`得到的是当前线程的Looper对象。

现在来看一个例子，模拟从网络获取数据，加载到ListView的过程：

	public class ListProgressDemo extends ListActivity {
 
    	@Override
    	public void onCreate(Bundle savedInstanceState) {
        	super.onCreate(savedInstanceState);
        	setContentView(R.layout.listprogress);
 
        	((Button) findViewById(R.id.load_Handler)).setOnClickListener(new View.OnClickListener(){
 
            	@Override
            	public void onClick(View view) {
               		data = null;
                	data = new ArrayList<String>();
 
                	adapter = null;
 
                	showDialog(PROGRESS_DIALOG);
                	new ProgressThread(handler, data).start();
            	}
        	});
    	}
 
    @Override
    protected Dialog onCreateDialog(int id) {
        switch(id) {
        case PROGRESS_DIALOG:
                 return ProgressDialog.show(this, "",  
                   "Loading. Please wait...", true);
 
        default: return null;
        }
    }
 
    private class ProgressThread extends Thread {
 
        private Handler handler;
        private ArrayList<String> data;
 
        public ProgressThread(Handler handler, ArrayList<String> data) {
            this.handler = handler;
            this.data = data;
        }
 
        @Override
        public void run() {
            for (int i=0; i<8; i++) {
                data.add("ListItem"); //后台数据处理
                try {
                    Thread.sleep(100);
                }catch(InterruptedException e) {
                     
                    Message msg = handler.obtainMessage();
                    Bundle b = new Bundle();
                    b.putInt("state", STATE_ERROR);
                    msg.setData(b);
                    handler.sendMessage(msg);  
                     
                }
            }
            Message msg = handler.obtainMessage();
            Bundle b = new Bundle();
            b.putInt("state", STATE_FINISH);
            msg.setData(b);
            handler.sendMessage(msg);
        }
         
    }
 
    // 此处甚至可以不需要设置Looper，因为Handler默认就使用当前线程的Looper
    private final Handler handler = new Handler(Looper.getMainLooper()) {
 
        public void handleMessage(Message msg) { // 处理Message，更新ListView
            int state = msg.getData().getInt("state");
            switch(state){
                case STATE_FINISH:
                    dismissDialog(PROGRESS_DIALOG);
                    Toast.makeText(getApplicationContext(),
                            "加载完成!",
                            Toast.LENGTH_LONG)
                         .show();
 
                    adapter = new ArrayAdapter<String>(getApplicationContext(),
                            android.R.layout.simple_list_item_1,
                            data );
                             
                    setListAdapter(adapter);
 
                    break;
 
                case STATE_ERROR:
                   dismissDialog(PROGRESS_DIALOG);
                   Toast.makeText(getApplicationContext(),
                           "处理过程发生错误!",
                           Toast.LENGTH_LONG)
                        .show();
 
                   adapter = new ArrayAdapter<String>(getApplicationContext(),
                           android.R.layout.simple_list_item_1,
                           data );
                            
                      setListAdapter(adapter);
 
                      break;
 
               default:
 
            }
        }
    };
 
 
    private ArrayAdapter<String> adapter;
    private ArrayList<String> data;
 
    private static final int PROGRESS_DIALOG = 1;
    private static final int STATE_FINISH = 1;
    private static final int STATE_ERROR = -1;
}

这个例子，我自己写完后觉得还是有点乱，要稍微整理才能看明白线程间交互的过程以及数据的前后变化。随后了解到AsyncTask类，相应修改后就很容易明白了！
 
#### AsyncTask

AsyncTask版：

	((Button) findViewById(R.id.load_AsyncTask)).setOnClickListener(new View.OnClickListener(){
 
    @Override
    public void onClick(View view) {
        data = null;
        data = new ArrayList<String>();
 
        adapter = null;
 
        //显示ProgressDialog放到AsyncTask.onPreExecute()里
        //showDialog(PROGRESS_DIALOG);
        new ProgressTask().execute(data);
    }
});
 
	private class ProgressTask extends AsyncTask<ArrayList<String>, Void, Integer> {
 
	/* 该方法将在执行实际的后台操作前被UI thread调用。可以在该方法中做一些准备工作，如在界面上显示一个进度条。*/
	@Override
	protected void onPreExecute() {
    	// 先显示ProgressDialog
    	showDialog(PROGRESS_DIALOG);
	}
 
	/* 执行那些很耗时的后台计算工作。可以调用publishProgress方法来更新实时的任务进度。 */
	@Override
	protected Integer doInBackground(ArrayList<String>... 	datas) {
    	ArrayList<String> data = datas[0];
    	for (int i=0; i<8; i++) {
        	data.add("ListItem");
    	}
    	return STATE_FINISH;
	}
 
	/* 在doInBackground 执行完成后，onPostExecute 方法将被UI thread调用，后台的计算结果将通过该方法传递到UI thread.*/
	@Override
	protected void onPostExecute(Integer result) {
    	int state = result.intValue();
    	switch(state){
    	case STATE_FINISH:
        	dismissDialog(PROGRESS_DIALOG);
        		Toast.makeText(getApplicationContext(),
                	"加载完成!",
                	Toast.LENGTH_LONG)
             		.show();
 
        	adapter = new ArrayAdapter<String>(getApplicationContext(),
                android.R.layout.simple_list_item_1,
                data );
                 
        	setListAdapter(adapter);
 
        	break;
         
    	case STATE_ERROR:
       		dismissDialog(PROGRESS_DIALOG);
       		Toast.makeText(getApplicationContext(),
               "处理过程发生错误!",
               Toast.LENGTH_LONG)
            	.show();
 
       		adapter = new ArrayAdapter<String>(getApplicationContext(),
               android.R.layout.simple_list_item_1,
               data );
 
          	setListAdapter(adapter);
 
          	break;
          	default:break;
        }
	}
	
Android另外提供了一个工具类：AsyncTask。它使得UI thread的使用变得异常简单。它使创建需要与用户界面交互的长时间运行的任务变得更简单，不需要借助线程和Handler即可实现。

1. 子类化AsyncTask
2. 实现AsyncTask中定义的下面一个或几个方法
	1. onPreExecute() 开始执行前的准备工作；
    2. doInBackground(Params...) 开始执行后台处理，可以调用publishProgress方法来更新实时的任务进度；
    3. onProgressUpdate(Progress...)  在publishProgress方法被调用后，UI thread将调用这个方法从而在界面上展示任务的进展情况，例如通过一个进度条进行展示。
    4. onPostExecute(Result) 执行完成后的操作，传送结果给UI 线程。
 
这4个方法都不能手动调用。而且除了doInBackground(Params...)方法，其余3个方法都是被UI线程所调用的，所以要求：
    
1. AsyncTask的实例必须在UI thread中创建；
2. AsyncTask.execute方法必须在UI thread中调用；
        
同时要注意：该task只能被执行一次，否则多次调用时将会出现异常。而且是不能手动停止的，这一点要注意，看是否符合你的需求！
 
在使用过程中，发现AsyncTask的构造函数的参数设置需要看明白：

	AsyncTask<Params, Progress, Result>
	
1. Params对应doInBackground(Params...)的参数类型。而new AsyncTask().execute(Params... params)，就是传进来的Params数据，你可以execute(data)来传送一个数据，或者execute(data1, data2, data3)这样多个数据。
2. Progress对应onProgressUpdate(Progress...)的参数类型；
3. Result对应onPostExecute(Result)的参数类型。

**当以上的参数类型都不需要指明某个时，则使用`Void`，注意不是`void`。**不明白的可以参考上面的例子，或者API Doc里面的例子。

转载自百度空间 zhao_xu_dong的分享，原文地址为：http://apps.hi.baidu.com/share/detail/31067249。

感谢原文作者的无私分享。
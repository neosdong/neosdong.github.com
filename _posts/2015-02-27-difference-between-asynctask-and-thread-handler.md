---
layout: post
title: AsyncTask与Thread加Handler的区别
category : Android
tags : Android Java 多线程

---
Thread+Handler机制，Thread通过handler向主线程发送消息、传递数据，来更新ＵＩ，但是大量的子线程会分享主线程的系统资源，从而会使主线程因资源受限而导致应用性能降低，更好的方法是使用AsyncTask。

## Thread+Handler

### 需要自己实现同步和线程池机制

Thread是Java早期为实现多线程而设计的，比较简单不支持concurrent中很多特性在同步和线程池类中需要自己去实现很多的东西，对于分布式应用来说更需要自己写调度代码，而为了Android UI的刷新google引入了Handler和Looper机制，它们均基于消息实现，有可能消息队列阻塞或其他原因无法准确的使用。

要实现线程池要用Runnable。参考：

* [介绍new Thread的弊端及Java四种线程池的使用，对Android同样适用，包括无限线程池,定长线程池,定时周期线程池,单线程池。](http://www.trinea.cn/android/java-android-thread-pool/
)
* Thinking in Java:线程池


## AsyncTask

Android 1.5提供了一个工具类：AsyncTask，它使创建需要与用户界面交互的长时间运行的任务变得更简单。相对来说AsyncTask更轻量级一些，适用于简单的异步处理，不需要借助线程和Handler即可实现。

AysncTask内部实现了线程池。

AsyncTask是抽象类.AsyncTask定义了三种泛型类型 Params，Progress和Result。

* Params 启动任务执行的输入参数，比如HTTP请求的URL。
* Progress 后台任务执行的百分比。通常是Integer类型。
* Result 后台执行任务最终返回的结果，比如String,Bitmap。

比如下载图片的任务：

`class ImageDownloadTask extends AsyncTask<String, Integer, Bitmap> {}`

关于AsyncTask的使用见之前的文章：[link](http://neosdong.github.io/2014/12/11/how-to-use-android-thread.html#title16
)
 
## 初步了解AsyncTask的实现

在android.os包

### Feild

* sPoolWorkQueue队列，存储Runnable
* Executor THREAD_POOL_EXECUTOR，并发线程池
* Executor SERIAL_EXECUTOR，序列执行的线程池，是SerialExecutor类的实例对象。

### SerialExecutor

AsyncTask第227行，execute函数

* 封装一个匿名Runnable
  1. 执行传进来的Runnable r
  1. finally，从mTask队列取出一个Runnable给并行线程池执行
  
这个机制怎样分析？

大概是先用单线程处理，执行异常了才通过在序列中拿出来给并行线程执行。这个应该是AsyncTask的处理线程的核心算法吧。

### 使用Handler传送doInBackground返回值

详见(316)postResult的实现

    private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = sHandler.obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }

由(637)InternalHandler接收，根据消息内容分别发给：结果回调函数finish和进度回调onProgressUpdate
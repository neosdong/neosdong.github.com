---
layout: post
title: Android AsyncTask源码分析
category : Android
tags : Android 多线程

---

相信搞过android开发的朋友们都不陌生。AsyncTask内部封装了Thread和Handler，可以让我们在后台进行计算并且把计算的结果及时更新到UI上，而这些正是Thread+Handler所做的事情，没错，AsyncTask的作用就是简化Thread+Handler，让我们能够通过更少的代码来完成一样的功能，这里，我要说明的是：AsyncTask只是简化Thread+Handler而不是替代，实际上它也替代不了。

转载自：http://blog.csdn.net/singwhatiwanna/article/details/17596225

## AsyncTask使用示例

~~~

	private class DownloadFilesTask extends AsyncTask<URL, Integer, Long> {  
     	protected Long doInBackground(URL... urls) {  
         	int count = urls.length;  
         	long totalSize = 0;  
         	for (int i = 0; i < count; i++) {  
             	totalSize += Downloader.downloadFile(urls[i]);  
             	publishProgress((int) ((i / (float) count) * 100));  
             	// Escape early if cancel() is called  
             	if (isCancelled()) break;  
         	}  
         	return totalSize;  
     	}  
  
     	protected void onProgressUpdate(Integer... progress) {  
        	setProgressPercent(progress[0]);  
     	}  
  
     	protected void onPostExecute(Long result) {  
         	showDialog("Downloaded " + result + " bytes");  
     	}  
 	}

~~~
  

## 使用AsyncTask的规则

* AsyncTask的类必须在UI线程加载（从4.1开始系统会帮我们自动完成）
* AsyncTask对象必须在UI线程创建
* execute方法必须在UI线程调用
* 不要在你的程序中去直接调用onPreExecute(), onPostExecute, doInBackground, onProgressUpdate方法
* 一个AsyncTask对象只能执行一次，即只能调用一次execute方法，否则会报运行时异常
* AsyncTask不是被设计为处理耗时操作的，耗时上限为几秒钟，如果要做长耗时操作，强烈建议你使用Executor，ThreadPoolExecutor以及FutureTask
* 在1.6之前，AsyncTask是串行执行任务的，1.6的时候AsyncTask开始采用线程池里处理并行任务，但是从3.0开始，为了避免AsyncTask所带来的并发错误，AsyncTask又采用一个线程来串行执行任务

## AsyncTask到底是串行还是并行？
给大家做一下实验，请看如下实验代码：代码很简单，就是点击按钮的时候同时执行5个AsyncTask，每个AsyncTask休眠3s，同时把每个AsyncTask执行结束的时间打印出来，这样我们就能观察出到底是串行执行还是并行执行。

~~~

    @Override
    public void onClick(View v) {
        if (v == mButton) {
            new MyAsyncTask("AsyncTask#1").execute("");
            new MyAsyncTask("AsyncTask#2").execute("");
            new MyAsyncTask("AsyncTask#3").execute("");
            new MyAsyncTask("AsyncTask#4").execute("");
            new MyAsyncTask("AsyncTask#5").execute("");
        }

    }

    private static class MyAsyncTask extends AsyncTask<String, Integer, String> {

        private String mName = "AsyncTask";

        public MyAsyncTask(String name) {
            super();
            mName = name;
        }

        @Override
        protected String doInBackground(String... params) {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return mName;
        }

        @Override
        protected void onPostExecute(String result) {
            super.onPostExecute(result);
            SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            Log.e(TAG, result + "execute finish at " + df.format(new Date()));
        }
    }

~~~

我找了2个手机，系统分别是4.1.1和2.3.3，按照我前面的描述，AsyncTask在4.1.1应该是串行的，在2.3.3应该是并行的，到底是不是这样呢？请看Log

Android 4.1.1上执行：从下面Log可以看出，5个AsyncTask共耗时15s且时间间隔为3s，很显然是串行执行的

~~~

D/MainActivity$MyAsyncTask.onPostExecute(L:53)﹕ AsyncTask#1execute finish at 2015-07-05 08:21:12
D/MainActivity$MyAsyncTask.onPostExecute(L:53)﹕ AsyncTask#2execute finish at 2015-07-05 08:21:15
D/MainActivity$MyAsyncTask.onPostExecute(L:53)﹕ AsyncTask#3execute finish at 2015-07-05 08:21:18
D/MainActivity$MyAsyncTask.onPostExecute(L:53)﹕ AsyncTask#4execute finish at 2015-07-05 08:21:21
D/MainActivity$MyAsyncTask.onPostExecute(L:53)﹕ AsyncTask#5execute finish at 2015-07-05 08:21:24

~~~

Android2.3.7上执行，从下面Log可以看出，5个AsyncTask的结束时间是一样的，很显然是并行执行

~~~

D/MainActivity$MyAsyncTask.onPostExecute(L:63)﹕ AsyncTask#1execute finish at 2015-07-05 08:40:59
D/MainActivity$MyAsyncTask.onPostExecute(L:63)﹕ AsyncTask#2execute finish at 2015-07-05 08:40:59
D/MainActivity$MyAsyncTask.onPostExecute(L:63)﹕ AsyncTask#3execute finish at 2015-07-05 08:40:59
D/MainActivity$MyAsyncTask.onPostExecute(L:63)﹕ AsyncTask#4execute finish at 2015-07-05 08:40:59
D/MainActivity$MyAsyncTask.onPostExecute(L:63)﹕ AsyncTask#5execute finish at 2015-07-05 08:40:59

~~~

结论：从上面的两个Log可以看出，我前面的描述是完全正确的。下面请看源码，让我们去了解下其中的原理。

<!--
## 预备知识

### 双向队列

~~~



~~~

-->

## 源码分析（API22）

AsyncTask-API22-源码阅读 : https://gist.github.com/neosdong/610896f44ff7665c3f8e

调用过程

1. 构造函数
1. execute->executeOnExecutor->onPreExecute
1. onPreExecute
1. doInBackground返回结果->postResult通过MESSAGE_POST_RESULT消息发送返回值->`InternalHandler.handleMessage`->`finish(private)`->`onCancelled or onPostExecute`->`mStatus = Status.FINISHED`
1. onPostExecute
1. onCancelled

~~~

    private void finish(Result result) {
        if (isCancelled()) {
            onCancelled(result);
        } else {
            onPostExecute(result);
        }
        mStatus = Status.FINISHED;
    }
~~~


其他

* 实现`Executor`构建串行执行器

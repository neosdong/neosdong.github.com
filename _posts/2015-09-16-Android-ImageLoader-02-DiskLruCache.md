---

layout: post
title: 构建Android AdapterView的网络图片加载框架（二）：本地缓存
category : Android
tags : Android 多线程 本地缓存

---

只有内存缓存会导致内存升高，如果有电话接入（优先级高的任务）等情况，则APP被kill。重新打开又需要做新的网络请求。所以，要添加本地缓存。

## 封装本地缓存工具类BitmapDiskCache

类似内存缓存，要包含两个关键方法：

1. 获取bitmap:getBitmapFromDiskCache
1. 添加bitmap:addBitmapToDiskCache

然后，还想到郭林大神介绍DiskLruCache的文章：[Android DiskLruCache完全解析，硬盘缓存的最佳方案](http://blog.csdn.net/guolin_blog/article/details/28863651)

1. 介绍了如何添加DiskLruCache到项目
1. 详细介绍了DiskLruCache增删查改的方法

## 多线程操作

根据Google官方文档[Caching Bitmaps](http://developer.android.com/training/displaying-bitmaps/cache-bitmap.html#disk-cache)的建议，

1. 磁盘操作最好放在子线程中进行。
1. 磁盘读写工具可以用`DiskLruCache`，或者`ContentProvider`
   1. `DiskLruCache`，文档中的Sample Code所用，实现基本的缓存读写管理。
   1. `ContentProvider`适用于高频读写，例如AdapterView中的图片。因为`ContentProvider`自带队列与线程优化的属性。

Sample Code

~~~

private DiskLruCache mDiskLruCache; 
private final Object mDiskCacheLock = new Object(); 
private boolean mDiskCacheStarting = true; 
private static final int DISK_CACHE_SIZE = 1024 * 1024 * 10; // 10MB 
private static final String DISK_CACHE_SUBDIR = "thumbnails"; 
 
@Override 
protected void onCreate(Bundle savedInstanceState) { 
    ... 
    // Initialize memory cache 
    ... 
    // Initialize disk cache on background thread 
    File cacheDir = getDiskCacheDir(this, DISK_CACHE_SUBDIR); 
    new InitDiskCacheTask().execute(cacheDir); 
    ... 
} 
 
class InitDiskCacheTask extends AsyncTask<File, Void, Void> {
    @Override 
    protected Void doInBackground(File... params) {
        synchronized (mDiskCacheLock) { 
            File cacheDir = params[0]; 
            mDiskLruCache = DiskLruCache.open(cacheDir, DISK_CACHE_SIZE); 
            mDiskCacheStarting = false; // Finished initialization 
            mDiskCacheLock.notifyAll(); // Wake any waiting threads 
        } 
        return null; 
    } 
} 
 
class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    ... 
    // Decode image in background. 
    @Override 
    protected Bitmap doInBackground(Integer... params) {
        final String imageKey = String.valueOf(params[0]);
 
        // Check disk cache in background thread 
        Bitmap bitmap = getBitmapFromDiskCache(imageKey);
 
        if (bitmap == null) { // Not found in disk cache
            // Process as normal 
            final Bitmap bitmap = decodeSampledBitmapFromResource(
                    getResources(), params[0], 100, 100));
        } 
 
        // Add final bitmap to caches 
        addBitmapToCache(imageKey, bitmap);
 
        return bitmap;
    } 
    ... 
} 
 
public void addBitmapToCache(String key, Bitmap bitmap) {
    // Add to memory cache as before 
    if (getBitmapFromMemCache(key) == null) {
        mMemoryCache.put(key, bitmap);
    } 
 
    // Also add to disk cache 
    synchronized (mDiskCacheLock) { 
        if (mDiskLruCache != null && mDiskLruCache.get(key) == null) { 
            mDiskLruCache.put(key, bitmap); 
        } 
    } 
} 
 
public Bitmap getBitmapFromDiskCache(String key) {
    synchronized (mDiskCacheLock) { 
        // Wait while disk cache is started from background thread 
        while (mDiskCacheStarting) { 
            try { 
                mDiskCacheLock.wait(); 
            } catch (InterruptedException e) {} 
        } 
        if (mDiskLruCache != null) { 
            return mDiskLruCache.get(key); 
        } 
    } 
    return null; 
} 
 
// Creates a unique subdirectory of the designated app cache directory. Tries to use external 
// but if not mounted, falls back on internal storage. 
public static File getDiskCacheDir(Context context, String uniqueName) {
    // Check if media is mounted or storage is built-in, if so, try and use external cache dir 
    // otherwise use internal cache dir 
    final String cachePath =
            Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState()) ||
                    !isExternalStorageRemovable() ? getExternalCacheDir(context).getPath() :
                            context.getCacheDir().getPath();
 
    return new File(cachePath + File.separator + uniqueName);
} 

~~~

Sample Code是在Activity直接实现的。缺少了必要的封装。但是，可以GET的信息是：

1. 封装本地缓存工具类，除了获取Bitmap、添加Bitmap的方法之外，还要给两者及开启本地缓存方法加上一个异步操作的壳。
1. 继续使用`AsyncTask`做异步任务的管理，免去自定义线程管理的工作。
1. 关于Java同步代码块，锁的使用
1. 需求不同：
	1. Sample Code的`BitmapWorkerTask`仅仅完成从本地缓存获取Bitmap，如果Bitmap存在，则添加到内存缓存。
	1. 我的目标是将内存和本地的读写方法分离。

### 同步代码块与锁的分析

线程锁使其他线程处于等待状态：`InitDiskCacheTask`首先运行，其他使用`mDiskCacheLock`作为线程锁的同步代码块的线程，例如`BitmapWorkerTask`后台操作的同步代码块的线程，处理等待状态。

然后，本地缓存开启完毕，`InitDiskCacheTask`的`mDiskCacheLock.notifyAll();`其他使用`mDiskCacheLock`作为线程锁的同步代码块的线程。

关于	wait方法的说明

~~~
     * <p>A waiting thread can be sent {@code interrupt()} to cause it to
     * prematurely stop waiting, so {@code wait} should be called in a loop to
     * check that the condition that has been waited for has been met before
     * continuing.
~~~
 
大概意思是要一个有标识位的循环来保持线程的wait状态，于是有这段**等待本地缓存开启完毕，然后执行获取缓存中的bitmap的代码**。<!--【TEST01,AsyncTask默认只有一个线程，要用自定义线程池进一步测试后续代码运行效果】-->

~~~

    synchronized (mDiskCacheLock) { 
        // Wait while disk cache is started from background thread 
        while (mDiskCacheStarting) { 
            try { 
                mDiskCacheLock.wait(); 
            } catch (InterruptedException e) {} 
        } 
        if (mDiskLruCache != null) { 
            return mDiskLruCache.get(key); 
        } 
    } 

~~~





## 用线程封装的本地缓存工具类

~~~

    /***** Start of CRUD Task********/
    public InitDiskCacheTask initDiskCacheTask(){
        return new InitDiskCacheTask();
    }

    public class InitDiskCacheTask extends AsyncTask<Void,Void,Void>{

        public InitDiskCacheTask() {
        }

        @Override
        protected Void doInBackground(Void... params) {
            synchronized (mDiskCacheLock){
                open(1);
                isDiskCacheStarting = false;
                mDiskCacheLock.notifyAll();
            }
            return null;
        }
    }

    public AddBitmapTask addBitmapTask(String url,Bitmap bitmap){
        return new AddBitmapTask(url,bitmap);
    }

    class AddBitmapTask extends AsyncTask<Void,Integer,Void>{
        private String mUrl;
        private Bitmap mBitmap;

        public AddBitmapTask(String url,Bitmap bitmap) {
            mUrl = url;
            mBitmap = bitmap;
        }

        @Override
        protected Void doInBackground(Void... params) {
            synchronized (mDiskCacheLock){
                String key = hashKeyForDisk(mUrl);
                addBitmapToDiskCache(key, mBitmap);
            }
            return null;
        }
    }

    public GetBitmapTask getBitmapTask(String url,Handler uiAsyncHandler){
        return new GetBitmapTask(url,uiAsyncHandler);
    }



    class GetBitmapTask extends AsyncTask<Void,Integer,Bitmap>{

        private String mUrl;
        private Handler uiAsycHandler;

        public GetBitmapTask(String url,Handler uiAsycHandler) {
            this.uiAsycHandler = uiAsycHandler;
            this.mUrl = url;
        }

        @Override
        protected Bitmap doInBackground(Void... params) {
            Bitmap bitmap = null;
            synchronized (mDiskCacheLock){
                while (isDiskCacheStarting){
                    try {
                        mDiskCacheLock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                String key = hashKeyForDisk(mUrl);
                bitmap = getBitmapFromDiskCache(key);
            }
            return bitmap;
        }


        @Override
        protected void onPostExecute(Bitmap bitmap) {
            super.onPostExecute(bitmap);
            Message msg = new Message();
            if (bitmap!=null){
                msg.what = RESULT_GET_BITMAP_SUCCESS;
                msg.obj = new BitmapHolder(mUrl,bitmap);
                uiAsycHandler.sendMessage(msg);
            }else {
                msg.what = RESULT_GET_BITMAP_NULL;
                uiAsycHandler.sendEmptyMessage(RESULT_GET_BITMAP_NULL);
            }
        }
    }




    /***** End of CRUD Task********/
	

~~~

## 其他

### 调试

* 给磁盘缓存的关键操作输出Log。测试步骤是否复合预想。

### 注意

* 注意写入数据后要调用Flush方法

<!--
~~~



~~~


* 【TEST01】loadBitmap的方法使用多线程
* 【TEST00】功能测试：
	1. 在线加载图片，断点测试是否有添加图片
	1. 离线后重新开启是否能直接加载图片
	1. 注意写入数据后要调用Flush方法
* 【TEST02】测试屏幕翻转
	
-->



## 参考

* [Java Thread wait, notify and notifyAll Example | JournalDev](http://www.journaldev.com/1037/java-thread-wait-notify-and-notifyall-example)
* [Android DiskLruCache完全解析，硬盘缓存的最佳方案 - 郭霖的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/guolin_blog/article/details/28863651)
* [Caching Bitmaps | Android Developers](http://developer.android.com/training/displaying-bitmaps/cache-bitmap.html#disk-cache) 

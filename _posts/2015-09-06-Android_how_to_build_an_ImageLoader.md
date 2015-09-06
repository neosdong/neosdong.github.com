---
layout: post
title: 构建Android AdapterView的网络图片加载框架（一）
category : Android
tags : Android 多线程 HTTP请求

---

预备知识，AsyncTask的使用和线程特性。参考之前的文章，《[Android AsyncTask源码分析](/2015/02/28/AsyncTask%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.html)》

## 热身：单张图片的异步加载

`AsyncTask`类提供了一个在后台线程执行一些操作的简单方法，它还可以把后台的执行结果呈现到UI线程中。

1. 使用AsyncTask(Use a AsyncTask)
2. 使用WeakReference引用ImageView，防止异步加载回调时，ImageView已经不被UI使用，要被回收，却因为被AsyncTask持有而无法被GC回收。

为ImageView使用WeakReference确保了AsyncTask所引用的资源可以被垃圾回收器回收。由于当任务结束时不能确保ImageView仍然存在，因此我们**必须在onPostExecute()里面对引用进行检查**。该ImageView在有些情况下可能已经不存在了，例如，在任务结束之前用户使用了回退操作，或者是配置发生了改变（如旋转屏幕等）。

~~~
public class BitmapWorkerTask extends AsyncTask<String,Integer,Bitmap>{
    private WeakReference<ImageView> mImageViewWeakReference;
    private String mUrl;

    public BitmapWorkerTask(ImageView imageView){
        mImageViewWeakReference = new WeakReference<ImageView>(imageView);
    }

    @Override
    protected Bitmap doInBackground(String... url) {
        mUrl = url[0];
        return loadBitmapFromUrl(mUrl);
    }



    @Override
    protected void onPostExecute(Bitmap bitmap) {
        super.onPostExecute(bitmap);
        if (bitmap!=null&&mImageViewWeakReference!=null){
            ImageView imageView = mImageViewWeakReference.get();
            imageView.setImageBitmap(bitmap);
        }
    }

    /**
     * strSrc->URL->HttpURLConnection->InputStream->Bitmap
     * @param strSrc
     * @return
     */
    private Bitmap loadBitmapFromUrl(String strSrc) {
        //TODO
        try {
            URL url = new URL(strSrc);
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setDoInput(true);
            conn.connect();

            InputStream is = conn.getInputStream();
            Bitmap bitmap = BitmapFactory.decodeStream(is);
            return bitmap;
        } catch (Exception e) {
            Log.e(getClass().getSimpleName(),e.getMessage());
        }
        return null;
    }
}

~~~

## AdapterView的图片加载

### 一、直接在getView使用单张图片的加载方法

AdapterView的ITEMS有复用机制，getView如果使用了convertView则滚动过程中，ITEMS的View是不会重新构建，只是内容根据列表项数据有所修改。

于是，会出现图片错位，然后闪烁到正确图片的现象。

参考：关于ListView复用错位的原理和针对复用错误的解决方法：[本文主要分析Android ListView滚动过程中图片显示重复、错乱、闪烁的原因及解决方法，顺带提及ListView的缓存机制](http://www.trinea.cn/android/android-listview-display-error-image-when-scroll/) 

### 二、setTag给ImageView绑定对应URL

因为AdapterView的ITEMS的复用，滚动显示新ITEMS到对应图片加载完成前有由旧图（之前显示的图）闪烁到新图的现象。

解决步骤一：

在getView的实现中，在Bitmap开始加前给ImageView加上空白图片

~~~
BitmapWorkerTask{//...

    public BitmapWorkerTask(Context context,ImageView imageView){
        //set the init default image
        imageView.setImageDrawable(ContextCompat.getDrawable(context, R.drawable.blank));

        //weak reference in case of the ImageView has been destroyed while callback
        mImageViewWeakReference = new WeakReference<ImageView>(imageView);
    }


~~~

滚动快了还是会有闪烁，是因为ListView的缓存机制导致，我们看到的都是缓存的ITEMS。一个缓存的ITEMS可能调用了多次getView从而异步加载多次图片，最后一次才是正确的

解决步骤二：

每次getView都给ImageView setTag(url)，异步回调判断url匹配才setImageBitmap。调用的setTag的url参数可定是当前可见的，因为是顺序执行的，可以理解为可见就立即setTag。

~~~

getView{
	//...
    iv.setTag(mStringsUrl[position]);
    bitmapWorkerTask.execute(mStringsUrl[position]);
}

BitmapWorkerTask{
    //...

    @Override
    protected void onPostExecute(Bitmap bitmap) {
        super.onPostExecute(bitmap);
        if (bitmap!=null&&mImageViewWeakReference!=null){
            ImageView imageView = mImageViewWeakReference.get();
            if (imageView!=null&&imageView.getTag().equals(mUrl))
                imageView.setImageBitmap(bitmap);
        }
        //TODO fail to load the bitmap,should make a call back.
    }
}


~~~

以上代码封装一下：

~~~
getView{
    ...
    bitmapWorkerTask.loadBitmap(iv,mStringsUrl[position]);
}

BitmapWorkerTask{
    //...

    public void loadBitmap(ImageView imageView,String url){
        imageView.setTag(url);
        this.execute(url);
    }
}
~~~

于是，`getView`的代码没有任何改变，这个库可以解决图片错位的问题了。

### 三、cancel重复的请求

从显示上，这样的库是没有问题的。问题是，以上算法只是在请求回调时去掉无效的setBitmap方法（通过url过来），没有在源头解决问题，无效的请求依然运行下去。

解决思路：

1. 在loadBitmap方法里判断是否已经存在不用于显示的任务。优先处理要显示的数据请求，不用于显示的数据请求先简单cancel。
2. 如何判断？简单点说，就是ImageView是否存在任务，自然就想到setXXX绑定一些信息了。
3. 已经给ImageView setTag了。对！还有一个`public void setImageDrawable(Drawable drawable)`方法，给ImageView绑定一个`BitmapDrawable`对象。

步骤一：

构建`BitmapDrawable`的子类`AsyncBitmapDrawable`：

1. 持有信息：任务信息`WeakReference<BitmapWorkerTask>`。使用弱引用也是为了Task结束后（例如，列表图片加载完，并在不滚动状态）能被GC回收。
2. 获取BitmapWorkerTask的方法，在loadBitmap方法做判断时使用。

~~~

AsyncBitmapDrawable.java

public class AsyncBitmapDrawable extends BitmapDrawable{


    private WeakReference<BitmapWorkerTask> mBitmapWorkerTaskWeakReference;

    public AsyncBitmapDrawable(Resources res, Bitmap bitmap, WeakReference mBitmapWorkerTaskWeakReference) {
        super(res, bitmap);
        this.mBitmapWorkerTaskWeakReference = mBitmapWorkerTaskWeakReference;
    }

    public BitmapWorkerTask getBitmapWorker() {
        return mBitmapWorkerTaskWeakReference.get();
    }
}

~~~

步骤二：

在loadBitmap方法，启动Task之前做判断。

* 通过URL是否一致，判断是否要创建新的Task。
  * 一致则不再创建新Task。
  * 如果是新的URL，则判断ImageView是否存在旧Task，如果存在，则cancel掉。 
    * 最后，创建新Task。

~~~
private boolean cancelPotencialTask(ImageView imageView) {
    if (imageView.getTag()==mUrl){
        //this ImageView has the task of the some mUrl, then go on the task
        return false;
    }else {
        Drawable drawable = imageView.getDrawable();
        if (drawable!=null&&drawable instanceof AsyncBitmapDrawable){
            ((AsyncBitmapDrawable) drawable).getBitmapWorker().cancel(true);
        }
        return true;
    }
}
~~~

思路通过代码落实了。下面调试测试一下：

* 【测试1】缓慢滚动不会cancel task，而滚动超过一屏（缓存ITEM Views的数量）则会cancel task。 
   * 因为缓慢滚动不会产生多次请求，超过一屏会有两个或以上的请求。测试通过。
* 【测试2】回调setImage前没有做url判断，可视的ITEM Views没有被两次setImage。因为在loadImage已经对重复的Task做了清理。不过，这个判断还是很有必要的，预防万一。
   * 保留任务回调中的url判断。
* 【测试3】在onCancel方法set一个测试图，会发现滚动超过一屏或者来回滚动就出现测试图，然后被正确的图覆盖，这是因为一个ImageView对应了两个或以上的Task，其中一个被cancel了。
   * 这是对【测试1】对应UI回调做补充，正式代码在`onCancel`用回`placeHolderBitmap`，等待最新的请求回调更新正确的图片。
 
cancel task的回调代码：

~~~  
    @Override
    protected void onCancelled(Bitmap bitmap) {
        super.onCancelled(bitmap);
        ImageView imageView = mImageViewWeakReference.get();
        if (imageView!=null){
            imageView.setImageBitmap(placeHolderBitmap);
        }
    }
~~~

### 四、内存缓存

因为没有缓存，每次刷新（`notifyDataSetChanged`）或者回滚都会重新加载图片。

在loadBitmap方法，优先加载在缓存中的Bitmap。

步骤一：使用LruCache构建缓存工具类

~~~
public class BitmapMemCache {

    private static final int MAX_SIZE = 20*1024*1024;
    LruCache<String ,Bitmap> mLruCache;

    private static BitmapMemCache mInstance = null;

    public static BitmapMemCache getInstance(){
        if (mInstance==null){
            mInstance = new BitmapMemCache();
        }
        return mInstance;
    }

    private BitmapMemCache() {
        mLruCache = new LruCache<>(MAX_SIZE);
    }

    public void addBitmapToMemCache(String url,Bitmap bitmap){
        if (getBitmapFromMemCache(url) == null) {
            mLruCache.put(url, bitmap);
        }
    }

    public Bitmap getBitmapFromMemCache(String url){
        return mLruCache.get(url);
    }
}
~~~

步骤二：loadBitmap方法中优先加载内存缓存中存在的Bitmap

如果已经有缓存的Bitmap，则将cancel当前ImageView对应的其他Task。

步骤三：处理回调关系

1、因为有缓存而取消任务的，onCancel回调方法中不再设置占位符。

~~~
@Override
protected void onCancelled(Bitmap bitmap) {
    super.onCancelled(bitmap);
    ImageView imageView = mImageViewWeakReference.get();
    if (imageView!=null){
        if (mResult == STAT_OTHER_TASK_REQUEST_THE_MISMATCH_URL){
            imageView.setImageBitmap(placeHolderBitmap);
        }
    }
}
~~~

2、补全loadBitmap方式的标识位，使程序结构更统一清晰，便于扩展

~~~

    private int mResult = STAT_NORMAL;
    private static final int STAT_NORMAL = 0x0;
    private static final int STAT_OTHER_TASK_REQUEST_THIS_URL = 0x1;
    private static final int STAT_HAS_MEM_CACHE = 0x1<<1;
    private static final int STAT_FIRST_REQUEST = 0x1<<2;
    private static final int STAT_OTHER_TASK_REQUEST_THE_MISMATCH_URL = 0x1<<3;

    private boolean cancelPotencialTask(ImageView imageView) {

        mBitmapMemCache = BitmapMemCache.getInstance();
        Bitmap bitmap = mBitmapMemCache.getBitmapFromMemCache(mUrl);
        if (bitmap!=null){
            mResult = STAT_HAS_MEM_CACHE;//then start no task and cancel all the other task
            Drawable drawable = imageView.getDrawable();
            if (drawable!=null&&drawable instanceof AsyncBitmapDrawable){
                ((AsyncBitmapDrawable) drawable).getBitmapWorker().cancel(true);
            }
            imageView.setImageBitmap(bitmap);
            return false;
        }


        if (imageView.getTag()==mUrl){
            mResult = STAT_OTHER_TASK_REQUEST_THIS_URL;
            //this ImageView has the task of the some mUrl, then go on the task
            return false;
        }else {
            Drawable drawable = imageView.getDrawable();
            if (drawable!=null&&drawable instanceof AsyncBitmapDrawable){
                BitmapWorkerTask bitmapWorkerTask = ((AsyncBitmapDrawable) drawable).getBitmapWorker();
                if (bitmapWorkerTask!=null){
                    mResult = STAT_OTHER_TASK_REQUEST_THE_MISMATCH_URL;//TODO Maybe Cache? Save Request and CPU,except MEM.SO,save to Disk.
                    bitmapWorkerTask.cancel(true);
                }
            }else {
                //if drawable==null && not AsyncBitmapDrawable , it is said that the ImageView is loading image first time.
                mResult = STAT_FIRST_REQUEST;

            }
            return true;
        }
    }
~~~





## 参考

1. [android-training-course-in-chinese/process-bitmap.md at master · kesenhoo/android-training-course-in-chinese](https://github.com/kesenhoo/android-training-course-in-chinese/blob/master/graphics/displaying-bitmaps/process-bitmap.md)
1. [LruCache Android code example de1 Codota](http://www.codota.com/android/scenarios/52fcbd67da0a69b603b5ede1/android.util.LruCache?tag=dragonfly)

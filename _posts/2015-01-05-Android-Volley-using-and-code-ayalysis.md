---
layout: post
title: Android Volley使用与源码解析
category: Android
tags: Android
keywords: Android
description:
 
---

![](https://raw.githubusercontent.com/neosdong/neosdong.github.com/master/img/20150105-Volley-app.png)

<!--

## Request衍生类的使用

## 自定义Request

### XMLRequest

### GsonRequest

## 图像的加载

### ImageLoader

### NetworkImageView

-->

## Velly源码解析

![](https://raw.githubusercontent.com/neosdong/neosdong.github.com/master/img/20150105-Volley-architecture.png)

### 代码过程

回顾Request类的使用步骤，从获取一个RequestQueue对象开始，看其源代码。

#### 1. 创建一个RequestQueue对象。 

	public static RequestQueue newRequestQueue(Context context) {  
    	return newRequestQueue(context, null);  
	}  

调用的是两个参数的`newRequestQueue()`方法，代码如下：

    public static RequestQueue newRequestQueue(Context context, HttpStack stack) {
        File cacheDir = new File(context.getCacheDir(), DEFAULT_CACHE_DIR);

        String userAgent = "volley/0";
        try {
            String packageName = context.getPackageName();
            PackageInfo info = context.getPackageManager().getPackageInfo(packageName, 0);
            userAgent = packageName + "/" + info.versionCode;
        } catch (NameNotFoundException e) {
        }

        if (stack == null) {
            if (Build.VERSION.SDK_INT >= 9) {
                stack = new HurlStack();
            } else {
                // Prior to Gingerbread, HttpUrlConnection was unreliable.
                // See: http://android-developers.blogspot.com/2011/09/androids-http-clients.html
                stack = new HttpClientStack(AndroidHttpClient.newInstance(userAgent));
            }
        }

        Network network = new BasicNetwork(stack);

        RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network);
        queue.start();

        return queue;
    }

##### 创建HttpStack对象

如果第二个参数为`null`，那么，构造方法会根据系统版本创建`HurlStack`或者`HttpClientStack`对象。

* SDK>=9创建`HurlStack`对象。`HurlStack`类是基于`HttpURLConnection`的`HttpStack`。
* 之前的版本调用`HttpClientStack`,An `HttpStack` that performs request over an `HttpClient`.

刚开始学Android的时候就没有将网络请求的API分的那么细了，为什么这样使用？参考：[Android访问网络，使用HttpURLConnection还是HttpClient？](http://blog.csdn.net/guolin_blog/article/details/12452307
)

`HurlStack`和`HttpClientStack`类都实现了接口`HttpStack`。`HttpStack`接口只有一个抽象方法，**发送带参数的HTTP请求**。

	    public HttpResponse performRequest(Request<?> request, Map<String, String> additionalHeaders)
        throws IOException, AuthFailureError;

##### 创建Network对象

使用`HurlStack`或`HttpClientStack`对象作为参数，创建一个Network对象。

构造方法BasicNetwork（实现Network接口）

    /**
     * @param httpStack HTTP stack to be used
     * @param pool a buffer pool that improves GC performance in copy operations
     */
    public BasicNetwork(HttpStack httpStack, ByteArrayPool pool) {
        mHttpStack = httpStack;
        mPool = pool;
    }

##### 创建RequestQueue对象

参数：
1. 一个DiskBasedCache的对象
2. 一个Network对象

##### RequestQueue.start()

    /**
     * Starts the dispatchers in this queue.
     */
    public void start() {
        stop();  // Make sure any currently running dispatchers are stopped.
        // Create the cache dispatcher and start it.
        mCacheDispatcher = new CacheDispatcher(mCacheQueue, mNetworkQueue, mCache, mDelivery);
        mCacheDispatcher.start();

        // Create network dispatchers (and corresponding threads) up to the pool size.
        for (int i = 0; i < mDispatchers.length; i++) {
            NetworkDispatcher networkDispatcher = new NetworkDispatcher(mNetworkQueue, mNetwork,
                    mCache, mDelivery);
            mDispatchers[i] = networkDispatcher;
            networkDispatcher.start();
        }
    }
    
这里先是创建了一个CacheDispatcher的实例，然后调用了它的start()方法，接着在一个for循环里去创建NetworkDispatcher的实例，并分别调用它们的start()方法。

这里的CacheDispatcher和NetworkDispatcher都是继承自Thread的，而默认情况下for循环会执行四次，也就是说当调用了Volley.newRequestQueue(context)之后，就会有五个线程一直在后台运行，不断等待网络请求的到来，其中CacheDispatcher是缓存线程，NetworkDispatcher是网络请求线程。

#### 2. 创建一个Request对象。

...

#### 3. 将Request对象添加到RequestQueue

例如，`requestQueue.add(stringRequest);`。将一个Request对象添加到RequestQueue。

之前，创建`RequestQueue`对象的结果是，创建了一个缓存线程和默认4个网络请求线程。

接下来，查看`RequestQueue.add(Request request)`的源码：

    /**
     * Adds a Request to the dispatch queue.
     * @param request The request to service
     * @return The passed-in request
     */
    public Request add(Request request) {
        // Tag the request as belonging to this queue and add it to the set of current requests.
        request.setRequestQueue(this);
        synchronized (mCurrentRequests) {
            mCurrentRequests.add(request);
        }

        // Process requests in the order they are added.
        request.setSequence(getSequenceNumber());
        request.addMarker("add-to-queue");

        // If the request is uncacheable, skip the cache queue and go straight to the network.
        if (!request.shouldCache()) {
            mNetworkQueue.add(request);
            return request;
        }

        // Insert request into stage if there's already a request with the same cache key in flight.
        synchronized (mWaitingRequests) {
            String cacheKey = request.getCacheKey();
            if (mWaitingRequests.containsKey(cacheKey)) {
                // There is already a request in flight. Queue up.
                Queue<Request> stagedRequests = mWaitingRequests.get(cacheKey);
                if (stagedRequests == null) {
                    stagedRequests = new LinkedList<Request>();
                }
                stagedRequests.add(request);
                mWaitingRequests.put(cacheKey, stagedRequests);
                if (VolleyLog.DEBUG) {
                    VolleyLog.v("Request for cacheKey=%s is in flight, putting on hold.", cacheKey);
                }
            } else {
                // Insert 'null' queue for this cacheKey, indicating there is now a request in
                // flight.
                mWaitingRequests.put(cacheKey, null);
                mCacheQueue.add(request);
            }
            return request;
        }
    }


`add`方法实现了，添加`Request`到网络请求队列`mNetworkQueue`或者缓存队列`mCacheQueue`。在默认情况下，每条请求都是可以缓存的，当然我们也可以调用`Request`的`setShouldCache(false)`方法来改变这一默认行为。

然后，网络线程和缓存线程分别处理其队列。

##### 缓存线程

缓存线程`CacheDispatcher`继承自Thread，查看其run方法可知缓存线程对缓存队列`mCacheQueue`的处理过程：

    @Override
    public void run() {
        if (DEBUG) VolleyLog.v("start new dispatcher");
        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);

        // Make a blocking call to initialize the cache.
        mCache.initialize();

        while (true) {
            try {
                // Get a request from the cache triage queue, blocking until
                // at least one is available.
                final Request request = mCacheQueue.take();
                request.addMarker("cache-queue-take");

                // If the request has been canceled, don't bother dispatching it.
                if (request.isCanceled()) {
                    request.finish("cache-discard-canceled");
                    continue;
                }

                // Attempt to retrieve this item from cache.
                Cache.Entry entry = mCache.get(request.getCacheKey());
                if (entry == null) {
                    request.addMarker("cache-miss");
                    // Cache miss; send off to the network dispatcher.
                    mNetworkQueue.put(request);
                    continue;
                }

                // If it is completely expired, just send it to the network.
                if (entry.isExpired()) {
                    request.addMarker("cache-hit-expired");
                    request.setCacheEntry(entry);
                    mNetworkQueue.put(request);
                    continue;
                }

                // We have a cache hit; parse its data for delivery back to the request.
                request.addMarker("cache-hit");
                Response<?> response = request.parseNetworkResponse(
                        new NetworkResponse(entry.data, entry.responseHeaders));
                request.addMarker("cache-hit-parsed");

                if (!entry.refreshNeeded()) {
                    // Completely unexpired cache hit. Just deliver the response.
                    mDelivery.postResponse(request, response);
                } else {
                    // Soft-expired cache hit. We can deliver the cached response,
                    // but we need to also send the request to the network for
                    // refreshing.
                    request.addMarker("cache-hit-refresh-needed");
                    request.setCacheEntry(entry);

                    // Mark the response as intermediate.
                    response.intermediate = true;

                    // Post the intermediate response back to the user and have
                    // the delivery then forward the request along to the network.
                    mDelivery.postResponse(request, response, new Runnable() {
                        @Override
                        public void run() {
                            try {
                                mNetworkQueue.put(request);
                            } catch (InterruptedException e) {
                                // Not much we can do about this.
                            }
                        }
                    });
                }

            } catch (InterruptedException e) {
                // We may have been interrupted because it was time to quit.
                if (mQuit) {
                    return;
                }
                continue;
            }
        }
    }

缓存线程的工作机制

* 缓存线程循环运行
* 从消息队列`mCacheQueue`取出一个请求
* 取出响应结果
 * 如果为空则把这条请求加入网络请求队列`mNetworkQueue`
 * 如果不为空，再判断缓存是否过期，如果已经过期则同样把请求加入网络请求
* 否则就认为不需要重发网络请求，直接使用缓存中的数据即可。使用`request.parseNetworkResponse`解析结果。
* 将解析出来的数据进行回调，`mDelivery.postResponse(request, response);`


#### 网络线程

    @Override
    public void run() {
        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
        Request request;
        while (true) {
            try {
                // Take a request from the queue.
                request = mQueue.take();
            } catch (InterruptedException e) {
                // We may have been interrupted because it was time to quit.
                if (mQuit) {
                    return;
                }
                continue;
            }

            try {
                request.addMarker("network-queue-take");

                // If the request was cancelled already, do not perform the
                // network request.
                if (request.isCanceled()) {
                    request.finish("network-discard-cancelled");
                    continue;
                }

                // Tag the request (if API >= 14)
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.ICE_CREAM_SANDWICH) {
                    TrafficStats.setThreadStatsTag(request.getTrafficStatsTag());
                }

                // Perform the network request.
                NetworkResponse networkResponse = mNetwork.performRequest(request);
                request.addMarker("network-http-complete");

                // If the server returned 304 AND we delivered a response already,
                // we're done -- don't deliver a second identical response.
                if (networkResponse.notModified && request.hasHadResponseDelivered()) {
                    request.finish("not-modified");
                    continue;
                }

                // Parse the response here on the worker thread.
                Response<?> response = request.parseNetworkResponse(networkResponse);
                request.addMarker("network-parse-complete");

                // Write to cache if applicable.
                // TODO: Only update cache metadata instead of entire record for 304s.
                if (request.shouldCache() && response.cacheEntry != null) {
                    mCache.put(request.getCacheKey(), response.cacheEntry);
                    request.addMarker("network-cache-written");
                }

                // Post the response back.
                request.markDelivered();
                mDelivery.postResponse(request, response);
            } catch (VolleyError volleyError) {
                parseAndDeliverNetworkError(request, volleyError);
            } catch (Exception e) {
                VolleyLog.e(e, "Unhandled exception %s", e.toString());
                mDelivery.postError(request, new VolleyError(e));
            }
        }
    }
    
* 网络请求线程也不断运行
* 调用Network的performRequest()方法来去发送网络请求。Network是接口，`mNetwork`变量来自创建`RequestQueue`时传入的参数。所以，该方法具体实现是`BasicNetwork`的`performRequest()`方法。


##### `BasicNetwork`的`performRequest()`方法

* 发送网络请求`httpResponse = mHttpStack.performRequest(request, headers);`
* 获取网络请求状态码，做相应处理。
* 如果状态码为200，返回`NetworkResponse`对象。`return new NetworkResponse(statusCode, responseContents, responseHeaders, false);`











### 回顾

我们在主线程中调用RequestQueue的add()方法来添加一条网络请求，这条请求会先被加入到缓存队列当中，如果发现可以找到相应的缓存结果就直接读取缓存并解析，然后回调给主线程。

---

参考：

* [Volley: Easy, Fast Networking for Android — Google I/O 2013](https://developers.google.com/events/io/sessions/325304728)
  * [commondatastorage.googleapis.com/io-2013/presentations/110 - Volley- Easy, Fast Networking for Android.pdf](http://commondatastorage.googleapis.com/io-2013/presentations/110%20-%20Volley-%20Easy,%20Fast%20Networking%20for%20Android.pdf) 
* [Transmitting Network Data Using Volley - Android Developers](http://developer.android.com/training/volley/index.html)
* [Android Volley完全解析(四)，带你从源码的角度理解Volley - 郭霖的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/guolin_blog/article/details/17656437)

<!--
Log

* 20150103 创建。
* 20150105 发布。
-->
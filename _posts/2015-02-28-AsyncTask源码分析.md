---
layout: post
title: Android AsyncTask源码分析
category : Android
tags : Android 多线程

---

相信搞过android开发的朋友们都不陌生。AsyncTask内部封装了Thread和Handler，可以让我们在后台进行计算并且把计算的结果及时更新到UI上，而这些正是Thread+Handler所做的事情，没错，AsyncTask的作用就是简化Thread+Handler，让我们能够通过更少的代码来完成一样的功能，这里，我要说明的是：AsyncTask只是简化Thread+Handler而不是替代，实际上它也替代不了。

## AsyncTask使用示例

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

  

## 使用AsyncTask的规则


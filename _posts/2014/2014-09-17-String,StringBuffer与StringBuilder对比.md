---
layout: post
category : program
tags : [Android, Java]

---


* String 字符串常量
* StringBuffer 字符串变量（线程安全）
* StringBuilder 字符串变量（非线程安全）




## String与StringBuffer

简要的说， String 类型和 StringBuffer 类型的主要性能区别其实在于 String 是不可变的对象, 因此在每次对 String 类型进行改变的时候其实都等同于生成了一个新的 String 对象，然后将指针指向新的 String 对象，所以经常改变内容的字符串最好不要用 String ，因为每次生成对象都会对系统性能产生影响，特别当内存中无引用对象多了以后， JVM 的 GC 就会开始工作，那速度是一定会相当慢的。

经常改变内容的应用场景：文件流的读写，网络数据流的发送接收。

## StringBuffer:补充StringBuilder在多线程的场景（网络、非UI线程的耗时操作）使用
Java.lang.StringBuffer线程安全的可变字符序列。

一个类似于 String 的字符串缓冲区，但不能修改。虽然在任意时间点上它都包含某种特定的字符序列，但通过某些方法调用可以改变该序列的长度和内容。

## StringBuilder:单线程，常改变内容的情况使用

java.lang.StringBuilder一个可变的字符序列是5.0新增的。此类提供一个与 StringBuffer 兼容的 API，但不保证同步。该类被设计用作 StringBuffer 的一个简易替换，用在字符串缓冲区被单个线程使用的时候（这种情况很普遍）。如果可能，建议优先采用该类，因为在大多数实现中，它比 StringBuffer 要快。两者的方法基本相同。

## 参考
* [String,StringBuffer与StringBuilder的区别](http://blog.csdn.net/rmn190/article/details/1492013)



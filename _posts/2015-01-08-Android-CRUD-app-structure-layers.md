---
layout: post
title: Android CURD APP的构建
category : Android
tags : Android

---

## Event

## View Changed

## Model:Bean,Java Data Types

## Structured Data:JSON,XML

## DataSource

数据来源包括网络、文件、数据库。

根据数据分解的步骤，这一层可以再细分。从原始的数据形态转换为**键值对**形式。

### 一、网络

封装层级从上到下，依次为：

1. 表明请求目的
	* 输入：传递参数与CRUD类型
	* 回调：上层数据类型
2. HTTP请求
	* 发送请求
	* 接收响应

### 二、文件

1. 缓存：根据键值对操作
2. 文件读写等

### 三、数据库

1. DAO：键值对层
2. 

<!--
Log
* 20150108 创建。完成这篇可以再写一篇，系列的构建，都是基本数据结构的运用而已。
-->





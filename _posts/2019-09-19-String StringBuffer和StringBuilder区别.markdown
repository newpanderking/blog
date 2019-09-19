---
layout: post
title:  "String StringBuffer 和 StringBuilder的区别"
date: 2019-09-18
categories: 技术
tags: java
description: String StringBuffer 和 StringBuilder 的区别是什么? String 为什么是不可变的?
---

### 1、可变性
> 简单来说，String类中使用final关键字修饰字符数组来保存字符串，`private final char value[]`, 所以String对象是不可变的。而`StringBuilder`与`StringBuffer`都继承自`AbstractStringBuilder`类, 在`AbstractStringBuilder`中也是使用字符数组保存字符串，但是没有使用final关键字修饰，所以这两种对象都是可变的。`StringBuilder`与`StringBuffer`的构造方法都是通过调用父类构造方法也就是`AbstractStringBuilder`实现。

### 2、线程安全性
> String中的对象是不可变的，也可以理解为常量，是线程安全的。`AbstractStringBuilder`是`StringBuilder`与`StringBuffer`的父类，定义了一些字符串的基本操作，如`expandCapacity`、`append`、`insert`、`indexOf`等公共方法。`StringBuffer`对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的， 而`StringBuilder`没有加同步锁，所以是非线程安全的。
 
 
### 3、性能
> 每次对`String`对象进行改变时，都会生成一个新的`String`对象，然后将指针指向新的`String`对象。`StringBuffer`每次都会对`StringBuffer`对象本身进行操作，而不会生成新的对象并改变对象应用。相同情况下使用`StringBuilder`比使用`StringBuffer`能够获得10% - 15%所有的性能提升，但是要冒多线程不安全的风险。

### 4、总结
- 操作少量的数据：适用`String`。
- 单线程操作字符串缓冲区下操作大量数据：适用`StringBuilder`。
- 多线程操作字符串缓冲区下操作大量数据：适用`StringBuffer`。
---
title: "volatile关键字"
date: 2018-04-17
keywords: volatile关键字
tags: Java
author: "zzb"
---


###  volatile的工作原理？
 1.保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。
 2.禁止进行指令重排序。

>注意：
volatile只能保证变量的可见性，并不能保证变量的原子性。

###  什么时候适合用volatile？
 1. 运行结果并不依赖变量的当前值，或者能够确保只有单一的线程修改变量的值。
 2. 变量不需要与其他的状态变量共同参与不变约束。

###  synchronized和volatile对比
 1. 关键字volatile是线程同步的轻量级实现，所以volatile性能肯定比synchronized要好，并且volatile只能修饰变量，而synchronized可以修饰方法，以及代码块。
 2. 多线程访问volatile不会发生阻塞，而synchronized会出现阻塞。
 3. volatile能保证数据的可见性，但不能保证原子性；而synchronized可以保证原子性也可以间接保证可见性，因为它会将私有内存和公共内存中的数据做同步。
 4. volatile解决的是变量在多个线程之间的可见性，而synchronized关键字解决的是多个线程之间访问资源的同步性。

>#### 延伸知识
> * Java内存模型 JMM (主内存和工作内存)
> * 指令重排
> * 内存屏障（四种）
> * 原子变量是一种“更好的volatile”

### 参考
* http://www.importnew.com/18126.html
* https://mp.weixin.qq.com/s/DZkGRTan2qSzJoDAx7QJag
* 《Java多线程编程核心技术》
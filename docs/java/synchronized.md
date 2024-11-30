---
title: "Synchronized关键字"
date: 2018-08-12
keywords: Synchronized
description: ""
tags: Java
author: "zzb"

---
# Synchronized关键字

* 同步代码块

```
package demo1;

public class SynchronizedTest {
    public void method(){
        synchronized (this){
            System.out.println("hello");
        }
    }
}

```

编译成class文件，然后在反编译命令为： javap -c SychronizedTest 或者
javap -verbose SynchronizedTest(更详细)
发现会有以下指令控制

![示例](https://raw.githubusercontent.com/pugongyingbo/pugongyingbo.github.io/master/images/sync2.png)
 
主要是通过monitorenter和monitorexit指令控制， 
monitorexit出现两次是因为第一次为正常退出释放锁，第二次是发生异常退出释放锁。编译器必须确保无论方法通过何种方式完成，方法中调用过的每条monitorenter指令都必须执行其对应的monitorexit指令，而无论这个方法是正常结束还是异常结束。（notify,wait也是通过这俩指令）
 
> JVM要求，在执行monitorenter指令时，首先要尝试获取对象的锁。如果这个对象没有锁定，或者当前线程已经拥有了那个对象的锁，把锁的计数器加1，相应的，执行monitorexit指令时会将锁计数器减1，当计数器为0时，锁就被释放。如果获取对象锁失败，那当前线程就要阻塞等待，知道对象锁被另一个线程释放为止。
 
 * 同步方法
 
```
public class SynchronizedMethod {
    public synchronized void method() {
        System.out.println("Hello World!");
    }
}
```

查看反编译结果：方法的同步并没有通过指令monitorenter和monitorexit来完成，而是根据常量池方法表结构中的ACC_SYNCHRONIZED访问标识符判断。
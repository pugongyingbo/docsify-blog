---
title: "单例模式的几种写法"
date: 2018-04-16
keywords: 单例模式
desc: 
tags: Java设计模式
author: "zzb"
---

## 单例的几种写法
> * 饿汉式
> * 懒汉式
> * 双重检查
> * 使用volitile关键字
> * 枚举

### 懒汉与饿汉
> * 要想让一个类只能构建一个对象，自然不能让他随便去做new操作，因此Singleton的构造方法是私有的。
> * instance是Singleton类的静态成员，也是我们的单利对象。它的初始值可以写成null，也可以写成new Singleton()。
> * getInstance是获取单例对象的方法。
 * 如果单例初始值是null，还未构建，则构建单例对象并返回。这个写法属于单例模式当中的懒汉模式。
 * 如果单例对象一开始就被new Singleton()主动构建，则不再需要判空操作，这种属于饿汉模式。饿汉主动找食物吃，懒汉躺在地上等人喂。

#### 1.饿汉式 （静态常量）
```
public class Singleton {

    private final static Singleton INSTANCE = new Singleton();

    private Singleton(){}

    public static Singleton getInstance(){
        return INSTANCE;
    }
}
```
缺点：在类装载的时候就完成实例化，没有达到Lazy Loading的效果。如果从始至终从未使用过这个实例，则会造成内存的浪费。
### 2.饿汉（静态代码块）

```
public class Singleton {

    private static Singleton instance;

    static {
        instance = new Singleton();
    }

    private Singleton() {}

    public Singleton getInstance() {
        return instance;
    }
}
```
缺点：在类装载的时候就完成实例化，没有达到Lazy Loading的效果。如果从始至终从未使用过这个实例，则会造成内存的浪费。
### 3.懒汉式（线程不安全）
```
public class Singleton {

    private static Singleton singleton;

    private Singleton() {}

    public static Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }
}
```
这种写法起到了Lazy Loading的效果，但是只能在单线程下使用。如果在多线程下，一个线程进入了if (singleton == null)判断语句块，还未来得及往下执行，另一个线程也通过了这个判断语句，这时便会产生多个实例。所以在多线程环境下不可使用这种方式。
### 4.懒汉式（线程安全 同步方法 效率低）
```
public class Singleton {

    private static Singleton singleton;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }
}
```
缺点：效率太低了，每个线程在想获得类的实例时候，执行getInstance()方法都要进行同步。而其实这个方法只执行一次实例化代码就够了，后面的想获得该类实例，直接return就行了。
### 5.懒汉式（线程安全 同步代码块）

```
public class Singleton {

    private static Singleton singleton;

    private Singleton() {}

    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                singleton = new Singleton();
            }
        }
        return singleton;
    }
}
```
由于第四种实现方式同步效率太低，所以摒弃同步方法，改为同步产生实例化的的代码块。但是这种同步并不能起到线程同步的作用。跟第3种实现方式遇到的情形一致，假如一个线程进入了if (singleton == null)判断语句块，还未来得及往下执行，另一个线程也通过了这个判断语句，这时便会产生多个实例。
### 6.双重检查
```
public class Singleton {

    private static volatile Singleton singleton;

    private Singleton() {}

    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```
* 为了防止new Instance被执行多次，因此在new操作之前加上synchronized同步锁，锁住整个类。
* 进入synchronized临界区后，还要再做一次判空，因为当两个线程同时访问的时候，线程a构建完对象，线程b也已经通过了最初的判空验证，不做第二次判空的话，线程b也已经通过了最初的判空验证，不做第二次判空的haul，线程b还是会再次构建instance对象。

> ### 使用volatile关键字
 volatile修饰符阻止了变量访问前后的指令重拍，保证了指令执行顺序。进过volatile的修饰，当线程a执行instance = new Singleton的时候，JVM的执行顺序，
  1、分配对象的内存空间
  2、初始化对象
  3、设置instance指向刚分配的内存地址
 如此在线程b看来，instance对象的引用要么指向null，要么指向一个初始化完毕的instance，而不会出现某个中间态，保证了安全。

### 7.静态内部类
```
public class Singleton {

    private Singleton() {}

    private static class SingletonInstance {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonInstance.INSTANCE;
    }
}
```
这种方式跟饿汉式方式采用的机制类似，但又有不同。两者都是采用了类装载的机制来保证初始化实例时只有一个线程。不同的地方在饿汉式方式是只要Singleton类被装载就会实例化，没有Lazy-Loading的作用，而静态内部类方式在Singleton类被装载时并不会立即实例化，而是在需要实例化时，调用getInstance方法，才会装载SingletonInstance类，从而完成Singleton的实例化。

类的静态属性只会在第一次加载类的时候初始化，所以在这里，JVM帮助我们保证了线程的安全性，在类进行初始化时，别的线程是无法进入的。

优点：避免了线程不安全，延迟加载，效率高。
> 从外部类无法访问静态内部类，只有调用getInstance方法的时候，才能得到单例对象。
INSTANCE对象初始化的时机并不是在单例类Singleton被加载的时候，而是在调用getInstance方法，使得静态内部类LazyHolder被加载的时候。利用classloader的加载机制来实现懒加载，并保证构建单例的线程安全。

### 8.使用枚举
有了enum语法糖，JVM会阻止反射获取枚举类的私有构造函数。

```
public enum SingletonEnum(){
    INSTANCE;
}
```
利用反射打破单例
1、获得构造器
2、把构造器设置为可访问
3、使用getInstance方法构造对象
```
//获得构造器
Constructor con = Singleton.class.getDeclaredConstructor()
//设置可访问
con.setAccsessible(true);
//构造不同对象
Singleton s1 = (Singleton)con.newInstance();
Singleton s2 = (Singleton)con.newInstance();
//验证
System.out.println(s1.equals(s2));//false
```

---
* 总结

|单例实现模式 |是否线程安全 | 是否懒加载 | 是否防止反射构建|
 :-:|:-:|:-:|:-:
 双重检查|是|是|否
 静态内部类|是|是|否
 枚举|是|否|是



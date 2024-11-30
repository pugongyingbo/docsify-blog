---
title: JDK动态代理和CGLib动态代理
date: 2018-05-20
keywords: 
description: 
tags: Java
author: 
---

#### 静态代理

静态代理的缺点很明显：一个代理类只能对一个业务接口的实现类进行包装，如果有多个业务接口的话就要定义很多实现类和代理类才行。而且，如果代理类对业务方法的预处理、调用后操作是一样的，则多个代理类就会有很多重复代码。这是我们可以定义这样一个代理类，它能代理所有实现类的方法调用：根据传进来的业务实现类和方法名进行具体调用。 --这就是动态代理。

#### JDK动态代理

JDK动态代理所用到的代理类在程序调用到代理类对象时才由JVM真正创建，JVM根据传进来的 业务实现类对象 以及 方法名 ，动态地创建了一个代理类的class文件并被字节码引擎执行，然后通过该代理类对象进行方法调用。

> * Spring AOP用到动态代理。
> * 当定义好一个Mapper接口(UserDao)里，我们并不需要去实现这个类，但sqlSession.getMapper()最终会返回一个实现该接口的对象。这个对象是Mybatis利用jdk的动态代理实现的。

* InvocationHandle : 该接口中仅定义了一个方法invoke(Object proxy, Method method, Object[] args)，第一个参数一般指代理类，method是被代理的方法，args为该方法的参数数组。
* proxy ： 动态代理类
    * getProxyClass(ClassLoader loader,Class<?>... interfaces): loader是类装载器，interfaces 是真实类所拥有的全部接口的数组。
    * newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h)：返回一个代理类的一个实例，返回后的代理类可以当做被代理类使用。

* 接口

```
public interface Book {  
    public void addBook();  
    public void deleteBook();  
} 
```

* 实际业务（RealSubject）

```
public class BookImpl implements Book {  
  
    @Override  
    public void addBook() {  
        System.out.println("add book 。。。");   
    }  
  
    @Override  
    public void deleteBook() {  
        System.out.println("delete book 。。。");  
    }  
    
}  
```

* 动态代理类

```
import java.lang.reflect.InvocationHandler;  
import java.lang.reflect.Method;  
import java.lang.reflect.Proxy;  
  
public class BookProxy implements InvocationHandler {  
    private Object target;  
  
    /** 
     *  
     * @param target 
     * @return 
     */  
    public Object bind(Object target) {  
        this.target = target;  
        // 取得代理对象  
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),  
                target.getClass().getInterfaces(), this);  
    }  
  
    @Override  
    public Object invoke(Object proxy, Method method, Object[] args)  
            throws Throwable {  
        Object result=null;  
        System.out.println("Proxy start...");  
        System.out.println("method name:"+method.getName());  
        result=method.invoke(target, args);  
        System.out.println("Proxy end...");  
        return result;  
    }  
  
}  
```

* 测试

```
public class TestProxy {  
  
/** 
 * @param args 
 */  
public static void main(String[] args) {  
    BookProxy proxy = new BookProxy();  
    Book bookProxy = (Book) proxy.bind(new BookImpl());  
    bookProxy.addBook();  
    bookProxy.deleteBook();  
}
```

#### CGLib

JDK的动态代理机制只能代理实现了接口的类，而不能实现接口的类就不能实现JDK的动态代理，cglib是针对类来实现代理的，原理对指定的目标类生成一个子类，并覆盖其中方法实现增强。但因为采用的是继承，所以不能对final修饰的类进行代理。

* 业务类

```
public class Book {
    public void addBook(){
        System.out.println("---addBook---");
    }
}

```

* 代理类

```
import java.lang.reflect.Method;

import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

public class BookProxy implements MethodInterceptor{
    
    private Object target;
    /**
     * 创建代理对象
     * @param target
     * @return
     */
    public Object getInstance(Object target){
        this.target = target;
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(this.target.getClass());
        enhancer.setCallback(this);
        return enhancer.create();
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, 
            MethodProxy proxy) throws Throwable {
        System.out.println("start");
        proxy.invokeSuper(obj, args);
        System.out.println("end");
        return null;
    }

}

```

* 测试

```
public class TestCglib {

    public static void main(String[] args) {
        BookProxy cglib = new BookProxy();
        Book book  = (Book)cglib.getInstance(new Book());
        book.addBook(); 
    }

}
```

> 要导入cglib的jar包，和asm的jar包。没有asm的jar包时报错：java.lang.NoClassDefFoundError: org/objectweb/asm/Type。
> 另外：cglib版本为3.0以上，org.objectweb.asm版本为3.1.0时，版本冲突，报错java.lang.IncompatibleClassChangeError: class net.sf.cglib.core.DebuggingClassWriter has interface org.objectweb.asm.ClassVisitor as super class 。使用cglib 2.2 可解决此问题,该版本中的DebuggingClassWriter的父类为ClassWriter


#### 参考
* https://blog.csdn.net/hintcnuie/article/details/10954631


---
title: "mybatis源码:plugin源码分析"
date: 2018-05-21
keywords: mybatis源码
desc: ""
tags: mybatis
author: ""
---
#### 官方文档解释

MyBatis 允许你在已映射语句执行过程中的某一点进行拦截调用。默认情况下，MyBatis 允许使用插件来拦截的方法调用包括：

* Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
* ParameterHandler (getParameterObject, setParameters)
* ResultSetHandler (handleResultSets, handleOutputParameters)
* StatementHandler (prepare, parameterize, batch, update, query)

这些类中方法的细节可以通过查看每个方法的签名来发现，或者直接查看 MyBatis 发行包中的源代码。 如果你想做的不仅仅是监控方法的调用，那么你最好相当了解要重写的方法的行为。 因为如果在试图修改或重写已有方法的行为的时候，你很可能在破坏 MyBatis 的核心模块。 这些都是更低层的类和方法，所以使用插件的时候要特别当心。

通过 MyBatis 提供的强大机制，使用插件是非常简单的，只需实现 Interceptor 接口，并指定想要拦截的方法签名即可。

#### plugins源码实现

具体使用了JDK动态代理

* Intercept接口

```
package org.apache.ibatis.plugin;

import java.util.Properties;

public interface Interceptor {
     //jdk动态代理中的InvocationHandler.invoke()方法执行里，这个方法会被调用
    Object intercept(Invocation var1) throws Throwable;
    //生成一个代理对象
    Object plugin(Object var1);
    //设置属性  
    void setProperties(Properties var1);
}

```


* Plugin实现Intercept接口

```
public class Plugin implements InvocationHandler {
    //被代理的原始对象
    private final Object target;
    //拦截器
    private final Interceptor interceptor;
    //拦截器对原始对象的哪些方法有效
    private final Map<Class<?>, Set<Method>> signatureMap;

    private Plugin(Object target, Interceptor interceptor, Map<Class<?>, Set<Method>> signatureMap) {
        this.target = target;
        this.interceptor = interceptor;
        this.signatureMap = signatureMap;
    }

    public static Object wrap(Object target, Interceptor interceptor) {
        //对拦截器中取出方法签名，是通过注解来声明需要拦截的方法签名的。
        Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);
        Class<?> type = target.getClass();
        //获取被代理或拦截的对象的所有接口
        Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
        //如果有实现接口，将使用JDK动态代理，如果没有实现任何接口，JDK将无法代理。
        return interfaces.length > 0 ? Proxy.newProxyInstance(type.getClassLoader(), interfaces, new Plugin(target, interceptor, signatureMap)) : target;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        try {
            Set<Method> methods = (Set)this.signatureMap.get(method.getDeclaringClass());
            //如果不空需要拦截，如果为空直接执行
            return methods != null && methods.contains(method) ? this.interceptor.intercept(new Invocation(this.target, method, args)) : method.invoke(this.target, args);
        } catch (Exception var5) {
            throw ExceptionUtil.unwrapThrowable(var5);
        }
    }
    //从注解中读取方法签名
    private static Map<Class<?>, Set<Method>> getSignatureMap(Interceptor interceptor) {
        Intercepts interceptsAnnotation = (Intercepts)interceptor.getClass().getAnnotation(Intercepts.class);
        if (interceptsAnnotation == null) {
            throw new PluginException("No @Intercepts annotation was found in interceptor " + interceptor.getClass().getName());
        } else {
            Signature[] sigs = interceptsAnnotation.value();
            Map<Class<?>, Set<Method>> signatureMap = new HashMap();
            Signature[] var4 = sigs;
            int var5 = sigs.length;
            //未使用foreach语句，是为什么
            for(int var6 = 0; var6 < var5; ++var6) {
                //这两步比老版本多出来的，是为了强转类型么？
                Signature sig = var4[var6];
                Set<Method> methods = (Set)signatureMap.get(sig.type());
                if (methods == null) {
                    methods = new HashSet();
                    signatureMap.put(sig.type(), methods);
                }

                try {
                    Method method = sig.type().getMethod(sig.method(), sig.args());
                    ((Set)methods).add(method);
                } catch (NoSuchMethodException var10) {
                    throw new PluginException("Could not find method on " + sig.type() + " named " + sig.method() + ". Cause: " + var10, var10);
                }
            }

            return signatureMap;
        }
    }

    private static Class<?>[] getAllInterfaces(Class<?> type, Map<Class<?>, Set<Method>> signatureMap) {
        HashSet interfaces;
        for(interfaces = new HashSet(); type != null; type = type.getSuperclass()) {
            Class[] var3 = type.getInterfaces();
            int var4 = var3.length;

            for(int var5 = 0; var5 < var4; ++var5) {
                Class<?> c = var3[var5];
                if (signatureMap.containsKey(c)) {
                    interfaces.add(c);
                }
            }
        }

        return (Class[])interfaces.toArray(new Class[interfaces.size()]);
    }
}
```

* InterceptorChain

```
public class InterceptorChain {
    //在mybatis-config.xml配置的拦截器
    private final List<Interceptor> interceptors = new ArrayList();

    public InterceptorChain() {
    }
    //用所有的拦截器生成对象
    public Object pluginAll(Object target) {
        Interceptor interceptor;
        //调用了interceptor.plugin()方法来生成代理对象，从for-each改成iterator为什么？
        for(Iterator var2 = this.interceptors.iterator(); var2.hasNext(); target = interceptor.plugin(target)) {
            interceptor = (Interceptor)var2.next();
        }

        return target;
    }

    public void addInterceptor(Interceptor interceptor) {
        this.interceptors.add(interceptor);
    }

    public List<Interceptor> getInterceptors() {
        return Collections.unmodifiableList(this.interceptors);
    }
}

```


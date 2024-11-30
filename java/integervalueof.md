---
title: "sonar代码检查-Integer.valueOf()"
date: 2018-08-12
keywords: sonar
tags: Java
author: zzb
---
#### sonar代码检查会提示将new  Integer()替换为Integer.valueOf()
Using new Integer(int) is guaranteed to always result in a new object whereas Integer.valueOf(int) allows caching of values to be done by the compiler, class library, or JVM. Using of cached values avoids object allocation and the code will be faster.

Values between -128 and 127 are guaranteed to have corresponding cached instances and using valueOf is approximately 3.5 times faster than using constructor. For values outside the constant range the performance of both styles is the same.

Unless the class must be compatible with JVMs predating Java 1.5, use either autoboxing or the valueOf() method when creating instances of Long, Integer, Short, Character, and Byte.

---
翻译

保证使用新的Integer（int）始终生成新对象，而Integer.valueOf（int）允许由编译器，类库或JVM完成值的缓存。 使用缓存值可避免对象分配，代码将更快。

保证-128和127之间的值具有相应的高速缓存实例，并且使用valueOf比使用构造函数快大约3.5倍。 对于超出恒定范围的值，两种样式的性能是相同的。

除非该类必须与Java 1.5之前的JVM兼容，否则在创建Long，Integer，Short，Character和Byte实例时，请使用autoboxing或valueOf（）方法。

> 另外，在将字符串转换为整形时sonar推荐的是Integer.parseInt，而不是Integer.valueOf，原因是前者效率更高。后者会额外创建一个临时Integer对象，尤其在循环中慎用Integer.valueOf，优先选择Integer.parseInt

---
jdk源码

```
public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```


```
private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```

---
* 举个例子

```
public static void main(String []args) {  
      int a = 100;  
      int b = 100;  
      System.out.println(a==b);  
  
      Integer c = new Integer(100);  
      Integer d = new Integer(100);  
      System.out.println(c==d);  
}

结果
true
false
```

---
title: "Ratelimiter"
date: 2019-06-29
keywords: Ratelimiter
description: ""
tags: Java
author: ""

---

#### google guava 包里的限流工具


```
// RateLimiter提供了两个工厂方法，最终会调用下面两个函数，生成RateLimiter的两个子类。

@VisibleForTesting
    static RateLimiter create(RateLimiter.SleepingStopwatch stopwatch, double permitsPerSecond) {
        RateLimiter rateLimiter = new SmoothBursty(stopwatch, 1.0D);
        rateLimiter.setRate(permitsPerSecond);
        return rateLimiter;
    }
    
    
    @VisibleForTesting
    static RateLimiter create(RateLimiter.SleepingStopwatch stopwatch, double permitsPerSecond, long warmupPeriod, TimeUnit unit) {
        RateLimiter rateLimiter = new SmoothWarmingUp(stopwatch, warmupPeriod, unit);
        rateLimiter.setRate(permitsPerSecond);
        return rateLimiter;
    }
```




```

import com.google.common.util.concurrent.RateLimiter;

import java.util.concurrent.TimeUnit;

public class RateLimitTest {

    public static void main(String[] args){

        test3();
    }

    /**
     * 平滑突发限流(SmoothBursty)
     * testSmoothBursty
     * 使用 RateLimiter的静态方法创建一个限流器，设置每秒放置的令牌数为5个。返回  的RateLimiter对象可以保证1秒内不会给超过5个令牌，并且以固定速率进行放置， 达 到平滑输出的效果。
     *
     * get 1 token 0.0s
     * get 1 token 0.199359s
     * get 1 token 0.188792s
     * get 1 token 0.199977s
     * get 1 token 0.19958s
     * get 1 token 0.199001s
     * get 1 token 0.199661s
     */
    public static void test(){
        //每秒不会超过5个
        RateLimiter rateLimiter = RateLimiter.create(5);

        while (true){
            System.out.println("get 1 token " + rateLimiter.acquire() +"s");
        }
    }

    /**
     *  testSmoothBursty3
     * ---------start-------
     * get 1 token 0.0s
     * get 1 token 0.0s
     * get 1 token 0.0s
     * get 1 token 0.499817s
     * get 1 token 0.433772s
     * ---------end-------
     * ---------start-------
     * get 1 token 0.499415s
     * get 1 token 0.0s
     * get 1 token 0.0s
     * get 1 token 0.499247s
     * get 1 token 0.499943s
     * ---------end-------
     */
    public static void test1(){
        //每秒不会超过2个
        RateLimiter rateLimiter = RateLimiter.create(2);

        while (true){
            System.out.println("---------start-------");
            System.out.println("get 1 token " + rateLimiter.acquire() +"s");
            try {
                Thread.sleep(1000);
            }catch (Exception e){
                e.printStackTrace();
            }
            System.out.println("get 1 token " + rateLimiter.acquire() +"s");
            System.out.println("get 1 token " + rateLimiter.acquire() +"s");
            System.out.println("get 1 token " + rateLimiter.acquire() +"s");
            System.out.println("get 1 token " + rateLimiter.acquire() +"s");
            System.out.println("---------end-------");

        }
    }


    /**
     *
     * testSmoothBursty
     *
     * get 5 token 0.0s
     * get 1 token 0.99923s
     * get 1 token 0.168286s
     * get 1 token 0.103337s
     * get 1 token 0.158038s
     * get 1 token 0.131698s
     * ------------end----------
     * get 5 token 0.12993s
     * get 1 token 0.990453s
     * get 1 token 0.08746s
     * get 1 token 0.14798s
     * get 1 token 0.168628s
     * get 1 token 0.182927s
     * ------------end----------
     */
    public static void test2(){
        //每秒不会超过5个
        RateLimiter rateLimiter = RateLimiter.create(5);

        while (true){
            System.out.println("get 5 token " + rateLimiter.acquire(5) +"s");
            System.out.println("get 1 token " + rateLimiter.acquire(1) +"s");
            System.out.println("get 1 token " + rateLimiter.acquire(1) +"s");
            System.out.println("get 1 token " + rateLimiter.acquire(1) +"s");
            System.out.println("get 1 token " + rateLimiter.acquire(1) +"s");
            System.out.println("get 1 token " + rateLimiter.acquire(1) +"s");
            System.out.println("------------end---------- ");
        }
    }

    /**
     * 平滑预热限流
     * SmoothWarmingUp
     * 创建一个平均分发令牌速率为2，预热期为3分钟。由于设置了预热时间是3秒，令牌桶一开始并不会0.5秒发一个令牌，
     * 而是形成一个平滑线性下降的坡度，频率越来越高，在3秒钟之内达到原本设置的频率，以后就以固定的频率输出。
     * 这种功能适合系统刚启动需要一点时间来“热身”的场景。
     *
     * get 1 token 0.0s
     * get 1 token 1.332856s
     * get 1 token 0.964291s
     * get 1 token 0.543042s
     * get 1 token 0.418027s
     * get 1 token 0.340004s
     * ------------end----------
     * get 1 token 0.464712s
     * get 1 token 0.498872s
     * get 1 token 0.499375s
     * get 1 token 0.499742s
     * get 1 token 0.499116s
     * get 1 token 0.499375s
     * ------------end----------
     * get 1 token 0.499662s
     * get 1 token 0.499117s
     *
     *
     */
    public static void test3(){
        //每秒不会超过5个
        RateLimiter rateLimiter = RateLimiter.create(2,3, TimeUnit.SECONDS);

        while (true){
            System.out.println("get 1 token " + rateLimiter.acquire(1) +"s");
            System.out.println("get 1 token " + rateLimiter.acquire(1) +"s");
            System.out.println("get 1 token " + rateLimiter.acquire(1) +"s");
            System.out.println("get 1 token " + rateLimiter.acquire(1) +"s");
            System.out.println("get 1 token " + rateLimiter.acquire(1) +"s");
            System.out.println("get 1 token " + rateLimiter.acquire(1) +"s");
            System.out.println("------------end---------- ");
        }
    }

}
```



---
title: "LocalDate、LocalTime、LocalDateTime、DateFormatter"
date: 2019-06-29
keywords: LocalDate,LocalTime,LocalDateTime,DateFormatter
desc: ""
tags: Java
author: ""

---

### Java 8 里新的日期时间处理类
#### LocalDate,LocalTime,LocalDateTime,Instant通用方法

方法 | 是否静态方法 |描述
---|---|---
from | 是|依照传入的Temporal对象创建实例
now | 是|依照系统时钟创建Temporal对象
of | 是|由Temporal对象的某个部分创建该对象的实例
parse|是|由字符串创建Temporal对象的实例
atOffset|是|将Temporal对象和某个时区偏移结合
atZone|否|将Temporal对象和某个时区结合
format|否|使用指定的格式器将Temporal对象转换为字符串
get|否|读取Temporal对象的某一部分的值
minus|否|创建Temporal对象的一个副本，通过将当前Temporal的值减去一定时长创建该副本
plus|否|创建temporal对象的一个副本，将当前temporal对象的值加上一定的时长创建该副本
with|否|以该temporal对象为模板，对某些状态进行修改创建该对象的副本


```
    /**
     * year:2019
     * month:MAY
     * dayofmonth:5
     * day:30
     * dayofWeek:THURSDAY
     * len:31
     * isLeap:false
     * today:2019-06-30
     * date1:2019-06-30
     */
    public static void localDateTest(){
        LocalDate date = LocalDate.of(2019,5,30);
        int year = date.getYear();
        Month month = date.getMonth();
        int month1= date.getMonthValue();
        int day = date.getDayOfMonth();
        DayOfWeek dayOfWeek = date.getDayOfWeek();
        int len = date.lengthOfMonth();
        boolean isLeap = date.isLeapYear();
        LocalDate today = LocalDate.now();
        LocalDate date1 = LocalDate.parse("2019-06-30");

        System.out.println("year:"+year);
        System.out.println("month:"+month);
        System.out.println("dayofmonth:"+month1);
        System.out.println("day:"+day);
        System.out.println("dayofWeek:"+dayOfWeek);
        System.out.println("len:"+len);
        System.out.println("isLeap:"+isLeap);
        System.out.println("today:"+today);
        System.out.println("date1:"+date1);
    }

    /**
     *
     hour:12
     minute:11
     second:10
     time1:12:11:10
     datetime:2019-05-30T12:11:10
     datetime1:2019-05-30T12:11:10
     datetime2:2019-05-30T12:11:10
     datetime3:2019-05-30T12:11:10
     date1:2019-05-30
     time2:12:11:10
     */
    public static void testLocalTime(){
        LocalTime time = LocalTime.of(12,11,10);
        int hour = time.getHour();
        int minut = time.getMinute();
        int second = time.getSecond();

        LocalDate date = LocalDate.parse("2019-05-30");
        LocalTime time1 = LocalTime.parse("12:11:10");
        //合并日期时间
        LocalDateTime dateTime = LocalDateTime.of(date,time);
        LocalDateTime dateTime1 = date.atTime(12,11,10);
        LocalDateTime dateTime2 = date.atTime(time1);
        LocalDateTime dateTime3 = time1.atDate(date);
        //提取LocalDate LocalTime
        LocalDate date1 = dateTime.toLocalDate();
        LocalTime time2 = dateTime.toLocalTime();

        System.out.println("hour:"+hour);
        System.out.println("minute:"+minut);
        System.out.println("second:"+second);
        System.out.println("time1:"+time1);
        System.out.println("datetime:"+dateTime);
        System.out.println("datetime1:"+dateTime1);
        System.out.println("datetime2:"+dateTime2);
        System.out.println("datetime3:"+dateTime3);
        System.out.println("date1:"+date1);
        System.out.println("time2:"+time2);

    }

    /**
     * duration:PT10M
     * duration1:PT10M
     * period:P1M
     */
    public static void testDuration(){
        LocalTime time1 = LocalTime.parse("12:11:10");
        LocalTime time2 = LocalTime.parse("12:21:10");
        LocalDate date = LocalDate.parse("2019-05-30");
        LocalDate date1 = LocalDate.parse("2019-06-30");

        LocalDateTime dateTime1 = LocalDateTime.of(date,time1);
        LocalDateTime dateTime2 = LocalDateTime.of(date,time2);
        Duration duration = Duration.between(time1,time2);
        Duration duration1 = Duration.between(dateTime1,dateTime2);

        Duration threeMinuts = Duration.ofMinutes(3);
        Period tendays = Period.ofDays(10);
        Period threeWeeks = Period.ofWeeks(3);
        Period twoyearSixmonthOneday = Period.of(2,6,1);



        Period period = Period.between(date,date1);
        System.out.println("duration:"+duration);
        System.out.println("duration1:"+duration1);
        System.out.println("period:"+period);
    }
```
#### DateTimeFormatter

```

import org.apache.commons.lang3.time.StopWatch;
import java.text.SimpleDateFormat;
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.ZoneOffset;
import java.time.format.DateTimeFormatter;
import java.util.Date;

public class LocalDateTest {

    /**
     *  2019-06-21T12:12:12
     */
    public static void test(){
        String dateStr = "2019-06-21 12:12:12";
        String pattern = "yyyy-MM-dd HH:mm:ss";
        LocalDateTime dateTime = LocalDateTime.parse(dateStr, DateTimeFormatter.ofPattern(pattern));
        System.out.println(dateTime);
    }

    /**
     * 2019-06-21 16:14:39
     *
     * 20190621164500
     * 351
     * 20190621164500
     * 50
     *
     *
     */
    public static void test1(){
        StopWatch watch = new StopWatch();
        watch.start();
        LocalDateTime now = LocalDateTime.now();
        String pattern = "yyyyMMddHHmmss";
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern(pattern);
        String format = formatter.format(now);
        System.out.println(format);
        watch.stop();
        System.out.println(watch.getTime());
        watch.reset();


        watch.start();
        SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMddHHmmss");
        Date nowtime = new Date();
        System.out.println(sdf.format(nowtime));
        watch.stop();

        System.out.println(watch.getTime());

    }

    /** 时间戳
     * 1561105950639
     */
    public static void testDateToTimeMillis() {
        LocalDateTime dateTime = LocalDateTime.now();
        long epochMilli = dateTime.toInstant(ZoneOffset.of("+8")).toEpochMilli();
        System.out.println(epochMilli);
    }

    public static String getYesterdayByFormat(String timeFormat){
        return LocalDateTime.now().minusDays(1).format(DateTimeFormatter.ofPattern(timeFormat));
    }

    public static String getTomorrowByFormat(String timeFormat){
        return LocalDateTime.now().plusDays(1).format(DateTimeFormatter.ofPattern(timeFormat));
    }
}

```


#### TemporalAdjuster

> 除了TemporalAdjusters提供的这些方法，也可以自己通过实现TemporalAdjuster接口创建自己需要的实现类。


方法名 |描述 
---|---
dayOFWeekInMonth | 同一个月中每一周的第几天
firstDayOfMonth | 当月第一天
firstDayOfMonth| 下月的第一天
firstDayOfYear| 今年的第一天
firstDayOfNextYear|明年的第一天
firstInMonth|同一个月中第一个符合星期几要求的值
lastDayOfMonth | 当月最后一天
lastDayOfMonth| 下月的最后一天
lastDayOfYear| 今年的最后一天
lastDayOfNextYear|明年的最后一天
lastInMonth|同一个月中最后一个符合星期几要求的值



```
  LocalDate date1 = LocalDate.parse("2019-06-30");
        LocalDate date2 = date1.with(TemporalAdjusters.firstDayOfMonth());
        System.out.println("date2:"+date2);
        //date2:2019-06-01
```

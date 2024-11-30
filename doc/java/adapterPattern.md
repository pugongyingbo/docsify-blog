---
title: 适配器模式
date: 2018-04-23
keywords: Java设计模式
desc: 
tags: Java设计模式
author: zzb
---

#### 解释

* 现实世界例子

> 想象你有一些图片在内存卡中，你需要传输它们到电脑。为了传输它们你需要一些兼容电脑接口的适配器，你才能连接电脑。在这个例子中，读卡器是一个适配器。另一个例子更加贴合适配器， 一个三叉口的插头不能插入两叉口的插座，它需要使用转换头。另一个例子是翻译工作。

* 用浅显的话来说

>  适配器模式允许你把一个不兼容的对象包装在适配器中，使其符合另一个类。

* 维基百科

> 在软件工程中，适配器模式是一种软件设计模式，允许将现有类的接口用作另一个接口。它经常被用来无需修改现有类的源码和其他类一起工作。

#### 代码解释

想象一个船长只会划船不会航行。
首先我们有RowingBoat和FishingBoat接口
```
public interface RowingBoat {
  void row();
}

public class FishingBoat {
  private static final Logger LOGGER = LoggerFactory.getLogger(FishingBoat.class);
  public void sail() {
    LOGGER.info("The fishing boat is sailing");
  }
}
```

另外船长期望实现一个划船接口可以移动。

```
public class Captain implements RowingBoat {

  private RowingBoat rowingBoat;

  public Captain(RowingBoat rowingBoat) {
    this.rowingBoat = rowingBoat;
  }

  @Override
  public void row() {
    rowingBoat.row();
  }
}
```
现在海盗即将到来，船长需要逃跑，但是这里只有一个渔船。我们需要创建一个适配器允许船长操作这个渔船用他的划船技巧。

```
public class FishingBoatAdapter implements RowingBoat {

  private static final Logger LOGGER = LoggerFactory.getLogger(FishingBoatAdapter.class);

  private FishingBoat boat;

  public FishingBoatAdapter() {
    boat = new FishingBoat();
  }

  @Override
  public void row() {
    boat.sail();
  }
}
```

现在这个船长可以使用渔船躲避海盗了。

```
Captain captain = new Captain(new FishingBoatAdapter());
captain.row();
```

#### 什么时候使用
* 要使用现有类，但是其接口不匹配你的要求
* 要创建一个可重用的类与合作无关的意料之外的类，那些不一定具有兼容接口的类。
* 您需要使用几个现有的子类，但通过子类化每个子类来适配它们的接口是不切实际的。 对象适配器可以适配其父类的接口。
* 大多数使用第三方库的应用程序都使用适配器作为应用程序和第三方库之间的中间层，以将应用程序与库分离。 如果必须使用另一个库，则只需要用于新库的适配器而无需更改应用程序代码。

#### 代码实例
* java.util.Arrays#asList()
* java.util.Collections#list()
* java.util.Collections#enumeration()
* javax.xml.bind.annotation.adapters.XMLAdapter

> 翻译自[java设计模式大全](https://github.com/iluwatar/java-design-patterns)
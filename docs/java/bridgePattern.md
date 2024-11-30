---
title: 桥接模式
date: 2018-04-24
keywords: Java设计模式
desc: 
tags: Java设计模式
author: 
---


#### 目的
将抽象从其实现中分离出来，以使两者可以独立变化。

> “Decouple an abstraction from its implementation so that the two can vary independently”

#### 解释
* 现实世界例子

> 考虑你有一个不同魔法的武器，并且你打算允许混合不同的拥有不同魔法的武器。 你会怎么做？为每个魔法创建多个复制品关于每一个武器，或者你创建独立的魔法并且设置它作为需要的武器？桥接模式允许你做第二种方法。

* 简单来说

> 桥接模式是关于选择组合而非继承。实现细节从层次结构推送到具有单独层次结构的另一个对象。

* 维基百科解释

> 桥接模式是一种设计模式被用在软件工程中，意思是将抽象从其实现中分离出来，以使两者可以独立变化。

#### 代码示例

- 先来个容易理解的： 颜色和形状。

我们有一个接口结构包括接口和实现如下图所示：
![1](https://raw.githubusercontent.com/pugongyingbo/pugongyingbo.github.io/master/images/Bridge-Interface-Hierarchy.png)

现在我们用桥接模式分离接口和实现，如下面UML图：
![2](https://raw.githubusercontent.com/pugongyingbo/pugongyingbo.github.io/master/images/bridge-design-pattern.png)

注意在形状和颜色接口使用了合成实现桥接模式。

* Color

```
package com.journaldev.design.bridge;

public interface Color {

    public void applyColor();
}
```

* Shape

```
package com.journaldev.design.bridge;

public abstract class Shape {
    //Composition - implementor
    protected Color color;
    
    //constructor with implementor as input argument
    public Shape(Color c){
        this.color=c;
    }
    
    abstract public void applyColor();
}
```

* 三角形

```
package com.journaldev.design.bridge;

public class Triangle extends Shape{

    public Triangle(Color c) {
        super(c);
    }

    @Override
    public void applyColor() {
        System.out.print("Triangle filled with color ");
        color.applyColor();
    } 

}
```

* 五角形

```
package com.journaldev.design.bridge;

public class Pentagon extends Shape{

    public Pentagon(Color c) {
        super(c);
    }

    @Override
    public void applyColor() {
        System.out.print("Pentagon filled with color ");
        color.applyColor();
    } 

}
```

* RedColor.java

```
package com.journaldev.design.bridge;

public class RedColor implements Color{

    public void applyColor(){
        System.out.println("red.");
    }
}
```

* GreenColor.java
 
```
package com.journaldev.design.bridge;

public class GreenColor implements Color{

    public void applyColor(){
        System.out.println("green.");
    }
}
```

* BridgePatternTest.java

```
package com.journaldev.design.test;

import com.journaldev.design.bridge.GreenColor;
import com.journaldev.design.bridge.Pentagon;
import com.journaldev.design.bridge.RedColor;
import com.journaldev.design.bridge.Shape;
import com.journaldev.design.bridge.Triangle;

public class BridgePatternTest {

    public static void main(String[] args) {
        Shape tri = new Triangle(new RedColor());
        tri.applyColor();
        
        Shape pent = new Pentagon(new GreenColor());
        pent.applyColor();
    }

}

//输出：
//Triangle filled with color red.
//Pentagon filled with color green.

```


---
以下是原文带的例子

* 这里我们有武器等级制度

```
public interface Weapon {
  void wield();
  void swing();
  void unwield();
  Enchantment getEnchantment();
}

public class Sword implements Weapon {

  private final Enchantment enchantment;

  public Sword(Enchantment enchantment) {
    this.enchantment = enchantment;
  }

  @Override
  public void wield() {
    LOGGER.info("The sword is wielded.");
    enchantment.onActivate();
  }

  @Override
  public void swing() {
    LOGGER.info("The sword is swinged.");
    enchantment.apply();
  }

  @Override
  public void unwield() {
    LOGGER.info("The sword is unwielded.");
    enchantment.onDeactivate();
  }

  @Override
  public Enchantment getEnchantment() {
    return enchantment;
  }
}

public class Hammer implements Weapon {

  private final Enchantment enchantment;

  public Hammer(Enchantment enchantment) {
    this.enchantment = enchantment;
  }

  @Override
  public void wield() {
    LOGGER.info("The hammer is wielded.");
    enchantment.onActivate();
  }

  @Override
  public void swing() {
    LOGGER.info("The hammer is swinged.");
    enchantment.apply();
  }

  @Override
  public void unwield() {
    LOGGER.info("The hammer is unwielded.");
    enchantment.onDeactivate();
  }

  @Override
  public Enchantment getEnchantment() {
    return enchantment;
  }
}
```

*　还有独立的魔法等级制度

```
public interface Enchantment {
  void onActivate();
  void apply();
  void onDeactivate();
}

public class FlyingEnchantment implements Enchantment {

  @Override
  public void onActivate() {
    LOGGER.info("The item begins to glow faintly.");
  }

  @Override
  public void apply() {
    LOGGER.info("The item flies and strikes the enemies finally returning to owner's hand.");
  }

  @Override
  public void onDeactivate() {
    LOGGER.info("The item's glow fades.");
  }
}

public class SoulEatingEnchantment implements Enchantment {

  @Override
  public void onActivate() {
    LOGGER.info("The item spreads bloodlust.");
  }

  @Override
  public void apply() {
    LOGGER.info("The item eats the soul of enemies.");
  }

  @Override
  public void onDeactivate() {
    LOGGER.info("Bloodlust slowly disappears.");
  }
}
```

执行这两个等级

```
Sword enchantedSword = new Sword(new SoulEatingEnchantment());
enchantedSword.wield();
enchantedSword.swing();
enchantedSword.unwield();
// The sword is wielded.
// The item spreads bloodlust.
// The sword is swinged.
// The item eats the soul of enemies.
// The sword is unwielded.
// Bloodlust slowly disappears.

Hammer hammer = new Hammer(new FlyingEnchantment());
hammer.wield();
hammer.swing();
hammer.unwield();
// The hammer is wielded.
// The item begins to glow faintly.
// The hammer is swinged.
// The item flies and strikes the enemies finally returning to owner's hand.
// The hammer is unwielded.
// The item's glow fades.
```

#### 什么时候使用
* 你想避免抽象及其实现之间的永久绑定。例如，必须在运行时选择或转换实现。
* 两个抽象及其实现应该被子类扩展。桥接模式允许你将不同的抽象和实现组合起来，并独立扩展它们。
* 抽象的实现变化不应该对客户产生影响，也就是说，他们的代码不应该被重新编译。
* 你有很多类，恰好有个类层次结构表明需要将对象拆分成两部分。
* 你想在多个类之间共享一个实现，这个实现细节应该对客户端隐藏

#### 延伸

> 聚合(Aggregation)表示一种弱的拥有关系，体现的是A对象可以包含B对象，但B对象不是A对象的一部分；合成(composition)则是一种强的拥有关系，体现了严格的部分和整体的关系，部分和整体的生命周期一样。--《设计模式解析》
优先使用对象的合成/聚合将有助于你保持每个类被封装，并被集中在单个任务上。这样类和类继承层次会保持较小规模，并且不太可能增长为不可控制的庞然大物。--《设计模式》

* 桥接模式与适配器模式在代码实现上是相同的。差别在于适配器模式是已经有一个进水管和家用水龙头存在了，这时候生产一个接头来连接进水管和水龙头，这个接头就是适配器。桥接是定下生产接头策略，再让进水管有开发的自由度。只是先后的问题。


> 翻译自[Java设计模式](https://github.com/iluwatar/java-design-patterns/tree/master/bridge)

#### 参考
* https://www.journaldev.com/1491/bridge-design-pattern-java
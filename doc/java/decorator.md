---
title: 装饰器模式
date: 2018-04-25
keywords: 装饰器模式
desc: 
tags: Java设计模式
author: 
---

#### 举个例子

> 有一个巨怪住在山附近，平时它都是空手的，但是有时它有一个武器，为了武装自己，它不必创建一个新的巨怪，而是用合适的武器动态的装饰自己。

#### 简单的说
> 装饰器模式让你在运行时动态改变对象行为通过包装他们在一个装饰器类的对象中。

#### 维基百科解释

> 在面向对象编程中，装饰器模式是一个设计模式，允许给一个独立的对象添加行为，不管是动态的还是静态的，而不会影响来自同一个类的其他对象的行为。装饰器模式被经常用来遵循单一职责原则，因为它允许在具有独特区域的类之间划分功能。

#### 代码解释

假设我们想实现不同类型的车，我们可以创建车接口去定义组装方法，并且我们可以有一个基础车，我们可以扩展它成为运动车和豪华车，将如下图所示：
![1](https://raw.githubusercontent.com/pugongyingbo/pugongyingbo.github.io/master/media/inheritance-hierarchy.png)

但是我们想得到一辆车在运行时，既拥有运动车的特征，又拥有豪华车的特征，然后它的实现又复杂了，更进一步我们想指定那些功能应该先添加，它会变得更复杂。现在想象如果我们有十种车，用继承和组合的实现逻辑将难以想象。为了解决这个问题，我们使用装饰器模式。

1.接口部分--Car
```
package com.journaldev.design.decorator;

public interface Car {

    public void assemble();
}
```
2.实现部分--BasicCar
```
package com.journaldev.design.decorator;

public class BasicCar implements Car {

    @Override
    public void assemble() {
        System.out.print("Basic Car.");
    }

}
```
3.装饰器--装饰器类实现了接口部分，并且它与接口是“is-a”关系，组件变量应该被子类访问，所以我们将这个变量设置成protected


```
package com.journaldev.design.decorator;

public class CarDecorator implements Car {

    protected Car car;
    
    public CarDecorator(Car c){
        this.car=c;
    }
    
    @Override
    public void assemble() {
        this.car.assemble();
    }

}
```
4. 具体的装饰对象 --扩展基础装饰器功能并相应的修改组件行为。

* 运动车

```
package com.journaldev.design.decorator;

public class SportsCar extends CarDecorator {

    public SportsCar(Car c) {
        super(c);
    }

    @Override
    public void assemble(){
        super.assemble();
        System.out.print(" Adding features of Sports Car.");
    }
}
```

* 豪华车

```
package com.journaldev.design.decorator;

public class LuxuryCar extends CarDecorator {

    public LuxuryCar(Car c) {
        super(c);
    }
    
    @Override
    public void assemble(){
        super.assemble();
        System.out.print(" Adding features of Luxury Car.");
    }
}
```

#### 类图
![2](https://raw.githubusercontent.com/pugongyingbo/pugongyingbo.github.io/master/media/decorator-pattern.png)

* 测试类

```
ackage com.journaldev.design.test;

import com.journaldev.design.decorator.BasicCar;
import com.journaldev.design.decorator.Car;
import com.journaldev.design.decorator.LuxuryCar;
import com.journaldev.design.decorator.SportsCar;

public class DecoratorPatternTest {

    public static void main(String[] args) {
        Car sportsCar = new SportsCar(new BasicCar());
        sportsCar.assemble();
        System.out.println("\n*****");
        
        Car sportsLuxuryCar = new SportsCar(new LuxuryCar(new BasicCar()));
        sportsLuxuryCar.assemble();
    }

}
```

* 输出

```
Basic Car. Adding features of Sports Car.
*****
Basic Car. Adding features of Luxury Car. Adding features of Sports Car.
```

#### JDK例子
* java.io.InputStream, java.io.OutputStream, java.io.Reader and java.io.Writer
* java.util.Collections#synchronizedXXX()
* java.util.Collections#unmodifiableXXX()
* java.util.Collections#checkedXXX()

#### 什么时候使用

* 动态的给一个对象添加一些额外的职责，在不影响其他方法的情况下。
* 对于可撤回的职责
* 当通过继承进行扩展是不切实际的。

#### 延伸
 >　装饰模式，动态的给一个对象添加一些额外的职责，就增加功能来说，装饰模式比生成子类更为灵活。《设计模式》
 
![3](https://raw.githubusercontent.com/pugongyingbo/pugongyingbo.github.io/master/media/decorator.png)

 > Component是定义一个对象接口，可以给这些对象动态的添加职责。ConcreteComponent是定义了一个具体的对象，也昆虫该这个对象添加一些职责。Decorator，装饰抽象类，继承了从外类来扩展Component功能，但对于Component来说，是无需知道Decortator的存在的。至于ConcreteDecorator就是具体的装饰对象，起到给Component添加职责的功能。《设计模式解析》
 
#### 参考
 * https://www.journaldev.com/1540/decorator-design-pattern-in-java-example
 * 还可以参考这篇博客 https://www.jianshu.com/p/720925a6df7a
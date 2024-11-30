---
title: 工厂设计模式
date: 2018-05-02
keywords: 工厂设计模式
desc:
tags: Java设计模式
author: ""
---


#### 解释

>定义一个用于创建对象的接口，让子类决定示例化哪一个类，工厂方法使一个类的实例化延迟到其子类。

#### 超类

工厂模式中的超类可以是一个接口，抽象类或普通的Java类。

```
package com.journaldev.design.model;

public abstract class Computer {
    
    public abstract String getRAM();
    public abstract String getHDD();
    public abstract String getCPU();
    
    @Override
    public String toString(){
        return "RAM= "+this.getRAM()+", HDD="+this.getHDD()+", CPU="+this.getCPU();
    }
}
```

#### 子类

PC和Server两个子类

```
package com.journaldev.design.model;

public class PC extends Computer {

    private String ram;
    private String hdd;
    private String cpu;
    
    public PC(String ram, String hdd, String cpu){
        this.ram=ram;
        this.hdd=hdd;
        this.cpu=cpu;
    }
    @Override
    public String getRAM() {
        return this.ram;
    }

    @Override
    public String getHDD() {
        return this.hdd;
    }

    @Override
    public String getCPU() {
        return this.cpu;
    }

}
```

* server子类

```
package com.journaldev.design.model;

public class Server extends Computer {

    private String ram;
    private String hdd;
    private String cpu;
    
    public Server(String ram, String hdd, String cpu){
        this.ram=ram;
        this.hdd=hdd;
        this.cpu=cpu;
    }
    @Override
    public String getRAM() {
        return this.ram;
    }

    @Override
    public String getHDD() {
        return this.hdd;
    }

    @Override
    public String getCPU() {
        return this.cpu;
    }

}
```

#### 工厂类

1. 我们可以保持工厂类单例，或者使用static方法返回子类。
2. 注意基于不同的输入参数，不同的子类被创建并且返回，getComputer是工厂方法。

```
package com.journaldev.design.factory;

import com.journaldev.design.model.Computer;
import com.journaldev.design.model.PC;
import com.journaldev.design.model.Server;

public class ComputerFactory {

    public static Computer getComputer(String type, String ram, String hdd, String cpu){
        if("PC".equalsIgnoreCase(type)) return new PC(ram, hdd, cpu);
        else if("Server".equalsIgnoreCase(type)) return new Server(ram, hdd, cpu);
        
        return null;
    }
}
```

* 测试代码

```
package com.journaldev.design.test;

import com.journaldev.design.factory.ComputerFactory;
import com.journaldev.design.model.Computer;

public class TestFactory {

    public static void main(String[] args) {
        Computer pc = ComputerFactory.getComputer("pc","2 GB","500 GB","2.4 GHz");
        Computer server = ComputerFactory.getComputer("server","16 GB","1 TB","2.9 GHz");
        System.out.println("Factory PC Config::"+pc);
        System.out.println("Factory Server Config::"+server);
    }

}
```

* 输出

```
Factory PC Config::RAM= 2 GB, HDD=500 GB, CPU=2.4 GHz
Factory Server Config::RAM= 16 GB, HDD=1 TB, CPU=2.9 GHz
```

#### 优点
1. 工厂设计模式为接口而不是实现提供了代码方法。
2. 工厂设计模式从客户端代码中移除了实现类的实例化。工厂模式让我们的代码更健壮，耦合更少且易于扩展。例如我们可以很简单的实现PC类，因为客户端程序没有意识到它。
3. 工厂模式通过继承在实现类和客户端类之间提供了抽象。

#### JDK例子
1. java.util.Calendar ,ResourceBundle ,NumberFormat 的getInstance()方法
2. 包装类的valueOf()方法，Boolean,Integer等
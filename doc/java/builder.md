---
title: 建造者模式
date: 2018-05-13
keywords: Java设计模式
description: ""
tags: Java设计模式
author: ""

---

#### 解释
> 将一个复杂对象的构建与它的表示分离，使的同样的构建过程可以创建不同的表示。

* 当object有许多属性时，工厂和抽象工厂会有三个问题。

1. 太多的参数从客户端传递到factory类可能出错，因为大部分的时间参数的类型都是相同的，而从客户端来看，它很难维护参数的顺序。
2. 一些参数可能是可选的，但在工厂模式下，我们被迫发送所有需要发送的参数和为null的可选的参数。
3. 如果对象很重且器创建过程很复杂，那么所有这些复杂性都将成为令人困惑的factory类的一部分。
 
* 通过为构造函数提供所需的参数，然后使用不同的setter方法来设置可选参数，我们可以解决大量参数的问题。这种方法的问题是，除非所有属性都被明确设置，否则对象状态将不一致。
* 构造器模式通过提供一种逐步构建对象的方法并提供实际返回最终对象的方法类解决具有大量不可选参数和不一致的问题。

#### 例子

* 首先你需要创建一个静态嵌套类，然后将所有参数从外部类复制到builder类，我们应该遵循命名约定，如果类名是computer，那么builder类应命名为ComputerBuilder。
* builder类应该有一个公共构造函数，并具有所有必须的属性作为参数。
* builder类应该有设置可选参数的方法，并且在设置可选属性之后它应该返回相同的builder对象。
* 最后一步是在构造器类中提供一个build方法，该方法将返回客户端程序所需要的对象，为此，我们需要在具有builder类的类中使用一个私有构造函数作为参数。


```
public class Computer {
    
    //required parameters
    private String HDD;
    private String RAM;
    
    //optional parameters
    private boolean isGraphicsCardEnabled;
    private boolean isBluetoothEnabled;
    

    public String getHDD() {
        return HDD;
    }

    public String getRAM() {
        return RAM;
    }

    public boolean isGraphicsCardEnabled() {
        return isGraphicsCardEnabled;
    }

    public boolean isBluetoothEnabled() {
        return isBluetoothEnabled;
    }
    
    private Computer(ComputerBuilder builder) {
        this.HDD=builder.HDD;
        this.RAM=builder.RAM;
        this.isGraphicsCardEnabled=builder.isGraphicsCardEnabled;
        this.isBluetoothEnabled=builder.isBluetoothEnabled;
    }
    
    //Builder Class
    public static class ComputerBuilder{

        // required parameters
        private String HDD;
        private String RAM;

        // optional parameters
        private boolean isGraphicsCardEnabled;
        private boolean isBluetoothEnabled;
        
        public ComputerBuilder(String hdd, String ram){
            this.HDD=hdd;
            this.RAM=ram;
        }

        public ComputerBuilder setGraphicsCardEnabled(boolean isGraphicsCardEnabled) {
            this.isGraphicsCardEnabled = isGraphicsCardEnabled;
            return this;
        }

        public ComputerBuilder setBluetoothEnabled(boolean isBluetoothEnabled) {
            this.isBluetoothEnabled = isBluetoothEnabled;
            return this;
        }
        
        public Computer build(){
            return new Computer(this);
        }

    }

}
```

* 注意，computer类只有getter方法，没有公共构造函数，所以唯一的办法是通过computerBuilder类的计算机对象。

```
import com.journaldev.design.builder.Computer;

public class TestBuilderPattern {

    public static void main(String[] args) {
        //Using builder to get the object in a single line of code and 
                //without any inconsistent state or arguments management issues     
        Computer comp = new Computer.ComputerBuilder(
                "500 GB", "2 GB").setBluetoothEnabled(true)
                .setGraphicsCardEnabled(true).build();
    }

}
```
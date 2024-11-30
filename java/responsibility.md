---
title: "责任链模式"
date: 2018-04-26
keywords: 
description: ""
tags: Java设计模式
author: ""
---

#### 解释
>使多个对象都有机会处理请求，从而避免请求的发送者和接收着之间的耦合关系。将这个对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。

#### 代码例子

ATM取款机是一个很好的例子。用户输入指定的数额，然后机器分配货币按照定义好的数额例如50,20,10块的。如果用户输入一个数字不是10的倍数，它会抛出异常。我们用责任链模式来实现解决方案。

#### 基本的接口和类
* Currency.java

```
package com.journaldev.design.chainofresponsibility;

public class Currency {

    private int amount;
    
    public Currency(int amt){
        this.amount=amt;
    }
    
    public int getAmount(){
        return this.amount;
    }
}
```
基本接口应该有一个方法来定义下一个处理在责任链中，并且这个处理会传递请求。ATM的分配接口如下：

* DispenseChain.java

```
package com.journaldev.design.chainofresponsibility;

public interface DispenseChain {

    void setNextChain(DispenseChain nextChain);
    
    void dispense(Currency cur);
}
```

####  链的实现
我们需要创建不同的处理类，这些类将会实现分配链接口，并且提供分配方法的实现。我们开发我们的系统用三种类型的货币-50 、20 、10，我们将会创建三个具体的实现。

* Dollar50Dispenser.java

```
package com.journaldev.design.chainofresponsibility;

public class Dollar50Dispenser implements DispenseChain {

    private DispenseChain chain;
    
    @Override
    public void setNextChain(DispenseChain nextChain) {
        this.chain=nextChain;
    }

    @Override
    public void dispense(Currency cur) {
        if(cur.getAmount() >= 50){
            int num = cur.getAmount()/50;
            int remainder = cur.getAmount() % 50;
            System.out.println("Dispensing "+num+" 50$ note");
            if(remainder !=0) this.chain.dispense(new Currency(remainder));
        }else{
            this.chain.dispense(cur);
        }
    }

}
```

* Dollar20Dispenser.java

```
package com.journaldev.design.chainofresponsibility;

public class Dollar20Dispenser implements DispenseChain{

    private DispenseChain chain;
    
    @Override
    public void setNextChain(DispenseChain nextChain) {
        this.chain=nextChain;
    }

    @Override
    public void dispense(Currency cur) {
        if(cur.getAmount() >= 20){
            int num = cur.getAmount()/20;
            int remainder = cur.getAmount() % 20;
            System.out.println("Dispensing "+num+" 20$ note");
            if(remainder !=0) this.chain.dispense(new Currency(remainder));
        }else{
            this.chain.dispense(cur);
        }
    }

}
```

* Dollar10Dispenser.java
 
```
package com.journaldev.design.chainofresponsibility;

public class Dollar10Dispenser implements DispenseChain {

    private DispenseChain chain;
    
    @Override
    public void setNextChain(DispenseChain nextChain) {
        this.chain=nextChain;
    }

    @Override
    public void dispense(Currency cur) {
        if(cur.getAmount() >= 10){
            int num = cur.getAmount()/10;
            int remainder = cur.getAmount() % 10;
            System.out.println("Dispensing "+num+" 10$ note");
            if(remainder !=0) this.chain.dispense(new Currency(remainder));
        }else{
            this.chain.dispense(cur);
        }
    }

}
```

#### 创建链
这是非常重要的一步，我们应该仔细的创建链，否则一个处理可能不会得到任何请求。比如，如果我们让第一个处理链是10块和20 块的分配方法，那么该请求将不会被转发到第二个处理，并且链也将变得无用。

* ATMDispenseChain.java

```
package com.journaldev.design.chainofresponsibility;

import java.util.Scanner;

public class ATMDispenseChain {

    private DispenseChain c1;

    public ATMDispenseChain() {
        // initialize the chain
        this.c1 = new Dollar50Dispenser();
        DispenseChain c2 = new Dollar20Dispenser();
        DispenseChain c3 = new Dollar10Dispenser();

        // set the chain of responsibility
        c1.setNextChain(c2);
        c2.setNextChain(c3);
    }

    public static void main(String[] args) {
        ATMDispenseChain atmDispenser = new ATMDispenseChain();
        while (true) {
            int amount = 0;
            System.out.println("Enter amount to dispense");
            Scanner input = new Scanner(System.in);
            amount = input.nextInt();
            if (amount % 10 != 0) {
                System.out.println("Amount should be in multiple of 10s.");
                return;
            }
            // process the request
            atmDispenser.c1.dispense(new Currency(amount));
        }

    }

}
```

* 输出

```
Enter amount to dispense
530
Dispensing 10 50$ note
Dispensing 1 20$ note
Dispensing 1 10$ note
Enter amount to dispense
100
Dispensing 2 50$ note
Enter amount to dispense
120
Dispensing 2 50$ note
Dispensing 1 20$ note
Enter amount to dispense
15
Amount should be in multiple of 10s.
```

#### 注意的点

* 客户端不知道链的哪一部分将处理请求，并且它将把请求发送给链中的第一个对象。
* 链中的每个对象都有自己的实现类来处理请求，可以是全部或部分，也可以将它发送给链中的下一个对象。
* 链中的每个对象都应该引用链中的下一个对象类转发请求，它由Java组合实现。
* 仔细的创建链非常重要，否则可能吹出现请求永远不会转发到特定处理器的情况，或链中没有任何对象能够处理该请求。
* 责任链模式可以很好的实现解耦，但如果大多数代码在所有实现类中都很常见，那么它会带来很多实现类和维护问题。

#### 实际例子
* java.util.logging.Logger#log()
* javax.servlet.Filter#doFilter()
* [Apache Commons Chain] (https://commons.apache.org/proper/commons-chain/index.html)

>翻译自 https://www.journaldev.com/1617/chain-of-responsibility-design-pattern-in-java
---
title: "策略模式"
date: 2018-05-08
keywords: 策略模式
description: ""
tags: Java设计模式
author: ""
---

#### 策略模式
> 我们定义了多种算法，并且让客户端应用传递被用来作为参数的算法。

* PaymentStrategy.java

```
package com.journaldev.design.strategy;

public interface PaymentStrategy {

    public void pay(int amount);
}
```

* CreditCardStrategy.java

```
package com.journaldev.design.strategy;

public class CreditCardStrategy implements PaymentStrategy {

    private String name;
    private String cardNumber;
    private String cvv;
    private String dateOfExpiry;
    
    public CreditCardStrategy(String nm, String ccNum, String cvv, String expiryDate){
        this.name=nm;
        this.cardNumber=ccNum;
        this.cvv=cvv;
        this.dateOfExpiry=expiryDate;
    }
    @Override
    public void pay(int amount) {
        System.out.println(amount +" paid with credit/debit card");
    }

}
```

* PaypalStrategy.java

```
package com.journaldev.design.strategy;

public class PaypalStrategy implements PaymentStrategy {

    private String emailId;
    private String password;
    
    public PaypalStrategy(String email, String pwd){
        this.emailId=email;
        this.password=pwd;
    }
    
    @Override
    public void pay(int amount) {
        System.out.println(amount + " paid using Paypal.");
    }

}
```

* Item.java

```
package com.journaldev.design.strategy;

public class Item {

    private String upcCode;
    private int price;
    
    public Item(String upc, int cost){
        this.upcCode=upc;
        this.price=cost;
    }

    public String getUpcCode() {
        return upcCode;
    }

    public int getPrice() {
        return price;
    }
    
}
```

* ShoppingCart.java

```
package com.journaldev.design.strategy;

import java.text.DecimalFormat;
import java.util.ArrayList;
import java.util.List;

public class ShoppingCart {

    //List of items
    List<Item> items;
    
    public ShoppingCart(){
        this.items=new ArrayList<Item>();
    }
    
    public void addItem(Item item){
        this.items.add(item);
    }
    public void removeItem(Item item){
        this.items.remove(item);
    }
    
    public int calculateTotal(){
        int sum = 0;
        for(Item item : items){
            sum += item.getPrice();
        }
        return sum;
    }
    
    public void pay(PaymentStrategy paymentMethod){
        int amount = calculateTotal();
        paymentMethod.pay(amount);
    }
}
```
注意，购物车的付款方式需要支付算法作为参数，并不会将其作为实例变量存储在任何位置。

* ShoppingCartTest.java

```
package com.journaldev.design.strategy;

public class ShoppingCartTest {

    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();
        
        Item item1 = new Item("1234",10);
        Item item2 = new Item("5678",40);
        
        cart.addItem(item1);
        cart.addItem(item2);
        
        //pay by paypal
        cart.pay(new PaypalStrategy("myemail@example.com", "mypwd"));
        
        //pay by credit card
        cart.pay(new CreditCardStrategy("Pankaj Kumar", "1234567890123456", "786", "12/15"));
    }
}

```

* 输出

```
50 paid using Paypal.
50 paid with credit/debit card
```


* 我们可以使用组合来为策略创建实例变量，但我们应该避免这种情况，因为我们夕阳将特定策略应用于特定任务。在将比较器作为参数的Collections.sort()和Array.sort()方法中也是如此。
* 策略模式与状态模式非常相似。不同之处在于contex包含state作为实例变量，并且可以有多个任务，其实现可以依赖于状态，而策略模式将策略作为参数传递给方法，并且context对象没有任何变量来存储它。
* 当我们有针对特定任务的多种算法是，策略模式非常有用，我们希望我们的应用程序能够灵活地运行时为特定任务选择任何算法。

#### 参考
* https://www.journaldev.com/1754/strategy-design-pattern-in-java-example-tutorial
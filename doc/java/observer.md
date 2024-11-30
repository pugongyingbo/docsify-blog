---
title: 观察者模式
date: 2018-04-27
keywords: 观察者模式
desc: ""
tags: Java设计模式
author: ""
---
#### 解释

> 观察者模式定义了一种一对多的依赖关系，让多个观察者对象同时监听某一个主题对象。这个主题对象在状态发生变化时，会通知所有观察者对象，使它们能够自动更新自己。《设计模式》

* subject包含管观察者列表，以通知其状态的任何变化，所以它应该提供观察者可以注册 和注销自己。主题还包含一种方法通知所有观察者的任何改变，并且可以在通知观察者时发送更新，或者可以提供另一种获取更新的方法。

* Observer 应该有一个方法来设置要监视的对象以及subject将使用的另一种方法来通知它们任何更新。

#### 代码例子

我们实现一个简单的主题，观察者可以注册这个主题，每当有新消息发送到主题中时，所有的注册观察者会受到通知，他们可以使用该消息。

* Subject接口，它定义了由具体实现的契约方法。

```
package com.journaldev.design.observer;

public interface Subject {

    //methods to register and unregister observers
    public void register(Observer obj);
    public void unregister(Observer obj);
    
    //method to notify observers of change
    public void notifyObservers();
    
    //method to get updates from subject
    public Object getUpdate(Observer obj);
    
}
```

* 接下来，我们将为观察者创建合同，将有种方法将主题附加到观察者，另一个方法用于主题通知任何更改。

```
package com.journaldev.design.observer;

public interface Observer {
    
    //method to update the observer, used by subject
    public void update();
    
    //attach with subject to observe
    public void setSubject(Subject sub);
}
```

* 主题的具体实现

```
package com.journaldev.design.observer;

import java.util.ArrayList;
import java.util.List;

public class MyTopic implements Subject {

    private List<Observer> observers;
    private String message;
    private boolean changed;
    private final Object MUTEX= new Object();
    
    public MyTopic(){
        this.observers=new ArrayList<>();
    }
    @Override
    public void register(Observer obj) {
        if(obj == null) throw new NullPointerException("Null Observer");
        synchronized (MUTEX) {
        if(!observers.contains(obj)) observers.add(obj);
        }
    }

    @Override
    public void unregister(Observer obj) {
        synchronized (MUTEX) {
        observers.remove(obj);
        }
    }

    @Override
    public void notifyObservers() {
        List<Observer> observersLocal = null;
        //synchronization is used to make sure any observer registered after message is received is not notified
        synchronized (MUTEX) {
            if (!changed)
                return;
            observersLocal = new ArrayList<>(this.observers);
            this.changed=false;
        }
        for (Observer obj : observersLocal) {
            obj.update();
        }

    }

    @Override
    public Object getUpdate(Observer obj) {
        return this.message;
    }
    
    //method to post message to the topic
    public void postMessage(String msg){
        System.out.println("Message Posted to Topic:"+msg);
        this.message=msg;
        this.changed=true;
        notifyObservers();
    }

}
```

* 观察者的实现，它将监视这个主题

```
public class MyTopicSubscriber implements Observer {
    
    private String name;
    private Subject topic;
    
    public MyTopicSubscriber(String nm){
        this.name=nm;
    }
    @Override
    public void update() {
        String msg = (String) topic.getUpdate(this);
        if(msg == null){
            System.out.println(name+":: No new message");
        }else
        System.out.println(name+":: Consuming message::"+msg);
    }

    @Override
    public void setSubject(Subject sub) {
        this.topic=sub;
    }

}
```

* test

```
public class ObserverPatternTest {

    public static void main(String[] args) {
        //create subject
        MyTopic topic = new MyTopic();
        
        //create observers
        Observer obj1 = new MyTopicSubscriber("Obj1");
        Observer obj2 = new MyTopicSubscriber("Obj2");
        Observer obj3 = new MyTopicSubscriber("Obj3");
        
        //register observers to the subject
        topic.register(obj1);
        topic.register(obj2);
        topic.register(obj3);
        
        //attach observer to subject
        obj1.setSubject(topic);
        obj2.setSubject(topic);
        obj3.setSubject(topic);
        
        //check if any update is available
        obj1.update();
        
        //now send message to subject
        topic.postMessage("New Message");
    }

}
```

* 输出

```
Obj1:: No new message
Message Posted to Topic:New Message
Obj1:: Consuming message::New Message
Obj2:: Consuming message::New Message
Obj3:: Consuming message::New Message
```

#### 参考
* https://www.journaldev.com/1739/observer-design-pattern-in-java
---
title: 原型设计模式
date: 2018-05-10
keywords: 
desc: ""
tags: Java设计模式
author: ""
---

#### 解释
> 用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

 举一个例子来理解原型设计模式会很容易。假设我们有一个车从数据库加载数据的对象，现在我们需要多次修改程序中的这些数据，因此使用new关键字创建object并从数据库再次加载所有数据并不是一个好主意。更好的方法是将现有对象克隆到新对象中，然后执行数据操作。
 原型设计模式要求你正在复制的对象提供复制功能。这不应该用其他任何类来完成。但是，是否使用对象属性的浅或深克隆取决于需求及其设计决策。

* Employees.java

```
import java.util.ArrayList;
import java.util.List;

public class Employees implements Cloneable{

    private List<String> empList;
    
    public Employees(){
        empList = new ArrayList<String>();
    }
    
    public Employees(List<String> list){
        this.empList=list;
    }
public void loadData(){
        //read all employees from database and put into the list
        empList.add("Pankaj");
        empList.add("Raj");
        empList.add("David");
        empList.add("Lisa");
    }
    
    public List<String> getEmpList() {
        return empList;
    }

    @Override
    public Object clone() throws CloneNotSupportedException{
            List<String> temp = new ArrayList<String>();
            for(String s : this.getEmpList()){
                temp.add(s);
            }
            return new Employees(temp);
    }
    
}
```
 
 注意到clone()方法被重写来提供一个员工列表的深克隆。

* PrototypePatternTest.java

```
import com.journaldev.design.prototype.Employees;

public class PrototypePatternTest {

    public static void main(String[] args) throws CloneNotSupportedException {
        Employees emps = new Employees();
        emps.loadData();
        
        //Use the clone method to get the Employee object
        Employees empsNew = (Employees) emps.clone();
        Employees empsNew1 = (Employees) emps.clone();
        List<String> list = empsNew.getEmpList();
        list.add("John");
        List<String> list1 = empsNew1.getEmpList();
        list1.remove("Pankaj");
        
        System.out.println("emps List: "+emps.getEmpList());
        System.out.println("empsNew List: "+list);
        System.out.println("empsNew1 List: "+list1);
    }

}
```

* 输出

```
emps List: [Pankaj, Raj, David, Lisa]
empsNew List: [Pankaj, Raj, David, Lisa, John]
empsNew1 List: [Raj, David, Lisa]
```

如果没有提供对象克隆，我们将不得不每次都进行数据库调用以获取员工列表，然后做操作，这将是资源和时间的消耗。

#### 深克隆和浅克隆

object中的克隆方法是浅克隆，JDK规定了克隆需要满足的一些条件，对某个对象进行克隆，对象的成员变量如果包括引用类型或者数组，那么克隆的时候其实不会把这些对象也带着复制到克隆出来的对象里面的，只是复制一个引用，这个引用指向被克隆对象的成员对象，但是基本数据类型是会跟着被带到克隆对象里面去的，而深度可能是把对象所有属性都渎职一份新的到目标对象里面去。


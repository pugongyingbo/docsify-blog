---
title: "代理模式"
date: 2018-05-17
keywords: 代理模式
desc: ""
tags: Java设计模式
author: ""
---
#### 解释
> 为其他对象提供一种代理以控制对这个对象的访问。

代理模式的常见用途是控制访问或提供包装器实现以获得更好的性能。
假设我们有一个可以在系统上运行一些命令的类，现在，如果我们正在使用它，它很好，但如果我们想将此程序提供给客户端应用程序，它可能会有严重问题，因为客户端可以发出命令来删除某些系统文件或更改某些不想要的设置。

#### 结构
* Subject类，定义了RealSubject和Proxy的共用接口。
* RealSubject类，定义Proxy代表的真实实体。
* Proxy类，保存一个引用使得代理可以访问实体，并提供一个与Subject的接口相同的接口，这样代理可以替代实体。
* client，客户端。


#### 接口和实现类

* CommandExecutor.java

```

public interface CommandExecutor {

    public void runCommand(String cmd) throws Exception;
}
```

* CommandExecutorImpl.java

```
public class CommandExecutorImpl implements CommandExecutor {

    @Override
    public void runCommand(String cmd) throws IOException {
                //some heavy implementation
        Runtime.getRuntime().exec(cmd);
        System.out.println("'" + cmd + "' command executed.");
    }

}
```

#### 代理类

* CommandExecutorProxy.java

```
public class CommandExecutorProxy implements CommandExecutor {

    private boolean isAdmin;
    private CommandExecutor executor;
    
    public CommandExecutorProxy(String user, String pwd){
        if("Pankaj".equals(user) && "J@urnalD$v".equals(pwd)) isAdmin=true;
        executor = new CommandExecutorImpl();
    }
    
    @Override
    public void runCommand(String cmd) throws Exception {
        if(isAdmin){
            executor.runCommand(cmd);
        }else{
            if(cmd.trim().startsWith("rm")){
                throw new Exception("rm command is not allowed for non-admin users.");
            }else{
                executor.runCommand(cmd);
            }
        }
    }

}
```

* ProxyPatternTest.java

```
import com.journaldev.design.proxy.CommandExecutor;
import com.journaldev.design.proxy.CommandExecutorProxy;

public class ProxyPatternTest {

    public static void main(String[] args){
        CommandExecutor executor = new CommandExecutorProxy("Pankaj", "wrong_pwd");
        try {
            executor.runCommand("ls -ltr");
            executor.runCommand(" rm -rf abc.pdf");
        } catch (Exception e) {
            System.out.println("Exception Message::"+e.getMessage());
        }
        
    }

}
```

#### 用途
* 远程代理，也就是为一个对象在不同的地址空间提供局部代表。这样可以隐藏一个对象存在于不同地址空间的事实。
* 虚拟代理，是根据需要创建开销很大的对象。通过它来存放实例化需要很长时间的真实对象。
* 安全代理，用来控制真实对象访问时的权限。
* 智能指引，是指当调用真实的对象时，代理处理另一些事。

#### 参考
* https://www.journaldev.com/1827/java-design-patterns-example-tutorial#proxy-pattern
* 《大话设计模式》
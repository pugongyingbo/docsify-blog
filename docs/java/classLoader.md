---
title: "类加载器ClassLoader"
date: 2018-04-22T14:01:52+08:00
keywords: 类加载器
tags: Java
author: ""
---


### 类加载器
#### 类加载的七个阶段

 加载-验证-准备-解析-初始化-使用-卸载

#### 类加载器的工作机制
*  装载：查找和导入Class文件
*  链接：执行校验、准备、解析，其中解析是可以选择的。
    * 校验：检查载入class文件数据的准确性
        * 文件格式验证
        * 元数据验证
        * 字节码验证
        * 符号引用验证
    * 准备：给类的静态变量分配存储空间。private static int var = 100, 准备阶段完成后，value为0 ，而不是100，在初始化阶段才把100赋值给value。如果是private static final int value = 100 ，编译时javac会为value生成ConstantValue属性，在准备阶段虚拟机就会根据ConstantValue的设置将value赋值为100。
         
    * 解析：将符号引用转换成直接引用
        * 符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用是能无歧义地定位到目标即可
        * 直接引用可以是直接指向目标的指针、相对偏移量或是一个能间接定位到目标的句柄。
* 初始化：对类的引用转换成直接引用

方法|说明
:-:|:-:
getParent()|获取类装载器的父 装载器。除根装载器外，所有的类装载器都有且只有一个父装载器。ExtClassLoader的父装载器是根装载器，因为根装载器非Java语言编写，所以无法获得，将返回null。
loadClass(String name)|name参数指定类加载器需要装载类的名字，必须使用权限定类名。该方法有一个重载方法laodClass(String name,boolean resolve),resolve参数告诉类加载器是都需要解析该类。在初始哈市类之前，应考虑进行类解析的工作，但不是所有的类都需要解析，如果JVM只需要知道该类是否存在或找出该类的超累，那么久不需要解析。
defineClass(String name,byte[] b,int off,int len)|将类文件的字节数组转换成JVM内部java.lang.Class对象，字节数组可以从本地文件系统、远程网络获取。name为字节数组对应的全限定类名。
findSystemClass(String name)|从本地文件系统载入class文件。如果本地文件系统不粗载该class文件，将抛出ClassNotFoundException异常。该方法是JVM默认是用的装载机制。
findLoadClass(String name)|调用该方法来查看ClassLoader是否已经载入某个类，如果已装入，那么返回java.lang.Class对象；否则返回null。如果强行装载已存在的类，那么将会抛出链接错误。

#### JVM提供了3种类加载器：
1、启动类加载器（Bootstrap ClassLoader）：负责加载 JAVAHOME\lib 目录中的，或通过-Xbootclasspath参数指定路径中的，且被虚拟机认可（按文件名识别，如rt.jar）的类。 
2、扩展类加载器（Extension ClassLoader）：负责加载 JAVAHOME\lib\ext 目录中的，或通过java.ext.dirs系统变量指定路径中的类库。 
3、应用程序类加载器（Application ClassLoader）：负责加载用户路径（classpath）上的类库。

* 根装载器是ExtClassLoader的父装载器，ExtClassLoader是AppClassLoader的父装载器。JVM装载类时使用“全盘负责委托机制”，全盘负责是指当一个ClassLoader装载一个类时，除非显示地使用另一个ClassLoader载入，委托机制是指先委托父装载器寻找目标类，只有在找不到的情况下才从自己的类路径中查找并装载目标类。

#### 双亲委派模型
 如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载器请求最终都应该传送到顶层的启动类加载器中，只有父加载器反馈自己无法完成这个加载请求时，子加载器才会尝试自己去加载。 

* 有5种情况必须对类进行初始化，这5种场景中的行为称为对一个类进行主动引用。除此之外多有引用类的方法都不会触发初始化，称为被动引用。
    * 遇到new、getstatic、putstatic或invokestatic这4条字节码指令时，如果类没有进行过初始化，则先需要触发其初始化。生成这4条指令的最常见的Java代码场景是：使用new关键字实例化对象的时候、读取或设置一个类的静态字段（被final修饰、已在编译期把结果放入常量池的静态字段除外）的时候，以及调用一个类的静态方法的时候。
    * 使用java.lang.reflect包的方法对类进行反调用的时候，如果类没有进行过初始化，则需要先触发器初始化。
    * 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发器父类的初始化。
    * 当虚拟机启动时，用户需要指定一个要执行的主类（包含main方法的那个类），虚拟机会先初始化这个主类。
    * 当使用JDK1.7的动态语言支持时，如果一个java.lang.invoke.MethdHandle实例最后的解析结果REF_getstatic、REF_putstatic、REF_invokestatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化。
* 被动引用的例子，不会触发初始化
    * 通过子类引用父类的静态字段，不会导致子类初始化。输出结果是parent init! 100

    ```
    public class Parent {
        static int a = 100;
        static {
            System.out.println("parent init!");
        }
    }
    class child extends  Parent{
        static {
            System.out.println("child init!");
        }
    }
    
        public class Init {
        public static void main(String[] args) {
            System.out.println(child.a);
        }
    }

    ```

    * 通过数组定义来引用类，不会触发此类的初始化.无输出
    
    ```
    public class Init {
        public static void main(String[] args) {
            Parent[] parents = new Parent[10];
         }
    }
    
    
    ```

    * 常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。输出：100。说明没有触发Const的初始化，在编译阶段，Const类中常量A的值100存储到init类的常量池中，这两个类在编译成class文件之后就没有联系了。
    
    ```
        public class Conts {
        static final int A = 100;
        static {
            System.out.println("Const init!");
        }
    }
    
        public class Init {
        public static void main(String[] args) {
            System.out.println(Conts.A);
         }
    }

    ```

    * 通过类名获取class对象，不会触发类的初始化。输出：cat is load
    
    ```
    public class Cat {
        private String name;
        private int age;
        static {
            System.out.println("Cat is load");
        }
    }
    
    public class Dog {
        private String name;
        private int age;
        static {
            System.out.println("Dog is load");
        }
    }
    
    public class Init {
        public static void main(String[] args) throws ClassNotFoundException {
            Class dog = Dog.class;
            Class clazz = Class.forName("ClassLoader.Cat");
    
         }
    }

    ```

    * 通过Class.forName()加载指定类时，如果指定参数initialize为false时，也不会触发类初始化，其实这个参数是告诉虚拟机，是否要对类进行初始化。无输出。
    
    ```
    public class Init {
        public static void main(String[] args) throws ClassNotFoundException {
            Class clazz = Class.forName("ClassLoader.Cat",false,Cat.class.getClassLoader());
         }
    }
    ```
    
    * 通过ClassLoader默认的loadClass方法，也不会触发初始化动作。无输出。
    
    ```
        public static void main(String[] args) throws ClassNotFoundException {
            new ClassLoader(){}.loadClass("ClassLoader.Cat");
         }
    }
    ```



> java.lang.NoSuchMethodError基本上是由JVM的全盘负责委托机制引起的，例如，在类路径下放置了多个不同版本的类包，如commons-lang2.x.jar和commons-lang4.x.jar都位于类路径下，代码中用到了commons-lang4.x类的方法，而这个方法在commons-lang2.x中并不存在，JVM加载器碰巧又从commons-lang2.x.jar中加载类，运行时就会抛出这个错。

* 参考
* [java类加载的那些事](https://mp.weixin.qq.com/s/-Q255hbgbMIVymxceH_odw)
* [深入探讨类加载器](https://www.ibm.com/developerworks/cn/java/j-lo-classloader/#download)
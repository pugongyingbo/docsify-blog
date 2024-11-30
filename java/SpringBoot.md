---
title: "SpringBoot的第一个web"
date: 2018-05-01
keywords: SpringBoot
description: ""
tags: SpringBoot
author: ""
---

> 学习都是从简单的Hello World 到打印数据库的数据，入门很简单，但是也有不少坑，需要亲自动手才能遇见。

#### 首先分层

* java包
    * com.example.controller - Controller层
    * com.example.dao - Dao层
    * com.example.entity(domain) - 实体类
    * com.example.service - 业务逻辑层
    * Application -应用启动类

* resources包
    * mapper - *Mapper.xml
    * mabatis - mabatis.xml
    * static -静态文件
    * application.properties - 应用配置文件

#### pom.xml依赖
* SpringBoot 父依赖
* SpringBoot web 依赖
* SpringBoot Test依赖
* SpringBoot Mybatis依赖
* MySQL驱动依赖

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>war</packaging>

    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.2</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>RELEASE</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>

```


#### 配置文件内容

```

##配置文件位置
mybatis.config-location=classpath:mybatis/mybatis-config.xml
## mapper xml 文件地址
mybatis.mapper-locations=classpath:mapper/*Mapper.xml
mybatis.typeAliasesPackage = 实体类包路径
mybatis.typeHandlersPackage = type handlers 
## 检查 mybatis 配置是否存在，一般命名为mybatis-config.xml
mybatis.check-config-location = 
##执行模式。默认是 SIMPLE
mybatis.executorType = 

##端口号
server.port=8080

##数据库url
spring.datasource.url=jdbc:mysql://localhost:3306/test?characterEncoding=utf8
##数据库用户名
spring.datasource.username=root
##数据库密码
spring.datasource.password=root
##数据库驱动
spring.datasource.driver-class-name=com.mysql.jdbc.Driver


```

#### 注解
* 启动类除了默认的 @SpringBootApplication 增加 @MapperScan("com.example.dao")
* dao接口使用 @Mapper
* controller 使用 @RestController

#### 碰到的问题
##### SpringBoot不能识别@RestController和@RequestMapping注解

* 需要导入类
 
```
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
//或者
import org.springframework.web.bind.annotation.*
```

##### 启动报错

```
Description:

Failed to auto-configure a DataSource: 'spring.datasource.url' is not specified and no embedded datasource could be auto-configured.

Reason: Failed to determine a suitable driver class


Action:

Consider the following:
    If you want an embedded database (H2, HSQL or Derby), please put it on the classpath.
    If you have database settings to be loaded from a particular profile you may need to activate it (no profiles are currently active).

```

>解决方法:
     需要在启动类的@EnableAutoConfiguration或@SpringBootApplication中添加exclude ={DataSourceAutoConfiguration.class}，排除此类的autoconfig。启动以后就可以正常运行。

这是因为添加了数据库组件，所以autoconfig会去读取数据源配置，而我新建的项目还没有配置数据源，所以会导致异常出现。

即：@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})



##### 访问路径报错信息

```
Whitelabel Error Page
This application has no explicit mapping for /error, so you are seeing this as a fallback.

Fri Apr 07 13:02:50 CST 2017
There was an unexpected error (type=Internal Server Error, status=500).
No message available
```

 * 原因：访问路径不对，Springboot项目的main方法所在的Application启动类必须在groupID对应的包下，放在最外侧。比如你的项目groupId名叫com.example,但是DemoApplication启动类却放在com.example.demo包下，就会找不到路径。
 * 还有一种原因：
在springboot的配置文件:application.yml或application.properties中关于视图解析器的配置问题: 
    * 当pom文件下的spring-boot-starter-paren版本高时使用: 
spring.mvc.view.prefix/spring.mvc.view.suffix 
    * 当pom文件下的spring-boot-starter-paren版本低时使用: 
spring.view.prefix/spring.view.suffix

##### 使用IDEA工具时使用@Resource和@Autowired自动注解bean时会显示红色，但是项目能运行 
*  参考文章 https://stackoverflow.com/questions/39890849/what-exactly-is-field-injection-and-how-to-avoid-it
 解决方法： 
File – Settings – Inspections。在Spring Core – Autowring for Bean Class 中将Severity的级别由之前的error改成warning。

##### IDEA新增mabaties的xml配置
点击file--otherSetting--Default Setting，搜索template。然后点 绿色＋添加模板， name无所谓，extension我写xml。然后添加上内容，最后，把划了横线的那个按钮（enable live Template）点上，即是使用模板的意思。

#### 参考
* https://www.bysocket.com/?p=1610
* https://www.jianshu.com/p/8b545a537fd0

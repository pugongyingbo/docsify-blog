

### Sonar配置与使用

1. 下载并解压sonarqube
2. 编辑sonar.properties


```
vim conf/sonar.properties
```

```
sonar.jdbc.url=jdbc:mysql://127.0.0.1:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.sorceEncoding=UTF-8
sonar.login=admin
sonar.password=admin

sonar.web.host=http://127.0.0.1
sonar.web.port=9000
sonar.web.context=sonarqube
```

* 创建一个名为sonar的用户组和一个用户名为sonar密码为sonar的用户

```
groupadd sonar

useradd sonar -g sonar -p sonar
```


* 为该用户分配文件夹权限


```
chown -R sonar:sonar /opt/sonarqube/sonarqube-7.7
```


3. 启动sonarquebe

```
cd /sonarqube/sonarqube-7.7/bin/linux-86-64/

sh sonar.sh start
```

4. 访问localhost:9000，点击login，输入初始账户密码admin/admin即可。


#### 配置sonar-scanner
1. 对应系统下载安装包
2. 修改配置文件 conf下
3. 配置环境变量
4. Linux在bin启动sonar-scanner.sh,
   windows在项目配置的sonar.properties目录下输入cmd,再执行sonar-scanner命令
5. 执行完毕即可访问sonarqube

#### 集成Jenkins插件
1. 在插件中心安装sonarqube插件
2. 系统管理>系统设置与全局工具设置，配上sonar-scanner的目录与sonarqube的地址
3. 在Jinkens任务项目下配置中：

新增一个构建步骤，选择Execute SonarQube Scanner，选择你的jdk版本，若没有，请在全局工具配置中配置好jdk位置。再选择好sonarqube scanner的版本。

> Task to run 输入框中输入 scan，即分析代码；
Path to project properties：可选择的输入框，可以指定一个 sonar-project.properties 文件，如果不指定则使用项目默认的 properties 文件；
Analysis properties：输入一些配置参数传递给 SonarQube，这里的参数优先级高于 sonar-project.properties 文件里面的参数，所以可以在这里来配置所有的参数以替代 sonar-project.properties 文件
注：SonarQube Scanner配置可以直接在项目根目录中创建一个文件sonar-project.properties，然后使用Path to project properties中指定属性文件，或者直接在Analysis Properties中配置
Additional arguments：可以输入一些附加的参数，示例中的-X指进入 SonarQube Scanner 的 Debug 模式，输出更多的日志信息

```
#项目key (随意输入，必填项)
sonar.projectKey=my-job

#项目名称和版本（必填项）
sonar.projectName=my-job
sonar.projectVersion=1.0

#源码位置（必填项，相对于jenkins的workspace路径，例如，我此时的绝对路径为~/.jenkins/workspace/Test/test-webapp/src/main/java）
sonar.sources=test-webapp/src/main/java

#编译后的class位置（必填项，旧版本此项可不填，建议还是填入，相对路径同上）

sonar.java.binaries=test-webapp/target/classes

```

#####查看日志

cd /logs

日志分为sonar.log，es.log，web.log，若不明原因启动失败，可依次查看这几个日志。




###### es报错


```
ERROR: [3] bootstrap checks failed[1]: 
max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
[2]: max number of threads [1024] for user [lee] is too low, increase to at least [4096]
[3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

```

解决：切换到root用户，编辑limits.conf 添加类似如下内容


```
vi /etc/security/limits.conf 
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096
```



```
max number of threads [1024] for user [lish] likely too low, increase to at least [2048]
```

这个命令也可以 ulimit -u 2048

vi /etc/security/limits.d/90-nproc.conf 

修改如下内容：

* soft nproc 1024

#修改为

* soft nproc 2048

问题
```
：max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]
```

解决：切换到root用户修改配置sysctl.conf

vi /etc/sysctl.conf 

添加下面配置：

vm.max_map_count=655360

并执行命令：

sysctl -p

  

###### 官方建议
- vm.max_map_count is greater or equals to 262144
- fs.file-max is greater or equals to 65536
- the user running SonarQube can open at least 65536 file descriptors
- the user running SonarQube can open at least 2048 threads
```
sysctl -w vm.max_map_count=262144
sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 2048
```
> To set these values more permanently, you must update either /etc/sysctl.d/99-sonarqube.conf (or /etc/sysctl.conf as you wish) to reflect these values.
> 
> If the user running SonarQube (sonarqube in this example) does not have the permission to have at least 65536 open descriptors, you must insert this line in /etc/security/limits.d/99-sonarqube.conf (or /etc/security/limits.conf as you wish):
> 

```
sonarqube   -   nofile   65536
sonarqube   -   nproc    2048
```




```
es[][o.e.b.JNANatives] unable to install syscall filter:
java.lang.UnsupportedOperationException: seccomp unavailable: requires kernel 3.5+ with CONFIG_SECCOMP and CONFIG_SECCOMP_FILTER compiled in
        at org.elasticsearch.bootstrap.SystemCallFilter.linuxImpl(SystemCallFilter.java:329) ~[elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.SystemCallFilter.init(SystemCallFilter.java:617) ~[elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.JNANatives.tryInstallSystemCallFilter(JNANatives.java:260) [elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Natives.tryInstallSystemCallFilter(Natives.java:113) [elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:108) [elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:170) [elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:333) [elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:159) [elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:150) [elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86) [elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:124) [elasticsearch-cli-6.6.2.jar:6.6.2]
        at org.elasticsearch.cli.Command.main(Command.java:90) [elasticsearch-cli-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:116) [elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:93) [elasticsearch-6.6.2.jar:6.6.2]

```

原因:  因为Centos6不支持SecComp,而ES默认bootstrap.system_call_filter为true进行检测,所以导致检测失败,失败后直接导致ES不能启动解决:修改elasticsearch.yml 添加一下内容 ：
bootstrap.system_call_filter: false

或者在配置文件里
sonar.search.javaAdditionalOpts=-Dbootstrap.system_call_filter=false

```
java.lang.RuntimeException: can not run elasticsearch as root
        at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:103) ~[elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:170) ~[elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:333) [elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:159) [elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:150) [elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86) [elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:124) [elasticsearch-cli-6.6.2.jar:6.6.2]
        at org.elasticsearch.cli.Command.main(Command.java:90) [elasticsearch-cli-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:116) [elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:93) [elasticsearch-6.6.2.jar:6.6.2]

```

解决办法：以其他用户启动

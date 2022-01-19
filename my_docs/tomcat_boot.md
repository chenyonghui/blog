# Tomcat 介绍

[TOC]

# 简介

Tomcat是Apache 软件基金会（Apache Software Foundation）的Jakarta 项目中的一个核心项目，由[Apache](https://baike.baidu.com/item/Apache/6265)、Sun 和其他一些公司及个人共同开发而成。

Tomcat 服务器是一个免费的开放源代码的Web 应用服务器，属于轻量级应用，由于有了Sun 的参与和支持，最新的Servlet 和JSP 规范总是能在Tomcat 中得到体现，支持运行JSP 页面和Servlet。

![](media/pic/tomcat/tomcat_0.png)

Tomcat 之父詹姆斯•邓肯•戴维森（James Duncan Davidson）希望Tomcat最终能够开源。由于大部分开源项目O'Reilly（世界上在[UNIX](https://baike.baidu.com/item/UNIX)、X、[Internet](https://baike.baidu.com/item/Internet/272794)和其他开放[系统](https://baike.baidu.com/item/%E7%B3%BB%E7%BB%9F)图书领域具有领导地位的出版公司）都会出一本相关的书，并且将其封面设计成某个动物的素描，而且他希望这种动物能够自己照顾自己。最终，他将其命名为Tomcat（英语公猫或其他雄性猫科动物）。

因为Tomcat 技术先进、性能稳定，而且免费，因而深受Java 爱好者的喜爱并得到了部分软件开发商的认可，成为比较流行的Web 应用服务器。

# 目录

![Tomcat目录结构说明](media/pic/tomcat/tomcat_2.png "Tomcat目录结构说明")

<center style="font-size:14px;text-decoration:underline">Tomcat目录结构说明</center> 


![](media/pic/tomcat/tomcat_1.png)
<center style="font-size:14px;text-decoration:underline">server.xml</center> 
![](media/pic/tomcat/tomcat_3.png)

# 总体架构

![](media/pic/tomcat/tomcat_4.jpg)
<center style="font-size:14px;text-decoration:underline">Tomcat 架构图</center> 
![](media/pic/tomcat/tomcat_5.png)
<center style="font-size:14px;text-decoration:underline">Jetty 架构图</center> 

# 生命周期

**核心类：**

两大组件：Connector、Container

**关注点：** 生命周期

**流程  ：**

初始化流程、启动流程、请求处理流程
<figure class="third">
<img src="media/pic/tomcat/tomcat_7.png"  width="35%" />
<img src="media/pic/tomcat/tomcat_6.png" width="30%"/>
<img src="media/pic/tomcat/tomcat_8.png"  width="30%" />
</figure>


- 生命周期方法调用：
实际调用实现类的xxxinitInternal（）

<img src="media/pic/tomcat/tomcat_9.png" style="zoom:80%;" />

![](media/pic/tomcat/tomcat_11.png)


- 设置生命周期状态并触发监听器事件

<img src="media/pic/tomcat/tomcat_10.png" style="zoom:80%;" />

![](media/pic/tomcat/tomcat_12.png)

生命周期状态在调用生命周期方法前、中、后时开始改变,而监听器随着对应生命周期状态的设置被唤醒

![](media/pic/tomcat/tomcat_13.png)

![](media/pic/tomcat/tomcat_14.png)



## 初始化流程

![](media/pic/tomcat/tomcat_15.png)

1. startup.bat/sh-->bootstrap.jar-->Bootstrap.main()
2. 实例化org.apache.catalina.startup.Bootstrap, 初始化类加载器 ，反射调用Catalina.start()
3. load()-->parseServerXml()  Digester  解析  server.xml 并生成StandardServer实例，调用Catalina的setServer（）
4. getServer().init()，实际调用的是实现类的initInternal()
  * 注册Mbean
  * ShareClassLoader加载jar
  * Service.init()
5. Service.initInternal()
  * Engine.init()
  * 线程池初始化，用于子容器的初始化
  * mapperListener初始化用于映射请求到容器
  * Connector.init()
6. Engine.initInternal()，初始化startStopExecutor，用于start（）阶段启停子容器
7. Connector.initInternal  ()
  * 实例化CoyoteAdapter，用于Coyote到HttpServlet的request和response的适配
  * 初始化protocolHandler-->初始化endpoint-->endpoint.bind()-->配置实例化ServerSocketChannel，绑定端口

### 类加载器

- **BoostrapClassLoader**  ： 启动类加载器，它是用本地代码实现的类装载器，负责将 JDK 中 jre/lib 下的 类库 或者 XBootClassPath指定的类库加载到内存中，C++实现，开发者无法直接获取到该加载器的引用（本地代码实现），所以无法通过引用操作

- **ExtClassLoader**  ： 拓展类加载器，由sun公司sun.misc.Launcher$ExtClassLoader实现的，负责将JDK中 jre/lib/ext 或者 -Djava.ext.dir指定的类库加载到内存，开发者可以直接使用拓展类加载器

- **AppClassLoader**  ： 系统加载器，，由sun公司sun.misc.Launcher$AppClassLoader实现的，负责将类路径 java -classpath 或者 -Djava.class.path 指定的类库加载到内存中，开发者可以直接使用系统加载器

- **CatalinaLoader**  ： 负责加载tomcat中catalina-home/lib下的类库加载到内存，由配置文件指定路径：

- **CommonLoader**     和   **SharedLoader**     ：  负责加载catalina-home/conf/catalina.properties中具体指定的类库到内存：

- **ParallelWebappClassLoader**  ： 负责加载tomcat中每个应用的类包，每个应用对应一个 ParallelWebappClassLoader。StandardContext中创建该类加载器并进行应用依赖加载*因为一个StandardContext对应一个应用，所以可以通过不同ParallelWebappClassLoader加载不同应用来实现类加载依赖隔离。

<figure class="half">
<img src="media/pic/tomcat/tomcat_16.png"  width="40%" />
<img src="media/pic/tomcat/tomcat_17.png" width="50%"/>
</figure>



#### ParallelWebappClassLoader

![](media/pic/tomcat/tomcat_19.png)

![](media/pic/tomcat/tomcat_18.png)

![](media/pic/tomcat/tomcat_20.png)

### Digester解析

Apache的Digester库是专门用解析管理xml文档，在tomcat中也使用了这个第三方类库来解析xml文档，包括server.xml和web.xml

**模式**： 对于下图xml文档，根元素是Server，那么Server元素的模式就是Server，对于Server的子元素Service，模式是Server/Service,模式用来标识唯一元素。模式的规则：父元素名+"/"+子元素名
![](media/pic/tomcat/tomcat_22.png)



**规则**： 规则是指当Digester对象开始解析某个元素的时候需要执行的一个或者多个动作。规则的实例是org.apache.commons.digester.Rule。Digester可以包含0个或者多个规则。当Digester类匹配到对应模式的元素的开始标签的时候，会调用对应Rule对像的begin方法，当解析到标签内部的时候会触发body方法，当解析标签结束的时候会触发end方法。
![](media/pic/tomcat/tomcat_21.png)

Tomcat使用解析server.xml部分代码：


<figure class="half">
<img src="media/pic/tomcat/tomcat_23.png"  width="50%" />
<img src="media/pic/tomcat/tomcat_24.png" width="45%"/>
</figure>

#### org.apache.tomcat.util.digester常用方法：
1. 创建对象
![](media/pic/tomcat/tomcat_25.png)
1. 设置属性
![](media/pic/tomcat/tomcat_26.png)

3. 调用方法：在遇到与该规则相关联的匹配模式的时候调用内部栈最顶端的对象的某个方法

![](media/pic/tomcat/tomcat_27.png)

4. 创建对象之间的关系：

Digester  内部有个对象栈，用于临时存储创建的对象  ，当使用addObjectCreate  方法实例化一个对象的时候，创建成功的对象就会被  push  到栈中  ,  调用对象被作为第一个对象被 push 到栈中 。当调用2次addObjectCreate方法以后栈中就存在两个对象，这时候再调用addSetNext方法，那么就会把第二个对象作为参数传递给第一个对象指定的方法。

![](media/pic/tomcat/tomcat_28.png)
5. 添加规则集
![](media/pic/tomcat/tomcat_30.png)

6. 自定义规则处理：继承Rule，重写begin、body、end
   ![](media/pic/tomcat/tomcat_31.png)
   ![](media/pic/tomcat/tomcat_29.png)


Tomcat解析server.xml关键规则：

1. Catalina调用Digester解析时，作为内部栈的第一个对象，解析Server时进行Servers设置：

![](media/pic/tomcat/tomcat_32.png)

2. 添加ListenerCreateRule、ConnectorCreateRule自定义规则来初始化Listener、Connector：

![](media/pic/tomcat/tomcat_33.png)

3. 添加HostRuleSet，HostRuleSet中添加HostConfig用于启动阶段应用的部署（核心） ：

![](media/pic/tomcat/tomcat_34.png)



### Servicie.init()

![](media/pic/tomcat/tomcat_35.png)

### Connector.init()

![](media/pic/tomcat/tomcat_36.png)

![](media/pic/tomcat/tomcat_37.png)

![](media/pic/tomcat/tomcat_38.png)

## 启动流程

![](media/pic/tomcat/tomcat_39.png)

init阶段执行完，进入到start阶段，getServer().start()，实际调用的是实现类的startInternal()
1. Server.startInternal()
  * configure_start监听事件调用
  * globalNamingResources启动
  * Service启动
2. Service.startInternal  ()
  * Engine.start()
  * 请求工作线程池Executor.start()
  * mapperListener.start()，注册context、wrapper，并生成host及context路径映射
  * Connector.start()
3. Connector.start  ()-->  Endpoint.startInternal  ()
  * 开启Poller线程组用于监听Socket事件
  * 创建Accepter启动监听
4. Engine.startInternal()
  * pipeline机制开启
  * Host.start()
5. Host.startInternal  ()
  * 设置用于错误报告的ErrorReportValve
  * 设置生命周期为starting唤醒 HostConfig
    * HostConfig.start()部署应用
    * 生成Context容器并设置ContextConfig
  * Context.start()
6. Context.startInternal  ()
  * 创建webapp类加载器
  * 触发 ContextConfig 生命周期事件，解析web.xml,获取配置的Servlet、Filter、Listener，生成Wrapper
  * Wrapper.start()
7. Wrapper.startInternal()

### Service.start()

![](media/pic/tomcat/tomcat_40.png)

### Connector.start()

![](media/pic/tomcat/tomcat_41.png)

![](media/pic/tomcat/tomcat_42.png)

ContainerBase是Container下四个容器的父类，调用生命周期方法会执行父类ContainerBase的通用方法：

![](media/pic/tomcat/tomcat_44.png)

![](media/pic/tomcat/tomcat_43.png)

![](media/pic/tomcat/tomcat_45.png)

### HostConfig解析

![](media/pic/tomcat/tomcat_46.png)

![](media/pic/tomcat/tomcat_48.png)

![](media/pic/tomcat/tomcat_47.png)

![](media/pic/tomcat/tomcat_49.png)

![](media/pic/tomcat/tomcat_50.png)

![](media/pic/tomcat/tomcat_52.png)

![](media/pic/tomcat/tomcat_51.png)

![](media/pic/tomcat/tomcat_53.png)



## 请求处理流程

![](media/pic/tomcat/tomcat_55.png)

![](media/pic/tomcat/tomcat_56.png)

### NIO模型

* NIO（Non-blocking I/O ）特点：
  * 非阻塞式I/O模型。（服务器端提供一个单线程的Selector来统一管理客户端socket接入的所有连接，并负责每个连接所关心的事件）
  * 弹性伸缩能力强。（服务器端不再是通过多个线程来处理请求，而是通过一个线程来处理所有请求，对应关系变为 N:1）
  * 单线程节省资源。（线程频繁创建、销毁，线程之间的上下文切换等带来的资源消耗得要缓解）
* NIO核心类
  * Channel：通道
  * Buffer：缓冲区
  * Selector：选择器 或 多路复用器

![](media/pic/tomcat/tomcat_54.png)

### Connector解析

LimitLatch：maxConnections，指tomcat能同时接受的最大请求数，-1不启用LimitLatch

* Connector连接数的配置：
  * maxConnections：指tomcat能同时接受的最大请求数，使用LimitLatch来实现，-1不启用LimitLatch
  * acceptCount：tcp全连接队列大小原理是设置backlog参数，服务器的端口全连接队列大小由操作系统的		      somaxconn及server.xml中accpectCount的最小值决定
  * MaxThreads：请求处理线程池的最大线程数

![](media/pic/tomcat/tomcat_57.png)

合理的设置maxConnections、acceptCount、MaxThreads可以提高Tomcat性能，避免请求过多导致全连接队列满的请求失败

Acceptor：请求接收器，监听绑定的端口并将请求封装成Channel放入请求等待处理队列

Poller：NIO模型中的Selector，轮询从等待处理队列中选择Channel包装成SocketProcessor将给线程线处理

SocketProcessor：任务定义器，通过读取socket字节流、解析http协议生成对应任务

Excutor：处理传入的SocketProcessor，CoyoteAdapter->EngineValve···->WrapperValve->FilterChain->Servlet


### Tomcat

#### 1.Tomcat基础

##### 1.1web概念

###### 1.软件架构

1.B/S:浏览器/服务器端架构

2.C/S:客户端/服务器端架构

###### 2.资源分类

1.静态资源：html，css，js（可被浏览器直接解析的资源）

2.动态资源：jsp，servlet，php，asp（动态资源被访问后，要先被转化为静态资源再返回给浏览器解析）

###### 3.网络协议三要素

1.ip：唯一标识一台主机

2.端口号：唯一标识一个应用

3.传输协议：规定了数据传输的规则

​	1.基础协议

​		1.tcp：安全协议，三次握手，速度慢

​		2.udp：不安全的协议，速度快

##### 1.2常见的web服务器

###### 1.概念

服务器：安装了服务器软件的计算机

服务器软件：接收用户请求，处理请求，做出响应

web服务器软件：接收用户请求，处理请求，做出响应

​	在web服务器软件中，可以部署web项目，让用户通过浏览器来访问这些项目

###### 2.常见的web服务器软件

1.webLogic:oracle公司，大型javaee服务器，收费

2.webSphere:大型javaee服务器，收费

3.JBOSS：JBOSS公司，大型javaee服务器，收费

4.Tomcat:中小型javaee服务器，免费

#### 2.Tomcat架构

##### 2.1http工作原理

​	http协议是浏览器与服务器之间的数据传输协议，作为应用层协议，基于TCP/IP协议来传递数据，http不涉及数据包传输，主要规定了服务器与客户端的通信格式。

![屏幕快照 2019-12-17 下午4.01.59](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午4.01.59.png)

流程：

![屏幕快照 2019-12-17 下午4.10.15](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午4.10.15.png)

​	Tomcat作为一个http服务器，在这个过程中主要做了**接收连接、解析请求数据、处理请求和发送响应**

##### 2.2Tomcat整体架构

###### 1.http服务器请求处理

​		浏览器发送给服务端的是一个http格式的请求，http接收到这个请求时，需要调用服务端程序（java类）来处理。

![屏幕快照 2019-12-17 下午4.13.46](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午4.13.46.png)

![屏幕快照 2019-12-17 下午4.16.37](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午4.16.37.png)

###### 2.servlet容器工作流程

![屏幕快照 2019-12-17 下午4.22.23](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午4.22.23.png)

###### 3.Tomcat整体架构

两个核心功能

​	1.处理Socket连接，负责网络字节流与 Request、Response对象的转化。-->**连接器**：对外交流

​	2.加载和管理Servlet，以及具体处理Request请求。-->**容器**：内部处理

![屏幕快照 2019-12-17 下午4.31.07](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午4.31.07.png)

##### 2.3连接器--Coyote

###### 1.架构介绍

![屏幕快照 2019-12-17 下午4.37.20](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午4.37.20.png)

**主要作用**：客户端通过Coyote与服务器建立连接、发送请求与接收响应

![屏幕快照 2019-12-17 下午4.39.30](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午4.39.30.png)

###### 2.IO模型与协议

![屏幕快照 2019-12-17 下午4.43.01](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午4.43.01.png)

​	Tomcat为了实现**支持多种I/O模型和应用层协议，一个容器可以对接多个连接器**，但是单独的连接器或者容器都不能对外提供服务，需要组装起来才能工作，组装后的整体叫做Service组件。Service仅仅是把连接器和容器组装到一起，本身并未做重要的事情。Tomcat可能有多个Service，这样的设计出于灵活性的考虑。**通过在Tomcat配置多个Service，可以实现通过不同的端口号来访问同一台机器上的不同应用**。

###### 3.连接器组件

![屏幕快照 2019-12-17 下午5.13.07](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午5.13.07.png)

各组件功能：

1.EndPoint：具体的Socket接收和发送对象

![屏幕快照 2019-12-17 下午5.18.12](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午5.18.12.png)

2.Processor：实现HTTP协议，将字节流解析为Request和Response对象

![屏幕快照 2019-12-17 下午5.19.20](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午5.19.20.png)

3.ProtocalHandler：实现针对具体协议的处理能力

![屏幕快照 2019-12-17 下午5.19.26](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午5.19.26.png)

4.Adapter：将Request转换为ServletRequset（适配器模式）

![屏幕快照 2019-12-17 下午5.19.33](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午5.19.33.png)

##### 2.4容器--Catalina

​	Tomcat是一个由一系类可配置的组件构成的Web容器，而Catalina是Tomcat的Servlet容器。

###### 1.Catalina地位

Tomcat的模块分层结构图

![屏幕快照 2019-12-17 下午5.39.27](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午5.39.27.png)

Tomcat本质是一款Servlet容器，Catalina才是Tomcat的核心，其他模块都是对Catalina提供支撑的。

###### 2.Catalina结构

![屏幕快照 2019-12-17 下午5.47.52](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午5.47.52.png)

Catalina负责管理Server（表示整个服务器），Server下有多个Service，每个服务都包含多个连接器组件Connector（Coyote）和一个容器组件Cotainer，当Tomcat启动时，会初始化一个Catalina的实例



Catelina各组件的职责：

| 组件      | 职责                                                         |
| --------- | ------------------------------------------------------------ |
| Catalina  | 负责解析Tomcat的配置文件，以此来常见服务器Server组件，并根据命令对其管理 |
| Server    | 服务器表示整个Catalina Servlet容器及其他组件，负责组装并启动Servlet引擎，Tomcat连接器。Server通过实现Lifecycle接口，提供了一种优雅的启动和关闭整个系统的方式 |
| Service   | 服务是Server内部的组件，一个Server包含多个Service。他将多个连接器组件帮到一个Container上 |
| Connector | 连接器，处理与客户端的通信，他负责接收客户请求，然后转给相应的容器处理，最后想客户返回响应结果 |
| Container | 容器，负责处理用户的Servlet请求，并返回对象给web用户的模块   |

###### 3.Container结构

​	Tomcat设计了四种容器，分别是Engine、Host、Context、Wrapper。这四种容器是父子关系，通过此分层架构，使得Servlet容器具有很好的灵活性。	

![屏幕快照 2019-12-17 下午6.29.48](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午6.29.48.png)

各组件含义：

| 容器    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| Engine  | 表示整个Catalina的Servlet引擎，用来管理多个虚拟站点，一个Service最多**只能有一个Engine**，但是一个引擎可以包括多个Host |
| Host    | 代表一个虚拟主机，或者说是一个站点，可以给Tomcat配置多个虚拟主机地址，而一个虚拟主机下可包括多个Context |
| Context | 代表一个Web应用程序，一个Web应用可包含多个Wrapper            |
| Wrapper | 表示一个Servlet，Wrapper作为最底层，不能包括子容器           |

​	Tomcat一设计模式中的**组合模式**来管理这些容器，所有容器组件都实现了Container接口，因此组合模式可以使得用户对单容器对象(Wrapper)和组合容器对象(Wrappe的父类)的使用具有一致性

![屏幕快照 2019-12-17 下午6.41.59](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午6.41.59.png)

##### 2.5Tomcat启动流程

###### 	1.流程

![屏幕快照 2019-12-17 下午6.50.57](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午6.50.57.png)

步骤：

![屏幕快照 2019-12-17 下午7.00.30](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午7.00.30.png)

**总结：加载Tomcat的配置文件，初始化容器组件，监听对应的端口号，准备接受客户端的请求。**

###### 2.源码解析

1.Lifecycle接口：基于生命周期管理抽象而成的接口

方法：

- init()：初始化组件
- start()：启动组件
- stop()：停止组件
- destory()：销毁组件

![屏幕快照 2019-12-17 下午7.11.35](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午7.11.35.png)

2.各组件的默认实现

![屏幕快照 2019-12-17 下午7.12.42](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午7.12.42.png)

![屏幕快照 2019-12-17 下午7.17.33](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午7.17.33.png)

![屏幕快照 2019-12-17 下午7.17.24](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-17 下午7.17.24.png)

###### 3.总结

![屏幕快照 2019-12-21 上午9.37.37](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 上午9.37.37.png)

##### 2.6Tomcat请求流程

###### 1.流程

Tomcat通过Mapper组件来确定每一个请求应该用哪个wrapper容器里的Servlet处理。

**Mapper组件的功能就是通过用户请求的URL来定位到一个Servlet**，他的工作原理是：**Mapper组件保存了Web应用的配置信息（容器组件与访问路径的映射关系）**。比如hsot容器里配置的域名，Context容器里的Web应用路径，以及Wrapper容器里的Servlet映射的请求路径。

当一个请求到来时，Mapper组件通过解析请求的URL里的域名和路径，再到自己保存的map里去查找，就能定位到一个Servlet。



![屏幕快照 2019-12-21 上午9.54.24](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 上午9.54.24.png)

*设计架构层面的处理流程分析：*

![屏幕快照 2019-12-21 上午10.02.21](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 上午10.02.21.png)



###### 2.源码解析

![屏幕快照 2019-12-21 上午10.32.52](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 上午10.32.52.png)

Tomcat各组件各司其职，组件之间松耦合，确保了整体架构的伸缩性与扩展性，在组件内部，每个Container组件采用了**责任链模式**来完成具体的请求处理。

在Tomcat中定义了Pipeline和Valve两个接口，**Pipeline用于构建责任链，Valve代表责任链上的每个处理器。**Popeline中维护了一个基础的Valve，它始终位于Pipeline的末端，封装可具体的处理和输出响应的过程。我们可以调用addValve()为Pipeline添加其他的Valve，后添加的位于基础的Valve之前，并按照添加顺序执行，Pipeline通过获得第一个Vavle来启动整掉链条的执行

#### 3.Jasper

##### 3.1简介

​	Jasper模块是Tomcat的JSP核心引擎，Jsp本质为一个Servlet，**Tomcat使用 Jasper对JSP进行解析，生成Servlet并生成class字节码文件**，**用户在访问jsp时，最终将访问的结果直接响应在浏览器端**。在运行期间Jasper会检测JSP是否修改，如果修改则重新编译。



图示：（红框内表示Jasper的作用）

![屏幕快照 2019-12-21 上午10.48.17](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 上午10.48.17.png)

##### 3.2jsp编译方式

###### 1.运行时编译

​	在tomcat的config目录下的文件web.xml中配置了JspServlet，该servlet实现即时运行时编译的入口

![屏幕快照 2019-12-21 上午11.20.05](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 上午11.20.05.png)

流程图

![屏幕快照 2019-12-21 上午11.21.35](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 上午11.21.35.png)

编译结果：

![屏幕快照 2019-12-21 上午11.26.34](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 上午11.26.34.png)

###### 2.预编译

![屏幕快照 2019-12-21 上午11.27.36](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 上午11.27.36.png)

#### 4.Tomcat服务器配置

##### 4.1 server.xml

​	server.xml是tomcat服务器的核心配置文件，包含了tomcat的servlet容器（catalina）的所有配置。

###### 1.server

   server内嵌的子元素有listener、GlobalNamingResources、service

1.1默认配置的五个监听器：

```xml
 <!-- 用以日志形式输出服务器、os、jvm的版本信息-->
<Listener className="org.apache.catalina.startup.VersionLoggerListener" />
<!-- 用以加载和销毁APR-->
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
 <!-- 用以避免JRE内存泄漏问题-->
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
 <!-- 用以加载和销毁的群居命名服务-->
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
 <!-- 用以在Context停止时冲减Executor池中的线程，避免ThreadLocal内存泄漏-->
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
```

默认配置的全局命名服务

```xml
<GlobalNamingResources>
    
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>
```



**1.3service**

该元素用于创建Service实例，默认使用StrandardService

内嵌元素为：

- listener：用于为Service添加生命周期监听器
- executor：用于配置Service共享线程池
- connector：用于配置Service包含的连接器
- engine：用于配置Service中连接器对应的Servlet容器引擎

```xml
<Service name="Catalina">
	...
</Service>
```

一个Server可以包含多个Service



1.3.1Executor

未配置则每个连接器用一个共享线程池，如果想添加一个共享线程池线程池（添加后每个连接器可以使用配置的共享线程池），可以在service添加如下配置

```java
<Executor 
  name="tomcatThreadPool" 
  namePrefix="catalina-exec-"
  maxThreads="150" 			最大线程数
  minSpareThreads="4"		核心线程数
  maxIdleTime="6000"		最大空闲时长
  maxQueueSize="" 			阻塞队列
/>
```

![屏幕快照 2019-12-21 下午12.02.13](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 下午12.02.13.png)



1.3.2Connector

```xml
<!--http协议的连接器-->
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
<!--ajp协议的连接器-->
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
```

属性：

- port：监听的端口号
- protocol：当前connector支持的访问协议，默认为为HTTP/1.1 ,并采用自动切换机制选择一个局域Java Nio的连接器或者基于本地APR的链接器
- connectionTimeout：接收连接后的等待超时时间（ms），-1表示不超时
- redirectPort：客户端传过来的请求访问的资源需要 https协议来进行访问，此时会自动重定向到指定的端口号
- executor：指定共享线程池的名称
- URIEncoding：指定uri的字符编码



1.3.3Engine

​	Engine作为Servlet引擎的顶级元素，内部可以嵌入:Cluster、listener、Realm、Valve、Host

```xml
<Engine name="Catalina" defalutHost="localhost">
	...
</Engine>
```

defalutHost:默认使用的虚拟主机名称



1.3.3.1Host

​	配置一个虚拟主机 ![屏幕快照 2019-12-21 下午1.01.57](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 下午1.01.57.png)

**可以在一个引擎下配置多个host来访问不同的web应用**



1.3.3.1.1Context

​	Context用于配置一个WEB应用（域名之后的项目路径）

![屏幕快照 2019-12-21 下午1.12.41](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 下午1.12.41.png)

![屏幕快照 2019-12-21 下午1.16.33](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 下午1.16.33.png)

##### 4.2tomcat-users.xml

该配置文件中，主要配置的是Tomcat的用户角色等信息，用来控制Tomcat中manager，host-manager的访问权限（详见6.Tomcat管理配置）



#### 5.Web应用配置

##### 5.1 ServletContext初始化参数

```xml
在web.xml中配置：

<context-param>
	<param-name>test</param-name>
  <param-value>test01</param-value>
</context-param>
```

```java
//java获取初始化参数
void doGet(HttpServletRequest req,...){
  req.getServletContext().getInitParameter("test");
}
```

##### 5.2会话配置

​	用于配置Web应用会话，包括超时时间、Cookie配置以及会话追踪模式。它将覆盖server.xml和context.xml的配置

会话流程：

![屏幕快照 2019-12-21 下午6.51.42](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 下午6.51.42.png)

- 浏览器向服务器发起无状态的http请求，服务器将此次会话的信息存在session中，然后返回给浏览器名为JSESSIONID的会话信息到cookie对象
- 下次浏览器发出的无状态的http请求则会附带Cookie中存放的JSESSINID

会话配置：  

```XML
<session-config>
	<session-timeout>30</session-timeout>//超时时间20分钟
  <cookie-config>
  	<name>JSESSIONID</name>//cookie名
    <domain>www.it.com</domain>//域名
    <path>/</path>
    <comment>Session Cookie</comment>//注释信息
    <http-only>true</http-only>//cookie对象只能通过http方式新型访问，js无法读取和修改。
    <secure>false</secure>//此cookie只能通过https连接产地到服务器，而http连接则不会传递该信息。注意是从浏览器传递到服务器，服务器端的cookie对象不受此影响
    <max-age>3600</max-age>//有效期3600秒
  </cookie-config>
  <tracking-mode>COOKIE</tracking-mode>//跟踪模式,使用cookie跟踪session会话（可配置cookie url ssl）
</session-config>
```

​	*默认session的id值就是JSESSIONID*

##### 5.3Servlet配置

```xml
<servlet>
	<servlet-name>testServlet</servlet-name>
  <servlet-class>study.TestServlet</servlet-class>
  <!--指定当前servlet的初始化参数，可以通过HttpServlet.getInitParamter获取-->
<!--
	<init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
   </init-param>
-->
  <init-param>
  	<param-name>fileName</param-name>
    <param-value>init.conf</param-value>
  </init-param>
  <!--用于控制在Web应用启动时，Servlet的加载顺序。如果值小于0，web应用启动时不加载该servlet,第一次访问时加载（若不配置，则第一次访问时加载）-->
  <load-on-startup>1</load-on-startup>
  <!--true表示当前servlet可接受请求-->
  <enabled>true</enabled>
</servlet>

<servlet-mapping>
	<servlet-name>testServlet</servlet-name>
  <url-pattern>*.do</url-pattern>
	<url-pattern>/myServlet/*</url-pattern>
</servlet-mapping>
```

##### 5.4 Lisetener配置

​	Listener用于监听Servlet中的事件，如context、request、session对象的创建、修改、删除，并触发对应的事件。Listener是**观察者模式**的实现，主要对以上对象的生命周期进行监控。

```xml
<listener>
  <listener-class></listener-class>
</listener>
```

##### 5.5 Filter配置

​	filter用于配置web应用过滤器，用来过滤资源请求及相应。经常用于认证、日志、加密、数据转换等操作

```xml
<filter>
  <filter-name>characterEncodingFilter</filter-name>
  <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
  <!--该过滤器是否支持异步-->
  <async-supported>true</async-supported>
  <!--初始化参数 通过FilterConfig.getInitParameter获取-->
  <init-param>
    <param-name>encoding</param-name>
    <param-value>UTF-8</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>characterEncodingFilter</filter-name>
  <url-pattern> /* </url-pattern>
</filter-mapping>
```

##### 5.6欢迎页面

```xml
<!--在tomcat的web.xml配置如下-->
<welcome-file-list>
  <welcome-file>/index.jsp</welcome-file>
  <welcome-file>/index.html</welcome-file>
  <welcome-file>/index.htl</welcome-file>
</welcome-file-list>
<!--可在项目下的web.xml中配置自己的欢迎页面-->
<welcome-file-list>
  <welcome-file>/login.jsp</welcome-file>
</welcome-file-list>
```

##### 5.7异常页面

```xml
<error-page>
  <error-code>500</error-code>
  <location>/500.html</location>
</error-page>
<error-page>
  <error-code>404</error-code>
  <location>/404.html</location>
</error-page>
<error-page>
	<error-type>java.lang.Exception</error-type>
	<location>/error.jsp</location>
</error-page>
```



#### 6.Tomcat管理配置

##### 6.1Host-manager

​	*管理虚拟主机*

![屏幕快照 2019-12-21 下午8.05.35](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 下午8.05.35.png)

配置如下(tomcat-users.xml)

```xml
<role rolename="admin-gui"/>
<role rolename="admin-script"/>
<user username="test" password="test" role="admin-gui,admin-script"/>
```

#####  6.2 manager

​	*管理web应用*

配置如下(tomcat-users.xml)

```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="test" password="test" role="manager-gui,manager-script"/>
```

#### 7.JVM配置

​	在catalina.sh中配置，在manager中查看

![屏幕快照 2019-12-21 下午8.24.43](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 下午8.24.43.png)

#### 8.Tomcat集群

##### 8.1简介

​	由于单台的tomcat承受能力有限，当我的业务系统用户量比较大，请求压力比较大时，单台tomcat时扛不住的，这个时候，就需要搭建tomcat集群，而目前比较流行的做法就是通过Nginx来实现Nginx来实现tomcat的负载均衡

![屏幕快照 2019-12-21 下午8.31.34](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 下午8.31.34.png)

##### 8.2环境准备

###### 1.准备tomcat

​	在server.xml中更改端口号

| server.xml                                                   | 初始值 | 第一台 | 第二台 |
| ------------------------------------------------------------ | ------ | ------ | ------ |
| <Server port="8005" shutdown="SHUTDOWN">                     | 8005   | 8015   | 8025   |
| <Connector port="8080" protocol="HTTP/1.1"               connectionTimeout="20000" redirectPort="8443" /> | 8080   | 8888   | 9999   |
| <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" /> | 8009   | 8019   | 8029   |



###### 2.安装配置Nginx

在当前服务器上，安装Nginx，然后配置ngnix.conf

```
upstream serverpool{
	server localhost:8888;
	server localhost:9999;
}
server{
	listen 99;					(监听的端口号)			
	server_name locahost;	（访问的服务地址）
	location / {			（拦截的映射的路径为/,也就是根路径）
		proxy_pass http://serverpool/;（反向代理的配置）
	}
}
```

#####  8.3 负载均衡 策略

###### 1.轮询

最基本的配置方法，它是upstream模块默认的负载均衡默认策略。每个请求会按时间顺讯逐一分配到不同的后端服务器

```
upstream serverpool{
	server localhost:8888;
	server localhost:9999;
}
```

参数说明

![屏幕快照 2019-12-21 下午8.59.35](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 下午8.59.35.png)

###### 2.weight权重

权重方式，在轮询策略的基础上指定轮询的几率（适合服务器的硬件配置差别较大的情况）

```
upstream serverpool{
	server localhost:8888 weight=3;
	server localhost:9999 weight=1;
}
```

weight用于指定轮询几率，默认值为1，weight的数值与访问比率成正比

###### 3.ip_hash

指定负载均衡器按照基于客户端ip的分配方式，这个方式确保了相同的客户端的请求一直发送到相同的服务区，以保证session会话，这样每个访客都固定一个后端服务器，可以解决session不能跨服务区的问题

```
upstream serverpool{
	ip_hash;
	server localhost:8888 weight=3;
	server localhost:9999 weight=1;
}
```

##### 8.4 session共享方案

​	在tomcat集群中，如果nginx做了负载均衡，就会出现如下问题

![屏幕快照 2019-12-21 下午9.14.24](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 下午9.14.24.png)

###### 1.ip_hash策略

​	保证一个用户发起的请求只会转发到一台相同的服务器上

###### 2.session复制

​	通过服务器间复制session来确保session共享,小型项目可以使用。大型项目会耗费大量资源（超过四个节点不推荐使用）

步骤：

​	1.编写session.jsp

![屏幕快照 2019-12-21 下午9.29.06](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 下午9.29.06.png)

​	2.在两个tomcat中的server.xml中配置

```xml
<Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>
```

​	.在web.xml中加入标签<distributable/>

###### 3.sso-单点登录

​	单点登录（single sign on），简称sso，是目前比较流行的企业业务整合的解决方案之一。sso的定义是**在多个应用系统中，用户只需要登录一次就可以访问所有的相互信任的应用系统**，也是用来解决集群环境session共享的方案之一

![屏幕快照 2019-12-21 下午9.32.16](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-21 下午9.32.16.png)

**就是用一个专门的认证服务通过查询redis数据库来确定当前用户是否存在session会话**



#### 9.Tomcat安全

##### 9.1 配置安全

1.删除webapps目录下的所有文件，禁用tomcat管理界面

2.注释或删除tomcat-users.xml文件内的所有用户权限

3.更改关闭tomcat指令或禁用

​	tomca的server.xml中定义了可以直接关闭Tomcat实例的管理端口（默认8005）。可以通过telnet连接上该端口（**telnet 域名 8005**）之后没输入SHUDOWN（默认关闭指令）即可关闭Tomcat实例（实例关闭，进程存在）。

​	方案1：更改端口号和指令

```xml
<Server port="8456" shutdown="test_shut"></Server>
```

​	方案2：禁用8005端口

```xml
<Server port="-1" shutdown="SHUTDOWN"></Server>
```

4.定义错误页面

##### 9.2 应用安全

在大部分的Web应用中，特别是一些后台应用系统，都会实现自己的安全管理模块（权限模块），用于控制应用系统的安全访问，基本包含两个部分：认证（登录/单点登录）和授权（功能权限、数据权限）两部分。对于当前的业务系统，可以做一套适用于自己业务系统的权限模块，也有很多的应用系统直接使用一些功能完善的安全框架，将其继承到我们的Web应用中，如：SpringSecurity、Apache Shiro等。

##### 9.3 传输安全

###### 1.HTTPS介绍

​	HTTPS的全称为超文本传输安全协议，是一种网络安全传输协议。在HTTP的基础上加入SSL/TLS来进行数据加密，保护交换数据不被泄露、窃取。



​	SSL和TLS 是用于网络通信安全的加密协议，它允许客户端和服务器之间的安全链接通信。

SSL协议的三个特性

- 保密：通过SSL链接阐述的数据是加密的
- 鉴别：通信双方的身份鉴别，通常是可选的，单至少有一方需要验证
- 完整性：传输数据的完整性检查

从性能角度考虑，加解密是一项计算昂贵的处理，因为尽量不要将整个Web应用采用SSL链接，实际部署过程中，选择有必要进行安全加密的页面采用SSL通信



HTTPS和HTTP的区别主要为以下四点：

- HTTPS协议需要到证书颁发机构CA申请SSL证书，然后与域名进行绑定，HTTP不用申请证书
- HTTP是超文本传输协议，属于应用层信息传输，HTTPS则是具有SSL加密安全性传输协议，对数据的传输进行加密，相当于HTTP的升级版 
- HTTP和HTTPS使用的完全不同的连接方式，用的端口也不一样，前者是8080，后者为8443
- HTTP的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比HTTP协议安全



HTTPS协议优势：

- 提高SEO
- 隐私信息加密，防止流量劫持
- 浏览器受信任

###### 2.Tomcat支持HTTPS

1.生成秘钥文件

![屏幕快照 2019-12-22 上午9.38.23](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-22 上午9.38.23.png)

2.将秘钥文件复制到tomcat/conf目录下

3.配置server.xml

![屏幕快照 2019-12-22 上午9.40.33](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-22 上午9.40.33.png)

4.访问 localhost:8443



#### 10.Tomcat性能调优

##### 10.1 Tomcat性能测试

![屏幕快照 2019-12-22 上午9.48.21](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-22 上午9.48.21.png)

apache beach:https://www.cnblogs.com/liuyu2014/p/11855681.html

![屏幕快照 2019-12-22 上午10.08.58](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-22 上午10.08.58.png)

##### 10.2 Tomcat性能优化

###### 1.JVM参数调优

优化重点：**内存分配和GC策略**

1.内存分配

| 参数                 | 作用                                               | 建议                  |
| -------------------- | -------------------------------------------------- | --------------------- |
| -server              | 启动server，以服务端模式运行（开启慢，处理请求快） | 服务端模式建议开启    |
| -Xms                 | 最小堆内存                                         | 建议与-Xmx设置相同    |
| -Xmx                 | 最大堆内存                                         | 建议设为可用内存的80% |
| -XX:MetaspaceSize    | 初始元空间大小                                     |                       |
| -XX:MaxMetaspaceSize | 最大元空间大小                                     | 默认无限              |
| -XX:MaxNewSize       | 新生代最大内存                                     | 默认16M               |
| -XX:NewRatio         | 年轻代与老年代大小比值,默认2                       | 不建议修改            |
| -XX:SurvivorRatio    | 伊甸园区和幸存区大小比值，默认8                    | 不建议修改            |

在catalina.sh中添加如下配置：

```sh
JAVA_OPTS="-server -Xms2048m -Xmx2048m -XX:MaxMetaspaceSize=256m -XX:MaxMetaspaceSize=512m"
```

```java
jmap -heap 进程id查看
```



2.GC策略

​	GC性能的主要指标

- 吞吐量：工作时间（程序运行事件和内存分配时间）占总时间的百分比
- 暂停时间：由gc导致的应用程序停止响应次数/事件

选择准则：

![屏幕快照 2019-12-22 上午11.02.50](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-22 上午11.02.50.png)



*查看tomcat的默认垃圾回收器*

1.在catalina.sh中加入如下配置

```sh
JAVA_OPTS=" -Djava.rmi.server.hostname=192.169.192.138 -Dcom.sun.management.jmxremote.port=8999 -Dcom.sun.management.jmxremote.rmi.port=8999 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false" 
```

2.打开jconsole ，查看远程tomcat的概要信息 



*更改tomcat的垃圾回收器*

1.在catalina.sh中加入如下配置

```sh
JAVA_OPTS=" -XX:+UseConcMarkSweepGc -XX:+printGCDetails" 
```



###### 2.Tomcat配置调优

​	主要调整链接器的配置(server.xml)，提升应用服务器性能

| 参数           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| maxConnections | 最大连接数，当到达该值时，服务器接收单不会处理更多的请求，额外的请求将会阻塞知道连接数低于该值。可通过ulimit -a查看服务器限制 |
| maxThreads     | 最大线程数，需要根据服务器硬件情况进行合理设置               |
| acceptCount    | 最大排队等待数。一台tomcat的最大请求数量为maxConnections+acceptCount |



#### 11.WebSocket

###### 11.1介绍

![屏幕快照 2019-12-22 上午11.35.14](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-22 上午11.35.14.png)

![屏幕快照 2019-12-22 上午11.38.35](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-22 上午11.38.35.png)

首先，webSocket连接必须由浏览器发起，因为请求协议是一个标准的HTTP请求，格式如下

![屏幕快照 2019-12-22 上午11.41.51](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-22 上午11.41.51.png)

该请求与http请求的几点不同

![屏幕快照 2019-12-22 上午11.42.32](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-22 上午11.42.32.png)

###### 11.2Tomcat对webSocket的支持

![屏幕快照 2019-12-22 上午11.44.38](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-22 上午11.44.38.png)

![屏幕快照 2019-12-22 上午11.48.35](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-22 上午11.48.35.png)

![屏幕快照 2019-12-22 上午11.52.49](/Users/yinwenbo/Documents/img/屏幕快照 2019-12-22 上午11.52.49.png)


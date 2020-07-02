# SpringCloud

## 一：微服务基础

### .系统架构演变

####  1.单体应用架构

​	Web应用早期将所有功能模块放到一个Web容器内运行

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200622105725543.png" alt="image-20200622105725543" style="zoom:50%;" />

##### 优点：

- 所有功能在一个应用中，架构简单，前期开发成本低，周期短

##### 缺点

- 所有功能集中在一个工程中，耦合度高，不易维护
- 系统性能扩展只能通过扩展集群节点，成本高，有瓶颈
- 技术栈无法扩展

#### 2.垂直应用架构

​	单一应用不满足访问量逐渐变大，将单一应用拆分成几个不相干的应用，提升效率

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200622110400613.png" alt="image-20200622110400613" style="zoom:50%;" />

##### 优点：

- 所有功能在一个应用中，架构简单，前期开发成本低，周期短
- 不同项目可采用不同的技术

##### 缺点

- 所有功能集中在一个工程中，耦合度高，不易维护
- 系统性能扩展只能通过扩展集群节点，成本高，有瓶颈

#### 3.分布式SOA架构

##### 1.SOA简介

​	SOA即面向服务的架构，是一个组件模型，它将应用程序的不同功能单元（称为服务）进行拆分，并通过这些服务之间定义良好的接口和协议联系起来。接口是采用中立的方式进行定义的，它应该独立于实现服务的硬件平台、操作系统和编程语言。这使得构件在各种各样的系统中的服务可以以一种统一和通用的方式进行交互

​	特点：分布式、可重用、扩展灵活、松耦合

##### 2.SOA架构

​	当垂直应用逐渐增多，应用之间不可避免交互，将核心业务抽取出来作为独立服务

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200622110713370.png" alt="image-20200622110713370" style="zoom:50%;" />

##### 优点：

- 抽取公共功能为服务，提高开发效率
- 对不同的服务进行集群化部署解决系统压力
- 基于ESB/DUBBO减少系统耦合

##### 缺点

- 抽取服务的力度较大
- 服务提供方和调用方接口耦合度高

#### 4.微服务架构

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200622112553022.png" alt="image-20200622112553022" style="zoom:50%;" />



##### 优点

- 通过服务的原子化拆分，以及微服务的独立打包、部署和升级，小团队的交付周期将缩短，运维成本也将大幅度下降
- 微服务遵循单一原则。微服务之间采用 Restful 等轻量协议传输。

##### 缺点

- 微服务过多，服务治理成本高，不利于系统维护。
- 分布式系统开发的技术成本高（容错、分布式事务等）。



##### 区别：SOA与微服务

**SOA** (Service oriented Architecture）：“面向服务的架构“是一种设计方法，其中包含多个服务，服务之间通过相互依赖最终提供一系列的功能。一个服务通常以独立的形式存在与操作系统进程中。各个服务之间通过网络调用。

**微服务架构：**其实和 SOA 架构类似，微服务是在 SOA 上做的升华，微服务架构强调的一个重点是业务需要**彻底的组件化和服务化**，原有的单个业务系统会拆分为多个可以独立开发、设计、运行的小应用。这些小应用之间通过服务完成交互和集成。微服务就是去掉ESB的SOA

### 2.分布式核心知识

#### 1.分布式中的远程调用

​	在微服务架构中，通常存在多个服务之间的远程调用的需求。远程调用通常包含两个部分：**序列化和通信协议**。常见的序列化协议包括 json、xml、hession、protobuf、thrift、text、bytes 等，目前主流的远程调用技术有基于 HTTP 的 **RESTFUL 接口**以及基于 TCP的**RPC 协议**

#####  1）RESTFUL

​	REST，即 Representational State Transferk 的缩写，如果一个架构符合 REST 原则，就称它为 RESTFUL 架构

###### 资源（Resources)

​	所谓“资源”，就是网络上的一个实体，或者说是网络上的个具体信息。它可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的实在。你可以用一个 URI（统一资源定位符）指向它，每种资源对应一个特定的 UR。要获取这个资源，访问它的 URI 就可以，因此 UR 就成了每一个资源的地址或独一无二的识别符。REST 的名称“表现层状态转化“中，省略了主语。“表现层“其实指的是“资源“(Resources）的'表现层

###### 表现层（Representation)

​	资源是一种信息实体，它可以有多种外在表现形式。我们把资源“具体呈现出来的形式，叫做它的“表现层“(Representation）。比如，文本可以用 txt 格式表现，也可以用 HTML 格式、XML 格式、JSON 格式表现，甚至可以采用二进制格式；图片可以用 PG 格式表现，也可以用 PNG 格式表现。URI 只代表资源的实体，不代表它的形式。严格地说，有些网址最后的”html“后缀名是不必要的，因为这个后缀名表示格式，属于“表现层“”范畴，而 URI 应该只代表“资源的位置。

###### 状态转化（State Transfer)

​	访问一个网站，就代表了客户端和服务器的一个互动过程。在这个过程中，势必涉及到数据和状态的变化。互联网通信协议 HTTP 协议，是一个无状态协议。这意味着，所有的状态都保存在服务器端。因此，如果客户端想要操作服务器，必须通过某种手段，让服务器端发生“状态转化“(State Transfer）。客户端用到的手段，只能是 HTTP 协议。具体来说，就是 HTP 协议里面，四个表示操作方式的动词 GET、POST、PUT、DELETE。

| 操作方式 | 对应操作       |
| -------- | -------------- |
| GET      | 获取资源select |
| POST     | 新建资源insert |
| PUT      | 更新资源update |
| DELETE   | 删除资源delete |

总结：

- 每一个 URI 代表一种资源

- 客户端通过四个HTTP动词，对服务器端资源进行操作，实现“表现层状态转化“。

##### 2）RPC

​	RPC（Remote Procedure Call）：远程过程调用，是一个计算机通信协议。该协议允许运行于一台计算机的程序调用另一台计算机的子程序，而程序员无需额外地为这个交互作用编程，两个或多个应用程序都分布在不同的服务器上，它们之间的调用都像是本地方法调用一样

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200622123655824.png" alt="image-20200622123655824" style="zoom:50%;" />

##### 3）区别和联系

|          | RESTFUL      | RPC       |
| -------- | ------------ | --------- |
| 使用协议 | HTTP         | 一般为TCP |
| 性能比较 | 略低         | 略高      |
| 灵活度   | 高           | 低        |
| 应用情况 | 常见于微服务 | 常见于SOA |

1、HTTP 相对更规范，更标准，更通用，无论哪种语言都支持 http协议。如果你是对外开放 API，例如开放平台，外部的编程语言多种多样，你无法拒绝对每种语言的支持，现在开源中间件，基本最先支持的几个协议都包含 RESTFUL

2、RPC 框架作为架构微服务化的基础组件，它能大大降低架构微服务化的成本，提高调用方与服务提供方的研发效率，屏蔽跨进程调用函数（服务）的各类复杂细节。让调用方感觉就像调用本地函数一样调用远端函数、让服务提供方感觉就像实现一个本地函数一样来实现服务。

#### 2.CAP理论

C：一致性(consistency)，数据一致更新，所有数据的变化都是同步的

A：可用性(availability)，在集群某节点故障后，集群是否还能响应客户端的请求

P：分区容忍性(partition tolerance)，在某节点故障后，并不影响整个系统的运行



​	CAP三者只能取其二（在保证分区容忍性的情况下，某节点宕机后，若保证数据一致则节点肯定不可用，若可用则数据肯定不一致），出现下面三种情况：

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200622124904706.png" alt="image-20200622124904706" style="zoom:50%;" />

| CA   | 放弃分区容忍性，出现故障系统瘫痪，传统关系数据库的选择。     |
| ---- | ------------------------------------------------------------ |
| CP   | 放弃可用性，追求数据的强一致性。常用于金融系统               |
| AP   | 放弃一致性。对读一致性的要求很低，有些场合对写一致性要求并不高，允许实现最终一致性时使用。常用于分布式系统中 |

## 二：SpringCloud

### 1.概述

#### 1.介绍

​	Spring Cloud是一系列框架的有序集合。它利用 Spring Boote 的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务注册与发现、配置中心、消息总线、负載均衡、断路器、数据监控等，都可以用 Spring Boota 的开发风格做到一键启动和部署。Spring Cloud 并没有重复制造轮子，它只是将目前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过 Spring Boot 风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包

官网：https://spring.io/projects/spring-cloud/

文档：https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/htmlsingle/

![image-20200622131252950](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200622131252950.png)

#### 2.核心组件

![image-20200622132803213](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200622132803213.png)

##### 1.服务注册与发现

**服务注册**：服务实例将自身服务信息注册到注册中心。这部分服务信息包括服务所在主机IP和提供服务的 Port，以及暴露服务自身状态以及访问协议等信息。

**服务发现**：服务实例请求注册中心获取所依赖服务信息。服务实例通过注册中心，获取到注册到其中的服务实例的信息，通过这些信息去请求它们提供的服务。

##### 2.负载均衡

负载均衡是高坑用网络基础框架的关键组件，通常用于将工作**负载分布到多个服务器**来提高网站，应用、数据库或其他服务的性能和可靠性

##### 3.服务熔断

​	熔断这一概念来源于电子工程中的断路器（Circuit Breaker）。在互联网系统中，当下游服务因访可压力过大而响应变慢或失败，上游服务为了保护系统整体的可用性，可以暂时**切断对下游服务的调用**。这种牺牲局部，保全整体的措施就叫做熔断。

##### 4.链路追踪

​	随着微服务架构的流行，服务按照不同的维度进行拆分，一次请求往往需要涉及到多个服务。互联网应用构建在不同的软件模块集上，这些软件模块，有可能是由不同的团队开发、可能使用不同的编程语言来实现、有可能布在了几干台服务器，横跨多个不同的数据中心。因此，就需要**对一次请求涉及的多个服务链路进行日志记录**，性能监控即链路追踪

##### 5.服务网关

​	随着微服务的不断増多，不同的微服务一般会有不同的网络地址，而外部客户端可能需要调用多个服务的接口オ能完成一个业务需求，如果让客户端直接与各个微服务通信可能出现

- 客户端需要调用不同的 ur 地址，增加难度

- 再一定的场景下，存在跨域请求的可题

- 每个微服务都需要进行单独的身份认证

针对这些问题，API 网关顺势而生。

服务网关直面意思是将所有 API调用统一接入到服务网关层，由网关层统一接入和输出。一个网关的基本功能有：统一接入、安全防护、协议适配、流量管控、长短链接支持、容错能力。有了网关之后，各个API服务提供团队可以专注于自己的的业务逻辑处理，而 API 网关更专注于安全、流量、路由等问题

#### 3.体系架构

![image-20200622220854348](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200622220854348.png)

### 2.项目搭建

#### 1.前期准备

```yml
java :	8
maven : 3.6.1
springcloud : Hoxton.SR1、alibaba 2.1.0.RELEASE
springboot : 2.2.RELEASE
mysql : 5.7.27
```

boot和cloud的版本选择：https://spring.io/projects/spring-cloud#overview

![image-20200623102613774](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200623102613774.png)

#### 2.父工程搭建

（1）create new project——>maven——>Create from archetype(maven-archetype-site)——>填写工程名——>finish

（2）更改字符编码(utf-8)

（3）java编译版本（8）

（4）pom文件

```xml
 <!-- 统一管理jar包版本 -->
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <junit.version>4.12</junit.version>
    <log4j.version>1.2.17</log4j.version>
    <!--<lombok.version>1.16.18</lombok.version>-->
    <mysql.version>5.1.47</mysql.version>
    <!--<druid.version>1.1.16</druid.version>-->
    <!--<mybatis.spring.boot.version>1.3.0</mybatis.spring.boot.version>-->
  </properties>

	<dependencyManagement>
    <dependencies>
      <!--spring boot 2.2.2-->
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>2.2.2.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <!--spring cloud Hoxton.SR1-->
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>Hoxton.SR1</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <!--spring cloud alibaba 2.1.0.RELEASE-->
      <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-alibaba-dependencies</artifactId>
        <version>2.1.0.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
      </dependency>
      <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.0.9</version>
      </dependency>
      <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.1.0</version>
      </dependency>
      <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>${junit.version}</version>
      </dependency>
      <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>${log4j.version}</version>
      </dependency>
      <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.12</version>
        <optional>true</optional>
      </dependency>
    </dependencies>
  </dependencyManagement>
```

​	**注：在父工程中使用DependencyManagement和Dependencies区别**

```java
DependencyManagement：只是声明依赖，并不实现引入，因此子项目需要显示的声明需要用的依赖。如果不在子项目中声明依赖，是不会从父项目中继承下来的；只有在子项目中写了该依赖项，并且没有指定具体版本，才会从父项目中继承该项，并且version和scope都读取自父pom;另外如果子项目中指定了版本号，那么会使用子项目中指定的jar版本。（子项目重写才引入）

Dependencies：即使在子项目中不写该依赖项，那么子项目仍然会从父项目中继承该依赖项（直接全部引入）
```

 （5）父工程创建完成执行mvn:insall将父工程发布到仓库方便子工程继承

#### 3.微服务工程搭建

##### 1.Cloud-provider-payment8001 微服务提供者Module模块

###### 1.建model

​	NEW-->MODEL-->MODEL SDK(1.8)-->填写名称-->finish

###### 2.改pom

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  <dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
  </dependency>
  <dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
  </dependency>
  <dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.10</version>
  </dependency>
  <!--mysql-connector-java-->
  <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
  </dependency>
  <!--jdbc-->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
```

###### 3.写yml

```yml
server:
  port: 8001
spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource 
    url: jdbc:mysql:///springcloud?useUnicode=true&characterEncoding=utf8
    username: root
    password: root
mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: study.springcloud.entities    # 所有Entity别名类所在包
```

###### 4.主启动

```java
@SpringBootApplication
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class);
    }
}
```

###### 5.业务类

controller

```java
@RestController
public class PaymentController {

  @Autowired
  private PaymentService paymentService;

  @Value("${server.port}")
  private String serverPort;

  @PostMapping(value = "/payment/create")
  public CommonResult create(@RequestBody Payment payment){
    int result=paymentService.create(payment);
    System.out.println("插入结果===="+result);

    if(result>0){
      return new CommonResult(200,"insert success serverPort="+serverPort,result);
    }else {
      return new CommonResult(444,"insert failure");
    }
  }

  @GetMapping(value = "/payment/get/{id}")
  public CommonResult getPaymentById(@PathVariable("id") Long id){
    Payment payment= paymentService.getPaymentById(id);
    System.out.println("查找结果===="+payment);
    if(payment != null){
      return new CommonResult(200,"search success serverPort="+serverPort,payment);
    }else {
      return new CommonResult(444,"search failure");
    }
  }
}
```

service

```java
public interface PaymentService {
  int create(Payment payment);

  Payment getPaymentById(@Param("id") Long id);
}
@Service
public class PaymentServiceImpl implements PaymentService {

  @Autowired
  private PaymentDao paymentDao;

  @Override
  public int create(Payment payment) {

    return paymentDao.create(payment);
  }

  @Override
  public Payment getPaymentById(Long id) {
    return paymentDao.getPaymentById(id);
  }
}

```

dao

```java
@Mapper
public interface PaymentDao {
  int create(Payment payment);

  Payment getPaymentById(@Param("id") Long id);
}

```

domain

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class CommonResult<T> {

  private Integer code;
  private String message;
  private T   data;

  public CommonResult(Integer code, String message) {
    this.code = code;
    this.message = message;
  }
}

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Payment implements Serializable {
    private Long id;
    private String serial;
}
```

resources->mapper-->PaymentMapper

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >

<mapper namespace="study.springcloud.dao.PaymentDao">

  <insert id="create" parameterType="Payment" useGeneratedKeys="true" keyProperty="id">
    insert into payment(serial)  values(#{serial});
  </insert>

  <resultMap id="BaseResultMap" type="study.springcloud.entities.Payment">
    <id column="id" property="id" jdbcType="BIGINT"/>
    <id column="serial" property="serial" jdbcType="VARCHAR"/>
  </resultMap>

  <select id="getPaymentById" parameterType="Long" resultMap="BaseResultMap">
    select * from payment where id=#{id};
  </select>

</mapper>
```

建表sql

```sql
DROP TABLE IF EXISTS `payment`;
CREATE TABLE `payment` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `serial` varchar(200) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=6;

BEGIN;
INSERT INTO `payment` VALUES (1, 'adasd');
INSERT INTO `payment` VALUES (2, 'dasd');
COMMIT;
```

###### 6.测试

http://localhost:8001/payment/get/1

##### 2.cloud-consumer-order80 微服务消费者订单Module模块

###### 1.建model

​	NEW-->MODEL-->MODEL SDK(1.8)-->填写名称-->finish

###### 2.改pom

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>

```

###### 3.写yml

```yml
server:
  port: 80
spring:
  application:
    name: cloud-order-service
```

###### 4.主启动

```java
@SpringBootApplication
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class);
    }
}
```

###### 5.业务类

controller

```java
@RestController
public class OrderController {
  
  public static final String URL="http://localhost:8001";
  @Autowired
  private RestTemplate restTemplate;

  @GetMapping("/consumer/payment/create")
  public CommonResult<Payment> create(Payment payment){
    return restTemplate.postForObject(URL+"/payment/create",payment,CommonResult.class);
  }

  @GetMapping("/consumer/payment/get/{id}")
  public CommonResult<Payment> getPayment(@PathVariable("id") Long id){
    return restTemplate.getForObject(URL+"/payment/get/"+id,CommonResult.class);
  }
}
```

domain

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class CommonResult<T> {

  private Integer code;
  private String message;
  private T   data;

  public CommonResult(Integer code, String message) {
    this.code = code;
    this.message = message;
  }
}

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Payment implements Serializable {
    private Long id;
    private String serial;
}
```

config

```java
@Configuration
public class ApplicationContextConfig {
  @Bean
  public RestTemplate getRestTemplate(){
    return new RestTemplate();
  }
}
```

###### 6.测试

http://localhost/consumer/payment/get/1

##### 3.系统重构

问题：系统有重复部分（domain）

解决：

新建cloud-api-common，此模块防止系统公共使用的资源类

（1）pom:

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.1.0</version>
  </dependency>
</dependencies>
```

（2）domain

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class CommonResult<T> {

  private Integer code;
  private String message;
  private T   data;

  public CommonResult(Integer code, String message) {
    this.code = code;
    this.message = message;
  }
}

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Payment implements Serializable {
    private Long id;
    private String serial;
}
```

（3）maven命令clean install

（4）订单80和支付8001分别改造

- 删除各自的原先的domain文件夹
- 各自粘贴POM内容

```xml
<dependency>
  <groupId>study</groupId>
  <artifactId>cloud-api-commons</artifactId>
  <version>1.0-SNAPSHOT</version>
</dependency>
```

##### 附加：热部署

（1）Adding devtools to your project

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
  <scope>runtime</scope>
  <optional>true</optional>
</dependency>
```

（2）Adding plugin to your pom.xml

```xml
<!--在父工程中加入-->
<!--热部署-->
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <configuration>
        <fork>true</fork>
        <addResources>true</addResources>
      </configuration>
    </plugin>
  </plugins>
</build>
```

（3）Enabling automatic build

![image-20200623115014128](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200623115014128.png)

（4）Update the value of

​	快捷键：ctrl+shift+alt+/  选择Registry	下图中第一行和最后一行打钩

![image-20200623115345083](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200623115345083.png)

（5）重启idea

## 三：服务注册Eureka

### 1.Eureka基础

#### 1.服务治理

​	Spring Cloud 封装了 Netflix 公司开发的 Eureka 模块来实现服务治理

​	在传统的 rpc 远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务于服务之间依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册。

#### 2.服务注册

​	Eureka 采用了 CS 的设计架构，Eureka Server 作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用 Eureka 的客户端连接到 Eureka Server？并维持心跳连接。这样系统的维护人员就可以通过 Eureka Server 3 来监控系统中各个微服务是否正常运行。

​	在服务注册与发现中，有一个注册中心。当服务器启动的时候，会把当前自己服务器的信息比如服务地址通讯地址等以別名方式注册到注册中心上。另一方（消费者服务提供者），以该别名的方式去注册中心上获取到实际的服务通讯地址，然后再实现本地 RPC 调用 。RPC 远程调用框架核心设计思想在于注册中心，因为使用注册中心管理每个服务与服务之间的一个依赖关系（服务治理概念。在任何 rpc 远程框架中，都会有一个注册中心（存放服务地址相关信息接口地址）

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200623182248409.png" alt="image-20200623182248409" style="zoom:50%;" />

#### 3.Eureka两组件

Eureka 包含两个组件：Eureka Server 和 Eureka Client

Eureka Serve 提供服务注册服务：

​	各个微服务节点通过配置启动后，会在 Eureka Serverl 中进行注册，这样 Eureka Server 中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观看到。

Eurekaclient 通过注册中心进行访问：

​	是一个 Java 客户端，用于简化 Eureka Server 的交互，客户端同时也具备一个内置的、使用**轮询**负载算法的负载均衡器。在应用启动后，将会向 Eureka Server发送心跳（默认周期为 30 秒）。如果 Eureka Server 在多个心跳周期内没有接收到某个节点的心跳，Eureka Server 将会从服务注册表中把这个服务节点移除（默认 90 秒）



### 2.Erueka项目构建

#### 1.单机版

##### 1.EurekaServer端构建

（1）建model

​	NEW-->MODEL-->MODEL SDK(1.8)-->填写名称-->finish

（2）改pom

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
  </dependency>
  <dependency>
    <groupId>study</groupId>
    <artifactId>cloud-api-commons</artifactId>
    <version>1.0-SNAPSHOT</version>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
```

（3）写yml

```yml
server:
  port: 7001
eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false #不注册自己
    fetch-registry: false 
    service-url:  #设置eureka server交互地址
      defaultZone: http://eureka7002.com:7002/eureka/
```

（4）主启动

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaMain7001 {
  public static void main(String[] args) {
    SpringApplication.run(EurekaMain7001.class);
  }
}
```

（5）测试

​	http://localhost:7001/

##### 2.将服务提供者8001注册到erueka

更改payment8001模块

（1）改pom

```xml
<!--加入eureka client端依赖-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

（2）写yml

```yml
eureka:
  client:
    register-with-eureka: true #注册自己
    fetchRegistry: true #eureka集群时必须设置为true才能使用ribbon的负载均衡
    service-url:  #设置eureka server交互地址
      defaultZone: http://localhost:7001/eureka
  instance:
    instance-id: order80	#更改eureka界面中的名称显示
    prefer-ip-address: true   #访问路径可以显示IP地址
```

（3）主启动

```java
@SpringBootApplication
@EnableEurekaClient	//添加注解
public class PaymentMain8001 {
  public static void main(String[] args) {
    SpringApplication.run(PaymentMain8001.class);
  }
}
```

（4）测试

​	先启动eureka7001，在启动payment8001，访问http://localhost:7001/

##### 3.将服务消费者order80注册到eureka

更改order80模块

（1）改pom

```xml
<!--加入eureka client端依赖-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

（2）写yml

```yml
eureka:
  client:
    register-with-eureka: true #注册自己
    fetchRegistry: true #eureka集群时必须设置为true才能使用ribbon的负载均衡
    service-url:  #设置eureka server交互地址
      defaultZone: http://localhost:7001/eureka
  instance:
    instance-id: payment8001	#更改eureka界面中的名称显示
    prefer-ip-address: true   #访问路径可以显示IP地址
```

（3）主启动

```java
@SpringBootApplication
@EnableEurekaClient	//添加注解
public class OrderMain80 {
  public static void main(String[] args) {
    SpringApplication.run(OrderMain80.class);
  }
}
```

（4）测试

​	先启动eureka7001，在启动OrderMain80，刷新http://localhost:7001/，访问http://localhost/consumer/payment/get/1

#### 2.集群版

##### 1.集群说明

![image-20200623191137504](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200623191137504.png)

​	eureka集群防止出现单点故障，单机eureka宕机使得整个系统瘫痪，搭建集群实现负载均衡+服务容错

##### 2.集群搭建

###### 1.eureka集群搭建

（1）根据eureka7001新建eureka7002模块

（2）更改etc/hosts 文件

```yml
#配置eureka
127.0.0.1       eureka7001.com
127.0.0.1       eureka7002.com
```

（3）更改yml

​	7001:

```yml
server:
  port: 7001
eureka:			#互相注册，相互守望
  instance:
    hostname: eureka7001.com	#更改主机名
  client:
    register-with-eureka: false #不注册自己
    fetch-registry: false 
    service-url:  #设置eureka server交互地址
      defaultZone: http://eureka7002.com:7002/eureka/
```

​	7002:

```yml
server:
  port: 7002
eureka:
  instance:
    hostname: eureka7002.com
  client:
    register-with-eureka: false #不注册自己
    fetch-registry: false 
    service-url:  #设置eureka server交互地址
      defaultZone: http://eureka7001.com:7001/eureka/
```

​	payment8001 && order80 ：

```yml
eureka:
  client:
    register-with-eureka: true #注册自己
    fetchRegistry: true 
    service-url: 
    	#注册进集群
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka

```

（4）测试：

​	先启动两个eureka，在启动payment和order，访问测试

###### 2.payment集群搭建

（1）根据payment8001新建payment8002模块

（2）更改yml

```yml
server:
  port: 8002

spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:mysql:///springcloud?useUnicode=true&characterEncoding=utf8
    username: root
    password: root

mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: study.springcloud.domain    

eureka:
  client:
    register-with-eureka: true #注册自己
    fetchRegistry: true #
    service-url: 
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
  instance:
    instance-id: payment8002  #修改eureka微服务名称
    prefer-ip-address: true #访问路径可以显示IP地址
```

（3）复制业务类

（4）实现订单调用支付的负载均衡

​		更改order的controller

```java
public static final String URL="http://CLOUD-PAYMENT-SERVICE";
```

​		在config的方法上加注解@LoadBalanced

```java
@Configuration
public class ApplicationContextConfig {	
	// ribbon负载均衡
    @Bean
    @LoadBalanced//负载均衡，配置后可轮询访问payment集群
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

（5）测试

​	启动两个eureka，在启动两个payment和一个order，访问http://localhost/consumer/payment/get/1

### 3.服务发现

​	对于注册eureka里面的微服务,可以通过服务发现来获得该服务的信息

（1）修改cloud-provider-payment8001的Controller

```java
@GetMapping("/payment/discovery")
public Object discovery(){
  List<String> services = discoveryClient.getServices();
  for(String s:services){
    System.out.println("==============  s:"+s);
  }

  List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
  for(ServiceInstance instance:instances){
    System.out.println(instance.getServiceId()+" host:"+instance.getHost()+" port"+instance.getPort());
  }
  return this.discoveryClient;
}
```

（2）8001的启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient//新增
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class);
    }
}
```

（3）测试：http://localhost:8001/payment/discovery

### 4.Eureka自我保护

​	在某时刻 一个微服务不可用了,Eureka不会立刻清理,依旧会对该服务的信息进行保存，属于CAP里面的AP分支。

​	微服务第一次注册成功之后，每 30 秒会发送一次心跳将服务的实例信息注册到注册中心。通知 Eureka Server 该实例仍然存在。如果超过 90 秒没有发送更新，则服务器将从注册信息中将此服务移除。

​	Eureka Server。在运行期间，会统计心跳失败的比例在 15 分钟之内是否低于 85%，如果出现低于的情况（在单机调试的时候很容易满足，实际在生产环境上通常是由于网络不稳定导致）, Eureka Server 会将当前的实例注册信息保护起来，同时提示这个警告。保护模式主要用于一组客户端和 Eureka Server 之间存在网络分区场景下的保护。一旦进入保护模式，Eureka Server将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据（也就是不会注销任何微服务）

​	验证完自我保护机制开启后，并不会马上呈现到 web 上，而是默认需等待 5分钟（可以通过eureka. server.wait-time-in- ms-when-sync- empty 配置），即 5 分钟后你会看到下面的提示信息

![image-20200623215721142](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200623215721142.png)

出产默认,自我保护机制是开启的，可已通过以下配置禁止自我保护

```yml
eureka:
  instance:
    hostname: eureka7001.com
  server:
    enable-self-preservation: false #关掉自我保护机制
```

### 5.核心源码

#### 1.服务注册

##### 1.服务端接收注册

```java
//public abstract class AbstractInstanceRegistry implements InstanceRegistry
//线程安全的map，存放所有注册的实例
private final ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>> registry
            = new ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>>();
//注册方法
public void register(InstanceInfo registrant, int leaseDuration, boolean isReplication) {
  try {
    read.lock();
    Map<String, Lease<InstanceInfo>> gMap = registry.get(registrant.getAppName());
    REGISTER.increment(isReplication);
    if (gMap == null) {
      final ConcurrentHashMap<String, Lease<InstanceInfo>> gNewMap = new ConcurrentHashMap<String, Lease<InstanceInfo>>();
      gMap = registry.putIfAbsent(registrant.getAppName(), gNewMap);
      if (gMap == null) {
        gMap = gNewMap;
      }
    }
    Lease<InstanceInfo> existingLease = gMap.get(registrant.getId());
    // Retain the last dirty timestamp without overwriting it, if there is already a lease
    if (existingLease != null && (existingLease.getHolder() != null)) {
      Long existingLastDirtyTimestamp = existingLease.getHolder().getLastDirtyTimestamp();
      Long registrationLastDirtyTimestamp = registrant.getLastDirtyTimestamp();
      logger.debug("Existing lease found (existing={}, provided={}", existingLastDirtyTimestamp, registrationLastDirtyTimestamp);

      // this is a > instead of a >= because if the timestamps are equal, we still take the remote transmitted
      // InstanceInfo instead of the server local copy.
      if (existingLastDirtyTimestamp > registrationLastDirtyTimestamp) {
        logger.warn("There is an existing lease and the existing lease's dirty timestamp {} is greater" +
                    " than the one that is being registered {}", existingLastDirtyTimestamp, registrationLastDirtyTimestamp);
        logger.warn("Using the existing instanceInfo instead of the new instanceInfo as the registrant");
        registrant = existingLease.getHolder();
      }
    } else {
      // The lease does not exist and hence it is a new registration
      synchronized (lock) {
        if (this.expectedNumberOfClientsSendingRenews > 0) {
          // Since the client wants to register it, increase the number of clients sending renews
          this.expectedNumberOfClientsSendingRenews = this.expectedNumberOfClientsSendingRenews + 1;
          updateRenewsPerMinThreshold();
        }
      }
      logger.debug("No previous lease information found; it is new registration");
    }
    Lease<InstanceInfo> lease = new Lease<InstanceInfo>(registrant, leaseDuration);
    if (existingLease != null) {
      lease.setServiceUpTimestamp(existingLease.getServiceUpTimestamp());
    }
    gMap.put(registrant.getId(), lease);
    synchronized (recentRegisteredQueue) {
      recentRegisteredQueue.add(new Pair<Long, String>(
        System.currentTimeMillis(),
        registrant.getAppName() + "(" + registrant.getId() + ")"));
    }
    // This is where the initial state transfer of overridden status happens
    if (!InstanceStatus.UNKNOWN.equals(registrant.getOverriddenStatus())) {
      logger.debug("Found overridden status {} for instance {}. Checking to see if needs to be add to the "
                   + "overrides", registrant.getOverriddenStatus(), registrant.getId());
      if (!overriddenInstanceStatusMap.containsKey(registrant.getId())) {
        logger.info("Not found overridden id {} and hence adding it", registrant.getId());
        overriddenInstanceStatusMap.put(registrant.getId(), registrant.getOverriddenStatus());
      }
    }
    InstanceStatus overriddenStatusFromMap = overriddenInstanceStatusMap.get(registrant.getId());
    if (overriddenStatusFromMap != null) {
      logger.info("Storing overridden status {} from map", overriddenStatusFromMap);
      registrant.setOverriddenStatus(overriddenStatusFromMap);
    }

    // Set the status based on the overridden status rules
    InstanceStatus overriddenInstanceStatus = getOverriddenInstanceStatus(registrant, existingLease, isReplication);
    registrant.setStatusWithoutDirty(overriddenInstanceStatus);

    // If the lease is registered with UP status, set lease service up timestamp
    if (InstanceStatus.UP.equals(registrant.getStatus())) {
      lease.serviceUp();
    }
    registrant.setActionType(ActionType.ADDED);
    recentlyChangedQueue.add(new RecentlyChangedItem(lease));
    registrant.setLastUpdatedTimestamp();
    invalidateCache(registrant.getAppName(), registrant.getVIPAddress(), registrant.getSecureVipAddress());
    logger.info("Registered instance {}/{} with status {} (replication={})",
                registrant.getAppName(), registrant.getId(), registrant.getStatus(), isReplication);
  } finally {
    read.unlock();
  }
}
```

##### 2.服务剔除

```java
////public abstract class AbstractInstanceRegistry implements InstanceRegistry
public void evict(long additionalLeaseMs) {
  logger.debug("Running the evict task");

  if (!isLeaseExpirationEnabled()) {
    logger.debug("DS: lease expiration is currently disabled.");
    return;
  }

  // We collect first all expired items, to evict them in random order. For large eviction sets,
  // if we do not that, we might wipe out whole apps before self preservation kicks in. By randomizing it,
  // the impact should be evenly distributed across all applications.
  List<Lease<InstanceInfo>> expiredLeases = new ArrayList<>();
  for (Entry<String, Map<String, Lease<InstanceInfo>>> groupEntry : registry.entrySet()) {
    Map<String, Lease<InstanceInfo>> leaseMap = groupEntry.getValue();
    if (leaseMap != null) {
      for (Entry<String, Lease<InstanceInfo>> leaseEntry : leaseMap.entrySet()) {
        Lease<InstanceInfo> lease = leaseEntry.getValue();
        if (lease.isExpired(additionalLeaseMs) && lease.getHolder() != null) {
          expiredLeases.add(lease);
        }
      }
    }
  }

  // To compensate for GC pauses or drifting local time, we need to use current registry size as a base for
  // triggering self-preservation. Without that we would wipe out full registry.
  int registrySize = (int) getLocalRegistrySize();
  int registrySizeThreshold = (int) (registrySize * serverConfig.getRenewalPercentThreshold());
  int evictionLimit = registrySize - registrySizeThreshold;

  int toEvict = Math.min(expiredLeases.size(), evictionLimit);
  if (toEvict > 0) {
    logger.info("Evicting {} items (expired={}, evictionLimit={})", toEvict, expiredLeases.size(), evictionLimit);

    Random random = new Random(System.currentTimeMillis());
    for (int i = 0; i < toEvict; i++) {
      // Pick a random item (Knuth shuffle algorithm)
      int next = i + random.nextInt(expiredLeases.size() - i);
      Collections.swap(expiredLeases, i, next);
      Lease<InstanceInfo> lease = expiredLeases.get(i);

      String appName = lease.getHolder().getAppName();
      String id = lease.getHolder().getId();
      EXPIRED.increment();
      logger.warn("DS: Registry: expired lease for {}/{}", appName, id);
      internalCancel(appName, id, false);
    }
  }
}

```

#### 2.服务发现

源码在DiscoverClient类中

##### 1.服务注册

```java
boolean register() throws Throwable {
  logger.info(PREFIX + "{}: registering service...", appPathIdentifier);
  EurekaHttpResponse<Void> httpResponse;
  try {
    httpResponse = eurekaTransport.registrationClient.register(instanceInfo);
  } catch (Exception e) {
    logger.warn(PREFIX + "{} - registration failed {}", appPathIdentifier, e.getMessage(), e);
    throw e;
  }
  if (logger.isInfoEnabled()) {
    logger.info(PREFIX + "{} - registration status: {}", appPathIdentifier, httpResponse.getStatusCode());
  }
  return httpResponse.getStatusCode() == Status.NO_CONTENT.getStatusCode();
}
```

##### 2.服务下架

```java
public synchronized void shutdown() {
  if (isShutdown.compareAndSet(false, true)) {
    logger.info("Shutting down DiscoveryClient ...");

    if (statusChangeListener != null && applicationInfoManager != null) {
      applicationInfoManager.unregisterStatusChangeListener(statusChangeListener.getId());
    }

    cancelScheduledTasks();

    // If APPINFO was registered
    if (applicationInfoManager != null
        && clientConfig.shouldRegisterWithEureka()
        && clientConfig.shouldUnregisterOnShutdown()) {
      applicationInfoManager.setInstanceStatus(InstanceStatus.DOWN);
      unregister();
    }

    if (eurekaTransport != null) {
      eurekaTransport.shutdown();
    }

    heartbeatStalenessMonitor.shutdown();
    registryStalenessMonitor.shutdown();

    logger.info("Completed shut down of DiscoveryClient");
  }
}
```

##### 3.心跳续约

```java
/**
 * Renew with the eureka service by making the appropriate REST call
*/
boolean renew() {
  EurekaHttpResponse<InstanceInfo> httpResponse;
  try {
    httpResponse = eurekaTransport.registrationClient.sendHeartBeat(instanceInfo.getAppName(), instanceInfo.getId(), instanceInfo, null);
    logger.debug(PREFIX + "{} - Heartbeat status: {}", appPathIdentifier, httpResponse.getStatusCode());
    if (httpResponse.getStatusCode() == Status.NOT_FOUND.getStatusCode()) {
      REREGISTER_COUNTER.increment();
      logger.info(PREFIX + "{} - Re-registering apps/{}", appPathIdentifier, instanceInfo.getAppName());
      long timestamp = instanceInfo.setIsDirtyWithTime();
      boolean success = register();
      if (success) {
        instanceInfo.unsetIsDirty(timestamp);
      }
      return success;
    }
    return httpResponse.getStatusCode() == Status.OK.getStatusCode();
  } catch (Throwable e) {
    logger.error(PREFIX + "{} - was unable to send heartbeat!", appPathIdentifier, e);
    return false;
  }
}
```

### 附：注册中心对比

| 组件      | 语言 | CAP   | 一致性算法 | 服务健康检查 | 对外暴露接口 |
| --------- | ---- | ----- | ---------- | ------------ | ------------ |
| Eureka    | Java | AP    | 无         | 可配         | HTTP         |
| Consul    | Go   | CP    | Raft       | 支持         | HTTP/DNS     |
| Zookeeper | Java | CP    | Paxos      | 支持         | 客户端       |
| Nacos     | Java | AP/CP | Raft       | 支持         | HTTP         |

## 四：服务调用Ribbon&Feign

### 1.Ribbon

#### 1.概述

##### 1.简介

​	Ribbon是 Netflixfa 发布的一个负载均衡器，有助于控制 HTTP 和 TCP 客户端行为。在Springcloud 中，Eureka一般配合 Ribbon 进行使用，Ribbon 提供了客户端负载均衡的功能，Ribbon 利用从 Eureka 中读取到的服务信息，在调用服务节点提供的服务时，会合理的进行负载。

​	在 Spring Cloude 中可以将注册中心和 Ribbon 配合使用，Ribbon 自动的从注册中心中获取服务提供者的列表信息，并基于内置的负载均衡算法，请求服务

##### 2.作用

###### 1.服务调用

​	基于Ribbon实现服务调用，是通过拉取到的所有服务列表组成（服务名-请求路径）映射关系。借助RestTemplate进行调用

###### 2.负载均衡

​	当多个服务提供者时，Ribbon可以根据负载均衡的算法自动选择需要调用的服务地址

（1）服务端负載均衡

​	先发送请求到负载均衡服务器或者软件，然后通过负載均衡算法，在**多个服务器之间选择一个**进行访问；即在服务器端再进行负载均衡算法分配（nginx）

（2）客户端负载均衡

​	客户端会有一个服务器地址列表，在发送请求前通过负载均衡算法选择一个服务器，然后进行访问，这是客户端负载均衡；即在客户端就进行负载均衡算法分配

![image-20200623224544201](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200623224544201.png)

#### 2.Ribbon组件	

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200624105308617.png" alt="image-20200624105308617" style="zoom:30%;" />

- ServerList：可以响应客户端的特定服务的服务器列表
- ServerListFilter：可以动态获得的具有所需特征的候选胡无趣列表的过滤器
- ServerListUpdater：可以执行动态服务器端列表更新
- Rule：负载均衡策略，用于确定从服务器列表返回的哪个服务器
- Ping：客户端用于快速检查服务器当时是否处于活动状态
- LoadBalancer：负载均衡器，负责负载均衡调度的管理

##### 1.Ribbon负载均衡

###### 1.支持的负载策略

| 负载均衡策略                            | 解释                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| com.netflix.loadbalancer.RoundRobinRule | 轮询                                                         |
| com.netflix.loadbalancer.RandomRule     | 随机                                                         |
| com.netflix.loadbalancer.RetryRule      | 先按照RoundRobinRule的策略获取服务,如果获取服务失败则在指定时间内进行重试,获取可用的服务 |
| WeightedResponseTimeRule                | 对RoundRobinRule的扩展,响应速度越快的实例选择权重越多大,越容易被选择 |
| BestAvailableRule                       | 会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务,然后选择一个并发量最小的服务 |
| AvailabilityFilteringRule               | 先过滤掉故障实例,再选择并发较小的实例                        |
| ZoneAvoidanceRule                       | 默认规则,复合判断server所在区域的性能和server的可用性选择服务器 |

###### 2.负载策略替换

​	官方文档指出，自定义配置负载的配置类不能放在`@Componentscan`所描的当前包下以及子包下,否则我们自定义的这个配置类就会被所有的 Ribbon 客户端所共享，达不到特殊化定制的目的了。

​	`@SpringBootApplication` 中含有 `@Componentscan`注解，所以在替换时，要跳出主启动类所在包。

（1）创建包myrule，在创建配置类

![image-20200624104126311](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200624104126311.png)

（2）主启动类中添加`@RibbonClient(name="CLOUD-PAYMENT-SERVICE",configuration = LRRule.class)`

（3）测试 http://localhost/consumer/payment/get/1

##### 2.自定义负载均衡

###### 1.Ribbon轮询负载源码

```java
public class RoundRobinRule extends AbstractLoadBalancerRule {

  private AtomicInteger nextServerCyclicCounter;
  private static final boolean AVAILABLE_ONLY_SERVERS = true;
  private static final boolean ALL_SERVERS = false;

  private static Logger log = LoggerFactory.getLogger(RoundRobinRule.class);

  public RoundRobinRule() {
    nextServerCyclicCounter = new AtomicInteger(0);
  }

  public RoundRobinRule(ILoadBalancer lb) {
    this();
    setLoadBalancer(lb);
  }

  public Server choose(ILoadBalancer lb, Object key) {
    if (lb == null) {
      log.warn("no load balancer");
      return null;
    }

    Server server = null;
    int count = 0;
    while (server == null && count++ < 10) {
      List<Server> reachableServers = lb.getReachableServers();
      List<Server> allServers = lb.getAllServers();
      int upCount = reachableServers.size();
      int serverCount = allServers.size();

      if ((upCount == 0) || (serverCount == 0)) {
        log.warn("No up servers available from load balancer: " + lb);
        return null;
      }

      int nextServerIndex = incrementAndGetModulo(serverCount);
      server = allServers.get(nextServerIndex);

      if (server == null) {
        /* Transient. */
        Thread.yield();
        continue;
      }

      if (server.isAlive() && (server.isReadyToServe())) {
        return (server);
      }

      // Next.
      server = null;
    }

    if (count >= 10) {
      log.warn("No available alive servers after 10 tries from load balancer: "
               + lb);
    }
    return server;
  }

  /**
     * Inspired by the implementation of {@link AtomicInteger#incrementAndGet()}.
     *
     * @param modulo The modulo to bound the value of the counter.
     * @return The next value.
     */
  private int incrementAndGetModulo(int modulo) {
    for (;;) {
      int current = nextServerCyclicCounter.get();
      int next = (current + 1) % modulo;
      if (nextServerCyclicCounter.compareAndSet(current, next))
        return next;
    }
  }

  @Override
  public Server choose(Object key) {
    return choose(getLoadBalancer(), key);
  }

  @Override
  public void initWithNiwsConfig(IClientConfig clientConfig) {
  }
}
```

###### 2.手写负载均衡算法

```java
import org.springframework.cloud.client.ServiceInstance;
import java.util.List;
public interface LoadBalance {
  //获得所有提供服务的微服务
  ServiceInstance instance(List<ServiceInstance> serviceInstances);
}


//实现类
@Component
public class MyLB implements LoadBalance{

  private AtomicInteger atomicInteger=new AtomicInteger(0);

  public final int getAndIncrement(){
    int current;
    int next;
    for(;;){
      current=this.atomicInteger.get();
      next=current>Integer.MAX_VALUE?0:current+1;
      if(atomicInteger.compareAndSet(current,next)){
        System.out.println("==========当前值为"+next);
        return next;
      }
    }
  }

  @Override
  public ServiceInstance instance(List<ServiceInstance> serviceInstances) {
    int index=getAndIncrement() % serviceInstances.size();

    return serviceInstances.get(index);
  }
}
```

### 2.OpenFeign

#### 1.概述

##### 1.简介

​	Feign是一个声明式的Web服务客户端,让编写Web服务客户端变得非常容易,只需 **创建一个接口并在接口上添加注解即可，在服务消费端使用**

官网：https://github.com/spring-cloud/spring-cloud-openfeign

​	Feign 是个声明式 Webservice 客户端。使用 Feign 能让编写 Web Service？客户端更加简单。它的使用方法是定义一个服务接口然后在上面添加注解。Feign 也支持可拔插式的编码器和解码器。Spring Cloud 对 Feign 进行了封装，使其支持了 Spring MVCT 标准注解和 Httpmessage Converters。Feign 可以与 Eureka 和 Ribbon 组合使用以支持负载均衡	

##### 2.Feign 能干什么

（1）Feign 旨在使编写 Java Http 客户端变得更容易

​	前面在使用 Ribbon+ RestTemplate时，利用 RestTemplate 对 http 请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用。所以，Feign 在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。在 Feign 的实现下我们只需创建一个接口并使用注解的方式来配置它（以前是 Da 接口上面标注 Mapper 注解现在是一个微服务接口上面标注 eign 注解即可），即可完成对服务提供方的接口绑定，简化了使用 Spring cloud Ribbon 时，自动封装服务调用客户端的开发量。

（2）Feign 集成了 Ribbon

​	利用 Ribbon 维护了 Paymente 的服务列表信息，并且通过轮询实现了客户端的负载均衡。而与 Ribbon，不同的是，通过 feign 只需要定义服务绑定接口且以声明式的方法，优雅而筒单的实现了服务调用

#### 2.项目构建

（1）新建Model：cloud-consumer-feign-order80

（2）改pom

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
  </dependency>
  <dependency>
    <groupId>study</groupId>
    <artifactId>cloud-api-commons</artifactId>
    <version>1.0-SNAPSHOT</version>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>

```

（3）写yml

```yml
server:
  port: 80

eureka:
  client:
    register-with-eureka: false #不注册自己
    service-url:  #设置eureka server交互地址
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```

（4）主启动

```java
@SpringBootApplication
@EnableFeignClients				//添加openFeign的注解
public class OrderFeignMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderFeignMain80.class);
    }
}

```

（5）业务类

​	controller

```java
@RestController
public class OrderFeignController {

  @Resource
  private PaymentFeignService pfs;

  @GetMapping("/consumer/payment/get/{id}")
  public CommonResult<Payment> getPaymentById(@PathVariable("id")Long id){
    return pfs.getPaymentById(id);
  }

}
```

​	service

```java
@Component
@FeignClient("CLOUD-PAYMENT-SERVICE")//指定微服务
public interface PaymentFeignService {
  
  @GetMapping("/payment/get/{id}")
  CommonResult<Payment> getPaymentById(@PathVariable("id")Long id);

}
```

（6）测试：http://localhost/consumer/payment/get/1

​			【发现】OpenFeign自带负载均衡配置项

#### 3.超时控制

​	OpenFeign默认等待1秒钟,当请求payment服务时若超过1s会报错,

![image-20200624175139494](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200624175139494.png)

可以更改yml中配置，使其特定业务时可以等待长时间访问

```yml
#feign的超时控制
ribbon:
  #建立连接所用的时间
  ReadTimeOut: 5000
  #从建立连接后服务端读取可用资源所用的时间
  ConnectTimeout: 5000


```

#### 4.日志级别

​	在开发或者运行阶段往往希望看到 Fegn 请求过程的日志记录，默认情況下 Feign 的日志是没有开启的。`feign. client.config.shop-service-product.loggerlevel`：配置 Feign 的日志 Feign 有四种日志级别：

- NONE【性能最佳，适用于生产】：不记录任何日志（默认值

-  BASIC【适用于生产环境追踪问题】：仅记录请求方法、URL、响应状态代码以及执行时间
-   HEADERS：记录 BASIC 级别的基础上，记录请求和响应的 header。 

- FULL【比较适用于开发及测试环境定位问题】：记录请求和响应的 header、body 和元数据



配置步骤：

（1）写yml

```yml
logging:
  level:
  	#Feign 日志只会对日志级别为 debug 的做出响应
    study.springcloud.service.PaymentFeginService: debug
```

 （2）配置bean

```java
@Configuration
public class FeignConfig {
  @Bean
  Logger.Level feignLoggerLever(){
    return Logger.Level.FULL;
  }
}
```

## 五：服务熔断Hystrix

### 1.概述

#### 1.分布式系统面临的问题

###### 服务雪崩

​	在微服务架构中，一个请求需要调用多个服务是非常常见的。如客户端访问 A 服务，而 A 服务需要调用 B 服务，B 服务需要调用服务，由于网络原因或者自身的原因，如果 B 服务或者 C 服务不能及时响应，A 服务将处于阻塞状态，直到 B 服务 C 服务响应。此时若有大量的请求涌入，容器的线程资源会被消耗完毕，导致服务瘫痪。服务与服务之间的依赖性，故障会传播，造成连锁反应，会对整个微服务系统造成灾难性的严重后果，这就是服务故障的'雪崩效应。

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200624190252399.png" alt="image-20200624190252399" style="zoom:50%;" />

​	简单说就是一个微服务不可用后导致**级联故障**，最后使得系统瘫痪

#### 2.Hytrix简介

​	Hystrix 是个用于**处理分布式系统的延迟和容错**的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix 能够保证在一个依赖出问题的情況下，不会导致整体服务失败，避兔级联故障，以提高分布式系统的弹性。

​	官网：https://github.com/Netflix/hystrix/wiki

#### 3.Hytrix作用

##### 1.服务降级

​	所谓降级，就是当某个服务熔断之后，服务器将不再被调用，此时客户端可以自己准备一个本地的 fallback 回调，返回一个缺省值。也可以理解为兜底

​	给最差情况的一个解决方案。

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200624192218205.png" alt="image-20200624192218205" style="zoom:50%;" />

触发服务降级的情况

- 程序运行异常
- 超时
- 服务熔断触发服务降级
- 线程池/信号量也会导致服务降级

##### 2.服务熔断

​	断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。类比保险丝达到最大服务访问后,直接拒绝访问,拉闸限电,然后调用服务降级的方法并返回友好提示

###### 熔断器的状态

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200624192133273.png" alt="image-20200624192133273" style="zoom:33%;" />

​	熔断器有三个状态CLOSED、OPEN、HALF_OPEN， 熔断器默认关闭状态，当触发熔断后状态变更为 OPEN，在等待到指定的时间，Hystrix 会放请求检测服务是否开启，这期间熔断器会变为 HALF_OPEN 半开启状态，熔断探测服务可用则继续变更为 CLOSED 关闭熔断器。

- Closed：关闭状态（断路器关闭），所有请求都正常访问。代理类维护了最近调用失败的次数，如果某次调用失败，则使失败次数加 1。如果最近失败次数超过了在给定时间内允许失败的阈值则代理类切換到断开（Open）状态。此时代理开启了一个超时时钟，当该时钟超过了该时间，则切换到半断开（Haf-Open）状态。该超时时间的设定是给了系统一次机会来修正导致调用失败的错误

- Open：打开状态（断路器打开），所有请求都会被降级。Hystix 会对请求情況计数，当一定时间内失败请求百分比达到阈值，则触发熔断，断路器会完全关闭。默认失败比例的阈值是 50%，请求次数最少不低于 20 次。
-  Half Open：半开状态，open 状态不是永久的，打开后会进入休眠时间（默认是 55)。随后断路器会自动进入半开状态。此时会释放 1 次请求通过，若这个请求是健康的，则会关闭断路器，否则继续保持打开，再次进行 5 秒休眠计时。



​	熔断机制是应对雪崩效应的一种微服务链路保护机制。当扇出链路的某个微服务出错不可用或者响应时间太长时会进行服务的降级，进而断该节点微服务的调用，快速回错误的响应信息，当检测到该节点微服务调用响应正常后，恢复调用链路

##### 3.服务限流

​	限流可以认为服务降级的一种，限流就是限制系统的输入和输出流量已达到保护系统的目的。一般来说系统的吞吐量是可以被测算的，为了保证系统的稳固运行，一旦达到的需要限制的阈值，就需要限制流量并采取少量措施以完成限制流量的目的。比方：推退解決，拒绝解决，或者者部分拒绝解決等等。

​	比如秒杀高并发等操作,严禁一窝蜂的过来拥挤,大家排队,一秒钟N个,有序进行

### 2.项目构建

#### 1.hystrix-payment

（1）新建cloud-provider-hystrix-payment8001

（2）改pom

```xml
<!--引入以下新增依赖-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
<!--eureka client-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

（3）写yml

```yml
server:
  port: 8001
spring:
  application:
    name: cloud-provider-hystrix-payment
eureka:
  client:
    register-with-eureka: true #注册自己
    fetchRegistry: true #
    service-url: 
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```

（4）主启动

```java
@SpringBootApplication
@EnableEurekaClient
public class PaymentHystrix8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentHystrix8001.class);
    }
}
```

（5）业务类

​	controller

```java
@RestController
public class PaymentController {
  @Autowired
  private PaymentService paymentService;

  @Value("${server.port}")
  private String serverPort;

  @GetMapping("/payment/hystrix/ok/{id}")
  public String paymentInfo_OK(@PathVariable("id") Integer id)
  {
    String result = paymentService.paymentInfo_OK(id);
    System.out.println("*****result: "+result);
    return result;
  }

  @GetMapping("/payment/hystrix/timeout/{id}")
  public String paymentInfo_TimeOut(@PathVariable("id") Integer id)
  {
    String result = paymentService.paymentInfo_TimeOut(id);
    System.out.println("*****result: "+result);

    return result;
  }

}
```

​	Service

```java
@Service
public class PaymentService {
  public String paymentInfo_OK(Integer id){
    return "线程池:  "+Thread.currentThread().getName()+"paymentInfo_OK,id:  "+id;
  }
  public String paymentInfo_TimeOut(Integer id) {
    try {TimeUnit.SECONDS.sleep(5);}catch(InterruptedException e){ e.printStackTrace();}
    return "线程池:  "+Thread.currentThread().getName()+" id:  "+id;
  }
}
```

（6）测试

​	http://localhost:8001/payment/hystrix/ok/31

​	http://localhost:8001/payment/hystrix/timeout/31

​	以上项目没有问题，但是在jmeter测试中，2000个线程访问timeout，使得ok也不能正常访问。因为8001同一层次的其他接口被困死,因为tomcat线程池里面的工作线程已经被挤占完毕，所以导致了客户端访问响应缓慢,转圈圈

#### 2.hystrix-order

（1）建Model	cloud-consumer-feign-hystrix-order80

（2）改pom

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<!--hystrix-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
<!--eureka client-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

（3）写yml

```yml
server:
  port: 80

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```

（4）主启动

```java
@SpringBootApplication
@EnableFeignClients
public class OrderHystrixMain80 {
  public static void main(String[] args){
    SpringApplication.run(OrderHystrixMain80.class,args);
  }
}

```

（5）业务类

​	controller

```java
@RestController
//配置全局的降级方法
@DefaultProperties(defaultFallback = "global_fallback")
public class OrderHystrixController {

  @Resource
  private PaymentHystrixService ps;

  @GetMapping("/consumer/payment/hystrix/ok/{id}")
  public String paymentInfo_OK(@PathVariable("id") Integer id)
  {
    String result = ps.paymentInfo_OK(id);
    return result;
  }

  @GetMapping("/consumer/payment/hystrix/timeout/{id}")
  public String paymentInfo_Timeout(@PathVariable("id") Integer id) {
    String result = ps.paymentInfo_TimeOut(id);
    return result;
  }
  public String paymentInfo_TimeOutHandler(@PathVariable("id") Integer id){
    return "order触发服务降级";
  }
}
```

​	service

```java
@Component
@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT",fallback = PaymentFallbackService.class)
public interface PaymentHystrixService {
    @GetMapping("/payment/hystrix/ok/{id}")
     String paymentInfo_OK(@PathVariable("id") Integer id);

    @GetMapping("/payment/hystrix/timeout/{id}")
     String paymentInfo_TimeOut(@PathVariable("id") Integer id);
}
```

（6）测试

​	http://localhost/consumer/payment/hystrix/ok/1

#### 3.配置服务降级

##### 1.服务提供端

业务类启用@HystrixCommand

```java
@HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler")
public String paymentInfo_TimeOut(Integer id) {
 
  try {TimeUnit.SECONDS.sleep(5);}catch(InterruptedException e){ e.printStackTrace();}
  return "线程池:  "+Thread.currentThread().getName()+" id:  "+id;
}
//降级的兜底方法
public String paymentInfo_TimeOutHandler(Integer id) {
  return "线程池:  "+Thread.currentThread().getName()+" id:  "+id;
}
```

主启动中启用@EnableCircuitBreaker

```java
@EnableCircuitBreaker
public class PaymentHystrix8001 {
  public static void main(String[] args) {
    SpringApplication.run(PaymentHystrix8001.class);
  }
}
```

##### 2.服务消费端

改yml

```yml
#开启hystrix的支持
feign:
  hystrix:
    enabled: true
```

主启动

```java
@EnableHystrix
```

业务类controller

```java
@GetMapping("/consumer/payment/hystrix/timeout/{id}")
@HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler",commandProperties = {
  @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "1500")
})
public String paymentInfo_Timeout(@PathVariable("id") Integer id) {
  //当本身服务出错时也可降级
  int a=10/0;
  String result = ps.paymentInfo_TimeOut(id);
  return result;
}

public String paymentInfo_TimeOutHandler(@PathVariable("id") Integer id) {
  return "order触发服务降级";
}
```

##### 3.目前问题及解决

问题：

- 每个业务方法对应一个兜底的方法,代码膨胀
- 兜底的方法和业务逻辑混在一起

解决

（1）代码膨胀

​	使用全局的降级方法@DefaultProperties

```java
@DefaultProperties(defaultFallback = "global_fallback")
public class OrderHystrixController {
  @HystrixCommand //使用全局的服务降级方法
  public String paymentInfo_Timeout(@PathVariable("id") Integer id) {
    //出错后也可降级
    int a=10/0;
    String result = ps.paymentInfo_TimeOut(id);
    return result;
  }
  public String global_fallback(){
    return "使用全局服务降级的处理方法";
  }
}
```

（2）业务混乱

​	根据cloud-consumer-feign-hystrix-order80已经有的PaymentHystrixService接口,重新新建一个类(PaymentFallbackService)实现接口,统一为接口里面的方法进行异常处理·

```java
@Component
//实现服务降级,为了使业务逻辑和服务降级的方法解耦
public class PaymentFallbackService implements PaymentHystrixService{
  @Override
  public String paymentInfo_OK(Integer id) {
    return "PaymentOK fallback";
  }

  @Override
  public String paymentInfo_TimeOut(Integer id) {
    return "PaymentTimeout fallback";
  }
}
```



#### 4.配置服务熔断

##### 1.服务提供端

​	service

```java
//服务熔断
@HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback",commandProperties = {
  //开启断路器后，在时间窗口期的时间内，发生了10次请求，其中失败率达到了60%，就跳闸。以上参数在HystrixCommandProperties
  @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),// 是否开启断路器
  @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "5"),// 请求次数
  @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "5000"), // 时间窗口期
  @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "10"),// 失败率达到多少后跳闸
})
public String paymentCircuitBreaker(@PathVariable("id")Integer id){
  if(id<0){
    throw new RuntimeException("id<0");
  }
  String str= IdUtil.simpleUUID();
  return Thread.currentThread().getName()+"\t"+"调用成功"+"id=" +str;
}
```

​	controller

```java
@GetMapping("/payment/circuit/{id}")
public String paymentCircuitBreaker(@PathVariable("id") Integer id) {
  String result = paymentService.paymentCircuitBreaker(id);
  System.out.println("*****result: "+result);

  return result;
}
```

##### 2.服务熔断流程图

![image-20200624232514719](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200624232514719.png)

Hystrix的工作流程：
1.每次调用都会创建一个HystrixCommand
2.执行execute或queue做同步\异步调用
3.判断熔断器是否打开,如果打开跳到步骤8，否则进入步骤4
4.判断线程池/信号量是否跑满，如果跑满进入步骤8,否则进入步骤5
5.调用HystrixCommand的run方法，如果调用超时进入步骤8
6.判断是否调用成功，返回成功调用结果，如果失败进入步骤8
7.计算熔断器状态,所有的运行状态(成功, 失败, 拒绝,超时)上报给熔断器，用于统计从而判断熔断器状态
8.降级处理逻辑，根据上方的步骤可以得出以下四种情况会进入降级处理：

1. 熔断器打开
2. 线程池/信号量跑满
3. 调用超时
4. 调用失败

9.返回执行成功结果

#### 5.Hystrix DashBoard

##### 1.介绍

​	除了隔离依赖服务的调用以外，Hystrix 还提供了准实时的调用监控（Hystrix Dashboard), Hystrix 会持续记录所有通过 Hystrix 发起的请求的执行信息，并以统计报表和图形的形式展示给用户，包括每秒执行多少请求多少成功，多少失败等。Neti 通过 hystrix-metrics- event- stream 项目实现了对以上指标的监控。Spring Cloud 也提供了 Hystrix Dashboardl 的整合，对监控内容转化成可视化界面。

##### 2.项目搭建

（1）建Model： 新建cloud-consumer-hystrix-dashboard9001

（2）改pom

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

（3）写yml

```yml
server:
  port: 9001
```

（4）主启动

```java
@SpringBootApplication
@EnableHystrixDashboard
public class DashboardMain9001 {
  public static void main(String[] args) {
    SpringApplication.run(DashboardMain9001.class);
  }
}
```

（5）测试：http://localhost:9001/hystrix

（6）9001监控payment8001

![image-20200624233635569](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200624233635569.png)

​	在以上填入地址：http://localhost:8001/hystrix.stream

​	然后测试：http://localhost:8001/payment/circuit/1

​					http://localhost:8001/payment/circuit/-1

​	观察dashboard中的图形化界面以观测微服务的健康状况

## 六：服务网关GateWay

### 1.概述

#### 1.网关简介

##### 1.微服务网关介绍

​	因为不同的微服务一般有不同的网络地址，客户端要访问同一系统下不同微服务时必须记住几十个上百个地址，对于客服端过于复杂。如果让客户端直接与微服务通讯，可能出现以下问题：

- 客户端会请求多个不同的服务，需要维护不同的请求地址，增加开发难度
- 在某些场景下存在跨域请求的问题
- 加大身份认证的难度，每个微服务需要独立认证



​	因此，我们需要一个微服务网关，介于客户端与服务器之间的中间层，所有的外部请求都会先经过微服务网关。客户端只需要与网关交互，只知道一个网关地址即可，这样简化了开发还有以下优点： 

- 易于监控，可以在网关收集监控数据并将其推送到外部系统进行分析

- 易于认证，可以在网关上进行认证，然后再将请求转发到后端的微服务，而无须在每个微服务中进行认证

- 減少了客户端与各个微服务之间的交互次数

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200625112927750.png" alt="image-20200625112927750" style="zoom:50%;" />

​	由上面可以看出，微服务网关是一个服务器，是系统对外的唯一入口。API 网关封装了系统内部架构，为每个客户端提供个定制的 API。API 网关方式的核心要点是，所有的客户端和消费端都通过统一的网关接入微服务, 在 网关层处理所有的非业务功能。通常,网关也是提供REST/HTP的访可API。服务端通过API-GW 注册和管理服务。

##### 2.GetaWay介绍

​	Gateway 是在 Spring 生态系统之上构建的 API 网关服务，基于 Spring5, Spring Boot2 和 Project Reactor 等技术。

​	Gateway 旨在提供一种简单而有效的方式来对 AP 进行路由，以圾提供一些强大的过滤器功能，例如：熔断、限流、重试等

​	Gateway使用的是Webflux中的**reactor-netty响应式编程组件,底层使用了Netty通讯框架**

官网：https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/

Webflux：https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#spring-webflux（通常是非阻塞服务器，而MVC在Servlet应用程序阻塞式IO）

##### 3.微服务架构

![image-20200625140959232](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200625140959232.png)

GateWay作用

- 反向代理
- 统一鉴权

- 流量控制

- 熔断
- 日志监控

##### 4.GateWay和Zuul1.x的区别

##### 1.Zuul 1.x

 	Zuul1.X 是基于 servlet上的一个阻塞式处理模型，即 Spring实现了处理所有 request 请求的个 servlet (Dispatcherservlet）并由该 servlet **阻塞式处理请求**。但是Servlet 是个简单的网络模型，当请求进入 servlet container 时，servlet containers 就会为其绑定一个线程，在并发不高的场景下这种模型是适用的。但是一旦高并发，线程数量就会上张，而线程资源代价是昂贵的（上线文切换，内存消耗大）严重影响请求的处理时间在一些简单业务场景下，不希望为每个 request 分配个线程，只需要 1 个或几个线程就能应对极大并发的请求，这种业务场景下 servlet 模型没有优势

![image-20200625143113345](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200625143113345.png)

##### 2.GateWay

​	Gateway使用的是Webflux中的**reactor-netty响应式编程组件,底层使用了Netty通讯框架**而Webflux支持 Servlet3.1，Servlet3.1 之后有了异步非阳塞的支持。因此Webflux 是个典型非阻塞异步的框架，它的核心是基于 Reactor 的相关 API 实现的。相对于传统的 web 框架来说，它可以运行在诸如 Netty, Undertow 及支持 Servlet3.1 的容器上非阻塞式+函数式编程

​	Spring Webflux 是 Spring5.0 引入的新的响应式框架，区別于 Spring MVC，它不需要依赖 Servlet Apl，它是完全异步非阻塞的，并且基于 Reactor 来实现响应式流规范。

#### 2.GateWay基础

##### 1.核心概念

- Route(路由)：路由是构建网关的基本模块,它由ID,目标URI,一系列的断言和过滤器组成,如断言为true则匹配该路由
- Predicate(断言)：参考的是Java8的java.util.function.Predicate，开发人员可以匹配HTTP请求中的所有内容(例如请求头或请求参数),如果请求与断言相匹配则进行路由
- Filter(过滤)：	指的是Spring框架中GatewayFilter的实例,使用过滤器,可以在请求被路由前或者之后对请求进行修改.

![image-20200625144349315](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200625144349315.png)

​	Web 请求，通过匹配条件，定位到真正的服务节点。并在这个转发过程的前后，进行一些精细化控制。predicate 就是我们的匹配条件；而 filter，就可以理解为一个无所不能的拦截器。有了这两个元素，再加上目标 uri，就可以实现一个具体的路由了

##### 2.请求流程

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200625145030511.png" alt="image-20200625145030511" style="zoom:33%;" />

​	客户端向 Spring Cloud Gateway 发出请求。然后在 Gateway **Handler Mapping【断言匹配路由**】 中找到与请求相匹配的路由，将其发送到 Gateway **Web Handler【创建过滤器链，调用过滤器**】。Handler 再通过指定的过滤器链来将请求发送到我们门实际的服务执行业务逻辑，然后返回。过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（"pre“）或之后（"post“）执行业务逻辑。
​	

​	核心逻辑：**路由转发+执行过滤器链**

### 2.项目搭建

#### 1.gateway9527

（1）建Model：cloud-gateway-gateway9527

（2）改pom

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<!--eureka-client-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

（3）写yml

```yml
server:
  port: 9527
spring:
  application:
    name: cloud-gateway

eureka:
  client:
    service-url:  #设置eureka server交互地址
      register-with-eureka: true #注册自己
      fetch-registry: true #
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
  instance:
    hostname: cloud-gateway-service
```

（4）主启动

```java
@SpringBootApplication
@EnableEurekaClient
public class GatewayMain9527 {
  public static void main(String[] args) {
    SpringApplication.run(GatewayMain9527.class);
  }
}
```

（5）路由映射

​	在payment8001上套一层9527，更改yml

```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
```

（6）测试：http://localhost:9527/payment/get/1

##### 路由配置方式

​	Gateway有两种路由配置方式，一是配置yml，而是代码配置

```java
//config包下
@Configuration
public class GatewayConfig {

  //配置路由规则
  @Bean
  public RouteLocator routes(RouteLocatorBuilder builder){
    RouteLocatorBuilder.Builder routes = builder.routes();
    //path：9527访问的路径    uri:映射其他服务的网址
    routes.route("path_route",
                 r->r.path("/guonei").uri("http://news.baidu.com/guonei")).build();
    return routes.build();
  }

}

```

#### 2.动态路由

​	默认情况下Gatway会根据注册中心注册的服务列表, 以注册中心上微服务名为路径创建动态路由进行转发,从而实现动态路由的功能

（1）改yml【需要注意的是uri的协议lb,表示启用Gateway的负载均衡功能】

```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
#          uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
#          uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
```

（2）测试：http://localhost:9527/payment/lb

#### 3.重写转发路径

​	在 Springcloud Gateway 中，路由转发是直接将匹配的路由 path 直接拼接到映射路径uri之后，那么在微服务开发中往往没有那么便利。这里就可以通过 Rewritepath 机制来进行路径重写。

（1）更改path

```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh 
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment-server/get/**         # 断言，路径相匹配的进行路由
```

​	重新启动网关,访问http://localhost:9527/payment-server/get/1,会抛出404。这是由于路由转发规则默认转发到商品微服务（http: //localhost:8001/payment-service/ get/1)路径上,而支付微服务又没有 payment- service对应的映射配置。

（2）添加RewritePath转发路径

```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh 
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment-server/get/**         # 断言，路径相匹配的进行路由
          filter:
            - RewritePath=/payment-server/(?<segment>.*),/$\{segment}
```

​	通过 Rewritepathk 配置重写转发的 url，将/ product-service/ (? *），重写为 segment，然后转发到订单微服务。比如在网页上请求 http://localhost:9527/payment-server/get/1，此时会将请求转发到http://localhost:8001/payment/get/1,值得注意的是在ym1文档中S 要写成 SN)

#### 3.Predicate

​	Spring Cloud Gateway 将路由匹配作为 Spring Webflux Handler Mapping 基础架均的—部分。

​	SpringCloudGateway 包括许多内置的 RoutePredicate 厂。所有这些 Predicate 都与 HTTP 请求的不同属性匹配。多个 Route Predicate 工厂可以进行组合。Spring Cloud Gateway 创建 Route 对象时，使用 Routepredicatefactory 创建 Predicate 对象，Predicate 对象可以赋值给Route。Spring Cloud Gateway 包含许多内置的 Route Predicate Factories

​	所有这些谓词都匹配 HTTP 请求的不同属性。多种谓词工厂可以组合，并通过**逻辑 and**

##### 常用的Route Predicate

1.After Route Predicate 

2.Before Route Predicate 

3.Between Route Predicate 

4.Cookie Route Predicate 

5.Header Route Predicate 

6.Host Route Predicate

7.Method Route Predicate 

8.Path Route Predicate

9.Query Route Predicate 

10.RemoteAddr Route Predicate

11.Weight Route Predicate

```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
            - After=2020-02-21T15:51:37.485+08:00[Asia/Shanghai]
            - Cookie=username,ywb
            - Header=X-Request-Id, \d+  # 请求头要有X-Re
            - Method=GET
            - Query=username, \d+    #要有参数名且为整数才能访问
```

​	Predicate就是为了实现一组匹配规则, 让请求过来找到对应的Route进行处理

#### 4.Filter的使用

​	路由过滤器可用于修改进入的 HTTP 请求和返回的 HTTP 响应，路由过滤器只能指定路由进行使用 Spring Cloud Gateway 内置了多种路由过器，他们都由 Gateway Filterf 的工厂类来产生

##### 1.过滤器的生命周期

- PRE：这种过滤器在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择请求的微服务、记录调试信息等。

- POST：这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准的 HTP Header、收集统计信息和指标、将响应从微服务发送给客户端等。

##### 2.过滤器的种类

- Gateway Filter：应用到单个路由或者一个分组的路由上。https://cloud.spring.io/spring-cloud-gateway/2.2.x/reference/html/#gatewayfilter-factories

- Global Filter：应用到所有的路由上。https://cloud.spring.io/spring-cloud-gateway/2.2.x/reference/html/#global-filters

##### 3.自定义过滤器Globalfilter

```java
@Component
public class MyLogGatewayFilter implements GlobalFilter, Ordered {

  @Override
  public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
    System.out.println("come in MyLogGatewayFilter"+new Date());
    String uname = exchange.getRequest().getQueryParams().getFirst("uname");

    if(uname==null){
      System.out.println("用户名为null，非法用户禁止访问");
      exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
      return exchange.getResponse().setComplete();
    }
    return chain.filter(exchange);
  }

  //加载过滤器的顺序，数字越小优先级越高
  @Override
  public int getOrder() {
    return 0;
  }
}
```

#### 5.网关限流

##### 1.常见限流算法

###### 计数器

​	计数器限流算法是最筒单的一种限流实现方式。其本质是通过维护一个单位时间内的计数器，每次请求计数器加 1, 当单位时间内计数器累加到大于设定的阈值，则之后的请求都被拒绝，直到单位时间已经过去，再将计数器重置为零

###### 2.漏桶算法

​	漏桶算法可以很好地限制容量池的大小，从而防止流量暴增。漏桶可以看作是一个带有常量服务时间的单服务器队列，如果漏桶（包缓存）溢出，那么数据包会被丢弃。在网络中，漏桶算法可以控制端口的流量输出速率，平滑网络上的突发流量，实现流量整形，从而为网络提供一个稳定的流量。

###### 3.令牌桶算法

​	令牌桶算法是对漏桶算法的一种改进，桶算法能够限制请求调用的速率，而令牌桶算法能够在限制调用的平均速率的同时还允许一定程度的突发调用。在令牌桶算法中，存在一个桶，用来存放固定数量的令牌。算法中存在一种机制，以一定的速率往桶中放令牌。每次请求调用需要先获取令牌，只有拿到令牌，才有机会继续执行，否则选择选择等待可用的令牌、或者直接拒绝。放令牌这个动作是持续不断的进行，如果桶中令牌数达到上限，就丢弃令牌，所以就存在这种情況，桶中一直有大量的可用令牌，这时进来的请求就可以直接拿到令牌执行，比如设置 qps 为 100, 那么限流器初始化完成一秒后，桶中就已经有 100 个令牌了，这时服务还没完全启动好，等启动完成对外提供服务时，该限流器可以抵挡瞬时的 100 个请求。所以，只有桶中没有令牌时，请求オ会进行等待，最后相当于以一定的速率执行。

##### 2.配置限流

（1）java配置

```java
@Configuration
public class KeyResolverConfiguration {
  //基于请求路径限流
  @Bean
  public KeyResolver path(){
    return exchange -> Mono.just(
      exchange.getRequest().getPath().toString()
    );
  }
  //基于ip限流
  @Bean
  public KeyResolver ip(){
    return exchange -> Mono.just(
      exchange.getRequest().getHeaders().getFirst("X-Forwarded-For")
    );
  }
  //基于user限流
  @Bean
  public KeyResolver user(){
    return exchange -> Mono.just(
      exchange.getRequest().getQueryParams().getFirst("user")
    );
  }
}
```

（2）yml

```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh 
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment-server/get/**         # 断言，路径相匹配的进行路由
          filter:
            - RewritePath=/payment-server/(?<segment>.*),/$\{segment}
            - name: RequestRateLimiter
            	args:
            		key-resolver: '#{@path}'	#使用SPEL从容器获取对象
            		redis-rate-limiter.relenishRate: 1  #令牌桶每秒平均填充速率
            		redis-rate-limiter.burstCapacity: 3	#令牌桶总容量
  redis:
  	host: localhost
  	port: 6739
```

### 3.网关高可用

（1）复制一个model：gateway9528

（2）配置nginx

```shell
#配置多台服务器（这里只在一台服务器上的不同端口）
upstream gateway {
  server 127.0.0.1:9527;
  server 127.0.0.1:9528;
}

#请求转向 mysvr 定义的服务器列表
location /{
	proxy_pass http://gateway;
}
```

## 七：消息驱动Stream

### 1.概述

#### 1.介绍

​	在实际开发中，消息中间件是至关重要的组件之一。消息中间件主要解决**应用解耦，异步消息，流量削锋**等问题，实现高性能，高可用，可伸缩和最终一致性架构。不同的中间件其实现方式，内部结构是不一样的。如常见的 Rabbitmq 和 Kafka，由于这两个消息中间件的架构上的不同，像 Rabbitmq 有 exchange, kafka 有topic, partitions 分区，这些中间件的差异性导致我们实际项目开发给我们造成了一定的困扰，我们如果用了两个消息队列的其中一种，后面的业务需求，我想往另外一种消息队列进行迁移，这时候无疑就是一个灾难性的，一大堆东西都要重新推倒重新做，因为它跟我们的系统耦合了，这时候 springcloud Stream 给我们提供了一种解耦合的方式，**springcloud Stream 可以屏蔽底层消息中间件的差异，降低切换成本，统一消息的编程模型**

官网：https://spring.io/projects/spring-cloud-stream#overview

#### 2.架构

![image-20200625230649360](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200625230649360.png)

​	通过定义绑定器 Binder 作为中间层，实现了应用程序与消息中间件细节之间的隔离。Stream中的消息通讯方式遵循了发布-订阅模式，Topic主题进行广播

#### 3.核心概念

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200625231211612.png" alt="image-20200625231211612" style="zoom:50%;" />

##### 1.绑定器Binder

​	Binder 绑定器是 Spring Cloud Stream 中一个非常重要的概念。在没有绑定器这个概念的情况下，我们的 Spring Booth 应用要直接与消息中间件进行信息交互的时候，由于各消息中间件构建的初衷不同，它们的实现细节上会有较大的差异性，这使得我们实现的消息交互逻辑就会非常笨重，因为对具体的中间件实现细节有太重的依赖，当中间件有较大的变动升级、或是更換中间件的时候，我们就需要付出非常大的代价来实施。

​	通过定义绑定器作为中间层，实现了应用程序与消息中间件（Middleware）细节之间的隔离。通过向应用程序暴露统一的 Channel ，使得应用程序不需要再考虑各种不同的消息中间件的实现。当需要升级消息中间件，或者是更换其他消息中间件产品时，我们需要做的就是更换对应的 Bindery 绑定器而不需要修改任何应用逻辑。甚至可以任意的改变中间件的类型而不需要修改一行代码

##### 2.Channel

​	Channel：通道，是队列Queue的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过channel对队列进行配置

##### 3.Source和Sink

​	简单的可理解为参照对象是Springcloud Stream自身，从Stream发布消息就是输出，接受消息就是输入

##### 4.发布订阅模型

​	在 Spring Cloud Stream 中的消息通信方式遵循了发布订阅模式，当一条消息被投递到消息中间件之后，它会通过共享的 Topic 主题进行广播，消息消费者在订阅的主题中收到它并触发自身的业务逻辑处理。这里所提到的 Topic 主题是 Spring Cloud Stream 中的一个抽象概念，用来代表发布共享消息给消费者的地方。在不同的消息中间件中，Topic 可能对应着不同的概念，比如：在 Rabbit MQI 中的它对应了 Exchange、而在 Kaka 中则对应了 Kafka 中的 Topic

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200625230504209.png" alt="image-20200625230504209" style="zoom:50%;" />

#### 4.API和常用注解

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200625231547557.png" alt="image-20200625231547557" style="zoom:50%;" />

| 组成            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| Middleware      | 中间件，目前只支持 Rabbitmq 和 Kafka                         |
| Binder          | Binder 是应用与消息中间件之间的封装，目前实行了 Kaka 和 RabbitMQ 的 Binder，通过 Binders 可以很方便的连接中间件，可以动态的改变消息类型（对应于 Kafka 的 topic Rabbit 的 exchange），这些都可以通过配置文件来实现 |
| @input          | 注解标识入通道，通过该输入通道接收到的消息进入应用程序       |
| @Output         | 注解标识出通道，发布的消息将通过该通道离开应用程序           |
| @StreamListener | 监听队列，用于消者的队列的消息接收                           |
| @EnableBinder   | 指信道 Channe 和 exchange 绑定在一起                         |

### 2.项目搭建

先启动好rabbitMQ，然后准备三个项目

- cloud-stream-rabbitmq-provider8801，作为生产者进行消息模块
- cloud-stream-rabbitmq-consumer8802，作为消费者接受模块
- cloud-stream-rabbitmq-consumer8803，作为消费者接受模块

#### 1.provider8801

（1）建model

（2）改pom

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```

（3）写yml

```yml
server:
  port: 8801

spring:
  application:
    name: cloud-stream-provider
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        output: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息队列的具体实例

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: send-8801.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址

```

（4）主启动

```java
@SpringBootApplication
@EnableEurekaClient
public class StreamMqMain8801 {
  public static void main(String[] args) {
    SpringApplication.run(StreamMqMain8801.class);
  }
}
```

（5）业务类

​	controller

```java
@RestController
public class SendMessageController {

  @Autowired
  private IMessageProvide output;

  @GetMapping("/sendMessage")
  public String sendMessage(){
    return output.send();
  }

}

```

​	service

```java
public interface IMessageProvide {
  String send();
}

//生产方定义消息的推送管道
@EnableBinding(Source.class)
public class MessageProviderImpl implements IMessageProvide {

  @Autowired
  private MessageChannel output;//消息发送管道

  @Override
  public String send() {
    String serial= UUID.randomUUID().toString();
    output.send(MessageBuilder.withPayload(serial).build());
    System.out.println("serial="+serial);
    return serial;
  }
}
```

#### 2.consumer8802、8003

和8801基本相同，需要更改的配置

（1）yml：

```yml
spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
      	#=======================不同===================
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义，指定队列
          content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息队列的具体实例
eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: receive-8802.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址
```

（2）业务类

```java
@Component
@EnableBinding(Sink.class)
public class ReceiveMessage {

  @Value("${server.port}")
  private String serverPort;

  @StreamListener(Sink.INPUT)
  public void input(Message<String> message){
    System.out.println("消费者1号,----->接受到的消息: "+message.getPayload()+"\t  port: "+serverPort);
  }
}
```

### 3.消息的分组、持久化与分区与

#### 1.消息分组、持久化

​	通常在生产环境，我们的每个服务都不会以单节点的方式运行在生产环境，当同一个服务启动多个实例的时候，这些实例都会绑定到同个消息通道的目标主题（Topic）上。默认情况下，当生产者发出条消息到绑定通道上，这条消息会产生多个副本被每个消费者实例接收和处理【**可能会出现重复消费问题**】，但是有些业务场景之下，我们希望生产者产生的消息只被其中一个实例消费，这个时候我们需要为这些消费者设置消费组来实现这样的功能

​	直接在配置中添加属性即可实现消息分组

```yml
			bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息队列的具体实例
          group: ywb    #分组相同则不会重复消费，不同组可以重复消费
```

​	这样配置后8802/8803实现了轮询分组，每次只有一个消费者。8801模块的发的消息只能被8802或者8803其中一个接受到，这样避免了重复消费。

​	【**原理**】：微服务应用放置于同一个group中，就能够保证消息只会被其中一个应用消费一次。**不同的组是可以消费的，同一个组内会发生竞争关系，只有其中一个可以消费**。

​	同时，加入group属性也可使消息持久化，防止消息错过	

#### 2.消息分区

​	有一些场景需要满足同一个特征的数据被同一个实例消费，比如同一个 d 的传感器监测数据必须被同一个实例统计计算分析，否则可能无法获取全部的数据。又比如部分异步任务，首次请求启动 task，二次请求取消 task，此场景就必须保证**两次请求至同一实例**。

##### 实现

​	直接在配置中添加属性即可实现消息分区

###### 消息消费者yml

```yml
	cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息队列的具体实例
          group: ywb    #分组相同则不会重复消费，不同组可以重复消费
          consumer: 
            partitioned: true
      instance-count: 2
      instance-index: 0
```

从上面的配置中，我们可以看到增加了这三个参数：

1. Spring.cloud.stream.Bindings.input.consumer.Partitioned：通过该参数开启消费者分

区功能

2. Spring. Cloud. stream.instanceCount：该参数指定了当前消费者的总实例数量；

3. Spring. Cloud. stream. Instanceindex：该参数设置当前实例的索引号，从 0 开始，最大值为spring. Cloud, stream.instanceCount 参数-1。我们试验的时候需要启动多个实例，可以通过运行参数来为不同实例设置不同的索引值。

###### 消息生成者yml

```yml
	cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        output: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息队列的具体实例
          producer:
            partition-key-expression: payload
            partition-count: 2
```

从上面的配置中，我们可以看到増加了这两个参数

1. Spring.cloud.stream.Bindings.output.producer.partition-key-expression：通过该参数指定了分区键的表达式规则，我们可以根据实际的输出消息规则来配置 SpEL 来生成合适的分区键

2. Spring. Cloud. stream.Bindings.output.producer.partition-count:该参数指定了消息分

区的数量

​	到这里消息分区配置就完成了，我们可以再次启动这两个应用，同时消费者启动多个，但需要注意的是要为消费者指定不同的实例索引号，这样当同一个消息被发给消费组时，我们可以发现只有一个消费实例在接收和处理这些相同的消息。

## 八：分布式链路追踪Sleuth

### 1.概述

#### 1.微服务架构下的问题

​	在大型系统的微服务化构建中，一个系统会被拆分成许多模块。这些模块负责不同的功能，组合成系统，最终可以提供丰富的功能。在这种架构中，**一次请求往往需要涉及到多个服务。**互联网应用构建在不同的软件模块集上，这些软件模块，有可能是由不同的团队开发、可能使用不同的编程语言来实现有可能布在了几干台服务器，横跨多个不同的数据中心，也就意味着这种架构形式也会存在一些问题：

- 如何快速发现问题？
- 如何判断故暲影响范围？
- 如何梳理服务依赖以及依赖的合理性？
- 如何分析链路性能问题以及实时容量规划？

分布式链路追踪（Distributed Tracing），**就是将一次分布式请求还原成调用链路，进行日志记录，性能监控并将一次分布式请求的调用情况集中展示**。比如各个服务节点上的耗时、请求具体到达哪台机器上、每个服务节点的请求状态等等。

#### 2.Sleuth介绍

​	Sleuth提供了一套完整的服务跟踪的解决方案，在分布式系统中提供追踪解决方案并且兼容支持了zipkin

##### 1.链路图

![image-20200626002138757](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200626002138757.png)

​	上图表示一请求链路，一条链路通过 Trace Id 唯一标识，Span 标识发起的请求信息，各 span 通过parent id 关联起来

![image-20200626002407025](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200626002407025.png)

##### 2.相关概念

Trace ：类似于树结构的 Span 集合，表示一条调用链路，存在唯一标识

Span：表示调用链路来源，通俗的理解 span 就是一次请求信息

### 2.项目搭建

#### 1.前期准备

（1）下载zipkin的jar包  http://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/zipkin-server-2.12.9-exec.jar

（2）运行jar包   `java -jar zipkin-server-2.12.9-exec.jar`

（3）访问http://localhost:9411/zipkin

#### 2.服务提供方

（1）使用model：cloud-provider-payment8001

（2）改pom

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

（3）写yml

```yml
spring:
  application:
    name: cloud-payment-service
  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
      #采样率值介于 0 到 1 之间，1 则表示全部采集
      probability: 1
      
#指定了 zipkin servere 的地址，下面制定需采样的百分比，默认为 0.1, 即 10%，这里配置 1, 是记录全部的 sleuth 信息，是为了收集到更多的数据（仅供测试用）。在分布式系统中，过于频繁的采样会影响系统性能，所以这里配置需要采用一个合适的值
```

（4）业务类controller

```java
//测试zipkin
@GetMapping("/payment/zipkin")
public String zipkinPaymentLB(){
  return "zipkin";
}
```

#### 3.服务消费方

（1）使用model：cloud-comsumer-order80

（2）改pom

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

（3）写yml

```yml
spring:
  application:
    name: cloud-payment-service
  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
      #采样率值介于 0 到 1 之间，1 则表示全部采集
      probability: 1
      
#指定了 zipkin servere 的地址，下面制定需采样的百分比，默认为 0.1, 即 10%，这里配置 1, 是记录全部的 sleuth 信息，是为了收集到更多的数据（仅供测试用）。在分布式系统中，过于频繁的采样会影响系统性能，所以这里配置需要采用一个合适的值
```

（4）业务类controller

```java
//测试zipkin和sleuth
// ====================> zipkin+sleuth
@GetMapping("/consumer/payment/zipkin")
public String paymentZipkin()
{
  String result = restTemplate.getForObject("http://localhost:8001"+"/payment/zipkin/", String.class);
  return result;
}
```

（5）打开http://localhost:9411/zipkin测试

### 3.持久化

（1）找到Zipkin Serveri 持久 mysql 的数据库脚本。

（2）执行脚本

（3）配置启动服务端

```shell
java -jar zipkin-server-2.12.9-exec.jar --STORAGR_TYPE=mysql --MYSQL_HOST=127.0.0.1 --MYSQL_TCP_PORT=3306 --MYSQL_DB=zikpin --MYSQL_USER=root --MYSQL_PASS=root
```

## 九：服务注册Nacos

### 1.概述

#### 1.简介

​	Nacos（Naming Configuration Service）：是一个更易于构建云原生应用的动态服务发现，配置管理和服务管理平台。Nacos就是注册中心+配置中心的组合等价于=eureka+config+Bus

- 替代eureka做服务注册中心
- 替代Config做服务配置中心

官网：https://github.com/alibaba/nacos

文档：https://nacos.io/zh-cn/docs/quick-start.html

#### 2.安装

​	在官网https://github.com/alibaba/nacos/releases/tag/1.3.0下载，下载后解压安装包，运行bin目录下的`sh startup.sh -m standalone`，命令运行成功后直接访问http://localhost:8848/nacos。

### 2.项目构建

#### 1.基于Nacos的服务提供者

（1）建model

​	新建module：cloudalibaba-provicer-payment9001

（2）改pom

```xml
<!--新增-->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

（3）写yml

```yml
server:
  port: 9001

spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #配置Nacos地址

management:
  endpoints:
    web:
      exposure:
        include: '*'
```

（4）主启动

```java
@EnableDiscoveryClient
@SpringBootApplication
public class PaymentMain9001 {
  public static void main(String[] args) {
    SpringApplication.run(PaymentMain9001.class);
  }
}

```

（5）业务类

```java
@RestController
public class PaymentController {

  @Value("${server.port}")
  private String serverPort;

  @GetMapping(value = "/payment/nacos/{id}")
  public String getPayment(@PathVariable("id") Integer id)
  {
    return "nacos registry, serverPort: "+ serverPort+"\t id"+id;
  }
}

```

（6）测试

​	http: //localhost:9001/payment/nacos/1

（7）参照9001新建9002

#### 2.基于Nacos的服务消费者

（1）建model

​	新建module：cloudalibaba-consumer-nacos-order83

（2）改pom

```xml
<!--新增-->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

（3）写yml

```yml
server:
  port: 83

spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        
#消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
service-url:
  nacos-user-service: http://nacos-payment-provider

```

（4）主启动

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class OrderNacosMain83 {
    public static void main(String[] args) {
        SpringApplication.run(OrderNacosMain83.class);
    }
}
```

（5）业务类

```java
@RestController
public class OrderNacosController {

  @Autowired
  private RestTemplate restTemplate;

  @Value("${service-url.nacos-user-service}")
  private String serverUrl;

  @GetMapping(value = "/consumer/payment/nacos/{id}")
  public String paymentInfo(@PathVariable("id") Long id)
  {
    return restTemplate.getForObject(serverUrl+"/payment/nacos/"+id,String.class);
  }
}
```

config:

```java
@Configuration
public class ApplicationContextConfig {

  @Bean
  @LoadBalanced
  public RestTemplate getRestTemplate() {
    return new RestTemplate();
  }
}
```

（6）测试

​	http: //locahost:83/consumer/payment/nacos/1，测试发现已经实现了负载

### 3.集群搭建及持久化

#### 1.简介

官网：https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html

预计需要，1个Nginx+3nacos注册中心+1个mysql

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702122306930.png" alt="image-20200702122306930" style="zoom:50%;" />

#### 2.持久化

​	Nacos默认自带的嵌入式数据库derby，derby到mysql的切换配置步骤

（1）nacos-server-1.1.4\nacos\conf目录下找到sql脚本：nacos-mysql.sql，执行

（2）nacos-server-1.1.4\nacos\conf目录下找到application.properties，增加支持mysql数据源配置（目前只支持mysql），添加mysql数据源的url、用户名和密码。

```properties
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://11.162.196.16:3306/nacos_devtest?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=nacos_devtest
db.password=youdontknow
```

（3）启动nacos，可以看到一个全新的空记录界面，以前是记录近derby

#### 3.集群配置

（1）根据持久化内容配置Linux服务器上mysql数据库和propertise

（2）Linux服务器上nacos的集群配置cluster.conf，配置3台nacos机器的不同服务端口号

```shell
#hostname -i 查ip，然后配置
ip:端口
ip:端口
ip:端口
```

（3）编辑nacos的启动脚本startup.sh，使它能够接受不同的启动端口

​		集群启动，我们希望可以类似其它软件的 shel 命令，传递不同的端口号启动不同的 nacos 实例。命令: / startup.sh -p 3333 表示启动端口号为 3333的 nacos 服务器实例，和上一步的 cluster config 配置的一致

​		![image-20200702123726762](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702123726762.png)

![image-20200702123758254](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702123758254.png)

（4）Nginx的配置，由它作为负载均衡器，修改Nginx的配置文件，nginx.conf

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702124007467.png" alt="image-20200702124007467" style="zoom:50%;" />

（5）启动nacos和nginx

```shell
#启动nacos
sh startup.sh -p 3333
sh startup.sh -p 4444
sh startup.sh -p 5555
#启动nginx
/nginx -c /usr/local/nginx/conf/nginx.conf
```

（6）测试

​	测试通过Nginx访问nacos：http://192.168.111.144:1111/nacos/#/login

​	微服务注册，更改yml后访问http://192.168.111.144:1111/nacos/#/login

```yml
server:
  port: 9001

spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.111.144:1111 #配置Nacos地址
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

## 十：配置中心Nacos

### 1.基础配置

（1）建model：cloudalibaba-config-nacos-client3377

（2）改pom

```xml
<!--nacos-config-->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
<!--nacos-discovery-->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

（3）写yml

application.yml

```yml
spring:
  profiles:
    active: dev # 表示开发环境
```

bootstrap.yml

```yml
# nacos配置
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
#        group: DEV_GROUP   #指定分组
#        namespace: 7d8f0f5a-6a53-4785-9686-dd460158e5d4  #指定命名空间


# ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}
# nacos-config-client-dev.yaml

# nacos-config-client-test.yaml   ----> config.info
#命名空间+分组+dataID确认唯一配置文件
```

​	**bootstap.yml加载优先于application.yml**

（4）主启动

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosConfigClientMain3377 {
    public static void main(String[] args) {
        SpringApplication.run(NacosConfigClientMain3377.class);
    }
}
```

（5）业务类

```java
@RestController
@RefreshScope//通过 Spring Cloud 原生注解 @RefreshScope 实现配置自动更新
public class ConfigClientController {

  @Value("${config.info}")
  private String configInfo;

  @GetMapping("/config/info")
  public String getConfigInfo() {
    return configInfo;
  }
}
```

（6）在nacos添加配置信息

​	配置规则：

![image-20200702114546651](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702114546651.png)

​	最终公式：`${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}`

- prefix 默认为 spring. application.name的值

- spring, profile. active 即为当前环境对应的 profile，可以通过配置项 spring.profile. active 来配置。

- file-extension 为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file- extension 来配置

![image-20200702115406972](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702115406972.png)

​	根据规则在nacos中配置:

![image-20200702114939209](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702114939209.png)

（7）测试

​	http://localhost:3379/config/info

​	【通过 Spring Cloud 原生注解 @RefreshScope 实现配置自动更新】

### 2.分类配置

#### 1.问题及解决

（1）出现的问题：

​	问题 1:

实际开发中，通常一个系统会准备 dev开发环境、test 测试环境、prod 生产环境。

如何保证指定环境启动时服务能正确读取到 Nacos 上相应环境的配置文件呢？

​	问题 2:

一个大型分布式微服务系统会有很多微服务子项目，每个微服务项目又都会有相应的开发环境、测试环境、预发环境、正式环境那怎么对这些微服务配置进行管理呢？

（2）解决：

​	可以通过NameSpace+group id+Datald解决，类似 Java 里面的 package 名和类名最外层的 namespace 是可以用于区分部署环境的，Group和 Datald 逻辑上区分两个目标对象

![image-20200702120353544](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702120353544.png)

 Namespace 主要用来实现隔离，Nacos 默认的命名空间是 public，比方说我们现在有三个环境：开发、测试、生产环境，我们就可以创建三个 Namespace，不同的 Namespace 之间是隔离的。

Group 默认是 DEFAULT GROUP, Group 可以把不同的微服务划分到同一个分组里面去

Service 就是微服务；一个 Service 可以包含多个 Cluster（集群）, Nacos 默认 Cluster 是 DEFAULT, Cluster；是对指定微服务的个虚拟划分。比方说为了容灾，将 Servicel 微服务分别部署在了杭州机房和广州机房，这时就可以给杭州机房的 Service 微服务起一个集群名称（HZ),给广州机房的 Service微服务起一个集群名称（GZ），还可以尽量让同一个机房的微服务互相调用，以提升性能。

最后是 Instance，就是微服务的实例。

#### 2.配置实现

##### 1.dataId配置

​	指定spring.profile.active和配置文件的DataID来使不同环境下读取不同的配置,默认空间+默认分组+新建dev和test两个DataID。

![image-20200702120855706](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702120855706.png)

application.yml

```yml
spring:
  profiles:
    active: dev # 表示开发环境
    #active: test # 表示测试环境
    #active: info   #分组后读取
```

##### 2.group配置

​	通过Group实现环境区分,在nacos图形界面控制台上面新建配置文件DataID

![image-20200702121057942](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702121057942.png)

application.yml

```yml
spring:
  profiles:
    active: info   #分组后读取
```

bootstrap.yml

```yml
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
        group: DEV_GROUP   #指定分组
```

##### 3.nameSpace配置

​	新建dev/test的namespace

![image-20200702121545732](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702121545732.png)

​	回到服务管理-服务列表查看

![image-20200702121428035](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702121428035.png)

​	按照域名配置填写

![image-20200702121702873](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702121702873.png)

​	application.yml

```yml
spring:
  profiles:
    active: dev  
```

​	bootstrap.yml

```yml
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
        group: DEFAULT_GROUP   #指定分组
        namespace: 7d8f0f5a-6a53-4785-9686-dd460158e5d4  #指定命名空间
```

## 十一：服务熔断Sentinel

### 1.概述

#### 1.简介

​	Sentinel 是面向分布式服务架构的高可用防护组件，主要以流量为切入点，从流量控制、熔断降级、系统自适应保护等多个维度来帮助用户保障微服务的稳定性。

官网：https://github.com/alibaba/Sentinel

文档：https://sentinelguard.io/zh-cn/docs/flow-control.html

#### 2.安装

下载：https://github.com/alibaba/Sentinel/releases/，下载后到目录下运行`java -jar sentinel-dashboard-1.7.0.jar`	，运行后访问localhost:8080，输入账号密码（均为sentinel）登录

#### 3.流程图

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702145805081.png" alt="image-20200702145805081" style="zoom:50%;" />

### 2.项目搭建

（1）建model：cloudalibaba-sentinel-service8401

（2）改pom

```xml
<!--SpringCloud ailibaba nacos -->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<!--SpringCloud ailibaba sentinel-datasource-nacos 后续做持久化用到-->
<dependency>
  <groupId>com.alibaba.csp</groupId>
  <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
<!--SpringCloud ailibaba sentinel -->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
<!--openfeign-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

（3）写yml

```yml
server:
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
    sentinel:
      transport:
        dashboard: localhost:8080 #配置Sentinel dashboard地址
        port: 8719

management:
  endpoints:
    web:
      exposure:
        include: '*'

feign:
  sentinel:
    enabled: true # 激活Sentinel对Feign的支持
```

（4）主启动

```java
@SpringBootApplication
@EnableDiscoveryClient
public class SentinelMain8401 {
  public static void main(String[] args) {
    SpringApplication.run(SentinelMain8401.class);
  }
}
```

（5）业务类

```java
@RestController
public class FlowLimitController {

  @GetMapping("/testA")
  public String testA(){
    return "TestA.......";
  }

  @GetMapping("/testB")
  public String testB(){
    return "TestB.......";
  }

  @GetMapping("/testC")
  public String testC(){
    try {
      TimeUnit.SECONDS.sleep(1);
    }catch(InterruptedException e){
      e.printStackTrace();
    }
    return "testC.......";
  }
}
```

（6）测试

​	启动sentinel8080，启动微服务8401，启动8401微服务后查看sentinel控制台，发现页面并无信息，因为sentinel采用的是懒加载，访问http://localhost:8401/testA一次即可。

![image-20200702142548973](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702142548973.png)

### 3.sentinel使用

#### 1.流控规则

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702142641633.png" alt="image-20200702142641633" style="zoom:50%;" />

- 资源名：唯一名称，默认请求路径

- 针对来源：Sentinel 可以针对调用者进行限流，填写微服务名，默认 default（不区分来源）
- 阈值类型/单机阈值
  - QPS（每秒钟的请求数量）：当调用该 api 的 QPS 达到阈值的时候，进行限流 
  - 线程数：当调用 api 的线程数达到阈值的时候，进行限流
- 是否集群：不需要集群
- 流控模式
  - 直接：api 达到限流条件时，直接限流
  - 关联：当关联的资源达到值时，就限流自己
  - 链路：只记录指定链路上的流量（指定资源从入口资源进来的流量，如果达到阈值，就进行限流）【api 级别的针对来源】
- 流控效果
  - 快速失败：直接失败，抛出异常
  - Warm Up：根据 codefactor（冷加载因子，默认 3) 的值，从阈值/codefactor，经过预热时长，オ达到设置的 QPS 阈值。【冷启动，防止流量暴增压垮服务，应用**令牌桶算法**】
  - 排队等待：匀速排队，让请求以匀速的速度通过，阈值类型必须设置为 QPS，否则无效【用于以稳定的速率处理请求，通常用于处理突发请求，而不是拒绝它们。例如，消息突然流入。当大量请求同时到达时，系统可以固定速率处理所有这些传入请求，应用**漏斗算法**】

#### 2.降级规则

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702150150480.png" alt="image-20200702150150480" style="zoom:50%;" />

- RT(平均响应时间 (`DEGRADE_GRADE_RT`)：当 1s 内持续进入 N 个请求，对应时刻的平均响应时间（秒级）均超过阈值（`count`，以 ms 为单位），那么在接下的时间窗口（`DegradeRule` 中的 `timeWindow`，以 s 为单位）之内，对这个方法的调用都会自动地熔断（抛出 `DegradeException`）。注意 Sentinel 默认统计的 RT 上限是 4900 ms，**超出此阈值的都会算作 4900 ms**，若需要变更此上限可以通过启动配置项 `-Dcsp.sentinel.statistic.max.rt=xxx` 来配置。

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702150539989.png" alt="image-20200702150539989" style="zoom:50%;" />

- 异常比例 (`DEGRADE_GRADE_EXCEPTION_RATIO`)：当资源的每秒请求量 >= N（可配置），并且每秒异常总数占通过量的比值超过阈值（`DegradeRule` 中的 `count`）之后，资源进入降级状态，即在接下的时间窗口（`DegradeRule` 中的 `timeWindow`，以 s 为单位）之内，对这个方法的调用都会自动地返回。异常比率的阈值范围是 `[0.0, 1.0]`，代表 0% - 100%。
- 异常数 (`DEGRADE_GRADE_EXCEPTION_COUNT`)：当资源近 1 分钟的异常数目超过阈值之后会进行熔断。注意由于统计时间窗口是分钟级别的，若 `timeWindow` 小于 60s，则结束熔断状态后仍可能再进入熔断状态。

#### 3.热点key限流

何为热点？热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的 Top K 数据，并对其访问进行限制。比如：

- 商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
- 用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制

热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。

![image-20200702151022234](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702151022234.png)

Sentinel 利用 **LRU 策略**统计最近最常访问的热点参数，结合**令牌桶算法**来进行参数级别的流控。热点参数限流支持集群模式。

##### 配置

```java
//controller
@GetMapping("/testHotKey")
//名称随意，一般和mapping中相同,对应前台页面中的资源名
@SentinelResource(value = "testHotKey",blockHandler = "del_testHotKey")
public String testHotKey(@RequestParam(value = "p1",required = false)String p1,
                         @RequestParam(value = "p2",required = false)String p2){
  return "testHotKey.......";
}

//兜底方法，不配置限流时会出现error page
public String del_testHotKey(String p1, String p2, BlockException e){
  return "del_testHotKey";
}
```

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702151729087.png" alt="image-20200702151729087" style="zoom:50%;" />

#### 4.系统规则

官网：https://github.com/alibaba/Sentinel/wiki/%E7%B3%BB%E7%BB%9F%E8%87%AA%E9%80%82%E5%BA%94%E9%99%90%E6%B5%81

​	Sentinel 系统自适应限流从整体维度对应用入口流量进行控制，结合应用的 Load、CPU 使用率、总体平均 RT、入口 QPS 和并发线程数等几个维度的监控指标，通过自适应的流控策略，让系统的入口流量和系统的负载达到一个平衡，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702152657659.png" alt="image-20200702152657659" style="zoom:50%;" />

​	系统保护规则是从应用级别的入口流量进行控制，从单台机器的 load、CPU 使用率、平均 RT、入口 QPS 和并发线程数等几个维度监控应用指标，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。系统保护规则是应用整体维度的，而不是资源维度的，并且**仅对入口流量生效**。入口流量指的是进入应用的流量（`EntryType.IN`），比如 Web 服务或 Dubbo 服务端接收的请求，都属于入口流量。

系统规则支持以下的模式：

- **Load 自适应**（仅对 Linux/Unix-like 机器生效）：系统的 load1 作为启发指标，进行自适应系统保护。当系统 load1 超过设定的启发值，且系统当前的并发线程数超过估算的系统容量时才会触发系统保护（BBR 阶段）。系统容量由系统的 `maxQps * minRt` 估算得出。设定参考值一般是 `CPU cores * 2.5`。
- **CPU usage**（1.5.0+ 版本）：当系统 CPU 使用率超过阈值即触发系统保护（取值范围 0.0-1.0），比较灵敏。
- **平均 RT**：当单台机器上所有入口流量的平均 RT 达到阈值即触发系统保护，单位是毫秒。
- **并发线程数**：当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护。
- **入口 QPS**：当单台机器上所有入口流量的 QPS 达到阈值即触发系统保护。

#### 5.@SentinelResource

`@SentinelResource` 用于定义资源，并提供可选的异常处理和 fallback 配置项。 `@SentinelResource` 注解包含以下属性：

- `value`：资源名称，必需项（不能为空）

- `entryType`：entry 类型，可选项（默认为 `EntryType.OUT`）

- `blockHandler` / `blockHandlerClass`: `blockHandler` 对应处理 `BlockException` 的函数名称，可选项。blockHandler 函数访问范围需要是 `public`，返回类型需要与原方法相匹配，参数类型需要和原方法相匹配并且最后加一个额外的参数，类型为BlockException`。blockHandler 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `blockHandlerClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析。

- `fallback/fallbackClass`：fallback 函数名称，可选项，用于在抛出异常的时候提供 fallback 处理逻辑。fallback 函数可以针对所有类型的异常（除了exceptionsToIgnore

  里面排除掉的异常类型）进行处理。fallback 函数签名和位置要求：

  - 返回值类型必须与原函数返回值类型一致；
  - 方法参数列表需要和原函数一致，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常。
  - fallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `fallbackClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析。

- defaultFallback（since 1.6.0）：默认的 fallback 函数名称，可选项，通常用于通用的 fallback 逻辑（即可以用于很多服务或方法）。默认 fallback 函数可以针对所有类型的异常（除了exceptionsToIgnore里面排除掉的异常类型）进行处理。若同时配置了 fallback 和 defaultFallback，则只有 fallback 会生效。defaultFallback 函数签名要求：

  - 返回值类型必须与原函数返回值类型一致；
  - 方法参数列表需要为空，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常。
  - defaultFallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `fallbackClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析。

- `exceptionsToIgnore`（since 1.6.0）：用于指定哪些异常被排除掉，不会计入异常统计中，也不会进入 fallback 逻辑中，而是会原样抛出。

##### 1.按资源名称限流

```java
@RestController
public class RateLimitController {
  @GetMapping("/byResource")
  @SentinelResource(value = "byResource",blockHandler = "handleException")
  public CommonResult byResource() {
    return new CommonResult(200,"按资源名称限流测试OK",new Payment(2020L,"serial001"));
  }
  public CommonResult handleException(BlockException exception) {
    return new CommonResult(444,exception.getClass().getCanonicalName()+"\t 服务不可用");
  }
}
```

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702163440171.png" alt="image-20200702163440171" style="zoom:50%;" />

##### 2.按照URL地址限流

```java
@RestController
public class RateLimitController {
  @GetMapping("/rateLimit/byUrl")
  @SentinelResource(value = "byUrl")
  public CommonResult byUrl()
  {
    return new CommonResult(200,"按url限流测试OK",new Payment(2020L,"serial002"));
  }
}
```

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702163344825.png" alt="image-20200702163344825" style="zoom:50%;" />

##### 3.自定义限流处理

上述配置的问题：

- 系统默认的，没有体现我们自己的业务要求
- 依照现有条件，我们自定义的处理方法又和业务代码耦合在一块，不直观
- 每个业务方法都添加一个兜底的，那代码膨胀加剧。
- 全局统一的处理方法有体现。

解决：创建CustomerBlockHandler统一处理

​	（1）controller

```java
@RestController
public class RateLimitController {
  @GetMapping("/rateLimit/customerBlockHandler")
  //blockHandlerClass指定类进行降级处理,blockHandler指定类中的具体方法进行降级处理
  @SentinelResource(value = "customerBlockHandler",
                    blockHandlerClass = CustomerBlockHandler.class,
                    blockHandler = "handlerException2")
  public CommonResult customerBlockHandler()
  {
    return new CommonResult(200,"按客戶自定义",new Payment(2020L,"serial003"));
  }
}
```

​	（2）myhandler

```java
public class CustomerBlockHandler {
  
  public static CommonResult handlerException(BlockException exception){
    return new CommonResult(4444,"按客戶自定义,global handlerException---1");
  }
  
  public static CommonResult handlerException2(BlockException exception){
    return new CommonResult(4444,"按客戶自定义,global handlerException---2");
  }
}
```

#### 6.服务熔断

​	更改order83

##### 1.整合Ribbon

```java
@RestController
public class CircleBreakerController {
  public static final String SERVICE_URL = "http://nacos-payment-provider";

  @Autowired
  private RestTemplate restTemplate;

  @RequestMapping("/consumer/fallback/{id}")
  //    @SentinelResource(value = "fallback") //没有配置
  //    @SentinelResource(value = "fallback",fallback = "handlerFallback") //fallback只负责业务异常
  //    @SentinelResource(value = "fallback",blockHandler = "blockHandler") //blockHandler只负责sentinel控制台配置违规
  @SentinelResource(value = "fallback",fallback = "handlerFallback",blockHandler = "blockHandler",
                    exceptionsToIgnore = {IllegalArgumentException.class})
  public CommonResult<Payment> fallback(@PathVariable Long id)
  {
    CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/"+id,CommonResult.class,id);

    if (id == 4) {
      throw new IllegalArgumentException ("IllegalArgumentException,非法参数异常....");
    }else if (result.getData() == null) {
      throw new NullPointerException ("NullPointerException,该ID没有对应记录,空指针异常");
    }

    return result;
  }
  //本例是fallback
  public CommonResult handlerFallback(@PathVariable  Long id,Throwable e) {
    Payment payment = new Payment(id,"null");
    return new CommonResult<>(444,"兜底异常handlerFallback,exception内容  "+e.getMessage(),payment);
  }

  //本例是blockHandler
  public CommonResult blockHandler(@PathVariable  Long id, BlockException blockException) {
    Payment payment = new Payment(id,"null");
    return new CommonResult<>(445,"blockHandler-sentinel限流,无此流水: blockException  "+blockException.getMessage(),payment);
  }

  
}
```

##### 2.整合OpenFeign

```java
//service
@FeignClient(value = "nacos-payment-provider",fallback = PaymentFallbackService.class)
public interface PaymentService {

  @GetMapping(value = "/paymentSQL/{id}")
  CommonResult<Payment> paymentSQL(@PathVariable("id") Long id);

}

//serviceImpl
@Component
public class PaymentFallbackService implements  PaymentService{
  @Override
  public CommonResult<Payment> paymentSQL(Long id) {
    return new CommonResult<>(44444,"服务降级返回,---PaymentFallbackService",new Payment(id,"errorSerial"));
  }
}

//controller
@Autowired
private PaymentService paymentService;

@GetMapping(value = "/consumer/paymentSQL/{id}")
public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id){
  return paymentService.paymentSQL(id);
}
```

### 4.规则持久化

​	一旦我们重启应用，sentinel 规则将消失，生产环境需要将配置规则进行持久化。将限流配置规则時久化进 Nacos（保存，只要刷新 8401 某个 rest 地址，sentinel 控制台的流控规则就能看到，只要 Nacos 里面的配置不删除，针对 8401 上 sentinel 上的流控规则持续有效。

步骤：

修改model：8001

（1）改pom

```xml
<!--SpringCloud ailibaba sentinel-datasource-nacos 后续做持久化用到-->
<dependency>
  <groupId>com.alibaba.csp</groupId>
  <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

（2）写yml

```yml
spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
    sentinel:
      transport:
        dashboard: localhost:8080 #配置Sentinel dashboard地址
        port: 8719
      datasource:   #sentinel持久化
        ds1:
          nacos:
            server-addr: localhost:8848
            dataId: cloudalibaba-sentinel-service
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: flow
```

（3）添加Nacos业务规则

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702170824914.png" alt="image-20200702170824914" style="zoom:50%;" />

## 十二：分布式事务Seata

### 1.概述

#### 1.简介

​	一次业务操作需要垮多个数据源或需要垮多个系统进行远程调用,就会产生分布式事务问题。

​	Seata 是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。

官网：http://seata.io/zh-cn/

文档：http://seata.io/zh-cn/docs/overview/what-is-seata.html

#### 2.安装

（1）下载：https://github.com/seata/seata/releases

（2）下载后解压到指定目录并修改conf目录下的file.conf配置文件

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702190042147.png" alt="image-20200702190042147" style="zoom:33%;" />

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702190107350.png" alt="image-20200702190107350" style="zoom:33%;" />

（3）在mysql中新建数据库seata

（4）找到seata/conf.db_store.sql脚本在seata库里执行

（5）更改registry.conf

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702190533976.png" alt="image-20200702190533976" style="zoom:33%;" />

（6）启动Nacos和seata-server

#### 3.执行流程

分布式事务处理过程-------> 	ID+三组件模型

​	Transaction ID(XID)：全局唯一的事务id

​	Transaction Coordinator(TC)：事务协调器,维护全局事务的运行状态,负责协调并驱动全局事务的提交或回滚

​	Transaction Manager(TM)：控制全局事务的边界,负责开启一个全局事务,并最终发起全局提交或全局回滚的决议

​	Resource Manager(RM)：控制分支事务,负责分支注册、状态汇报,并接受事务协调的指令,驱动分支(本地)事务的提交和回滚

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702184928451.png" alt="image-20200702184928451" style="zoom:50%;" />



1.TM 向 TC 申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的 XID; 

2.XID 在微服务调用链路的上下文中传播

3.RM 向 TC 注册分支事务，将其纳入 XID 对应全局事务的管辖

4.TM 向 TC 发起针对 XID 的全局提交或回滚决议

5.TC 调度 XID 下管辖的全部分支事务完成提交或回滚请求。

### 2.订单/库存/账户业务数据库准备

#### 1.说明

​	这里创建三个服务，一个订单服务，一个库存服务，一个账户服务。当用户下单时，会在订单服务中创建一个订单，然后通过远程调用库存服务来扣减下单商品的库存，再通过远程调用账户服务来扣减用户账户里面的余额，最后在订单服务中修改订单状态为已完成，该操作跨越三个数据库，有两次远程调用，很明显会有分布式事务问题

​	下订单->减库存->扣余款->改状态

#### 2.准备库表

##### 1.创建数据库

seata_order:存储订单的数据库

seata_storage:存储库存的数据库

seata_account:存储账户信息的数据库

```sql
CREATE DATABASE seata_order
CREATE DATABASE seata_storage
CREATE DATABASE seata_account
```

##### 2.创表

seata_order库下新建t_order表

```sql
CREATE TABLE t_order (
'id' BIGINT(11) NOT NULL AUTO INCREMENT PRIMARY KEY,
'user_id' BIGINT(11) DEFAULT NULL COMMENT'用户 id',
'product_id' BIGINT(11) DEFAULT NULL,
'count' INT(11) DEFAULT NULL COMMENT'数量',
'money' DECIMAL(110) DEFAULT NULL COMMENT'金额',
'status' INT (1) DEFAULT NULL COMMENT '订单状态：0: 创建中；1: 已完结'
）ENGINE=INNODB AUTO INCREMENT=7 DEFAULT CHARSET=utf8

SELECT* FROM t order
```

seata_storage库下新建t_storage表

```sql
CREATE TABLE t_storage(
'id' BIGINT(11) NOT NULL AUTO INCREMENT PRIMARY KEY,
'product_id' BIGINT(11) DEFAULT NULL,
'total' INT(11) DEFAULT NULL COMMENT '总库存',
'used' INT (11) DEFAULT NULL COMMENT '已用库存',
'residue' INT(11) DEFAULT NULL COMMENT '剩余库存'
)ENGINE=INNODB AUTO INCREMENT=2 DEFAULT CHARSET=utf8;

INSERT INTO seata_storage.t_storage VALUES ('1','1','100','0','100'); 
SELECT * FROM t_storage
```

seata_account库下新建t_account表

```sql
CREATE TABLE t_account(
'id' BIGINT(11) NOT NULL AUTO INCREMENT PRIMARY KEY,
'user_id' BIGINT(11) DEFAULT NULL,
'total' DECIMAL(10,0) DEFAULT NULL COMMENT '总额度',
'used' DECIMAL(10,0) DEFAULT NULL COMMENT '已用额度',
'residue' DECIMAL(10,0) DEFAULT '0' COMMENT '剩余额度'
)ENGINE=INNODB AUTO INCREMENT=2 DEFAULT CHARSET=utf8;

INSERT INTO seata_account.t_account VALUES ('1','1','1000','0','1000'); 
SELECT * FROM t_account
```

##### 3.创建回滚日志

分别在各个库下执行seata-server-0.9.0\seata\conf\目录下的db_undo_log.sql脚本

### 3.项目搭建

#### 1.seata-order-service2001

（1）建model：seata-order-service2001

（2）改pom

```xml
<!--nacos-->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<!--seata-->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
  <exclusions>
    <exclusion>
      <artifactId>seata-all</artifactId>
      <groupId>io.seata</groupId>
    </exclusion>
  </exclusions>
</dependency>
<dependency>
  <groupId>io.seata</groupId>
  <artifactId>seata-all</artifactId>
  <version>0.9.0</version>
</dependency>
<!--feign-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

（3）写yml

```yml
server:
  port: 2001

spring:
  application:
    name: seata-order-service
  cloud:
    alibaba:
      seata:
        #自定义事务组名称需要与seata-server中的对应
        tx-service-group: default
    nacos:
      discovery:
        server-addr: localhost:8848
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/seata_order
    username: root
    password: 123456

feign:
  hystrix:
    enabled: false

logging:
  level:
    io:
      seata: info

mybatis:
  mapperLocations: classpath:mapper/*.xml
```

（4）在resources下新建file.conf

```conf
transport {
  # tcp udt unix-domain-socket
  type = "TCP"
  #NIO NATIVE
  server = "NIO"
  #enable heartbeat
  heartbeat = true
  #thread factory for netty
  thread-factory {
    boss-thread-prefix = "NettyBoss"
    worker-thread-prefix = "NettyServerNIOWorker"
    server-executor-thread-prefix = "NettyServerBizHandler"
    share-boss-worker = false
    client-selector-thread-prefix = "NettyClientSelector"
    client-selector-thread-size = 1
    client-worker-thread-prefix = "NettyClientWorkerThread"
    # netty boss thread size,will not be used for UDT
    boss-thread-size = 1
    #auto default pin or 8
    worker-thread-size = 8
  }
  shutdown {
    # when destroy server, wait seconds
    wait = 3
  }
  serialization = "seata"
  compressor = "none"
}

service {

  vgroup_mapping.fsp_tx_group = "default" #修改自定义事务组名称

  default.grouplist = "127.0.0.1:8091"
  enableDegrade = false
  disable = false
  max.commit.retry.timeout = "-1"
  max.rollback.retry.timeout = "-1"
  disableGlobalTransaction = false
}


client {
  async.commit.buffer.limit = 10000
  lock {
    retry.internal = 10
    retry.times = 30
  }
  report.retry.count = 5
  tm.commit.retry.count = 1
  tm.rollback.retry.count = 1
}

## transaction log store
store {
  ## store mode: file、db
  mode = "db"

  ## file store
  file {
    dir = "sessionStore"

    # branch session size , if exceeded first try compress lockkey, still exceeded throws exceptions
    max-branch-session-size = 16384
    # globe session size , if exceeded throws exceptions
    max-global-session-size = 512
    # file buffer size , if exceeded allocate new buffer
    file-write-buffer-cache-size = 16384
    # when recover batch read size
    session.reload.read_size = 100
    # async, sync
    flush-disk-mode = async
  }

  ## database store
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
    datasource = "dbcp"
    ## mysql/oracle/h2/oceanbase etc.
    db-type = "mysql"
    driver-class-name = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://127.0.0.1:3306/seata"
    user = "root"
    password = "root"
    min-conn = 1
    max-conn = 3
    global.table = "global_table"
    branch.table = "branch_table"
    lock-table = "lock_table"
    query-limit = 100
  }
}
lock {
  ## the lock store mode: local、remote
  mode = "remote"

  local {
    ## store locks in user's database
  }

  remote {
    ## store locks in the seata's server
  }
}
recovery {
  #schedule committing retry period in milliseconds
  committing-retry-period = 1000
  #schedule asyn committing retry period in milliseconds
  asyn-committing-retry-period = 1000
  #schedule rollbacking retry period in milliseconds
  rollbacking-retry-period = 1000
  #schedule timeout retry period in milliseconds
  timeout-retry-period = 1000
}

transaction {
  undo.data.validation = true
  undo.log.serialization = "jackson"
  undo.log.save.days = 7
  #schedule delete expired undo_log in milliseconds
  undo.log.delete.period = 86400000
  undo.log.table = "undo_log"
}

## metrics settings
metrics {
  enabled = false
  registry-type = "compact"
  # multi exporters use comma divided
  exporter-list = "prometheus"
  exporter-prometheus-port = 9898
}

support {
  ## spring
  spring {
    # auto proxy the DataSource bean
    datasource.autoproxy = false
  }
}
```

（5）在resources下新建registry.conf

```conf
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos"

  nacos {
    serverAddr = "localhost:8848"
    namespace = ""
    cluster = "default"
  }
  eureka {
    serviceUrl = "http://localhost:8761/eureka"
    application = "default"
    weight = "1"
  }
  redis {
    serverAddr = "localhost:6379"
    db = "0"
  }
  zk {
    cluster = "default"
    serverAddr = "127.0.0.1:2181"
    session.timeout = 6000
    connect.timeout = 2000
  }
  consul {
    cluster = "default"
    serverAddr = "127.0.0.1:8500"
  }
  etcd3 {
    cluster = "default"
    serverAddr = "http://localhost:2379"
  }
  sofa {
    serverAddr = "127.0.0.1:9603"
    application = "default"
    region = "DEFAULT_ZONE"
    datacenter = "DefaultDataCenter"
    cluster = "default"
    group = "SEATA_GROUP"
    addressWaitTime = "3000"
  }
  file {
    name = "file.conf"
  }
}

config {
  # file、nacos 、apollo、zk、consul、etcd3
  type = "file"

  nacos {
    serverAddr = "localhost"
    namespace = ""
  }
  consul {
    serverAddr = "127.0.0.1:8500"
  }
  apollo {
    app.id = "seata-server"
    apollo.meta = "http://192.168.1.204:8801"
  }
  zk {
    serverAddr = "127.0.0.1:2181"
    session.timeout = 6000
    connect.timeout = 2000
  }
  etcd3 {
    serverAddr = "http://localhost:2379"
  }
  file {
    name = "file.conf"
  }
}
```

（6）主启动

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
@EnableFeignClients
@EnableDiscoveryClient
public class SeataOrder2001 {

    public static void main(String[] args) {
        SpringApplication.run(SeataOrder2001.class);
    }

}
```

（7）业务类

config

```java
@Configuration
public class DataSourceProxyConfig {

  @Value("${mybatis.mapperLocations}")
  private String mapperLocations;

  @Bean
  @ConfigurationProperties(prefix = "spring.datasource")
  public DataSource druidDataSource(){
    return new DruidDataSource();
  }

  @Bean
  public DataSourceProxy dataSourceProxy(DataSource dataSource) {
    return new DataSourceProxy(dataSource);
  }

  @Bean
  public SqlSessionFactory sqlSessionFactoryBean(DataSourceProxy dataSourceProxy) throws Exception {
    SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
    sqlSessionFactoryBean.setDataSource(dataSourceProxy);
    sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(mapperLocations));
    sqlSessionFactoryBean.setTransactionFactory(new SpringManagedTransactionFactory());
    return sqlSessionFactoryBean.getObject();
  }

}


@Configuration
@MapperScan({"study.springcloud.dao"})
public class MyBatisConfig {
}
```

domain

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class CommonResult<T>
{
    private Integer code;
    private String  message;
    private T       data;

    public CommonResult(Integer code, String message)
    {
        this(code,message,null);
    }
}

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Order
{
    private Long id;

    private Long userId;

    private Long productId;

    private Integer count;

    private BigDecimal money;

    private Integer status; //订单状态：0：创建中；1：已完结
}
```

dao

```java
@Mapper
public interface OrderDao {
    //1 新建订单
    void create(Order order);

    //2 修改订单状态，从零改为1
    void update(@Param("userId") Long userId, @Param("status") Integer status);

}
```

mapper

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >

<mapper namespace="study.springcloud.dao.OrderDao">

  <resultMap id="BaseResultMap" type="study.springcloud.domain.Order">
    <id column="id" property="id" jdbcType="BIGINT"/>
    <result column="user_id" property="userId" jdbcType="BIGINT"/>
    <result column="product_id" property="productId" jdbcType="BIGINT"/>
    <result column="count" property="count" jdbcType="INTEGER"/>
    <result column="money" property="money" jdbcType="DECIMAL"/>
    <result column="status" property="status" jdbcType="INTEGER"/>
  </resultMap>

  <insert id="create">
    insert into t_order (id,user_id,product_id,count,money,status)
    values (null,#{userId},#{productId},#{count},#{money},0);
  </insert>


  <update id="update">
    update t_order set status = 1
    where user_id=#{userId} and status = #{status};
  </update>

</mapper>
```

service

```java
@FeignClient("seata-account-service")
public interface AccountService {
    @PostMapping("/account/decrease")
    CommonResult decrease(@RequestParam("userId")long userId, @RequestParam("money") BigDecimal money);
}

public interface OrderService {
    void create(Order order);
}

@FeignClient("seata-storage-service")
public interface StorageService {

    @PostMapping("/storage/decrease")
    CommonResult decrease(@RequestParam("productId")long productId,@RequestParam("count")Integer count);
}

@Service
public class OrderServiceImpl implements OrderService {

    @Resource
    private OrderDao dao;

    @Resource
    private StorageService ss;

    @Resource
    private AccountService as;

    @Override
    public void create(Order order) {
        System.out.println("创建订单");
        dao.create(order);
        System.out.println("库存微服务：减库存");
        ss.decrease(order.getProductId(),order.getCount());
        System.out.println("账户微服务：减钱");
        as.decrease(order.getUserId(),order.getMoney());

        //修改订单状态
        System.out.println("修改订单状态");
        dao.update(order.getUserId(),0);

        System.out.println("成功创建订单");
    }
}
```

controller

```java
@RestController
public class OrderController {
    @Autowired
    private OrderService orderService;

    @GetMapping("/order/create")
    public CommonResult create(Order order){
        orderService.create(order);
        return new CommonResult(200,"success");
    }
}
```

#### 2.seata-storage-service2002

与2001不同的是:

domain

```java
@Data
public class Storage {

  private Long id;

  /**
     * 产品id
     */
  private Long productId;

  /**
     * 总库存
     */
  private Integer total;

  /**
     * 已用库存
     */
  private Integer used;

  /**
     * 剩余库存
     */
  private Integer residue;
}
```

dao

```java
@Mapper
public interface StorageDao {

  //扣减库存
  void decrease(@Param("productId") Long productId, @Param("count") Integer count);
}
```

mapper

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >


<mapper namespace="com.atguigu.springcloud.alibaba.dao.StorageDao">

  <resultMap id="BaseResultMap" type="com.atguigu.springcloud.alibaba.domain.Storage">
    <id column="id" property="id" jdbcType="BIGINT"/>
    <result column="product_id" property="productId" jdbcType="BIGINT"/>
    <result column="total" property="total" jdbcType="INTEGER"/>
    <result column="used" property="used" jdbcType="INTEGER"/>
    <result column="residue" property="residue" jdbcType="INTEGER"/>
  </resultMap>

  <update id="decrease">
    UPDATE
    t_storage
    SET
    used = used + #{count},residue = residue - #{count}
    WHERE
    product_id = #{productId}
  </update>

</mapper>
```

service

```java
public interface StorageService {
  /**
     * 扣减库存
     */
  void decrease(Long productId, Integer count);
}

@Service
public class StorageServiceImpl implements StorageService {

  private static final Logger LOGGER = LoggerFactory.getLogger(StorageServiceImpl.class);

  @Resource
  private StorageDao storageDao;

  /**
     * 扣减库存
     */
  @Override
  public void decrease(Long productId, Integer count) {
    LOGGER.info("------->storage-service中扣减库存开始");
    storageDao.decrease(productId,count);
    LOGGER.info("------->storage-service中扣减库存结束");
  }
}
```

controller

```java
@RestController
public class StorageController {

  @Autowired
  private StorageService storageService;

  /**
     * 扣减库存
     */
  @RequestMapping("/storage/decrease")
  public CommonResult decrease(Long productId, Integer count) {
    storageService.decrease(productId, count);
    return new CommonResult(200,"扣减库存成功！");
  }
}
```

#### 3.seata-account-service2003

domain

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Account {

    private Long id;

    /**
     * 用户id
     */
    private Long userId;

    /**
     * 总额度
     */
    private BigDecimal total;

    /**
     * 已用额度
     */
    private BigDecimal used;

    /**
     * 剩余额度
     */
    private BigDecimal residue;
}
```

mapper

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >

<mapper namespace="com.atguigu.springcloud.alibaba.dao.AccountDao">

  <resultMap id="BaseResultMap" type="com.atguigu.springcloud.alibaba.domain.Account">
    <id column="id" property="id" jdbcType="BIGINT"/>
    <result column="user_id" property="userId" jdbcType="BIGINT"/>
    <result column="total" property="total" jdbcType="DECIMAL"/>
    <result column="used" property="used" jdbcType="DECIMAL"/>
    <result column="residue" property="residue" jdbcType="DECIMAL"/>
  </resultMap>

  <update id="decrease">
    UPDATE t_account
    SET
    residue = residue - #{money},used = used + #{money}
    WHERE
    user_id = #{userId};
  </update>

</mapper>
```

dao

```java
@Mapper
public interface AccountDao {

    /**
     * 扣减账户余额
     */
    void decrease(@Param("userId") Long userId, @Param("money") BigDecimal money);
}

```

service

```java
@Service
public class AccountServiceImpl implements AccountService {

  private static final Logger LOGGER = LoggerFactory.getLogger(AccountServiceImpl.class);


  @Resource
  AccountDao accountDao;

  /**
     * 扣减账户余额
     */
  @Override
  public void decrease(Long userId, BigDecimal money) {
    LOGGER.info("------->account-service中扣减账户余额开始");
    //模拟超时异常，全局事务回滚
    //暂停几秒钟线程
    try { TimeUnit.SECONDS.sleep(20); } catch (InterruptedException e) { e.printStackTrace(); }
    accountDao.decrease(userId,money);
    LOGGER.info("------->account-service中扣减账户余额结束");
  }
}
```

controller

```java
@RestController
public class AccountController {

  @Resource
  AccountService accountService;

  /**
     * 扣减账户余额
     */
  @RequestMapping("/account/decrease")
  public CommonResult decrease(@RequestParam("userId") Long userId, @RequestParam("money") BigDecimal money){
    accountService.decrease(userId,money);
    return new CommonResult(200,"扣减账户余额成功！");
  }
}
```

#### 4.测试

访问：http://localhost:2001/order/create?userId=1&productId=1&count=10&money=100

发生超时异常

添加@GlobalTransactional

```java
@Service
public class OrderServiceImpl implements OrderService {

    @Resource
    private OrderDao dao;

    @Resource
    private StorageService ss;

    @Resource
    private AccountService as;

    @Override
  	@GlobalTransactional//!!!!!!!!!!!!!!!!!!
    public void create(Order order) {
        System.out.println("创建订单");
        dao.create(order);
        System.out.println("库存微服务：减库存");
        ss.decrease(order.getProductId(),order.getCount());
        System.out.println("账户微服务：减钱");
        as.decrease(order.getUserId(),order.getMoney());

        //修改订单状态
        System.out.println("修改订单状态");
        dao.update(order.getUserId(),0);

        System.out.println("成功创建订单");
    }
}
```

再次请求测试成功

### 4.AT模式简介

官网：http://seata.io/zh-cn/docs/dev/mode/at-mode.html

##### 1.一阶段加载

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702195531869.png" alt="image-20200702195531869" style="zoom:50%;" />



在一阶段，Seata 会拦截“业务 sql

1.解析 sql 语义，找到““业务 sql”要更新的业务数据，在业务数据被更新前，将其保存成before image

2.执行“业务 sql“更新业务数据，在业务数据更新之后，

3 其保存成“after image“，最后生成行锁。

以上操作全部在一个数据库事务内完成，这样保证了一阶段操作的原子性

##### 2.二阶段提交

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702200031705.png" alt="image-20200702200031705" style="zoom:50%;" />

二阶段如是顺利提交的话，

因为“业务 SQL 在一的段已经提交至数据库，所以 Seata 框架只需将一阶段保存的快照数据和行锁删除，完成数据清理即可

##### 3.二阶段回滚

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200702200302832.png" alt="image-20200702200302832" style="zoom:50%;" />

二阶段如果是回滚的话，Seat 就需要回滚一阶段已经执行的业务 SQL ，还原业务数据。

回滚方式便是用“before image“还原业务数据；但在还原前要首先要校验脏写，对比“数据库当前业务数据”和after image 如果两份数据完全一致就说明没有脏写，可以还原业务数据，如果不一致就说明有脏写，出现脏写就需要转人工处理


# Spring

## IOC

### 1.IOC理解

​	IOC 控制反转，是将创建、管理对象的权利由开发者转移到IOC容器中的意思。开发者不再手动创建对象，而是将对象完全交于IOC容器管理，传统IOC底层用的是XML+工厂模式+反射。具体流程：开发时配置好XML文件，之后用XML解析技术解析XML中的数据，再交给工厂反射创建对象。



#### 附：ioc加载流程

​	解析配置文件->将bean注册到map中->获取bean->创建bean->给bean注入值

#### 附：循环依赖问题

##### 1.Autowired注入

1.singleton

​	结果：正常注入

​	原因：在bean还未初始化时，会将此bean通过addSingletonFactory()加入到名为singletonFactories的map缓存（三级缓存）中，如果A中先注入了B，B又注入了A，第二次的A就会从singletonFactories中获得。

2.prototype

​	结果：启动失败

​	原因：创建prototype实例时会把实例的beanName放到一个alreadyCreated这个set容器中，在注入时会检测正在创建的bean是否处于创建状态，如果是则会启动失败，打印发现了一个cycle

##### 2.构造注入

​	结果：启动失败

​	原因：因为在A实例化时就已经注入了B，此时还未将A加入singletonFactories中，B在注入A时无法获取到实例化的A，存在循环依赖

```java
DefaultSingletonBeanRegistry：

//一级缓存
/** Cache of singleton objects: bean name --> bean instance */
//用于存放完全初始化好的 bean，从该缓存中取出的 bean 可以直接使用
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);


//二级缓存
/** Cache of early singleton objects: bean name --> bean instance */
//提前曝光的单例对象的cache，存放原始的 bean 对象（尚未填充属性），用于解决循环依赖
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);


//三级缓存
/** Cache of singleton factories: bean name --> ObjectFactory */
//单例对象工厂的cache，存放 bean 工厂对象，用于解决循环依赖
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);


/** Names of beans that are currently in creation. */
// 它在Bean开始创建时放值，创建完成时会将其移出~
private final Set<String> singletonsCurrentlyInCreation = Collections.newSetFromMap(new ConcurrentHashMap<>(16));

/** Names of beans that have already been created at least once. */
// 至少被创建了一次的  都会放进这个set中
private final Set<String> alreadyCreated = Collections.newSetFromMap(new ConcurrentHashMap<>(256));


protected Object getSingleton(String beanName, boolean allowEarlyReference) {
  //获取单例bean，如果singletonObjects（一级缓存中存在），则直接ruturn
  Object singletonObject = this.singletonObjects.get(beanName);
  if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
    synchronized (this.singletonObjects) {
      //如果一级缓存中获取不到或者正在创建这个bean，则从二级缓存中获取，获取到直接return
      singletonObject = this.earlySingletonObjects.get(beanName);
      if (singletonObject == null && allowEarlyReference) {
        //如果二级缓存中扔获取不到，则从三级缓存中获取
        ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
        if (singletonFactory != null) {
          singletonObject = singletonFactory.getObject();
          //获取到后将bean放到二级缓存中
          this.earlySingletonObjects.put(beanName, singletonObject);
          //并从三级缓存中移除
          this.singletonFactories.remove(beanName);
        }
      }
    }
  }
  return singletonObject;
}
```

### 2.bean的生命周期

#### 附：bean实例化的三种方式

​	1.无参构造实例化

​	2.静态工厂实例化

​	3.实例工厂实例化

#### 附：bean注入的四种方式

​	1.构造注入

​	2.setter注入

​	3.自动注入

​	4.注解注入

#### spring实例化bean

首先要分2种情况
  第一：如果你使用BeanFactory作为Spring Bean的工厂类，则所有的bean都是在第一次使用该Bean的时候实例化
  第二：如果你使用ApplicationContext作为Spring Bean的工厂类，则又分为以下几种情况：
       （1）：如果bean的scope是singleton的，并且lazy-init为false（默认是false，所以可以不用设置），则ApplicationContext启动的时候就实例化该Bean，并且将实例化的Bean放在一个map结构的缓存中，下次再使用该Bean的时候，直接从这个缓存中取
       （2）：如果bean的scope是singleton的，并且lazy-init为true，则该Bean的实例化是在第一次使用该Bean的时候进行实例化
       （3）：如果bean的scope是prototype的，则该Bean的实例化是在第一次使用该Bean的时候进行实例化



*Bean的完整生命周期经历了各种方法调用，这些方法可以划分为以下几类：*

1. **Bean自身的方法**: 这个包括了Bean本身调用的方法和通过配置文件中`<bean>`的init-method和destroy-method指定的方法
2. **Bean级生命周期接口方法**: 这个包括了BeanNameAware、BeanFactoryAware、InitializingBean和DiposableBean这些接口的方法
3. **容器级生命周期接口方法**: 这个包括了InstantiationAwareBeanPostProcessor 和BeanPostProcessor 这两个接口实现，一般称它们的实现类为“后处理器”。
4. **工厂后处理器接口方法**: 这个包括了AspectJWeavingEnabler, ConfigurationClassPostProcessor, CustomAutowireConfigurer等等非常有用的工厂后处理器接口的方法。工厂后处理器也是容器级的。在应用上下文装配配置文件之后立即调用。



#### sington bean的生命周期

​	instantbefore  -> 执行bean的构造器（实例化） ->instantafter -> 注入属性 ->setBeanName ->setBeanFactroy  -> beforeinit-> init-method -> afterinit->执行业务方法 -> destory()->detroy-method

​	**在afterinit后会将bean放入ioc的缓存池中，并将bean引用返回给调用者**

```java
/*
  InstantiationAwareBeanPostProcessor调用postProcessBeforeInstantiation方法
  【构造器】调用Person的构造器实例化
  InstantiationAwareBeanPostProcessor调用postProcessAfterInstantiation方法
  InstantiationAwareBeanPostProcessor调用postProcessPropertyValues方法
  【注入属性】注入属性address
  【注入属性】注入属性name
  【注入属性】注入属性phone
  【BeanNameAware接口】调用BeanNameAware.setBeanName()
  【BeanFactoryAware接口】调用BeanFactoryAware.setBeanFactory()
  BeanPostProcessor接口方法postProcessBeforeInitialization对属性进行更改！
  【InitializingBean接口】调用InitializingBean.afterPropertiesSet()
  【init-method】调用<bean>的init-method属性指定的初始化方法 
  BeanPostProcessor接口方法postProcessAfterInitialization对属性进行更改！  
  InstantiationAwareBeanPostProcessor调用postProcessAfterInitialization方法
  容器初始化成功
  Person [address=广州, name=张三, phone=110]
  现在开始关闭容器！
  【DiposibleBean接口】调用DiposibleBean.destory()
  【destroy-method】调用<bean>的destroy-method属性指定的初始化方法
*/
```

#### prototype bean的生命周期

​	propotype的bean生命周期由用户管理，在getBean()时会返回两个不同的实例对象，同时会在执行一次生命周期的方法

## AOP

### 1.AOP相关概念

#### 1.理解

​	AOP 是面向切面编程，对比传统的OOP来说，他可以定义横向的关系，适用于那些与本身业务无关，但是对很多对象产生影响的公共行为和逻辑，它可以减少重复代码，降低耦合，增加系统的可维护性，常常应用在事务、日志和权限操作中。

#### 2.术语

- Joinpoint(连接点 ):所谓连接点是指那些被拦截到的点。在 spring 中,**连接点指的是方法**,因为 spring 只支持方法类型的连接点。

- Pointcut(切入点 ):所谓切入点是指我们要对哪些 Joinpoint 进行拦截的定义。

- Advice(通知/增强):所谓通知是指拦截到 Joinpoint 之后所要做的事情就是通知。 

  通知类型:

  - 前置通知：在目标方法被调用之前调用通知功能
  - 后置通知：在目标方法成功执行之后调用通知
  - 异常通知：在目标方法抛出异常后调用通知
  - 最终通知：在目标方法完成之后调用通知，此时不会关心方法的输出是什么
  - 环绕通知：通知包裹了被通知的方法，在被通知的方法调用之前和之后执行自定义的行为
  - 引介通知： 作用于类上

- Introduction(引介 ):引介是一种特殊的通知在不修改类代码的前提下, Introduction 可以在运行期为类动态地添加一些方法或 Field。

- Target(目标对象 ): 代理的目标对象。 

- Weaving(织入 ):是指把Advice应用到目标对象来创建新的代理对象的过程。spring 采用动态代理织入，而 AspectJ 采用编译期织入和类装载期织入。 

- Proxy(代理) : 一个类被 AOP 织入增强后，就产生一个结果代理类。

- Aspect(切面 ):是切入点和通知(引介)的结合。

#### 3.切入点表达式

​		表达式语法:**execution( [修饰符] 返回值类型 包名.类名.方法名(参数) )**

  - 返回值类型 :可用*表示任意返回值

  - 包名：

    - 可用*表示任意包  	
    - 可用**..**表示示当前包以及子包 （com ..类名）

  - 类名：可用*表示任意类

  - 方法名：可以使用*号，表示任意方法

  - 参数：

    - 参数列表可以使用*，表示参数可以是任意数据类型，但是**必须有参数**
    - 参数列表可以使用..，**表示有无参数均可**，有参数可以是任意类型

    **全通配写法：*  * .. * . * (..)**

```xml
<aop:config>
  <aop:pointcut id="pt1" expression="execution(* com.service.impl.*.*(..))" />
  <aop:aspect id="txAdvice" ref="txManager">
 		 <aop:around method="transactionAround" pointcut-ref="pt1"/>
  </aop:aspect>
</aop:config>
```

​		Advisor = advice + pointcut，**advice定义切面要做什么，pointcut定义切面在哪执行**，两者共同形成了切面Advisor。

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/image-20200912101218085.png" alt="image-20200912101218085" style="zoom:50%;" />

### 2.静态代理和动态代理

#### 1.静态代理

​		静态代理，就是AOP框架会在**编译阶段**生成AOP代理类，因此也称为编译时增强。

#### 2.动态代理

​		动态代理，就是说AOP框架不会去修改字节码，而是在**内存中临时为方法生成一个AOP对象**，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。

### 3.AspectJ的静态代理

​	在编译期间生成代理类，AspectJ的静态代理方式具有更好的性能，但是AspectJ需要特定的编译器进行处理，而Spring AOP则无需特定的编译器处理。

### 4.Spring的动态代理

​		*在 spring 中，框架会根据目标类是否实现了接口来决定采用哪种动态代理的方式。可以强制使用CGLib*

#### 1.两种方式

- JDK动态代理: JDK动态代理**通过“反射”**来接收被代理的类，并且要求被代理的类**必须实现一个接口**
- CGLIB动态代理:CGLIB通过**继承**的方式做的动态代理

#### 2.JDK动态代理

​	java动态代理是利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。

```java
//JDK动态代理实现InvocationHandler接口
public class JdkProxy implements InvocationHandler {
    private Object target ;//需要代理的目标对象
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("JDK动态代理，监听开始！");
        Object result = method.invoke(target, args);
        System.out.println("JDK动态代理，监听结束！");
        return result;
    }
    //定义获取代理对象方法
    private Object getJDKProxy(Object targetObject){
        //为目标对象target赋值
        this.target = targetObject;
        //JDK动态代理只能针对实现了接口的类进行代理，newProxyInstance 函数所需参数就可看出
        return Proxy.newProxyInstance(targetObject.getClass().getClassLoader(), targetObject.getClass().getInterfaces(), this);
    }
    
    public static void main(String[] args) {
        JdkProxy jdkProxy = new JdkProxy();//实例化JDKProxy对象
        UserManager user = (UserManager) jdkProxy.getJDKProxy(new UserManagerImpl());//获取代理对象
        user.addUser("admin", "123123");//执行新增方法
    }
}
```

#### 3.CGLIB动态代理

​	CGLIB动态代理是利用asm开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。

CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法。 因为是继承，所以该类或方法最好不要声明成final 

```java
//Cglib动态代理，实现MethodInterceptor接口
public class CglibProxy implements MethodInterceptor {
    private Object target;//需要代理的目标对象
    
    //重写拦截方法
    @Override
    public Object intercept(Object obj, Method method, Object[] arr, MethodProxy proxy) throws Throwable {
        System.out.println("Cglib动态代理，监听开始！");
        Object invoke = method.invoke(target, arr);//方法执行，参数：target 目标对象 arr参数数组
        System.out.println("Cglib动态代理，监听结束！");
        return invoke;
    }
    //定义获取代理对象方法
    public Object getCglibProxy(Object objectTarget){
        //为目标对象target赋值
        this.target = objectTarget;
        Enhancer enhancer = new Enhancer();
        //设置父类,因为Cglib是针对指定的类生成一个子类，所以需要指定父类
        enhancer.setSuperclass(objectTarget.getClass());
      	// 设置回调 
        enhancer.setCallback(this);
      	//创建并返回代理对象
        Object result = enhancer.create();
        return result;
    }
    
    public static void main(String[] args) {
        CglibProxy cglib = new CglibProxy();//实例化CglibProxy对象
        UserManager user =  (UserManager) cglib.getCglibProxy(new UserManagerImpl());//获取代理对象
        user.delUser("admin");//执行删除方法
    }
    
}
```

#### 4.二者的选择及转换

默认：

- 如果目标对象实现了接口，默认情况下会采用JDK的动态代理实现AOP 
- 如果目标对象实现了接口，可以强制使用CGLIB实现AOP 
- 如果目标对象没有实现了接口，必须采用CGLIB库，spring会自动在JDK动态代理和CGLIB之间转换	

选择：

- CGLib创建的对象的性能高于JDK
- JDK创建对象的速度高于CGLib
- 当对象为singleton或者有实例池则使用CGLib，反之使用JDK

转换：

```xml
<!--强制使用CGLib-->   
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

#### 5.advice的执行顺序

##### 没有异常情况下的执行顺序：

```java
around before advice
before advice
target method 执行
around after advice
after advice
afterReturning
```

##### 有异常情况下的执行顺序：

```java
around before advice
before advice
target method 执行
around after advice
after advice
afterThrowing:异常发生
java.lang.RuntimeException: 异常发生
```

## 声明式事务

### 1.事务及相关概念

#### 1.事务

​		事务是应用程序中一系列严密的操作，所有操作必须成功完成，否则在每个操作中所作的所有更改都会被撤消。也就是事务具有原子性，一个事务中的一系列的操作要么全部成功，要么一个都不做。

#### 2.特征

- 原子性：整个事务中的所有操作，要么全部完成，要么全部不完成，不可能停滞在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
- 隔离性：一个事务的执行不能被其他事务干扰。即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰
- 一致性：事务必须是使数据库从一个一致性状态变到另一个一致性状态（**一致性关注数据的可见性，中间状态的数据对外部不可见，只有最初状态和最终状态的数据对外可见**）
- 持久性：指一个事务一旦提交，它对数据库中数据的改变就应该是永久性的。

#### 3.安全性问题

不考虑隔离性引发安全性问题:

- 丢失修改：T1和T2两个事务都对一个数据进行修改，T1先修改，之后T2修改，此时T2会覆盖T1的修改
- 脏读 :一个事务读到了另一个事务的未提交的数据
- 不可重复读 :一个事务读到了另一个事务已经提交的 update 的数据导致多次查询结果不一致.
- 虚幻读 :一个事务读到了另一个事务已经提交的 insert 的数据导致多次查询结果不一致.

#### 4.事务的隔离级别

- DEFAULT 这是一个PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别.
- 未提交读（read uncommited） :是最低的事务隔离级别，它允许另外一个事务可以看到这个事务未提交的数据。(脏读，不可重复读，虚读都有可能发生)
- 已提交读 （read commited）:保证一个事物提交后才能被另外一个事务读取。另外一个事务不能读取该事物未提交的数据。(避免脏读。但是不可重复读和虚读有可能发生)                        		                				【**Oracle默认**】
- 可重复读 （repeatable read） :避免脏读和不可重复读.但是虚读有可能发生. 【**MySQL默认**】
- 串行化的 （serializable） :这是花费最高代价但最可靠的事务隔离级别。事务被处理为顺序执行。除了防止脏读，不可重复读之外，还避免了幻象读（避免三种）。

```xml
<tx:attributes>
  <tx:method name="find*" read-only="true" isolation="DEFAULT"/>
  <tx:method name="*" isolation="DEFAULT" propagation="REQUIRED"/>
</tx:attributes>
```

#### 5.事务的传播行为

​		事务传播行为（propagation behavior）指的就是**当一个事务方法被另一个事务方法调用时**，这个事务方法应该如何进行。 

* 保证同一个事务中
  - PROPAGATION_REQUIRED 支持当前事务，如果不存在 就新建一个(默认)
  - PROPAGATION_SUPPORTS 支持当前事务，如果不存在，就不使用事务
  - PROPAGATION_MANDATORY 支持当前事务，如果不存在，抛出异常

* 保证没有在同一个事务中
  - PROPAGATION_REQUIRES_NEW 如果有事务存在，挂起当前事务，创建一个新的事务(当一个操作，不管外层事务是否成功，都一定要提交，则使用此传播行为)
  - PROPAGATION_NOT_SUPPORTED 以非事务方式运行，如果有事务存在，挂起当前事务
  - PROPAGATION_NEVER 以非事务方式运行，如果有事务存在，抛出异常
  - PROPAGATION_NESTED 如果当前事务存在，则嵌套事务执行

### 2.声明式事务原理

#### 1.以前手动开启事务：

```java
 try{
	//设置手动提交事务
  	con.setAutocommit(false);
  	//执行sql
  	//提交
  	con.commit()
 }catch(Exception e)
   	//一旦其中一个操作出错都将回滚，所有操作都不成功
  	conn.rollback();    
	}	
```

#### 2.spring事务

​	Spring事务管理的核心接口是PlatformTransactionManager 

​	事务管理器接口通过getTransaction(TransactionDefinitiondefinition)方法根据指定的传播行为返回当前活动的事务或创建一个新的事务，这个方法里面的参数是TransactionDefinition类，这个类就定义了一些基本的事务属性。 在TransactionDefinition接口中定义了它自己的传播行为和隔离级别 

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/image-20200912102855221.png" alt="image-20200912102855221" style="zoom:50%;" />

​	**事务基于AOP实现**，事务对应的**Advisor**是BeanFactoryTransactionAttributeSourceAdvisor，它的**adivce**是TransactionInterceptor，**pointcut**是TransactionAttributeSourcePointcut。**spring的事务控制就是基于AOP 在 【操作1、 操作2、 操作3】 上进行advice**

```java
public static void main(String[] args) throws SQLException, InterruptedException {
  Datasource b = new Datasource();
  for(int i=0;i<3;i++) {
    new Thread(new Runnable() {
      @Override
      public void run() {
        //向操作类中传入数据源
        Operate u = new Operate(b);
        //创建事务管理类
        TransactionManager t = new TransactionManager(b);
        try {
          //1 开启事务
          t.start();
          // 业务流程
          u.buy();
          u.addShops();
          //2 提交事务
          t.commit();
          // 关闭连接
          t.close();
        } catch (Exception e) {
          try {
            //3 异常回滚
            t.rollBack();
          } catch (SQLException e1) {
            e1.printStackTrace();
          }
          e.printStackTrace();
        }
      }
    }).start();
}
```

### 3.方法不能被事务增强

##### 1.不能被增强的方法

1.JDK的动态代理中由于使用的接口做增强代理，所以被代理类为实现接口则不能被代理。

2.CGLib的动态代理是利用继承重写被代理的方法，若方法被声明为**static(父类方法不能被子类重写)或final、private**则不能被代理。

##### 2.解决方法

​	让外部的事务方法调用不能被事务增强的方法

## 设计模式

（1）工厂模式：BeanFactory就是简单工厂模式的体现，用来创建对象的实例；

（2）单例模式：Bean默认为单例模式。

（3）代理模式：Spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术；

（4）模板方法：用来解决代码重复的问题。比如. RestTemplate, JmsTemplate, JpaTemplate。

（5）观察者模式：定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知被制动更新，如Spring中listener的实现--ApplicationListener。

（6）职责链模式：在AOP中对代理对象进行调用时，会触发外层拦截器。外层拦截器根据代理配置信息，创建内层拦截器链。创建的过程中，会根据表达式判断当前拦截是否匹配这个拦截器。而这个拦截器链设计模式就是**职责链模式。**当整个链条执行到最后时，就会触发创建代理时那个尾部的默认拦截器，从而调用目标方法。最后返回

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/image-20200912101837047.png" alt="image-20200912101837047" style="zoom: 33%;" />

# SpringMVC

工作流程：

<img src="https://github.com/ywb-create/Learn-note/blob/master/img/image-20200912105230642.png" alt="image-20200912105230642" style="zoom:50%;" />

1、 用户发送请求至前端控制器DispatcherServlet

2、 DispatcherServlet收到请求调用HandlerMapping处理器映射器。

3、 处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。

4、 DispatcherServlet通过HandlerAdapter处理器适配器调用处理器

5、 执行处理器(Controller，也叫后端控制器)。

6、 Controller执行完成返回ModelAndView

7、 HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet

8、 DispatcherServlet将ModelAndView传给ViewReslover视图解析器

9、 ViewReslover解析后返回具体View

10、 DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）。

11、 DispatcherServlet响应用户返回数据

# Mybatis

> 一级缓存和二级缓存

​	一级缓存：一级缓存是 SqlSession 范围的缓存，在进行多次相同查询时，mybatis只会向数据库发出一次查询请求，其余的查询结果从缓存中获取。当调用 SqlSession 的修改，添加，删除，commit()，close()等方法时，就会清空一级缓存。

​	二级缓存：setting中设置（默认值为true，可不设置）。二级缓存是 mapper 映射级别的缓存，多个 SqlSession 去操作同一个 Mapper 映射的 sql 语句，多个SqlSession 可以共用二级缓存，二级缓存是跨 SqlSession 的。

>\#{}和${}的区别

​	\#{}表示一个占位符号。通过#{}可以实现 preparedstatement 向占位符中设置值，自动进行 java 类型和 jdbc 类型转换，#{}可以有效防止 sql 注入。#{}可以接收简单类型值或 pojo 属性值。如果 parameterlype 传输单个简单类型值，#{}括号中可以是 value或其它名称。

​	${}表示拼接 sql串。可以将 parametertype 传入的内容拼接在 sql中且不进行 jdbc 类型转换，\${}可以接收简单类型值或 pojo 属性值，如果 parametertype 传输单个简单类型值，\${}括号中只能是 value。



 



 











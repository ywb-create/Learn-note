# 权限

## 基础知识

#### 什么是权限管理? 

​	权限管理是系统的安全范畴，要求必须是合法的用户才可以访问系统(用户认证)，且必须具有该资源的访问权限才可以访问该资源(授权)。

##### 认证

​	对用户合法身份的校验，要求必须是合法的用户才可以访问系统。

![截屏2020-04-11下午8.54.37](/Users/yinwenbo/Documents/img/截屏2020-04-11下午8.54.37.png)

##### 授权

​	访问控制，必须具有该资源的访问权限才可以访问该资源。

授权的过程理解为: who对what(which)进行how操作。。
	1、who:主体即sulject, subject 在认证通过后系统进行访问控制。。
	2、what(which):资源(Resource), subject 必须具备资源的访问权限才可访问该资源。资源比如:系统用户列表页面、商品修改菜单、商品id为001的商品信息。
	3、资源分为资源类型和资源实例:。
		3.1、系统的用户信息就是资源类型，相当于java类。。
		3.2、系统中id为001的用户就是资源实例，相当于new的java对象。
	4、how:权限/许可(permission)，针对资源的权限或许可，subject 具有permission 访问资源，如何访问/操作需要定义permission权限比如:用户添加、用户修改、商品删除。

![截屏2020-04-11下午8.54.27](/Users/yinwenbo/Documents/img/截屏2020-04-11下午8.54.27.png)

#### 权限模型

​	标准权限数据模型包括:用户、角色、权限(包括资源和权限)、用户角色关系、角色权限关系。。

![截屏2020-04-10下午3.27.38](/Users/yinwenbo/Documents/img/截屏2020-04-10下午3.27.38.png)

##### 最终模型

![截屏2020-04-10下午3.28.16](/Users/yinwenbo/Documents/img/截屏2020-04-10下午3.28.16.png)

​	**注意**：在对数据进行操作时，不建议在数据库层面进行级联删除（会在service层执行dao层的业务），建议以在service层先删子表再删父表的操作

##### 权限分配

​	通过UI界面方便给用户分配权限，对上边权限模型进行增、删、改、查操作。。

##### 权限控制(RBAC)

###### 基于角色的权限控制

​	根据角色判断是否有操作权限，因为角色的变化性较高，如果角色修改需要修改控制代码，系统可扩展性不强。。

###### 基于资源的权限控制(推荐)

​	根据资源权限判断是否有操作权限，因为资源较为固定，如果角色修改或角色中权限修改不需要修改控制代码，使用此方法系统可维护性很强。建议使用。

#### 权限管理的解决方案:。

##### 对于粗颗粒权限管理（拦截器）

​	粗粒度权限管理，对资源类型的权限管理。资源类型比如:菜单、url 连接、用户添加页面、用户信息、类方法、页面中按钮
​	粗粒度权限管理比如:超级管理员可以访问户添加页面、用户信息等全部页面。部门管理员可以访问用户信息页面包括页面中所有按钮。

##### 对于细颗粒权限管理（业务层控制）	

​	细粒度权限管理，对**资源实例**的权限管理。资源实例就资源类型的具体化，比如:用户id为001的修改连接，1110班的用户信息、行政部的员工。

​	**细粒度权限管理就是数据级别的权限管理**。

​	细粒度权限管理比如:部门门经理只可以访问本部门的员工信息，用户只可以看到自己的菜单，大区经理只能查看本辖区的销售订单。

​	细颗粒权限管理是系统的业务逻辑，业务逻辑代码不方便抽取统一代码，建议在系统**业务层**进行处理。。

##### 粗粒度和细粒度例子:。

​	系统有一个用户列表查询页面，对用户列表查询分权限。

​	如果粗颗粒管理，张三和李四都有用户列表查询的权限，张三和李四都可以访问用户列表查询。
​	进一步进行细颗粒管理，张三(行政部)和李四(开发部)只可以查询自己本部门的用户信息。张三只能查看行政部的用户信息，李四只能查看开发部门的用户信息。**细粒度权限管理就是数据级别的权限管理。**

#### 基于url的权限管理(了解):

​	企业开发常用的方法，使用web应用中filter 来实现，用户请求url，通过filter 拦截，判断用户身份是否合法(用户认证)，判断请求的地址是否是用户权限范围内的url(授权)。

##### 优劣

​	优：使用基于url拦截的权限管理方式，实现起来比较简单，不依赖框架，使用web提供filter就可以实现。。

​	劣：需要将所有的url全部配置起来，有些繁琐，不易维护，url(资源)和权限表示方式不规范。。

#### shiro

​	Apache Shiro是一个强大且易用的Java安全框架,执行身份验证、授权、密码和会话管理。使用Shiro的易于理解的API,您可以快速、轻松地获得任何应用程序,从最小的移动应用程序到最大的网络和企业应用程序。

##### 基本构造

![截屏2020-04-10下午7.25.01](/Users/yinwenbo/Documents/img/截屏2020-04-10下午7.25.01.png)

##### 组件详解

1、**subject**:主体，可以是用户也可以是程序，主体要访问系统，系统需要对主体进行认证、授权。
2、**securityManager**:安全管理器，主体进行认证和授权都是通过securityManager进行。
3、**authenticator**:认证器，主体进行认证最终通过authenticator进行的。

4、**authorizer**:授权器，主体进行授权最终通过authorizer进行的。+
5、**sessionManager**: web 应用中一般是用web容器对session 进行管理，shiro 也提供一套 session管理的方式。（可以在service层访问到session，可用于C/S端架构）
6、**SessionDao**:通过SessionDao管理session数据，针对个性化的session数据存储需要使用sessionDao。(*可用于SSO单点登录*)

```java
	sessionDao可用于SSO单点登录(已被JWT替代)
    持久化session数据，写入数据库或文件持久层等。收到请求后，
    验证服务从持久层请求数据。该解决方案的优点在于架构清晰，
    而缺点是架构修改比较费劲，整个服务的验证逻辑层都需要重写，
    工作量相对较大。而且由于依赖于持久层的数据库或者问题系统，
    会有单点风险，如果持久层失败，整个认证体系都会挂掉。
```

7、**cache Manager**:缓存管理器，主要对session和授权数据进行缓存，比如将授权数据通过cacheManager进行缓存管理，和ehcache整合对缓存数据进行管理。（*将加载的权限放到缓存中，防止下次查询权限时访问数据库，造成数据库压力过大*）
8、**realm** :域，领域，相当于数据源，通过realm存取认证、授权相关数据。**注意:在realm中存储授权和认证的逻辑。**
9、**cryptography**:密码管理，提供了一套加密/解密的组件，方便开发。比如提供常用的散列、加/解密等功能。比如md5散列算法。。



##### 散列算法

​	通常需要对密码进行加密散列，常用的有md5、sha，。
​	对md5密码，如果知道散列后的值可以通过穷举算法，得到md5密码对应的明文。
​	建议对md5进行散列时加**salt** (盐)，进行加密相当于对原始密码+盐进行散列。

###### 正常使用时散列方法:

​	在程序中对原始密码+盐进行散列，将散列值存储到数据库中，并且还要将盐也要存储在数据库中。。
​	如果进行密码对比时，使用相同方法，将原始密码+盐进行散列，进行比对。

##### 权限标识符号规则

​	资源:操作:实例(中间使用半角:分隔)。

user: create:01 表示对用户资源的01实例进行create操作。。
user:create:表示对用户资源进行create操作，相当于user:create:*，对所有用户资源实例进行create操作。
user: *: 01表示对用户资源实例01 进行所有操作。。

##### shiro认证



![截屏2020-04-10下午8.39.55](/Users/yinwenbo/Documents/img/截屏2020-04-10下午8.39.55.png)



##### 执行流程

https://www.processon.com/diagraming/5e91b6f3f346fb4bdd5f59fd



##### shiro授权

![截屏2020-04-10下午9.33.13](/Users/yinwenbo/Documents/img/截屏2020-04-10下午9.33.13.png)



###### 授权流程

1、对subject进行授权，调用方法isPermitted ("permission 串")
2、SecurityManager 执行授权，通过ModularRealmAuthorizer执行授权。
3、ModularRealmAuthorizer执行realm (自定义的CustomRealm)从数据库查询权限数据。调用realm的授权方法: doGetAuthorizationInfo
4、realm从数据库查询权限数据，返回ModularRealmAuthorizers
5、ModularRealmAuthorizer 调用PermissionResolver 进行权限串比对。
6、如果比对后，isPermitted 中"permission串"在realm查询到权限数据中，说明用户访问permission串有权限，否则没有权限，抛出异常。。



###### 基于注解的授权

​	当调用controller的一个方法，由于该方法加了@RequiresPemissions("item:query")，shiro 调用realm获取数据库中的权限信息，看"item:query"是否在权限数据中存在，如果不存在就拒绝访问，如果存在就授权通过。通过后将用户的所有权限加入缓存，减少数据库的查询次数



##### 缓存清除

```java
//清除缓存(自定义realm类)
public void clearCache() {
  PrincipalCollection p=SecurityUtils.getSubject().getPrincipals();
        super.clearCache(p);
}
```

##### 记住我

​	用户登陆选择“自动登陆”本次登陆成功会向cookie写身份信息，下次登陆从cookie中取出身份信息实现自动登陆。



## 实际项目

### 功能模块图

![截屏2020-04-11下午11.17.02](/Users/yinwenbo/Documents/img/截屏2020-04-11下午11.17.02.png)

#### 负责模块

![截屏2020-04-11下午11.17.24](/Users/yinwenbo/Documents/img/截屏2020-04-11下午11.17.24.png)

#### 酒店管理数据库

role hotel role_hotel

role 经理 （id=3 type=2 air=4 ）  从role_menu表中找出 role_id=3相对应的meun_id(1、2、3)权限   从hotel中增删改查type<=2且air=4的数据





### 前端约定

api：主要是一些url

返回信息格式：

```json
//查询
{
  "status" : 0 ,          //执行状态码
  "msg"    : "SUCCESS",   //说明文字信息，没有为NULL
  "data":{
    //对象中嵌套数组，数组是返回的数据，
    "user":[{ "id": 1 ,"name"  : "xiaohong"},{"id": 2,"name":"xiaoming"}],
    "totle":3,
    "size":3,
    "current":3,
    "pages":2
  }
}
//增 删 改
{
"status" : 0 ,          //执行状态码
"msg"    : "SUCCESS",   //说明文字信息，没有为NULL
}

```

提交方法：get post delete put

表单标签<input> 的name属性名称

### 数据拟定

| 用户  | 角色         | 权限                                                         |
| ----- | ------------ | ------------------------------------------------------------ |
| 用户0 | 超级管理员   | 一切权限                                                     |
| 用户1 | ceo          | 一切权限                                                     |
| 用户2 | 总经理（省） | 系统管理（省日志、省用户（经理和员工）、省角色（经理和员工））**省酒店管理** |
| 用户3 | 经理（酒店） | 系统管理（酒店日志、用户（员工）、角色（员工））、**酒店管理** |
| 用户4 | 员工         | 酒店管理                                                     |

### 数据库表

#### log：日志表 继承dataEntity

##### dataentity（有mybatisplus自动注入）

属性：

createid创建者

createdate创建日期

updateid更新者

updatedata更新日期

delflag删除标志（逻辑删除 y正常 n删除 a审核）

remarks备注

##### 表字段：

请求类型

日志标题

ip地址

用户昵称

请求uri

操作方式

请求类型

提交的数据

sessionID

返回内容

方法执行时间

浏览器信息

地区

省 市 

网络服务提供商

异常信息

#### menu权限资源表

##### 继承treeEntity

属性：

parentid 父节点id 

level节点层次

parentids路径

sort菜单排序，根据排序显示

children：list；看有无子节点

##### menu菜单实体类

name

icon 图标

href 链接地址

target打开方式

isShow是否显示

isMenu 类型（0表示菜单，1表示按钮，-1表示目录）

bgcolor按钮颜色

permission权限标识

datacount

#### role角色表继承dataentity

字段：

角色名称

menuset：Set根据角色获取菜单

userset：根据角色获取用户

#### user用户表 继承dataentity

登录名

昵称

登录密码

salt：shior加密盐

tel手机号码

email

locked账户是否锁定，锁定则无法登陆

icon 用户头像

rolelist：set<role> 有哪些角色

menus：set<menu>有哪些菜单(权限)

#### user_role表&role_menu表

##### 获取用户的所有权限（三表查询）

![截屏2020-04-11下午11.24.23](/Users/yinwenbo/Documents/img/截屏2020-04-11下午11.24.23.png)



### 辅助类（entity.vo）

##### 菜单辅助类 showmenu

属性

id

pid父id

title

icon

href访问地址

spread菜单是展开还是收起

children：list存放子节点的列表

##### 树形菜单 treemenu

属性同上

##### 树形菜单ztreevo

id pid name url 

open是否打开

isParent

iconchildren

### 工具类

##### 常量类constants

属性（static final）：

HASH_ALGORITHM：shior采用加密算法

HASH_INTERATIONS:生成hash值的迭代次数

SALT_SIZE:生成盐的长度

validate_code:验证码

isvoice：是否开启语音提示

defalut_password:系统用户默认密码

ALLOWEDORIGINS:允许跨域请求路径

##### layui工具类LayerData

属性:

code(0代表返回成功)

count表示记录的条数

data:list 返回的数据

msg 返回的消息

##### toolUtil

属性

log：logger

方法：

entryPassword(User user)加密方法

getClientIp(httpservletrequest re)获取客户端的ip信息

![截屏2020-04-08上午11.35.08](/Users/yinwenbo/Documents/img/截屏2020-04-08上午11.35.08.png)

isAjax(httpservletrequest re)判断是否是ajax请求

getOsAndBrowseringo() 获取os 浏览器及其版本

getAdressByip()通过id获取地址及运营商

getPastDate()获取过去第几天的日期

##### httputils

doget()

dopost()

sslClient()



##### RestResponse

用于表示ajax、rest类型的web信息。继承hashmap

###### 返回RestResponse的静态方法

​	success();failure();success(string message)；

###### 属性

是否成功 、状态码、返回消息、分页信息（page、current、limit、totle） 



##### 异常工具类Exceptions

##### 编码解码工具类Encodes

##### 验证码生成工具类VerifyCodeUtil

### 配置类（config）

##### redis配置

```java
@enable
@configretion
class rediscacheconfig{

方法1：替换默认序列化：默认采用jdk徐泪花

方法2：对rediscachemanager的自定义配置
}

class cacheutils{
  @caching(evict={
    //根据userid清除当前用户的redis缓存
    @cacheEvict(value="user",key="user_id")
  })
  public user clear()//清除用户的redis缓存
}
```

##### shiroConfig

```java

@Configuration
public class ShiorConfig{
  
  private static final Logger log=LoggerFactory.getLogger(LogController.class);
  @Value("${spring.redis.host}")
  private String redisHost;
  @Value("${spring.redis.port}")
  private String redisport;
  @Value("${spring.redis.password}")
  private String redispwd;
  
  //注册一个过滤器
  @Bean
  public FilterRegistrationBean delegatingFilterProxy(){
    FilterRegistrationBean f=new FilterRegistrationBean();
    DelegatingFilterProxy d=new delegatingFilterProxy();
    //设置生命周期
    p.setTargetFilterLifeCycle(true);
  }
}


```



##### mybatisplusConfig

配置分页插件

##### springMVCconfig

配置跨域 上传文件 拦截器等组件

![截屏2020-04-08下午2.48.51](/Users/yinwenbo/Documents/img/截屏2020-04-08下午2.48.51.png)

@value()中是yml中配置好的属性

 

### 系统功能模块

#### 登录模块

在resources中添加mapper.userMapper.xml

在main方法上添加注解@mapperScan(xxx.xxx.xxx.dao)

##### controller

###### LoginController

```java
@Controller
@RequestMapping("/api")
public class LoginController{
  private static final Logger log=LoggerFactory.getLogger(LoginController.class);
  @value("${server.port}")
  private string prot;
  
  //验证码
  //前端请求 http://localhost:8080/genCaptcha
  @GetMapping("/genCaptcha")
  public void genCaptcha(req,res){
    res.setHeader("Pragma","no-cache");//设置页面不缓存
    res.setHeader("Cache-Control","no-cache");
    res.setDateHeader("Exprires",0);
    //生成验证码
    string verifyCode=工具类生成;
    //将验证码放入session中
    req.getSession().setAttribute(常量类.validate_code,verifyCode);
    log.info();
    //验证码是图形，将它返回到浏览器
    res.setContentType("image/jpeg");
    //写回浏览器
    BufferedImage bi=工具类.得到图形验证码;
    ImageIO.write(bi,"JPEG",res.getOutputStream);
  }
  
  //登录(账号密码登录)
  //前端请求http://localhost:8080/login/main
  @RequestMapping(value="login/main",produces="application/json;charset=utf-8")
  @ResponseBody
  @SysLog("用户登录")//自定义的注解，用于记录日志
  public RestResponse loginMain(req){
    string username=req.getParameter("username");
    string password=req.getParameter("username");
    string rememberMe=req.getParameter("rememberMe");
    //获取页面上输入的验证码
    string code=req.getParameter("code");
    //用来保存提示信息
    string error=null;
    Map<string,object> map=new HashMap<>();
    //判断用户名或密码是否为空
    if(username==null && "".equals(username)||password){
      return RestResponse.failure("用户名或密码不能为空");
    }
    if(rememberMe==null && "".equals(rememberMe){
      return RestResponse.failure("rememberMe不能为空");
    }
    if(code==null && "".equals(code){
      return RestResponse.failure("code不能为空");
    }
    HttpSession session=req.getSession();
    if(session==null) 
       return RestResponse.failure("session超时");
    //从session中拿到验证码
    string trueCode=session.getParamter(常量类.validate_code);
    if(truecode==null && "".equals(truecode)){
      return RestResponse.failure("验证码超时");
    }
    //判断两个验证码是否相等
    if(不相等){
      error="验证码错误";
    }else{
      //获取当前用户
      subject user=SecurityUtils.getSubject();
      //生成token
      UsernamePasswordToken token=new UsernamePasswordToken(username,password,Boolean.valueof(reemberMe));
      try{
        //执行认证提交
        user.login(token);
        //判断用户是否验证
        if(user.isAuthenticated()){
          map.put("user",user.getPrincipal());
        }
      }catch(){
				error="登录密码错误";
      }catch(){
        error="账号禁用";
      }catch(){
        error="账号锁定";
      }catch(){
        error="账号过期";
      }catch(){
        error="账号不存在";
      }catch(){
        error="没有授权";
      }
    }
       if(error==null){
         return RestResponse.success("登录成功").setData(map);
       }else{
         return RestResponse.failure(error).setData(map);
       }
  }
       
       
	//退出系统
  @RequestMappong("/systemLogout")
  @ResponseBody     
  public RestResponse systemLogOut (){
    log.info();
    //logout() subject的退出方法
    SecurityUtils.getSubject().logout;
    return RestResponse.success("退出成功").setCode(0);
  }
}
```

###### UserController&&BaseController

```java
//base包下的类
public class BaseController{
  @Autowried
  protected UserService us;
  @Autowried
  protected menuService ms;
  @Autowried
  protected RoleService rs;
  @Autowried
  protected LogService ls;
  
  //获取当前用户信息
  public User getCurrentUser(){
		ShiroUser s=SecurityUtils.getSubject().getPrincipal();
  	if(s==null){
      return null;
    }
    User user=us.selectById(s.getId());
    return user;
  }
}

@Controller
@RequestMapping("api/system/user")
public class UserController extends BaseController{
  //获取用户所拥有的菜单列表
  @RequestMapping("/getUserMenu")
  @ResponseBody   
  public RestResponse getUserMenu(){
		long userid=base.MySysUser.id();
    List<showMenu> list=ms.getShowMenuByUser(userid);
    return RestResponse.success("成功").setData(list);
  }
}
```

#### 菜单模块

##### menuController

```java
@RestController//里面有了Responsebody
@RequestMapping("api/system/menu")
public class menuController extends BaseController{
  
  //获取用户所拥有的菜单列表
  @RequestMapping("/getUserMenu")  
  public RestResponse treelist(){
		Map<String,Object> map=new HashMap();
    map.put("isShwo",false);
    map.put("parentId",null);
    return RestResponse.success("成功").setData(ms.selectAllMenus(map));
  }
  
  //添加菜单 包括父级菜单和子级菜单
  @RequestMapping("/add")
  @RequirePermission("sys:menu:add")//有此权限才能添加
  public RestResponse add(Menu menu){
    if(menu.getname()!=null && "".equals()){
      return RestResponse.failure("不能为空");
    }
    if(ms.getConutByName(menu.getName())>0){
      return RestResponse.failure("以存在");
    }
    //添加父菜单
    if(menu.getParentId()==null){
      menu.setLevel(1);
      object o=ms.slectObj();
      int sort=0;
      if(null!=o){
        sort=o+10;
        menu.setSort(sort);
      }else{
        //获取父菜单。。。
        //判断父菜单是否存在
        if(pmenu==null){
          return RestResponse.failure("父菜单不存在");
        }       
        //存在则设置当前菜单的子菜单和level。。
      }
      
    }
    //保存菜单
    ms.saveOrUpdateMenu(menu);
    return RestResponse.success("成功")
  }
  
   //编辑菜单 包括父级菜单和子级菜单
  @RequestMapping("/edit")
  @RequirePermission("sys:menu:edit")//有此权限才能编辑
  public RestResponse edit(Menu menu){
  	if(menu.getname()!=null && "".equals()){
      return RestResponse.failure("不能为空");
    }
    //流程
    //1.获取老数据，判断新老名称是否相同
    //2.看是否排序了
    //3.若正确，返回success
  }
  
  //删除菜单 
  @RequestMapping("/delete")
  @RequirePermission("sys:menu:delete")//有此权限才能编辑
  public RestResponse delete(@RequestParam(value="id",required=true)Long id){
    Menu menu=ms.selectById(id);
    //逻辑删除，不删除表中数据
    ms.setDelFlag(true);
    ms.savaOrUpdateMenu(menu);
    retrun RestResponse.success();
}

```





##### service

```java
public interface MenuService{
  List<showMenu> getShowMenuByUser(Long userid);
  List selectAllMenus(Map map);
  int getConutByName(name);
  void saveOrUpdateMenu(Menu menu);
}

@Service
@Transactional(readOnly=true,rollbackFor=Expection.class)
public class MenuServiceImpl implements MenuService{
  
  public List<showMenu> getShowMenuByUser(Long userid){
    Map<String,Object> map=new HashMap();
    map.put("userId",id);
    map.put("parentId",null);
    return baseMapper.slectShowMenuByUser(map);
  }
  
  public List selectAllMenus(Map map){
    return baseMapper.getMenus(map);
  }
  
  public int getConutByName(name){
    return baseMapper.selectCount(name);
  }
  
    	@Transactional(readOnly=false,rollbackFor=Expection.class)
  public void saveOrUpdateMenu(Menu menu){
    //mybatiesplus自带
    insertOrUpdate(menu);
  }
  
}
```

##### Dao

```java
public interface MenuDao extends BaseMapper<Menu>{
  List<showMenu> slectShowMenuByUser(Long userid);
  List<Menu> getMenus(map)
}
```

##### mapper

MenuMapper.xml

![截屏2020-04-08下午5.17.58](/Users/yinwenbo/Documents/img/截屏2020-04-08下午5.17.58.png)

##### entity

```java
@TableName("sys_menu")//映射数据库的表
public class Menu extends TreeEntity<Menu>{
	//各种属性  
}
```

#### 角色管理

##### 前端api

![截屏2020-04-08下午10.39.47](/Users/yinwenbo/Documents/img/截屏2020-04-08下午10.39.47.png)



##### controller

```java
@RestController
@RequirePermission("api/system/role")
public class RoleController extends BaseController{
  private Logger log;
  
  //获取角色列表
  @PostMapping("list")
  @RequiredPermissions("sys:role:list")
  public LayerData<Role> list(@RequestParm(value="page",defalutValue="1")int page,value="limit",defalutValue="10")int limit,httpServletRequest req){
    Map map=WebUntils.getPraametersStartingWith(req,"s_");
    LayerData<Role> roleLayerData=new LayerDate<>();
    EntityWrapper<Role> roleEntityWrapper=new EntityWrapper<>();
    roleEntityWrapper.eq("del_flag",false);
    if(!map.isEmpty()){
      String key=(String)map.get("key");
      if(StringUtils.isNotBlank(key)){
        roleEntityWrapper.like("name",key);
      }
    }
    Page<Role> rolePage=roleService.selectPage(new Page<>()page,limit,roleEntityWrapper);
    roleLayerData.setCount(rolePage.getTotal());
    roleLayerData.setData(setUserTORole(rolePage.getRecords());
  	return roleLayerData;
  }
  
  //根据创建者id或者更新者id得到用户名称
  private List<Role> setUserTORole(List<Role> roles){}
    
                          

                          
                          
  //获取所有的菜单（方便给予角色权限）
  @GetMapping("getAllMenuList")                        
  public RestResponse getAllMenuList(){
    Map map=new HashMap();
    map.put("parentId",null);
    map.put("isShow",false);
    List<Menu> menuList=ms.selectALLMenus(map);
    return RestResponse.success().setData(menuList);
  }  
  
  //新增用户角色 
  @PostMapping("add")
  @RequiredPermissions("sys:role:add")                        
  public RestResponse add(@RequestBody Role role){
    //1.判断角色名是否为空
    if(role.getName()==null){
      return RestResponse.failure("用户名为空");
    }
    //2.判断是否已经存在这个角色，存在返回failure
    if(rs.getRoleNameCount(role.getName())>0){
      return RestResponse.failure("用户名已存在");
    }
    //3.将用户保存
    rs.saveRole(role);
    return RestResponse.success()
  }     
  
  //编辑用户角色 
  @PostMapping("edit")
  @RequiredPermissions("sys:role:edit")                        
  public RestResponse add(@RequestBody Role role){
    //1.判断角色名是否为空
    if(role.getName()==null){
      return RestResponse.failure("角色名为空");
    }
    //2.判断是否已经存在这个角色，存在返回failure
    if(rs.getRoleNameCount(role.getName())>0){
      return RestResponse.failure("角色名已存在");
    }
    //3.将角色更新
    rs.updateRole(role);
    return RestResponse.success()
  }   
                          
  //根据角色获取菜单(供编辑时拉取已有权限)
  @PostMapping("getMenuByRole")
  public RestResponse getMenuByRole(Long id){
    Role role=rs.SelectById(id);
    StringBuilder sb=new StringBuilder();
    //根据角色获取菜单
    if(null!=role){
      Set<Menu> menus=new HashSet();
      if(menus!=null){
        for(Menu m:menus){
          sb.append(m.getId().toString()).append(",");
        }
      }
    }
    //查询所有菜单
    Map map=new HashMap();
    map.put("parentId",null);
    map.put("isShow",false);
    List<Menu> menuList=ms.selectALLMenus(map);
    //返回所有菜单和已有的权限菜单
    Map resMap=new HashMap();
    resMap.put("menuList",list);
    resMap.put("sb",sb);
    return RestResponse.success().setData(resMap);
  }
                          
	//根据id删除角色
  @PostMapping("delete")
  @RequirePermission("sys:menu:delete")//有此权限才能删除
  public RestResponse delete(@RequestParam(value="id",required=true)Long id){                    
    if(id==null) return RestResponse.failure("角色名为空");
    //获取角色
    Role role=rs.getRoleById(id);
    //删除角色
    rs.deleteRole(role);
    return RestResponse.success();
  }
                          
  //批量删除角色
  @PostMapping("delete")
  @RequirePermission("sys:menu:delete")//有此权限才能删除
  public RestResponse deleteSome(@RequestBody List<Role> roles){                    
    if(roles==null) return RestResponse.failure("为空");
    //删除角色
    for(Role role:roles){
      rs.deleteRole(role);
    }
    return RestResponse.success();
  }                        
}
```



##### service

```java
public interface RoleService extends IService<Role>{
  //根据角色名称统计记录
  int getRoleNameCount(String name);
  Role saveRole(Role role);
  Role updateRole(Role role);
  Role getRoleById(Long id);
  void deleteRole(Role role);
  
  //获取所有角色（添加用户时使用）
  List<Role> selectAll();
  
}


@Service
@Transcational(readOnly=true)
public class RoleServiceImpl extends ServiceImpl<RoleDao,Role> implements RoleService{
  
  //根据角色名称统计记录
  public int getRoleNameCount(String name){
    return baseMapper.selectCount(name);
  }
  //保存角色
  @Transcational(readOnly=false)
  public Role saveRole(Role role){
    //保存角色
    baseMapper.insert(role);
    //保存角色拥有的菜单
    baseMapper.saveRoleMenus(role.getId(),role.getMenuSet());
    return role;
  }
  //更新角色
  @Transcational(readOnly=false)
  public Role updateRole(Role role){
    //更新角色
    baseMapper.updateById(role);
    //删除角色拥有的菜单
    baseMapper.dropRoleMenus(role.getId());
    //保存角色更新后的菜单
    baseMapper.saveRoleMenus(role.getId(),role.getMenuSet());
    return role;
  }
  
  //根据角色id获取角色信息
  public Role getRoleById(Long id){
    Role role=bs.selectRoleById(id);
    return role;
  }
  
  //根据角色删除角色
   @Transcational(readOnly=false)
  public void deleteRole(Role role){
    //更新角色
    bs.updateById(role);
    //删除角色和菜单的关系
    bs.dropRoleMenus(role.getId());
    //删除角色和用户的关系
    bs.dropRoleUsers(role.getId());
  }
  //获取所有角色列表
  public List<Role> selectAll(){
    EntityWrapper<Role> rew=new EntityWrapper();
    wrapper.eq("del_flag",false);
    List<Role> roleList=baseMapper.selectAll(wrapper);
    return roleList;
  }
  
  
}
```



##### dao

```java
public interface RoleDao extends BaseMapper<Role>{
  //保存角色的菜单
  void saveRoleMenus(@Param("roleId")Long id,@Param("menus")Set menus)
  //删除角色和菜单关系
  void dropRoleMenus(@Param("roleId")Long id)
  //根据角色id获取角色信息
  Role selectRoleById(@Param("id")Long id);
  //删除角色和用户的关系
  void dropRoleUsers(@Param("id")Long id);
}



```



##### entity

```java
@tableName("sys_role")
public class Role{
  
}
```



##### mapper

```xml
<insert id="saveRoleMenus">
	insert into sys_role_menu(role_id,menu_id)
  values
  <foreach collection="menus" item="m" index="index">
  	(#{roleId},#{m.id})
  </foreach>
</insert>

<delete id="dropRoleMenus" parameterType="Long">
	delete from sys_role_menu where role_id=#{roleId}
</delete>

<delete id="dropRoleUsers" parameterType="Long">
	delete from sys_user_role where role_id=#{roleId}
</delete>


```

#### 用户管理

##### 前端API

![截屏2020-04-09下午12.02.58](/Users/yinwenbo/Documents/img/截屏2020-04-09下午12.02.58.png)

##### Controller

```JAVA
@RestController
@RequirePermission("api/system/user")
public class RoleController extends BaseController{
  private Logger log;
  
  //获取用户所拥有的菜单列表
  @PostMapping("getUserMenu")
  @RequiredPermissions("sys:user:list")
  public RestResponse getUserMenu(){
		Long userId = MySysUser.id();
    List<ShowMenu> list=ms.getShowMneuByUser(userId);
    return RestResponse.success().setData(list);
  }
  
  //获取用户列表
  @RequiredPermissions("sys:user:list")
  @RequsetMapping("list")
  public LayerData list(@RequestParam(value="page",defalutValue="1")int page,@RequestParam(value="limit",defalutValue="10")int limit,HttpServletRequest req){
    //将表单中的参数存到map中
		Map map="表单中的所有参数";
    //自定义的数据格式
  	LayerData<User> userData=new LayerData();
  	EntityWrapper<User> uew=new EntityWrapper();
    //判断获取的map中是否为空
    if(!map.isEmpty()){
			String keys=map.get("key");
      if(key!=null&&"".equlas(key)){
        //查询key中的login_name并放入到uew中
        uew.like("login_name",key);
      }
    }
    Page<User> userPage=us.seletPage(new Page<>(page,limit),uew);
    //将page放到自定义的数据格式里
    userData.setCount(userPage.getTotal());
    userData.setData(userPage.getRecords());
    return userData;
  }
  
  //获取所有角色列表(添加用户时要指定角色)
  @PostMapping("getAllRoles")
  public RestResponse getAllRoles(){
    List<Role> roleList=rs.selectAll();
    return roleList;
  }
  
  //新增用户
  @RequiredPermissions("sys:user:add")
  @RequsetMapping("add")
  public RestResponse add(@RequsetBody User user){
    //1.判断登录名是否为空
    //2.判断有新增时是否给新用户添加了角色
    //3.判断该用户是否已经存在
    //4.判断有否已使用
    //5.判断手机号有无绑定
    //给新用户设置默认密码
    user.setPassword(常量类.DafultPwd);
    //保存用户
    us.savaUser(user);
    //保存用户和角色的关系
    us.saveUserRoles(user.getId(),user.getRoleLists());
    return RestResponse.success();
  }
  
  //根据用户获取角色（编辑用户时需要获得他的角色关系）
  @GetMapping("getRoleByUsers")
  public RestResponse getRoleByUsers(Long id){
    User user=us.findUserById(id);
    StringBuffer sb=new StringBuffer();
    //将此用户的所有角色添加到sb中
    if(user!=null){
    	Set<Role> sets=user.getRoleList();
      if(set!=null){
        for(Role role:sets){
					sb.append(role.getId().toString().append(","));
        }
      }
    }
    //查出所有角色
    List list=rs.selectAll();
    Map map=new HashMap();
    //将已有角色和所有角色返回
    map.put("roleIds",sb);
    map.put("roleList",list);
    return RestResponse.success().setData(map);   
  } 
  
  //编辑用户
  @RequiredPermissions("sys:user:edit")
  @RequsetMapping("edit")
  public RestResponse edit(@RequsetBody User user){
    //1.判断登录名是否为空
    //2.判断有新增时是否给新用户添加了角色
    //3.判断该用户是否已经存在
    //4.判断邮箱否已使用
    //5.判断手机号有无绑定
    //判断在编辑时有没有选择角色
    //给新用户设置默认密码
    //编辑用户
    us.updateUser(user);
    //删除用户和角色的关系
    us.dropUserRolesByUserId(user.getId());
    //新增用户和角色的关系
    us.saveUserRoles(user.getId(),user.getRoleLists());
    return RestResponse.success();
  }   
  
  //根据id删除用户
  @PostMapping("delete")
  @RequirePermission("sys:user:delete")//有此权限才能删除
  public RestResponse delete(@RequestParam(value="id",required=true)Long id){                    
    if(id==null) return RestResponse.failure("用户名为空");
    //获取角色
    User user=us.getUserById(id);
    //判断用户是否为空
    //删除角色
    us.deleteUser(user);
    return RestResponse.success();
  }
                          
  //批量删除用户
  @PostMapping("deleteSome")
  @RequirePermission("sys:menu:delete")//有此权限才能删除
  public RestResponse deleteSome(@RequestBody List<User> users){                    
    if(users==null) return RestResponse.failure("为空");
    //删除角色
    for(User user:users){
      //id=1为管理员不删除
      if(user.getId()==1){
				return RestResponse.failure("不能删除管理员");
      }
      us.deleteUser(user);
    }
    return RestResponse.success();
  }                    
  
  //修改用户密码
  @PostMapping("changePwd")
  @RequirePermission("sys:user:changePwd")
  public RestResponse deleteSome(@RequestParma(value="oldPwd",required=false) String oldPwd,@RequestParma(value="newPwd",required=false) String newPwd,@RequestParma(value="conPwd",required=false) String conPwd){
    //判断旧密码是否为空
    //判断新密码是否为空
    //判断确认密码是否为空
    //判断新密码和确认密码是否相同
    
    //得到当前用户
    User user = us.findUserById(MySysUser.id());
    //判断输入旧密码是否正确
    
    user.setPassword(newPwd);
    us.updateUser(user);
    return RestResponse.success();
  }
  
  //修改用户个人信息
  @PostMapping("saveUserInfo")
  public RestResponse saveUserInfo(User user){
    //1.判断用户id、用户名
    //获取信息
    User oleUser=us.findUserById(user.getId());
    //2.判断新改的手机号和邮箱
    //更新用户
    us.updateUser(user);
    return RestResponse.success();
  }
  
  //清除用户缓存
  @GetMapping("clearUserCache")
   public RestResponse saveUserInfo(User user){
    CacheUtils.clearUserCache();
    return RestResponse.success("成功").setCode(0);
  }
}

```



##### Service

```java
public interface UserService extends Iservice<User>{
  User findUserbyLoginName(String name);
  User findUserById(Long id);
  int userCount(String param);
  User savaUser(User user);
  //保存用户和角色之间的关系（添加用户时使用）
  void saveUserRoles(Long id,Set<Role> roleSet);
  
  User updateUser(User user);
  //删除用户和角色的关系
  void dropUserRolesByUserId(Long id);
  
  User deleteUser(User user);
}


@Service
@Transactional(readOnly=true,rollbackFor=Exception.class)
public class  UserServiceImpl implements UserService{ 
  
  public User findUserbyLoginName(String name){
		Map map=new HashMap();
    map.put("loginName",name);
    User u=baseMapper.selectUserByMap(map);
    return u;
  }
  
  public User findUserById(Long id){
		Map<String,Object> map=new HashMap();
    map.put("id",id);
    User u=baseMapper.selectUserByMap(map);
    return u;
  }
  
  //统计登录名
  public int userCount(String param){
		EntityWrapper<User> w=new EntityWrapper();
    //判断登录名是否有重复
    w.eq("longin_name",param);
    int count=baseMapper.selectCount(w);
    return count;
  }
  //新增用户
  @Transactionna(readOnly=false)
  public User savaUser(User user){
    //1.工具类加密密码
    //2.设置user锁定状态为未被锁
    baseMapper.insert(user);
    return findUserByid(user.getId());
  }
  
  //保存用户和角色之间的关系（添加用户时使用）
  @Transactionna(readOnly=false)
  public void saveUserRoles(Long id,Set<Role> roleSet){
    baseMapper.saveUserRoles(id,roleSet);
  }
  
  @Transactionna(readOnly=false)
  public User updateUser(User user){
     baseMapper.updateByUserId(user.getId());
     return user;
  }
  
  @Transactionna(readOnly=false)
  public void dropUserRolesByUserId(Long id){
    baseMapper.dropUserRolesByUserId(id);
  }
  
  @Transactionna(readOnly=false)
  public User deleteUser(User user){
    //逻辑删除
    user.setDelFlag(true);
    user.updateById();
  }
}
```



##### Dao

```java
public interface UserDao extends BaseMappero<User>{
	User selectUserByMapper(Map<String,Object> map);
  
  void saveUserRoles(@Param("userId")Long id,@Param("roleSet")Set<Role> roleSet);
  
  void dropUserRolesByUserId(@Param("userId")Long id);

}
```

##### mapeer

```xml
<insert id="saveUserRoles">
	insert into sys_user_role(user_id,role_id)
  values
  <foreach collection="roleIds" item="m" index="index" separator=",">
  	(#{user_id},#{m.id})
  </foreach>
</insert>


<delete id="dropUserRolesByUserId" prameterType="Long">
  delete from sys_user_role where user_id=#{userId}
</delete>
```

#### 日志管理

##### 前端接口

![截屏2020-04-09下午6.13.25](/Users/yinwenbo/Documents/img/截屏2020-04-09下午6.13.25.png)

##### Controller

```java
@RestContreller
@RequestMapping("api/system/log")
public class LogController extends BaseController{
  private static final Logger log=LoggerFactory.getLogger(LogController.class);
  
  //返回日志列表
  @RequiredPermissions("sys:log:list")
  @RequsetMapping("list")
  public LayerData list(@RequestParam(value="page",defalutValue="1")int page,@RequestParam(value="limit",defalutValue="10")int limit,HttpServletRequest req){
    //将表单中的参数存到map中
		Map map="表单中的所有参数";
    //自定义的数据格式
  	LayerData<Log> logData=new LayerData();
  	EntityWrapper<Log> lew=new EntityWrapper();
    //判断获取的map中是否为空
    if(!map.isEmpty()){
      //查询type字段
			String type=map.get("type");
      if(type!=null&&"".equlas(type)){
        //查询数据库中的type字段并放入到uew中
        uew.eq("type",type);
      }
      //模糊查询title字段
      String title=map.get("title");
      if(type!=null&&"".equlas(type)){
        //查询数据库中的title字段并放入到uew中
        uew.like("title",title);
      }
      //查询用户名字段
      String username=map.get("username");
      if(username!=null&&"".equlas(username)){
        //查询数据库中的username字段并放入到uew中
        uew.eq("username",username);
      }
      //根据请求方法查询
      String method=map.get("method");
      if(method!=null&&"".equlas(method)){
        //查询数据库中的method字段并放入到uew中
        uew.eq("method",method);
      }  
    }
    Page<Log> logPage=logService.selectPage(new Page<>(page,limit),uew);
    //将page放到自定义的数据格式里
    logData.setCount(logPage.getTotal());
    logData.setData(logPage.getRecords());
    return logData;
  } 
  
  //删除日志
  @RequiredPermissions("sys:log:delete")
  @RequsetMapping("delete")
  public RestResponse delete(@RequestParam("ids[]")List<Long> list){
    if(list==null && list.size()==0){
      return RestResponse.failure("请选择要删除的记录");
    }
    //调用Iservice的批量删除方法
    ls.deleteBatchIds(list);
    return RestResponse.success();
  }
  
  
}

```



##### Service

```java
//Iservice是mybatis plus的公共接口，Iservice已经有了基础的seletPage方法
public interface logService extends Iservice<Log>{
  
}

//ServiceImpl是mybatis plus的公共实现类
@Service
@Transactional(readOnly=true,rollbackFor=Exception.class)
public class logServiceImpl extends ServiceImpl<LogDao,Log> implements LogService{
  
}
```



##### Dao

```java
//BaseMapper是mybatis plus的公共接口,不需要xml就可使用基本的方法
public interface LogDao extends BaseMapper<>{
  
}
```



##### annotation&&aspect新增日志就在每个方法上加入@SysLog

```java
//annotation包（这是日志的切入点）
@Target(ElementType.Method)
@Retention(RetentionPolicy.Runtime)
@Documented
public @interface SysLog{
  String value() dafault"";
}


//aspect包
@Aspect
@Order(5)
@Component
public class WebLogAspect{
  private ThreadLocal<Long> startTime =new ThreadLocal();
  
  @Autowired
  private GlobalExceptionHandler exceptionHandle;
  
  private Log sysLog=null;
  
  private static final Logger log=LoggerFactory.getLogger(LogController.class);
  
  //定义切入点
  @Pointcut("@annotation(xx.xx.annotation.SysLog)")
  public void WebLog(){}
  
  //前置通知
  @Before("webLog()")
  public void doBefore(JointPoint jp){
    //ThreadLocal设置开始时间
    startTime.set(System.currentTimeMillis());
    //接收到请求，记录请求的内容
    ServletRequestAttribute sra=(ServletRequestAttribute)RequestContextHolder.getRequestAttribute();
    //得到request对象
    HttpServletRequest request=sra.getRequest();
    //获取session对象
    HttpSession session=(HttpSession)sra.resolveReference(RequestAttributes.REEERENCE_SESSION);
    //定义日志对象
    sysLog=new Log();
    //设置sysLog中的值
    sysLog.setXXX(jp.getXXX);
    sysLog.setHttpMethod(request.getMethod());
    //获取传入目标的参数!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    Object[] args =jp.getArgs();
    for(int i=0;i<arg.length;i++){
      if(o instanceof ServletRequest || Response || MutipartFile){
        arg[i]=o.toString();
      }
    }
    String str=JSONObject.toJsonONString(args);
    sysLog=setParams(str.length()>5000?JSONObject.toJsonONString("过长不显示"):str)；
  	String ip =ToolUtil.getClientIp(request);
  	//设置ip
  	sysLog.setRemoteAddr(ip);
    //设置uri
  	sysLog.setRequestUri(request.getRequestUrl().toString());
    //判断session是否为空设置session
  	sysLog.setSession(session);
  	
  	MethodSignature s=jp.getSignature();
  	Method method=s.getMethod();
  	SysLog myLog=method.getAnnotation(xx.xx.annotation.SysLog.class);
  	if(myLog!=null){
      //获取自定义注解上的信息并设置到sysLog中
      sysLog.setTitle(myLog.value());
    }
    //设置浏览器及其os的信息
  	//设置请求类型
    //获取登录名
  }
  
  
  //后置通知
  @AfterReturning(returning="ret",pointcut="webLog()")
  public void after(Object ret){
    //设置用户名、response、时间
    //将日志添加到数据库
    sysLog.insert();
  }
  
  //环绕通知
  @Around("webLog()")
  public void round(ProceedingJointPoint pjp)throw Throwable{
    try{
      Object o=pjp.proceed();
      retrun o;
    }catch(Exception e){
      e.print();
      sysLog.setException(e.getMession());
      throw e;
    }
  }
}


```

#### 项目体会

最后开发的日志模块，后来就要添加到每个方法上，显式的调用日志的添加和删除方法，很麻烦，然后就想到了使用aop的方式，自定义了一个注解，切入到注解上，在每个模块的日志处添加这个注解就可以了，简便开发

threadloca的应用：日志记录时down了一个日期工具类，里面的simpleDateFromat的prase()加锁了，了解了是因为parse()调用了其中的全局变量calender的add和clear方法，线程一线以后调用会导致解析不正确，然后就改成了ThreadLocal存储simpleDateFromat
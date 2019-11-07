---
title: Shiro—小而美的安全框架
categories: [Coding, Spring-Boot]
tags:
  - Shiro
id: shiro-in-practice
date: 2019-08-05 11:21:40
description: Shiro—小而美的安全框架，中小项目的选择，更灵活的定制
keywords: Shiro,Spring-Security,安全框架,轻巧
---

## 写在前面
在一款应用的整个生命周期，我们都会谈及该应用的数据安全问题。用户的合法性与数据的可见性是数据安全中非常重要的一部分。但是，一方面，不同的应用对于数据的合法性和可见性要求的维度与粒度都有所区别；另一方面，以当前微服务、多服务的架构方式，如何共享Session，如何缓存认证和授权数据应对高并发访问都迫切需要我们解决。Shiro的出现让我们可以快速和简单的应对我们应用的数据安全问题

## Shiro介绍
### Shiro简介
这个官网解释不抽象，所以直接用官网解释：Apache Shiro™是一个强大且易用的 Java 安全框架，可以执行身份验证、授权、加密和会话管理等。基于 Shiro 的易于理解的API，您可以快速、轻松地使任何应用程序变得安全（从最小的移动应用到最大的网络和企业应用）。

谈及安全，多数 Java 开发人员都离不开 Spring 框架的支持，自然也就会先想到 Spring Security，那我们先来看二者的差别

| Shiro  | Spring Security  |
|---|---|
| 简单、灵活  |复杂、笨重   |
|可脱离Spring   |不可脱离Spring   |
|粒度较粗   | 粒度较细  |

虽然 Spring Security 属于名震中外 Spring 家族的一部分，但是了解 Shiro 之后，你不会想 “嫁入豪门”，而是选择追求「诗和远方」冲动。

横看成岭侧成峰，远近高低各不同 (依旧是先了解概念就好)
### 远看 Shiro 看轮廓

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-05/image001.png)

#### Subject
它是一个主体，代表了当前“用户”，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是Subject，如网络爬虫，机器人等；即一个抽象概念；所有 Subject 都绑定到 SecurityManager，与 Subject 的所有交互都会委托给SecurityManager；可以把 Subject 认为是一个门面；SecurityManager 才是实际的执行者

#### SecurityManager
安全管理器；即所有与安全有关的操作都会与 SecurityManager 交互；且它管理着所有 Subject；可以看出它是 Shiro 的核心，它负责与后边介绍的其他组件进行交互，如果学习过 SpringMVC，你可以把它看成 DispatcherServlet前端控制器

#### Realm
域，Shiro 从 Realm 获取安全数据（如用户、角色、权限），就是说 SecurityManager 要验证用户身份，那么它需要从 Realm 获取相应的用户进行比较以确定用户身份是否合法；也需要从 Realm 得到用户相应的角色/权限进行验证用户是否能进行操作；可以把 Realm 看成 DataSource，即安全数据源。

### 近看 Shiro 看细节
看图瞬间懵逼？别慌，会为你拆解来看，结合着图看下面的解释，这不是啥大问题，且看:

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-05/image002.png)

#### Subject
主体，可以看到主体可以是任何可以与应用交互的 “用户”

#### SecurityManager
相当于 SpringMVC 中的 DispatcherServlet；是 Shiro 的心脏；所有具体的交互都通过 SecurityManager 进行控制；它管理着所有 Subject、且负责进行认证和授权、及会话、缓存的管理

#### Authenticator
认证器，负责主体认证的，这是一个扩展点，如果用户觉得 Shiro 默认的不好，可以自定义实现；需要自定义认证策略（Authentication Strategy），即什么情况下算用户认证通过了

#### Authrizer
授权器，或者访问控制器，用来决定主体是否有权限进行相应的操作；即控制着用户能访问应用中的哪些功能

#### Realm
可以有 1 个或多个 Realm，可以认为是安全实体数据源，即用于获取安全实体的；可以是JDBC实现，也可以是LDAP实现，或者内存实现等等；由用户提供；注意：Shiro 不知道你的用户/权限存储在哪及以何种格式存储；所以我们一般在应用中都需要实现自己的Realm

#### SessionManager
如果写过 Servlet 就应该知道 Session 的概念，Session 需要有人去管理它的生命周期，这个组件就是 SessionManager；而Shiro 并不仅仅可以用在 Web 环境，也可以用在如普通的 JavaSE 环境、EJB等环境；所以，Shiro 就抽象了一个自己的Session 来管理主体与应用之间交互的数据；这样的话，比如我们在 Web 环境用，刚开始是一台Web服务器；接着又上了台EJB 服务器；这时又想把两台服务器的会话数据放到一个地方，我们就可以实现自己的分布式会话（如把数据放到Memcached 服务器）

#### SessionDAO
DAO大家都用过，数据访问对象，用于会话的 CRUD，比如我们想把 Session 保存到数据库，那么可以实现自己的SessionDAO，通过如JDBC写到数据库；比如想把 Session 放到 Memcached 中，可以实现自己的 Memcached SessionDAO；另外 SessionDAO 中可以使用 Cache 进行缓存，以提高性能；

#### CacheManager
缓存控制器，来管理如用户、角色、权限等的缓存的；因为这些数据基本上很少去改变，放到缓存中后可以提高访问的性能

#### Cryptography
密码模块，Shiro提高了一些常见的加密组件用于如密码「加密/解密」的

**注意上图的结构，我们会根据这张图来逐步拆分讲解，记住这张图也更有助于我们理解 Shiro 的工作原理**，所以依旧是打开两个网页一起看就好喽

## 搭建概览
多数小伙伴都在使用 Spring Boot， Shiro 也很应景的定义了 starter，做了更好的封装，对于我们来说使用起来也就更加方便，来看选型概览

| 序号  | 名称  | 版本   |
|---|---|---|
| 1  |Springboot   | 2.0.4  |
|  2 | JPA	  | 2.0.4  |
| 3  | Mysql  |8.0.12   |
|  4 | Redis  |2.0.4   |
| 5  |Lombok	   |1.16.22   |
| 6  |Guava   |26.0-jre   |
| 7  |Shiro   | 1.4.0  |

使用 Spring Boot，大多都是通过添加 starter 依赖，会自动解决依赖包版本，所以自己尝试的时候用最新版本不会有什么问题，比如 Shiro 现在的版本是 1.5.0 了，整体问题不大，大家自行尝试就好

### 添加 Gradle 依赖管理
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-05/2019-08-05-11-06-41%402x.png)

### 大体目录结构
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-05/2019-08-05-09-29-54.png)


### application.yml 配置

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-05/2019-08-05-11-07-29%402x.png)

### 基本配置
![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-05/2019-08-05-11-08-17%402x.png)


![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-05/6af89bc8gw1f8su5fhsizg208c08ct9n.gif)

你就让我看这？这只是一个概览，先做到心中有数，我们来看具体配置，逐步完成搭建

其中 shiroFilter bean 部分指定了拦截路径和相应的过滤器，”/user/login”, ”/user”, ”/user/loginout” 可以匿名访问，其他路径都需要授权访问，shiro 提供和多个默认的过滤器，我们可以用这些过滤器来配置控制指定url的权限(先了解个大概即可)：

| 配置缩写          | 对应的过滤器                   | 功能                                                                                                                                                                                                                                                         |
|-------------------|--------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| anon              | AnonymousFilter                | 指定url可以匿名访问                                                                                                                                                                                                                                          |
| authc             | FormAuthenticationFilter       | 指定url需要form表单登录，默认会从请求中获取username、password,rememberMe等参数并尝试登录，如果登录不了就会跳转到loginUrl配置的路径。我们也可以用这个过滤器做默认的登录逻辑，但是一般都是我们自己在控制器写登录逻辑的，自己写的话出错返回的信息都可以定制嘛。 |
| authcBasic        | BasicHttpAuthenticationFilter  | 指定url需要basic登录                                                                                                                                                                                                                                         |
| Logout            | LogoutFilter                   | 登出过滤器，配置指定url就可以实现退出功能，非常方便                                                                                                                                                                                                          |
| noSessionCreation | NoSessionCreationFilter        | 禁止创建会话                                                                                                                                                                                                                                                 |
| perms             | PermissionsAuthorizationFilter | 需要指定权限才能访问                                                                                                                                                                                                                                         |
| port              | PortFilter                     | 需要指定端口才能访问                                                                                                                                                                                                                                         |
| rest              | HttpMethodPermissionFilter     | 将http请求方法转化成相应的动词来构造一个权限字符串，这个感觉意义不大，有兴趣自己看源码的注释                                                                                                                                                                 |
| roles             | RolesAuthorizationFilter       | 需要指定角色才能访问                                                                                                                                                                                                                                         |
| ssl               | SslFilter                      | 需要https请求才能访问                                                                                                                                                                                                                                        |
| user              | UserFilter                     | 需要已登录或“记住我”的用户才能访问                                                                                                                                                                                                                           |

### 数据库表设计
数据库表设计请参考 entity package下的 bean，通过@Entity 注解与 JPA 的设置自动生成表结构 (你需要简单的了解一下 JPA 的功能)。

我们要说重点啦～～～
## 身份认证
身份认证是一个证明 “李雷是李雷，韩梅梅是韩梅梅” 的过程，回看上图，Realm 模块就是用来做这件事的，Shiro 提供了 IniRealm，JdbcReaml，LDAPReam等认证方式，但自定义的 Realm 通常是最适合我们业务需要的，认证通常是校验登录用户是否合法。

### 新建用户 User
```java
@Data
@Entity
public class User implements Serializable {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;

    @Column(unique =true)
    private String username;

    private String password;

    private String salt;

}
```

### 定义 Repository
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    public User findUserByUsername(String username);

}
```

### 编写UserController：

```java
@GetMapping("/login")
public void login(String username, String password) {
    UsernamePasswordToken token = new UsernamePasswordToken(username, password);
    token.setRememberMe(true);
    Subject currentUser = SecurityUtils.getSubject();
    currentUser.login(token);
}
```

### 自定义 Realm
自定义 Realm，主要是为了重写 doGetAuthenticationInfo(…)方法
```java
@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
    UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
    String username = token.getUsername();
    User user = userRepository.findUserByUsername(username);
    SimpleAuthenticationInfo simpleAuthenticationInfo = new SimpleAuthenticationInfo(user, user.getPassword(), getName());
    simpleAuthenticationInfo.setCredentialsSalt(ByteSource.Util.bytes(user.getSalt()));
    return simpleAuthenticationInfo;
}
```
这些代码我需要做一个说明，你可能也满肚子疑惑:
1. 这段代码怎么应用了 shiro？
2. controller 是怎么调用到 custom realm 的？
3. 重写的  doGetAuthenticationInfo(…) 方法目的是什么？

### 认证流程说明
用户访问` /user/login` 路径，生成 UsernamePasswordToken,  通过SecurityUtils.getSubject()获取Subject（currentUser），调用 login 方法进行验证，让我们跟踪一下代码，瞧一瞧就知道自定义的CustomRealm怎样起作用的，一起来看源码：

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-05/image013.png)


到这里我们要停一停了，请回看 Shiro 近景图，将源码追踪路径与其对比，是完全一致的

## 授权
身份认证是验证你是谁的问题，而授权是你能干什么的问题，

> 产品经理：申购模块只能科室看
> 程序员：好的
> 产品经理：科长权限大一些，他也能看申购模块
> 程序员：好的(黑脸)
> 产品经理：科长不但能看，还能修改数据
> 程序员：关公提大刀，拿命来
…

作为程序员，我们的宗旨是：「能动手就不吵吵」; 硝烟怒火拔地起，耳边响起驼铃声（Shiro）：「放下屠刀，立地成佛」授权没有那么麻烦，大家好商量…

整个过程和身份认证基本是一毛一样，你对比看看
### 角色实体创建
涉及到授权，自然要和角色相关，所以我们创建 Role 实体:
```java
@Data
@Entity
public class Role {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;

    @Column(unique =true)
    private String roleCode;

    private String roleName;
}
```

### 新建 Role Repository

```java
@Repository
public interface RoleRepository extends JpaRepository<Role, Long> {

    @Query(value = "select roleId from UserRoleRel ur where ur.userId = ?1")
    List<Long> findUserRole(Long userId);

    List<Role> findByIdIn(List<Long> ids);

}
```

### 定义权限实体 Permission
```java
@Data
@Entity
public class Permission {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;

    @Column(unique =true)
    private String permCode;

    private String permName;
}
```

### 定义 Permission Repository
```java
@Repository
public interface PermissionRepository extends JpaRepository<Permission, Long> {

    @Query(value = "select permId from RolePermRel pr where pr.roleId in ?1")
    List<Long> findRolePerm(List<Long> roleIds);

    List<Permission> findByIdIn(List<Long> ids);
}

```

### 建立用户与角色关系
其实可以通过 JPA 注解来制定关系的，这里为了说明问题，以单独外键形式说明
```java
@Data
@Entity
public class UserRoleRel {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;

    private Long userId;

    private Long roleId;


}
```

### 建立角色与权限关系
```java
@Data
@Entity
public class RolePermRel {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;

    private Long permId;

    private Long roleId;

}
```

### 编写 UserController
```java
@RequiresPermissions("user:list:view")
@GetMapping()
public void getAllUsers(){
    List<User> users = userRepository.findAll();
}
```
`@RequiresPermissions("user:list:view")` 注解说明具有用户：列表：查看权限的才可以访问），官网明确给出权限定义格式，包括通配符等，我希望你自行去查看

自定义 CustomRealm (主要重写 doGetAuthorizationInfo) 方法:

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-05/2019-08-05-11-10-16%402x.png)

与认证流程如出一辙，只不过多了用户，角色，权限的关系罢了
### 授权流程说明
这里通过过滤器（见Shiro配置）和注解二者结合的方式来进行授权，和认证流程一样，最终会走到我们自定义的 CustomRealm 中，同样 Shiro 默认提供了许多注解用来处理不同的授权情况

| 注解                    | 功能                                                                               |
|-------------------------|------------------------------------------------------------------------------------|
| @RequiresGuest          | 只有游客可以访问                                                                   |
| @RequiresAuthentication | 需要登录才能访问                                                                   |
| @RequiresUser           | 已登录的用户或“记住我”的用户能访问                                                 |
| @RequiresRoles          | 已登录的用户需具有指定的角色才能访问                                               |
| @RequiresPermissions    | 已登录的用户需具有指定的权限才能访问（如果不想和产品经理华山论剑，推荐用这个注解） |

授权官网给出明确的授权策略与案例，请查看：http://shiro.apache.org/permissions.html

上面的例子我们通过一直在通过访问 Mysql 获取用户认证和授权信息，这中方式明显不符合生产环境的需求

## Session会话管理
做过 Web 开发的同学都知道 Session 的概念，最常用的是 Session 过期时间，数据在 Session 的 CRUD，同样看上图，我们需要关注 SessionManager 和 SessionDAO 模块，Shiro starter 已经提供了基本的 Session配置信息，我们按需在YAML中配置就好（官网https://shiro.apache.org/spring-boot.html 已经明确给出Session的配置信息）

| Key                                               | Default Value | Description                                                                                       |
|---------------------------------------------------|---------------|---------------------------------------------------------------------------------------------------|
| shiro.enabled                                     | true          | Enables Shiro’s Spring module                                                                     |
| shiro.web.enabled                                 | true          | Enables Shiro’s Spring web module                                                                 |
| shiro.annotations.enabled                         | true          | Enables Spring support for Shiro’s annotations                                                    |
| shiro.sessionManager.deleteInvalidSessions        | true          | Remove invalid session from session storage                                                       |
| shiro.sessionManager.sessionIdCookieEnabled       | true          | Enable session ID to cookie, for session tracking                                                 |
| shiro.sessionManager.sessionIdUrlRewritingEnabled | true          | Enable session URL rewriting support                                                              |
| shiro.userNativeSessionManager                    | false         | If enabled Shiro will manage the HTTP sessions instead of the container                           |
| shiro.sessionManager.cookie.name                  | JSESSIONID    | Session cookie name                                                                               |
| shiro.sessionManager.cookie.maxAge                | -1            | Session cookie max age                                                                            |
| shiro.sessionManager.cookie.domain                | null          | Session cookie domain                                                                             |
| shiro.sessionManager.cookie.path                  | null          | Session cookie path                                                                               |
| shiro.sessionManager.cookie.secure                | false         | Session cookie secure flag                                                                        |
| shiro.rememberMeManager.cookie.name               | rememberMe    | RememberMe cookie name                                                                            |
| shiro.rememberMeManager.cookie.maxAge             | one year      | RememberMe cookie max age                                                                         |
| shiro.rememberMeManager.cookie.domain             | null          | RememberMe cookie domain                                                                          |
| shiro.rememberMeManager.cookie.path               | null          | RememberMe cookie path                                                                            |
| shiro.rememberMeManager.cookie.secure             | false         | RememberMe cookie secure flag                                                                     |
| shiro.loginUrl                                    | /login.jsp    | Login URL used when unauthenticated users are redirected to login page                            |
| shiro.successUrl                                  | /             | Default landing page after a user logs in (if alternative cannot be found in the current session) |
| shiro.unauthorizedUrl                             | null          | Page to redirect user to if they are unauthorized (403 page)                                      |

分布式服务中，我们通常需要将Session信息放入Redis中来管理，来应对高并发的访问需求，这时只需重写SessionDAO即可完成自定义的Session管理

### 整合Redis
```java
@Configuration
public class RedisConfig {

    @Autowired
    private RedisConnectionFactory redisConnectionFactory;

    @Bean
    public RedisTemplate<String, Object> stringObjectRedisTemplate() {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        return template;
    }

}
```

### 重写SessionDao

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-05/2019-08-05-11-10-59%402x.png)

查看源码，可以看到调用默认SessionManager的retriveSession方法，我们重写该方法，将Session放入HttpRequest中，进一步提高session访问效率

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-05/2019-08-05-11-11-37%402x.png)

### 向ShiroConfig中添加配置
 其实在概览模块已经给出代码展示，这里单独列出来做说明:
```java
/**
 * 自定义RedisSessionDao用来管理Session在Redis中的CRUD
 * @return
 */
@Bean(name = "redisSessionDao")
public RedisSessionDao redisSessionDao(){
    return new RedisSessionDao();
}

/**
 * 自定义SessionManager,应用自定义SessionDao
 * @return
 */
@Bean(name = "customerSessionManager")
public CustomerWebSessionManager customerWebSessionManager(){
    CustomerWebSessionManager customerWebSessionManager = new CustomerWebSessionManager();
    customerWebSessionManager.setSessionDAO(redisSessionDao());
    return customerWebSessionManager;
}

/**
 * 定义Security manager
 * @param customRealm
 * @return
 */
@Bean(name = "securityManager")
public DefaultWebSecurityManager defaultWebSecurityManager(CustomRealm customRealm) {
    DefaultWebSecurityManager  securityManager = new DefaultWebSecurityManager ();
    securityManager.setRealm(customRealm);
    securityManager.setSessionManager(customerWebSessionManager()); // 可不指定，Shiro会用默认Session manager
    securityManager.setCacheManager(redisCacheManagers());  //可不指定，Shiro会用默认CacheManager
//        securityManager.setSessionManager(defaultWebSessionManager());
    return securityManager;
}

/**
 * 定义session管理器
 * @return
 */
@Bean(name = "sessionManager")
public DefaultWebSessionManager defaultWebSessionManager(){
    DefaultWebSessionManager defaultWebSessionManager = new DefaultWebSessionManager();
    defaultWebSessionManager.setSessionDAO(redisSessionDao());
    return defaultWebSessionManager;
}
```
至此，将 session 信息由 redis 管理功能就这样完成了


## 缓存管理
应对分布式服务，对于高并发访问数据库权限内容是非常低效的方式，同样我们可以利用Redis来解决这一问题，将授权数据缓存到Redis中
### 新建 RedisCache
```java
@Slf4j
@Component
public class RedisCache<K, V> implements Cache<K, V> {

    public static final String SHIRO_PREFIX = "shiro-cache:";

    @Resource
    private RedisTemplate<String, Object> stringObjectRedisTemplate;

    private String getKey(K key){
        if (key instanceof String){
            return (SHIRO_PREFIX + key);
        }
        return key.toString();
    }

    @Override
    public V get(K k) throws CacheException {
        log.info("read from redis...");
        V v = (V) stringObjectRedisTemplate.opsForValue().get(getKey(k));
        if (v != null){
            return v;
        }
        return null;
    }

    @Override
    public V put(K k, V v) throws CacheException {
        stringObjectRedisTemplate.opsForValue().set(getKey(k), v);
        stringObjectRedisTemplate.expire(getKey(k), 100, TimeUnit.SECONDS);
        return v;
    }

    @Override
    public V remove(K k) throws CacheException {
        V v = (V) stringObjectRedisTemplate.opsForValue().get(getKey(k));
        stringObjectRedisTemplate.delete((String) get(k));
        if (v != null){
            return v;
        }
        return null;
    }

    @Override
    public void clear() throws CacheException {
        //不要重写，如果只保存shiro数据无所谓
    }

    @Override
    public int size() {
        return 0;
    }

    @Override
    public Set<K> keys() {
        return null;
    }

    @Override
    public Collection<V> values() {
        return null;
    }
}
```

### 新建 RedisCacheManager

```java
public class RedisCacheManager implements CacheManager {

    @Resource
    private RedisCache redisCache;

    @Override
    public <K, V> Cache<K, V> getCache(String s) throws CacheException {
        return redisCache;
    }
}
```

至此，我们不用每次访问 Mysql DB 来获取认证和授权信息，而是通过 Redis 来缓存这些信息，大大提升了效率，也满足分布式系统的设计需求

## 总结
回复公众号 「demo」获取 demo 代码。这里只是梳理了Springboot整合Shiro的流程，以及应用Redis最大化利用Shiro，Shiro的使用细节还很多，官网说的也很明确，带着上面的架构图来理解Shiro会事半功倍，感觉这里面的代码挺多挺头大的？那是你没有自己动手去尝试，结合官网与 demo 相信你会对 Shiro 有更好的理解，另外你可以理解 Shiro 是 mini 版本的 Spring Security，我希望以小见大，当需要更细粒度的认证授权时，也会对理解 Spring Security 有很大帮助，点击文末「阅读原文」，效果更好

落霞与孤鹜齐飞 秋水共长天一色，产品经理和程序员一片祥和…  

## 灵魂追问
1. 都说 Redis 是单线程，但是很快，你知道为什么吗？
2. 你们项目中是怎样控制认证授权的呢？当授权有变化，对于程序员来说，这个修改是灾难吗？

## 提高效率工具

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E6%84%9F%E6%83%B3%E4%B8%8E%E6%80%BB%E7%BB%93/_image/2019-06-18/b.png)

### MarkDown 表格生成器
本文的好多表格是从官网粘贴的，如何将其直接转换成 MD table 呢？那么 https://www.tablesgenerator.com/markdown_tables 就可以帮到你了，无论是生成 MD table，还是粘贴内容生成 table 和内容都是极好的，当然了不止 MD table，自己发现吧，更多工具，公众号回复 「工具」获得

![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/Spring/_image/2019-08-05/2019-08-05-10-52-55.png)

--------

## 推荐阅读
+ [只会用 git pull ？有时候你可以尝试更优雅的处理方式 ](https://mp.weixin.qq.com/s/6dg3u2PkcTSQHu_3T_QYnA)
+ [双亲委派模型：大厂高频面试题，轻松搞定](https://mp.weixin.qq.com/s/Dnr1jLebvBUHnziZzSfcrA)
+ [面试还不知道BeanFactory和ApplicationContext的区别？](https://mp.weixin.qq.com/s/YBQB086ADBjHUmwrFQrWew)
+ [如何设计好的RESTful API](https://mp.weixin.qq.com/s/hR1TqkVzwZ_T8fuMnsM4hQ)
+ [程序猿为什么要看源码？](https://mp.weixin.qq.com/s/V7h8O6pVFQ-nr_iA2SNqtw)

--------
> ### 欢迎持续关注公众号：「日拱一兵」
> - 前沿 Java 技术干货分享
> - 高效工具汇总  回复「工具」
> - 面试问题分析与解答
> - 技术资料领取 回复「资料」

后续会推出「多线程」与「ElasticSearch」等连载内容

> 以读侦探小说思维轻松趣味学习 Java 技术栈相关知识，本着将复杂问题简单化，抽象问题具体化和图形化原则逐步分解技术问题，技术持续更新，请持续关注......


![](https://rgyb.sunluomeng.top/%E5%85%AC%E4%BC%97%E8%B4%A6%E5%8F%B7%E6%96%87%E7%AB%A0/%E6%84%9F%E6%83%B3%E4%B8%8E%E6%80%BB%E7%BB%93/_image/2019-06-18/a%20%281%29.png)

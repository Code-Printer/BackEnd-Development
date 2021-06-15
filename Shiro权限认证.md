项目中使用Shiro安全框架技术实现管理员、老师、学生不同角色赋予不同权限，以限制不同角色，访问不同的功能。例如老师有开课功能，管理员有管理用户的功能。
## Shiro  
Shiro安全框架是一种可以对用户进行角色分配，把权限分配给角色，不同的角色具有不同的权限。并在用户登录时进行角色验证的安全认证技术。内置了数据加密算法MD5+盐(salt)+多次加密对数据进行保护，内置了过滤器和JdbcRealm(进行数据库的角色更新查询操作)
### Shiro的授权流程    
调用授权方法(
    //初始化数据源，给用户赋予
        accountRealm.addAccount("Jack", "123","root","admin");
        //构建环境
    defaultSecurityManager.setRealm(accountRealm);)之后底层自动调用到Realm进行授权。
底层实现过程：  
![result](https://static01.imgkr.com/temp/9f82cb46272e4d96994b790b9457bf85.png) 

### Shiro的认证流程  
调用认证方法( 
    //创建操作主体
        Subject subject = SecurityUtils.getSubject();

        //获取用户输入的账号密码，和Realm中的一致
        UsernamePasswordToken usernamePasswordToken =
                new UsernamePasswordToken("jack", "456");

        //认证，调用此方法即执行认证
        subject.login(usernamePasswordToken);)   
底层实现：  
![result](https://static01.imgkr.com/temp/8d022101a9ef4941a031b214da46404f.png)  

### Shiro的常用API  
```java
//是否有对应的角色，返回boolean
subject.hasRole()
//获取subject名，即用户名
subject.getPrincipal()
//检查是否有对应的角色，无返回值，若无角色会报错
subject.checkRole()
//检查是否具有对应的权限，无返回值，若无权限会报错
subject.checkPermission()
//是否有对应的权限，返回boolean
subject.isPermitted()    
//是否通过认证
subject.isAuthenticated()
//退出登录
subject.logout();
```  
# Shiro安全框架
Apache shiro是一个可以处理身份授权、验证、会话管理、加密的开源安全框架。  
### Apache Shiro特性
Apache Shiro是一个全面的、蕴含丰富功能的安全框架。下图为描述Shiro功能的框架图：  
![result](https://static01.imgkr.com/temp/e06bcd2461874a40b43b8b0d04e223a6.png)  

1、Authentication（认证）：用户身份识别，通常被称为用户“登录”  
2、Authorization（授权）：访问控制。比如某个用户是否具有某个操作的使用权限。  
3、Session Management（会话管理）：特定于用户的会话管理,甚至在非web 或 EJB 应用程序。  
4、Cryptography（加密）：在对数据源使用加密算法加密的同时，保证易于使用。    
特别是对以下的功能支持：  
Web支持：Shiro的Web支持API有助于保护Web应用程序。  
缓存：缓存是Apache Shiro API中的第一级，以确保安全操作保持快速和高效。  
并发性：Apache Shiro支持具有并发功能的多线程应用程序。  
测试：存在测试支持，可帮助您编写单元测试和集成测试，并确保代码按预期得到保障。  
“运行方式”：允许用户承担另一个用户的身份(如果允许)的功能，有时在管理方案中很有用。  
“记住我”：记住用户在会话中的身份，所以用户只需要强制登录即可。   
### Shiro架构  
hiro 架构包含三个主要的理念：Subject,SecurityManager和 Realm。  
![result](https://static01.imgkr.com/temp/1c687e64bf214ed5a590e4b58897d9d0.png)   

1、subject：当前用户，Subject 可以是一个人，但也可以是第三方服务、守护进程帐户、始终守护任务或者其它–当前和软件交互的任何事件。  
2、SecurityManager：管理所有Subject，SecurityManager 是 Shiro 架构的核心，配合内部安全组件共同组成安全伞。  
3、Realms：用于进行权限信息的验证，我们自己实现。Realm 本质上是一个特定的安全 DAO：它封装与数据源连接的细节，得到Shiro 所需的相关的数据。在配置 Shiro 的时候，你必须指定至少一个Realm 来实现认证（authentication）和/或授权（authorization）。    
我们需要实现Realms的Authentication 和 Authorization。其中 Authentication 是用来验证用户身份，Authorization 是授权访问控制，用于对用户进行的操作授权，证明该用户是否允许进行当前操作，如访问某个链接，某个资源文件等。  
### Shiro的认证过程  
代码过程实现：  
1、首先我们新建一个Maven工程，然后在pom.xml中引入相关依赖：  
```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.4.0</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
```  
2、新建一个【AuthenticationTest】测试类：  
```java
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.mgt.DefaultSecurityManager;
import org.apache.shiro.realm.SimpleAccountRealm;
import org.apache.shiro.subject.Subject;
import org.junit.Before;
import org.junit.Test;

public class AuthenticationTest {

    SimpleAccountRealm simpleAccountRealm = new SimpleAccountRealm();

    @Before // 在方法开始前添加一个用户
    public void addUser() {
        simpleAccountRealm.addAccount("wmyskxz", "123456");
    }

    @Test
    public void testAuthentication() {

        // 1.构建SecurityManager环境
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        defaultSecurityManager.setRealm(simpleAccountRealm);

        // 2.主体提交认证请求
        SecurityUtils.setSecurityManager(defaultSecurityManager); // 设置SecurityManager环境
        Subject subject = SecurityUtils.getSubject(); // 获取当前主体

        UsernamePasswordToken token = new UsernamePasswordToken("wmyskxz", "123456");
        subject.login(token); // 登录

        // subject.isAuthenticated()方法返回一个boolean值,用于判断用户是否认证成功
        System.out.println("isAuthenticated:" + subject.isAuthenticated()); // 输出true

        subject.logout(); // 登出

        System.out.println("isAuthenticated:" + subject.isAuthenticated()); // 输出false
    }
}
```  
流程如下：
1、首先调用 Subject.login(token) 进行登录，其会自动委托给 Security Manager，调用之前必须通过 SecurityUtils.setSecurityManager() 设置；
2、SecurityManager 负责真正的身份验证逻辑；它会委托给 Authenticator 进行身份验证；
3、Authenticator才是真正的身份验证者，Shiro API 中核心的身份认证入口点，此处可以自定义插入自己的实现；
4、Authenticator可能会委托给相应的AuthenticationStrategy进行多Realm身份验证，默认ModularRealmAuthenticator会调用 AuthenticationStrategy进行多Realm身份验证；
5、Authenticator会把相应的token传入Realm，从 Realm 获取身份验证信息，如果没有返回/抛出异常表示身份验证失败了。此处可以配置多个Realm，将按照相应的顺序及策略进行访问。  
### Shiro的授权过程  
代码过程实现：   
1、新建一个【AuthenticationTest】测试类：  
```java
public class AuthenticationTest {

    SimpleAccountRealm simpleAccountRealm = new SimpleAccountRealm();

    @Before // 在方法开始前添加一个用户,让它具备admin和user两个角色
    public void addUser() {
        simpleAccountRealm.addAccount("wmyskxz", "123456", "admin", "user");
    }

    @Test
    public void testAuthentication() {

        // 1.构建SecurityManager环境
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        defaultSecurityManager.setRealm(simpleAccountRealm);

        // 2.主体提交认证请求
        SecurityUtils.setSecurityManager(defaultSecurityManager); // 设置SecurityManager环境
        Subject subject = SecurityUtils.getSubject(); // 获取当前主体

        UsernamePasswordToken token = new UsernamePasswordToken("wmyskxz", "123456");
        subject.login(token); // 登录

        // subject.isAuthenticated()方法返回一个boolean值,用于判断用户是否认证成功
        System.out.println("isAuthenticated:" + subject.isAuthenticated()); // 输出true
        // 判断subject是否具有admin和user两个角色权限,如没有则会报错
        subject.checkRoles("admin","user");
//        subject.checkRole("xxx"); // 报错
    }
}
```  
### 自定义 Realm  
实际进行权限信息验证的是我们的Realm，Shiro 框架内部默认提供了两种实现，一种是查询.ini文件的IniRealm，另一种是查询数据库的JdbcRealm，这两种来说都相对简单，我们着重就来介绍介绍自定义实现的 Realm吧。
1、我们先在合适的包下创建一个【MyRealm】类，继承 Shiro 框架的 AuthorizingRealm 类，并实现默认的两个方法：  
```java
package com.demo.realm;

import org.apache.shiro.authc.*;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;

import java.util.*;

public class MyRealm extends AuthorizingRealm {

    /**
     * 模拟数据库数据
     */
    Map<String, String> userMap = new HashMap<>(16);

    {
        userMap.put("wmyskxz", "123456");
        super.setName("myRealm"); // 设置自定义Realm的名称，取什么无所谓..
    }

    /**
     * 授权
     *
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        String userName = (String) principalCollection.getPrimaryPrincipal();
        // 从数据库获取角色和权限数据
        Set<String> roles = getRolesByUserName(userName);
        Set<String> permissions = getPermissionsByUserName(userName);

        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        simpleAuthorizationInfo.setStringPermissions(permissions);
        simpleAuthorizationInfo.setRoles(roles);
        return simpleAuthorizationInfo;
    }

    /**
     * 模拟从数据库中获取权限数据
     *
     * @param userName
     * @return
     */
    private Set<String> getPermissionsByUserName(String userName) {
        Set<String> permissions = new HashSet<>();
        permissions.add("user:delete");
        permissions.add("user:add");
        return permissions;
    }

    /**
     * 模拟从数据库中获取角色数据
     *
     * @param userName
     * @return
     */
    private Set<String> getRolesByUserName(String userName) {
        Set<String> roles = new HashSet<>();
        roles.add("admin");
        roles.add("user");
        return roles;
    }

    /**
     * 认证
     *
     * @param authenticationToken 主体传过来的认证信息
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        // 1.从主体传过来的认证信息中，获得用户名
        String userName = (String) authenticationToken.getPrincipal();

        // 2.通过用户名到数据库中获取凭证
        String password = getPasswordByUserName(userName);
        if (password == null) {
            return null;
        }
        SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo("wmyskxz", password, "myRealm");
        return authenticationInfo;
    }

    /**
     * 模拟从数据库取凭证的过程
     *
     * @param userName
     * @return
     */
    private String getPasswordByUserName(String userName) {
        return userMap.get(userName);
    }
}
``` 
2、编写测试类，来验证是否正确  
```java
import com.demo.realm.MyRealm;
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.mgt.DefaultSecurityManager;
import org.apache.shiro.subject.Subject;
import org.junit.Test;

public class AuthenticationTest {

    @Test
    public void testAuthentication() {

        MyRealm myRealm = new MyRealm(); // 实现自己的 Realm 实例

        // 1.构建SecurityManager环境
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        defaultSecurityManager.setRealm(myRealm);

        // 2.主体提交认证请求
        SecurityUtils.setSecurityManager(defaultSecurityManager); // 设置SecurityManager环境
        Subject subject = SecurityUtils.getSubject(); // 获取当前主体

        UsernamePasswordToken token = new UsernamePasswordToken("wmyskxz", "123456");
        subject.login(token); // 登录

        // subject.isAuthenticated()方法返回一个boolean值,用于判断用户是否认证成功
        System.out.println("isAuthenticated:" + subject.isAuthenticated()); // 输出true
        // 判断subject是否具有admin和user两个角色权限,如没有则会报错
        subject.checkRoles("admin", "user");
//        subject.checkRole("xxx"); // 报错
        // 判断subject是否具有user:add权限
        subject.checkPermission("user:add");
    }
}
```  
### Shiro 加密  
我们在数据库中保存的密码都是明文的，一旦数据库数据泄露，那就会造成不可估算的损失，所以我们通常都会使用非对称加密算法，而md5加密算法就是符合这样的一种算法。   
<font color="white">加盐 + 多次加密</font>  
由于相同的密码 md5 一样，当密码相同时，即可获得相同的md5值，即可破解密码。解决办法是md5再加一个随机数，然后再进行 md5 加密，这个随机数就是我们说的盐(salt)，这样处理下来就能得到不同的 Md5 值，当然我们需要把这个随机数盐也保存进数据库中，以便我们进行验证。另外我们可以通过多次加密的方法，即使黑客通过一定的技术手段拿到了我们的密码 md5 值，但它并不知道我们到底加密了多少次，所以这也使得破解工作变得艰难。  
代码实现：在 Shiro 框架中，对于这样的操作提供了简单的代码实现：

```java
String password = "123456";
String salt = new SecureRandomNumberGenerator().nextBytes().toString();
int times = 2;  // 加密次数：2
String alogrithmName = "md5";   // 加密算法

String encodePassword = new SimpleHash(alogrithmName, password, salt, times).toString();

System.out.printf("原始密码是 %s , 盐是： %s, 运算次数是： %d, 运算出来的密文是：%s ",password,salt,times,encodePassword);
```    



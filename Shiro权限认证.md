项目中使用Shiro安全框架技术实现管理员、老师、学生不同角色赋予不同权限，以限制不同角色，访问不同的功能。例如老师有开课功能，管理员有管理用户的功能。还有使用shiro自带的非对称加密模块，使用md5+盐+多次加密的方式对用户密码进行加密存储并比对。
## Shiro  
Shiro安全框架是一种具有授权、认证、会话管理和加密功能的，可以对用户进行角色分配，把权限分配给角色，不同的角色具有不同的权限。并在用户登录时进行角色验证的安全认证技术。内置了数据加密算法MD5+盐(salt)+多次加密对数据进行保护，内置了过滤器和JdbcRealm(进行数据库的角色更新查询操作)
### Shiro的授权流程(授权在于根据用户信息查看用户的角色及权限信息)    
代码演示：
```java
public class QuickStartTest4_3 {
    private SimpleAccountRealm accountRealm = new SimpleAccountRealm();
    private DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();

    @Before
    public void init() {
        //初始化数据源，将Jack用户赋予root用户和管理员的角色存储在Realm容器中
        accountRealm.addAccount("Jack", "123","root","admin");
        //构建环境
        defaultSecurityManager.setRealm(accountRealm);
    }


    @Test
    public void testAuthentication() {

        SecurityUtils.setSecurityManager(defaultSecurityManager);

        //当前操作主体，主要是调用login方法
        Subject subject = SecurityUtils.getSubject();

        //用户输入的账号密码
        UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken("Jack", "123");
        //将用户输入数据存储在subject中
        subject.login(usernamePasswordToken);

        System.out.println(" 认证结果:" + subject.isAuthenticated()); //底层会调用认证器与Realm中的数据进行比对，没有会报错
        System.out.println(" 此用户是否有对应的root角色:"+subject.hasRole("root")); //true
        System.out.println(" 此用户是否有对应的管理员角色:"+subject.hasRole("admin")); //true
        System.out.println(" 操作主题Subject的名字是：" + subject.getPrincipal()); //Jack
        subject.logout();
        System.out.println("logout后认证结果:"+subject.isAuthenticated()); //false

    }
}
```
底层实现过程：  
![result](https://static01.imgkr.com/temp/9f82cb46272e4d96994b790b9457bf85.png) 

### Shiro的认证流程(认证在于查看此用户名及密码是否存在于该系统数据库)  
代码实现
```java
public class QuickStartTest4_2 {
    //创建Realm，存放数据域和认证相关信息
    private SimpleAccountRealm accountRealm = new SimpleAccountRealm();
    //用于构建SecurityManager环境
    private DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
    @Before
    public void init() {
        //初始化Realm中的数据源，添加定义认证授权的相关信息
        accountRealm.addAccount("jack", "456");

        //构建环境
        defaultSecurityManager.setRealm(accountRealm);
    }
    @Test
    public void testAuthentication() {

        //构建环境
        SecurityUtils.setSecurityManager(defaultSecurityManager);

        //创建操作主体
        Subject subject = SecurityUtils.getSubject();

        //用户输入的账号密码
        UsernamePasswordToken usernamePasswordToken =
                new UsernamePasswordToken("jack", "456");

        //认证，调用此方法即执行认证
        subject.login(usernamePasswordToken);

        System.out.println("认证结果:"+subject.isAuthenticated()); //true
    }
}
```
底层实现：  
![result](https://static01.imgkr.com/temp/8d022101a9ef4941a031b214da46404f.png)  

认证与授权都是通过构建SecurityManager环境，底层自动调用认证或者授权器，通过获取主体(subject)和当前登录用户(token)，进行login的验证(底层是调用从Realm处获取的数据进行比对验证的)
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
3、Realms：用于进行权限信息的验证，我们自己实现。Realm 本质上是一个特定的安全 DAO：它封装与数据源连接的细节，得到Shiro 所需的相关的数据。在配置 Shiro 的时候，你必须指定至少一种Realms(iniRealm、jdbcRealm或自实现的Realms) 来实现认证（authentication）和/或授权（authorization）。    
我们需要实现Realms的Authentication 和 Authorization所需的相关信息。其中 Authentication 是用来验证用户身份，Authorization 是授权访问控制，用于对用户进行的操作授权，证明该用户是否允许进行当前操作，如访问某个链接，某个资源文件等。  
### Shiro的认证过程  
#### 使用简单的SimpleAccountRealm的代码过程实现：  
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
#### 使用简单的SimpleAccountRealm代码过程实现：   
1、新建一个AuthenticationTest测试类：  
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
        //System.out.println("isAuthenticated:" + subject.isAuthenticated()); // 输出true
        // 判断subject是否具有admin和user两个角色权限,如没有则会报错
        subject.checkRoles("admin","user");
//        subject.checkRole("xxx"); // 报错
    }
}
```  
## Realms的三种实现方式  
### 基于ini文件实现的Realms
代码实现过程
1、创建shiro.ini文件配置用户信息及角色，角色及权限  
```ini
#定义用户及其对应的角色，格式：用户=密码, 角色1, 角色2, ...

[users]
jack = 456, user, visitor
rose = 123, visitor, damin

#定义角色的权限，格式：角色=模块名:权限名, 模块名:权限名, ...
#可以使用通配符*，表示所有权限

[roles]
#表示user角色拥有对video模块的查询和购买的权限
user = video:find,video:buy
#表示visitor角色拥有对goods模块的所有权限
visitor = goods:*
#表示admin角色拥有对所有模块的所有权限
admin = *
```    
2、认证代码
```java
   @Test
    public void testAuthentication() {

        IniRealm iniRealm = new IniRealm("classpath:shiro.ini");

        //构建SecurityManager环境
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        defaultSecurityManager.setRealm(iniRealm);
        SecurityUtils.setSecurityManager(defaultSecurityManager);

        //创建操作主体
        Subject subject = SecurityUtils.getSubject();

        //用户的账号和密码
        UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken("jack", "456");
        subject.login(usernamePasswordToken);

        System.out.println(" 认证结果:"+subject.isAuthenticated()); //true

        System.out.println(" 是否有对应的user角色:"+subject.hasRole("user")); //true

        System.out.println(" 操作主体是：" + subject.getPrincipal()); //jack

        System.out.println( "是否有video:find 权限："+ subject.isPermitted("video:find")); //true

        System.out.println( "是否有video:delete 权限："+ subject.isPermitted("video:delete")); //false
    }
```  
### 基于JDBC实现的Realms进行用户验证
users表  
![result](https://static01.imgkr.com/temp/219f2a4266a84179a06dde34b7a1670e.png)      
user_roles表  
![result](https://static01.imgkr.com/temp/f24c635e9ee243edbfeff1c27267e64d.png)    
roles_permissions表  
![result](https://static01.imgkr.com/temp/7c4e6f1a291343aca711f6d6540204de.png)  
#### 使用配置文件及JDBCRealms自带的数据库查询语句，认证时自动会调用该查询语句  
![result](https://static01.imgkr.com/temp/8f32c71775fa4d3c9f560a80551e92ef.png)  
根据上述查询语句，创建的数据表必须满足上述的查询名称,且三个表都需存在，即  
1、jdbc.ini的配置文件，使用配置文件实现datasource配置    
```ini
#声明Realm，指定realm类型
jdbcRealm=org.apache.shiro.realm.jdbc.JdbcRealm
#配置数据源
dataSource=com.alibaba.druid.pool.DruidDataSource
dataSource.driverClassName=com.mysql.cj.jdbc.Driver
dataSource.url=jdbc:mysql://localhost:3306/test?characterEncoding=UTF-8&serverTimezone=UTC&useSSL=false
dataSource.username=root
dataSource.password=234414
#指定数据源
jdbcRealm.dataSource=$dataSource
#手动开启查找权限，默认不开启
jdbcRealm.permissionsLookupEnabled=true
#指定SecurityManager的Realms实现，多个用逗号隔开
securityManager.realms=$jdbcRealm
```  
2、测试代码  
```java
public void testAuthentication() {

        Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:user.ini");

        // 1.构建SecurityManager环境
        SecurityManager securityManager = factory.getInstance();


        // 2.主体提交认证请求
        SecurityUtils.setSecurityManager(securityManager); // 设置SecurityManager环境
        Subject subject = SecurityUtils.getSubject(); // 获取当前主体

        UsernamePasswordToken token = new UsernamePasswordToken("jack", "123456");
        subject.login(token); // 登录

        // subject.isAuthenticated()方法返回一个boolean值,用于判断用户是否认证成功
        System.out.println("isAuthenticated:" + subject.isAuthenticated()); // 输出true
        // 判断subject是否具有admin和user两个角色权限,如没有则会报错
        subject.hasRole("admin");
//        subject.checkRole("xxx"); // 报错
    }
```
#### 不使用配置文件,但是用JdbcRealms默认查询语句
代码实现
```java
 @Test
    public void testAuthentication() {
        //配置数据库连接池
        DataSource dataSource = new DruidDataSource();
        ((DruidDataSource) dataSource).setDriverClassName("com.mysql.cj.jdbc.Driver");
        ((DruidDataSource) dataSource).setUrl("jdbc:mysql://localhost:3306/test?characterEncoding=UTF-8&serverTimezone=UTC&useSSL=false");
        ((DruidDataSource) dataSource).setUsername("root");
        ((DruidDataSource) dataSource).setPassword("234414");
        // 构建SecurityManager环境
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        JdbcRealm jdbcRealms = new JdbcRealm();
        jdbcRealms.setPermissionsLookupEnabled(true);
        jdbcRealms.setDataSource(dataSource);
        defaultSecurityManager.setRealm(jdbcRealms);
        SecurityUtils.setSecurityManager(defaultSecurityManager);
        //操作主体
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken("jack", "123456");
        subject.login(token); // 登录
        // subject.isAuthenticated()方法返回一个boolean值,用于判断用户是否认证成功
        System.out.println("isAuthenticated:" + subject.isAuthenticated()); // 输出true
        // 判断subject是否具有admin和user两个角色权限,如没有则会报错
        subject.hasRole("admin");
//        subject.checkRole("xxx"); // 报错
    }
```
#### 不使用配置文件及不使用jdbcRealms自带的查询语句，自己设置查询语句(这样对数据库的表和属性就没有限制了)  
代码实现
```java
   @Test
    public void testAuthentication() {
        //配置数据库连接池
        DataSource dataSource = new DruidDataSource();
        ((DruidDataSource) dataSource).setDriverClassName("com.mysql.cj.jdbc.Driver");
        ((DruidDataSource) dataSource).setUrl("jdbc:mysql://localhost:3306/test?characterEncoding=UTF-8&serverTimezone=UTC&useSSL=false");
        ((DruidDataSource) dataSource).setUsername("root");
        ((DruidDataSource) dataSource).setPassword("234414");
        // 构建SecurityManager环境
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        JdbcRealm jdbcRealms = new JdbcRealm();
        String sql = "select password from user where name = ?";
        jdbcRealms.setAuthenticationQuery(sql);//使用自己创建的查询语句进行查询
        jdbcRealms.setPermissionsLookupEnabled(true);
        jdbcRealms.setDataSource(dataSource);
        defaultSecurityManager.setRealm(jdbcRealms);
        SecurityUtils.setSecurityManager(defaultSecurityManager);
        //操作主体
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken("jack", "123456");
        subject.login(token); // 登录
        // subject.isAuthenticated()方法返回一个boolean值,用于判断用户是否认证成功
        System.out.println("isAuthenticated:" + subject.isAuthenticated()); // 输出true
        // 判断subject是否具有admin和user两个角色权限,如没有则会报错
        //subject.hasRole("admin");
//        subject.checkRole("xxx"); // 报错
    }
```
### 自定义 Realms(过程：继承AuthorizingRealm重写doGetAuthorizationInfo授权方法和doGetAuthenticationInfo认证方法)   
1、我们先在合适的包下创建一个MyRealm类，继承 Shiro 框架的 AuthorizingRealm 类，并实现默认的两个方法：  
```java
package com.demo.realm;

import org.apache.shiro.authc.*;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;

import java.util.*;

public class MyRealm extends AuthorizingRealm {

    /**
     * 模拟数据库数据，数据格式为用户名，密码
     */
    Map<String, String> userMap = new HashMap<>(16);

    {
        userMap.put("wmyskxz", "123456");
        super.setName("myRealm"); // 设置自定义Realm的名称，取什么无所谓..
    }

    /**
     * 授权过程：根据用户名获取用户的角色及权限信息。返回AuthorizationInfo
     *
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        String userName = (String) principalCollection.getPrimaryPrincipal();
        // 从数据库获取角色和权限数据
        Set<String> roles = getRolesByUserName(userName);
        //从数据库中根据用户名获取权限数据
        Set<String> permissions = getPermissionsByUserName(userName);

        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        simpleAuthorizationInfo.setStringPermissions(permissions);
        simpleAuthorizationInfo.setRoles(roles);
        return simpleAuthorizationInfo;
    }

    /**
     * 模拟从数据库中根据用户名获取权限数据
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
     * 模拟从数据库中根据用户名获取角色数据
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
     * 认证过程：根据传入的用户名查看数据库表中是否存在该信息，存在返回AuthenticationInfo，不存在返回null
     *
     * @param authenticationToken 主体传过来的认证信息，认证的目的是查看它的用户信息是否存在，即密码和账户信息
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
        SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo("wmyskxz", password, "myRealm");  //底层使用简单的认证信息类SimpleAuthenticationInfo
        return authenticationInfo;
    }

    /**
     * 模拟从数据库取凭证的过程
     *
     * @param userName
     * @return
     */
    private String getPasswordByUserName(String userName) {
        //操作应该为根据用户名从数据库中获取密码
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
### Shiro中使用 CredentialsMatcher进行验证密码是否正确  
注意：MD5加密指的是对数据库中的数据加密，将数据加密(可选加盐)后存入数据库，从数据库读出的也是解密后的数据   
1、在验证用户的方法中写如下代码：  
```java
HashedCredentialsMatcher hashedCredentialsMatcher = new 		HashedCredentialsMatcher();
//散列算法，使用MD5算法;
hashedCredentialsMatcher.setHashAlgorithmName("md5");
//散列的次数，比如散列两次，相当于 md5(md5("xxx"));
hashedCredentialsMatcher.setHashIterations(2);
//customRealm是之前创建的Realm类的引用变量（iniRealm或JdbcRealm或自定义的Realm）
customRealm.setCredentialsMatcher(HashedCredentialsMatcher);      
```  
2、如果是自己写的Realm，在重写的doGetAuthenticationInfo方法中声明使用的盐(若未使用盐则无需此步骤)：  
```java
simpleAuthenticationInfo.setCredentialsSalt(ByteSource.Util.bytes("使用的盐"));
return simpleAuthenticationInfo
```



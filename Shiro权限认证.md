项目中使用Shiro安全框架技术实现管理员、老师、学生不同角色赋予不同权限，以限制不同角色，访问不同的功能。例如老师有开课功能，管理员有管理用户的功能。
## Shiro  
Shiro安全框架是一种可以对用户进行角色分配，把权限分配给角色，不同的角色具有不同的权限。并在用户登录时进行角色验证的安全认证技术。 内置了过滤器和JdbcRealm(进行数据库的角色更新查询操作)
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

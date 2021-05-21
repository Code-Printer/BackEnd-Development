## 会话技术  
1、一次会话包含多个请求和响应(一次会话表示浏览器第一次给服务器发送请求，会话建立，直到一方断开为止)。  
2、功能：在一次会话的多次请求间共享数据。  
3、方式：保存在客户端(浏览器)会话技术：Cookie，保存在服务端会话技术：Session  
## Cookie(服务器会把Cookie对象发给用户浏览器，浏览器会存储该Cookie对象，注意是浏览器不是浏览器端)
1、概念：将数据保存在客户端，客户端有了Cookie后，每次请求都会将Cookie发送给服务器端。  
2、使用步骤：  
1、创建Cookie对象，绑定数据(键值对)  
2、客户端向服务器发送请求后，服务器向客户端发送Cookie对象。  
3、客户端收到Cookie后，每次请求，都会发送Cookie对象，服务器获取从客户端发送的Cookie对象数组Cookie[]request.getCookies();  
4、服务器得到每个Cookie对象后使用getName和getValue方法得到Cookie对象的数据。  
代码演示：  
1、(1)和(2)  
```java
public class CookieTest1 extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1. 创建Cookie对象，参数类似键值对
        Cookie cookie = new Cookie("msg", "hello");
        //2.服务器向浏览器发送Cookie
        response.addCookie(cookie);
    }
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request,response);
    }
}
```
2、(3)和(4)  
```java
public class CookieTest2 extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //3. 服务器获取浏览器发过来的Cookie对象
        Cookie[] cookies = request.getCookies();
        //4. 服务器获取Cookie对象的键值对
        for (Cookie cookie :
                cookies) {
            String name = cookie.getName();
            String value = cookie.getValue();
            System.out.println("获得的Cookie对象的值：" + name + ":" + value);
        }
    }
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request, response);
    }
}
```  
运行结果：先让浏览器请求访问ServletCookieTest1，此时服务器会返回给客户端一个Cookie对象，后面再次访问SerevletCookieTest2时，浏览器就会连获取的Cookie对象一起发送给服务器，服务器可以获取它的键值对。  
上述运行过程：  
![result](https://static01.imgkr.com/temp/b4e7adcd22d54f16a8b11a0e64150d47.png)  

### Cookie的细节  
1、服务器一次可以发送多个Cookie对象给客户端，使用response调用多次addCookie方法即可。  
2、Cookie在浏览器中保存的时间：  
(1)默认情况下，当当前浏览器关闭时，Cookie对象中的数据被销毁。  
(2)持久化存储，使用Cookie对象的setMaxAge(int seconds) (该方法中的参数为正数时(秒数)，将Cookie数据写进硬盘，时间一到，数据失效，此时间指的是Cookie对象创建后开始计时，并不是关闭浏览器后才开始计时；参数为负数，默认情况；参数为0，删除Cookie对象数据)。  
3、在Tomcat8之后Cookie可以存储中文，但特殊字符仍不支持，建议使用URL编码格式，先对需要赋值的中文数据进行编码，再进行cookie的构造方法赋值(同样的，当需要从cookie对象中取出中文数据时，需要对取出的数据进行URLEncoder解码decode，才能显示为中文数据)。(URLEncoder.encode("含中文的内容","UTF-8");)  
4、Cookie的共享问题：(1) 一个Tomcat服务器中，部署了多个web项目，这些web项目cookie的共享说明：
a. 默认情况，参数是web工程路径，只有这个工程才可以访问到，其余工程无法访问
b. 如果要共享，使用Cookie对象的setPath(String path)方法设置cookie的获取范围：可以设置参数为”/” ( /被浏览器解析得到的地址为http://ip:port/ )
(2) 不同的Tomcat服务器间cookie的共享说明：
使用Cookie对象的setDomain(String path)方法，参数设置为一级域名，则一级域名相同的不同服务器之间Cookie可共享
如：setDomain(“.baidu.com”)，则tieba.baidu.com与news.baidu.com等的cookie可共享。  
### Cookie的特点和作用  
1、客户端一旦有了Cookie以后，每次请求都会把Cookie对象发送给服务器。  
2、浏览器对单个Cookie有大小限制(4kB),对同一个域名下的总cookies的数量也有限制(20个)  
3、(1)Cookie一般用于存储少量的安全性较低的数据。
(2)<font color = "white">在不登陆的情况下，完成服务器对客户端的身份识别</font>，如没有登录百度账号的前提下打开百度，设置搜索引擎搜索时不提示，以后打开浏览器访问百度时，不会再出现搜索提示框，原 理：百度服务器将设置的Cookie信息保存到浏览器，下次访问百度时，百度服务器获取浏览器的Cookie，根据Cookie的值决定要不要显示提示框。  
### Cookie的应用实例  
需求：访问一个Servlet程序：
(1) 如果是第一次访问，提示：您好，欢迎您首次访问
(2) 如果不是第一次访问，提示：欢迎回来，您上次的访问时间是：xxxx
分析：
使用Cookie来完成，在服务器判断客户端是否有一个名为lastTime的cookie对象
(1) 有，不是第一次访问：
①在浏览器显示：欢迎回来，您上次的访问时间是：xxxx
②将现在的时间写回名为lastTime的cookie中
(2) 无，是第一次访问：
①在浏览器显示：您好，欢迎您首次访问
②将现在的时间写回名为lastTime的cookie中  
方法：使用jsp实现，让被访问的Servlet程序被访问时重定向到该jsp页面就好了。
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<%  
    Cookie[] cookies = request.getCookies();
    boolean flage = false;
    if (cookies != null && cookies.length > 0){
        for (Cookie cookie:cookies){
            if ("lastTime".equals(cookie.getName())){
                flage = true;
                String lastTime = cookie.getValue();
                lastTime = URLDecoder.decode(lastTime,"UTF-8");
                out.write("欢迎回来，您上次的访问时间是" + lastTime);
                Date date = new Date();
                SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
                String format = simpleDateFormat.format(date);
                format = URLEncoder.encode(format,"UTF-8");
                cookie.setValue(format);
                cookie.setMaxAge(60 * 60 * 24 * 30);
                response.addCookie(cookie);
                break;
            }
        }
    }
    %>
<%
            if (cookies == null || cookies.length == 0 || flage == false){
                Date date = new Date();
                SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
                String format = simpleDateFormat.format(date);
                format = URLEncoder.encode(format,"UTF-8");
                Cookie cookie = new Cookie("lastTime",format);
                cookie.setMaxAge(60 * 60 * 24 * 30);
                response.addCookie(cookie);
                out.write("您好，欢迎首次访问");
            }
    %>
</body>
</html>
```  
## Session介绍  
1、概念：Session是服务端的会话技术，在一次会话的多次请求间共享数据，将数据保存到服务器端，常用来保存用户登录的信息(用户名和密码)。  
2、常用方法：  
(1)获取HttpSession对象  
HttpSession session = request.getSession();
第一次调用是在创建Session会话，后面调用就是前面创建的Session会话对象。(这与Cookie创建(Cookie cookie = new Cookie(name,value),response.add(cookie))有本质的区别)。  
(2)HttpSession的方法：  
oid setAttribute(String name, Object value);(设置该对象的键值)  
Object getAttribute(String name);(根据名字获取值)  
void removeAttribute(String name);(根据名字删除键值)  
代码演示： 
1、SessionTest1 
```java
public class SessionDemo1 extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1. 创建Session会话
        HttpSession session = request.getSession();
        //2. 存储数据
        session.setAttribute("msg", "Hello! Session!");
    }
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request, response);
    }
}
```  
2、SessionTest2  
```java
public class SessionDemo2 extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1. 获取Session
        HttpSession session = request.getSession();
        //2. 获取数据
        Object msg = session.getAttribute("msg");
        System.out.println(msg);
    }
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request, response);
    }
}
```  

3、上述程序原理：<font color="white">Session底层是基于Cookie实现的。</font>  
![result](https://static01.imgkr.com/temp/2977225269934c18999dd20794c305ba.png)  

注意：每个Seeion都有其唯一的id号，这个id号是由服务器在创建Session对象时自动生成的，可通过getId()获取当前Session对象的id号。     
### Session的细节  
Session的销毁方式：  
(1)服务器关闭  
(2)Session对象调用invalidate()方法  
(3)Session对象不设置存活时间默认存活30分钟，可以在web.xml中配置修改默认的存活时间。  
```xml
<session-config>
    <session-timeout>30</session-timeout>
  </session-config>
```  
1、如果客户端(浏览器)关闭后重启但服务器未关闭，两次获取的Session是否是同一个？  
(1)默认情况下，不是，因为Cookie消失了，Session自然也消失了。  
(2)如果需要相同，则需要设置Cookie不会随浏览器的关闭而失效。故需要设置存活时间存储到硬盘上。  
```java
Cookie c = new Cookie("JSESSION",session.getId());
c.setMaxAge(60*60) //表示一小时  
response.addCookie(c);
``` 
2、如果客户端不关闭，而服务器端关闭，两次获取的Session对象是否是同一个？  
不是同一个，但为了保证数据不丢失，Tomcat服务器会自动完成保存Session数据的操作：  
(1)、Session的钝化：在服务器正常关闭之前，将Session对象序列化到硬盘上。
(2)、Session的活化：在服务器重新启动后，将Session文件反序列化成为内存中的Session对象。(即Session对象肯定不是同一个，但Session对象中的数据是相同的)。  
Session的特点：  
1、一次会话只有一个Session对象，Session用于存储一次会话中的多次请求数据(一般存储的是安全性较高的数据)。  
2、Session中值可以是任意类型的，任意大小的数据。  
### Session和Cookie的区别：  
(1)Session存储在服务器端，Cookie存储在客户端。  
(2)Session在一次会话中只能有一个，Cookie在一次会话中可以有多个。  
(3)Session可以存储的值是任意类型，而Cookie存储的值只能是字符串。    
(4)Session存储的数据没有大小限制，Cookie存储的数据有(4kb)。  
(5)Session数据安全，Cookie数据相对不安全。  

## Cookie实例：免用户名登录  
说明：成功登录后，关闭浏览器，重新启动浏览器，浏览器的用户名栏记住了上次登录的用户名(QQ的登录功能)。  
代码演示：  
1、login.jsp
```jsp
<body>
    <form action="http://localhost:8080/MyTest/LoginServlet" method="post">
        用户名：<input type="text" name="username" value="${cookie.username.value}"> <br>
        密码：<input type="password" name="password"> <br>
        <input type="submit" value="登录">
    </form>
</body>
```    
2、LoginServlet.java  
```java
public class LoginServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        //设置正确的用户名为周杰伦，密码是123
        if ("jay".equals(username) && "123".equals(password)) {
            Cookie cookie = new Cookie("username", username);
            cookie.setMaxAge(60 * 60 * 24 * 7); //cookie保存一周
            response.addCookie(cookie);
            System.out.println("登陆成功！");
        } else {
            System.out.println("登陆失败！");
        }
    }
}
```  
运行结果：使用正确的登录用户密码登录后，再次访问登录页面，用户名输入框会自动的填入上次的用户名。  
上述程序的运行逻辑如下图：  
![result](https://static01.imgkr.com/temp/10e56fda9bb84e44bc214d2e0beeebe9.png)  

## 谷歌浏览器查看Cookie信息  
![result](https://static01.imgkr.com/temp/8d6490772cae444cb3087e7acc2376e9.png)  

## 谷歌验证码的底层原理   
![result](https://static01.imgkr.com/temp/caa59d53311f424789b6469a0f6b6f9b.png)  

### 谷歌验证码使用代码演示(使用谷歌验证码的jar包)  
谷歌验证码使用步骤：  
1、在maven的pom.xml中配置谷歌的验证码的jar包。  
```xml
<dependency>
    <groupId>com.github.penggle</groupId>
    <artifactId>kaptcha</artifactId>
    <version>2.3.2</version>
</dependency>
```
2、在web.xml中配置谷歌验证码的映射路径  
```xml
<!--访问kaptcha.jpg就会得到一张验证码，并将此验证码的内容数据保存在Session中键名为KAPTCHA_SESSION_KEY -->
<servlet>
    <servlet-name>KaptchaServlet</servlet-name>
    <servlet-class>com.google.code.kaptcha.servlet.KaptchaServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>KaptchaServlet</servlet-name>
    <url-pattern>/kaptcha.jpg</url-pattern>
</servlet-mapping>
```  
3、html验证码数据提交页面  
```html
<body>
    <form action="http://localhost:8080/MyTest/Servlet">
        验证码：<input type="text" style="width: 80px;" name="code">
        <img src="http://localhost:8080/MyTest/kaptcha.jpg" alt="验证码没有找到"
             style="width: 100px; height: 28px;" id="code_img"> <br>
        <input type="submit" value="登录">
    </form>
</body>
```  
4、进行验证码数据验证  
```java
import static com.google.code.kaptcha.Constants.KAPTCHA_SESSION_KEY;
public class Servlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //获取Session中的验证码
        String attribute = (String) request.getSession().getAttribute(KAPTCHA_SESSION_KEY);
        //删除Session中的验证码
        request.getSession().removeAttribute(KAPTCHA_SESSION_KEY);
        //获取用户输入的验证码
        String code = request.getParameter("code");
        if (attribute.equalsIgnoreCase(code)) {
            System.out.println("验证码正确！");
        } else {
            System.out.println("验证码错误！");
        }
    }
}
```
### 点击图片切换验证码  
![result](https://static01.imgkr.com/temp/ad96e4cc1a58482f90c5c4312022ef09.png)  

代码实现跳过浏览器缓存，点击验证码进行验证码切换(下面代码需要写到同一html页面中，在script标签中写下)：  
1、第一种在同一页面的script标签中写js事件响应函数(实质就是改变img标签的src属性)  
```js
window.onload = function () {
    //window.onload表示页面加载完毕后立马执行的事件
    //通过验证码图片的id属性值绑定单击事件
    var elementById = document.getElementById("code_img");
    elementById.onclick = function () {
        //1. 事件响应的function函数中的this对象是当前正在响应事件的标签的dom对象
        //2. src属性可读可写
        this.src = "http://localhost:8080/MyTest/kaptcha.jpg?d=" + new Date();
    }
}
```  
2、既然是想改变img标签的src属性的值，那就直接通过绑定onclick属性来改写imgsrc的值。    
```html
        <img src= "http://localhost:8080/Cookie_Session/Kaptcha.jpg" alt="图片还未找到" style="width: 100px;height: 28px;" id="codeimg"  onclick="this.src=this.src+'?'+new Date();">
```

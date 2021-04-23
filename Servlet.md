## Servlet(服务程序)
Servlet、Filter(过滤器)、Listener(监听器)是javaWeb的三大组件。Servlet是运行在服务器端的java程序，可以接受客户端的请求，并返回数据给客户端。  
### 手动创建一个Servlet程序
访问页面路径为：http://localhost:8080/ServletProgram/hello。  其中http:// 表示该Web页面采用http(应用层的协议)协议，localhost表示的服务器(ip)地址，:8080表示服务器(Tomcat)的端口号，ServletProgram表示的是工程路径，hello表示的是资源路径。  

![result](https://static01.imgkr.com/temp/358c845c71e04cb0a8077d1f442b12a2.png)
web项目下web.xml文件的配置解析：
```xml
    <!-- servlet标签给Tomcat配置Servlet程序 -->
    <servlet>
        <!-- servlet-name标签给Servlet程序起一个别名（一般是类名）-->
        <servlet-name>HelloServlet</servlet-name>
        <!-- servlet-class标签是Servlet程序的全类名-->
        <servlet-class>TestServlet</servlet-class>
    </servlet>

    <!-- servlet-mapping标签给servlet程序配置访问地址-->
    <servlet-mapping>
        <!-- servlet-name标签的作用是告诉服务器，我当前配置的地址给哪个Servlet程序使用-->
        <servlet-name>HelloServlet</servlet-name>
        <!-- url-pattern标签配置访问地址
                /：斜杠在服务器解析的时候，表示地址为：http://ip:port/工程路径
                hello: 表示地址为http://ip:port/过程路径/hello
                -->
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
```
访问哪些资源是由资源路径决定的，资源路径又是由web.xml文件配置的。通过资源路径到web.xml文件中找url-pattern相对应的地址，找到以后通过servlet-name就会判断这个地址是给谁用的，进而找到对应的Servlet程序，有可能是别名(一般都是类名)，所以需要通过servlet-class找到该Servlet程序对应的全类名，即找到了对应的java类，然后java类就会执行service方法。

### Servlet的生命周期：
当Servlet程序被访问的执行过程为：构造方法-init-service-destory。  
1、首先当Servlet程序每次启动时，先执行Servlet的构造方法，然后执行init()方法初始化。  
2、当每次页面刷新或初次访问页面会执行service()方法。  
3、当停止服务器时(点击暂停按钮)，会执行destroy()方法。

### 表单提交方式get与post的区别：
1.get是从服务器上获取数据，post是向服务器传送数据。  
2.get是把参数数据队列加到提交表单的action属性所指的url中，值和表单内各个字段一一对应，在url中可以看到。post是通过HTTP的post机制，将表单内各个字段与其内容放置在HTML header中一起传送到action属性所指的url地址，用户是看不到这个过程的。  
3.对于get方式，服务器端用Request.QueryString获取变量的值，对于post方式，服务器端用Request.Form获取提交的数据。  
4.get传送的数据量较小，不能大于2KB，post传送的数据量较大，一般被默认为不受限制。但理论上，IIS4中最大量为80KB，IIS5中为100KB。  
5.get安全性非常低，post安全性较高

### GET和POST请求的不同处理
在一个Servlet程序的service方法中改写成一下代码
```java
 public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        //转换的原因：HttpServletRequest有getMethod方法，可以得到请求的类型
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
        String method = httpServletRequest.getMethod();
        //method值为GET或POST，取决于表单提交的method
        if (method.equals("POST")){
            System.out.println("POST方式");
        } else if (method.equals("GET")) {
            System.out.println("GET方式");
        }
```
然后在web文件下创建一个名字.html页面.
```html
<body>
    <form action="http://localhost:8080/MyTest/hello" method="post">
        <!--action属性值与web.xml中的<url-pattern>标签内容一致，用于访问到service方法-->
        <!--action属性表示表单要提交到那个Servlet路径 ，method表示提交的方式是什么类型的-->
        <input type="submit">
    </form>
</body>
```
结果操作：服务器启动之后，在浏览器的地址栏中的后缀加上名字.html，即可访问此页面，点击提交标签，即可跳转到http://localhost:8080/MyTest/hello，执行该Servlet的service方法，控制台输出：POST方式。

### 继承HttpServlet类实现Servlet程序
在实际的web项目开发中都是使用继承HttpServlet类实现Servlet程序的方式，步骤如下：  
1、编写一个类继承HttpServlet类。  
2、根据需求重写doGet或doPost方法，由service方法根据表单的method属性值调用二者之一。  
3、到web.xml中配置Servlet程序的访问地址。  
代码演示：  
1、在src目录下创建此类
```java
public class TestServlet2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("doGet方法执行");
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("doPost方法执行");
    }
}
//HttpServlet的service方法会根据method方式调用二者之一
```
2、在web.xml中继续写配置
```xml
<!--不用删除原来的servlet标签，在<web-app>标签中继续写servlet标签-->
<servlet>
    <servlet-name>TestServlet2</servlet-name>
    <servlet-class>com.qizegao.servlet.TestServlet2</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>TestServlet2</servlet-name>
    <url-pattern>/Test2</url-pattern>
</servlet-mapping>
```
3、在web中创建Test.html页面
```html
<body>
    <form action="http://localhost:8080/MyTest/Test2" method="post">
        <!--action属性值与web.xml中的<url-pattern>标签内容一致，用于访问到service方法-->
        <input type="submit">
    </form>
</body>
```
运行结果：运行结果：服务器启动之后，在浏览器的地址栏中的后缀加上Test.html，即可访问此页面，点击提交标签， 即可跳转到http://localhost:8080/MyTest/Test2，执行service方法，进而执行doPost方法。

### Servlet接口的继承体系
![result](https://static01.imgkr.com/temp/63972b59d4ad49fb80950214089f3b54.png)

### ServletConfig接口
1、Servlet程序和ServletConfig对象是由Tomcat负责创建，程序员负责使用的。  
2、当Servlet程序默认是第一次被访问的时候创建，当Servlet程序创建时同时创建ServletConfig对象，当前的Servlet程序只可以获得自己的ServletConfig对象。是一一对应的。

ServletConfig接口的作用：  
(1) 可以获取本Servlet程序的别名(即web.xml的标签servlet-name的内容)(对象名.getServletName())。  
(2) 可以获取web.xml的初始化参数的值,即init-param标签中的标签param-value的值(对象名.getInitParameter("web.xml文件中的param-name值"))。  
(3) 可以获取ServletContext对象(对象名.getServletContext())。  
注意：重写init方法时，必须要在函数体内写：super.init(config);原因：父类GenericServlet中的init方法将参数config保存起来，子类若不调用则无法保存。

### ServletContext接口
1、ServletContext接口表示Servlet上下文对象。  
2、一个工程只存在一个ServletContext对象实例。  
3、ServletContext是在web工程启动时创建的，在web工程关闭时销毁的。  
4、ServletContext是一个域对象。（域对象：像Map一样存取数据的对象称为域对象，域指的是存取数据的操作范围）ServletContext对象的作用域是整个web工程。

![result](https://static01.imgkr.com/temp/26a545df8db44516a180ce0d50c4a1d8.png)

ServletContext接口的作用：  
1、获取web.xml中配置的上下文参数标签中的值。  
代码演示(xml文件配置内容)
```xml
<!--<context-param>标签中是上下文参数，属于整个web工程-->
<!--可以有多个，写在第一个<servlet>标签之外(之上)-->
<context-param>
    <param-name>username</param-name>
    <param-value>root</param-value>
</context-param>
<context-param>
    <param-name>password</param-name>
    <param-value>root</param-value>
</context-param>
<!--并写出下方的类对应的<servlet标签>-->
```  
2、获取当前工程的路径，格式：/工程路径，也就是Edit Configurations中Deployment中的 Application context的内容
3、获取工程部署后在硬盘上的绝对路径。  
4、像Map一样存取数据。  
代码演示:
```java
//默认执行doGet方法，故只重写doGet方法
     protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            //GenericServlet类中有public ServletContext getServletContext()方法，return this.getServletConfig().getServletContext();
            ServletContext servletContext = getServletContext();
            System.out.println(servletContext.getInitParameter("username"));
            System.out.println(servletContext.getInitParameter("password"));
            System.out.println(servletContext.getContextPath());
            System.out.println(servletContext.getRealPath("/"));
            servletContext.setAttribute("key1","value1");
            System.out.println(servletContext.getAttribute("key1"));
        }
```
注意：一个web工程只会创建一个ServletContext对象实例，换其他类输出servletContext得到的结果与上述相同，且一旦给此对象赋值(servletContext.setAttribute("key1","value1");)，即使换另一个类getAttribute(key1)，得到的结果也是value1。

### ServletContextListener监听器
1、Listener监听器介绍：  
(1) Listener监听器是JavaWeb的三大组件之一   
(2) Listener监听器是JavaEE的规范(接口)  
(3) Listener监听器的作用是监听某件事物的变化，然后通过回调函数反馈给程序做一些处理。  

2、ServletContextListener监听器  
ServletContextListener监听器可以监听ServletContext对象的创建和销毁(web工程启动时创建，停止时销毁)，监听到创建和销毁之后都会调用ServletContextListener监听器的方法进行反馈：  
```java
public interface ServletContextListener extends EventListener {
    //在ServletContext对象创建之后调用
    public void contextInitialized(ServletContextEvent sce);
    //在ServletContext对象销毁之后调用
    public void contextDestroyed(ServletContextEvent sce);
}
```  
3、ServletContextListener监听器的使用步骤：  
(1) 编写一个类实现ServletContextListener接口  
(2) 重写两个方法(监听初始化和监听销毁的方法)  
(3) 在web.xml文件中配置监听器
代码演示：  
1、创建一个类监听器实现ServletContextListener接口并重写方法
```java
public class ListenerTest implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        System.out.println("ServletContext对象创建");
    }
    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        System.out.println("ServletContext对象销毁");
    }
}
```  
2、在web.xml中配置该监听器
```xml
<listener>
    <!-- <listener-class>标签中写上述程序的全类名 -->
    <listener-class>com.qizegao.servlet.ListenerTest</listener-class>
</listener>
```
结果：  
Tomcat服务器启动之后控制台输出ServletContext对象创建  
Tomcat服务器停止之后控制台输出ServletContext对象销毁  


## Http协议
协议：表示双方或者多方协定好的规则，Http协议指的是客户端与服务器进行数据通信所要遵守的规则，Http协议中的数据又称作报文。
### 请求的Http协议格式
请求分为get请求与post请求两种：  
<font color=white size=3 face=“宋体”>get请求：</font>  get请求由请求头和请求行组成。

![result](https://static01.imgkr.com/temp/5720e493572848859c9744dbe5861710.png)
<font color=white size=3 face=“宋体”>get的请求行：</font>①请求的方式：GET  
②请求的资源路径：/06_servlet/a.html  
③请求的协议的版本号：HTTP/1.1   

<font color=white size=3 face=“宋体”>get的请求头：</font>①Accept：告诉服务器，客户端可以接收的数据类型  
②Accept-Language：告诉服务器，客户端可以接收的语言类型：
zh_CN：中文中国  en_US：英文美国  
③User-Agent：客户端浏览器的信息  
④Accept-Encoding：告诉服务器，客户端可以接收的数据编码(压缩)格式  
⑤Host：表示想请求的服务器ip和端口号  
⑥Connection：告诉服务器，当前连接如何处理：Keep-Alive：回传完数据不要马上关闭，保持一小段时间的连接  
Closed：回传完数据马上关闭。  
 
 <font color=white size=3 face=“宋体”>post请求：</font>post请求由请求行、请求头、空行、请求体组成。

 ![result](https://static01.imgkr.com/temp/c10a667569f4448dbe4e340845585bf5.png) 
<font color=white size=3 face=“宋体”>（仅谈及与get请求不同）</font>
<font color=white size=3 face=“宋体”>请求头：</font>  
①Referer：表示请求发起时，浏览器地址栏中的地址  
②Content-Type：表示发送的数据的类型：  
i. application/x-www-form-ur lencoded：表示提交的数据的格式是 name=value&name=value，然后对其进行url编码，url编码是把非英文内容转换为：%xx%xx  
ii. multipart/form-data：表示以多段的形式提交数据给服务器，即以流的形式提交，用于上传  
③Content-Length：表示发送的数据的长度  
④Cache-Control：表示控制缓存，no-cache不缓存  

### 响应的Http协议
![result](https://static01.imgkr.com/temp/f4968264df6f403e8e2bf3fdf011ee25.png)

### 常见的http响应码
200：请求成功  
301：表示重定向，请求页面永久转移；302表示请求临时重定向。  
404：服务器接收到请求，但请求未找到(有可能是请求的语法错误)  
500：表示服务器接收到请求，但服务器内部错误。  

### MIME类型说明
MIME类型是Http协议中的数据类型，格式是：大类型/小类型，并与某一种文件的扩展名相对应：
![result](https://static01.imgkr.com/temp/c5918967c9514405862138d5e1fe2948.png)

### HttpServletRequest
HttpServletRequest的作用：当客户端发送请求到Tomcat服务器时，Tomcat服务器会把请求来的Http协议信息解析好，封装到Request对象中，然后传递到service方法中，service方法会根据请求方式，调用程序员重写的doGet方法和doPost方法。  

HttpServletRequest的常用方法：  
getRequestURI()：获取请求的资源路径  
getRequestURL()：获取请求的绝对路径  
getRemoteHost()：获取客户端的ip地址  
getHeader()：获取请求头  
getParameter()：获取请求的参数  
getParameterValues()：获取请求的参数(多个值时使用)  
getMethod()：获取请求的方式(GET或POST)  
setAttribute(key, value)：设置域数据  
getAttribute(key)：获取域数据  
getRequestDispatcher()：获取请求转发对象  

代码演示获取参数并在控制台打印：
1、创建一个Servlet程序和对应在web目录下创建(不能在WEB-INF下创建，因为会访问不到)html页面。
```html
<form action="http://localhost:8080/ServletProgram/test3" method="get">
    用户名：<input type="text" name="username"><br/> <!-- </br>换行标签-->
    密码：<input type="password" name="password"><br/>
    兴趣爱好：<input type="checkbox" name="hobby" value="C++">C++
    <input type="checkbox" name="hobby" value="Java">Java
    <input type="checkbox" name="hobby" value="JS">JavaScript<br/>
    <input type="submit">
</form>
```
2、根据html页面中对应的请求方式在对应的Servlet中的方法中重写doGet或者doPost方法
```java
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    //doPost方法会出现中文请求乱码问题
        //需要在获取任何参数之前修改字符编码集，而不仅仅获取中文参数时才修改：
        request.setCharacterEncoding("UTF-8");
        //获取请求的参数(此方法参数中放name属性值，得到value属性值)
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        //获取请求的参数的多个值
        String[] hobbies = request.getParameterValues("hobby");
        //输出
        System.out.println("用户名：" + username);
        System.out.println("密码：" + password);
        //将数组转换为集合输出
        System.out.println("兴趣爱好：" + Arrays.asList(hobbies));
    }
```
### 请求转发
请求转发指的是服务器收到请求后，从一个资源(Servlet程序)转到另一个资源的操作。  

![result](https://static01.imgkr.com/temp/7233c9d3d62d4ce29186b195e6ac791e.png)
代码演示：
1. 在src下创建此类，并在web.xml中配置相应的数据
```java
public class Servlet1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //获取请求的参数(查看办事的材料)
        String username = request.getParameter("username");
        System.out.println("在Servlet1(柜台1)中查看参数(材料):" + username);
        //给材料盖章
        request.setAttribute("key1","柜台1的章");
        //获得通向Servlet2的路径(请求转发对象)
        //参数必须以斜杠打头，斜杠代表http://localhost:8080/工程名/，对应IDEA代码的web目录
        RequestDispatcher requestDispatcher = request.getRequestDispatcher("/Servlet2");
        //可以转发到WEB-INF目录下：request.getRequestDispatcher("/WEB-INF/xxx");
        //通过得到的路径走向Servlet2(柜台2)
        //forward方法将当前资源的request和response转发到该requestDispatcher指定的资源
        requestDispatcher.forward(request, response);
        //使得Servlet2中的request和response与Servlet1一致
    }
}
```
2、在src下创建此类，并在web.xml中配置相应的数据
```java
public class Servlet2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //获取请求的参数(查看办事的材料)
        String username = request.getParameter("username");
        System.out.println("在Servlet2(柜台2)中查看参数(材料):" + username);
        //查看是否有柜台1的章
        Object key1 = request.getAttribute("key1");
        System.out.println("柜台1的章为：" + key1);
        //出处理自己的业务
        System.out.println("Servlet2处理业务");
    }
}
```
在浏览器的地址栏中输入访问Servlet1的页面路径即可显示结果在控制台。可以得出地址栏的内容不发生变化，但页面自动跳转(访问)到了请求转发对象Servlet2中(即显示http://localhost:8080/MyTest/Servlet2的页面)  

### base标签(html中的标签)的作用(实际开发中的跳转路径采用的是绝对路径)
base标签可以设置当前页面中所有相对路径跳转时参照指定的路径来进行跳转，在href属性中设置指定路径。  
代码演示：
1、在web目录下创建a文件夹下创建b文件夹下创建c.html
```html
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    这是a下的b下的c.html<br/>
    <a href="../../index.html">跳到web下的index.html</a>
</body>
```
2、 在src下创建此类，并在web.xml中配置
```java
public class Forward extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        request.getRequestDispatcher("a/b/c.html").forward(request,response);
    }
}
```
3、在web目录下创建index.html
```html
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    这是web下的index.html页面<br/>
    <a href= http://localhost:8080/MyTest/Forward>请求转发：a/b/c.html</a>
</body>
```
在地址栏输入http://localhost:63342/FirstWeb/MyTest/web/index.html，点击后成功跳转到 http://localhost:8080/MyTest/Forward ，显示的是c.html
界面，点击跳转连接无法跳转，这是因为相对路径跳转时会参考当前浏览器地址栏的地址，而当前浏览器的地址栏地址为http://localhost:8080/MyTest/Forward 当c.html中的跳转路径../../index.html抵消后(抵消操作为一个../可以抵消当前路径的的一个文件夹)的真实跳转路径是http://localhost:8080/…/index.html ，显然这个路径是不对的，故不能跳转成功。
4、此时是由于c.html中的跳转链接参考的地址有问题，故可以修改c.html文件添加使用base标签，改变其跳转时参考的地址。
```html
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <!--base标签写在<title>标签之后-->
    <base href="http://localhost:8080/MyTest/a/b/">
</head>
<body>
    这是a下的b下的c.html<br/>
    <a href="../../index.html">跳到web下的index.html</a>
</body>
```

### HttpServletResponse类
每次只要有请求，Tomcat就会创建一个Response对象给Servlet对象，HttpServletResponse表示所有的响应信息(HttpServletRequest表示所有请求发来的信息)，可以通过HttpServletResponse对象设置返回给客户端的信息。  

HttpServletResponse类中两个输出流方法的说明:  
getOutputStream():字节输出流，常用与下载二进制数据。
getWriter()：字符输出流，常用于回传字符串。  
注：同一个HttpServletResponse对象两个流不可同时使用。  

代码演示：从服务器向客户端回传字符串数据  
在src下创建此类并在web.xml中进行配置   
```java
public class ResponseIO extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //防止中文乱码问题，在获取流对象之前调用此方法：
        //同时设置服务器和客户端都使用UTF-8字符集
        response.setContentType("text/html; charset=UTF-8");
        //获取流对象
        PrintWriter writer = response.getWriter();
        writer.write("I Love China！");
    }
}
```
### 请求重定向
请求重定向指的是客户端给服务器发送请求数据，服务器收到后会通知客户端去访问自己新的地址。  

![result](https://static01.imgkr.com/temp/7c5dc2f01ba047ee8398b6ccba2d2359.png)
代码演示：
1. 在src下创建此类并在web.xml中进行配置
```java
public class Response1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("会访问到Response1");
        //1.设置响应状态码302
        response.setStatus(302);
        //2.设置响应头说明新的地址在哪里
        response.setHeader("Location","http://localhost:8080/MyTest/Response2");
    }
}
```
2. 在src下创建此类并在web.xml中进行配置
```java
public class Response2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.getWriter().write("Response2's result!");
    }
}
```
注：
在Response1中使用以下代码替代两个步骤可得到同样的效果(推荐使用此方法)：  
1、response.sendRedirect(“http://localhost:8080/MyTest/Response2”);设置重定向的路径  
2、在Response1中request.setAttribute(“key1”, “value1”);在Response2中req.getAttribute(“key1”); 无法得到key1的值，结果为null
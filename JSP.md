## JSP(java server page java服务器页面)
JSP与HTML的区别：
HTML是静态页面（博客）开发技术，一般是已经写好内容放在服务器，由服务器将HTML发送给客户端。而JSP是动态页面(B站首页)开发技术(一般是用于用户界面部分)，是由JSP容器执行该页面的java部分实时生成的动态HTML(页面)。   

JSP与ASP的区别：1、页面的动态部分由java编写，不像ASP由VB(用户界面专用语言)或其他MS(微软系统)专用语言编写，更加强大易用。2、JSP易于移植到非MS(Microsoft System)平台。  

JSP与Servlet的区别：JSP可以很方便的编写或者修改HTML网页而不用面对很多的println语句。  

JSP与JavaScript的区别:虽然JavaScript可以在客户端动态生成HTML页面，但很难与服务器交互，例如访问数据库，因此难以实现复杂的操作。

JSP的作用主要是代替Servlet程序回传HTML页面的数据（后台服务器会解析JSP程序然后生成动态的html文件回传给客户端）。JSP可以通过网页提交的表单获取用户输入、访问数据库及其他数据源，然后动态的创建网页。  

### JSP的本质
JSP实质上是一个Servlet程序，当第一次访问JSP页面时，Tomcat会把JSP页面翻译成一个.java文件，然后编译生成.class文件。当打开.java文件会看到该类继承于一个HttpJspBase类，而该类继承自HttpServlet。

查看翻译后的Java源文件的方法：启动Tomcat服务器访问到JSP页面之后在控制台输出的信息的前端找到Using CATALINA_BASE中的路径，在硬盘中打开此目录，点击work --> Catalina --> localhost，找到对应的工程文件夹寻找即可
访问JSP页面其实是在执行对应的翻译后的Java代码的_jspService方法：翻译后的Java类中没有service方法，而是重写了父类的_jspService方法，这个方法会被父类的service方法调用

### JSP语法
JSP头部的page指令：
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
```
JSP头部的page指令可以修改JSP页面中的一些重要属性或行为
(以下属性均写在page指令中，默认page指令中没有出现的属性都采用默认值)：
(1) contentType属性：表示JSP返回的数据类型是什么，即response.setContentType()的参数值  
(2) language属性：表示JSP翻译之后是什么语言文件(目前只支持Java)  
(3) pageEncoding属性：表示当前JSP文件本身的字符集(可在IDEA右下角看到)  
(4) import属性：表示导包(导类)，与Java一致  
(5) autoFlush属性：设置当out输出流缓冲区满了之后是否自动刷新缓冲区，默认值是true  
(6) buffer属性：设置out缓冲区的大小，默认是8kb
注意：out缓冲区满了之后不能自动刷新的话会报错  
(7) errorPage属性：设置当JSP页面运行出错时自动跳转到的页面(错误信息页面)的路径，这个 路径一般都是以斜杠打头，表示请求的地址是http://ip:port/工程路径/，对应代码web目录  
(8) isErrorPage属性：设置当前JSP页面是否是错误信息页面，默认是false，如果是true可以 获取错误信息  
(9) session属性：设置访问当前JSP页面时是否会创建HttpSession对象，默认值是true  
(10) extends属性：设置JSP页面翻译出来的Java类默认继承谁  
注意：以上默认值除非有特殊需要，否则不建议修改  

### JSP中常用的脚本：
1、声明脚本：  
格式：<%! 该类中的成员属性及方法的声明语句 %>  
作用：可以给JSP翻译出Java类定义属性、方法、静态代码块、内部类等。  
特点：不会在浏览器的页面上显示出来，仅存在于翻译后的Java类中。  
代码演示：  
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page import="java.util.HashMap" %>
<%@ page import="java.util.Map" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <%--1.声明类属性--%>
    <%!
        private String name;
        private static Map<String, Object> map;
    %>
    <%--2.声明类方法--%>
    <%!
        public int sum() {
            return 12;
        }
    %>
    <%--3.声明静态代码块--%>
    <%!
        static {
            map = new HashMap<String, Object>();
            map.put("key1", "value1");
        }
    %>
</body>
</html>
```  
对应的翻译后的java源文件：  
![result](https://static01.imgkr.com/temp/44788c8163384018907930187333b99b.png)   

2、表达式脚本：  
格式：<%=表达式语句 %>  
作用：在浏览器的JSP页面上输出数据(只有此脚本可以在浏览器的页面上输出数据)。  
特点：(1) 所有的表达式脚本都会被翻译到对应的Java类的_jspService()方法中，故表达式脚本可以直接使用_jspService()方法参数中的对象。
(2) 表达式脚本都会被编译后的Java类中的out.print()方法输出到浏览器页面上。
(3) 表达式脚本中的表达式不能以分号结束。  
代码演示：继续在上面的JSP文件中写入  
```jsp
<%=22 %> <br/>
<%="可以输出字符串" %> <br/>
<%=map %> <br/>
<%--使用_jspService方法中的对象--%>
<%=request.getParameter("username") %>
```
运行程序，在浏览器中输入地址，结果如下：  
![result](https://static01.imgkr.com/temp/4686bcd8c8fd413684841d96c5de34de.png)  
对应的翻译后的Java源文件(在_jspService方法中)：  
![result](https://static01.imgkr.com/temp/79e6d6e618ed460dbc3df55bc769d991.png)  
注意：(1)、write方法中的标签、转义字符自动识别为对应的功能，不在页面输出，执行各自代表的功能。  
(2)、out的两个方法也在_jspService方法中，也都是java语言。  
(3)、只有print、write方法、表达式脚本中的内容才可在浏览器中显示，其余Java代码的输出在控制台输出。  

3、代码脚本  
格式：<% java语句 %>  
作用：在JSP页面中可以编写需要的Java代码  
特点：
(1) 代码脚本翻译后都在_jspService方法中，故代码脚本可以直接使用此方法参数中的对象  
(2) 可以由多个代码脚本块组合完成一个完整的Java语句  
(3) 代码脚本还可以和表达式脚本一起组合使用，在JSP页面上输出数据  
代码演示：继续在上面的JSP文件内写入  
```JSP
<%--1.if语句--%>
<%
    int i = 1;
    if (i == 1) {
        System.out.println("我爱祖国！");
    } else {
        System.out.println("我很爱祖国！");
    }
%> <br/>
<%--2.for循环语句--%>
<%
    for (int j = 0 ; j < 3; j++) {
        System.out.println("第" + j + "次循环");
    }
%> <br/>
<%--3.使用_jspService方法参数中的对象--%>
<%
    String username = request.getParameter("username");
    System.out.println("username对应的值为：" + username);
%>
```
运行结果在控制台显示：  
![result](https://static01.imgkr.com/temp/ef99829974744ed4b5bec1241606d39f.png)  

### JSP的三种注释
1、HTML注释：<!--HTML注释-->  
HTML注释会被翻译到JSP文件对应的Java类_jspService方法中，以out.write()输出到客户端，write方法会自动识别标签，执行标签对应的功能，不会在浏览器的页面上输出注释。  
2、Java注释：(1) //单行注释 (2) /*多行注释*/  
Java注释要写在声明脚本和代码脚本中才被认为是Java注释，会被翻译到JSP文件对应的Java类的_jspService方法中，在对应的Java类中也是注释   
3、JSP注释：<%--这是JSP注释--%>  
JSP注释中的内容不会在JSP文件翻译后的Java类中出现，即注释中的内容没有任何功能  

### JSP的九大内置对象
JSP的内置对象指的是Tomcat服务器将JSP页面翻译为Java类之后内部提供的九大对象：
(将page指令的isErrorPage属性写成true可以出现exception对象)  
![result](https://static01.imgkr.com/temp/aeb842002ddb49d3a39bf52a233e54ef.png)  
request：请求对象  
response：响应对象  
pageContext：JSP的上下文对象  
session：会话对象  
application：ServletContext对象  
config：ServletConfig对象  
out：JSP输出流对象  
page：指向当前JSP的对象  
exception：异常对象  

### JSP的四大域对象

![result](https://static01.imgkr.com/temp/40f45cbe5bed4532a14ce93350470f9c.png)  
域对象是和Map一样可以存取键值对数据的对象，上述四个对象的都是域对象，不同的在于对数据存取的范围不同。  
代码演示：四个域对象存取数据的范围的不同(在web目录下创建scope1.jsp)  
```JSP
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>scope1</title>
</head>
<body>
    <h1>scope1.jsp页面</h1>
    <%
        //向四个域对象中分别保存数据
        pageContext.setAttribute("key", "pageContext");
        request.setAttribute("key", "request");
        session.setAttribute("key", "session");
        application.setAttribute("key", "application");
    %>
    <%-- <jsp:forward page=""></jsp:forward>是请求转发标签，
         page属性设置请求转发的路径 --%>
    <jsp:forward page="/scope2.jsp"></jsp:forward>
</body>
```
2：在web目录下创建scope2.jsp  
```JSP
<head>
    <title>Title</title>
</head>
<body>
    <h1>scope2.jsp页面</h1>
    <%-- JSP页面中不加任何标签直接输入的内容被write方法输出在浏览器的页面上   --%>
    pageContext域是否有值：<%=pageContext.getAttribute("key")%> <br>
    request域是否有值：<%=request.getAttribute("key")%> <br>
    session域是否有值：<%=session.getAttribute("key")%> <br>
    application域是否有值：<%=application.getAttribute("key")%> <br>
</body>
```
运行结果1：当访问scope.jsp页面时，会先访问scope1页面然后由于设置了转发功能，就会跳转到scope2页面，但浏览器的地址没变，这时候显示scope2的页面内容，只有pageCoontext对象显示的值为null，其他域对象都在存取范围内。  
运行结果2：当再直接访问scope2页面路径时，由于又超出请求域对象的存取范围，所以pageContext对象与request对象的值都显示null，其他两个对象都在存取范围内。  
注意：若四个域对象在使用时范围都可满足要求，则使用的优先顺序是(范围从小到大)：pageContext --> request --> session --> application。  
### JSP中的out和response.getWriter.writer输出的异同
相同点：都是输出内容给客户端。
不同点：

![result](https://static01.imgkr.com/temp/2333f56144cf418f9ea9c700edd81654.png)  
注意：由于官方的代码中翻译后的Java代码底层都是使用out进行输出，故一般都使用out进行 输出，out又分为write方法和print方法：  
(1) out.print()：会将任何内容转换成字符串后调用write方法输出  
(2) out.write()：输出字符串没有问题，但输出int型时会将int转换成char输出，导致输出的并非是想要的数字而是数字对应的ASCII码  
结论：<font color=white size=3 face=“宋体”>JSP页面的代码脚本中任何要输出在浏览器的内容均使用out.print()方法</font>  
### JSP常用标签
1、静态包含(对被包含的JSP文件进行copy，但不编译该文件)  
格式：<%@include file=""%>   
其中file属性设置要包含的JSP页面，以/打头，代表http://ip:port/工程路径/，对应web目录  
(2)、使用的场景：

![result](https://static01.imgkr.com/temp/009f3836e2d24dc68009e31c1cb5040c.png)  
代码演示：(1)：在web目录下创建body.jsp
```JSP
<body>
    头部信息 <br>
    主体信息 <br>
    <%@include file="/foot.jsp"%>
</body>
```
(2)、在web目录下创建foot.jsp  
```JSP
<body>
    页脚信息 <br>
</body>
```
运行结果：当访问body.jsp时，会按行输出头部信息、主体信息、页脚信息。原因在于在body.jsp中使用了静态包含，包含了foot.jsp，所以body.jsp会拷贝foot.jsp页面内容。  
(3)、静态包含的特点：  
①静态包含不会将被包含的JSP页面翻译成.java.class文件  
②静态包含是把被包含的页面的代码拷贝到body.jsp对应的Java文件的对应位置执行。  

2、动态包含(对被包含的jsp页面进行编译并执行)  
格式：<jsp:include page=””></jsp:include>  
其中page属性设置要包含的JSP页面，与静态包含一致  
(2)动态包含的特点：  
①动态包含将被包含的JSP页面翻译成.java.class文件  
②动态包含还可以传递参数  
③动态包含底层使用如下代码调用被包含的JSP页面执行输出：
org.apache.jasper.runtime.JspRuntimeLibrary.include(request, response, “/foot.jsp”, out, false);  
代码演示：(1)：在web目录下创建body.jsp 
```JSP
<body>
    头部信息 <br>
    主体信息 <br>
    <jsp:include page="/foot.jsp">
        <jsp:param name="username" value="Jaychou"/>
        <jsp:param name="password" value="root"/>
    </jsp:include>
</body>
```
(2)、在web目录下创建foot.jsp
```JSP
<body>
    页脚信息 <br>
    <%=request.getParameter("username")%>
</body>
```
 
运行结果：按行输出头部信息、主体信息、页脚信息还有参数username的value值。    
注意：设置请求参数的标签要写在body.jsp的动态包含之中  
出现Expecting “jsp:param” standard action with “name” and “value” attributes异常，两个原因：  
①动态包含中未设置参数但没有把<jsp:include page=””></jsp:include>放在一行上  
②动态包含中加了注释 


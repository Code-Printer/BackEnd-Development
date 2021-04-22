## JSP(java server page java服务器页面)
JSP与HTML的区别：
HTML是静态页面（博客）开发技术，一般是已经写好内容放在服务器，由服务器将HTML发送给客户端。而JSP是动态页面(B站首页)开发技术(一般是用于用户界面部分)，是由JSP容器执行该页面的java部分实时生成的动态HTML(页面)。   

JSP与ASP的区别：1、页面的动态部分由java编写，不像ASP由VB(用户界面专用语言)或其他MS(微软系统)专用语言编写，更加强大易用。2、JSP易于移植到非MS(Microsoft System)平台。  

JSP与Servlet的区别：JSP可以很方便的编写或者修改HTML网页而不用面对很多的println语句。  

JSP与JavaScript的区别:虽然JavaScript可以在客户端动态生成HTML页面，但很难与服务器交互，例如访问数据库，因此难以实现复杂的操作。

JSP的作用主要是代替Servlet程序回传HTML页面的数据（后台服务器会解析JSP程序然后生成动态的html文件回传给客户端）。JSP可以通过网页提交的表单获取用户输入、访问数据库及其他数据源，然后动态的创建网页。  

### JSP的本质
JSP实质上是一个Servlet程序，当第一次访问JSP页面时，Tomcat会把JSP页面翻译成一个.java文件，然后编译生成.class文件。当打开.java文件会看到该类继承于一个HttpJspBase类，而该类继承自HttpServlet。

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
```jsp
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
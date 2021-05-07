## 文件的上传和下载  
### 一、点击图片，进行文件的上传  
1、要有一个form标签表单，请求方式method=post，因为文件的长度一般会超过get请求的限制。  
2、form标签的encType属性必须为multipart/form-data。(表示提交的数据以多段(一个表单项(form中的input标签)为一段)的形式进行拼接，以二进制流的方式发送给服务器。)  
3、在form标签中使用input标签，type = file表示添加上传的文件功能。  
4、在form标签中使用input标签，type=submit提交数据到服务器功能  
5、编写Servlet程序接收、处理上传的文件  
代码演示：  
1、input标签的背景样式fileInputContainer和input标签的透明化样式fileInput  
```css
.fileInputContainer{
    height:49px;
    background:url(../image/select-1.png);
    position:relative;
    width: 64px;
}
.fileInput{
    height:49px;
    overflow: hidden;
    font-size: 64px;
    position:absolute;
    right:0;
    top:0;
    opacity: 0;
    filter:alpha(opacity=0);
    cursor:pointer;
}
```  
2、WEB网页相应功能的html样式  
```html
<html>
<head>
    <title>Title</title>
    <link rel="stylesheet" href="css/imageSelect.css" />
</head>
<body>
<form action="http://localhost:8080/Upload_DownloadTest/upload" method="post" enctype="multipart/form-data">
    用户名：<input type="text" name="username" /> <br/>
    <div class="fileInputContainer">
        <input class="fileInput" id="" type="file" name="">
    </div>
    <input type="submit" value="上传" >
</form>
</body>
</html>
```  
3、编写Servlet程序处理上传文件的请求响应  
```java
public class upLoadServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request,
           HttpServletResponse response) throws ServletException, IOException {
        System.out.println("上传成功了！");
    }
}
```  
4、在xml文件中配置Servlet映射   
```xml
<servlet>
    <servlet-name>UploadServlet</servlet-name>
    <servlet-class>com.mrgao.WebfileTest.UploadServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>UploadServlet</servlet-name>
    <url-pattern>/upload</url-pattern>
</servlet-mapping>
```  
文件上传是发送的HTTP协议内容(谷歌浏览器中上传的文件的数据显示的是空行，但服务器可以接收到数据)：  
![result](https://static01.imgkr.com/temp/ae83640dda224a74a4dba1725e03fa16.png)  

### 二、服务器对上传文件数据进行解析保存  
1、首先导包，导入commons-fileupload.jar包和commons-io.jar  
2、使用两个包中的类和方法：  
(1) ServletFileUpload类，用于解析上传的数据  
①public static final boolean isMultipartContent(HttpServletRequest request)如果上传的数据是多段的形式，返回true，只有多段的数据才是文件上传的  
②public ServletFileUpload()
空参构造器  
③public ServletFileUpload(FileItemFactory fileItemFactory)
参数为工厂实现类的构造器  
④public List parseRequest(HttpServletRequest request)
解析上传的数据，返回包含每一个表单项的List集合  
(2) FileItem类，表示每一个表单项  
①public boolean isFormField()
如果当前表单项是普通表单项，返回true，文件类型返回false  
②public String getFieldName()
获取当前表单项的name属性值  
③public String getString()
获取当前表单项的value属性值，参数为”UTF-8”可解决乱码问题  
④public String getName()
获取上传的文件名  
⑤public void write(File file)
将上传的文件写到参数File所指向的硬盘位置  
代码演示：
1、在Servlet中重写doPost方法。  
```java
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        if (ServletFileUpload.isMultipartContent(req)){
            FileItemFactory fileItemFactory = new DiskFileItemFactory();//创建工厂实现类(FileItemFactory是一个接口，需要new其实现类)
            ServletFileUpload servletFileUpload = new ServletFileUpload(fileItemFactory);//创建用于解析上传数据的工具类ServletFileUpload类
            servletFileUpload.setHeaderEncoding("UTF-8"); //在不做任何设置的情况下利用FileUpload相关类解析的的文件，文件名会出现中文乱码问题，应该重新设置编码：servletFileUpload.setHeaderEncoding("UTF-8");
            try{
                List<FileItem> list = servletFileUpload.parseRequest(req);//解析上传的数据，得到每一个表单项FileItem
                for (FileItem fileItem:list){
                    if (fileItem.isFormField()){ //普通表单项
                        System.out.println(fileItem.getFieldName() + fileItem.getString("UTF-8"));//输出name和value值
                    }else{//上传的是文件类型
                        System.out.println(fileItem.getFieldName());//输出文件名
                        fileItem.write(new File("D:\\" + fileItem.getName()));//将文件写入服务器磁盘指定的位置
                    }
                }
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
```  
### 三、客户端下载服务端的文件  
1、下载的主要过程：  
(1)获取文件File对象  
(2)判断该文件对象是否存在  
(3)将获取的文件类型告诉客户端  
(4)告诉客户端收到的数据用于下载  
(5)获取要下载的文件并回传给客户端  
2、文件下载的详细过程  
(1)、获取要下载的文件路径形成File对象,判断该对象是否存在(file.exists())  
(2)、获取要下载的文件类型：
通过ServletContext的getMimeType(); 参数是要下载的文件所在路径，返回值是String类型  
(3)、将获取的文件类型告诉客户端：
通过HttpServletResponse的setContentType(); 参数是第二步的结果，无返回值  
(4)、告诉客户端收到的数据用于下载使用(没有此步则内容直接显示在页面上)：
通过HttpServletResponse的setHeader();
参数是"Content-Disposition", “attachment; fileName=xxx.xxx”
注意：Content-Disposition响应头表示客户端收到的数据如何处理
attachment表示附件，用于下载
filename表示下载的文件名，可以与原文件名不同
此方法无返回值  
(5)、获取要下载的文件并回传给客户端：使用传统的输入输出流写入的方法。  
代码演示:
```java
public class DownLoad extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        request.setCharacterEncoding("utf-8");
        response.setCharacterEncoding("utf-8");

//        //获取文件下载路径
        String path = "D:\\java Code\\Upload&DownloadTest\\web\\File\\";
        String filename = "java框架.docx";
        File file = new File(path + filename);
        if(file.exists()){
            
            ServletContext servletContext = getServletContext();
            String mimeType = servletContext.getMimeType(path + filename);//获取要下载的文件类型：
            response.setContentType(mimeType);将获取的文件类型告诉客户端
            //设置头信息
            if (request.getHeader("User-Agent").contains("Firefox")) {
                //火狐浏览器的解决方式
                response.setHeader("Content-Disposition","attachment; filename==?UTF-8?B?"
                        + new BASE64Encoder().encode(filename.getBytes("UTF-8")) + "?=");
            } else {
                //如果是IE浏览器或谷歌浏览器
                response.setHeader("Content-Disposition","attachment; filename="
                        + URLEncoder.encode(filename, "UTF-8"));
            }
            InputStream inputStream = new FileInputStream(file);
            ServletOutputStream ouputStream = response.getOutputStream();
            byte b[] = new byte[1024];
            int n ;
            while((n = inputStream.read(b)) != -1){
                ouputStream.write(b,0,n);
            }
            //关闭流
            ouputStream.close();
            inputStream.close();
        }else{
            System.out.println("失败");
            request.setAttribute("errorResult", "文件不存在,下载失败!");
            RequestDispatcher dispatcher = request.getRequestDispatcher("index.jsp");
            dispatcher.forward(request, response);
        }
    }

}
```   
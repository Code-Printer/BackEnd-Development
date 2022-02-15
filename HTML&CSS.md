## HTML&CSS
### HTML(超文本标记语言)
#### 网页的组成部分
1、内容：网页中可以看到的数据，使用HTML技术实现    
2、表现：内容在页面上的展示形式，如布局、颜色等，使用CSS技术实现  
3、行为：页面中元素与输入设备(鼠标键盘)交互的响应，使用javascript 技术实现  
#### HTML初始化文件解析
1、html中的注释语法:<!--注释内容-->  
2、HTML文件最基本的要素：  
```html
<!DOCTYPE html> 
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    </head>
<body>

</body>
</html>
```
第1行：<!DOCTYPE>表示告诉当前浏览器使用的是什么版本的html，html表示html文件的开始。  
第2行：告诉搜索引擎当前页面是html页面，lang意思就是“language”语言的意思，en表明当前页面为英文页面，zh表明当前页面为中文页面  
第3行：头部信息，一般包括title标签、css样式、js代码  
第4行：表示当前页面使用UTF-8编码字符集  
第5行：浏览器标签页的标题  
第6行：表示头部信息结束
第7行：标签中的内容是整个页面显示的主体内容  
第9行：主体内容的结束  
第10行：整个html的结束  
#### HTML标签介绍
标签的格式：1、双标签:<标签名></标签名>  2、单标签:<标签名/>    
注意：标签是不区分大小写的  
标签的两大属性：  
1、基本属性：可以简单修改自己的样式效果  
2、事件属性：可以设置事件响应后的代码  
标签语法注意事项:  
1、标签一定要关闭，无论是单标签还是双标签。    
2、属性必须有值，且必须加双引号。
```html
<font color="blue">早安</font>
```  
代码演示：  
```html
<!DOCTYPE html>
<html lang="zh_CN">
<head>
    <meta charset="UTF-8">
    <title>我的第一个网页</title>
</head>
<body bgcolor="#deb887"> <!-- bgcolor是页面的背景颜色属性 -->
    hello!这是我的第一个网页！
    <hr/> <!--换行加水平线-->
    有一个按钮
    <br/>
    <button onclick="alert('提示！')">按钮</button>
    <!-- onclick表示点击事件，alert()是javaScript语言提供的一个警告框函数，
         它可以接收任意参数，参数就是警告框内出现的信息 -->
</body>
</html>
```
结果： 点击按钮之后出现含有“提示”的警告框  
![result](https://static01.imgkr.com/temp/d1bdb22f79d54084a37c438c17d41673.png)  

#### 常用标签
##### 1、font标签  
作用：可以用来修改显示文本文字的字体、颜色、大小。  
常用属性：  
1、face属性：设置文字字体  
2、color属性：设置文字颜色  
3、size属性：设置文字的大小  
代码演示:网页显示：我是字体标签，并修改字体为宋体，颜色为红色，字体大小为8号。  
```html
<!DOCTYPE html>
<html lang="zh_CN">
<head>
    <meta charset="UTF-8">
    <title>字体标签的测试</title>
</head>
<body>
    <font face="宋体" , color="red", size="8">我是字体标签</font>
</body>
</html>
```  
##### 2、tr、td标签  
tr、td和th标签是在table标签中使用的，tr表示行，td表示列但文本显示为普通文本，th表示列，显示文本为加粗，一般表示表的头的文本内容
##### 3、特殊字符(不是标签)  
![result](https://static01.imgkr.com/temp/7120ff5449f84094aae065b43a2cc139.png)   

代码演示：把输出行标签<br>显示在页面中，并在标签前后加一个空格  
```html
<!DOCTYPE html>				    
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>特殊字符的使用</title>
</head>
<body>							
    &lt;&nbsp;br&nbsp;&gt;
</body>
</html>
```

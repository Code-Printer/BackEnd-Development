# JavaScript
## javaScript中需要注意的点
### 变量(variable)  
将变量理解为容器，将数据放在容器中，想要获取数据时从相应的容器中获取。  
### 关于类中属性和方法的定义  
属性表示的是该事物的特征，故一般用名词  
方法表示的是该事物的功能和行为，故一般用动词
### js的数据类型  
js数据类型分成两类：一类是简单数据类型(Number、String、Boolean、Null、Undefined)，一类是复杂数据类型(Object)。js中声明一个变量但没有赋值默认会赋值undefined。(一个整数+undefined=NaN(Not a Number不是一个数字)；一个整数+null=一个整数)  
### js中的数组  
1、创建数组：var 数组名 = new Array([n]);var 数组名=[];  
2、数组中可以存放任意类型的数据：var 数组名=['17.5',true,17.5];  
3、访问数组数据越界时，会返回undefined。      
### js定义函数的参数  
1、当不知道该函数会传入多少参数时，可用arguments来获取，js中的arguments实际上是一个内置对象，存储了传入的所有参数  
```js
function getMax(){
    var max = arguments[0];
    for(var i = 1;i < arguments.length;i++){
        if(arguments[i] > max){
            max = arguments[i];
        }
    }
    return max;
}

var result = getMax(1,33,77,55,1,2);
```  
### 变量的作用域  
内部域(函数)可以访问外部域(函数)的变量。

### 利用构造函数进行对象的定义和创建  
new对象的过程：
1、在构造代码执行开始前，先创建一个空对象  
2、修改this指针的指向，将this指针指向这个对象  
3、执行构造函数内的代码，给这个对象添加属性和方法  
4、在函数完成之后返回该对象(所以构造函数中不需要return关键字)  
```js
function 构造函数名(参数1，参数2){
    this.属性名1 = 参数1;
    this.属性名2 = 参数2;
    this.方法名=function() {

    };
}

var obj = new 构造函数名(参数1，参数2);
obj.方法名();
```   
### 方法中的传参  
1、当形参数据类型为基本数据类型时，形参传递的数据实际上是实参变量复制的值，因此方法中的参数数据改变不影响实参，除非方法有return并赋值(a = function(a))。  
2、当形参数据类型为复杂数据类型时，形参传递的数据实际上是实参变量复制的值，也就是对象的地址，故方法中改变数据会改变实参中地址所指向的对象内容。  

## JavaScript入门  
### js的关系运算符  
1、等于，符号：==，简单的做字面值的比较  
2、全等于，符号：===，除了做字面值的比较之外，还会比较两个变量的数据类型是否一致  
### js自定义对象  
var 对象名 = new Object()；//创建了一个空对象  
对象名.属性名 = 值；//给此对象定义属性  
对象名.函数名 = function() {} //给此对象定义函数  
对象名.属性名/函数名()；//对象的访问  
例：  
```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
 <script type="text/javascript">
     //对象的创建
     var obj = new Object();
     obj.name = "周杰伦";
     obj.age = 18;
     obj.fun = function () {
         alert("姓名：" + this.name + "，年龄：" + this.age);
     }
     //对象的访问
     obj.fun(); //姓名：周杰伦，年龄：18
 </script>
</body>
</html>
```  
### js常用的事件  
1、onload：页面加载完成事件，浏览器解析完页面之后就会自动触发的事件  
2、onclick：单击事件，常用于按钮的点击响应操作  
3、onblur：失去焦点事件(点中输入框准备输入数据时，光标闪烁，称为焦点)，常用于输入 框失去焦点后验证其输入内容是否合法  
4、onchange：内容发生改变事件，常用于下拉列表选中项发生改变后操作  
5、onsubmit：
①表单提交事件，常用于表单提交之前验证所有表单项是否合法②onsubmit事件中return false可以阻止表单提交   
### js事件的绑定  
1、静态事件  
通过HTML标签的事件属性直接赋予事件响应后的代码(写成函数形式)   
例：静态注绑定onload事件  
```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script type="text/javascript">
        //静态注册onload事件的函数
        function onloadFun() {
            alert("静态注册onload事件！");
        }
    </script>
</head>
<body onload="onloadFun()"> <!--静态注册onload事件--> //函数名后一定要加括号
</body>
</html>
运行结果：页面加载后出现提示框：静态注册onload事件！
```
2、动态事件  
(1) 通过document对象.getElementId()获取标签对象(Dom对象)  
(2) 通过标签对象.事件名 = function() {} 赋予事件响应后的代码，代表此标签具有此事件   
例：动态绑定onload事件  
```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script type="text/javascript">
    //window.onload页面加载完成之后自动执行function中的代码
    window.onload = function () {
        alert("动态注册的onload事件");
    }
    </script>
</head>
<body>
</body>
</html>
</html>
```   
### js的正则表达式  
```js
    1.字符串中包含字母e
	var patt = new RegExp("e");
    patt.test(输入的字符串);//测试输入的字符串是否符合正则表达式
	var patt = /e/; //也是正则表达式写法，功能类似上一行
	2.字符串中包含字母a或b或c
	var patt = /[abc]/;
	3.字符串包含小写字母
	var patt = /[a-z]/;
	4.字符串包含任意大写字母
	var patt = /[A-Z]/;
	5.字符串包含任意数字
	var patt = /[0-9]/;
	6.字符串包含字母，数字，下划线
	var patt = /\w/;
	7.字符串中包含至少一个a
	var patt = /a+/;
	8.字符串中包含零个或多个a
	var patt = /a*/;
	9.字符串包含一个或零个a
	var patt = /a?/;
	10.字符串包含连续X个a
	var patt = /a{X}/;
	11.字符串包含至少X个连续的a，最多Y个连续的a
	var patt = /a{X,Y}/;
	12.字符串包含至少X个连续的a
	var patt = /a{X,}/;
	13.字符串必须以a结尾
	var patt = /a$/;
	14.字符串必须以a打头
	var patt = /^a/;
	15.字符串只可包含3-5个连续的a
	var patt = /^a{3,5}$/;
``` 
### js的DOM模型  
1、Document对象的理解  
(1) Document管理了HTML文档的所有内容，将这些内容都对象化  
(2) Document是一种树形结构，有层级关系  
(3) 可以通过Document访问所有的对象  
![result](https://static01.imgkr.com/temp/69d5c17509f0443398b05d455db2bc5e.png) 

2、Document对象中的方法和属性介绍   
1、方法   
![result](https://static01.imgkr.com/temp/4132a4bcdf9742328b8e6aa88cef1f58.png)   
注意，带s返回的都是集合：  
1、查询优先顺序：id --> name --> tagName(越往后查询结果范围越大，还需过滤)  
2、三个查询方法一定要在页面加载完成之后执行才能查询到标签对象  
appendChild(ChildNode)；添加一个子节点，ChildNode为添加的子节点    
createTextNode(Text)；创建一个以Text为内容的文本节点   
2、属性：  
(1) childNodes：当前节点的所有子节点  
(2) firstChild：当前节点的第一个子节点  
(3) lastChild：当前节点的最后一个子节点  
(4) parentNode：当前节点的父节点  
(5) nextSibling：当前节点的下一个节点  
(6) previousSibling：当前节点的上一个节点  
(7) className：标签的class属性值  
(8) innerHTML：标签中的内容(包括两边的标签)  
(9) innerText：标签中的文本(不包括两边的标签)  
注意：可以连写：document.getElementById(“xxx”).事件名 = function(){}    
 1、getElementById()方法代码示例  
 ```js
 <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script type="text/javascript">
        /*需求：当用户点击了校验按钮之后，验证内容是否合法，
          验证的规则：必须由字母、数字、下划线组成，并且长度是5-12位*/
        function onlickFun() {
            var inputObj = document.getElementById("username");
            //获得的inputObj：[object HTMLInputElement]
            var usernameText = inputObj.value;
            //value会随着输入框的内容发生改变，不仅仅是初始值
            var usernameSpanObj = document.getElementById("usernameSpan");
            var patt = /^\w{5,12}$/;或者new RegExp("^\\w{5,12}$");//需要转义
            //正则表达式的test方法用于测试参数中的字符串是否符和正则表达式的规则
            if (patt.test(usernameText)) {
                //innerHTML属性表示起始标签与结束标签中的内容，此属性可读可写
                usernameSpanObj.innerHTML = "用户名合法";
            } else {
                usernameSpanObj.innerHTML = "用户名不合法";
            }
        }
    </script>
</head>
<body>
    用户名：<input type="text" id="username" value="默认值">
    <button onclick="onlickFun()">校验</button>
    <span id="usernameSpan" style="color: red"></span>
</body>
</html>
//运行结果：点击校验之后判断输入是否符合正则表达式(合法)：
 ```  
 2、getElementsByName()方法代码示例  
 ```js
 //全选、全不选、反选
 <!DOCTYPE html>
<html lang="zh_CN">			   
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script type="text/javascript">
        //全选
        function checkAll() {
            var hobbies = document.getElementsByName("hobby");
            //得到的hobbies为：[object NodeList]
            /*getElementsByName根据name属性返回多个标签对象集合，这个集合的
              每个元素都是dom对象，元素的顺序是在HTML页面中从上到下的顺序，
              这个集合的操作跟数组一样*/
            /*checked属性表示复选框的选中状态，如果选中是true，未选中是false
              此属性可读可写*/
            for (var i = 0; i < hobbies.length; i++) {
                hobbies[i].checked = true;
            }
        }
        //全不选
        function checkNo() {
            var hobbies = document.getElementsByName("hobby");
            for (var i = 0; i < hobbies.length; i++) {
                hobbies[i].checked = false;
            }
        }
        //反选
        function checkReverse() {
            var hobbies = document.getElementsByName("hobby");
            for (var i = 0; i < hobbies.length; i++) {
                hobbies[i].checked = !hobbies[i].checked;
            }
        }
    </script>
</head>
<body>
    <input type="checkbox" name="hobby" value="cpp" checked="checked"/>c++
    <input type="checkbox" name="hobby" value="java">Java
    <input type="checkbox" name="hobby" value="JS">JavaScript
    <br/>
    <button onclick="checkAll()">全选</button>
    <button onclick="checkNo()">全不选</button>
    <button onclick="checkReverse()">反选</button>
</body>
</html>
 ```    
3、createElement()代码示例  
```js
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script type="text/javascript">
        window.onload = function () {
            var divObj = document.createElement("div");
            var textNodeObj = document.createTextNode("中国，我爱你");
            divObj.appendChild(textNodeObj);
            //向body标签中添加元素，以便在页面中显示
            /*使用document.body可直接获得body标签对象*/
            document.body.appendChild(divObj);
            //代码执行下来直接使用body，body必须加载出来，故将代码放在window.onload中
        }
    </script>
</head>
//运行结果：中国，我爱你(没有两边的标签)
```

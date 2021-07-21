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

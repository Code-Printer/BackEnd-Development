# Json(JavaScript object notation)
json是javaScript对象表示的简化，多用于数据的传递和存储，json比xml更小更快、更易解析  

## JSON的数据格式  
json是由{键值对表示的，其中键可以用双引号，值的类型有数组[]，对象｛｝，数字，字符串，null}  
## json数据与java对象的转换  
json与java的相互转换需要json解析器，常见的解析器有jackson、Gson  
### json转java对象  
1、需要构建映射对象（ObjectMapper）  
2、使用转换方法(readValue)  
```java
public void test() throws IOException {
    String json = "{\"name\":\"jack\",\"age\":22,\"birthday\":1596299850579}";
    ObjectMapper mapper = new ObjectMapper();
    User user = mapper.readValue(json, User.class);
    System.out.println(user);
}
```  
### java对象转json数据  
1、需要构建映射对象（ObjectMapper）  
2、使用转换方法(writeValue、writeValueAsString)  
```java
@Test
public void test() throws IOException {
	//1.创建User对象
	User user = new User();
	user.setAge(22);
	user.setName("jack");
	user.setBirthday(new Date());
	//2.创建jackson的核心对象 ObjectMapper
	ObjectMapper mapper = new ObjectMapper();
	//3.转换
	/* 转换方法：
	writeValue(参数1，obj):
	    参数1：
	    File：将obj对象转换为JSON字符串，并保存到指定的文件中
	    Writer：将obj对象转换为JSON字符串，并将json数据填充到字符输出流中
	    OutputStream：将obj对象转换为JSON字符串，并将json数据填充到字节输出流中
	writeValueAsString(obj):将对象转为json字符串
	*/
	String json = mapper.writeValueAsString(user);
	System.out.println(json);
	File file = new File("D://a.txt");
	mapper.writeValue(file,json);
}
```  
## Jquery实现js字符串与json对象相互转换
1、json转字符串(JSON.stringify(json对象))  
2、js字符串转json对象($.parseJSON(字符串))  
```js
<script>
	//定义一个json对象
   var resData = {url:"123"}
   //json对象转为js字符串
   alert(JSON.stringify(resData))
   //js字符串转json对象
   var jsonObj = $.parseJSON('{"url":"123"}');
   alert(jsonObj)
   alert(jsonObj["url"])
</script>
```
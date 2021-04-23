## JDBC  
JDBC(java DataBase Connnectivity)  
JDBC本质：是由Sun公司定义的操作关系型数据库的规则接口，各个数据库厂商去实现这个接口，并提供数据库驱动jar包。真正执行的代码是驱动jar包中的实现类。  

![result](https://static01.imgkr.com/temp/5bbf34137c9e4f548a23a4cf233a8c49.png)  

### JDBC的主要步骤:  
1.导入驱动jar包  
2.注册驱动  
3.获取数据库连接对象 Connection  
4.定义sql语句  
5.获取执行sql语句的对象 Statement  
6.执行sql，接收返回结果  
7.处理结果  
8.释放资源，最后开启的资源要先释放释放的资源有ResultSet(仅查询时使用)、Statement、Connection.  

## 数据库连接使用到的相关类和接口
DriverManager类：驱动管理对象,使用静态getConnection方法获取连接对象  
Connection接口：数据库连接对象  
Statement接口：用于执行静态 SQL 语句并返回它所生成结果的对象   
ResultSet接口：接收Statement接口返回的结果集对象  
PreparedStatement接口：是Statement的子接口，执行预编译SQL语句的对象(解决注入问题)  
### DriverManager类(驱动管理对象)  
功能：  
1、注册驱动：告诉程序该使用哪一个数据库驱动jar包；
static void registerDriver(Driver driver) 向 DriverManager 注册给定驱动程序；
Jar包中com.mysql.jdbc.Driver类中有静态代码块：  
![result](https://static01.imgkr.com/temp/2e7a41ba077a428892e7de0443ae1320.png)  
故可以使用反射加载运行时类Class.forName(“com.mysql.jdbc.Driver”);自动执行静态代码块完成注册驱动。  
2、获取数据库连接：建立到给定数据库 URL 的连接
static Connection getConnection(String url, String user, String password)
url参数：指定连接的路径  
url语法格式:协议与子协议是固定写法，子名称是数据库地址:端口号/数据库名称  
![result](https://static01.imgkr.com/temp/1c5ceffb11164145ab28d35b017922bd.png)  
注意：url可以简写(jdbc:mysql:///数据库名称，即省略ip地址(域名)：端口号)。  
### Connection接口(数据库连接对象)  
功能:  
1、获取执行sql语句的对象  
a. Statement createStatement( )创建一个Statement对象来将SQL语句发送到数据库。    
b.PreparedStatement prepareStatement(String sql)。 
创建一个 PreparedStatement 对象来将参数化的 SQL 语句发送到数据库  
2、管理事务：  
开启事务：void setAutoCommit(boolean autoCommit) 参数设置为false开启事务  
提交事务：void commit()  
回滚事务：void rollback()  
### Statement接口
用于执行静态SQL语句并返回它所生成结果的对象  
1、int executeUpdate(String sql)  
执行除了select之外的其余sql语句，即不返回任何内容的sql语句，返回值是受影响的行数，可以通过返回的int值判断sql语句是否执行成功(>0则成功，反之失败)，不使用ResultSet  
2、ResultSet executeQuery(String sql)
执行给定的select语句，该语句返回单个 ResultSet 对象   
代码演示:JDBC的初步使用1  
```java
public static void main(String[] args) throws Exception {
    //所有的异常均抛出
    //导入jar包结束后，注册驱动
    Class.forName("com.mysql.jdbc.Driver");
    //获取连接
    Connection connection = 	DriverManager.getConnection("jdbc:mysql://localhost:3306/test","root","root");
    //创建执行sql语句的对象
    Statement statement = connection.createStatement();
    //定义sql语句
    String sql = "update customers set name = 'xxx' where id = 8 ";
    //执行sql语句
    int i = statement.executeUpdate(sql);
    System.out.println(i); //1行受影响，输出1
    //关闭资源
    connection.close();
    statement.close();
}
//运行成功后，mysql的customers表的id为8的行的name变成了xxx
```    








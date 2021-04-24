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
2、JDBC的初步使用2
```java
public static void main(String[] args) {
	//为了保证流一定被关闭，不再抛出异常
    Statement statement = null; //若在try中定义出了try无法使用，即无法关闭
    Connection connection = null; //若在try中定义出了try无法使用，即无法关闭
    try {
        Class.forName("com.mysql.jdbc.Driver");
        connection =
                DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "root");
        statement = connection.createStatement();
        String sql = "insert into user_table values('高奇泽','123','6523')";
        int count = statement.executeUpdate(sql);
        if (count > 0) {
            System.out.println("执行成功");
        } else {
            System.out.println("执行失败");
        }
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (SQLException throwables) {
        throwables.printStackTrace();
    } finally {
        if (statement != null) { //不判断可能会空指针异常
            try {
                statement.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
        if (connection != null) { //不判断可能会空指针异常
            try {
                connection.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
    }
}
/*注：两个关闭流的操作不可写在一个try——catch中，否则第一个流关闭异常会导致第二个流关闭失败*/
```  
### ResultSet接口
结果集对象(可认为是查询出的一张表)  
有一个游标指向第一行的上一行，需要调用方法(boolean next方法)使游标下移，再调用方法(xxx getXxxx())获取指向行中某一列的值，注意：无法获取整行的值，只可以获取行中某一列的值。  
![result](https://static01.imgkr.com/temp/b1807f70b8294f839abd5c19e128ffa9.png)  
1、boolean next()游标向下移动一行，如果有下一行可以被指向，返回true，若无，返回false。  
2、xxx getXxx(参数)获取当前行中由参数所指定列的值；注：Xxx代表数据类型。  
参数：1、int：代表列的编号，从1开始  
    2、String：代表列的名称。  
java和sql中对应类型的转换：  
![result](https://static01.imgkr.com/temp/da284c501d2a480d9a430a020d7e1d3e.png)  
代码演示：使用ResultSet获取并遍历结果集  
```java
public static void main(String[] args) {
    Connection connection = null;
    Statement statement = null;
    ResultSet resultSet = null;
    try {
        Class.forName("com.mysql.jdbc.Driver");
        connection = 
                DriverManager.getConnection("jdbc:mysql://localhost:3306/girls", "root", "root");
        statement = connection.createStatement();
        String sql = "select * from boys";
        resultSet = statement.executeQuery(sql);
        //循环的遍历结果集
        while (resultSet.next()) {
            int id = resultSet.getInt("id");
            String string = resultSet.getString(2);
            int userCP = resultSet.getInt("userCP");
            System.out.println(id + string + userCP);
        }
    } catch (ClassNotFoundException | SQLException e) {
        e.printStackTrace();
    } finally {
        if (resultSet != null) {
            try {
                resultSet.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
        if (statement != null) {
            try {
                statement.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
    }
}

```
### 使用JDBCUtils工具
目的：简化书写，将每次使用JDBC都要重复写的代码抽取出来(获取连接对象和关闭接口资源)，放在静态方法中，方便调用。  
步骤:
1.抽取注册驱动的代码  
2.抽取一个方法获取连接对象  
3.抽取一个方法释放资源  
代码演示：使用JDBCUtils对Order表进行查询，结果放入集合中，并返回集合  
1、创建一个Order类(存放的是查询的对象)  
```java
public class Order {
    //一个类代表一张表，类中的属性代表一列，一个对象代表一行(ORM编程思想)
    private int order_id; //对象的属性
    private String order_name;
    private Date order_date; //util下的Date
    //定义所有属性的getter/setter方法
    public int getOrder_id() {
        return order_id;
    }
    public void setOrder_id(int order_id) {
        this.order_id = order_id;
    }
    public String getOrder_name() {
        return order_name;
    }
    public void setOrder_name(String order_name) {
        this.order_name = order_name;
    }
    public Date getOrder_date() {
        return order_date;
    }
    public void setOrder_date(Date order_date) {
        this.order_date = order_date;
    }
    @Override
    public String toString() {
        return "Order{" +
                "order_id=" + order_id +
                ", order_name='" + order_name + '\'' +
                ", order_date=" + order_date +
                '}';
    }
}
```  
2、在src下创建一个配置文件jdbc.properties，只需去读这个文件，就可以知道数据库的相关信息，下次只需改这个文件就可以改变访问的数据表了  
```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/test
user=root
password=root //更改数据库或驱动只需更改配置文件即可，其余无需更改，体现可扩展性
```  
3.创建工具类JDBCUtils  
```java
public class JDBCUtils {
    //1.注册驱动
    private static String url;
    private static String user;
    private static String password;
    private static String password;
    private static String driver;
    //静态代码块
    static {
        try {
            //读取配置文件(都写在try中，一旦有一处出现异常其余没必要再执行)
            Properties properties = new Properties();
            properties.load(new FileReader("src/jdbc.properties")); //抛出异常
            url = properties.getProperty("url");
            user = properties.getProperty("user");
            password = properties.getProperty("password");
            driver = properties.getProperty("driver");
            //注册驱动
            Class.forName(driver);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
    //2.获取连接(抛出异常)
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(url,user,password);
    }
    /*可直接类名.getConnection注册驱动，静态方法调动类加载，从而调动静态代码块*/
    //3.关闭资源
    //可能需要关闭ResultSet，也可能不需要，故需重载
    public static void close(Statement stmt, Connection conn) {
        if (stmt!=null){
            try {
                stmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (conn!=null){
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    public static void close(ResultSet rs, Statement stmt, Connection conn) {
        if (rs!=null){
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (stmt!=null){
            try {
                stmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (conn!=null){
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

}
```
4.编写测试类进行查询
```java
public class MyTest {
    public List<Order> search() {
        List<Order> list = null;
        Connection connection = null;
        Statement statement = null;
        ResultSet resultSet = null;
        try {
            connection = JDBCUtils.getConnection();
            statement = connection.createStatement();
            String sql = "select * from orders";
            resultSet = statement.executeQuery(sql);
            //创建Order类对象、集合
            Order order = null;
            list = new ArrayList<>();
            //循环
            while (resultSet.next()) {
                int order_id = resultSet.getInt("order_id");
                String order_name = resultSet.getString("order_name");
                Date order_date = resultSet.getDate("order_date");
	        //sql包的Date是Util包Date的子类，多态                
	        //每次遍历一行需创建一个对象，保存一行的所有数据
                order = new Order();
                order.setOrder_date(order_date);/*使用set赋值，而不使用构造器赋值，因为可
                order.setOrder_id(order_id);       能用户查询的列数并没有合适个数的构造器*/
                order.setOrder_name(order_name);
                //将order装入集合
                list.add(order);
                //继续下一次遍历
            }
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            JDBCUtils.close(resultSet,statement,connection);
        }
        return list;
    }
    //执行
    public static void main(String[] args) {
        List<Order> listResult = new MyTest().search();
        System.out.println(listResult);
        System.out.println(listResult.size());
    }
}
```  
### PreparedStatement接口
执行预编译SQL语句：参数使用?作为占位符，则上述练习的sql语句为：String sql="select * from user where username= ? and password = ?";  
注意： 通过Connection获取PreparedStatement接口的方法为prepareStatement(String sql)；  
使用步骤：
获取PreparedStatement的对象之后，需要使用PreparedStatement接口的方法给?赋值：void setXxx(int p1，Xxx p2)；注：Xxx为该列属性的类型  
p1：? 出现的位置，从1开始  
p2：给 ? 赋予的值  
使用PreparedStatement接口操作数据库的空参方法：
(1) int executeUpdate(); (2) ResultSet executeQuery();  
代码演示：使用PreparedStatement完成上述练习  
```java
public class MyTest {
    //创建方法，根据是否可以登录成功返回boolean值
    public boolean logIn(String userName, String passWord) {
        if (userName == null || passWord == null) {
            return false;
        }
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        //获取连接，执行sql语句
        try {
            connection = JDBCUtils.getConnection();
            String sql = "select * from user where name = ? and password = ?";
            preparedStatement = connection.prepareStatement(sql);
            preparedStatement.setString(1,userName);
            preparedStatement.setString(2,passWord);
            resultSet = preparedStatement.executeQuery();
            return resultSet.next();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            JDBCUtils.close(resultSet, preparedStatement,connection);
            //preparedStatement使用了多态，传递给了父类
        }
        return false; //出现异常，则返回false
    }
    //测试
    public static void main(String[] args) {
        Scanner scan = new Scanner(System.in);
        System.out.println("输入姓名：");
        String name = scan.nextLine();
        System.out.println("输入密码：");
        String passWord = scan.nextLine();
        boolean b = new MyTest().logIn(name, passWord);
        if (b) {
            System.out.println("可以成功登录！");
        } else {
            System.out.println("无法成功登录！");
        }
    }
}
```




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
### PreparedStatement接口(防止数据库的注入问题)
PreparedStatement的好处：(以后均使用此接口，弃用Statement)  
1、解决sql的注入问题  
2、可以操作Blob的数据(二进制的大型数据)
void setBlob(int parameterIndex, InputStream inputStream)；
3、可以实现更高效的批量操作  
代码演示:上面已经演示过了防注入问题，下面是演示2和3的操作。 
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
### 结果集的元数据(ResultSetMetaData)  
调用结果集ResultSet中的ResultSetMetaData getMetaData()方法，得到ResultSetMetaData的对象，通过ResultSetMetaData接口中的方法可获得结果集的元数据：  
(1) int getColumnCount()；获取结果集中的列数。  
(2) String getColumnLabel(int i)；获取查询结果表的列名，有别名，获取别名 ，无别名，获取列名。  
(3) String getColumnName(int i)；仅可获取表的列名，无法获取别名(使用较少)  
注：i从1开始  
代码演示:  
1、对Customers表创建通用的查询方法  
```java
public class Customers {
    //属性的名字与表中的列名不同
    private int ID;
    private String NAME;
    private String EMAIL;
    private Date BIRTH;
    public int getID() {
        return ID;
    }
    public void setID(int ID) {
        this.ID = ID;
    }
    public String getNAME() {
        return NAME;
    }
    public void setNAME(String NAME) {
        this.NAME = NAME;
    }
    public String getEMAIL() {
        return EMAIL;
    }
    public void setEMAIL(String EMAIL) {
        this.EMAIL = EMAIL;
    }
    public Date getBIRTH() {
        return BIRTH;
    }
    public void setBIRTH(Date BIRTH) {
        this.BIRTH = BIRTH;
    }
    @Override
    public String toString() {
        return "Customers{" +
                "ID=" + ID +
                ", NAME='" + NAME + '\'' +
                ", EMAIL='" + EMAIL + '\'' +
                ", BIRHT=" + BIRTH +
                '}';
    }
}
```  
2、编写对customers表通用的查询操作  
```java
public class MyTest {
    /**
     * @param sql 要执行的sql语句
     * @param args sql语句中占位符?的值，有几个?写几个值，传递到可变个数形参中
     * @return 存放查询结果的List集合
     */
    public static List<Customers> Query(String sql, Object...args) {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        try {
            connection = JDBCUtils.getConnection();
            preparedStatement = connection.prepareStatement(sql);
            //通过循环给?赋值
            for (int i = 0; i < args.length; i++) {
                preparedStatement.setObject(i + 1, args[i]);
            }
            //执行sql语句
            resultSet = preparedStatement.executeQuery();
            /*不同的sql语句得到的结果集不同，需要获得结果集的列数和名字，
              通过结果集的列数和名字，可以得知Order类中的哪些属性要被赋值 */
            //获取结果集的元数据
            ResultSetMetaData metaData = resultSet.getMetaData();
            int columnCount = metaData.getColumnCount();
            //定义List集合
            List<Customers> list = new ArrayList<>();
            //开始遍历结果集并将结果放在List集合中
            while (resultSet.next()) {
                //在while中判断为真才创建对象，若结果集无数据但创建了对象，造成空间浪费
                Customers customers = new Customers();
                for (int i = 0; i < columnCount; i++) {
                    //获取所在行的指定列的值，通过反射，将此值存入对应的属性
                    Object columnValue = resultSet.getObject(i + 1);
                    /*获取所在行的指定列的名称(属性名)，由于属性名与列名不同，
                      故通过给列起别名使用getColumnLabel */
                    String columnLabel = metaData.getColumnLabel(i + 1);
                    //由于未知给哪个属性赋值，故通过反射将columnValue赋值给对应的属性
                    Field field = Customers.class.getDeclaredField(columnLabel);
                    field.setAccessible(true);
                    field.set(customers, columnValue);
                }
                list.add(customers);
            }
            return list;
        } catch (Exception throwables) {
            throwables.printStackTrace();
        } finally {
            JDBCUtils.close(resultSet, preparedStatement, connection);
        }
        return null;
    }
    //测试
    public static void main(String[] args) {
        String sql = "select email as EMAIL from customers where id = ? or id = ?";
        List<Customers> list = Query(sql,3,5);
        System.out.println(list);
    }
}
```  
3、对不同表创建通用的查询方法  
```java
public class MyTest {
    public static <T> List<T> Query(Class<T> clazz, String sql, Object...args) {
        try {
            Connection connection = JDBCUtils.getConnection();
            PreparedStatement preparedStatement = connection.prepareStatement(sql);
            for (int i = 0; i < args.length; i++) {
                preparedStatement.setObject(i + 1, args[i]);
            }
            ResultSet resultSet = preparedStatement.executeQuery();
            ResultSetMetaData metaData = resultSet.getMetaData();
            int columnCount = metaData.getColumnCount();
            List<T> list = new ArrayList<>();
            while (resultSet.next()) {
                T t = clazz.newInstance();
                for (int i = 0; i < columnCount; i++) {
                    Object columnValue = resultSet.getObject(i + 1);
                    String columnName = metaData.getColumnLabel(i + 1);
                    Field field = clazz.getDeclaredField(columnName);
                    field.setAccessible(true);
                    field.set(t, columnValue);
                }
                list.add(t);
            }
            return list;
        } catch (Exception throwables) {
            throwables.printStackTrace();
        }
        return null;
    }   
    //测试
    public static void main(String[] args) {
        String sql =
                "select id as ID,name as NAME from customers where id = ? or id = ?";
        List<Customers> list = Query(Customers.class, sql, 3,4);
        System.out.println(list);

        String sql2 =
                "select order_name as name,order_id as id from `order` where order_id = ?"; //order要加``是因为order是mysql的关键字
        List<Order> list2 = Query(Order.class, sql2, 2);
        System.out.println(list2);
    }
}
```
PreparedStatement的好处：(以后均使用此接口，弃用Statement)  
1、解决sql的注入问题  
2、可以操作Blob的数据(二进制的大型数据)
void setBlob(int parameterIndex, InputStream inputStream)；
3、可以实现更高效的批量操作  
代码演示:上面已经演示过了防注入问题，下面是演示2和3的操作。  
### 操作Blob数据及进行批量操作  
代码演示：在costumers表操作Blob类型的数据  
1、向customers表插入Blob类型的数据
```java
Connection connection = null;
PreparedStatement preparedStatement = null;
try {
    connection = JDBCUtils.getConnection();
    String sql = "insert into customers(id,photo) values(?,?)";
    preparedStatement = connection.prepareStatement(sql);
    preparedStatement.setInt(1,19);
    preparedStatement.setBlob(2,new FileInputStream("girl.jpg"));
    preparedStatement.executeUpdate();
} catch (SQLException | FileNotFoundException throwables) {
    throwables.printStackTrace();
} finally {
    JDBCUtils.close(preparedStatement,connection);
}
```  
2、读取customers表中Blob类型的数据，并输出到本地
```java
Connection connection = null;
PreparedStatement preparedStatement = null;
ResultSet resultSet = null;
FileOutputStream fileOutputStream = null;
InputStream inputStream = null;
try {
    connection = JDBCUtils.getConnection();
    String sql = "select photo from customers where id = ?";
    preparedStatement = connection.prepareStatement(sql);
    preparedStatement.setInt(1,19);
    resultSet = preparedStatement.executeQuery();
    while (resultSet.next()) {
        Blob blob = resultSet.getBlob("photo");
        //调用Blob中的getBinaryStream方法获取指定文件的输入流
        inputStream = blob.getBinaryStream();
        //指定输出流
        fileOutputStream = new FileOutputStream("result.jpg");
        byte[] buffer = new byte[1024];
        int len;
        while ((len = inputStream.read(buffer)) != -1) {
            fileOutputStream.write(buffer, 0, len);
        }
    }
} catch (SQLException | FileNotFoundException throwables) {
    throwables.printStackTrace();
} catch (IOException e) {
    e.printStackTrace();
} finally {
    JDBCUtils.close(resultSet,preparedStatement, connection);
    if (inputStream != null) {
        try {
            inputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    if (fileOutputStream != null) {
        try {
            fileOutputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
} /*执行完毕，当前moudle下会出现result.jpg，内容是表中id=19的photo列属性的值*/
```  



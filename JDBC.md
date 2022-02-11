#   第1章：概述
## 1.1体系结构
	JDBC接口(API)包括两个层次： 
		面向应用的API：JAVA API，抽象接口，供应用程序开发人员使用（连接数据库，执行SQL语句，获得结果）
		面向数据库的API：Java Driver API，供开发商开发数据库驱动程序用。

> **JDBC是sun公司提供一套用于数据库操作的接口，java程序员只需要面向这套接口编程即可。**
>
> **不同的数据库厂商，需要针对这套接口，提供不同实现。不同的实现的集合，即为不同数据库的驱动。																————面向接口编程**

## 1.2程序编写步骤

![image-20220123231352002](/Users/junhao/Library/Application Support/typora-user-images/image-20220123231352002.png)

> 补充：ODBC(**Open Database Connectivity**，开放式数据库连接)，只需要调用ODBC API，由 ODBC 驱动程序将调用转换成为对特定的数据库的调用请求。

## 1.3JavaWeb技术栈

![image-20220125132009951](/Users/junhao/Library/Application Support/typora-user-images/image-20220125132009951.png)


# 第2章：获取数据库连接

## 2.1 要素一：Driver接口实现类
### 2.1.1 Driver接口实现类
	程序中不需要直接去访问实现了 Driver 接口的类，而是由驱动程序管理器类(java.sql.DriverManager)去调用这些Driver实现。
	Oracle的驱动：oracle.jdbc.driver.OracleDriver
	mySql的驱动： com.mysql.jdbc.Driver
	将上述jar包拷贝到Java工程的一个目录中，习惯上新建一个lib文件夹。
 ![1566134718955](/Users/junhao/Library/Application Support/typora-user-images/1566134718955.png)

	在驱动jar上右键-->Build Path-->Add to Build Path
 ![1566134781682](/Users/junhao/Library/Application Support/typora-user-images/1566134781682.png)

	注意：如果是Dynamic Web Project（动态的web项目）话，则是把驱动jar放到WebContent（有的开发工具叫WebRoot）目录中的WEB-INF目录中的lib目录下即可
	![1566135290460](/Users/junhao/Library/Application Support/typora-user-images/1566135290460.png)
## 2.2 要素二：URL
JDBC URL 用于标识一个被注册的驱动程序，驱动程序管理器通过这个 URL 选择正确的驱动程序，从而建立到数据库的连接。
JDBC URL的标准由三部分组成，各部分间用冒号分隔。 

	jdbc：子协议:子名称
	协议：JDBC URL中的协议总是jdbc 
	子协议：子协议用于标识一个数据库驱动程序
	子名称：一种标识数据库的方法。子名称可以依不同的子协议而变化，用子名称的目的是为了**定位数据库**提供足够的信息。包含**主机名**(对应服务端的ip地址)**，端口号，数据库名**

举例：
![1555576477107](/Users/junhao/Library/Application Support/typora-user-images/1555576477107.png)

## 2.2 几种常用数据库的 JDBC URL
MySQL的连接URL编写方式：

	jdbc:mysql://主机名称:mysql服务端口号/数据库名称?参数=值&参数=值
	jdbc:mysql://localhost:3306/test
	jdbc:mysql://localhost:3306/test**?useUnicode=true&characterEncoding=utf8**（如果JDBC程序与服务器端的字符集不一致，会导致乱码，那么可以通过参数指定服务器端的字符集）
	jdbc:mysql://localhost:3306/test?user=root&password=123456

Oracle 9i的连接URL编写方式：

	jdbc:oracle:thin:@主机名称:oracle服务端口号:数据库名称
	jdbc:oracle:thin:@localhost:1521:test

SQLServer的连接URL编写方式：

	jdbc:sqlserver://主机名称:sqlserver服务端口号:DatabaseName=数据库名称
	jdbc:sqlserver://localhost:1433:DatabaseName=test


## 2.4 要素三：用户名和密码

	user,password可以用“属性名=属性值”方式告诉数据库
	可以调用 DriverManager 类的 getConnection() 方法建立到数据库的连接

## 2.5 数据库连接方式
```java
	@Test
    public void testConnection4() {
            //1.配置文件的4个基本要素：
            url="jdbc:mysql://localhost:3306/test"
            user="root"
            password="123456"
            driverClass="com.mysql.cj.jdbc.Driver"

            //2.加载配置文件
            InputStream is = ConnectionTest.class.getClassLoader().getResourceAsStream("jdbc.properties")
            Prrperties pros = new Properties();
            pros.load(is)
            //3.加载驱动 （①实例化Driver ②注册驱动）
            Class.forName(driverClass);

            //4.注册驱动
            //DriverManager.registerDriver(driver);
            /*
            可以注释掉上述代码的原因，是因为在mysql的Driver类中声明有：
            static {
                try {
                    DriverManager.registerDriver(new Driver());
                } catch (SQLException var1) {
                    throw new RuntimeException("Can't register driver!");
                }
            }
     				*/
          
            //3.获取连接
            Connection conn = DriverManager.getConnection(url, user, password);
            System.out.println(conn);
        } 

    }
```

> 说明：不必显式的注册驱动了。因为在DriverManager的源码中已经存在静态代码块，实现了驱动的注册。


# 第3章：PreparedStatement实现CRUD操作

### 3.1 操作和访问数据库

数据库连接被用于向数据库服务器发送命令和 SQL 语句，并接受数据库服务器返回的结果。其实一个数据库连接就是一个Socket连接。


	在 java.sql 包中有 3 个接口分别定义了对数据库的调用的不同方式：
	Statement：用于执行静态 SQL 语句并返回它所生成结果的对象。 
  - PrepatedStatement：SQL 语句被预编译并存储在此对象中，可以使用此对象多次高效地执行该语句。
  - CallableStatement：用于执行 SQL 存储过程

### 3.2 为什么使用Preparedment来进行增删改查
preparedSatement 的优点

1) 允许动态和参数化的query(易于维护)

2. 运行效率快

   其实就是预编译的问题。 PreparedSatement 会预编译， 而Statement基本上无法重用。 所以多次执行的话前者效率高很多。 生产环境中遇到过这种情况， 有几万条sql， 数据库的工作效率严重下降， 后来通过替换成PreparedStatement解决了这个问题

 3.安全性 预防sql注入的攻击

```java
Statement stmt = conn.createStatement("INSERT INTO students VALUES('" + user + "')");
stmt.execute();
```

If "user" came from user input and the user input was

```java
Robert'); DROP TABLE students; --
```

```mysql
//通过Mysql注入查询表中所有数据
SELECT user,password FROM user_table WHERE USER = '1' or ' AND PASSWORD = '='1' or '1' = '1';
```

然后自己写了代码实践了一下， 发现确实如果statement 会把 table drop掉。 而prepared的呢因为已经预编译了， 根本就不会执行这个drop语句。所以是安全的。　

4.除了解决Statement的拼串、sql注入之外，PreparedStatement还有哪些好处呢?

1. Preparedstatement 操作Blob的数据，而Statement做不到
2. PreparedStatement 可以实现更高效的批量操作

![image-20220130170536658](/Users/junhao/Library/Application Support/typora-user-images/image-20220130170536658.png)

### 3.3 Java与SQL对应数据类型转换表
数据库表中的字段名可以和属性名不同，但类型需要相同
| Java类型           | SQL类型                  |
| ------------------ | ------------------------ |
| boolean            | BIT                      |
| byte               | TINYINT                  |
| short              | SMALLINT                 |
| int                | INTEGER                  |
| long               | BIGINT                   |
| String             | CHAR,VARCHAR,LONGVARCHAR |
| byte   array       | BINARY  ,    VAR BINARY  |
| java.sql.Date      | DATE                     |
| java.sql.Time      | TIME                     |
| java.sql.Timestamp | TIMESTAMP                |

### 3.4查询操作的流程

![image-20220208012720082](/Users/junhao/Library/Application Support/typora-user-images/image-20220208012720082.png)




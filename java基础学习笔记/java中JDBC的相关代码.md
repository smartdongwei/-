# jdbc详解

## 一:什么是jdbc？

​    JDBC（Java Data Base Connectivity,java数据库连接）是一种用于执行SQL语句的Java API，可以为多种关系数据库提供统一访问，它由一组用Java语言编写的类和接口组成。JDBC提供了一种基准，据此可以构建更高级的工具和接口，使数据库开发人员能够编写数据库应用程序。

## 二: **常用接口**

### 2.1.Driver接口

　　Driver接口由数据库厂家提供，作为java开发人员，只需要使用Driver接口就可以了。在编程中要连接数据库，必须先装载特定厂商的数据库驱动程序，不同的数据库有不同的装载方法。如：

　　装载MySql驱动：Class.forName("com.mysql.jdbc.Driver");

　　装载Oracle驱动：Class.forName("oracle.jdbc.driver.OracleDriver");

### 2.2.Connection接口

Connection与特定数据库的连接（会话），在连接上下文中执行sql语句并返回结果。DriverManager.getConnection(url, user, password)方法建立在JDBC URL中定义的数据库Connection连接上。

　　连接MySql数据库：Connection conn = DriverManager.getConnection("jdbc:mysql://host:port/database", "user", "password");

　　连接Oracle数据库：Connection conn = DriverManager.getConnection("jdbc:oracle:thin:@host:port:database", "user", "password");

　　连接SqlServer数据库：Connection conn = DriverManager.getConnection("jdbc:microsoft:sqlserver://host:port; DatabaseName=database", "user", "password");

### 2.3.Statement接口

　　用于执行静态SQL语句并返回它所生成结果的对象。三种Statement类：

- Statement：由createStatement创建，用于发送简单的SQL语句（不带参数）。
- PreparedStatement ：继承自Statement接口，由preparedStatement创建，用于发送含有一个或多个参数的SQL语句。PreparedStatement对象比Statement对象的效率更高，并且可以防止SQL注入，所以我们一般都使用PreparedStatement。
- CallableStatement：继承自PreparedStatement接口，由方法prepareCall创建，用于调用存储过程。

常用Statement方法：

- execute(String sql):运行语句，返回是否有结果集
- executeQuery(String sql)：运行select语句，返回ResultSet结果集。
- executeUpdate(String sql)：运行insert/update/delete操作，返回更新的行数。
- addBatch(String sql) ：把多条sql语句放到一个批处理中。
- executeBatch()：向数据库发送一批sql语句执行。

### 2.4.ResultSet接口

ResultSet提供检索不同类型字段的方法，常用的有：

- getString(int index)、getString(String columnName)：获得在数据库里是varchar、char等类型的数据对象。
- getFloat(int index)、getFloat(String columnName)：获得在数据库里是Float类型的数据对象。
- getDate(int index)、getDate(String columnName)：获得在数据库里是Date类型的数据。
- getBoolean(int index)、getBoolean(String columnName)：获得在数据库里是Boolean类型的数据。
- getObject(int index)、getObject(String columnName)：获取在数据库里任意类型的数据。

ResultSet还提供了对结果集进行滚动的方法：

- next()：移动到下一行
- Previous()：移动到前一行
- absolute(int row)：移动到指定行
- beforeFirst()：移动resultSet的最前面。
- afterLast() ：移动到resultSet的最后面。

使用后依次关闭对象及连接：ResultSet → Statement → Connection

## **三、使用JDBC的步骤**

加载JDBC驱动程序 → 建立数据库连接Connection → 创建执行SQL的语句Statement → 处理执行结果ResultSet → 释放资源

**1.注册驱动 (只做一次)**

Class.forName(“com.MySQL.jdbc.Driver”);

**2.建立连接**

```java
　Connection conn = DriverManager.getConnection(url, user, password); 
```

**3.创建执行SQL语句的statement**

```java
String id = "5";
String sql = "delete from table where id=" +  id;
Statement st = conn.createStatement();  
st.executeQuery(sql);  
```

```java
 //PreparedStatement 有效的防止sql注入(SQL语句在程序运行前已经进行了预编译,当运行时动态地把参数传给PreprareStatement时，即使参数里有敏感字符如 or '1=1'也数据库会作为一个参数一个字段的属性值来处理而不会作为一个SQL指令)
String sql = “insert into user (name,pwd) values(?,?)”;  
PreparedStatement ps = conn.preparedStatement(sql);  
ps.setString(1, “col_value”);  //占位符顺序从1开始
ps.setString(2, “123456”); //也可以使用setObject
ps.executeQuery(); 
```

## **四、事务（ACID特点、隔离级别、提交commit、回滚rollback）**

### 4.1 事务的四大特点

atomicity(原子性)    consistency(一致性)     isolation(隔离性)   durability(持久性)



## 五：jdbc连接池


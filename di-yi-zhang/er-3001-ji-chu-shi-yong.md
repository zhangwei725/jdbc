# 二、基础使用

## 1、前期准备工作

1. [下载驱动jar包](http://www.oracle.com/technetwork/database/enterprise-edition/jdbc-112010-090769.html)

2. Oracle的jdbc驱动

   1、JDBC Thin\(开发常用\)

   ```
   thin是for thin client的意思，这种驱动一般用在运行在WEB浏览器中的JAVA程序。
   它不是通过OCI or Net8，而是通过Java sockets进行通信，是纯java实现的驱动，因此不需要在使用JDBC Thin的客户端机器上安装orcale客户端软件，所以有很好的移植性，通常用在web开发中。
   ```

   2、JDBC OCI

   ```
   oci是Oracle call interface的缩写，此驱动类似于传统的ODBC 驱动。因为它需要Oracle Call Interface and Net8，所以它需要在运行使用此驱动的Java程序的机器上安装客户端软件，其实主要是用到orcale客户端里以dll方式提供的oci和服务器配置。
   ```

   3、JDBC KPRB

   ```
   这种驱动由直接存储在数据库中的JAVA程序使用,是在服务器内部使用，他使用默认或当前的会话连接来访数据库，不需要用户名密码等，也不需要数据库url
   ```

3. 其他

   ```
   查看数据版本
   select * from v$versio

   查看具体的版本号对应的驱动
   http://tiantian0521.blog.163.com/blog/static/4172088320117294265766
   ```

## 2、基础步骤

### 1、加载驱动

1. 加载驱动方式一

   ```
    DriverManager.registerDriver(new Driver())；
   ```

2. 加载驱动方式二

   ```
    Class.forName("com.mysql.jdbc.Driver") ;
   ```

3. 注意：在实际开发中并不推荐采用registerDriver方法注册驱动。

   1、推荐采取采用方式一的方式加载,这样不会导致驱动对象在内存中重复出现，并且采用此种方式，程序仅仅只需要一个字符串，不需要依赖具体的驱动，即不需要import相关的包，使程序的灵活性更高

   2、方式二,查看Driver的源代码可以看到，如果采用此种方式，会导致驱动程序注册两次，也就是在内存中会有两个Driver对象。程序依赖mysql的api，脱离mysql的jar包，程序将无法编译，将来程序切换底层数据库将会非常麻烦。

4. 自动加载驱动

   ```
   JDK6之后，JDBC已经升级到4.0了
   自动加载java.sql.Driver，而不需要再调用class.forName。
   ```

### 2、建立连接

#### 2.1、核心代码

```
mysql
Connection connection = DriverManager
                            .getConnection("jdbc:mysql://127.0.0.1:3306/test", 
                                            "root",
                                                "root");
```

```
oracle
Connection connection = DriverManager
                            .getConnection("jdbc:oracle:thin:@127.0.0.1:1521:orcl",                                             "scott", 
                                        "123456");
```

#### 2.2、说明

1. 参数一URL标准语法由三部分组成:

   ```
   jdbc : 子协议 : 子名称
   jdbc   ---  协议, jdbc的协议总是jdbc
   子协议  --- 驱动程序,各种驱动程序都有不同的子协议
   子名称 即数据源的名称,是表示数据库的方法,为定位数据库提供确定的信息,每个子协议都自己的特有结构
   ```

#### 2.3、常用数据库URL

1. mysql

   ```
   MySQL Connector/J Driver
   下载地址:http://central.maven.org/maven2/mysql/mysql-connector-java/5.1.43/mysql-connector-java-5.1.43.jar
   驱动程序包名：MySQL-connector-Java-x.x.xx-bin.jar
   驱动程序类名: com.mysql.jdbc.Driver
   JDBC URL: jdbc:mysql://<host>:<port>/<database_name>
   默认端口3306，如果服务器使用默认端口则port可以省略
   MySQL Connector/J Driver 允许在URL中添加额外的连接属性
   jdbc:mysql://<host>:<port>/<database_name>?property1=value1&property2=value2
   ```

2. oracle

   ```
   Oracle Thin JDBC Driver
   驱动程序包名：ojdbc6.jar
   驱动程序类名: Oracle.jdbc.driver.OracleDriver
   JDBC URL: jdbc:oracle:thin:@<host>:<port>:<SID>
   ```

### 3、创建Statement对象

#### 3.1、核心代码

```
Statement statement = connection.createStatement();
```

#### 3.2、说明

1. 创建一个Statement对象主要是调用该对象的方法执行SQL语句
2. 返回相应的结果

#### 3.3、编写sql语句

1. 例如:

   \`\`\`  
   String sql1 =  "SELECT \* FROM EMP"  
   Statement statement1 = connection.createStatement\(\);  
   ResultSet  rs1 = statement.executeQuery\(sql1\);

String sql2 =  "SELECT ename,empno,job FROM EMP"  
   Statement statement2 = connection.createStatement\(\);  
   ResultSet  rs2 = statement.executeQuery\(sql2\);

```
2. 注意事项
```

JDBC没执行一条SQL语句,都需要Connection对象的createStatement相关的方法来创建Statement对象,  
   利用Statement对象向数据库执行sql语句

```
3.3、Statement对象常用方法

1. 执行DML操作(insert  update delete)
```

int  executeUpdate\(String sql\);  
   返回影响的行数

```
2. 执行DQL操作
```

ResultSet   executeQuery\(String sql\);  
   返回查询的对象信息

```
3. 返回多个结果的SQL操作
```

boolean execute\(String sql,  
                   int\[\] columnIndexes\)  
                   throws SQLException  
   如果结果为 ResultSet 对象，则返回 true；如果其为更新计数或者不存在更多结果，则返回 false  
   一般用于执行DDL操作

```
4. 执行示例图

   ![](http://opzv089nq.bkt.clouddn.com/17-8-9/79348473.jpg)

### 4、处理结果

4.1、执行更新返回的是本次操作影响到的记录数。
```

int count = statement.executeUpdate\(sql\);

```
4.2、执行查询返回的结果是一个ResultSet对象。

1. ResultSet包含符合SQL语句中条件的所有行，并且它通过一套get方法提供了对这些行中数据的访问。  
2. 使用结果集（ResultSet）对象的访问方法获取数据
```

while\(rs.next\(\)\){  
         String name = rs.getString\("name"\) ;  
         String pass = rs.getString\(1\) ; // 此方法比较高效  
     }

```
4.3、ResultSet示意图

![](http://opzv089nq.bkt.clouddn.com/17-8-9/87477406.jpg)

###  5、关闭JDBC对象

操作完成以后要把所有使用的JDBC对象全都关闭，以释放JDBC资源，关闭顺序和声明顺序相反 

1. 关闭记录集   

2. 关闭声明   

3. 关闭连接对象
```

```
if(rs != null){   // 关闭记录集   
       try{   
           rs.close() ;   
       }catch(SQLException e){   
           e.printStackTrace() ;   
       }   
         }   
         if(stmt != null){   // 关闭声明   
       try{   
           stmt.close() ;   
       }catch(SQLException e){   
           e.printStackTrace() ;   
       }   
         }   
         if(conn != null){  // 关闭连接对象   
        try{   
           conn.close() ;   
        }catch(SQLException e){   
           e.printStackTrace() ;   
        }   
}  
```

\`\`\`

​

### 6、jdbc开发步骤图

![](http://opzv089nq.bkt.clouddn.com/17-8-9/26866494.jpg)

### 7、常用的对应的数据类型

| mysql | oracle | java |
| :--- | --- | --- |
| TIMESTAMP | Timestamp | java.sql.Timestamp |
| VARCHAR | VARCHAR2 CLOB | java.lang.String |
| TEXT | VARCHAR2 CLOB | java.lang.String |
| INT | NUMBER\(10,0\) | java.lang.Integer |
| date\(年月日\) |  | java.sql.Date |
|  | date | java.sql.Date |

​


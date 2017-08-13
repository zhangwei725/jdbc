# Apache开源框架-DbUtils

## 一、简介

1. DbUtils主要是封装了JDBC的代码，
2. 简化dao层的操作的轻量级开源框架，
3. 帮助程序猿提高程序的开发效率

## 二、为什么需要Dbutils ？

在使用Dbutils 之前，我们Dao层使用的技术是JDBC，那么分析一下JDBC的弊端：

1. 数据库链接对象、sql语句操作对象，封装结果集对象，这三大对象会重复定义
2. 封装数据的代码重复，而且操作复杂，代码量大
3. 释放资源的代码重复
4. 程序员在开发的时候，有大量的重复代码，大大的延长了程序开发周期，工作效率低，维护困难。

## 三、Dbutils核心类介绍

### 3.1、DbUtils

1. 说明

   ```
   DbUtils是一个做关闭连接，装载JDBC驱动程序等工作的类，它里面所有的方法都是静态的
   ```

2. 常用api

   1、public static void close(…) throws java.sql.SQLException 

   ```
   DbUtils类提供了三个重载的关闭方法。这些方法检查所提供的参数是不是NULL，如果不是的话，它们就关闭Connection、Statement和ResultSet。
   ```

   2、public static void closeQuietly(…) 

   ```
   这一类方法不仅能在Connection、Statement和ResultSet为NULL情况下避免关闭，还能隐藏一些在程序中抛出的SQLException。
   ```

   3、public static void commitAndCloseQuietly(Connection conn) 

   ```
   用来提交连接，然后关闭连接，并且在关闭连接时不抛出SQL异常。
   ```

### 3.2、QueryRunner

1. 说明

```
该类简单化了SQL查询，它与ResultSetHandler组合在一起使用可以完成大部分的数据库操作，能够大大减少编码量,可以设置查询结果集的封装策略，线程安全
```

1. 构造方法

   1、QueryRunner()

   ```
   创建一个与数据库无关的QueryRunner对象，后期再操作数据库的会后，需要手动给一个Connection对象，它可以手动控制事务
   ```

   2、QueryRunner(DataSource ds)

   ```
   创建一个与数据库关联的queryRunner对象，后期再操作数据库的时候，不需要Connection对象，自动管理事务。
   DataSource：数据库连接池对象
   ```

2. 常用API

   |  回值   | 方法名                                      | 说明                       |
   | :---: | ---------------------------------------- | ------------------------ |
   | int[] | batch(Connection conn, String sql, Object[][] params) | 批量执行INSERT、UPDATE或DELETE |
   | int[] | batch(String sql, Object[][] params)     | 批量执行INSERT、UPDATE或DELETE |
   |   T   | insert(Connection conn, String sql, ResultSetHandler rsh) | 执行一个插入查询语句               |
   |   T   | insert(Connection conn, String sql, ResultSetHandler rsh, Object… params) | 执行一个插入查询语句               |
   |   T   | insert(String sql, ResultSetHandler rsh) | 执行一个插入查询语句               |
   |   T   | insert(String sql, ResultSetHandler rsh, Object… params) | 执行一个插入查询语句               |
   |   T   | insertBatch(Connection conn, String sql, ResultSetHandler rsh, Object[][] params) | 批量执行插入语句                 |
   |   T   | insertBatch(String sql, ResultSetHandler rsh, Object[][] params) | 批量执行插入语句                 |
   |   T   | query(Connection conn, String sql, ResultSetHandler rsh) | 查询                       |
   |   T   | query(Connection conn, String sql, ResultSetHandler rsh, Object… params) | 查询                       |
   |   T   | query(String sql, ResultSetHandler rsh)  | 查询                       |
   |   T   | query(String sql, ResultSetHandler rsh, Object… params) | 查询                       |
   |  int  | update(Connection conn, String sql)      | 执行INSERT、UPDATE或DELETE   |
   |  int  | update(Connection conn, String sql, Object… params) | 执行INSERT、UPDATE或DELETE   |
   |  int  | update(Connection conn, String sql, Object param) | 执行INSERT、UPDATE或DELETE   |
   |  int  | update(String sql)                       | 执行INSERT、UPDATE或DELETE   |
   |  int  | update(String sql, Object… params)       | 执行INSERT、UPDATE或DELETE   |
   |  int  | update(String sql, Object param)         | 执行INSERT、UPDATE或DELETE   |

### 3.3、ResultSetHandle

#### 1、说明

```
将封装结果集中的数据，转换到另一个对象
```

#### 2、子类

1. ArrayHandler：     将查询结果的第一行数据，保存到Object数组中
2. ArrayListHandler     将查询的结果，每一行先封装到Object数组中，然后将数据存入List集合
3. BeanHandler     将查询结果的第一行数据，封装到user对象
4. BeanListHandler     将查询结果的每一行封装到user对象，然后再存入List集合
5. ColumnListHandler     将查询结果的指定列的数据封装到List集合中
6. MapHandler     将查询结果的第一行数据封装到map结合（key==列名，value==列值）
7. MapListHandler     将查询结果的每一行封装到map集合（key==列名，value==列值），再将map集合存入List集合
8. BeanMapHandler     将查询结果的每一行数据，封装到User对象，再存入mao集合中（key==列名，value==列值）
9. KeyedHandler     将查询的结果的每一行数据，封装到map1（key==列名，value==列值 ），然后将map1集合（有多个）存入map2集合（只有一个）
10. ScalarHandler     封装类似count、avg、max、min、sum......函数的执行结果

## 四、ResultSetHandle子类详解

### 4.1、ScalarHandler

返回一个对象，用于读取结果集中第一行指定列的数据

```
public void queryByScalarHandler() throws SQLException
{
   QueryRunner runner = new QueryRunner();
   Long count = runner.query(conn, "select count(*) from emp",
         new ScalarHandler<Long>());
}
```

### 4.2、ArrayHandler

ArrayHandler会返回一个数组，用于将结果集第一行数据转换为数组。

```
@Test
public void queryByArrayHandler() throws SQLException
{
   QueryRunner runner = new QueryRunner();
   Object[] results = runner.query(conn, "select * from emp",
         new ArrayHandler());
   System.out.println(Arrays.asList(results));
}
```

### 4.3、ArrayListHandler

ArrayListHandler会返回一个集合，集合中的每一项对应结果集指定行中的数据转换后的数组。

```
public void queryByArrayListHandler() throws SQLException
{
   QueryRunner runner = new QueryRunner();
   List<Object[]> results = runner.query(conn, "select * from emp",
         new ArrayListHandler());
   for (Object[] object : results)
   {
      System.out.println(Arrays.asList(object));
   }
}
```

### 4.4、KeyedHandler

KeyedHandler会返回一个Map，我们可以指定某一列的值作为该Map的键，Map中的值为对应行数据转换的键值对，键为列名。

```
public void queryByKeyedHandler() throws SQLException
{

   QueryRunner runner = new QueryRunner();
   Map<String, Map<String, Object>> results = runner.query(conn,
         "select * from emp", new KeyedHandler<String>());
   System.out.println(results);
}
```

### 4.5、ColumnListHandler

ColumnListHandler会返回一个集合，集合中的数据为结果集中指定列的数据。

```
public void queryByColumnListHandler() throws SQLException
{
   QueryRunner runner = new QueryRunner();
   List<String> results = runner.query(conn, "select * from emp",
         new ColumnListHandler<String>());
   System.out.println(results);
}
```

### 4.6、MapHandler

MapHandler会将结果集中第一行数据转换为键值对，键为列名。

```
public void queryByMapHandler() throws SQLException
{
   printCurrentMethodName();
   QueryRunner runner = new QueryRunner();
   Map<String, Object> results = runner.query(conn,
         "select * from emp", new MapHandler());
}
```

### 4.7、MapListHandler

MapHandler会将结果集中的数据转换为一个集合，集合中的数据为对应行转换的键值对，键为列名

```
@Test
public void queryByMapListHandler() throws SQLException
{
   printCurrentMethodName();
   QueryRunner runner = new QueryRunner();
   List<Map<String, Object>> results = runner.query(conn,
         "select * from emp", new MapListHandler());
   System.out.println(results);
}
```

### 4.8、BeanHandler(常用)

BeanHandler实现了将结果集第一行数据转换为Bean对象，在实际应用中非常方便。

```
public class Emp {

    private Integer empno;
    private String ename;
    private String job;
    private Integer mgr;
    private Date hiredate;
    private Double sal;
    private Double comm;
    ...
    
}    
```

```
public void queryByBeanHandler() throws SQLException
{
   QueryRunner runner = new QueryRunner();
   Emp results = runner.query(conn, "select * from emp",
         new BeanHandler<Emp>(Emp.class));
   System.out.println(results);
}
```

### 4.9、BeanListHandler(常用)

BeanHandler只转换结果集的第一行，而BeanListHandler会将结果集的所有行进行转换，返回一个集合。

```
public void queryByBeanListHandler() throws SQLException
{
   QueryRunner runner = new QueryRunner();
   List<Emp> results = runner.query(conn, "select * from Emp",
         new BeanListHandler<Emp>(Emp.class));
   System.out.println(results);
}
```


# JDBC核心类

## 一、Driver接口

### 1、说明

装载驱动程序的管理器,创建和管理应用程序与驱动程序之间的结构

### 2、常用API

1. 装载MySql驱动：Class.forName\("com.mysql.jdbc.Driver"\);
2. 装载Oracle驱动：Class.forName\("oracle.jdbc.driver.OracleDriver"\);

## 二、Connection接口

### 1、说明

​    Connection与特定数据库的连接（会话），在连接上下文中执行sql语句并返回结果。DriverManager.getConnection\(url, user, password\)方法建立在JDBC URL中定义的数据库Connection连接上。

### 2、常用API

1. createStatement\(\)

   ```
   创建向数据库发送sql的statement对象。
   ```

2. prepareStatement\(sql\)

   ```
   创建向数据库发送预编译sql的PrepareSatement对象。
   ```

3. prepareCall\(sql\)

   ```
   创建执行存储过程的callableStatement对象。
   ```

4. setAutoCommit\(boolean autoCommit\)

   ```
   设置事务是否自动提交。
   ```

5. commit\(\)

   ```
   在链接上提交事务。
   ```

6. rollback\(\)

   ```
   在此链接上回滚事务。
   ```

## 三、Statement接口

### 1、说明

​    用于执行静态SQL语句并返回它所生成结果的对象

### 2、Statement类：

1. Statement

   ```
   由createStatement创建，用于发送简单的静态SQL语句（不带参数）。
   ```

2. PreparedStatement

   ```
   继承自Statement接口，由preparedStatement创建，用于发送含有一个或多个参数的动态SQL语句。PreparedStatement对象比Statement对象的效率更高，并且可以防止SQL注入，一般都使用PreparedStatement
   ```

3. CallableStatement

   ```
   继承自PreparedStatement接口，由方法prepareCall创建，用于调用存储过程。
   ```

### 3、常用API

1. execute\(String sql\)

   ```
   运行语句，返回是否有结果集
   ```

2. executeQuery\(String sql\)

   ```
   运行select语句，返回ResultSet结果集。
   ```

3. executeUpdate\(String sql\)

   ```
   运行insert/update/delete操作，返回更新的行数。
   ```

4. addBatch\(String sql\)

   ```
   把多条sql语句放到一个批处理中。
   ```

5. executeBatch\(\)

   ```
   向数据库发送一批sql语句执行
   ```

## 四、ResultSet接口

### 1、说明

​    SELECT 语句执行完成后返回的数据结果集\(包括行跟列\)

### 2、ResultSet提供检索不同类型字段的方法

1. getString\(int index\)、getString\(String columnName\)

   ```
   获得在数据库里是varchar(varchar2)、 char等类型的数据对象
   ```

2. getFloat\(int index\)、getFloat\(String columnName\)

   ```
   获得在数据库里是Float类型的数据对象
   ```

3. getDate\(int index\)、getDate\(String columnName\)

   ```
   获得在数据库里是Date类型的数据
   ```

4. getBoolean\(int index\)、getBoolean\(String columnName\)

   ```
   获得在数据库里是Boolean类型的数据。
   ```

5. getObject\(int index\)、getObject\(String columnName\)

   ```
   获取在数据库里任意类型的数据
   ```

### 3、ResultSet还提供了对结果集进行滚动的方法

1. next\(\)

   ```
   移动到下一行
   ```

2. Previous\(\)

   ```
   移动到前一行
   ```

3. absolute\(int row\)

   ```
   移动到指定行
   ```

4. beforeFirst\(\)

   ```
   移动resultSet的最前面。
   ```

5. afterLast\(\)

   ```
   移动到resultSet的最后面。
   ```

## 五、DatabaseMetaData 接口

### 1、说明

```
 关于数据库结构,数据库系统和驱动程序等的元数据信息
```

### 2 、常用API

1. ResultSet getTables\(String catalog,String schemaPattern,String tableNamePattern,String\[\] types\);

   ```
   获取表信息
   ```

2. ResultSet getPrimaryKeys\(String catalog,String schema,String table\);

   ```
   获取表主键信息
   ```

3. ResultSet getIndexInfo\(String catalog,String schema,String table,boolean unique,boolean approximate\);

   ```
   获取表索引信息
   ```

4. ResultSet getColumns\(String catalog,String schemaPattern,String tableNamePattern,String columnNamePattern\);

   ```
   获取表列信息
   ```

   ​

   ​




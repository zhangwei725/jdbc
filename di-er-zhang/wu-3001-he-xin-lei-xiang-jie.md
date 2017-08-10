# 核心类详解

## 一、PerparedStatement

### 1、说明

PerparedStatement是Statment的子类,在任何时候都不要使用Statement

### 2 、为什么要使用PerparedStatemnt?

#### 2.1、极大地提高了安全性

​    Statemnt会引起SQL注入问题,SQL注入是用户利用某些系统没有对输入数据进行充分的检查，从而进行恶意破坏的行为。但安全是相对的不是绝对的。

​

#### 2.2、PreparedStatement尽最大可能提高性能.

​    数据库都会尽最大努力对预编译语句提供最大的性能优化.因为预编译语句有可能被重复调用.所以语句在被DB的编译器编译后的执行代码被缓存下来,那么下次调用时只要是相同的预编译语句就不需要编译,只要将参数直接传入编译过的语句执行代码中,就会得到执行.这并不是说只有一个 Connection中多次执行的预编译语句被缓存,而是对于整个DB中,只要预编译的语句语法和缓存中匹配.那么在任何时候就可以不需要再次编译而可以 直接执行.而statement的语句中,即使是相同一操作,而由于每次操作的数据不同所以使整个语句相匹配的机会极小,几乎不太可能匹配

#### 2.3、代码的可读性和可维护性

​    SQL语句中可以包含?占位符,可以把?替换成变量

#### 2.4、获取自动生成的键值

​    JDBC3.0规范中的一个可选特性提供了一种能力, 可以取得刚刚插入到表中的记录的自动生成的键值

```
int rowcount = stmt.executeUpdate("insert into tb_user (username) values ('?')",       
// 插入行并返回键值     
Statement.RETURN_GENERATED_KEYS);     
// 得到生成的键值     
 ResultSet rs = stmt.getGeneratedKeys ();
```

#### 2.5、使用PreparedStatement的Batch功能

​    Update或INSERT大量的数据时, 可以使用PreparedStatement的AddBatch\(\)方法一次性发送多个查询给数据库

```
PreparedStatement ps = conn.prepareStatement(     
   "INSERT INTO TB_USER(USER_ID,USERNAME,PASSWORD,) values (SEQ_TBUSER_USERID,?, ?)");     
for (n = 0; n < 100; n++) {     
  ps.setString(1,"HEHE");  
  ps.setString(2,"123");
  ps.addBatch(); 
}     
ps.executeBatch();
```

## 二、Result详解

### 1、说明

​    SELECT 语句执行完成后返回的数据结果集\(包括行跟列\)

### 2、 结构示意图

![](http://opzv089nq.bkt.clouddn.com/17-8-10/37474997.jpg)

### 3、遍历结果集

1. 通过while循环遍历\(推荐\)

   ```
   while (rs.next()) {
      System.out.println(rs.getInt(1));
   }
   ```

2. 通过for循环遍历

   ```
   PreparedStatement ps =connection.prepareStatement(sql,
                                    ResultSet.TYPE_SCROLL_INSENSITIVE,
                                    ResultSet.CONCUR_READ_ONLY)
   for (resultSet.first(); !resultSet.isAfterLast(); resultSet.next()) {
                   int id = resultSet.getInt(1);
                   System.out.println(id);
               }
   ```

### 4、ResultSet滚动结果集

#### 4.1、 说明

​    ResultSet还提供了对结果集进行滚动和更新的方法。若想设置可滚动的结果集，则在创建Statement对象时，不能像前文那样调用无参方法

#### 4.2、相关方法说明

1. 常量API

   TYPE\_FORWARD\_ONLY ：该常量指示指针只能向前移动的 ResultSet 对象的类型。  
   TYPE\_SCROLL\_INSENSITIVE ：该常量指示可滚动但通常不受其他的更改影响的 ResultSet 对象的类型。  
   TYPE\_SCROLL\_SENSITIVE：该常量指示可滚动并且通常受其他的更改影响的 ResultSet 对象的类型。

   CONCUR\_READ\_ONLY：指示该结果集的指针只能移动不能修改结果集

   CONCUR\_UPDATABLE：指示ResultSet对象可以执行数据库的新增、修改、和移除

2. 滚动结果集相关方法

   next\(\)：移动到下一行

   previous\(\)：移动到前一行

   absolute\(int row\)：移动到指定行

   beforeFirst\(\)：移动resultSet的最前面

   afterLast\(\) ：移动到resultSet的最后面

   updateString\(int columnIndex, String x\) ：用 String 值更新指定列。

   updateString\(String columnLabel, String x\) ：用 String 值更新指定列。

   updateRow\(\) ：更新行数据，最后要调用这个方法来确认

#### 4.3、示例d代码：

```
// 设置可滚动结果集
public void scrollResult() throws Exception {

  Class.forName("com.mysql.jdbc.Driver");
  Connection conn = DriverManager.getConnection("jdbc:mysql:///test",
          "root", "root");
  String sql = select * from tb_user"
  // 参数：TYPE_SCROLL_SENSITIVE 可滚动 、CONCUR_UPDATABLE 可修改
   PreparedStatement ps = conn.createStatement(sql,ResultSet.TYPE_SCROLL_SENSITIVE,
          ResultSet.CONCUR_UPDATABLE);
  ResultSet rs = stmt.executeQuery();
  // 游标移动到rs的第三行
  rs.absolute(3);
  // 更新第四列的值
  rs.updateString(4, "萝拉");
  // 确认动作
  rs.updateRow();
  rs.close();
  stmt.close();
  conn.close();
 }
```

可以用以下两种方式使用更新方法：

更新当前行中的列值。在可滚动的 ResultSet 对象中，可以向前和向后移动光标，将其置于绝对位置或相对于当前行的位置。以下代码片段更新 ResultSet 对象 rs 第五行中的 NAME 列，然后使用方法 updateRow 更新导出 rs 的数据源表。

```
 rs.absolute(5);
 rs.updateString("NAME", "玛利亚"); 
 rs.updateRow();
```

​    将列值插入到插入行中。可更新的 ResultSet 对象具有一个与其关联的特殊行，该行用作构建要插入的行的暂存区域 \(staging area\)。以下代码片段将光标移动到插入行，构建一个三列的行，并使用方法 insertRow 将其插入到 rs 和数据源表中。

```
 rs.moveToInsertRow();
 rs.updateString(1, "xiaoming");
 rs.updateInt(2,35); 
 rs.insertRow();
 rs.moveToCurrentRow();
```

#### 4.4、注意事项

1. 能否使用可更新结果集，要看使用的数据库驱动是否支持，
2. 还有只能用于单表且表中有主键字段（可能会是联合主键），不能够有表连接，会取所有非空字段且没有默认值。
3. 能否使用JDBC2.0 ResultSet的新特性要看数据库驱动程序是否支持。




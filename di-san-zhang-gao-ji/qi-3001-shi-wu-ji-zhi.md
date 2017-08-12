# 数据库事务\(Database Transaction\)

## 一、简介

​    事务\(Transaction\):是并发控制的单元，是用户定义的一个操作序列。这些操作要么都做，要么都不做，是一个不可分割的工作单位。通过事务，sql  能将逻辑相关的一组操作绑定在一起，以便服务器保持数据的完整性。事务通常是以begin transaction开始，以commit或rollback结束。Commint表示提交，即提交事务的所有操作。具体地说就是将事务中所有对数据的更新写回到磁盘上的物理数据库中去，事务正常结束。Rollback表示回滚，即在事务运行的过程中发生了某种故障，事务不能继续进行，系统将事务中对数据库的所有已完成的操作全部撤消，滚回到事务开始的状态

设想网上购物的一次交易，其付款过程至少包括以下几步数据库操作：

1. 更新客户所购商品的库存信息 
2. 保存客户付款信息--可能包括与银行系统的交互 
3. 生成订单并且保存到数据库中
4. 更新用户相关信息，例如购物数量等等  

正常的情况下，这些操作将顺利进行，最终交易成功，与交易相关的所有数据库信息也成功地更新。但是，如果在这一系列过程中任何一个环节出了差错，例如在更新商品库存信息时发生异常、该顾客银行帐户存款不足等，都将导致交易失败。一旦交易失败，数据库中所有信息都必须保持交易前的状态不变，比如最后一步更新用户信息时失败而导致交易失败，那么必须保证这笔失败的交易不影响数据库的状态--库存信息没有被更新、用户也没有付款，订单也没有生成。否则，数据库的信息将会一片混乱而不可预测。

数据库事务正是用来保证这种情况下交易的平稳性和可预测性的技术

JDBC连接是在默认情况下会自动提交，也就是说每个SQL语句都是在其完成时提交到数据库。

事务只针对DML操作

## 二、事务的特性\(ACID\)

1. 举个栗子:小明账户向小花账号汇钱的例子来说明如何通过数据库事务保证数据的准确性和完整性

   1、从小明账号中把余额读出来（1000）  
   2、对小明账号做减法操作（1000-100）  
   3、把结果写回小明账号中（900）  
   4、从小花账号中把余额读出来（500\)  
   5、对小花账号做加法操作（500+100）  
   6、把结果写回小花账号中（600）

### 2.1、原子性（atomicity）

1. 概念

   事务是数据库的逻辑工作单位，而且是必须是原子工作单位，对于其数据修改，要么全部执行，要么全部不执行。

2. 说明

   ```
   保证1-6所有过程要么都执行，要么都不执行。一旦在执行某一步骤的过程中发生问题，就需要执行回滚操作。 
   假如执行到第五步的时候，B账户突然不可用（比如被注销），那么之前的所有操作都应该回滚到执行事务之前的状态
   ```

### 2.2、一致性（consistency）

1. 概念

   事务在完成时，必须是所有的数据都保持一致状态。在相关数据库中，所有规则都必须应用于事务的修改，以保持所有数据的完整性。

2. 说明

   ```
   在转账之前，小明和小花的账户中共有1000+500=1500元钱。在转账之后，小明和小花的账户中共有900+600=1500元。
   也就是说，数据的状态在执行该事务操作之后从一个状态改变到了另外一个状态。同时一致性还能保证账户余额不会变成负数等
   ```

### 2.3、隔离性（isolation）

1. 概念

   也称为独立性，是指并行事务的修改必须与其他并行事务的修改相互独立。一个事务处理数据，要么是其他事务执行之前的状态，要么是其他事务执行之后的状态，但不能处理其他正在处理的数据。  
   企业级的数据库每一秒钟都可能应付成千上万的并发访问，因而带来了并发控制的问题。

2. 说明

   ```
   在小明向小花转账的整个过程中，只要事务还没有提交（commit），查询小明账户和小花账户的时候，两个账户里面的钱的数量都不会有变化。
   ```

### 2.4、持久性（durability）

1. 概念

   一个事务一旦提交，事物的操作便永久性的保存在DB中。即使此时再执行回滚操作也不能撤消所做的更改。

2. 说明

   ```
   一旦转账成功（事务提交），两个账户的里面的钱就会真的发生变化（会把数据写入数据库做持久化保存）
   ```

## 二、JDBC标准事务编程

### 2.1、事务控制的基本语句及功能

1. 提交事务            \(commit\)
2. 回滚事务           \(rollback\)
3. 设置保存点        \(savepoint\)
4. 回退到保存点         \(rolback to savepoint\)

### 2.2、示例代码

1. 建表语句

   ```mysql
   CREATE TABLE "TB_USER" (
   "USER_ID" NUMBER(4) NOT NULL ,
   "NAME" VARCHAR2(32 BYTE) NULL ,
   "MONEY" NUMBER(10,2) NULL 
   );
   INSERT INTO "TB_USER" VALUES (1, 'xm', 500);
   INSERT INTO "TB_USER" VALUES (2, 'xh', 500);
   ```

2. 工具类

   ```java
   public class DbManager {
       private static volatile DbManager instance = null;
       static ComboPooledDataSource ds = null;

       static {
           ds = new ComboPooledDataSource("mysql");
       }

       private DbManager() {
       }

       public static DbManager getInstance() {
           if (instance == null) {
               synchronized (DbManager.class) {
                   if (instance == null) {
                       instance = new DbManager();
                   }
               }
           }
           return instance;
       }

       public Connection getConnection() {
           Connection connection = null;
           try {
               connection = ds.getConnection();
           } catch (SQLException e) {
               e.printStackTrace();
           }
           return connection;
       }

       public void close(Statement statement, ResultSet rs) {
           if (rs != null) {
               try {
                   rs.close();
               } catch (SQLException e) {
                   e.printStackTrace();
               }finally{
                 close(statement;);
               }
           }
       }

       public void close(Statement statement) {
           if (statement != null) {
               try {
                   statement.close();
               } catch (SQLException e) {
                   e.printStackTrace();
               }
           }
       }

   }
   ```

3. 示例代码1

   ```java
   public static void testTransaction() {        
           Connection connection = null;
             PreparedStatement ps2= null;
           PreparedStatement ps1= null;
             PreparedStatement ps3= null;
           PreparedStatement ps4= null;
           try {
               connection =ConnPoolManager.getConn();
               connection.setAutoCommit(false);
               String sql1 = "SELECT  MONEY  FROM  TB_USER  WHERE  name =?";
               ps1 = connection.prepareStatement(sql1);
               ps1.setString(1, "xm");
               ResultSet rs = ps1.executeQuery();
               double money = 0;
               while (rs.next()) {
                   money = rs.getDouble(1);
               }
               money -= 100;
               String sql2 = "UPDATE   TB_USER   SET   MONEY = ? WHERE  NAME = ?";
               ps2 = connection.prepareStatement(sql2);
               ps2.setDouble(1, money);
               ps2.setString(2, "xm");
               ps2.executeUpdate();

               String sql3 = "SELECT MONEY  FROM  TB_USER  WHERE  USER_ID=?";
               ps3 = connection.prepareStatement(sql3);
               ps3.setInt(1, 2);
               ResultSet rs1 = ps3.executeQuery();

               double wifiMoney = 0;
               while (rs1.next()) {
                   wifiMoney = rs1.getDouble(1);
               }
               wifiMoney += 100;
               ps4 = connection.prepareStatement(sql2);
               ps4.setDouble(1, wifiMoney);
               ps4.setString(2, "xh");
               int i = ps4.executeUpdate();
               connection.commit();

           } catch (SQLException e) {
               try {
                   connection.rollback();
               } catch (SQLException e1) {
                   e1.printStackTrace();
               }
           }finally{
              DbManager.getInstance().close(ps1);
              DbManager.getInstance().close(ps2);
              DbManager.getInstance().close(ps3);
              DbManager.getInstance().close(ps4);
           }
   }
   ```

4. 示例代码2\(设置回滚点\)

   ```java
      public static void startTransactionSavePoint() {
           Connection connection = DbManager.getInstance().getConnection();
           PreparedStatement ps = null;
           PreparedStatement updatePs = null;
           PreparedStatement deletePs = null;
           String insertSql = "INSERT INTO TB_USER(NAME, MONEY)VALUES ('laowang', 1000)";
           String updateSql = "UPDATE TB_USER SET MONEY= ? WHERE USER_ID=? ";

           String deleteSql = "DELETE FROM TB_USER WHERE NAME=?";

           try {
               //关闭自动事务提交
               connection.setAutoCommit(false);

               ps = connection.prepareStatement(insertSql);
               int count = ps.executeUpdate();

               Savepoint point = connection.setSavepoint("point");
               //保存点开始,删除操作
               deletePs = connection.prepareStatement(deleteSql);
               deletePs.setString(1, "laowang");
               deletePs.executeUpdate();
               //更新操作
               updatePs = connection.prepareStatement(updateSql);
               updatePs.setDouble(1, 100001.00);
               updatePs.setInt(2, 3);
               updatePs.executeUpdate();

               //回滚到保存到指定保存点
               connection.rollback(point);
               //提交事务
               connection.commit();

               if (count > 0) {
                   System.out.println("添加用户成功!!!");
               }

           } catch (SQLException e) {
                  connection.rollback();
               e.printStackTrace();
           } finally {
               //关闭ps
               DbManager.getInstance().close(ps);
               DbManager.getInstance().close(updatePs);
               DbManager.getInstance().close(deletePs);
           }

       }​
   ```

# 三、事务隔离 - 并发控制

### 3.1、并发控制

​    数据库管理系统（DBMS）中的并发控制的任务是确保在多个事务同时存取数据库中同一数据时不破坏事务的隔离性和统一性以及数据库的统一性

### 3.2、不考虑事务的隔离性，会出现什么问题？

1. 脏读：一个事务读取到另一个事务的未提交数据
2. 不可重复读：两次读取的数据不一致\(强调update\)
3. 虚读\(幻读\)：两次读取的数据不一致\(强调insert\)
4. 丢失更新：两个事务对同一条记录进行操作，后提交的事务，将先提交的事务的修改覆盖了

### 3.3、四种隔离级别

1. Read uncommitted：最低级别，以上情况均无法保证。\(读未提交\)

2. Read committed：可避免脏读情况发生（读已提交）

3. Repeatable read：可避免脏读、不可重复读情况的发生。（可重复读）不可以避免虚读

4. Serializable：可避免脏读、不可重复读、虚读情况的发生。\(序列化，不仅有read、write锁还有range lock范围锁（没有where锁全表，有where锁where范围）；对一张表的所有增删改操作必须顺序执行，性能最差\)

   | 隔离级别 | 脏读 | 不可重复读 | 幻读 |
   | --- | --- | --- | --- |
   | Read uncommitted | √ | √ | √ |
   | Read committed | × | √ | √ |
   | Repeatable read | × | × | √ |
   | Serializable | × | × | × |

##### 1、Read uncommitted\(读未提交\)

1. 举个栗子

   ```
   又到月底了,小明的老婆要准备给小明发生活费了,小明的老婆给小明打了550块,但改事务并没有提交,而此时小明正好在查余额,发现是550块,高兴的差点蹦了起来.天有不测风云,突然小明的老婆发现多打了50块,于是回滚事务,修改金额,然后将事务提交,最后小明空欢喜异常。
   ```

2. 示例图

   ![](http://opzv089nq.bkt.clouddn.com/17-8-12/22982577.jpg)

##### 2、Read committed \(读提交,不可重复读\)

1. 举个栗子

   ```
   某个夜黑风高的夜晚，小明丰富的夜生活开始了,小明拿着工资卡去消费,pos机读取卡的信息的时候有500，
   而此时小红也正好在网上转账，把小明工资卡的500元转到另一账户，并小明之前提交了事务，当小明扣款时，
   系统检查到小明的工资卡已经没有钱，扣款失败，小明十分纳闷，明明卡里有钱,为什么会说余额不足,
   出现上述情况，即我们所说的不可重复读，两个并发的事务，“事务1：小明消费”、“事务2：小红网上转账”，事务1事先读取了数据，
   事务2紧接了更新了数据，并提交了事务，而事务1再次读取该数据时，数据已经发生了改变,
   当隔离级别设置为Read committed时，避免了脏读，但是可能会造成不可重复读
   ```

2. 示意图

   ![](http://opzv089nq.bkt.clouddn.com/17-8-12/63597271.jpg)

3. 备注

   ```
   Sql Server ,Oracle的默认级别
   ```

##### 3、Repeatable read 幻读

1. 举例说明

   ```
   小红最近发现小明总是很晚回家并且经常不接电话,于是小红开始查小明当月信用卡的总消费金额，
   消费金额为50,而小明此时正好在收银台买单，消费1000元，即新增了一条1000元的消费记录,并提交了事务，
   随后小红将小明当月信用卡消费的明细打印了出来，却发现消费总额为1050元，小红很诧异，以为出现了幻觉
   ```

2. 示例图

   ​    ![](http://opzv089nq.bkt.clouddn.com/17-8-12/40790591.jpg)

3. 备注

   ```
   MySQL的默认隔离级别
   ```

##### 4、Serializable 序列化

1. 说明

   ```
   最高级别：防止上述3种情况，事务串行执行，慎用
   这是最高的隔离级别，它通过强制事务排序，使之不可能相互冲突，从而解决不读脏，可重复读，不可幻读。简言之，它是在每个读的数据行上加上共享锁。在这个级别，可能导致大量的超时现象和锁竞争
   ```

5、更新丢失问题

1. 说明

   ```
   由于RDBMS都有锁机制，所以在并发事务中不存在更新丢失问题
   1>加锁阶段：在该阶段可以进行加锁操作。在对任何数据进行读操作之前要申请并获得S锁，在进行写操作之前要申请2>并获得X锁。加锁不成功，则事务进入等待状态，直到加锁成功才继续执行。
   解锁阶段：当事务释放了一个封锁以后，事务进入解锁阶段，在该阶段只能进行解锁操作不能再进行加锁操作
   ```

### 3.4、如何设置事务的隔离级别?

1. Connection.TRANSACTION\_READ\_UNCOMMITTED、
2. Connection.TRANSACTION\_READ\_COMMITTED、
3. Connection.TRANSACTION\_REPEATABLE\_READ
4. Connection.TRANSACTION\_SERIALIZABLE。

```
try {
    connection.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
    connection.setAutoCommit(false);
    connection.commit();
    } catch (SQLException e) {
    connection.rollback();
    e.printStackTrace();
}
```

### 3.5、示例代码




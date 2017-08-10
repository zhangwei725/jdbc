# JDBC连接管理

## 一、基本原理

​    在Java语言中，JDBC（Java DataBase Connection）是应用程序与数据库沟通的桥梁, 即Java语言通过JDBC技术访问数据库。JDBC是一种“开放”的方案，它为数据库应用开发人员，数据库前台工具开发人员提供了一种标准的应用程序设计接口，使开发人员可以用纯Java语言编写完整的数据库应用程序。

​    JDBC作为一种数据库访问技术，具有简单易用的优点。但使用这种模式进行Web应用程序开发，存在很多问题：首先，每一次Web请求都要建立一次数据库连接。建立连接是一个费时的活动，每次都得花费0.05s～1s的时间，而且系统还要分配内存资源。这个时间对于一次或几次数据库操作，或许感觉不出系统有多大的开销。可是对于现在的Web应用，尤其是大型电子商务网站，同时有几百人甚至几千人在线是很正常的事。在这种情况下，频繁的进行数据库连接操作势必占用很多的系统资源，网站的响应速度必定下降，严重的甚至会造成服务器的崩溃。不是危言耸听，这就是制约某些电子商务网站发展的技术瓶颈问题。其次，对于每一次数据库连接，使用完后都得断开。否则，如果程序出现异常而未能关闭，将会导致数据库系统中的内存泄漏，最终将不得不重启数据库。还有，这种开发不能控制被创建的连接对象数，系统资源会被毫无顾及的分配出去，如连接过多，也可能导致内存泄漏，服务器崩溃。

缺点总结  
用户每次请求都需要向数据库获得链接，而数据库创建连接通常需要消耗相对较大的资源，创建时间也较长。

假设网站一天10万访问量，数据库服务器就需要创建10万次连接，极大的浪费数据库的资源，并且极易造成数据库服务器内存溢出、拓机

## 二 、什么是数据库连接池

​    所谓数据库连接池，可以看作 ：在用户和数据库之间创建一个”池”，这个池中有若干个连接对象，当用户想要连接数据库，就要先从连接池中获取连接对象，然后操作数据库。一旦连接池中的连接对象被拿光了，下一个想要操作数据库的用户必须等待，等待其他用户释放连接对象，把它放回连接池中，这时候等待的用户才能获取连接对象，从而操作数据库。

## 三、数据库连接池技术的优点

### 1.资源重用：

​    由于数据库连接得以重用，避免了频繁创建，释放连接引起的大量性能开销。在减少系统消耗的基础上，另一方面也增加了系统运行环境的平稳性。

### 2.更快的系统反应速度

​    数据库连接池在初始化过程中，往往已经创建了若干数据库连接置于连接池中备用。此时连接的初始化工作均已完成。对于业务请求处理而言，直接利用现有可用连接，避免了数据库连接初始化和释放过程的时间开销，从而减少了系统的响应时间

### 3.新的资源分配手段

​    对于多应用共享同一数据库的系统而言，可在应用层通过数据库连接池的配置，实现某一应用最大可用数据库连接数的限制，避免某一应用独占所有的数据库资源

### 4.统一的连接管理，避免数据库连接泄露

​    在较为完善的数据库连接池实现中，可根据预先的占用超时设定，强制回收被占用连接，从而避免了常规数据库连接操作中可能出现的资源泄露

## 四、开源连接池

​    实现目前有众多的开源连接池实现，它们大多数是比较独立的组件，经过一定的配置就可以在我们的应用中使用。下面是三个用得比较多的连接池：

### 1、DataSource

```
通常被称为数据源，它包含连接池和连接池管理两个部分，习惯上也经常把 DataSource 称为连接池
```

### 2、druid

```
 阿里巴巴的开源连接池技术。
```

### 3、C3P0

```
它是随着hibernate一起发展起来的数据库连接池
```

### 4、Proxool

```
它是一个Java SQL Driver驱动程序，提供了各种JDBC驱动的连接池封装。
```

## 五、各种连接池实现

### 5.1、自己实现\(了解\)

1. 方式一\(核心代码\)

   ```
   package com.werner.entity;
   import java.lang.reflect.InvocationHandler;
   import java.lang.reflect.Method;
   import java.lang.reflect.Proxy;
   import java.sql.Connection;
   import java.sql.DriverManager;
   import java.sql.SQLException;
   import java.util.LinkedList;

   /**
    * 1. CustomConnPool.java 连接池类，
    * 2. 指定全局参数： 初始化数目、最大连接数、当前连接、 连接池集合
    * 3. 构造函数：循环创建3个连接
    * 4. 写一个创建连接的方法
    * 5. 获取连接
    * 5.1 判断： 池中有连接， 直接拿
    * 5.2 池中没有连接，判断，是否达到最大连接数； 达到，抛出异常；没有达到最大连接数，
    * 创建新的连接
    * 6. 释放连接
    * 连接放回集合中(..)
    */
   public class CustomConnPool {
       /**
        * 初始化连接数目
        */
       private static final int INIT_COUNT = 5;
       /**
        * 最大连接数
        */
       private static final int MAX_COUNT = 10;
       /**
        * 记录当前使用连接数
        */
       private int current_count = 0;
       /**
        * 存放所有的初始化连池
        */

       private LinkedList<Connection> pool = new LinkedList<>();

       //1. 构造函数中，初始化连接放入连接池
       public CustomConnPool() {
           // 初始化连接
           for (int i = 0; i < INIT_COUNT; i++) {
               // 记录当前连接数目
               current_count++;
               // 创建原始的连接对象
               Connection con = createConnection();
               // 把连接加入连接池
               pool.addLast(con);
           }
       }

       //2. 创建一个新的连接的方法
       private Connection createConnection() {
           try {
               Class.forName("com.mysql.jdbc.Driver");
               // 原始的目标对象
               final Connection con = DriverManager.getConnection("jdbc:mysql:///test", "root", "root");

               // 对con创建其代理对象
               Connection proxy = (Connection) Proxy.newProxyInstance(
                       con.getClass().getClassLoader(), // 类加载器
                       new Class[]{Connection.class}, // 目标对象实现的接口
                       new InvocationHandler() {    // 当调用con对象方法的时候， 自动触发事务处理器
                           public Object invoke(Object proxy, Method method, Object[] args)
                                   throws Throwable {
                               // 方法返回值
                               Object result = null;
                               // 当前执行的方法的方法名
                               String methodName = method.getName();

                               // 判断当执行了close方法的时候，把连接放入连接池
                               if ("close".equals(methodName)) {
                                   System.out.println("begin:当前执行close方法开始！");
                                   // 连接放入连接池 (判断..)
                                   pool.addLast(con);
                                   System.out.println("end: 当前连接已经放入连接池了！");
                               } else {
                                   // 调用目标对象方法
                                   result = method.invoke(con, args);
                               }
                               return result;
                           }
                       }
               );
               return proxy;
           } catch (Exception e) {
               throw new RuntimeException(e);
           }
       }

       //3. 获取连接
       public Connection getConnection() {
           // 3.1 判断连接池中是否有连接, 如果有连接，就直接从连接池取出
           if (pool.size() > 0) {
               return pool.removeFirst();
           }
           // 3.2 连接池中没有连接： 判断，如果没有达到最大连接数，创建；
           if (current_count < MAX_COUNT) {
               // 记录当前使用的连接数
               current_count++;
               // 创建连接
               return createConnection();
           }
           // 3.3 如果当前已经达到最大连接数，抛出异常
           throw new RuntimeException("当前连接已经达到最大连接数目 ！");
       }
         //4. 释放连接
      public void realease(Connection con) {
          // 4.1 判断：池的数目如果小于初始化连接，就放入池中
          if (pool.size() < INIT_COUNT) {
              pool.addLast(con);
          } else {
              try {
                  //4.2 关闭
                  current_count--;
                  con.close();
              } catch (SQLException e) {
                  throw new RuntimeException(e);
              }
          }
      }
   }
   ```

2. 方式二\(核心代码\)

   ```
   public class ConnPoolDataSource  implements DataSource{
       // 存储Connection对象的集合.
          private List<Connection> conns = new LinkedList<Connection>();

          public ConnPoolDataSource(String properties) throws IOException,
                  ClassNotFoundException, SQLException {
              // 连接池中Connection对象的数量
              int initialSize = 10;
              // 根据配置文件的名称获得InputStream对象
              InputStream in = ConnPoolDataSource.class.getClassLoader()
                      .getResourceAsStream(properties);
              // 使用Properties读取配置文件
              Properties prop = new Properties();
              prop.load(in);
              String url = prop.getProperty("jdbc_url");
              String user = prop.getProperty("username");
              String password = prop.getProperty("password");
              String driver = prop.getProperty("driver_class");

              // 加载数据库驱动
              Class.forName(driver);
              // 根据配置文件中的信息获取数据库连接
              for (int i = 0; i < initialSize; i++) {
                  Connection conn = DriverManager.getConnection(url, user, password);
                  conns.add(conn);
              }
          }

          @Override
          public Connection getConnection() throws SQLException {
              // 获取从集合中移除的Connection对象, 并将其包装为MyConnection对象后返回.
              Connection conn = conns.remove(0);
              // 测试代码
              System.out.println("获取连接之后, 连接池中的Connection对象的数量为: " + conns.size());
              return new CustomConnection(conn, conns);
          }

          private class CustomConnection implements Connection {
              private Connection conn;
              private List<Connection> conns;

              public CustomConnection(Connection conn, List<Connection> conns) {
                  this.conn = conn;
                  this.conns = conns;
              }

              // 重写close()方法, 实现调用该方法时将连接归还给连接池
              @Override
              public void close() throws SQLException {
                  conns.add(conn);
              }
              @Override
              public void clearWarnings() throws SQLException {
                  conn.clearWarnings();
              }

              @Override
              public void commit() throws SQLException {
                  conn.commit();
              }
          }
      }
   }
   ```

### 5.2、开源框架c3p0

1. 导包\(c3p0-x.x.x.x.jar , mchange-commons-java-x.x.xx,数据库驱动包\)

   ```
   使用第三方技术必须导包
   ```

2. 配置c3p0-config.xml

   ```
   <c3p0-config>
       <!-- 默认配置 -->
       <default-config>
           <property name="jdbcUrl">jdbc:mysql://localhost:3306/jshop</property>
           <property name="user">root</property>
           <property name="password">root</property>
           <property name="driverClass">com.mysql.jdbc.Driver</property>
           <property name="initialPoolSize">5</property>
           <property name="maxPoolSize">8</property>
           <property name="checkoutTimeout">3000</property>
       </default-config>

       <!-- mysql的连接配置 -->
       <named-config name="mysql">
           <property name="jdbcUrl">jdbc:mysql://localhost:3306/test</property>
           <property name="user">root</property>
           <property name="password">root</property>
           <property name="driverClass">com.mysql.jdbc.Driver</property>
           <property name="initialPoolSize">5</property>
           <property name="maxPoolSize">8</property>
           <property name="checkoutTimeout">3000</property>
       </named-config>

       <!-- oracle配置-->
       <named-config name="oracle">
           <property name="jdbcUrl">jdbc:oracle:thin:@127.0.0.1:orcl</property>
           <property name="user">root</property>
           <property name="password">root</property>
           <property name="driverClass">oracle.jdbc.driver.OracleDriver</property>
           <property name="initialPoolSize">5</property>
           <property name="maxPoolSize">8</property>
           <property name="checkoutTimeout">3000</property>
       </named-config>
   ```

3. 配置详解

   ```
   <named-config name="test">
          <!--连接池中保留的最大连接数。默认值: 15 -->
          <property name="maxPoolSize">20</property>
          <!-- 连接池中保留的最小连接数，默认为：3-->
          <property name="minPoolSize">2</property>
          <!-- 初始化连接池中的连接数，取值应在minPoolSize与maxPoolSize之间，默认为3-->
          <property name="initialPoolSize">2</property>
          <!--最大空闲时间，60秒内未使用则连接被丢弃。若为0则永不丢弃。默认值: 0 -->
          <property name="maxIdleTime">60</property>
          <!-- 当连接池连接耗尽时，客户端调用getConnection()后等待获取新连接的时间，超时后将抛出SQLException，如设为0则无限期等待。单位毫秒。默认: 0 -->
          <property name="checkoutTimeout">3000</property>
          <!--当连接池中的连接耗尽的时候c3p0一次同时获取的连接数。默认值: 3 -->
          <property name="acquireIncrement">2</property>
          <!--定义在从数据库获取新连接失败后重复尝试的次数。默认值: 30 ；小于等于0表示无限次-->
          <property name="acquireRetryAttempts">0</property>
          <!--重新尝试的时间间隔，默认为：1000毫秒-->
          <property name="acquireRetryDelay">1000</property>
          <!--关闭连接时，是否提交未提交的事务，默认为false，即关闭连接，回滚未提交的事务 -->
          <property name="autoCommitOnClose">false</property>
          <!--c3p0将建一张名为Test的空表，并使用其自带的查询语句进行测试。如果定义了这个参数那么属性preferredTestQuery将被忽略。你不能在这张Test表上进行任何操作，它将只供c3p0测试使用。默认值: null -->
          <property name="automaticTestTable">Test</property>
          <!--如果为false，则获取连接失败将会引起所有等待连接池来获取连接的线程抛出异常，但是数据源仍有效保留，并在下次调用getConnection()的时候继续尝试获取连接。如果设为true，那么在尝试获取连接失败后该数据源将申明已断开并永久关闭。默认: false-->
          <property name="breakAfterAcquireFailure">false</property>
          <!--每60秒检查所有连接池中的空闲连接。默认值: 0，不检查 -->
          <property name="idleConnectionTestPeriod">60</property>
          <!--c3p0全局的PreparedStatements缓存的大小。如果maxStatements与maxStatementsPerConnection均为0，则缓存不生效，只要有一个不为0，则语句的缓存就能生效。如果默认值: 0-->
          <property name="maxStatements">100</property>
          <!--maxStatementsPerConnection定义了连接池内单个连接所拥有的最大缓存statements数。默认值: 0 -->
          <property name="maxStatementsPerConnection">0</property>
   ```

   注意事项

   1、c3p0-config.xml 如果java项目直接放在src根目录下

   2、xml文件命名为c3p0-config固定格式

4. java代码

   ```
   public class ConnPoolManager {
      private static volatile ConnPoolManager instance = null;
      static ComboPooledDataSource ds = null;

      static {
          ds = new ComboPooledDataSource("mysql");
      }

      private ConnPoolManager() {
      }

      public static ConnPoolManager getInstance() {
          if (instance == null) {
              synchronized (ConnPoolManager.class) {
                  if (instance == null) {
                      instance = new ConnPoolManager();
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
              }
          }

          if (statement != null) {
              try {
                  statement.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
      }
      public static void main(String[] args) {
          Connection connection = ConnPoolManager.getInstance().getConnection();
          String sql = "SELECT * FROM  t_product";
          PreparedStatement ps = null;
          ResultSet rs = null;
          try {
              ps = connection.prepareStatement(sql);
              rs = ps.executeQuery();
              while (rs.next()) {
                  System.out.println(rs.getInt(1));
              }
          } catch (SQLException e) {
              e.printStackTrace();
          } finally {
              ConnPoolManager.getInstance().close(ps, rs);
          }
      }
    }
   ```

5. [官方文档](http://www.mchange.com/projects/c3p0/index.html)




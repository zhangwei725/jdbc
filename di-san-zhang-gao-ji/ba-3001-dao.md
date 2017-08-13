# DAO开发

## 一、什么是DAO

​    数据访问对象（Data Access Object）,是标准 J2EE 设计模式之一。开发人员用这种模式将底层数据访问操作与高层业务逻辑分离开。

## 二、四要素

1. 一个 DAO 工厂类
2. 一个 DAO 接口
3. 一个实现了 DAO 接口的具体类
4. 数据传输对象\(有时称为值对象\)

## 三、具体类

1. DAO接口

   ```
   用于声明对于数据库的操作
   ```

2. VO\(Value Object\):

   ```
   于存放一行数据即一条记录的类
   ```

3. DAOImpl

   ```
   必须实现DAO接口，真实实现DAO接口的函数，但是不包括数据库的打开和关闭
   ```

4. DAOFactory

   ```
   工厂类，含有getInstance()创建一个DAO类
   ```

## 四、分层开发

### 4.1、特点

1. 每一层都有自己的职责
2. 上一层不用关心下一层的实现细节，上一层通过下一层提供的对外接口来使用其功能
3. 上一层调用下一层的功能，下一层不能调用上一层功能

### 4.2、好处

1. 层专注于自己功能的实现，便于提高质量
2. 便于分工协作，提高开发效率
3. 便于代码复用
4. 便于程序扩展

## 五、编写DAO

1. 建立数据库表

2. 创建实体类

   ```
   1. java类必须要实现java.io.Serializable接口；
   2. 类中的字段必须使用private封装，封装好的属性必须提供get和set方法
   3. 类中的属性不允许使用基本数据类型，都必须使用基本数据类型的包装类。原因是基本数据类型的默认值是0，而包装类的默认数据类型是null
   4. 类中必须保留无参构造方法
   6. java类的名称尽量与表名称保持一致。表名称如果是这种TB_USER类名称为USER
   ```

3. 创建DAO的基类（接口类）

4. 创建DAO的实现类

5. 创建具体表的DAO类

6. 创建具体表的DAO类的实现类

7. 创建DAO类工厂类

## 六、示例代码

### 6.1、基础入门

1. 建表语句\(\)

   ```MYSQL
   --Mysql
   CREATE  TABLE   MOBILE_TELEPHONE(
       MID INT(6)  AUTO_INCREMENT PRIMARY KEY, --主键自动增长
       BRAND VARCHAR(50) NOT NULL, --品牌
       MODEL VARCHAR(50) NOT NULL,--型号
       PRICE  DOUBLE(9,2) NOT NULL, --价格
       COUNT  INT NOT NULL,    --数量
       VERSION   VARCHAR(50) NOT NULL,--版本
       CONSTRAINT UK_MT_BRAND  UNIQUE(BRAND)
   )
   -- oracle
   CREATE  TABLE   MOBILE_TELEPHONE(
       MID NUMBER(6)  PRIMARY KEY, --主键
       BRAND VARCHAR2(50) NOT NULL, 
       MODEL VARCHAR2(50) NOT NULL,
       PRICE  NUMBER(9,2) NOT NULL, 
       COUNT  NUMBER(8) NOT NULL,    
       VERSION   VARCHAR(50) NOT NULL,
       CONSTRAINT "UK_MT_BRAND" UNIQUE(BRAND)
   )
   --索引 主键自动增长
   CREATE SEQUENCE SEQ_MT_MID  --序列名
   INCREMENT BY 1   -- 每次加几个  
   START WITH 1       -- 从1开始计数  
   NOMAXVALUE        -- 不设置最大值  
   NOCYCLE               -- 一直累加，不循环  
   NOCACHE;  --不缓存
   ```

2. c3p0-config.xml

   ```
   <c3p0-config>
       <!-- 默认配置 -->
       <default-config>
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

       </default-config>

       <!-- mysql的连接配置 -->
       <named-config name="mysql">
           <property name="jdbcUrl">jdbc:mysql://localhost:3306/scott</property>
           <property name="user">root</property>
           <property name="password">root</property>
           <property name="driverClass">com.mysql.jdbc.Driver</property>
       </named-config>
    </c3p0-config>
   ```

3. DbManager

   ```java
   public class DbManager {
       private static volatile DbManager instance = null;
       /**
        * 数据库连接池核心类
        */
       static ComboPooledDataSource ds = null;
       /**
        * 数据库线程池配置名称
        */
       private static final String C3P0_CONFIG_NAME = "mysql";

       static {
           ds = new ComboPooledDataSource(C3P0_CONFIG_NAME);
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

       public Connection getConn() {
           Connection connection = null;
           try {
               connection = ds.getConnection();
           } catch (SQLException e) {
               e.printStackTrace();
           }
           return connection;
       }

       public void close(ResultSet rs, Statement statement, Connection conn) {
           if (conn != null) {
               try {
                   conn.close();
               } catch (SQLException e) {
                   e.printStackTrace();
               } finally {
                   close(rs, statement);
               }
           }
       }

       public void close(Statement statement, Connection conn) {
           if (statement != null) {
               try {
                   statement.close();
               } catch (SQLException e) {
                   e.printStackTrace();
               } finally {
                   if (conn != null) {
                       try {
                           conn.close();
                       } catch (SQLException e) {
                           e.printStackTrace();
                       }
                   }
               }
           }

       }

       public void close(ResultSet rs, Statement statement) {
           if (rs != null) {
               try {
                   rs.close();
               } catch (SQLException e) {
                   e.printStackTrace();
               } finally {
                   close(statement);
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

       /**
        * 专门用于发送增删改语句的方法
        *
        * @param
        * @return true表示成功  false表示失败
        */
       public boolean executeUpdate(String sql, Object... params) throws SQLException {
           int count = 0;
           Connection conn = getConn();
           PreparedStatement ps = null;
           try {
               conn.setAutoCommit(false);
               ps = conn.prepareStatement(sql);
               setParams(ps, params);
               //使用Statement对象发送SQL语句
               count = ps.executeUpdate();
               getConn().commit();
           } catch (SQLException e) {
               conn.rollback();
           } finally {
               close(ps, conn);
           }
           return count > 0;
       }

       public boolean executeBatch(String sql, List<Object[]> data) {
           Connection conn = getInstance().getConn();
           int[] ints = new int[0];
           PreparedStatement ps = null;
           try {
               conn.setAutoCommit(false);
               ps = conn.prepareStatement(sql);
               if (data != null && data.size() > 0)
                   for (int i = 0; i < data.size(); i++) {
                       Object[] obj = data.get(i);
                       for (int j = 0; j < obj.length; j++) {
                           ps.setObject(j, obj[j]);
                           ps.addBatch();
                       }
                   }
               ints = ps.executeBatch();
               conn.commit();
           } catch (SQLException e) {
               try {
                   conn.rollback();
               } catch (SQLException e1) {
                   e1.printStackTrace();
               }
           } finally {
               close(ps, conn);
           }
           return ints[0] > 0;
       }

       /**
        * 执行查询
        *
        * @param conn   数据库连接
        * @param sql    sql语句
        * @param params 占位符数组
        * @return
        */
       public ResultSet execQuery(Connection conn, PreparedStatement ps, String sql, Object... params) {
           ResultSet rs = null;
           try {
               ps = conn.prepareStatement(sql);
               setParams(ps, params);
               //使用Statement对象发送SQL语句
               rs = ps.executeQuery();
           } catch (SQLException e) {
               e.printStackTrace();
           }
           return rs;
       }

       private void setParams(PreparedStatement ps, Object... params) throws SQLException {
           if (params != null && params.length > 0) {
               for (int i = 0; i < params.length; i++) {
                   ps.setObject(1, params[i]);
               }
           }
       }

   }
   ```

4. MobileDao

   ```
   1>批量插入5条测试数据
   2>查询所有的数据
   3>通过主键删除记录
   4>根据
   ```

### 6.2、Dao封装

\(未经过严格测试,仅供学习使用,请不要用于实战开发\(建议使用hibernate,mybatis框架\)\)

\(未经过严格测试,仅供学习使用,请不要用于实战开发\(建议使用hibernate,mybatis框架\)\)

\(未经过严格测试,仅供学习使用,请不要用于实战开发\(建议使用hibernate,mybatis框架\)\)

1. BaseDao

   ```java
   public interface IBaseDao {
       /**
        * 封装增删改的操作
        *
        * @return
        */
       public boolean executeUpdate(String sql, Object... params);

       /**
        * 查询单个对象
        *
        * @param sql
        * @param entity
        * @param params
        * @param <T>
        * @return
        */
       public <T> T query(String sql, Class<T> entity, Object... params);

       /**
        * 查询多个对象
        *
        * @param sql
        * @param entity
        * @param params
        * @param <T>
        * @return
        */
       public <T> List<T> queryList(String sql, Class<T> entity, Object... params);

       /**
        * 保存对象
        * @param entity
        * @param <T>
        */
       public <T> void save(T entity);

       /**
        * 通过主键更新对象
        * @param entity
        * @param <T>
        * @throws Exception
        */
       public <T> void update(T entity) throws Exception;

       /**
        * 通过主键删除对象
        * @param entity
        * @param <T>
        * @throws Exception
        */
       public <T> void delete(T entity) throws Exception;
   }
   ```

2. BaseDaoImpl

   ```java
   import com.werner.jdbc.dao.IBaseDao;
   import com.werner.jdbc.utils.DbManager;

   import java.lang.reflect.Field;
   import java.lang.reflect.Method;
   import java.sql.*;
   import java.util.ArrayList;
   import java.util.Iterator;
   import java.util.List;

   /**
    * 未经过严格测试,仅供学习使用,请不要用于实战开发(建议使用hibernate,mybatis框架)
    */
   public class BaseDaoImpl implements IBaseDao {
       @Override
       public boolean executeUpdate(String sql, Object... params) {
           boolean flag = false;
           try {
               flag = DbManager.getInstance().executeUpdate(sql, params);
           } catch (SQLException e) {
               e.printStackTrace();
           }
           return flag;
       }

       @Override
       public <T> T query(String sql, Class<T> entity, Object... params) {
           return queryList(sql, entity, params).get(0);
       }
          @Override
      public <T> List<T> queryList(String sql, Class<T> entity, Object... params) {
          Connection conn = DbManager.getInstance().getConn();
          PreparedStatement ps = null;

          ResultSet rs = DbManager.getInstance().execQuery(conn, ps, sql, params);
          List<T> list = new ArrayList<>();
          if (rs != null) {
              try {
                  Field[] fields = entity.getDeclaredFields();
                  while (rs.next()) {
                      T t = entity.newInstance();
                      list.add(t);
                      for (Field field : fields) {
                          field.setAccessible(true);
                          Object value = rs.getObject(field.getName());
                          setValue(t, field, value);
                      }
                  }
              } catch (SQLException | IllegalAccessException | InstantiationException e) {
                  e.printStackTrace();
              } finally {
                  DbManager.getInstance().close(rs, ps, conn);
              }
          }
          return list;
      }
         private  <T> void setValue(T t, Field f, Object value) throws IllegalAccessException {
          if (null == value)
              return;
          String valueStr = value.toString();
          String typeName = f.getType().getName();
          if ("java.lang.Byte".equals(typeName) || "byte".equals(typeName)) {
              f.set(t, Byte.parseByte(valueStr));
          } else if ("java.lang.Short".equals(typeName) || "short".equals(typeName)) {
              f.set(t, Short.parseShort(valueStr));
          } else if ("java.lang.Integer".equals(typeName) || "int".equals(typeName)) {
              f.set(t, Integer.parseInt(valueStr));
          } else if ("java.lang.Long".equals(typeName) || "long".equals(typeName)) {
              f.set(t, Long.parseLong(valueStr));
          } else if ("java.lang.Float".equals(typeName) || "float".equals(typeName)) {
              f.set(t, Float.parseFloat(valueStr));
          } else if ("java.lang.Double".equals(typeName) || "double".equals(typeName)) {
              f.set(t, Double.parseDouble(valueStr));
          } else if ("java.lang.String".equals(typeName)) {
              f.set(t, value.toString());
          } else if ("java.lang.Character".equals(typeName) || "char".equals(typeName)) {
              f.set(t, value);
          } else if ("java.util.Date".equals(typeName)) {
              f.set(t, new Date(((java.sql.Date) value).getTime()));
          } else if ("java.util.Timer".equals(typeName)) {
              f.set(t, new Time(((java.sql.Time) value).getTime()));
          } else if ("java.sql.Timestamp".equals(typeName)) {
              f.set(t, value);
          } else if ("java.math.BigDecimal".equals(typeName)) {
              f.set(t, value);
          } else {
              System.out.println("Sql：不支持的数据类型");
          }

      }
       /**
       * 保存
       */
      public <T> void save(T entity) {
          PreparedStatement ps = null;
          Connection conn = DbManager.getInstance().getConn();

          //SQL语句,insert into table name (
          String sql = "insert into " + entity.getClass().getSimpleName().toLowerCase() + "(";
          //获得带有字符串get的所有方法的对象
          List<Method> list = this.matchPojoMethods(entity, "get");
          Iterator<Method> iter = list.iterator();
          //拼接字段顺序 insert into table emp(empno,ename,job,
          while (iter.hasNext()) {
              Method method = iter.next();
              sql += method.getName().substring(3).toLowerCase() + ",";
          }
          //去掉最后一个,符号insert insert into table name(empno,ename,job) values(
          sql = sql.substring(0, sql.lastIndexOf(",")) + ") values(";
          //拼装预编译SQL语句insert insert into table name(empno,name,email) values(?,?,?,
          for (int j = 0; j < list.size(); j++) {
              sql += "?,";
          }
          //去掉SQL语句最后一个,符号insert insert into table name(empno,ename,job) values(?,?,?);
          sql = sql.substring(0, sql.lastIndexOf(",")) + ")";
          //到此SQL语句拼接完成,打印SQL语句
          System.out.println(sql);
          //获得预编译对象的引用
          try {
              ps = DbManager.getInstance().getConn().prepareStatement(sql);
              int i = 0;
              //把指向迭代器最后一行的指针移到第一行.
              iter = list.iterator();
              while (iter.hasNext()) {
                  Method method = iter.next();
                  //此初判断返回值的类型,因为存入数据库时有的字段值格式需要改变,比如String,SQL语句是'aaa'
                  if (method.getReturnType().getSimpleName().indexOf("String") != -1) {
                      ps.setString(++i, this.getString(method, entity));
                  } else if (method.getReturnType().getSimpleName().indexOf("Date") != -1) {
                      ps.setDate(++i, this.getDate(method, entity));
                  } else {
                      ps.setInt(++i, this.getInt(method, entity));
                  }
              }
              ps.executeUpdate();
          } catch (Exception e) {
              e.printStackTrace();
          } finally {
              DbManager.getInstance().close(ps, conn);
          }

      }

      /**
       * 修改
       */
      public <T> void update(T entity) throws Exception {
          String sql = "update " + entity.getClass().getSimpleName().toLowerCase() + " set ";

          //获得该类所有get方法对象集合
          List<Method> list = this.matchPojoMethods(entity, "get");
          //临时Method对象,负责迭代时装method对象.
          Method tempMethod = null;

          //由于修改时不需要修改ID,所以按顺序加参数则应该把Id移到最后.
          Method idMethod = null;
          Iterator<Method> iter = list.iterator();
          while (iter.hasNext()) {
              tempMethod = iter.next();
              //如果方法名中带有ID字符串并且长度为2,则视为ID.
              if (tempMethod.getName().lastIndexOf("Id") != -1 && tempMethod.getName().substring(3).length() == 2) {
                  //把ID字段的对象存放到一个变量中,然后在集合中删掉.
                  idMethod = tempMethod;
                  iter.remove();
                  //如果方法名去掉set/get字符串以后与pojo + "id"想符合(大小写不敏感),则视为ID
              } else if ((entity.getClass().getSimpleName() + "Id").equalsIgnoreCase(tempMethod.getName().substring(3))) {
                  idMethod = tempMethod;
                  iter.remove();
              }
          }

          //把迭代指针移到第一位
          iter = list.iterator();
          while (iter.hasNext()) {
              tempMethod = iter.next();
              sql += tempMethod.getName().substring(3).toLowerCase() + "= ?,";
          }

          //去掉最后一个,符号
          sql = sql.substring(0, sql.lastIndexOf(","));

          //添加条件
          sql += " where " + idMethod.getName().substring(3).toLowerCase() + " = ?";

          //SQL拼接完成,打印SQL语句
          System.out.println(sql);

          PreparedStatement statement = DbManager.getInstance().getConn().prepareStatement(sql);

          int i = 0;
          iter = list.iterator();
          while (iter.hasNext()) {
              Method method = iter.next();
              //此初判断返回值的类型,因为存入数据库时有的字段值格式需要改变,比如String,SQL语句是'aaa'
              if (method.getReturnType().getSimpleName().indexOf("String") != -1) {
                  statement.setString(++i, this.getString(method, entity));
              } else if (method.getReturnType().getSimpleName().indexOf("Date") != -1) {
                  statement.setDate(++i, this.getDate(method, entity));
              } else {
                  statement.setInt(++i, this.getInt(method, entity));
              }
          }
          //为Id字段添加值
          if (idMethod.getReturnType().getSimpleName().indexOf("String") != -1) {
              statement.setString(++i, this.getString(idMethod, entity));
          } else {
              statement.setInt(++i, this.getInt(idMethod, entity));
          }

          //执行SQL语句
          statement.executeUpdate();
          //关闭预编译对象
          statement.close();
             //关闭连接
          conn.close();

      }

       /**
       * 删除
       */
      public <T> void delete(T entity) throws Exception {
          String sql = "delete from " + entity.getClass().getSimpleName().toLowerCase() + " where ";
          //存放字符串为主键的字段对象
          Method idMethod = null;
          //取得字符串为"id"的字段对象
          List<Method> list = this.matchPojoMethods(entity, "get");
          Iterator<Method> iter = list.iterator();
          while (iter.hasNext()) {
              Method tempMethod = iter.next();
              //如果方法名中带有ID字符串并且长度为2,则视为ID.
              if (tempMethod.getName().lastIndexOf("Id") != -1 && tempMethod.getName().substring(3).length() == 2) {
                  //把ID字段的对象存放到一个变量中,然后在集合中删掉.
                  idMethod = tempMethod;
                  iter.remove();
                  //如果方法名去掉set/get字符串以后与pojo + "id"想符合(大小写不敏感),则视为ID
              } else if ((entity.getClass().getSimpleName() + "Id").equalsIgnoreCase(tempMethod.getName().substring(3))) {
                  idMethod = tempMethod;
                  iter.remove();
              }
          }
          sql += idMethod.getName().substring(3).toLowerCase() + " = ?";

          PreparedStatement statement = DbManager.getInstance().getConn().prepareStatement(sql);

          //为Id字段添加值
          int i = 0;
          if (idMethod.getReturnType().getSimpleName().indexOf("String") != -1) {
              statement.setString(++i, this.getString(idMethod, entity));
          } else {
              statement.setInt(++i, this.getInt(idMethod, entity));
          }

      }

      /**
       * 过滤当前Pojo类所有带传入字符串的Method对象,返回List集合.
       */
      private <T> List<Method> matchPojoMethods(T entity, String methodName) {
          //获得当前Pojo所有方法对象
          Method[] methods = entity.getClass().getDeclaredMethods();
          //List容器存放所有带get字符串的Method对象
          List<Method> list = new ArrayList<Method>();
          //过滤当前Pojo类所有带get字符串的Method对象,存入List容器
          for (int index = 0; index < methods.length; index++) {
              if (methods[index].getName().indexOf(methodName) != -1) {
                  list.add(methods[index]);
              }
          }
          return list;
      }
      /**
       * 方法返回类型为int或Integer类型时,返回的SQL语句值.对应get
       */
      private <T> Integer getInt(Method method, T entity) throws Exception {
          return (Integer) method.invoke(entity, new Object[]{});
      }

      /**
       * 方法返回类型为String时,返回的SQL语句拼装值.比如'abc',对应get
       */
      private <T> String getString(Method method, T entity) throws Exception {
          return (String) method.invoke(entity, new Object[]{});
      }

      /**
       * 方法返回类型为Date时,返回的SQL语句拼装值,对应get
       */
      private <T> Date getDate(Method method, T entity) throws Exception {
          return (Date) method.invoke(entity, new Object[]{});
      }

       /**
       * 参数类型为Integer或int时,为entity字段设置参数,对应set
       */
      private <T> Integer setInt(Method method, T entity, Integer arg) throws Exception {
          return (Integer) method.invoke(entity, new Object[]{arg});
      }

      /**
       * 参数类型为String时,为entity字段设置参数,对应set
       */
      private <T> String setString(Method method, T entity, String arg) throws Exception {
          return (String) method.invoke(entity, new Object[]{arg});
      }

      /**
       * 参数类型为Date时,为entity字段设置参数,对应set
       */
      private  <T> Date setDate(Method method, T entity, Date arg) throws Exception {
          return (Date) method.invoke(entity, new Object[]{arg});
      }

   }
   ```

## 七、其他

### 7.1、po与vo的区别

1. PO \(persistant object\)持久对象

   ```
   持久对象,可以看成是与数据库中的表相映射的java对象。
   最简单的PO就是对应数据库中某个表中的一条记录，多个记录可以用PO的集合。PO中应该不包含任何对数据库的操作
   ```

2. VO:\(value object\)值对象,\(view object\)视图对象




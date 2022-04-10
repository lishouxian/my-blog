---
title: 'JDBC(二)'
date: 2020-08-29 15:14:00
tags: [数据库,javaweb]
categories: javaweb
published: true
hideInList: true
feature: 
isTop: false
---

## 操作Blob类型数据

#### mysql中Blob类型

- mysql中Blob是一个二进制的大型对象，是一个可以存储大量数据的容器，他能容纳不同大小的数据
- 插入blob类型的数据必须使用Preparedstatement，因为BLOB类型的数据无法使用字符串拼接
- Mysql中有四种类型的Blob（除了在储存的最大容量上的差别外，他们是等同的）

| 类型       | 大小（单位字节） |
| ---------- | ---------------- |
| TinyBlob   | 最大255字节      |
| Blob       | 最大65K          |
| MediumBlob | 最大16M          |
| LongBlob   | 最大4G           |

- 在实际开发过程中根据存入护具大小定义不同类型的BLob
- 需要注意的是假如储存的数据的容量过大，数据库的性能会下降

<!-- more -->

```java 
FileInputStream is = new FileInputStream(new File("1.jpg"))
//以流的方式来传输
ps.setObject(3,is);
```

```java
Blob photo = re.getBlob("photo")
InputStream is = photo.getBinaryStream();
FileOutputStream fos = new FileOutputStream("li.jpg")
byte[] buffer = new byte[1024];
int len;
while((len = is.read(buffer))!= 1){
    fos.write(buffer,0,len);
}
is.close();
fos.close();
```

如果在指定了相关的Blob类型以后，还报错：xxx too large，那么在mysql的安装目录下，找my.ini文件加上如下的配置参数： **max_allowed_packet=16M**。同时注意：修改了my.ini文件之后，需要重新启动mysql服务。

## 批量操作数据

当需要批量插入活着更新记录时可以使用Java的批量更新机制，这一机制允许多条语句一次性提交给数据库批量处理。通常情况下会比单独提交更有效率。

使用prepardStatement如何实现更加高效的批量操作

- addBathch(String) 添加需要批量处理的SQL语句或者参数
- excuteBatch() 执行批量处理语句
- clearBatch() 清空缓存的数据

通常情况下我们会遇到两种批量执行SQL语句的情况：

- 多条SQL语句的批量处理
- 一个SQL语句的批量传参

如何使用preparedStatement批量传参呢


```java
/*
* 使用Connection 的 setAutoCommit(false)  /  commit()
*/
@Test
public void testInsert2() throws Exception{
	long start = System.currentTimeMillis();
		
	Connection conn = JDBCUtils.getConnection();
		
	//1.设置为不自动提交数据
	conn.setAutoCommit(false);
		
	String sql = "insert into goods(name)values(?)";
	PreparedStatement ps = conn.prepareStatement(sql);
		
	for(int i = 1;i <= 1000000;i++){

    public void testJDBCTransaction() {
    	Connection conn = null;
    	try {
    		// 1.获取数据库连接
    		conn = JDBCUtils.getConnection();
    		// 2.开启事务
    		conn.setAutoCommit(false);
    		// 3.进行数据库操作
    		String sql1 = "update user_table set balance = balance - 100 where user = ?";
    		update(conn, sql1, "AA");
    
    		// 模拟网络异常
    		//System.out.println(10 / 0);
    
    		String sql2 = "update user_table set balance = balance + 100 where user = ?";
    		update(conn, sql2, "BB");
    		// 4.若没有异常，则提交事务
    		conn.commit();
    	} catch (Exception e) {
    		e.printStackTrace();
    		// 5.若有异常，则回滚事务
    		try {
    			conn.rollback();
    		} catch (SQLException e1) {
    			e1.printStackTrace();
    		}
        } finally {
            try {
    			//6.恢复每次DML操作的自动提交功能
    			conn.setAutoCommit(true);
    		} catch (SQLException e) {
    			e.printStackTrace();
    		}
            //7.关闭连接
    		JDBCUtils.closeResource(conn, null, null); 
        }  
    }
    
```


其中，对数据库操作的方法为：
```java

//使用事务以后的通用的增删改操作（version 2.0）
public void update(Connection conn ,String sql, Object... args) {
	PreparedStatement ps = null;
	try {
		// 1.获取PreparedStatement的实例 (或：预编译sql语句)
		ps = conn.prepareStatement(sql);
		// 2.填充占位符
		for (int i = 0; i < args.length; i++) {
			ps.setObject(i + 1, args[i]);
		}
		// 3.执行sql语句
		ps.execute();
	} catch (Exception e) {
		e.printStackTrace();
	} finally {
		// 4.关闭资源
		JDBCUtils.closeResource(null, ps);

	}
}
```

事务的ACID属性
      
1. **原子性（Atomicity）**
原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。 
2. **一致性（Consistency）**
事务必须使数据库从一个一致性状态变换到另外一个一致性状态。

3. **隔离性（Isolation）**
事务的隔离性是指一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。

4. **持久性（Durability）**
持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来的其他操作和数据库故障不应该对其有任何影响。

#### 数据库的并发问题

- 对于同时运行的多个事务, 当这些事务访问数据库中相同的数据时, 如果没有采取必要的隔离机制, 就会导致各种并发问题:
- **脏读**: 对于两个事务 T1, T2, T1 读取了已经被 T2 更新但还**没有被提交**的字段。之后, 若 T2 回滚, T1读取的内容就是临时且无效的。
- **不可重复读**: 对于两个事务T1, T2, T1 读取了一个字段, 然后 T2 **更新**了该字段。之后, T1再次读取同一个字段, 值就不同了。
- **幻读**: 对于两个事务T1, T2, T1 从一个表中读取了一个字段, 然后 T2 在该表中**插入**了一些新的行。之后, 如果 T1 再次读取同一个表, 就会多出几行。

- **数据库事务的隔离性**: 数据库系统必须具有隔离并发运行各个事务的能力, 使它们不会相互影响, 避免各种并发问题。

- 一个事务与其他事务隔离的程度称为隔离级别。数据库规定了多种事务隔离级别, 不同隔离级别对应不同的干扰程度, **隔离级别越高, 数据一致性就越好, 但并发性越弱。**

#### 四种隔离级别

- 数据库提供的4种事务隔离级别：

![1555586275271](https://blog-1251782526.cos.ap-shanghai.myqcloud.com/uPic/007S8ZIlly1gi9rdmk2caj30lu0680ur.jpg)

- Oracle 支持的 2 种事务隔离级别：**READ COMMITED**, SERIALIZABLE。 Oracle 默认的事务隔离级别为: **READ COMMITED** 


- Mysql 支持 4 种事务隔离级别。Mysql 默认的事务隔离级别为: **REPEATABLE READ。**

#### 在MySql中设置隔离级别

- 每启动一个 mysql 程序, 就会获得一个单独的数据库连接. 每个数据库连接都有一个全局变量 @@tx_isolation, 表示当前的事务隔离级别。

- 查看当前的隔离级别: 

```mysql
SELECT @@tx_isolation;
```

- 设置当前 mySQL 连接的隔离级别:  

```mysql
set  transaction isolation level read committed;
```

- 设置数据库系统的全局的隔离级别:

```mysql
set global transaction isolation level read committed;
```

- 补充操作：

- 创建mysql数据库用户：

```mysql
create user tom identified by 'abc123';
```

- 授予权限

```mysql
#授予通过网络方式登录的tom用户，对所有库所有表的全部权限，密码设为abc123.
grant all privileges on *.* to tom@'%'  identified by 'abc123'; 

#给tom用户使用本地命令行方式，授予atguigudb这个库下的所有表的插删改查的权限。
grant select,insert,delete,update on atguigudb.* to tom@localhost identified by 'abc123'; 

```


## 数据库连接池

### 数据库连接池的必要性

- 在开发基于数据库的web项目的时候，传统的模式基本是按照下面的步骤
- 在主程序中（如servlet，beans）中创建数据库连接
- 进行sql操作
- 断开数据库连接
- 这种模式开发，存在的问题有
- 普通的数据库连接使用的是DriverManager来获取，每次向数据库建立连接的时候都要将Connection对象加载到内存中，在验证用户名和密码(得花费0.05秒或0.1秒)。需要数据库连接时，就像数据库要求一个，执行完成之后就断开连接，这种方式会大量损耗资源和时间。数据库的连接资源没有得到很好的重复利用吗，如果同时有几百人甚至几千人在线，频繁的进行数据库连接操作会占用很多的系统资源，严重的会造成服务器的崩溃。
- 对于每一次的连接，用完之后都要断开数据库的连接，假设程序出现异常没有断开数据连接，将会造成系统数据库系统中的内存泄漏，最终将导致重启数据库。
- 这种开发不能控制被创建的连接对象数目，系统资源会毫无顾忌的被分配出去，假设连接过多，也会造成系统内存泄漏，最终将导致重启数据库。

### 数据库连接池技术是什么

- 为解决传统开发过程中数据库连接问题，可以使用数据库连接池技术。
- 数据库连接池的基本思想是：为数据库创建一个“缓冲池”。预先在缓冲池中放入一定数目的连接，当需要建立数据库连接时，只需要从》缓冲池中取出一个，使用完毕之后再放回去就好
- 数据库连接池负责分配、管理和释放数据库连接，它允许应用程序重复使用一个现有的数据库连接而不是重新创建一个。
- 数据库连接池在初始化时将创建一定数目的数据连接放置到数据库连接池中，这些数据库连接的数量是有最小的数据库连接数来设定的。无论这些数据库连接是否被使用，连接池都将一直保证至少有这么多的连接数目。连接池的最大数据库连接数目限定了这个连接能占有的最大连接数目，当应用程序向数据库连接池请求的连接大于最大连接数目时，这些请求将被加入到等待队列中.

- **工作原理：**

![1555593598606](https://blog-1251782526.cos.ap-shanghai.myqcloud.com/uPic/007S8ZIlly1gi6qwygh07j30fj067wf3.jpg)

- **数据库连接池技术的优点**

**1. 资源重用**

由于数据库连接得以重用，避免了频繁创建，释放连接引起的大量性能开销。在减少系统消耗的基础上，另一方面也增加了系统运行环境的平稳性。

**2. 更快的系统反应速度**

数据库连接池在初始化过程中，往往已经创建了若干数据库连接置于连接池中备用。此时连接的初始化工作均已完成。对于业务请求处理而言，直接利用现有可用连接，避免了数据库连接初始化和释放过程的时间开销，从而减少了系统的响应时间

**3. 新的资源分配手段**

对于多应用共享同一数据库的系统而言，可在应用层通过数据库连接池的配置，实现某一应用最大可用数据库连接数的限制，避免某一应用独占所有的数据库资源

**4. 统一的连接管理，避免数据库连接泄漏**

在较为完善的数据库连接池实现中，可根据预先的占用超时设定，强制回收被占用连接，从而避免了常规数据库连接操作中可能出现的资源泄露

### 多种开源的数据哭连接池

- JDBC 的数据库连接池使用 javax.sql.DataSource 来表示，DataSource 只是一个接口，该接口通常由服务器(Weblogic, WebSphere, Tomcat)提供实现，也有一些开源组织提供实现：
- **DBCP** 是Apache提供的数据库连接池。tomcat 服务器自带dbcp数据库连接池。**速度相对c3p0较快**，但因自身存在BUG，Hibernate3已不再提供支持。
- **C3P0** 是一个开源组织提供的一个数据库连接池，**速度相对较慢，稳定性还可以。**hibernate官方推荐使用
- **Proxool** 是sourceforge下的一个开源项目数据库连接池，有监控连接池状态的功能，**稳定性较c3p0差一点**
- **BoneCP** 是一个开源组织提供的数据库连接池，速度快
- **Druid** 是阿里提供的数据库连接池，据说是集DBCP 、C3P0 、Proxool 优点于一身的数据库连接池，但是速度不确定是否有BoneCP快
- DataSource 通常被称为数据源，它包含连接池和连接池管理两个部分，习惯上也经常把 DataSource 称为连接池
- **DataSource用来取代DriverManager来获取Connection，获取速度快，同时可以大幅度提高数据库访问速度。**
- 特别注意：
- 数据源和数据库连接不同，数据源无需创建多个，它是产生数据库连接的工厂，因此**整个应用只需要一个数据源即可。**
- 当数据库访问结束后，程序还是像以前一样关闭数据库连接：conn.close(); 但conn.close()并没有关闭数据库的物理连接，它仅仅把数据库连接释放，归还给了数据库连接池。

#### Druid（德鲁伊）数据库连接池

Druid是阿里巴巴开源平台上一个数据库连接池实现，它结合了C3P0、DBCP、Proxool等DB池的优点，同时加入了日志监控，可以很好的监控DB池连接和SQL的执行情况，可以说是针对监控而生的DB连接池，**可以说是目前最好的连接池之一。**

```java
package com.atguigu.druid;

import java.sql.Connection;
import java.util.Properties;

import javax.sql.DataSource;

import com.alibaba.druid.pool.DruidDataSourceFactory;

public class TestDruid {
public static void main(String[] args) throws Exception {
Properties pro = new Properties();		 pro.load(TestDruid.class.getClassLoader().getResourceAsStream("druid.properties"));
DataSource ds = DruidDataSourceFactory.createDataSource(pro);
Connection conn = ds.getConnection();
System.out.println(conn);
}
}

```

其中，resrouse下的配置文件为：【druid.properties】

```java
url=jdbc:mysql://localhost:3306/mybatis?rewriteBatchedStatements=true
username=root
password=123456
driverClassName=com.mysql,cj.jdbc.Driver

initialSize=10
maxActive=20
maxWait=1000
filters=wall
```

- 详细配置参数：

| **配置**                      | **缺省** | **说明**                                                     |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| name                          |          | 配置这个属性的意义在于，如果存在多个数据源，监控的时候可以通过名字来区分开来。   如果没有配置，将会生成一个名字，格式是：”DataSource-” +   System.identityHashCode(this) |
| url                           |          | 连接数据库的url，不同数据库不一样。例如：mysql :   jdbc:mysql://10.20.153.104:3306/druid2      oracle :   jdbc:oracle:thin:@10.20.149.85:1521:ocnauto |
| username                      |          | 连接数据库的用户名                                           |
| password                      |          | 连接数据库的密码。如果你不希望密码直接写在配置文件中，可以使用ConfigFilter。详细看这里：<https://github.com/alibaba/druid/wiki/%E4%BD%BF%E7%94%A8ConfigFilter> |
| driverClassName               |          | 根据url自动识别   这一项可配可不配，如果不配置druid会根据url自动识别dbType，然后选择相应的driverClassName(建议配置下) |
| initialSize                   | 0        | 初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时 |
| maxActive                     | 8        | 最大连接池数量                                               |
| maxIdle                       | 8        | 已经不再使用，配置了也没效果                                 |
| minIdle                       |          | 最小连接池数量                                               |
| maxWait                       |          | 获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。 |
| poolPreparedStatements        | false    | 是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭。 |
| maxOpenPreparedStatements     | -1       | 要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100 |
| validationQuery               |          | 用来检测连接是否有效的sql，要求是一个查询语句。如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会其作用。 |
| testOnBorrow                  | true     | 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。 |
| testOnReturn                  | false    | 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能 |
| testWhileIdle                 | false    | 建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。 |
| timeBetweenEvictionRunsMillis |          | 有两个含义： 1)Destroy线程会检测连接的间隔时间2)testWhileIdle的判断依据，详细看testWhileIdle属性的说明 |
| numTestsPerEvictionRun        |          | 不再使用，一个DruidDataSource只支持一个EvictionRun           |
| minEvictableIdleTimeMillis    |          |                                                              |
| connectionInitSqls            |          | 物理连接初始化的时候执行的sql                                |
| exceptionSorter               |          | 根据dbType自动识别   当数据库抛出一些不可恢复的异常时，抛弃连接 |
| filters                       |          | 属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有：   监控统计用的filter:stat日志用的filter:log4j防御sql注入的filter:wall |
| proxyFilters                  |          | 类型是List，如果同时配置了filters和proxyFilters，是组合关系，并非替换关系 |

## Apache-DBUtils实现CRUD操作

### 简介

commons-dbutils 是 Apache 组织提供的一个开源 JDBC工具类库，它是对JDBC的简单封装，学习成本极低，并且使用dbutils能极大简化jdbc编码的工作量，同时也不会影响程序的性能。

### 主要API的使用

#### DbUtils

- DbUtils ：提供如关闭连接、装载JDBC驱动程序等常规工作的工具类，里面的所有方法都是静态的。主要方法如下：
- **public static void close(…) throws java.sql.SQLException**：　DbUtils类提供了三个重载的关闭方法。这些方法检查所提供的参数是不是NULL，如果不是的话，它们就关闭Connection、Statement和ResultSet。
- public static void closeQuietly(…): 这一类方法不仅能在Connection、Statement和ResultSet为NULL情况下避免关闭，还能隐藏一些在程序中抛出的SQLEeception。
- public static void commitAndClose(Connection conn)throws SQLException： 用来提交连接的事务，然后关闭连接
- public static void commitAndCloseQuietly(Connection conn)： 用来提交连接，然后关闭连接，并且在关闭连接时不抛出SQL异常。 
- public static void rollback(Connection conn)throws SQLException：允许conn为null，因为方法内部做了判断
- public static void rollbackAndClose(Connection conn)throws SQLException
- rollbackAndCloseQuietly(Connection)
- public static boolean loadDriver(java.lang.String driverClassName)：这一方装载并注册JDBC驱动程序，如果成功就返回true。使用该方法，你不需要捕捉这个异常ClassNotFoundException。


#### QueryRunner类

- **该类简单化了SQL查询，它与ResultSetHandler组合在一起使用可以完成大部分的数据库操作，能够大大减少编码量。**

  - QueryRunner类提供了两个构造器：
  - 默认的构造器
  - 需要一个 javax.sql.DataSource 来作参数的构造器

  - QueryRunner类的主要方法：
- **更新**
  - public int update(Connection conn, String sql, Object... params) throws SQLException:用来执行一个更新（插入、更新或删除）操作。
  - ......
- **插入**
  - public <T> T insert(Connection conn,String sql,ResultSetHandler<T> rsh, Object... params) throws SQLException：只支持INSERT语句，其中 rsh - The handler used to create the result object from the ResultSet of auto-generated keys.  返回值: An object generated by the handler.即自动生成的键值
  - ....
- **批处理**
  - public int[] batch(Connection conn,String sql,Object[][] params)throws SQLException： INSERT, UPDATE, or DELETE语句
  - public <T> T insertBatch(Connection conn,String sql,ResultSetHandler<T> rsh,Object[][] params)throws SQLException：只支持INSERT语句
  - .....
- **查询**
  - public Object query(Connection conn, String sql, ResultSetHandler rsh,Object... params) throws SQLException：执行一个查询操作，在这个查询中，对象数组中的每个元素值被用来作为查询语句的置换参数。该方法会自行处理 PreparedStatement 和 ResultSet 的创建和关闭。
  - ...... 


#### ResultSetHandler接口及实现类

- 该接口用于处理 java.sql.ResultSet，将数据按要求转换为另一种形式。

- ResultSetHandler 接口提供了一个单独的方法：Object handle (java.sql.ResultSet .rs)。

- 接口的主要实现类：

- ArrayHandler：把结果集中的第一行数据转成对象数组。
- ArrayListHandler：把结果集中的每一行数据都转成一个数组，再存放到List中。
- **BeanHandler：**将结果集中的第一行数据封装到一个对应的JavaBean实例中。
- **BeanListHandler：**将结果集中的每一行数据都封装到一个对应的JavaBean实例中，存放到List里。
- ColumnListHandler：将结果集中某一列的数据存放到List中。
- KeyedHandler(name)：将结果集中的每一行数据都封装到一个Map里，再把这些map再存到一个map里，其key为指定的key。
- **MapHandler：**将结果集中的第一行数据封装到一个Map里，key是列名，value就是对应的值。
- **MapListHandler：**将结果集中的每一行数据都封装到一个Map里，然后再存放到List
- **ScalarHandler：**查询单个值对象
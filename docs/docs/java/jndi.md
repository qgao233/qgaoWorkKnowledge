# JNDI粗略了解

参考：[JNDI学习总结（一）：JNDI到底是什么？](https://blog.csdn.net/wn084/article/details/80729230)

---

程序员开发时，要访问数据库，如果不用JNDI我们怎样做？用了JNDI后我们又将怎样做？

## 1 不用JNDI

将一个对 MySQL JDBC 驱动程序类的引用进行了编码，并通过使用适当的 JDBC URL 连接到数据库。

>题外话：何为驱动程序，个人理解，对于第三方的程序（mysql）或者硬件（显卡），最开始开发时，计算机没有和它们商量好对应的接口，于是出现了驱动程序这个中间件，它对上层（计算机）提供接口调用，内部向下调用第三方程序的接口，可以参考“适配器模式”。

就像以下代码这样：

```
Connection conn=null; 
try { 
	Class.forName("com.mysql.jdbc.Driver", true, Thread.currentThread().getContextClassLoader()); 
	conn=DriverManager.getConnection("jdbc:mysql://MyDBServer?user=xxx&password=xxx"); 
	...... 
	conn.close(); 
} catch(Exception e) { 
	e.printStackTrace(); 
} finally { 
	if(conn!=null) { 
		try { 
			conn.close(); 
		} catch(SQLException e) {} 
	}
}
```

这是传统的做法，但**存在的问题**：

1. 数据库服务器名称MyDBServer 、用户名和口令都可能需要改变，由此引发JDBC URL需要修改；
2. 数据库可能改用别的产品，如改用DB2或者Oracle，引发JDBC驱动程序包和类名需要修改；
3. 随着实际使用终端的增加，原配置的连接池参数可能需要调整；
4. ......

**解决办法**：

程序员应该不需要关心：

* 具体的数据库后台是什么？
* JDBC驱动程序是什么？
* JDBC URL格式是什么？
* 访问数据库的用户名和口令是什么？

等等这些问题，程序员编写的程序应该：

* 没有对 JDBC 驱动程序的引用，
* 没有服务器名称，
* 没有用户名称或口令 
* 甚至没有数据库池或连接管理。

而是把这些问题交给J2EE容器(tomcat)来配置和管理，程序员只需要对这些配置和管理进行引用即可。

## 2 用JNDI

以tomcat为例：

### 2.1 配置数据源

tomcat/conf/context.xml

```
<Resource name="jdbc/jd_mainframe" 
        auth="Container"
        validationQuery="select 1"
        type="javax.sql.DataSource" 
        driverClassName="com.microsoft.sqlserver.jdbc.SQLServerDriver"
        url="jdbc:sqlserver://127.0.0.1;DatabaseName=qgao"
        username="admin" password="admin" 
        maxActive="100" 
        maxIdle="50"
        maxWait="-1"
/>
```

这里，定义了一个名为`jdbc/jd_mainframe`的数据源，其参数包括JDBC的URL，驱动类名，用户名及密码等。

### 2.2 在程序中引用数据源

在spring中使用。

* application.properties

```
jdbc.jndi=jdbc/jd_mainframe
```

* applicationContext-resources-jndi.xml

```
<!-- JNDI DataSource for J2EE environments -->
<jee:jndi-lookup id="dataSource" jndi-name="${jdbc.jndi}" />
```

>这里定义完dataSource后，可以在其他地方使用第三方程序（如mybatis或hibernate）使用。

## 3 总结

J2EE 规范要求所有 J2EE 容器（tomcat或jboss）都要提供 JNDI 规范的实现。

可见：

* 为了应对主程序和第三方程序的联动，出现了jdbc驱动程序。
* 为了应对程序员开发时更好的业务分层，又在主程序和jdbc驱动之间插入了jndi。

>现在常用mybatis代替jndi的使用。
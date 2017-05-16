# 使用RowLoader加载现有数据 #
当您使用RowStore作为缓存时，您可以配置一个SQL数据加载器，该数据加载器被触发以从RowStore中的一个未完成的后端存储库加载数据。<br/>

当分布式缓存不能满足唯一标识行的传入查询请求时，将调用加载程序以从外部源检索数据。RowStore锁定关联的行，并阻止尝试从同一行获取相同行的并发读取器来覆盖后端数据库。<br/>

> 注意：当为表配置加载程序时，对表执行select语句将内部获取并将数据插入到RowStore表中。由于此行为，当您查询使用加载程序配置的表（例如，如果加载程序尝试从后端数据库中插入违反RowStore中的表约束的数据）时，可能会收到约束违规）。

- **RowStore如何调用** RowLoader只有满足某些条件时，RowStore才会调用已配置的RowLoader。

- **实现RowLoader接口** 要创建RowLoader，您可以定义一个实现com.pivotal.snappydata.callbacks.RowLoader接口的类，然后使用系统过程将您的实现附加到表中。该实现然后充当SQL数据加载器。

- **使用JDBCRowLoader示例** JDBCRowLoader类提供了一个从归档关系数据存储中加载数据的示例RowLoader实现。

## RowStore如何调用RowLoader ##
当满足某些条件时，RowStore仅调用已配置的RowLoader。<br/>

RowStore在以下情况下调用已配置的RowLoader：<br/>

- 对具有WHERE子句的表执行SELECT查询，该WHERE子句包括作为表主键的一部分的每个组件的等于（=）测试，以及
- 内存中不存在指定主键的行。

请注意，从结果行请求或使用哪些列并不重要。

例如，考虑以下示例表定义：

	CREATE TABLE LoaderTest(id INTEGER, name VARCHAR(10), result 
	VARCHAR(10), primary key (id, name));
在“LoaderTest”表中，列“id”和“name”形成主键。如果RowStore中的数据尚未提供，那么在主键上使用等式测试的查询将调用表的附加加载器实现。例如，这些查询将调用一个加载器：

	SELECT * FROM LoaderTest WHERE id = 5 AND name='Joe';
	SELECT result FROM LoaderTest WHERE id = 6 AND name='Sam'; 
但是，这些查询将不会调用附加的加载程序：

	SELECT result FROM LoaderTest WHERE id < 6 AND name='Sam'; 
	SELECT * FROM LoaderTest; 
	SELECT * FROM LoaderTest WHERE id = 5;
在上述示例中，没有一个SELECT语句对主键的所有组件（“id”和“name”）都使用相等性测试。

## 实现RowLoader接口 ##
要创建RowLoader，您可以定义一个实现com.pivotal.snappydata.callbacks.RowLoader接口的类，然后使用系统过程将您的实现附加到表中。该实现然后充当SQL数据加载器。

### 程序 ###



1. 包括一个不需要参数的public静态方法，并返回加载器的一个实例。例如：

		ExampleRowLoader.java 
		
		package example.callbacks; 
		import java.sql.*; 
		import javax.sql.*; 
		import javax.sql.rowset.*; 
		import com.pivotal.snappydata.callbacks.RowLoader; 
		
		public class MyRowLoader implements RowLoader{ 
		
		  public static MyRowLoader create() { 
		         return new ExampleRowLoader(); 
		  } 



1. 实现一个init（String initStr）方法来获取在使用表注册实现时定义的参数。当RowStore调用您实现的init（）方法SYS.ATTACH_LOADER过程用于注册的表RowLoader。所有参数都在单个String对象中传递。


1. RowStore调用getRow（String schemaName，String tableName，Object [] primarykey）方法实现，以提供模式名称，表名称和作为主键值的对象数组（按表定义显示的顺序），每次调用加载器以从外部源获取数据。您的这种方法的实现必须返回以下之一：

		-   An object array with one element with the value for each column, including the primary key, in the order defined by the RowStore table definition.
		-   null, if there is no element found.
		-   An instance of java.sql.ResultSet, possibly the result of a query against another database. Only the first row will be used. The result columns should match the RowStore table.
		-   An empty java.sql.ResultSet if no element is found.

1. 编译你RowLoader实现并将其添加到类路径之后，通过执行内置的注册表格中RowLoader SYS.ATTACH_LOADER程序。

## 使用JDBCRowLoader示例 ##
JDBCRowLoader类提供了一个从归档关系数据存储中加载数据的示例RowLoader实现。<br/>

- **特点和要求**

- **JDBCRowLoader示例实现**

### 特点和要求 ###
JDBCRowLoader示例具有以下功能：

- 可以为任何JDBC数据源实现（只要驱动程序在服务器的类路径中可用）

- 它可以为任何表实现，虽然为每个表创建了RowLoader的单独实例。

- 它将JDBC Connections和PreparedStatements存储在一起，并具有可配置的最小和最大连接数。

- 它使用Connection.isReadOnly（true）设置来请求驱动程序优化数据库读取的事务设置。


一个`query-string`参数传递给JDBCRowLoader来告诉它要对归档数据库使用什么SQL语句。这是必需的参数。<br/>

如果存档表的列布局与RowStore表的列布局匹配，则可以在查询字符串中使用SELECT *。<br/>

如果归档表的列布局与RowStore表的列布局不匹配，则必须在SELECT语句中显式提供和排序列名，以便结果集与RowStore表的布局相匹配。<br/>

没有要求RowStore中的模式或表名称与归档数据库中的模式和/或表名称相匹配。<br/>

当主键被调用时，主键的元素将按照RowStore表中的列的顺序被传递到JDBCRowLoader中。它们将按照该顺序作为参数传递到PreparedStatement中，因此您必须构造查询字符串的WHERE子句，以便以正确的顺序传递元素。<br/>

接受参数与示例：<br/>

- url = jdbc：oracle：thin：@localhost：1521：XE（必需）
- query-string = SELECT * FROM LoaderArchive WHERE id =？AND name =？（需要）
- 用户应用程式=
- 密码=应用
- 分的连接= 1
- 最大的连接= 5
- connection-timeout = 3000（毫秒）
所有属性在创建时也传递给JDBC连接。


## JDBCRowLoader示例实现 ##
RowStore在SnappyData_RowStore_13_b NNNNN _ platform /examples/JDBCRowLoader.java中安装一个示例JDBCRowLoader 实现。请注意，此示例需要gemfire.jar（从Apache Geode产品）才能运行。<br/>






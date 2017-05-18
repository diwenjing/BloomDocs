#核心JDBC类，接口和方法
以下部分描述了选定的JDBC类，接口和方法的RowStore实现。

##java.sql.BatchUpdateException类
此类是在执行批处理语句期间发生故障时抛出的SQLException的扩展。JDBC Statement，PreparedStatement或CallableStatement类的批处理操作可能会发生这种故障。其他批次元素导致的异常在getNextException（）链中维护，因为它们用于SQLExceptions。
<br/>

此类为名为getUpdateCounts（）的SQLException类添加一个新方法。此方法返回在发生异常之前成功执行的语句的更新计数数组。<br/>
##java.sql.Connection接口

直到从该连接创建的所有其他JDBC对象显式关闭或本身都是垃圾回收之前，RowStore Connection对象不会被垃圾回收。一旦连接关闭，就不能再对连接中创建的对象进行JDBC请求。在您不再需要执行语句之前，不要显式关闭Connection对象。<br/>

会话严重性或更高的异常导致连接关闭，并且所有其他JDBC对象都被关闭。<br/>


###java.sql.Connection.setTransactionIsolation方法
只有java.sql.Connection.TRANSACTION_NONE和java.sql.Connection.TRANSACTION_READ_COMMITTED事务隔离级别在RowStore中可用。
<br/>
TRANSACTION_NONE是默认的隔离级别。<br/>

改变当前隔离与连接setTransactionIsolation提交当前事务并开始新的事务。<br/>

注意： TRANSACTION \ _NONE RowStore的特殊注意事项即使在TRANSACTION \ _NONE隔离级别也为行级操作提供了原子性和线程安全性。<br/>

有关事务隔离的更多详细信息，请参阅事务和并发。<br/>


###不支持连接功能
java.sql.Connection.setReadOnly和isReadOnly methods RowStore不支持只读连接。调用setReadOnly值为true将抛出“不支持”功能异常，isReadOnly始终返回false。<br/>

此版本的RowStore不支持与XA DataSources（javax.sql.XADataSource）的连接。<br/>

RowStore不使用目录名称。此外，以下可选方法会引发“不支持功能”异常：<br/>
	
	createArrayOf（java.lang.String，java.lang.Object []）
	createNClob（）
	createSQLXML（）
	CreateStruct（java.lang.String，java.lang.Object []）
	getTypeMap（）
	setTypeMap（java.util.Map）
	




##java.sql.DatabaseMetaData接口

	以下部分描述了java.sql.DatabaseMetaDataRowStore中的功能。 - DatabaseMetaData结果集 - java.sql.DatabaseMetaData.getProcedureColumns方法 - getProcedureColumns的参数 - java.sql.DatabaseMetaData.getBestRowIdentifier方法


DatabaseMetaData结果集<br/>
即使将auto-commit设置为true，DatabaseMetaData结果集也不会关闭其他语句的结果集。<br/>

如果用户对导致自动提交发生的JDBC对象执行任何其他操作，则DatabaseMetaData结果集将关闭。如果需要DatabaseMetaData结果集在执行可能导致自动提交的其他操作时可访问，请使用setAutoCommit（false）关闭自动提交。<br/>


java.sql.DatabaseMetaData.getProcedureColumns方法<br/>
RowStore支持Java过程，并允许您在SQL语句中调用Java过程。RowStore返回有关getProcedureColumns调用中参数的信息，并返回由CREATE PROCEDURE定义的所有Java过程的信息。<br/>

getProcedureColumns返回一个ResultSet。每行描述单个参数或返回值。<br/>


getProcedureColumns的参数<br/>
JDBC API为getProcedureColumns方法调用定义了以下参数。<br/>

目录在RowStore中<br/>
始终为此参数使用null。<br/>

schemaPattern <br/>
Java程序有一个模式。<br/>

procedureNamePattern <br/>
表示过程名称模式的String对象。<br/>

column-Name-Pattern <br/>
表示参数名称或返回值名称的名称模式的String对象。Java程序的参数名称与CREATE PROCEDURE语句中定义的参数名称相匹配。使用“％”查找所有参数名称。<br/>


由getProcedureColumns返回的ResultSet中的列<br/>
由getProcedureColumns返回的ResultSet中的列如API所述。一些特定列的更多细节：<br/>
	
	PROCEDURE_CAT在RowStore中<br/>
	始终为“null”。<br/>
	
	PROCEDURE_SCHEM 
	模式为Java程序。
	
	PROCEDURE_NAME程序的
	名称
	
	COLUMN_NAME参数的
	名称 请参阅参数下的列名称模式以获取ProProcedureColumns。
	
	COLUMN_TYPE 
	指示行描述的缩写。它始终是DatabaseMetaData.procedureColumnIn方法参数，除非参数是一个数组。如果是，则为DatabaseMetaData.procedureColumnInOut。它总是返回DatabaseMetaData.procedureColumnReturn返回值。
	
	TYPE_NAME类型的
	RowStore特定名称。
	
	NULLABLE 
	始终返回DatabaseMetaData.procedureNoNulls为原语参数和DatabaseMetaData.procedureNullable为对象参数
	
	REMARKS描述
	方法参数的java类型的字符串。
	
	COLUMN_DEF 
	描述列的默认值的字符串（可以为空）。
	
	SQL_DATA_TYPE 
	由JDBC规范保留以供将来使用。
	
	SQL_DATETIME_SUB 
	由JDBC规范保留以备将来使用。
	
	CHAR_OCTET_LENGTH 
	基于二进制和字符的列的最大长度（或任何其他数据类型，返回值为NULL）。
	
	ORDINAL_POSITION 
	从1开始的顺序位置，用于过程的输入和输出参数。
	
	IS_NULLABLE 
	描述参数可空性的字符串（YES表示参数可以包含NULL，否则表示不能）。
	
	SPECIFIC_NAME 
	在其模式中唯一标识此过程的名称。
	
	METHOD_ID 
	RowStore特定列。
	
	PARAMETER_ID 
	RowStore特定列。


java.sql.DatabaseMetaData.getBestRowIdentifier方法<br/>
该java.sql.DatabaseMetaData.getBestRowIdentifier方法查找以下顺序标识：<br/>
	
	表上的主键。
	表上唯一的约束或唯一索引。
	表中的所有列。
	此订单可能不会返回唯一的行。

注意：如果java.sql.DatabaseMetaData.getBestRowIdentifier方法找不到主键，唯一约束或唯一索引，则该方法必须在表中的所有列中查找标识符。当该方法以这种方式查找标识符时，该方法将始终找到一组标识行的列。但是，如果表中有重复的行，则可能无法识别唯一的行。<br/>























##java.sql.Driver接口




java.sql.Driver.getPropertyInfo方法返回一个DriverPropertyInfo对象。在RowStore系统中，它包含数据库连接URL属性数组。（请参阅配置属性。）要获取DriverPropertyInfo对象，请从驱动程序管理器请求JDBC驱动程序：<br/>

		String url = "jdbc:snappydata:";
		Properties info = new Properties();
		java.sql.DriverManager.getDriver(url).getPropertyInfo(url, info);

java.sql.DriverManager.getConnection方法<br/>


使用JDBC API的Java应用程序通过获取Connection对象来建立与分布式系统的连接。获取Connection对象的标准方法是调用DriverManager.getConnection方法，该方法采用包含连接URL的String。JDBC连接URL（统一资源定位符）标识数据源。
<br/>
DriverManager.getConnection除连接URL之外还可以使用一个参数，一个Properties对象。您可以使用Properties对象来设置连接URL属性。
<br/>
您还可以提供表示用户名和密码的字符串。当它们被提供时，如果启用了用户认证，RowStore会检查它们是否对当前系统有效。用户名作为授权标识符传递给RowStore，用于确定用户是否被授权访问数据库并确定默认模式。当建立连接时，如果没有提供用户，RowStore将默认模式设置为APP。如果提供了用户，默认模式与用户名相同。
<br/>

RowStore连接URL语法<br/>
RowStore连接URL由连接URL协议（jdbc:），后跟子协议（gemfirexd:）和可选属性组成。
<br/>

对等客户端的连接URL的语法<br/>
对于在对等客户端中运行的应用程序，连接URL的语法为
<br/>
	    *jdbc:snappydata:_[;attributes]*_*
	jdbc:gemfirexd
在JDBC术语，gemfirexd是子协议用于连接到的GemFire分布式系统。子协议始终是gemfirexd，并不变化。
<br/>
  ###attributes
指定配置属性中详细说明的0个或更多连接URL 属性。
<br/>

附加SQL语法<br/>
RowStore还支持以下SQL标准语法来获取对服务器端JDBC例程中当前连接的引用：<br/>

      *jdbc:default:connection*















##java.sql.PreparedStatement接口


RowStore提供了所有必需的JDBC类型转换，并且还允许setXXX对每个类型使用单个方法，就像setObject(Value, JDBCTypeCode)调用一样。这意味着setString可以用于任何内置的目标类型。RowStore不支持游标; setCursorName方法抛出不支持的功能异常。<br/>
###准备语句和流列<br/>
setXXXStream在客户端和服务器之间请求流数据。JDBC允许将IN参数设置为Java输入流，以便以较小的块传递大量数据。当语句运行时，JDBC驱动程序会重复调用此输入流。RowStore支持JDBC API提供的三种类型的流：<br/>

setBinaryStream 用于包含未解释字节的流。<br/>
setAsciiStream 用于包含ASCII字符的流。<br/>
setUnicodeStream 用于包含Unicode字符的流。<br/>
传递给这三个方法的流对象可以是标准Java流对象，也可以是实现标准接口的用户自己的子类<br/>java.io.InputStream。根据JDBC标准，流只能存储在列中，数据类型如下表所示。<br/>


###可流水的JDBC数据类型
|列数据类型|	相应的Java类型|	AsciiStream|	UnicodeStream|	BinaryStream|
|--|--|--|--|
|CLOB|	java.sql.Clob中|	X|	X|	“|
|CHAR	|“	|X	|X	|“|
|VARCHAR|	“|	X|	X|	“|
|LONGVARCHAR|	“|	X|	X|	“|
|BINARY|	“|	X|	X|	X|
|BLOB|	java.sql.Blob中|	X|	X|	X|
|VARBINARY|	“|	X|	X|	X|
|LONGVARBINARY|	“|	X|	X|	X|
###注意：

大X表示流类型的首选目标数据类型。请参阅将java.sql.Types映射到SQL类型。
如果流存储在类型不同于LONG VARCHAR或LONG VARCHAR FOR BIT DATA的列中，则整个流必须能够一次适应内存。存储在LONG VARCHAR和LONG VARCHAR FOR BIT DATA列中的流没有此限制。
流不能存储在其他内置数据类型或用户定义数据类型列的列中。<br/>

例<br/>
以下示例显示用户如何将流式存储java.io.File在LONG VARCHAR列中：<br/>
	
	Statement s = conn.createStatement();
	s.executeUpdate("CREATE TABLE atable (a INT, b LONG VARCHAR)");
	conn.commit();
	java.io.File file = new java.io.File("derby.txt");
	int fileLength = (int) file.length();
	// first, create an input stream
	java.io.InputStream fin = new java.io.FileInputStream(file);
	PreparedStatement ps = conn.prepareStatement("INSERT INTO atable VALUES (?, ?)"); 
	ps.setInt(1, 1);
	// set the value of the input parameter to the input stream
	ps.setAsciiStream(2, fin, fileLength);
	ps.execute();
	conn.commit();






##java.sql.ResultSet接口

不要对结果集行承担任何隐式排序。如果您需要订购，请使用一个ORDER BY子句。

<br/>





##java.sql.SavePoint类
此版本的RowStore不支持SavePoints。<br/>
##类java.sql.SQLException

RowStore抛出的SQLExceptions并不总是匹配Derby数据库抛出的异常。在某些情况下，该getSQLState()方法SQLException也可能返回不同的值。






##java.sql.Statement类

RowStore不实现setEscapeProcessing方法java.sql.Statement。此外，取消方法引发“不支持功能”异常。
<br/>

ResultSet对象<br/>
首次执行SELECT语句时发生的错误会阻止ResultSet在其上打开对象。ResultSet如果在ResultSet打开后发生相同的错误，则不会关闭。例如，在executeQuery方法被调用java.sql.Statement或java.sql.PreparedStatement抛出异常并且根本没有返回结果集时发生的除以零的错误，而如果在对象next上调用该方法时发生相同的错误ResultSet，则不会关闭结果集。<br/>

ResultSet如果系统在获取第一行之前部分执行查询，则首先创建一个错误。任何使用多个表的查询和使用聚合GROUP BY，ORDER BY，DISTINCT，INTERSECT，EXCEPT或UNION的查询的任何查询都可能会发生此错误。（请参阅产品限制。）关闭Statement导致ResultSet该语句上的所有打开对象都被关闭。ResultSet可以在执行语句之前设置一个游标的游标名。但是，一旦执行，光标名不能被更改。<br/>







javax.sql.XADataSource中
<br/>
此版本的RowStore不支持javax.sql.XADataSource。<br/>
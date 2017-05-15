#  Snappy sql 规范
##一  查询
###1      根据表中内置的现有表或常量创建虚拟表。
语法：
<br/>
<code>
{ 
( Query ) 
| 
SelectExpression  | VALUES 
}
</code>
<br/>
描述：查询根据现有的表或内建的常量创建一个虚拟表。
<br/>
举例：
<br/>
<code>
-- a Select expression 
<br/>
SELECT * FROM TRADE.CUSTOMERS 
<br/>
-- a subquery 
<br/>
SELECT * FROM TRADE.CUSTOMERS C WHERE EXISTS (SELECT * FROM TRADE.PORTFOLIO F WHERE F.CID = C.CID AND TID =? AND F.QTY >927)
<br/>
-- a values expression 
<br/>
VALUES (1,2,3
</code>
<br/>
2描述：该查询必须使用单个列计算为单个行。
有时也称为表达式子查询。
<br/>
例
<br/>
-- avg always returns a single value, so the subquery is a ScalarSubquery 
<br/>
SELECT S.CID FROM TRADE.SELLORDERS S WHERE (SELECT SUM(QTY) FROM TRADE.PORTFOLIO F WHERE F.CID = S.CID GROUP BY F.CID) > 1277 AND TID =1
<br/>
###TableSubquery
<br/>
TableSubquery是一个返回多行的子查询。
<br/>
与ScalarSubquery不同，TableSubquery仅允许使用EXISTS，IN或量化比较。
<br/>
当与EXISTS一起使用时，只有使用*返回多个列时，才会返回多个列。
<br/>
当与IN或量化比较一起使用时，它必须返回单个列。
<br/>
注意：不支持在FROM子句中使用TableSubquery作为TableExpression，即使这不会引发语法错误。
<br/>
例子：
<br/>
<code>
-- a subquery used with EXISTS 
<br/>

SELECT C.CID FROM TRADE.CUSTOMERS C WHERE EXISTS (SELECT * FROM TRADE.PORTFOLIO F WHERE F.CID = C.CID AND TID = 1 AND F.QTY >927) 
-- a subquery used with IN 
SELECT F.CID FROM TRADE.PORTFOLIO F WHERE  F.CID IN (SELECT CID FROM  TRADE.CUSTOMERS C WHERE SINCE > ?) 
<br/>
-- a subquery used with a quantified comparison 
<br/>
SELECT S.CID FROM TRADE.SELLORDERS S WHERE (SELECT SUM(QTY) FROM TRADE.PORTTOLIO F WHERE F.CID = S.CID GROUP BY F.CID) > 1231 
</code>
<br/>
##二 语句

### 1  ALTER TABLE 使用ALTER TABLE语句将列和约束添加到现有表，从表中删除它们，或修改其他表功能，如AsyncEventListener实现和网关发送器配置。
<code>
ALTER TABLE table-name
<br/>
{
<br/>
  ADD COLUMN column-definition |
<br/>
  ADD table-constraint |
<br/>
  ALTER [ COLUMN ] column-name SET GENERATED ALWAYS AS IDENTITY |
<br/>
  DROP [ COLUMN ] column-name [ RESTRICT ] |
<br/>
  DROP { PRIMARY KEY | FOREIGN KEY constraint-name | UNIQUE 
<br/>
    constraint-name | CHECK constraint-name | CONSTRAINT constraint-name } |
<br/>
  SET EVICTION MAXSIZE integer-constant |
<br/>
  SET EVICTION FREQUENCY integer-constant { SECONDS | HOURS | DAYS } [ START { D 'date-constant' | T 'time-constant' | TS 'timestamp-constant' } ] |
  SET GATEWAYSENDER ( [sender-name] [, sender-name] * ) |
  SET ASYNCEVENTLISTENER ( [listener-name] [, listener-name] * )
<br/>
}
<br/>
</code>
###2 CREATE语句 使用Create语句来创建函数，索引，过程，模式，同义词，表，触发器和视图。
<br/>
创建：表
<br/>
语法：
<br/>
<code>
CREATE TABLE table-name {
<br/>
    ( { column-definition | table-constraint }
<br/>
    [ , { column-definition | table-constraint } ] * )
| 
    [ ( column-name [, column-name ] * ) ]
    AS query-expression
    WITH NO DATA
<br/>
}
<br/>
    [ partitioning-clause | REPLICATE ]
<br/>
    [ server-groups-clause ]
<br/>
    [ gateway-sender-clause ]
<br/>
    [ async-event-listener-clause ]
<br/>
    [ eviction-clause | eviction-by-criteria-clause]
<br/>
    [ expiration-clause ] *
<br/>
    [ persistence-clause ]
<br/>
    [ offheap-clause ]
<br/>

 CREATE TABLE HOTELAVAILABILITY
<br/>
    (HOTEL_ID INT NOT NULL, 
<br/>
     BOOKING_DATE DATE NOT NULL,
<br/>
     ROOMS_TAKEN INT DEFAULT 0,
<br/>
     PRIMARY KEY (HOTEL_ID, BOOKING_DATE));
<br/>
###CREATE PROCEDURE 创建可以使用CALL语句调用的Java存储过程。
<br/>
CREATE PROCEDURE procedure-name
<br/>
([ procedure-parameter [, procedure-parameter] * ])
<br/>
[ procedure-element ] *
<br/>

###CREATE PROCEDURE创建可以使用CALL语句调用的Java存储过程。在当前成员上调用一个过程。因为这个过程是用Java代码编写的，所以它可以在内部使用JDBC调用SQL命令。合格的过程名称不能使用为内部使用保留的SYS模式。以下部分描述语法。
<br/>
RowStore不支持在过程或函数正文中执行DDL语句。
<br/>

### 3 CREATE HDFSSTORE 创建与Hadoop名称节点的连接，以便将一个或多个表持久化到HDFS。
<br/>
创建与Hadoop名称节点的连接，以便将一个或多个表持久化到HDFS。每个连接定义用于持久化数据的HDFS NameNode和目录，以及用于配置用于持久化表事件的队列，启用持久性连接，压缩HDFS操作日志等的RowStore特定选项。
<br/>
<code>
CREATE HDFSSTORE store-name
<br/>
  NAMENODE 'url'
<br/>
  [ HomeDir 'directory-name' ]<br/>
  [ BatchSize integer-constant ]<br/>
  [ BatchTimeInterval integer-constant { MILLISECONDS | SECONDS | MINUTES | HOURS | DAYS } ]<br/>
  [ MaxQueueMemory integer-constant ]<br/>
  [ DispatcherThreads integer-constant ]<br/>
  [ QueuePersistent boolean-constant ] <br/>
  [ DiskSynchronous boolean-constant ]<br/>
  [ DiskStoreName store-name ]<br/>
  [ MinorCompact boolean-constant ]<br/>
  [ MaxInputFileSize integer-constant ]<br/>
  [ MinInputFileCount integer-constant ]<br/>
  [ MaxInputFileCount integer-constant ]<br/>
  [ MinorCompactionThreads integer-constant ]<br/>
  [ MajorCompact boolean-constant ]<br/>
  [ MajorCompactionInterval integer-constant { MILLISECONDS | SECONDS | MINUTES | HOURS | DAYS } ]<br/>
  [ MajorCompactionThreads integer-constant ]<br/>
  [ MaxWriteOnlyFileSize integer-constant ]<br/>
  [ WriteOnlyFileRolloverInterval  integer-constant { MILLISECONDS | SECONDS | MINUTES | HOURS | DAYS } ]<br/>
  [ BlockCacheSize integer-constant ]<br/>
  [ ClientConfigFile 'file-name' ]<br/>
  [ PurgeInterval integer-constant { MILLISECONDS | SECONDS | MINUTES | HOURS | DAYS } ]<br/>
</code>
<br/>
注意：
<br/> HDFSSTORE可以使用HDFS只写模型或HDFS读/写模型来保存表数据; 您可以使用CREATE TABLE语句指定HDFS持久性模型。虽然多个表可以使用相同的HDFSSTORE来保持持久性，但通常需要创建多个HDFSSTORE配置来修改每个表的队列和压缩行为。
<br/>
HDFS访问的所有身份验证和授权与执行RowStore成员进程的用户相关联。例如，当您在RowStore中创建HDFS存储时，商店的默认目录结构是在HDFS 的/ user / * user-name *目录下创建的，其中* user-name *是RowStore的进程所有者。（您可以通过指定以“/”字符开头的绝对路径，或者使用hdfs-root-dir属性为服务器设置备用HDFS根目录来覆盖此默认值。）RowStore用户必须能够使用Kerberos进行身份验证，并且必须对每个Hadoop集群节点上的HDFS存储目录具有读/写权限。请参阅安全HDFS的配置要求。
<br/>
###CREATE VIEW 视图是由查询形成的虚拟表。
<br/>
CREATE VIEW view-name<br/>
    [ ( simple-column-name [, simple-column-name] * ) ]<br/>
AS view_query <br/>
视图是由查询形成的虚拟表。视图是一个字典对象，您可以使用它，直到放置它。视图不可更新。<br/>
注意：
<br/>您在CREATE VIEW语句中提供的查询不能使用以下任何SQL子句： - ORDER BY -OFFSET和FETCH FIRST子句 此外，RowStore不支持涉及分组，聚合，独立或连接操作的视图一个分区表。<br/>
视图所有者自动获得视图上的SELECT权限。SELECT权限无法从视图所有者撤销。启动进程的分布式成员所有者自动获得该视图的SELECT权限，并且可以将此权限授予其他用户。分配的成员所有者不能撤销SELECT权限。CREATE SCHEMA提供额外的所有权细节。<br/>
###4   DELETE 从表中删除行。
<br/>
     语法：
<br/>
{
<br/>
    DELETE FROM table-name [ [ AS ] correlation-name ]<br/>
        [ WHERE ]<br/>
}<br/>
描述：<br/>
此表单称为搜索删除，删除由表名和WHERE子句标识的所有行。<br/>
###5   DROP语句   删除语句用于删除函数，索引，过程，模式，表，触发器和视图。
###DROP INDEX 删除指定的索引。
<br/>
DROP INDEX [ IF EXISTS ] [schema-name.]index-name<br/>
描述：<br/>
删除给定模式中的索引（或当前模式，如果没有提供）。IF EXISTS只有当RowStore中存在指定的索引时，才包括子句来执行该语句。<br/>
###DROP DISKSTORE 从RowStore集群中删除磁盘存储配置。
<br/>
DROP DISKSTORE [ IF EXISTS ] store-name
<br/>
IF EXISTS 只有当RowStore中存在指定的磁盘存储区时，才
包括该IF EXISTS子句来执行该语句。<br/>
store-name <br/>
要删除的磁盘存储配置的用户定义名称。可用的名称存储在SYSDISKSTORES系统表中。<br/>
例：<br/>
此命令从集群中删除磁盘存储“STORE1”：<br/>
DROP DISKSTORE store1;<br/>
###DROP SCHEMA 
<br/>
删除指定的空模式。<br/>
句法<br/>
DROP SCHEMA [ IF EXISTS ] schema-name RESTRICT<br/>
描述<br/>
IF EXISTS仅当RowStore中存在指定的模式时，才包括子句来执行该语句。RESTRICT关键字强制执行规则，即在指定的架构中不能定义要从数据库中删除的架构的对象。RESTRICT关键字是必需的，因为RowStore不能删除包含数据库对象的模式。<br/>
例<br/>
<code>
DROP SCHEMA EMP RESTRICT;</code>
<br/>
如果EMP模式中没有定义任何对象，则EMP架构只能从数据库中删除。
<br/>
###DROP TABLE 删除指定的表。<br/>
DROP TABLE [ IF EXISTS ] [schema-name.]table-name<br/>
DROP TRIGGER 删除指定的触发器。<br/>
删除指定的视图。<br/>
句法：<br/>
DROP VIEW [ IF EXISTS ] view-name<br/>
描述：<br/>
   删除指定的视图。IF EXISTS只有当RowStore中存在指定的视图时，才包括子句来执行该语句。<br/>
###6   GRANT 启用特定用户或所有用户的权限。
<br/>
###7    INSERT INSERT语句创建一行或多行并将其存储在命名表中。在INSERT语句中分配的值的数目必须与指定或隐含列的数量相同。
<br/>
###8    REVOKE 撤销对表或例程的权限。
<br/>
###9     SELECT 具有可选ORDER BY CLAUSE和可选的FOR UPDATE子句的查询。
<br/>
<code>
SELECT COUNT(*),SID,AVG(QTY)<br/>
FROM TRADE.PORTFOLIO<br/>
GROUP BY SID<br/>
ORDER BY 3<br/>
</code>
WHERE子句使得根据布尔表达式从结果中过滤行。结果中只返回表达式求值为TRUE的行。<br/>
GROUP BY子句将结果中的行分组为具有一个或多个列的匹配值的子集。GROUP BY子句通常与聚合一起使用。<br/>
RowStore处理一个SelectExpression按以下顺序：<br/>
###FROM子句<br/>
###WHERE子句
###GROUP BY（或隐式GROUP BY）
###HAVING子句
###SELECT子句
###10    SET ISOLATION 更改连接的事务隔离级别。
<br/>
###11    SET SCHEMA 设置或更改连接会话的默认模式。
<br/>
###12    TRUNCATE TABLE 从表中删除所有内容，并将其返回到其初始的空状态。TRUNCATE TABLE清除指定表的所有内存中数据以及持久存储到RowStore磁盘存储的任何数据。对于HDFS读写表，TRUNCATE TABLE还标记表的HDFS持久性文件以进行验证。对于HDFS只写表，TRUNCATE TABLE保留存储在HDFS日志文件中的所有表数据，以便稍后使用MapReduce或HAWQ进行处理。
<br/>
WHERE子句使得根据布尔表达式从结果中过滤行。结果中只返回表达式求值为TRUE的行。<br/>
GROUP BY子句将结果中的行分组为具有一个或多个列的匹配值的子集。GROUP BY子句通常与聚合一起使用。<br/>
###13   UPDATE 更新一个或多个列的值。
<br/>
句法：
<br/>
<code>
{ UPDATE table-name [ [ AS ] correlation-name]
        SET column-name = value
        [, column-name = value} ]*
        [ WHERE ]    
}
</code>
描述：<br/>
      这种形式的UPDATE语句称为搜索更新。它将为WHERE子句评估为TRUE的表的所有行更新一个或多个列的值。
<br/>为更新值指定DEFAULT将列的值设置为为该表定义的默认值。<br/>
UPDATE语句返回更新的行数。<br/>


###表达式
<code>
{
<br/>
VALUES ( Value {, Value }* )
<br/>
[ , ( Value {, Value }* ) ]* |
<br/>
VALUES Value [ , Value ]*
<br/>
}<br/>
</code>
其中Value定义为：<br/>
<code>
Expression | DEFAULT<br/>
INSERT INTO TRADE.CUSTOMERS (CID, CUST_NAME, ADDR,TID)
<br/>
VALUES (1, 'JOHN', 'VMWARE ',1),
<br/>
(2,'SMITH', 'VMWARE', 2)
<br/>
INSERT INTO TRADE.SELLORDERS (OID , CID, SID, STATUS, TID)
<br/>
VALUES (1,1,2,DEFAULT,2)
<br/>
-- using a built-in function
<br/>
VALUES CURRENT_DATE
<br/>
-- getting the value of an arbitrary expression
<br/>
VALUES (3*29, 26.0E0/3)
</code>
<br/>
###WHERE子句
<code>
WHERE Boolean expressions
<br/>
SELECT *
<br/>
FROM FlightAvailability
<br/>
WHERE business_seats_taken IS NULL
<br/>
OR business_seats_taken = 0
<br/>

SELECT THIS_EMP.EMPNO, THIS_EMP.SALARY, DINFO.AVGSALARY, DINFO.EMPCOUNT
<br/>
 FROM EMPLOYEE THIS_EMP,<br/>
   (SELECT OTHERS.WORKDEPT AS DEPTNO,<br/>
           AVG(OTHERS.SALARY) AS AVGSALARY,<br/>
           COUNT(*) AS EMPCOUNT<br/>
    FROM EMPLOYEE OTHERS<br/>
    GROUP BY OTHERS.WORKDEPT<br/>
   )AS DINFO<br/>
 WHERE THIS_EMP.JOB = 'SALESREP'<br/>
           AND THIS_EMP.WORKDEPT = DINFO.DEPTNO<br/>






SELECT [ DISTINCT | ALL ] SelectItem [ , SelectItem ]*<br/>
FROM<br/>
[ WHERE]<br/>
GROUP BY<br/>
HAVING<br/>

SELECT COUNT(*),SID,AVG(QTY)<br/>
FROM TRADE.PORTFOLIO<br/>
GROUP BY SID<br/>
ORDER BY 3<br/>
</code>
##数据类型Data Types
###BIGINT
       为长整数值提供8字节存储。
###BLOB
        二进制大对象表示长度不同的原始字节数组。
###CHAR
       提供固定长度的字符串。
###CHAR FOR BIT DATA
###提供固定长度的字节串
###CLOB
###DATE
###提供将日期存储为年月日
###DECIMAL提供具有指定精度和比例的精确十进制值。
###DOUBLE  Provides 8-byte storage for numbers using IEEE floating-point notation.
###DOUBLE PRECISION
###FLOAT
###INTEGER
###JSON
###LONG VARCHAR
###LONG VARCHAR FOR BIT DATA
###NUMERIC
###REAL
###SMALLINT
###TIME
###TIMESTAMP
###User-Defined Types
###VARCHAR
###VARCHAR FOR BIT DATA
###XML

###BIGINT为长整数值提供8字节存储。如果该值溢出较小类型允许的最大值，则尝试将BIGINT值放入具有较小尺寸/精度的另一精确数字类型（例如INT）中。
###对于表达式中其他类型的行为，请参阅表达式中的数字类型升级，将另一数字数据类型的列中的一个数字数据类型的值存储。
<br/>
等效的Java类型      	java.lang.Long<br/>
最小值            	java.lang.Long.MIN_VALUE（-9223372036854775808）<br/>
最大值           	java.lang.Long.MAX_VALUE（9223372036854775807）<br/>
JDBC元数据类型   	java.sql.Types.BIGINT<br/>
JDBC方法	            ResultSet.getLong，PreparedStatement.setLong<br/>
###BLOB
###二进制大对象表示长度不同的原始字节数组。
<br/>
等效的Java类型	        java.lang.Blob<br/>
最大长度（也是默认长度）  	2 GB - 1（或2,147,483,647）<br/>
JDBC元数据类型	        java.sql.Types.BLOB<br/>
JDBC方法	                ResultSet.getBlob，PreparedStatement.setBlob<br/>

###{ BLOB | BINARY LARGE OBJECT } [ ( length [{ K | M | G }] ) ] 
默认情况下，BLOB的长度以字节数表示。后缀K，M和G代表千字节，兆字节和千兆字节，分别使用1024，1024 * 1024或1024 * 1024 * 1024的倍数。
<br/>
<code>
CREATE TABLE blob_data(id INT primary key, data BLOB(10M)); 
–- search for a blob 
select length(data) from blob_data where id = 100;
</code>
###CHAR
提供固定长度的字符串。如果字符串值小于预期长度，则插入空格以将字符串填充到预期长度。如果字符串值长于预期长度，则会修剪任何尾部空白以使长度与预期长度相同，如果需要截断空格以外的字符，则会引发异常。对于比较操作，较短的CHAR字符串用空格填充到较长的值。类似地，当在表达式中混合CHAR和VARCHAR时，较短的值用空格填充到更长的字符串长度。
<br/>
要在字符串中表示单引号，请使用两个引号：<br/>
VALUES 'going to Chandra''s place' 
<br/>
CHAR的长度是无符号整数常数。<br/>
	
等效的Java类型	java.lang.String<br/>
最大长度	        java.lang.Integer.MAX_VALUE（2147483647）<br/>
默认长度       	1<br/>
JDBC元数据类型	java.sql.Types.CHAR<br/>
JDBC方法	        ResultSet.getString，PreparedStatement.setString<br/>
CHAR            [ACTER] [(length)] <br/>
###字数数据
提供固定长度的字节串。由于非结构化数据或其他原因，CHAR类型不合适时使用。如果值小于预期长度，则填充0x20字节值。对于CHAR FOR BIT DATA和VARCHAR FOR BIT DATA类型，根据SQL-92标准（这与某些DBMS的行为方式不同），不执行填充。涉及VITAR FOR BIT DATA和CHAR FOR BIT DATA的操作产生VARCHAR FOR BIT DATA类型的值。<br/>
长度是以字节为单位的值的长度的无符号整数常量。
	
等效的Java类型	根据需要使用byte []，java.io.InputStream或java.lang.OutputStream
最大长度	254字节
默认长度	1
JDBC元数据类型	java.sql.Types.BINARY
JDBC方法	ResultSet.getBytes，PreparedStatement.setBytes
###{ CHAR | CHARACTER }[(length)] FOR BIT DATA 
###CLOB
字符大对象表示长度不同的字符数组。它用于存储大量基于字符的数据，如文档。<br/>
长度以数字字符表示，除非您指定分别使用1024，1024 * 1024或1024 * 1024 * 1024的倍数的后缀K，M或G。<br/>
	
等效的Java类型	java.sql.Clob<br/>
最大长度（也是默认长度）	2 GB - 1（或2,147,483,647）<br/>
JDBC元数据类型	java.sql.Types.CLOB<br/>
JDBC方法	ResultSet.getClob，PreparedStatement.setClob<br/>
{ CLOB | CHARACTER LARGE OBJECT } [ ( length [{ K | M | G }] ) ] <br/>
<code>
CREATE TABLE clob_data(id INT primary key, text CLOB(10M)); <br/>
–- search for a clob<br/>
select text from clob_data where id = 100;<br/>
</code>

###日期   
###提供将日期存储为年月日。支持的格式有：yyyy-mm-dd  mm/dd/yyyy  dd.mm.yyyy 
年（yyyy）必须始终以四位数表示，而月（mm）和日（dd）可能有一位或两位数字。DATE，TIME和TIMESTAMP不得在表达式中彼此混合，除了具有明确的CAST。<br/>
	
等效的Java类型	java.sql.Date<br/>
JDBC元数据类型	java.sql.Types.DATE<br/>
JDBC方法	ResultSet.getDate，PreparedStatement.setDate<br/>
VALUES '2010-05-04'<br/>
VALUES DATE('2001-10-12')<br/>
后一个示例使用内置函数和过程中描述的DATE（）函数。<br/>
###DECIMAL
提供具有指定精度和比例的精确十进制值。精度是小数点左侧和右侧的总位数，而缩放比例是小数点右侧小数位数。<br/>
只要非分数精度不会丢失，数值（例如INT，BIGINT，SMALLINT）可以放入DECIMAL，否则抛出范围异常（SQLState：“22003”）。当截断来自DECIMAL的尾数时，该值被舍入。<br/>
对于表达式中其他类型的行为，请参阅表达式中的数字类型升级，十进制算术的缩放和另一数字数据类型的列中的一个数值数据类型的值。<br/>
	
等效的Java类型	java.math.BigDecimal<br/>
精度最小/最大	1到31<br/>
缩小最小/最大	小于或等于精度<br/>
默认精度	5<br/>
默认比例	0<br/>
JDBC元数据类型	java.sql.Types.DECIMAL<br/>
JDBC方法	ResultSet.getBigDecimal，PreparedStatement.setBigDecimal<br/>
{ DECIMAL | DEC } [(precision [, scale ])]<br/>
-- this cast loses fractional precision <br/>
values cast (23.8372 AS decimal(4,1)); <br/>
-–- results in:<br/> 
23.8 <br/>
-- this cast is outside the range <br/>
values cast (97824 AS decimal(4,1)); <br/>
–-- throws exception: <br/>
ERROR 22003: The resulting value is outside the range for the data type DECIMAL/NUMERIC(4,1). <br/>
###FLOAT
使用IEEE浮点表示法为数字提供8字节存储。<br/>
算术运算不会使其结果值为零。如果值太小，您将收到异常。数字浮点常数限制为30个字符的长度。<br/>
对于表达式中其他类型的行为，请参阅表达式中的数字类型升级，并在另一数字数据类型的列中存储一个数字数据类型的值。<br/>
	
等效的Java类型	java.lang.Double<br/>
注意：如下所述，最大/最小限制与java.lang.Double的不同。<br/>
最小值	-1.79769E + 30<br/>
最大值	1.79769E + 308<br/>
最小正值	2.225E-307<br/>
最大的负值	-2.225E-307<br/>
默认精度	5<br/>
默认比例	0<br/>
JDBC元数据类型	java.sql.Types.DOUBLE<br/>
JDBC方法	ResultSet.getDouble，PreparedStatement.setDouble<br/>
–- examples of valid values <br/>
values 233.31E3; <br/>
values 8928E+06; <br/>
-- this example will throw a range exception (SQLState: "42820") <br/>
values 123456789012345678901234567890123456789e0; <br/>
###双重精准<br/>
###Synonyn为DOUBLE。<br/>
###浮动<br/>
具有REAL或DOUBLE数据类型的别名，具体取决于指定的精度。默认精度为53，相当于DOUBLE。23或更小的精度使FLOAT等效于REAL，而大于23则相当于DOUBLE。<br/>
	
等效的Java类型	java.lang.Double或java.lang.Float取决于精度<br/>
最小/最大限度	如果精度小于23，则与FLOAT相同。否则，与DOUBLE相同的最小/最大极限相同。<br/>
默认精度	53<br/>
JDBC元数据类型	java.sql.Types.FLOAT<br/>
JDBC方法	ResultSet.getFloat / getDouble，PreparedStatement.setFloat / setDouble<br/>
FLOAT [(precision)]<br/>
###整数
为整数值提供4字节存储。INT可以用作CREATE TABLE中的INTEGER的同义词。<br/>
对于表达式中其他类型的行为，请参阅表达式中的数字类型升级，并在另一数字数据类型的列中存储一个数字数据类型的值。<br/>
	
等效的Java类型	java.lang.Integer<br/>
最小值	java.lang.Integer.MIN_VALUE（-2147483648）
<br/>
最大值	java.lang.Integer.MAX_VALUE（21474836487）
<br/>
JDBC元数据类型	java.sql.Types.INTEGER<br/>
JDBC方法	ResultSet.getInt，PreparedStatement.setInt   <br/>
###JSON
###JSON格式（JavaScript对象符号）字符串。JSON类型的列可以与JSON_EVALPATH函数一起使用，以提取JSON属性或使用JSON路径过滤查询结果。
###RowStore尝试在插入期间使用public com.gemstone.gemfire.pdx.JSONFormatter API 解析插入的数据为JSON 。如果RowStore无法将数据解析为JSON，则抛出SQLException并且插入失败。
###注意： RowStore不保留您插入的JSON数据中存在的空格或其他格式。此外，RowStore在查询JSON列值时，会添加空格和换行符来格式化字符串输出。RowStore不对JSON数据类型的列执行索引。
	
等效的Java类型	java.sql.Clob   <br/>
最大长度（也是默认长度）	2 GB - 1（或2,147,483,647字节）  <br/>
JDBC元数据类型	java.sql.Types.CLOB  <br/>
JDBC方法	ResultSet.getClob，PreparedStatement.setClob <br/>
<code>
CREATE TABLE json_store (id INT primary key, storedata JSON); 
–- insert JSON data as a string value delimited by single quotes
INSERT INTO json_data VALUES ( 1, 
'{ "store": {
    "book": [
      { "category": "reference",
        "author": "Nigel Rees",
        "title": "Sayings of the Century",
        "price": 8.95
      },
      { "category": "fiction",
        "author": "Evelyn Waugh",
        "title": "Sword of Honour",
        "price": 12.99
      },
      { "category": "fiction",
        "author": "Herman Melville",
        "title": "Moby Dick",
        "isbn": "0-553-21311-3",
        "price": 8.99
      },
      { "category": "fiction",
        "author": "J. R. R. Tolkien",
        "title": "The Lord of the Rings",
        "isbn": "0-395-19395-8",
        "price": 22.99
      }
    ],
    "bicycle": {
      "color": "red",
      "price": 19.95
    }
  }
}' );
</code>
<br/>
另请参见JSON_EVALPATH中的示例。<br/>
###LONG VARCHAR
与VARCHAR相同，但不必指定最大长度，为32700。<br/>
	
等效的Java类型	java.lang.String<br/>
最大长度	32700<br/>
JDBC元数据类型	java.sql.Types.LONGVARCHAR<br/>
JDBC方法	ResultSet.getString，PreparedStatement.setString<br/>
用于位数据的长VARCHAR<br/>
与VARCHAR FOR BIT DATA相同，除了不需要指定最大长度为32700。<br/>
	
等效的Java类型	根据需要使用byte []或java.io.InputStream / java.lang.OutputStream<br/>
最大长度	32700字节<br/>
JDBC元数据类型	java.sql.Types.LONGVARBINARY<br/>
JDBC方法	ResultSet.getBytes，PreparedStatement.setBytes<br/>
NUMERIC<br/>
DECIMAL数据类型的同义词。<br/>
下面列出了DECIMAL的元数据差异。否则，NUMERIC的行为与DECIMAL相同。<br/>
	
JDBC元数据类型	java.sql.Types.NUMERIC<br/>
###NUMERIC [(precision [, scale ])]<br/>
###REAL<br/>
使用IEEE浮点表示法为数字提供4字节存储。<br/>
算术运算不会使其结果值为零。如果值太小，您将收到异常。常量总是映射到DOUBLE - 使用显式CAST将常量转换为REAL。<br/>
对于表达式中其他类型的行为，请参阅表达式中的数字类型升级，将另一数字数据类型的列中的一个数字数据类型的值存储。<br/>
最大/最小限制与以下所述的java.lang.Float不同。<br/>
	
等效的Java类型	java.lang.Float<br/>
最小值	-3.402E + 38<br/>
最大值	3.402E + 38<br/>
最小正值	1.175E-37<br/>
最大的负值	-1.175E-37<br/>
JDBC元数据类型	java.sql.Types.REAL<br/>
JDBC方法	ResultSet.getFloat，PreparedStatement.setFloat<br/>
–- examples of valid values <br/>
values cast (233.31E3 as REAL); <br/>
values cast (8928E+06 as REAL); <br/>
-- this example will throw a range exception (SQLState: "42820") <br/>
values cast (123456789012345678901234567890123456789e0 as REAL);<br/>
###SMALLINT
为短整型值提供2字节存储。<br/>
对于表达式中其他类型的行为，请参阅表达式中的数字类型升级，并在另一数字数据类型的列中存储一个数字数据类型的值。<br/>
	
等效的Java类型	java.lang.Short<br/>
最小值	java.lang.Short.MIN_VALUE（-32768）<br/>
最大值	java.lang.Short.MAX_VALUE（32767）<br/>
JDBC元数据类型	java.sql.Types.SMALLINT<br/>
JDBC方法	ResultSet.getShort，PreparedStatement.setShort<br/>
时间TIME<br/>
提供以小时 - 分钟秒的时间存储。支持的格式有：<br/>
hh:mm[:ss] <br/>
hh.mm[.ss] <br/>
hh[:mm] {AM | PM} <br/>
小时（hh）可能有一个或两个数字，而分钟（mm）和秒（ss）（如果存在）必须有两位数字。DATE，TIME和TIMESTAMP不得在表达式中彼此混合，除了具有明确的CAST。<br/>
	
等效的Java类型	java.sql.Time<br/>
JDBC元数据类型	java.sql.Types.TIME<br/>
JDBC方法	ResultSet.getTime，PreparedStatement.setTime<br/>
VALUES '13:23:12' <br/>
VALUES TIME('13:23:12')<br/>
后一个例子使用内置函数和过程中描述的TIME（）函数。<br/>
###TIMESTAMP<br/>
提供DATE和TIME两者的组合值。此外，它允许具有最多六位数的小数秒。支持的格式有：<br/>
yyyy-MM-dd hh:mm:ss[.nnnnnn] <br/>
yyyy-MM-dd-hh.mm.ss[.nnnnnn] <br/>
年（yyyy）必须始终以四位数表示。月（MM），天（dd）和小时（hh）可能有一位或两位数字，而分钟（mm）和秒（ss）必须有两位数字。微秒（如果存在）可能有一到六位数。DATE，TIME和TIMESTAMP不得在表达式中彼此混合，除了具有明确的CAST。<br/>
	
等效的Java类型	java.sql.Timestamp<br/>
JDBC元数据类型	java.sql.Types.TIMESTAMP<br/>
JDBC方法	ResultSet.getTimestamp，PreparedStatement.setTimestamp<br/>
VALUES '2000-02-03 12:23:04' <br/>
VALUES TIMESTAMP(' 2000-02-03 12:23:04.827') <br/>
VALUES TIMESTAMP('2000-02-03 12:23:04')<br/>
后一个示例使用内置函数和过程中描述的TIMESTAMP（）函数。<br/>
用户定义的类型  User-Defined Types<br/>
RowStore允许您创建用户定义的类型。用户定义的类型是一个可序列化的Java类，其实例存储在列中。该类必须实现java.io.Serializable接口。为了有效的数据传输和存储，该类也应该实现java.io.Externalizable接口。如果该类实现了Apache Geode com.gemstone.gemfire.DataSerializable接口，那么它将被用于最佳的存储和对等传输。<br/>
有关创建和删除类型的信息，请参阅CREATE TYPE和DROP TYPE。<br/>
有关编写实现用户定义类型的Java类的信息，请参阅编程用户定义的类型。<br/>
###VARCHAR
提供长度最大限制的可变长度字符串。如果字符串值大于最大长度，则将修剪任何尾部空白以使其长度与最大长度相同，如果需要截断空格以外的字符，则会引发异常。在表达式中混合CHAR和VARCHAR时，较短的值用空格填充到更长的字符串长度。<br/>
字符串常量的类型是CHAR，而不是VARCHAR。要在字符串中表示单引号，请使用两个引号：<br/>
VALUES 'going to Chandra''s place' <br/>
VARCHAR的长度是无符号整数常数。<br/>
	
等效的Java类型	java.lang.String<br/>
最大长度	32672<br/>
JDBC元数据类型	java.sql.Types.VARCHAR<br/>
JDBC方法	ResultSet.getString，PreparedStatement.setString<br/>
{ VARCHAR | CHAR VARYING | CHARACTER VARYING }(length)<br/>
VARCHAR FOR BIT DATA<br/>
提供长度最大限制的可变长度字节串。当由于非结构化数据或其他原因而导致字符类型不合适时使用。对于CHAR FOR BIT DATA和VARCHAR FOR BIT DATA类型，根据SQL-92标准（这与某些DBMS的行为方式不同），不执行填充。涉及VITAR FOR BIT DATA和CHAR FOR BIT DATA的操作产生VARCHAR FOR BIT DATA类型的值。<br/>
字节字面值的类型始终是位数据的VARCHAR，而不是字节数据的CHAR。<br/>
长度是以字节为单位的值的长度的无符号整数常量。<br/>
	
等效的Java类型	根据需要使用byte []或java.io.InputStream / java.lang.OutputStream
<br/>
最大长度	32672字节<br/>
JDBC元数据类型	java.sql.Types.VARBINARY<br/>
JDBC方法	ResultSet.getBytes，PreparedStatement.setBytes<br/>
{ VARCHAR | CHAR VARYING | CHARACTER VARYING }[(length)] FOR BIT DATA <br/>
###XML
此类型可用于可扩展标记语言（XML）文档：<br/>
存储符合格式良好的XML（DOCUMENT（ANY））值的SQL / XML定义的XML文档<br/>
瞬态地为XML（SEQUENCE）值，可能不是格式正确的XML（DOCUMENT（ANY））值<br/>
注意： RowStore不支持SQL标准中的SQLXML类型，或者是SQLXML类型，而是将值作为普通字符串存储，并且用户必须根据需要明确地使用XMLSERIALIZE，XMLPARSE函数进行插入和查询。<br/>
对于执行XML相关功能的应用程序，应用程序必须使用内置JAXP / Xalan支持的JDK（例如Sun JDK5），或者必须为JAXP解析器和服务器类路径中的Xalan提供类（serializer.jar，xalan.jar ）。<br/>
要使用JDBC或ADO.NET驱动程序从RowStore数据库获取XML值，请在SQL查询中使用XMLSERIALIZE运算符。例如：
SELECT XMLSERIALIZE(doc as CLOB) FROM t1<br/>
然后使用与目标序列化类型相对应的ResultSet.getXXX方法检索XML值（例如，使用CLOB数据类型的上述示例中的getClob）。要使用JDBC将XML值存储到RowStore数据库中，请在SQL语句中使用XMLPARSE运算符。例如：<br/>
<code>
INSERT INTO t1(doc) VALUES XMLPARSE( DOCUMENT CAST (? AS CLOB) PRESERVE WHITESPACE)
</code>
然后使用与String类型兼容的任何PreparedStatement.setXXX方法（例如，在上述示例中使用
</br>
PreparedStatement.setString或PreparedStatement.setCharacterStream）。</br>
	
等效的Java类型	java.lang.String; 使用显式XML函数进行插入/查询，如上所述</br>
最大长度	java.lang.Integer.MAX_VALUE（2147483647）</br>
JDBC元数据类型	没有</br>
JDBC方法	根据目标序列化类型</br>
##Join

able-name [ INNER JOIN, [ LEFT OUTER JOIN, RIGHT ] OUTER ] JOIN table-name ON boolean-expression
<br/>
###INNER JOIN：指定具有explicit连接子句的两个表之间的连接。</br>
指定一个显式连接子句。</br>
句法</br>
<code>
TableExpression  [ INNER ] JOIN TableExpression  
{ ON boolean-expression }
</code>
<br/>
描述<br/>
另见查询功能和限制。<br/>
可以使用带有布尔表达式的ON来指定连接子句。ON子句中的表达式范围包括当前表和当前SELECT的外部查询块中的任何表。在以下示例中，ON子句引用当前表：<br/>
<code>
Select * from trade.securities INNER JOIN trade.portfolio 
     ON SECURITIES.sec_id = PORTFOLIO.sid and PORTFOLIO.tid = ?
</code>
<br/>
ON子句可以引用未连接的表，并且不必引用要连接的任何表（尽管通常是这样）。<br/>

例<br/>
<code>
-- Join the Customers and Portfolio tables<br/>
-- select all the columns from the customers table and<br/>
-- add the SID and QTY from the Portfolio table<br/>
-- to each row of the result<br/>
select C.* , F.sid, F.QTY from trade.customers  C JOIN trade.portfolio F ON C.cid = F.cid
</code><br/>
###LEFT OUTER JOIN：
<br/>
指定具有显式连接子句的两个表之间的连接，从第一个表保留不匹配的行。
<br/>
<code>
TableExpression  LEFT [ OUTER ] JOIN TableExpression <br/>
{
<br/>
ON boolean-expression<br/>
}
<br/>
</code>
描述<br/>
另见查询功能和限制。<br/>
ON子句中的表达式范围包括当前表和当前SELECT外部的查询块中的任何表。ON子句可以引用未连接的表，并且不必引用要连接的任何表（尽管通常是这样）。<br/>
例<br/>
<code>
--match all customers and their portfolio details, if any, having a specific<br/> 
--transaction ID<br/>
select * from trade.customers C LEFT OUTER JOIN trade.portfolio F on C.cid= F.cid where F.tid = ? <br/>
-- use the synonymous syntax, LEFT JOIN, to achieve exactly<br/>
-- the same results as in the example above<br/>
select * from trade.customers C LEFT  JOIN trade.portfolio F on C.cid= F.cid where F.tid = ?<br/>
</code><br/>
###RIGHT OUTER JOIN：指定具有显式连接子句的两个表之间的连接，从第二个表中保留不匹配的行。<br/>
在所有情况下，您可以在外连接子句或WHERE子句中连接的一个或两个表上指定其他限制。<br/>
指定一个JOIN子句，用于保留第二个（右）表中不匹配的行，并将其与第一个（左）表的形状连接。LEFT OUTER JOIN B等效于BRIGHT OUTER JOIN A，列的顺序不同。<br/>
###句法<br/>
<code>
TableExpression  RIGHT [ OUTER ] JOIN TableExpression <br/>
{<br/>
ON boolean-expression<br/>
}<br/>
</code><br/>
###描述<br/>
另见查询功能和限制。<br/>
ON子句中的表达式范围包括当前表和当前SELECT外部的查询块中的任何表。ON子句可以引用未连接的表，并且不必引用要连接的任何表（尽管通常是这样）。<br/>
注意： RowStore对受支持的加入查询类型施加限制。请参阅查询限制 - 连接，子查询，聚合。<br/>
例<br/>
--match all customers and their portfolio details, if any, having a specific <br/>
--transaction ID<br/>
select * from trade.portfolio F RIGHT OUTER JOIN trade.customers C on C.cid= F.cid where F.tid = ? <br/>
-- use the synonymous syntax, RIGHT JOIN, to achieve exactly<br/>
-- the same results as in the example above<br/>
select * from trade.portfolio F RIGHT JOIN trade.customers C on C.cid= F.cid where F.tid = ? <br/>
-- a TableExpression can be a joinOperation. Therefore<br/>
-- you can have multiple join operations in a FROM clause<br/>
-- List all the customers, there portfolio details, if any, and their sell <br/>
-- orders, if any<br/>
select * from trade.SELLORDERS SO RIGHT OUTER JOIN TRADE.PORTFOLIO F RIGHT OUTER JOIN TRADE.CUSTOMERS  c on f.cid = so.cid on c.cid= f.cid where f.tid = ?<br/>
###RIGHT OUTER JOIN：指定具有显式连接子句的两个表之间的连接，从第二个表中保留不匹配的行。<br/>
指定一个JOIN子句，用于保留第二个（右）表中不匹配的行，并将其与第一个（左）表的形状连接。LEFT OUTER JOIN B等效于BRIGHT OUTER JOIN A，列的顺序不同。<br/>
###句法
TableExpression  RIGHT [ OUTER ] JOIN TableExpression <br/>
{<br/>
ON boolean-expression<br/>
}<br/>
###描述
另见查询功能和限制。<br/>
ON子句中的表达式范围包括当前表和当前SELECT外部的查询块中的任何表。ON子句可以引用未连接的表，并且不必引用要连接的任何表（尽管通常是这样）。<br/>
注意： RowStore对受支持的加入查询类型施加限制。请参阅查询限制 - 连接，子查询，聚合。<br/>
例<br/>
<code>
--match all customers and their portfolio details, if any, having a specific 
--transaction ID<br/>
select * from trade.portfolio F RIGHT OUTER JOIN trade.customers C on C.cid= F.cid where F.tid = ? <br/>
-- use the synonymous syntax, RIGHT JOIN, to achieve exactly<br/>
-- the same results as in the example above<br/>
select * from trade.portfolio F RIGHT JOIN trade.customers C on C.cid= F.cid where F.tid = ? <br/>
-- a TableExpression can be a joinOperation. Therefore<br/>
-- you can have multiple join operations in a FROM clause<br/>
-- List all the customers, there portfolio details, if any, and their sell <br/>
-- orders, if any<br/>
select * from trade.SELLORDERS SO RIGHT OUTER JOIN TRADE.PORTFOLIO F RIGHT OUTER JOIN TRADE.CUSTOMERS  c on f.cid = so.cid on c.cid= f.cid where f.tid = ?<br/>

</code>





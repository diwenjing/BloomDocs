## Reader介绍

适配器是一个进程，将DFlow平台连接到特定类型的外部应用程序或文件（请参阅“概念指南”中的适配器）。选择哪个适配器决定了哪些属性必须指定数据源或目标。

### Readers

Readers通过在数据源和缓存中指定来使用。

一些 Reader 提供了一些解析器。例如，TCPReader可以使用BinaryParser，DSVParser或XMLParser。有关详细信息，请参阅解析器。


#### DataBaseReader

DatabaseReader可以从Microsoft SQL Server，MySQL、Postgres和Oracle等关系型数据库中的一个或多个表查询所需要的数据。

<table>
<tr><td>警告</td></tr>
<tr><td>必须按照安装数据库驱动程序中所述安装DBMS的JDBC驱动程序。</td></tr>
</table>


<table>
<tr><td>属性</td><td>类型</td><td>默认值</td><td>解释</td></tr>
<tr><td>ConnectionURL</td><td>java.lang.String</td><td></td><td>JDBC连接URL例如， jdbc:oracle:thin:@192.168.1.2:1521:orcl</td></tr>
<tr><td>ExcludedTables</td><td>	
java.lang.String</td><td></td><td>当指定通配符时，您可以在此处指定要从查询中排除的任何表。指定值Tables。例如，从SQL Server中读取时，要包括来自HR_EMPLOYEES和HR_DEPTS而不是HRMASTER的数据（因为您不能在Tables字符串中指定文字下划线）：<table><td>Tables = 'HR％'，
 ExcludedTables = 'HRMASTER'</td></table></td></tr>
<tr><td>FetchSize</td><td>java.lang.Integer</td><td>100</td><td>在单个JDBC方法执行中要从数据库中获取的最大记录数（请参阅JDBC驱动程序文档中的fetchsize讨论）</td></tr>
<tr><td>Password</td><td>Password</td><td></td><td>请参阅密码加密</td></tr>
<tr><td>Query</td><td>java.lang.String</td><td></td><td>SQL语句指定要返回的数据。您可以查询表，别名，同义词和视图。如果指定Tables，此属性将被忽略。
如果查询包含包含句点的同义词，则必须将其包含在转义引号中。例如：select * from
            \"synonym.name\"
</td></tr>
<tr><td>ReturnDateTimeAs</td><td>	
java.lang.String</td><td>JODA</td><td>设置为  String返回时间戳值作为字符串而不是Joda时间戳。此选项的主要目的是避免在将微秒时间转换为Joda毫秒时丢失精度。字符串的格式为yyyy-mm-dd hh：mm：ss.ffffff。</td></tr>
<tr><td>Tables</td><td>	
java.lang.String</td><td></td><td>要阅读的表格或视图。在MySQL和Oracle名称中区分大小写; 在SQL Server中不区分大小写。
对于Oracle和SQL Server，您可以将多个表和视图指定为以分号或以下通配符分隔的列表：
%：任何的字符
\_：MySQL或Oracle中的任何单个字符（请注意，使用OracleReader时不需要转义下划线）
_：SQL Server中的任何一个字符
例如，HR。％将读取HR模式中的所有表。
从Oracle读取时，_是一个字面的下划线，因此，例如，HR_％将包括HR_EMPLOYEES和HR_DEPTS，但不包括HRMASTER。（请注意，在使用OracleReader的情况下_是相反的情况：通配符  \_是字面的下划线）
从SQL Server读取时，无法指定下划线。
</td></tr>
<tr><td>Username</td><td>java.lang.String</td><td></td><td></td></tr>
</table>

输出类型为WAevent。

<table>
<tr><td>注意</td></tr>
<tr><td>要从Oracle CDB和PDB数据库中的表中读取，您必须创建两个DatabaseReader实例，每个实例一个。</td></tr>
</table>

如下示例为创建一个数据缓存，从MySQL中查询数据。

<code>

CREATE TYPE RackType(
  rack_id String KEY,
  datacenter_id String,
  rack_aisle java.lang.Integer,
  rack_row java.lang.Integer,
  slot_count java.lang.Integer
);
CREATE CACHE ConfiguredRacks USING DatabaseReader (
  ConnectionURL:'jdbc:mysql://10.1.10.149/datacenter',
  Username:'username',
  Password:'passwd',
  Query: "SELECT rack_id,datacenter_id,rack_aisle,rack_row,slot_count FROM RackList"
)
QUERY (keytomap:'rack_id') OF RackType;
The following example creates a cache of data retrieved from an Oracle table:
CREATE TYPE CustomerType (
  IPAddress  String KEY,
  RouterId  String,
  ConnectionMode  String,
  CustomerId  String,
  CustomerName  String
);
CREATE CACHE Customers USING DatabaseReader (
  Password: 'password',
  Username: 'admin',
  ConnectionURL: 'jdbc:oracle:thin:@node05.example.com:1521:test5',
  Query: 'SELECT ip_address, router_id, connection_mode, customer_id, customer_name FROM customers',
  FetchSize: 1000
)
QUERY (keytomap:'IPAddress') OF CustomerType;

</code>

DatabaseReader数据类型支持和转换：

<table>
<tr><td>JDBC列类型</td><td>CQL类型</td></tr>
<tr><td>Types.BIGINT</td><td>java.lang.Long</td></tr>
<tr><td>Types.BIT</td><td>java.lang.Boolean</td></tr>
<tr><td>Types.CHAR</td><td>java.lang.String</td></tr>
<tr><td>Types.DATE</td><td>org.joda.time.LocalDate</td></tr>
<tr><td>Types.DECIMAL</td><td>java.lang.String</td></tr>
<tr><td>Types.DOUBLE</td><td>java.lang.Double</td></tr>
<tr><td>Types.FLOAT</td><td>java.lang.Double</td></tr>
<tr><td>Types.INTEGER</td><td>java.lang.Integer</td></tr>
<tr><td>Types.NUMERIC</td><td>java.lang.String</td></tr>
<tr><td>Types.REAL</td><td>java.lang.Float</td></tr>
<tr><td>Types.SMALLINT</td><td>java.lang.Short</td></tr>
<tr><td>Types.TIMESTAMP</td><td>org.joda.time.DateTime</td></tr>
<tr><td>Types.TINYINT</td><td>java.lang.Short</td></tr>
<tr><td>Types.VARCHARCHAR</td><td>java.lang.String</td></tr>
<tr><td>其他类型</td><td>java.lang.String</td></tr>
</table>


#### FileReader

从磁盘中读取一个文件
 
<table>
<tr><td>属性</td><td>类型</td><td>默认值</td><td>解释</td></tr>
<tr><td>BlockSize</td><td>java.lang.Integer</td><td>64</td><td>每个读取操作的数据量（KB）</td></tr>
<tr><td>compressiontype</td><td>java.lang.String</td><td></td><td>当设置一个文件或gzip的文件时设置.</td></tr>
<tr><td>directory</td><td>java.lang.String</td><td></td><td>包含文件的目录的路径; 该路径可能相对于DFlow安装目录，例如Samples/PosApp/appdata或从根目录</td></tr>
<tr><td>positionbyeof</td><td>java.lang.Boolean

</td><td>True</td><td>如果设置为True，则读取从文件末尾开始，因此仅获取新数据。

如果设置为False，则读取将从文件的开头开始。

当FileReader与缓存一起使用时，此设置将被忽略，并且读取始终从文件的开头开始。

当您使用Source Preview创建一个FileReader时，将其设置为False。</td></tr>
<tr><td>rolloverstyle</td><td>java.lang.String</td><td>Default</td><td>设置为log4j如果读取使用RollingFileAppender创建的Log4J文件。</td></tr>
<tr><td>skipbom</td><td>java.lang.Boolean</td><td>True</td><td>如果设置为True，当通配符值指定多个文件时，DFlow将读取第一个文件中的字节顺序标记（BOM），并跳过所有其他文件中的BOM。如果设置为False，它将读取每个文件中的BOM。</td></tr>
<tr><td>WildCard</td><td>java.lang.String</td><td></td><td>文件的名称或匹配多个文件的通配符模式（例如* .xml）</td></tr>
</table>

FileReader可以选择解析器。有关详细信息，请参阅支持的读取器解析器组合。
输出类型是WAevent，除非使用JSONParser，在这种情况下，您必须定义适当的类型（请参阅JSONParser）。

来自PosApp示例应用程序的示例：

<code>
CREATE source CsvDataSource USING FileReader (
  directory:'Samples/PosApp/appData',
  wildcard:'posdata.csv',
  blocksize: 10240,
  positionByEOF:false
)
PARSE USING DSVParser (
  header:Yes,
  trimquote:false
) OUTPUT TO CsvStream;
</code>

有关其他示例，请参阅PosApp了解详细说明和MultiLogApp。

#### MultiFileReader

<table>
<tr><td>属性</td><td>类型</td><td>默认值</td><td>解释</td></tr>
<tr><td>BlockSize</td><td>java.lang.Integer</td><td>64</td><td>每个读取操作的数据量（KB）</td></tr>
<tr><td>
字符集</td><td>java.lang.String

</td><td>UTF-8</td><td></td></tr>

<tr><td>compressiontype</td><td>java.lang.String</td><td></td><td>当设置一个文件或gzip的文件时设置.</td></tr>
<tr><td>directory</td><td>java.lang.String</td><td></td><td>包含文件的目录的路径; 该路径可能相对于DFlow安装目录，例如Samples/PosApp/appdata或从根目录</td></tr>

<tr><td>
组模式</td><td>java.lang.String

</td><td></td><td>定义每组文件的滚动模式的正则表达式（请参阅使用正则表达式（正则表达式））
</td></tr>


<tr><td>positionbyeof</td><td>java.lang.Boolean</td><td>True</td><td>如果设置为True，则读取从文件末尾开始，因此仅获取新数据。
如果设置为False，则读取将从文件的开头开始。
</td></tr>
<tr><td>rolloverstyle</td><td>java.lang.String</td><td>Default</td><td>设置为log4j如果读取使用RollingFileAppender创建的Log4J文件。</td></tr>
<tr><td>skipbom</td><td>java.lang.Boolean</td><td>True</td><td>如果设置为True，当通配符值指定多个文件时，DFlow将读取第一个文件中的字节顺序标记（BOM），并跳过所有其他文件中的BOM。如果设置为False，它将读取每个文件中的BOM。</td></tr>

<tr><td>threadpoolsize
</td><td>
lava.lang.integer
</td><td>20</td><td>为获得最佳性能，请设置为一次读取的最大文件数。
</td></tr>

<tr><td>WildCard</td><td>java.lang.String</td><td></td><td>文件的名称或匹配多个文件的通配符模式（例如* .xml）</td></tr>

<tr><td>yieldAfter</td><td>lava.lang.integer

</td><td>20</td><td>线程将被切换到下一个读取过程的事件数
</td></tr>
</table>

输出类型是  WAevent，  除非使用JSONParser，在这种情况下，您必须定义适当的类型（请参阅JSONParser）。

该示例将识别log.proc1.0并log.proc1.1作为一个日志的一部分，log.proc2.0并将log.proc2.1其视为另一个日志的一部分，确保来自每个日志的所有事件将以正确的顺序读取。

<code>
CREATE SOURCE MFRtest USING MultiFileReader (
  directory:'Samples',
  WildCard:'log.proc*',
  grouppattern:'(?:(?:(?:<[^>]+>)*[^<.]*)*.){2}'
)
</code>

或者，您可以使用此语句来确保每个日志中的事件以正确的顺序读取：

<code>
CREATE SOURCE MFRtest USING MultiFileReader (
  directory:'Samples',
  WildCard:'log.proc*',
  grouppattern:'log\\.proc[0-9]{1,3}'
)
</code>


































































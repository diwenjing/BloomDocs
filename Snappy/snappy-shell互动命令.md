# snappy-shell互动命令 #
`snappy-shell`实现基于Apache Derby `ij`工具的交互式命令行工具。使用`snappy-shell`于针对RowStore集群中运行的脚本或交互式查询。

使用snappy -shell脚本启动交互式`snappy-shell`命令提示符，不提供任何其他选项：

	snappy-shell
system属性`snappydata.history`指定一个文件，用于存储在交互式`snappy-shell`会话期间执行的所有命令。例如：

	$ export JAVA_ARGS="-Dsnappydata.history=/Users/yozie/snappydata-history.sql"
	$ snappy
默认情况下，历史文件名为.snappydata.history，它存储在当前用户的主目录中。

`snappy-shell`接受几个命令来控制其使用JDBC。它将分号识别为`snappy-shell`SQL命令的结尾。它将SQL注释，字符串和分隔标识符中的分号视为这些结构的一部分，而不是命令的结尾。在分号snappy-shell或SQL语句的末尾需要分号。

所有`snappy-shell`命令，标识符和关键字都不区分大小写。

命令可以跨越多行，而不为线路末端使用任何特殊的转义字符。这意味着如果一个字符串跨越一行，则新行内容将显示在字符串中的值中。

`snappy-shell`将任何无法识别的命令视为传递到底层连接的SQL命令。这意味着`snappy-shell`命令中的任何句法错误都会交给SQL引擎，并且通常会导致SQL解析错误。

- **absolute** 将光标移动到 int指定的行，然后获取该行。

- **之后**， 将光标移动到最后一行之后，然后取出该行。

- **async** 在单独的线程中执行SQL语句。

- **autocommit** 打开或关闭连接的自动提交模式。

- **之前首先** 将光标移动到第一行之前，然后获取该行。

- **close** 关闭命名游标。

- **提交** 发出 java.sql.Connection.commit请求。

- **connect** 连接到由 ConnectionURLString指示的数据库。

- **连接客户端** 使用JDBC RowStore瘦客户端驱动程序，连接到由主机：port值指示的RowStore成员。

- **连接对等体** 使用JDBC RowStore对等客户机驱动程序，连接到具有指定引导和连接属性值的RowStore成员。

- **describe** 提供指定表或视图的描述。

- **断开** 断开与数据库的连接。

- **驱动程序** 发出一个 Class.forName请求来加载命名类。

- **elapsedtime** 显示语句执行期间的总时间。

- **execute** 使用动态参数执行准备好的语句或SQL命令。

- **exit** 完成应用`snappy-shell`程序并停止处理。

- **first** 将光标移动到 ResultSet的第一行，然后取出该行。

- **获得滚动不敏感游标** 创建一个可滚动的不敏感游标与名称标识符。

- **GetCurrentRowNumber** 返回指定滚动游标当前位置的行号。

- **help** 打印`snappy-shell`命令列表。

- **last** 将光标移动到 ResultSet中的最后一行，然后取出该行。

- **LocalizedDisplay** 指定是否以本地格式显示区域设置的区域设置敏感数据（如日期）`snappy-shell`。

- **MaximumDisplayWidth** 将列的最大显示宽度设置为指定的值。

- **next** 从使用get scroll insensitive cursor命令创建的命名游标中获取下一行。

- **prepare使用String的值** 创建 java.sql.PreparedStatement，该值可以`snappy-shell`通过给定的标识符访问。

- **以前** 将光标移动到当前行之前的行，然后取出该行。

- **协议** 将协议指定为String，用于建立连接并自动加载相应的驱动程序。

- **relative** 将光标移动到相对于当前行的 int行数，然后取出行。

- **remove** 从gfxd中删除以前准备好的语句。


- **回滚** 发出 java.sql.Connection.rollback请求。

- **run** 将字符串的值视为有效的文件名，并重定向`snappy-shell从`该文件读取的处理，直到其结束或执行exit命令。

- **set** connection 指定当多个连接打开时要连接的连接。

- **show** 显示有关活动连接和数据库对象的信息。

- **等待** 显示先前启动的异步命令的结果。

## 绝对 ##
将光标移动到int指定的行，然后取出该行。

### 句法 ###
	ABSOLUTE int Identifier

### 描述 ###
将光标移动到int指定的行，然后取出该行。光标必须使用**Get Scroll Insensitive Cursor**命令创建。它显示横幅和行的值。

### 例 ###
	snappy(PEERCLIENT)> select * from firsttable order by id;
	ID         |NAME
	------------------------
	10         |TEN
	20         |TWENTY
	30         |THIRTY
	40         |Forty
	50         |Fifty
	
	5 rows selected
	snappy(PEERCLIENT)> get scroll insensitive with nohold cursor scrollCursor as 'select * from firsttable order by id';
	snappy(PEERCLIENT)> absolute 3 scrollCursor;
	ID         |NAME
	------------------------
	30         |THIRTY

## 最后 ##
将光标移动到最后一行之后，然后取出该行。

### 句法 ###
	AFTER LAST Identifier

### 描述 ###
将光标移动到最后一行之后，然后取出该行。如果没有当前行，则返回消息：`"No current row."`

光标必须使用**Get Scroll Insensitive Cursor**命令创建。

### 例 ###
	snappy(PEERCLIENT)> get scroll insensitive with nohold cursor scrollCursor as 'select * from firsttable order by id';
	snappy(PEERCLIENT)> absolute 3 scrollCursor;
	ID         |NAME
	------------------------
	30         |THIRTY
	snappy(PEERCLIENT)> first scrollCursor;
	ID         |NAME
	------------------------
	10         |TEN
	snappy(PEERCLIENT)> next scrollCursor;
	ID         |NAME
	------------------------
	20         |TWENTY
	snappy(PEERCLIENT)> last scrollCursor;
	ID         |NAME
	------------------------
	50         |Fifty
	snappy(PEERCLIENT)> previous scrollCursor;
	ID         |NAME
	------------------------
	40         |Forty
	snappy(PEERCLIENT)> relative -1 scrollCursor;
	ID         |NAME
	------------------------
	30         |THIRTY
	snappy(PEERCLIENT)> after last scrollCursor;
	No current row

## 异步 ##
在单独的线程中执行SQL语句。

### 句法 ###
ASYNC Identifier String

### 描述 ###
ASYNC命令允许您在单独的线程中执行SQL语句。它与“ 等待命令”一起使用以获取结果。

您将提供SQL语句（它是任何有效的SQL语句）作为String。该标识符您必须提供异步SQL语句中使用等待命令是不区分大小写的`snappy-shell`标识符。标识符不能与当前连接上的async语句的任何其他标识符相同。您不能在此命令中引用之前准备并由`snappy-shell` Prepare命令命名的语句。

`snappy-shell`在当前或指定的连接中创建一个新的线程来发出SQL语句。语句完成后，单独的线程将关闭。

### 例子 ###
	snappy(PEERCLIENT)> async aInsert 'insert into firsttable values (40,''Forty'')';
	snappy(PEERCLIENT)> insert into firsttable values (50,'Fifty');
	1 row inserted/updated/deleted
	snappy(PEERCLIENT)> wait for aInsert;
	1 row inserted/updated/deleted
	-- the result of the asynchronous insert

## 自动提交 ##
打开或关闭连接的自动提交模式。

### 句法 ###
	AUTOCOMMIT { ON | OFF }

### 描述 ###
打开或关闭连接的自动提交模式。JDBC指定默认的自动提交模式为`ON`。某些类型的处理需要自动提交模式`OFF`。

如果自动提交模式在事务未完成时从关闭更改为打开，那么当当前事务提交时提交该工作，而不是在启动时自动提交。使用提交或回滚，在交易未完成之前开启自动提交，以便所有先前的工作在返回自动提交模式之前完成。

### 例 ###
	snappy(PEERCLIENT)> AUTOCOMMIT off;
	snappy(PEERCLIENT)> insert into airlines VALUES ('NA', 'New Airline', 0.20, 0.07, 0.6, 1.7, 20, 10, 5);
	1 row inserted/updated/deleted
	snappy(PEERCLIENT)> commit;

## 之前 ##
将光标移动到第一行之前，然后取出该行。

### 句法 ###
	BEFORE FIRST int Identifier

### 描述 ###
将光标移动到第一行之前，然后取出该行。如果没有当前行，则返回消息`No current row`。

光标必须使用**Get Scroll Insensitive Cursor**命令创建。

### 例 ###
	snappy(PEERCLIENT)> get scroll insensitive with nohold cursor scrollCursor as 'select * from firsttable order by id';
	snappy(PEERCLIENT)> absolute 3 scrollCursor;
	ID         |NAME
	------------------------
	30         |THIRTY
	snappy(PEERCLIENT)> before first scrollCursor;
	No current row

## 关 ##
关闭命名游标。

### 句法 ###
	CLOSE Identifier

### 描述 ###
关闭命名游标。以前使用`snappy-shell` **Get Scroll Insensitive Cursor**命令成功地创建了游标。

### 例 ###
	snappy(PEERCLIENT)> get scroll insensitive with nohold cursor scrollCursor as 'select * from firsttable order by id';
	snappy(PEERCLIENT)> before first scrollCursor;
	No current row
	snappy(PEERCLIENT)> GETCURRENTROWNUMBER scrollCursor;
	0
	snappy(PEERCLIENT)> relative 2 scrollCursor;
	ID         |NAME
	------------------------
	20         |TWENTY
	snappy(PEERCLIENT)> GETCURRENTROWNUMBER scrollCursor;
	2
	snappy(PEERCLIENT)> close scrollCursor;

## 承诺 ##
发出java.sql.Connection.commit请求。

### 句法 ###
	COMMIT

### 描述 ###
发出java.sql.Connection.commit请求。仅当自动提交关闭时才使用此命令。一个java.sql.Connection.commit要求提交当前活动的事务并启动一个新的事务。

### 例 ###
	snappy(PEERCLIENT)> AUTOCOMMIT off;
	snappy(PEERCLIENT)> insert into airlines VALUES ('NA', 'New Airline', 0.20, 0.07, 0.6, 1.7, 20, 10, 5);
	1 row inserted/updated/deleted
	snappy(PEERCLIENT)> commit;

## 连接 ##
连接到由ConnectionURLString指示的数据库。

### 句法 ###
CONNECT ConnectionURLString [ PROTOCOL Identifier ]
    [ AS Identifier ]

### 描述 ###
连接到由ConnectionURLString指示的数据库。您可以选择为连接指定名称。使用Set Connection命令在连接之间切换。如果不命名连接，系统将自动生成一个名称。

您还可以选择先前使用Protocol命令创建的命名协议。

> 注意：如果连接需要用户名和密码，请提供连接URL字符串中的那些，如示例所示。

如果连接成功，连接将成为当前连接，并`snappy-shell`显示新的提示，输入下一个命令。如果有多个打开的连接，连接的名称将显示在提示中。

所有其他命令将针对新的当前连接进行处理。

### 例 ###
	snappy> protocol 'jdbc:derby:';
	snappy> connect '//armenia:29303/myDB;user=a;password=a' as db5Connection; 
	snappy> show connections;
	DB5CONNECTION* -        jdbc:derby://armenia:29303/myDB
	* = current connection

## 连接客户端 ##
使用JDBC RowStore瘦客户端驱动程序，连接到由主机：port值指示的RowStore成员。

### 句法 ###
	CONNECT CLIENT 'host:port[;property=value]*' [ AS connectionName ]

### 描述 ###
使用JDBC RowStore瘦客户端驱动程序连接到由host：port值指示的RowStore成员。您可以为连接指定可选名称。使用设置连接在多个连接之间切换。如果不命名连接，系统将自动生成一个名称。

如果连接需要用户名和密码，请提供可选属性。

如果连接成功，连接将成为当前连接，并`snappy-shell`显示新的提示，输入下一个命令。如果有多个打开的连接，连接的名称将显示在提示中。

所有其他命令将针对新的当前连接进行处理。

### 例 ###
	snappy-shell version 1.4.0
	snappy> connect client 'localhost:1527' as clientConnection;
	snappy> show connections;
	CLIENTCONNECTION* -     jdbc:snappydata://localhost:1527/
	* = current connection

## 描述 ##
提供指定表或视图的描述。

### 句法 ###
	DESCRIBE { table-Name | view-Name }

### 描述 ###
提供指定表或视图的描述。有关当前模式中的表的列表，请使用“显示表​​”命令。有关当前模式中的视图列表，请使用“显示视图”命令。有关可用模式的列表，请使用“显示模式”命令。

如果表或视图位于特定模式中，请使用模式名称进行限定。如果表或视图名称区分大小写，请将其括在单引号中。您可以使用通配符“*”在单个显示中以单个模式显示所有表和视图中的所有列。参见下面的例子。

### 例 ###
	snappy(PEERCLIENT)> describe maps;
	COLUMN_NAME         |TYPE_NAME|DEC&|NUM&|COLUM&|COLUMN_DEF|CHAR_OCTE&|IS_NULL&
	------------------------------------------------------------------------------
	MAP_ID              |INTEGER  |0   |10  |10    |AUTOINCRE&|NULL      |NO
	MAP_NAME            |VARCHAR  |NULL|NULL|24    |NULL      |48        |NO
	REGION              |VARCHAR  |NULL|NULL|26    |NULL      |52        |YES
	AREA                |DECIMAL  |4   |10  |8     |NULL      |NULL      |NO
	PHOTO_FORMAT        |VARCHAR  |NULL|NULL|26    |NULL      |52        |NO
	PICTURE             |BLOB     |NULL|NULL|102400|NULL      |NULL      |YES
	
	6 rows selected
	snappy(PEERCLIENT)>

## 断开 ##
断开与数据库的连接。

### 句法 ###
	DISCONNECT [ ALL | CURRENT | ConnectionIdentifier ]

### 描述 ###
断开与数据库的连接。特别发出`java.sql.Connection.close`命令行上指示的连接的请求。在请求时必须有一个当前的连接。

如果指定了ALL，则所有已知连接都将关闭，并且不存在当前连接。

断开CURRENT与断开相同，不指示连接; 默认连接已关闭。

如果使用标识符指定连接名称，则该命令将断开命名的连接。该名称必须是Connect命令提供的当前会话中连接的名称。

如果没有使用AS子句的Connect命令，则可以提供为连接生成的系统名称。如果当前连接是命名的连接，则当命令完成时，将不存在当前连接，并且您必须发出Set Connection或Connect命令。

### 例 ###
	snappy(PEERCLIENT)> disconnect peerclient;
	snappy>

## 驱动 ##
发出一个Class.forName请求来加载命名类。

### 句法 ###
	DRIVER DriverNameString

### 描述 ###
获取DriverNameString的值，并发出一个Class.forName请求来加载命名类。该类应该是一个JDBC驱动程序，它向java.sql.DriverManager注册。

> 注意：在启动`snappy-shell`之前，您必须设置CLASSPATH环境变量。

在RowStore中，您可以使用此命令连接到用于加载或写入缓存数据的外部数据库。

如果驱动程序命令成功，`snappy-shell`则会出现下一个命令的新提示。

### 例 ###
此示例加载Apache Derby客户端驱动程序。这使您能够连接到数据库，假设Derby网络服务器在示例主机和端口上可用。

	snappy> driver 'org.apache.derby.jdbc.ClientDriver';
	snappy> connect 'jdbc:derby://armenia:29303/myDB;create=true';
	snappy>

## elapsedtime ##
显示执行语句期间的总时间。

### 句法 ###
	ELAPSEDTIME { ON | OFF }

### 描述 ###
何时elapsedtime打开，snappy-shell显示语句执行期间的总时间。默认值为OFF。

### 例 ###
	snappy(PEERCLIENT)> elapsedtime on;
	snappy(PEERCLIENT)> select * from airlines;
	A&|AIRLINE_FULL            |BASIC_RATE            |DISTANCE_DISCOUNT     |BUSINESS_LEVEL_FACTOR |FIRSTCLASS_LEVEL_FACT&|ECONOMY_SE&|BUSINESS_S&|FIRSTCLASS&
	-----------------------------------------------------------------------------------------------------------------------------------------------------------
	NA|New Airline             |0.2                   |0.07                  |0.6                   |1.7                   |20         |10         |5
	US|Union Standard Airlines |0.19                  |0.05                  |0.4                   |1.6                   |20         |10         |5
	AA|Amazonian Airways       |0.18                  |0.03                  |0.5                   |1.5                   |20         |10         |5
	
	3 rows selected
	ELAPSED TIME = 2 milliseconds

## 执行 ##
执行一个准备语句或一个带有动态参数的SQL命令。

### 句法 ###
	EXECUTE { SQLString | PreparedStatementIdentifier }
	[ USING { String | Identifier } ]

### 描述 ###
有几个用途：

- 执行SQLString输入的SQL命令，使用执行字符串样式。字符串传递到连接，无需进一步检查或处理`snappy-shell`。通常，您直接执行SQL命令，而不是执行命令。

- 执行由PreparedStatementIdentifier标识的命名命令。必须先`snappy-shell` 准备好准备命令。

- 要在该命令包含动态参数时执行命令风格，请指定USING子句中的值。在这种风格中，SQLString或之前准备的PreparedStatementIdentifier使用String或Identifier提供的值来执行。USING子句中的标识符必须具有结果集作为其结果。结果集的每一行都应用于要执行的命令的输入参数，因此USING结果集中的列数必须与Execute语句中的输入参数数量相匹配。执行Execute语句的结果将按照它们的形式进行显示。如果使用的结果集不包含行，则Execute的语句不会被执行。<br/>
当自动提交模式打开时，使用的结果集在第一次执行Execute语句时关闭。要确保执行Execute命令的多行执行，请使用Autocommit命令关闭自动提交。
### 例 ###
	snappy(PEERCLIENT)> create table firsttable (id int primary key,
	>     name varchar(12));
	0 rows inserted/updated/deleted
	snappy(PEERCLIENT)> insert into firsttable values
	>     (10,'TEN'),(20,'TWENTY'),(30,'THIRTY');
	3 rows inserted/updated/deleted
	snappy(PEERCLIENT)> select * from firsttable;
	ID         |NAME
	------------------------
	20         |TWENTY
	30         |THIRTY
	10         |TEN
	
	3 rows selected
	snappy(PEERCLIENT)> create table newtable (newid int primary key,
	>     newname varchar(12));
	0 rows inserted/updated/deleted
	snappy(PEERCLIENT)> prepare listFirstTable as 'select * from firsttable';
	snappy(PEERCLIENT)> autocommit off;
	snappy(PEERCLIENT)> execute 'insert into newtable(newid, newname)
	>     values(?,?)' using listFirstTable;
	1 row inserted/updated/deleted
	1 row inserted/updated/deleted
	1 row inserted/updated/deleted
	snappy(PEERCLIENT)> commit;
	snappy(PEERCLIENT)> select * from newtable;
	NEWID      |NEWNAME
	------------------------
	20         |TWENTY
	30         |THIRTY
	10         |TEN
	
	3 rows selected

## 出口 ##
完成应用snappy-shell程序并停止处理。

### 句法 ###
	EXIT

### 描述 ###
使应用`snappy-shell`程序完成并处理停止。在从Run命令或命令行开始的文件中发出此命令将导致最外面的输入循环停止。

`snappy-shell` 当输入退出命令时，或者如果在命令文件的末尾到达Java调用行时给出命令文件，则退出。

### 例 ###
	snappy(PEERCLIENT)> disconnect peerclient;
	snappy> exit;

## 第一 ##
将光标移动到ResultSet中的第一行，然后取出该行。

### 句法 ###
	FIRST Identifier

### 描述 ###
将光标移动到ResultSet中的第一行，然后取出该行。您首先使用Get Scroll Insensitive Cursor命令创建光标。它显示横幅和行的值。

### 例 ###
	snappy(PEERCLIENT)> get scroll insensitive with nohold cursor scrollCursor as 
	'select * from firsttable order by id';
	snappy(PEERCLIENT)> absolute 3 scrollCursor;
	ID         |NAME
	------------------------
	30         |THIRTY
	snappy(PEERCLIENT)> first scrollCursor;
	ID         |NAME
	------------------------
	10         |TEN

## 获取滚动不敏感的游标 ##
创建一个可滚动的不敏感游标与名称标识符。

### 句法 ###
	GET SCROLL INSENSITIVE WITH NOHOLD
	   CURSOR Identifier AS
	   String

> 注意： RowStore不支持可保存的结果集。


### 描述 ###
创建一个可滚动的不敏感游标与名称标识符。该命令发出一个createStatement（ResultSet.TYPE_SCROLL_INSENSITIVE，ResultSet.CONCUR_READ_ONLY）调用，然后对String的值执行java.sql.StatementExecuteQuery请求的语句。

如果String是不生成结果集的语句，则底层数据库的行为确定是否发出空结果集或错误。如果执行该语句时发生错误，则不会创建任何游标。

snappy-shell使用java.sql.Statement.setCursorName请求设置游标名称。关于重复游标名称的行为由底层数据库控制。RowStore不允许具有相同名称的多个打开的游标。

滚动不敏感的结果集不能滚动。这意味着光标可以在一个方向上移动。当结果集打开时，对数据库表的任何更改都不会反映出来。

不同于滚动不敏感，滚动感知结果集可以双向移动光标。当结果集打开时，它还启用对数据库表的更改。这是gfxd可滚动光标的默认值。

创建可滚动光标后，使用以下命令来处理结果集：

- **绝对**

- **最后**

- **之前**

- **关**

- **第一**

- **持续**

- **下一个**

- **以前**

- **相对的**


### 例 ###
	snappy(PEERCLIENT)> get with nohold cursor scrollCursor as 'select * 
	from firsttable';
	
	--another connection performs the following operations.
	snappy> select * from firsttable;
	ID         |NAME
	------------------------
	40         |Forty
	20         |TWENTY
	30         |THIRTY
	10         |TEN
	50         |Fifty
	
	5 rows selected
	snappy> update firsttable set name='FORTY' where id=40;
	1 row inserted/updated/deleted
	
	--the sensitive cursor reflects the update
	snappy(PEERCLIENT)> next scrollCursor;
	ID         |NAME
	------------------------
	40         |FORTY
	
	--scroll insensitive cursor
	snappy(PEERCLIENT)> get scroll insensitive with nohold cursor scrollCursor 
	as 'select * from firsttable';
	
	--another connection performs the following operation.
	snappy> update firsttable set name='Forty' where id=40;
	1 row inserted/updated/deleted
	
	--the insensitive cursor does not reflect the update
	snappy(PEERCLIENT)> next scrollCursor;
	ID         |NAME
	------------------------
	40         |FORTY

## GetCurrentRowNumber ##
返回指定滚动光标当前位置的行号。

### 句法 ###
	GETCURRENTROWNUMBER Identifier

### 描述 ###
使用**get滚动不敏感游标**创建光**标后**，返回指定滚动光标当前位置的行号。如果光标不在行上，则返回零（0）。

### 例 ###
	snappy(PEERCLIENT)> get scroll insensitive with nohold cursor scrollCursor as 'select * from firsttable order by id';
	snappy(PEERCLIENT)> absolute 3 scrollCursor;
	ID         |NAME
	------------------------
	30         |THIRTY
	snappy(PEERCLIENT)> before first scrollCursor;
	No current row
	snappy(PEERCLIENT)> GETCURRENTROWNUMBER scrollCursor;
	0
	snappy(PEERCLIENT)> relative 2 scrollCursor;
	ID         |NAME
	------------------------
	20         |TWENTY
	snappy(PEERCLIENT)> GETCURRENTROWNUMBER scrollCursor;
	2

## 帮帮我 ##
打印snappy-shell命令列表。

### 句法 ###
	HELP

### 描述 ###
打印出简要的`snappy-shell`命令列表。

## 持续 ##
将光标移动到ResultSet中的最后一行，然后取出该行。

### 句法 ###
	LAST Identifier

### 描述 ###
使用Get Scroll Insensitive Cursor命令创建光标后，将光标移动到ResultSet中的最后一行，然后取出该行。它显示横幅和行的值。

### 例 ###
	snappy(PEERCLIENT)> get scroll insensitive with nohold cursor scrollCursor as 'select * from firsttable order by id';
	snappy(PEERCLIENT)> absolute 3 scrollCursor;
	ID         |NAME
	------------------------
	30         |THIRTY
	snappy(PEERCLIENT)> first scrollCursor;
	ID         |NAME
	------------------------
	10         |TEN
	snappy(PEERCLIENT)> next scrollCursor;
	ID         |NAME
	------------------------
	20         |TWENTY
	snappy(PEERCLIENT)> last scrollCursor;
	ID         |NAME
	------------------------
	50         |Fifty

## LocalizedDisplay ##
指定是否以本地格式显示区域设置的区域设置敏感数据（如日期）`snappy-shell`。

### 句法 ###
LOCALIZEDDISPLAY { on | off }

### 描述 ###
指定是否以本地格式显示区域设置的区域设置敏感数据（如日期）snappy-shell。该`snappy-shell`区域是一样的Java系统语言环境。

> 注意：由于平台限制，使用J2ME / CDC / Foundation Profile时，NUMERIC和DECIMAL值不会本地化。

### 例 ###
以下演示了英语语言环境中的LocalizedDisplay：

	snappy(PEERCLIENT)> VALUES CURRENT_DATE;
	1
	----------
	2011-05-16
	
	1 row selected
	snappy(PEERCLIENT)> localizeddisplay on;
	snappy(PEERCLIENT)> VALUES CURRENT_DATE;
	1
	------------------
	May 16, 2011
	
	1 row selected

## MaximumDisplayWidth ##
将列的最大显示宽度设置为指定的值。

### 句法 ###
	MAXIMUMDISPLAYWIDTH integer_value

### 描述 ###
将列的最大显示宽度设置为指定的值。您通常使用此命令增加默认值以显示大块文本。

### 例 ###
	snappy(PEERCLIENT)> maximumdisplaywidth 4;
	snappy(PEERCLIENT)> VALUES 'NOW IS THE TIME!';
	1
	----
	NOW&
	
	1 row selected
	snappy(PEERCLIENT)> maximumdisplaywidth 30;
	snappy(PEERCLIENT)> VALUES 'NOW IS THE TIME!';
	1
	----------------
	NOW IS THE TIME!
	
	1 row selected

## 下一个 ##
从使用get scroll insensitive cursor命令创建的命名游标中获取下一行。

### 句法 ###
	NEXT Identifier

### 描述 ###
从使用Get Scroll Insensitive Cursor创建的命名光标中获取下一行。它显示横幅和行的值。

### 例 ###
	snappy(PEERCLIENT)> get scroll insensitive with nohold cursor 
	scrollCursor as 'select * from firsttable order by id';
	snappy(PEERCLIENT)> absolute 3 scrollCursor;
	ID         |NAME
	------------------------
	30         |THIRTY
	snappy(PEERCLIENT)> first scrollCursor;
	ID         |NAME
	------------------------
	10         |TEN
	snappy(PEERCLIENT)> next scrollCursor;
	ID         |NAME
	------------------------
	20         |TWENTY

## 准备 ##
使用String的值创建java.sql.PreparedStatement，该值可以`snappy-shell`通过给定的标识符访问。

### 句法 ###
	PREPARE Identifier AS String

### 描述 ###
使用String的值创建java.sql.PreparedStatement，该值可以`snappy-shell`通过给定的标识符访问。如果已经存在具有该名称的准备好的语句，snappy-shell则返回错误，并保留先前准备的语句。使用Remove命令先删除以前的语句。如果发生错误，则不会创建准备好的语句。

可以使用此命令准备底层连接准备语句中允许的任何SQL语句。

### 例 ###
	snappy(PEERCLIENT)> prepare listAirlines as 'SELECT * FROM airlines';
	snappy(PEERCLIENT)> execute listAirlines;
	A&|AIRLINE_FULL            |BASIC_RATE            |DISTANCE_DISCOUNT     |BUSINESS_LEVEL_FACTOR |FIRSTCLASS_LEVEL_FACT&|ECONOMY_SE&|BUSINESS_S&|FIRSTCLASS&
	-----------------------------------------------------------------------------------------------------------------------------------------------------------
	NA|New Airline             |0.2                   |0.07                  |0.6                   |1.7                   |20         |10         |5
	US|Union Standard Airlines |0.19                  |0.05                  |0.4                   |1.6                   |20         |10         |5
	AA|Amazonian Airways       |0.18                  |0.03                  |0.5                   |1.5                   |20         |10         |5
	
	3 rows selected

## 以前 ##
将光标移动到当前行之前的行，然后取出该行。

### 句法 ###
	PREVIOUS Identifier

### 描述 ###
在使用Get Scroll Insensitive Cursor命令创建光标后，将光标移动到当前行的前一行，然后取出行。它显示横幅和行的值。

### 例 ###
	snappy(PEERCLIENT)> get scroll insensitive with nohold cursor scrollCursor as 'select * from firsttable order by id';
	snappy(PEERCLIENT)> absolute 3 scrollCursor;
	ID         |NAME
	------------------------
	30         |THIRTY
	snappy(PEERCLIENT)> first scrollCursor;
	ID         |NAME
	------------------------
	10         |TEN
	snappy(PEERCLIENT)> next scrollCursor;
	ID         |NAME
	------------------------
	20         |TWENTY
	snappy(PEERCLIENT)> last scrollCursor;
	ID         |NAME
	------------------------
	50         |Fifty
	snappy(PEERCLIENT)> previous scrollCursor;
	ID         |NAME
	------------------------
	40         |Forty

## 协议 ##
将协议指定为String，用于建立连接并自动加载相应的驱动程序。

### 句法 ###
	PROTOCOL String [ AS Identifier ]

### 描述 ###
将协议指定为String，用于建立连接并自动加载相应的驱动程序。协议是适用于您的环境的数据库连接URL语法的一部分，包括连接到外部数据库的JDBC协议。

提供协议允许您使用缩短的数据库连接URL进行连接。您只能提供数据库名称（如果需要，还可以提供subsubprotocol名称），而不是完整的协议。此外，由于驱动程序自动加载，您不需要在启动时使用**驱动程序**命令或指定驱动程序。

如果您命名协议，可以参考Connect命令中的协议名称。

### 例 ###
	snappy> protocol 'jdbc:derby:';
	snappy> connect '//armenia:29303/myDB;create=true';
	snappy> connect '//armenia:29303/myDB2;create=true';
	snappy(CONNECTION1)>


## 相对的 ##
将光标移动到相对于当前行的int行数的行，然后取出该行。

### 句法 ###
	RELATIVE int Identifier

### 描述 ###
使用Get Scroll Insensitive Cursor命令创建光标后，将光标移动到相对于当前行的int行数的行，然后取出该行。它显示横幅和行的值。

### 例 ###
	snappy(PEERCLIENT)> get scroll insensitive with nohold cursor scrollCursor as 'select * from firsttable order by id';
	snappy(PEERCLIENT)> absolute 3 scrollCursor;
	ID         |NAME
	------------------------
	30         |THIRTY
	snappy(PEERCLIENT)> first scrollCursor;
	ID         |NAME
	------------------------
	10         |TEN
	snappy(PEERCLIENT)> next scrollCursor;
	ID         |NAME
	------------------------
	20         |TWENTY
	snappy(PEERCLIENT)> last scrollCursor;
	ID         |NAME
	------------------------
	50         |Fifty
	snappy(PEERCLIENT)> previous scrollCursor;
	ID         |NAME
	------------------------
	40         |Forty
	snappy(PEERCLIENT)> relative -1 scrollCursor;
	ID         |NAME
	------------------------
	30         |THIRTY

## 去掉 ##
从gfxd中删除之前准备好的语句。

### 句法 ###
	REMOVE Identifier

### 描述 ###
从gfxd中删除之前准备好的语句。标识符是声明准备的名称。该语句关闭以释放其数据库资源。

### 例 ###
	snappy(PEERCLIENT)> prepare listFirstTable as 'select * from firsttable'; snappy(PEERCLIENT)> execute listFirstTable;
	ID         |NAME
	------------------------
	20         |TWENTY
	30         |THIRTY
	10         |TEN
	
	3 rows selected
	snappy(PEERCLIENT)> remove listFirstTable;
	snappy(PEERCLIENT)> execute listFirstTable;
	GFXD ERROR: Unable to establish prepared statement LISTFIRSTTABLE

## 回滚 ##
发出java.sql.Connection.rollback请求。

### 句法 ###
	ROLLBACK 

### 描述 ###
发出java.sql.Connection.rollback请求。仅在自动提交关闭时使用。一个java.sql.Connection.rollback请求撤消当前活动的事务并启动一个新的事务。

### 例 ###
	snappy(PEERCLIENT)> set isolation read committed;
	0 rows inserted/updated/deleted
	snappy(PEERCLIENT)> values current isolation;
	1
	----
	CS
	
	1 row selected
	snappy(PEERCLIENT)> insert into airlines VALUES ('AN', 'Another New Airline', 0.20, 0.07, 0.6, 1.7, 20, 10, 5);
	1 row inserted/updated/deleted
	snappy(PEERCLIENT)> AUTOCOMMIT off;
	snappy(PEERCLIENT)> select * from airlines;
	A&|AIRLINE_FULL            |BASIC_RATE            |DISTANCE_DISCOUNT     |BUSINESS_LEVEL_FACTOR |FIRSTCLASS_LEVEL_FACT&|ECONOMY_SE&|BUSINESS_S&|FIRSTCLASS&
	-----------------------------------------------------------------------------------------------------------------------------------------------------------
	NA|New Airline             |0.2                   |0.07                  |0.6                   |1.7                   |20         |10         |5
	US|Union Standard Airlines |0.19                  |0.05                  |0.4                   |1.6                   |20         |10         |5
	AA|Amazonian Airways       |0.18                  |0.03                  |0.5                   |1.5                   |20         |10         |5
	AN|Another New Airline     |0.2                   |0.07                  |0.6                   |1.7                   |20         |10         |5
	
	4 rows selected
	snappy(PEERCLIENT)> rollback;
	snappy(PEERCLIENT)> select * from airlines;
	A&|AIRLINE_FULL            |BASIC_RATE            |DISTANCE_DISCOUNT     |BUSINESS_LEVEL_FACTOR |FIRSTCLASS_LEVEL_FACT&|ECONOMY_SE&|BUSINESS_S&|FIRSTCLASS&
	-----------------------------------------------------------------------------------------------------------------------------------------------------------
	NA|New Airline             |0.2                   |0.07                  |0.6                   |1.7                   |20         |10         |5
	US|Union Standard Airlines |0.19                  |0.05                  |0.4                   |1.6                   |20         |10         |5
	AA|Amazonian Airways       |0.18                  |0.03                  |0.5                   |1.5                   |20         |10         |5
	
	3 rows selected

## 运行 ##
将字符串的值视为有效的文件名，并重定向`snappy-shell`从该文件读取的处理，直到其结束或执行退出命令。

### 句法 ###
	RUN String

### 描述 ###
将字符串的值视为有效的文件名，并重定向snappy-shell从该文件读取的处理，直到其结束或执行退出命令。如果在没有`snappy-shell`退出的情况下到达文件的末尾，一旦达到文件的末尾，就从上一个输入源继续阅读。文件可以包含运行命令。

`snappy-shell` 在执行文件时打印文件中的语句。

`snappy-shell`当处理恢复时，文件对环境的任何更改都可以在环境中显示。

### 例 ###
	snappy(PEERCLIENT)> run 'ToursDB_schema.sql';

## 设置连接 ##
指定当多个连接打开时要连接到哪个连接。

### 句法 ###
	SET CONNECTION Identifier

### 描述 ###
当您打开多个连接时，允许您指定要连接的连接。使用“ 显示连接”命令显示打开的连接。

如果没有这样的连接，则会出现错误，并且当前连接不变。

### 例 ###
	snappy(PEERCLIENT)> set connection clientConnection;
	snappy(CLIENTCONNECTION)> show connections;
	CLIENTCONNECTION* -     jdbc:snappydata://localhost:1527/
	PEERCLIENT -    jdbc:snappydata:
	* = current connection
	snappy(CLIENTCONNECTION)>

## 显示 ##
显示有关活动连接和数据库对象的信息。

### 句法 ###
	SHOW
	{
	   CONNECTIONS |
	   IMPORTEDKEYS [ IN schemaName | FROM table-Name ] |
	   INDEXES [ IN schemaName | FROM table-Name ] |
	   PROCEDURES [ IN schemaName ] |
	   SCHEMAS |
	   SYNONYMS [ IN schemaName ] |
	   TABLES [ IN schemaName ] |
	   VIEWS [ IN schemaName ] |
	
	}

### 描述 ###
显示有关活动连接和数据库对象的信息。

### 显示连接 ###

如果没有连接，则SHOW CONNECTIONS命令返回“无连接可用”。

否则，该命令将显示连接名称和用于连接到它们的URL的列表。当前活动连接的名称后面加上*。

### 例 ###

	snappy(CLIENTCONNECTION)> show connections;
	CLIENTCONNECTION* -     jdbc:snappydata://localhost:1527/
	PEERCLIENT -    jdbc:snappydata:
	* = current connection
	snappy(CLIENTCONNECTION)>
### 显示IMPORTEDKEYS ###

SHOW IMPORTEDKEYS显示指定模式或表中的所有外键。如果省略了schema和table子句，gfxd将显示当前模式中所有表的所有外键。

### 例 ###

	snappy> show importedkeys;
	KTABLE_SCHEM |PKTABLE_NAME|PKCOLUMN_NAME   |PK_NAME     |FKTABLE_SCHEM|FKTABLE_NAME      |FKCOLUMN_NAME   |FK_NAME     |KEY_SEQ
	-------------------------------------------------------------------------------------------------------------------------------
	APP          |COUNTRIES   |COUNTRY_ISO_CODE|COUNTRIES_PK|APP          |CITIES            |COUNTRY_ISO_CODE|COUNTRIES_FK|1      
	APP          |FLIGHTS     |FLIGHT_ID       |FLIGHTS_PK  |APP          |FLIGHTAVAILABILITY|FLIGHT_ID       |FLIGHTS_FK2 |1      
	APP          |FLIGHTS     |SEGMENT_NUMBER  |FLIGHTS_PK  |APP          |FLIGHTAVAILABILITY|SEGMENT_NUMBER  |FLIGHTS_FK2 |2      
	
	3 rows selected
	snappy> show importedkeys from flightavailability;
	PKTABLE_NAME|PKCOLUMN_NAME |PK_NAME   |FKTABLE_SCHEM|FKTABLE_NAME      |FKCOLUMN_NAME |FK_NAME    |KEY_SEQ
	----------------------------------------------------------------------------------------------------------
	FLIGHTS     |FLIGHT_ID     |FLIGHTS_PK|APP          |FLIGHTAVAILABILITY|FLIGHT_ID     |FLIGHTS_FK2|1      
	FLIGHTS     |SEGMENT_NUMBER|FLIGHTS_PK|APP          |FLIGHTAVAILABILITY|SEGMENT_NUMBER|FLIGHTS_FK2|2      
	
	2 rows selected
### 显示索引 ###

SHOW INDEXES显示数据库中的所有索引。

如果IN schemaName指定，则只显示指定架构中的索引。

如果FROM table-Name指定，则只显示指定表上的索引。

### 例 ###

	snappy(PEERCLIENT)> show indexes in app;
	TABLE_NAME          |COLUMN_NAME         |NON_U&|TYPE|ASC&|CARDINA&|PAGES
	----------------------------------------------------------------------------
	AIRLINES            |AIRLINE             |false |3   |A   |NULL    |NULL
	CITIES              |CITY_ID             |false |3   |A   |NULL    |NULL
	CITIES              |COUNTRY_ISO_CODE    |true  |3   |A   |NULL    |NULL
	COUNTRIES           |COUNTRY_ISO_CODE    |false |3   |A   |NULL    |NULL
	COUNTRIES           |COUNTRY             |false |3   |A   |NULL    |NULL
	FLIGHTAVAILABILITY  |FLIGHT_ID           |false |3   |A   |NULL    |NULL
	FLIGHTAVAILABILITY  |SEGMENT_NUMBER      |false |3   |A   |NULL    |NULL
	FLIGHTAVAILABILITY  |FLIGHT_DATE         |false |3   |A   |NULL    |NULL
	FLIGHTAVAILABILITY  |FLIGHT_ID           |true  |3   |A   |NULL    |NULL
	FLIGHTAVAILABILITY  |SEGMENT_NUMBER      |true  |3   |A   |NULL    |NULL
	FLIGHTS             |FLIGHT_ID           |false |3   |A   |NULL    |NULL
	FLIGHTS             |SEGMENT_NUMBER      |false |3   |A   |NULL    |NULL
	FLIGHTS             |DEST_AIRPORT        |true  |3   |A   |NULL    |NULL
	FLIGHTS             |ORIG_AIRPORT        |true  |3   |A   |NULL    |NULL
	MAPS                |MAP_ID              |false |3   |A   |NULL    |NULL
	MAPS                |MAP_NAME            |false |3   |A   |NULL    |NULL
	
	16 rows selected
	snappy(PEERCLIENT)> show indexes from flights;
	TABLE_NAME          |COLUMN_NAME         |NON_U&|TYPE|ASC&|CARDINA&|PAGES
	----------------------------------------------------------------------------
	FLIGHTS             |FLIGHT_ID           |false |3   |A   |NULL    |NULL
	FLIGHTS             |SEGMENT_NUMBER      |false |3   |A   |NULL    |NULL
	FLIGHTS             |DEST_AIRPORT        |true  |3   |A   |NULL    |NULL
	FLIGHTS             |ORIG_AIRPORT        |true  |3   |A   |NULL    |NULL
	
	4 rows selected
	snappy(PEERCLIENT)>
### 展示程序 ###

SHOW PROCEDURES显示使用CREATE PROCEDURE语句创建的数据库中的所有过程以及系统过程。

如果IN schemaName指定，则只显示指定模式中的过程。

## 例 ##

	snappy(PEERCLIENT)> show procedures in syscs_util;
	PROCEDURE_SCHEM     |PROCEDURE_NAME                |REMARKS
	------------------------------------------------------------------------
	SYSCS_UTIL          |BACKUP_DATABASE               |com.pivotal.snappydata.&
	SYSCS_UTIL          |BACKUP_DATABASE_AND_ENABLE_LO&|com.pivotal.snappydata.&
	SYSCS_UTIL          |BACKUP_DATABASE_AND_ENABLE_LO&|com.pivotal.snappydata.&
	SYSCS_UTIL          |BACKUP_DATABASE_NOWAIT        |com.pivotal.snappydata.&
	SYSCS_UTIL          |BULK_INSERT                   |com.pivotal.snappydata.&
	SYSCS_UTIL          |CHECKPOINT_DATABASE           |com.pivotal.snappydata.&
	SYSCS_UTIL          |COMPRESS_TABLE                |com.pivotal.snappydata.&
	SYSCS_UTIL          |DISABLE_LOG_ARCHIVE_MODE      |com.pivotal.snappydata.&
	SYSCS_UTIL          |EMPTY_STATEMENT_CACHE         |com.pivotal.snappydata.&
	SYSCS_UTIL          |EXPORT_QUERY                  |com.pivotal.snappydata.&
	SYSCS_UTIL          |EXPORT_QUERY_LOBS_TO_EXTFILE  |com.pivotal.snappydata.&
	SYSCS_UTIL          |EXPORT_TABLE                  |com.pivotal.snappydata.&
	SYSCS_UTIL          |EXPORT_TABLE_LOBS_TO_EXTFILE  |com.pivotal.snappydata.&
	SYSCS_UTIL          |FREEZE_DATABASE               |com.pivotal.snappydata.&
	SYSCS_UTIL          |IMPORT_DATA                   |com.pivotal.snappydata.&
	SYSCS_UTIL          |IMPORT_DATA_LOBS_FROM_EXTFILE |com.pivotal.snappydata.&
	SYSCS_UTIL          |IMPORT_TABLE                  |com.pivotal.snappydata.&
	SYSCS_UTIL          |IMPORT_TABLE_LOBS_FROM_EXTFILE|com.pivotal.snappydata.&
	SYSCS_UTIL          |INPLACE_COMPRESS_TABLE        |com.pivotal.snappydata.&
	SYSCS_UTIL          |RELOAD_SECURITY_POLICY        |com.pivotal.snappydata.&
	SYSCS_UTIL          |SET_DATABASE_PROPERTY         |com.pivotal.snappydata.&
	SYSCS_UTIL          |SET_EXPLAIN_CONNECTION        |com.pivotal.snappydata.&
	SYSCS_UTIL          |SET_STATISTICS_TIMING         |com.pivotal.snappydata.&
	SYSCS_UTIL          |SET_USER_ACCESS               |com.pivotal.snappydata.&
	SYSCS_UTIL          |UNFREEZE_DATABASE             |com.pivotal.snappydata.&
	
	27 rows selected
	snappy(PEERCLIENT)>         
### 显示方案 ###

SHOW SCHEMAS显示当前连接中的所有模式。

### 例 ###

	snappy(PEERCLIENT)> create schema sample;
	0 rows inserted/updated/deleted
	snappy(PEERCLIENT)> show schemas;
	TABLE_SCHEM
	------------------------------
	APP
	NULLID
	SAMPLE
	SQLJ
	SYS
	SYSCAT
	SYSCS_DIAG
	SYSCS_UTIL
	SYSFUN
	SYSIBM
	SYSPROC
	SYSSTAT
	
	12 rows selected
### 显示SYNONYMS ###

SHOW SYNONYMS显示使用CREATE SYNONYMS语句创建的数据库中的所有同义词。

如果IN schemaName指定，则只显示指定模式中的同义词。

### 例 ###

	snappy(PEERCLIENT)> SHOW SYNONYMS;
	TABLE_SCHEM         |TABLE_NAME                    |REMARKS
	------------------------------------------------------------------------
	
	0 rows selected
	snappy(PEERCLIENT)> CREATE SYNONYM myairline FOR airlines;
	0 rows inserted/updated/deleted
	snappy(PEERCLIENT)> SHOW SYNONYMS;
	TABLE_SCHEM         |TABLE_NAME                    |REMARKS
	------------------------------------------------------------------------
	APP                 |MYAIRLINE                     |
	
	1 row selected
	snappy(PEERCLIENT)>
### 显示表 ###

SHOW TABLES显示当前模式中的所有表。

如果IN schemaName指定，则显示给定模式中的表。

### 例 ###

	snappy(PEERCLIENT)> show tables in app;
	TABLE_SCHEM         |TABLE_NAME                    |REMARKS
	------------------------------------------------------------------------
	APP                 |AIRLINES                      |
	APP                 |CITIES                        |
	APP                 |COUNTRIES                     |
	APP                 |FLIGHTAVAILABILITY            |
	APP                 |FLIGHTS                       |
	APP                 |FLIGHTS_HISTORY               |
	APP                 |MAPS                          |
	
	7 rows selected
	snappy(PEERCLIENT)>
### SHOW VIEWS ###

SHOW VIEWS显示当前模式中的所有视图。

如果IN schemaName指定，将显示给定模式中的视图。

### 例 ###

	snappy(PEERCLIENT)> create view v1 as select * from maps;
	0 rows inserted/updated/deleted
	snappy(PEERCLIENT)> show views;
	TABLE_SCHEM         |TABLE_NAME                    |REMARKS
	------------------------------------------------------------------------
	APP                 |V1                            |
	
	1 row selected

## 等待 ##
显示以前启动的异步命令的结果。

### 句法 ###
	WAIT FOR Identifier

### 描述 ###
显示以前启动的异步命令的结果。

**异步**命令的标识符必须在此连接上的一个以前的**异步**命令中使用。等待命令等待SQL语句完成执行（如果尚未），然后显示结果。如果该语句返回一个结果集，则等待命令将遍历行，而不是**async**命令。此操作可能会导致在结果显示期间进一步执行时间过去。

### 例 ###
见**异步**。
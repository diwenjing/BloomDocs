# 使用DBSynchronizer将DML应用于RDBMS #
DBSynchronizer是一个内置的AsyncEventListener实现，可用于将数据异步地持久保存到符合JDBC 4.0标准的第三方RDBMS。<br/>

Pivotal在RowStore安装目录的examples / com / snappydata / rowstore / callbacks子目录中包含 DBSynchronizer实现的示例代码。有关详细信息，请参阅该目录中的README.txt文件。<br/>

DBSynchronizer使用多个参数来配置与第三方数据库的连接，以及同步行为和错误报告。参数的完整列表在DBSynchronizer初始化参数中描述，配置和使用DBSynchronizer中将显示完整的示例配置。<br/>

- **DBSynchronizer的工作原理** 您可以在多个数据存储上安装DBSynchronizer作为AsyncEventListener，最好是两个。DBSynchronizer只能安装在数据存储上（host-data配置为“true”）。

- **限制和限制** 此版本的RowStore对DBSynchronizer的使用有若干限制和限制。

- **配置和使用DBSynchronizer** 您可以**使用CREATE ASYNCEVENTLISTENER语句**安装和配置DBSynchronizer，类似于安装任何其他AsyncEventListener实现的方式。但是，DBSynchronizer需要特定的初始化参数来建立与第三方数据库的连接。

## DBSynchronizer的工作原理 ##
您可以在多个数据存储上安装DBSynchronizer作为AsyncEventListener，最好是两个。DBSynchronizer只能安装在数据存储上（`host-data`配置为“true”）。<br/>

DBSynchronizer的每个实例都维护一个内部队列来批处理DML语句。该队列由单个专用线程提供服务，该线程从队列中挑选批量的DML语句，并使用预准备语句将其应用于外部数据库。DBSynchronizer以与写入队列相同的顺序将所有DML应用于后端数据库。<br/>

- **确保高可用性和可靠的交付**

- **故障转移和升级如何影响同步**


> 注意： RowStore仅为单个DML事件调用监听器; 批量DML或批处理SQL语句不调用监听器。

确保高可用性和可靠的交付----------------为持久性和冗余配置DBSynchronizer队列，以确保高可用性和可靠的事件传递。RowStore在您在CREATE ASYNCEVENTLISTENER命令中指定的目标服务器组中的每个数据存储上安装DBSynchronizer实例。在多个数据存储上安装DBSynchronizer可提供高可用性。在任何给定时间，只有一个成员具有活动的DBSynchronizer线程，用于在外部数据库上执行DML。（在DBSynchronizer服务器组中启动的第一个数据存储启动活动实例。）其他成员的线程处于待机状态（冗余），以保证执行DML时，如果具有活动DBSynchronizer线程的成员失败。作为最佳做法，出于性能和内存原因，安装不超过两个备用DBSynchronizer实例（最多两个冗余）。默认情况下，如果活动成员关闭，驻留在DBSynchronizer的内部队列中的任何未决DML操作都将丢失。您可以通过将DBSynchronizer的内部队列配置为持久来避免丢失操作。 故障切换和升级如何影响同步----------------如果具有活动DBSynchronizer线程的成员失败，DML操作可能会重新应用于RDBMS。如果具有活动DBSynchronizer的成员在发送一批操作时失败，则批处理中的某些DML语句可能已经被应用于RDBMS。在故障转移时，新的DBSynchronizer线程重新发送失败的批次，并重新应用初始DML操作。当这种情况发生时，RDBMS可能会取决于DML操作的性质，它如何修改表列以及列约束的存在或不存在同步。<br/>


> 注意：升级RowStore服务器时，如果客户端在重新启动手动或滚动升级服务器时在同步表上主动执行DML，也可能会发生故障转移相关问题。

**如果表中有任何约束（主键，唯一）定义，以下类型的DML操作都 没有，当他们在故障转移期间被重新应用引起外的同步问题：**

- 重新应用于具有主键的表的创建操作。发生主键约束违规并抛出SQLException，但DBSynchronizer会忽略该异常。
- 导致唯一约束违规的创建或更新操作。重新应用创建或更新操作将导致重复的值，违反唯一约束。DBSynchronizer忽略在这种情况下抛出的SQLException。
- 导致检查约束违规的创建或更新操作。重新应用创建或更新（例如，增加或减少列值）可能会导致检查约束被违反。DBSynchronizer忽略在这种情况下抛出的SQLException。
在上述情况下，约束会阻止表与RowStore数据不同步。

**重新应用某些更新操作（如`update T1 set Col1 = 5 where col2 =7`）和删除操作（例如`delete from T1 where col1 = 5`）不会导致同步异常问题。**

**但是，以下类型的DML操作确实在故障转移期间重新应用时会导致同步问题：**

- 在没有主键约束的表上创建操作。
在这种情况下，重新应用创建操作会创建其他行。

- 更新相对于其当前值修改列值的操作。
重新应用更新操作（如更新）**T1 set col1 = col1 +? where col2 = ?**会导致外部数据库与RowStore数据不同步。

## 限制和限制(Restrictions and Limitations) ##
此版本的RowStore对DBSynchronizer的使用有若干限制和限制。

- **冗余配置**<br/>
DBSynchronizer的冗余只能通过您在CREATE ASYNCEVENTLISTENER命令中指定的目标服务器组进行控制。如果您只希望仅部署一个DBSynchronizer实例（无冗余），则必须确保目标服务器组只包含一个RowStore数据存储。否则，DBSynchronizer实现部署到服务器组中的所有数据存储。<br/>
当多个数据存储主机DBSynchronizer时，没有直接的方法来确保特定的RowStore数据存储主持活动的DBSynchronizer实例。如果需要特定成员托管活动实例，请确保部署DBSynchronizer并附加表时只有该成员正在目标服务器组中运行。然后，您可以在服务器组中启动剩余的数据存储，并且它们将托管冗余的备用DBSynchronizer实例。

- **身份列**<br/>
如果表具有自动生成的标识列，则默认情况下，DBSynchronizer不会将生成的标识列值应用于已同步的数据库。DBSynchronizer期望后端数据库表也有一个标识列，数据库生成自己的标识值。如果您希望DBSynchronizer将相同的RowStore生成的标识列值应用于底层数据库，然后`skipIdentityColumns=false`在DBSynchronizer参数文件中设置。请参阅**配置和使用DBSynchronizer**。

- **交易行为**<br/>
如果在事务范围内更新了一个同步表，则DBSynchronizer会将完整的DML语句传递给底层数据库，而不是同步表值。

- **批量DML**<br/>
DBSynchronizer不将批量DML语句的结果同步到底层数据库，而是将批量DML语句传递到同步数据库。

- **DBSynchronizer，AsyncEventHelper示例代码**<br/>
RowStore将内置的DBSynchronizer和AsyncEventHelper类的源代码安装到/ examples / com / snappydata / rowstore / callbacks目录中。此源代码与RowStore内置的类实现相同。使用示例通过复制和修改公共API的代码或扩展类来创建新类。记住：
	- 因为它使用内部的RowStore类，所以 不能原样编译AsyncEventHelper.java代码。扩展安装的AsyncEventHelper实现或将示例中的公共API代码复制到您自己的类中，根据需要修改代码以创建自定义帮助器类。

	- 该DBSynchronizer.java文件不使用内部类。但是，必须将示例代码复制到您自己的类中，以避免与内置的DBSynchronizer实现冲突。

- **触发器**<br/>
不支持使用DBSynchronizer触发器。在外部数据库和RowStore系统上定义的触发器可能导致触发器生成的DML的多次执行。

- **处理数据库连接问题**<br/>
如果DBSynchronizer在更新或提交数据库时遇到异常，则会保留批处理，并且DBSynchronizer线程将继续尝试应用批处理直到成功。

- **并发DML操作**<br/>
如果在具有基于父表和子表的外键关系的RowStore系统上同时执行DML操作，则在将外部数据库应用DML操作时可能会发生外键冲突。即使RowStore系统成功执行DML，也可能会发生这种情况。虽然插入到父表和子表中的顺序是RowStore系统，插入可能会以相反的顺序到达DBSynchronizer。要避免此行为，请在单个应用程序线程中执行父表和子表的更新。请参阅**AsyncEventListener如何工作**。

- **排队DML操作**<br/>
有一个窗口，其中在RowStore系统上执行的DML操作尚未被放置到DBSynchronizer的内部队列中。仅当DML操作完成时，才能保证记录被放入内部DBSynchronizer队列中。
此限制也适用于放置在WAN复制的网关发送器队列上的DML操作。

- **不区分大小写的标识符**<br/>
如果您的第三方数据库提供了使用区分大小写的SQL标识符的选项，则必须将数据库配置为使用不区分大小写的标识符。例如，如果你使用MySQL作为后端数据库必须将`lower\_case\_table\_name`服务器系统变量1.见的lower_case_table_names MySQL文档中获取更多信息。<br/>
根据SQL-99标准，DBSynchronizer将标识符视为不敏感，除非它们以引号分隔。具体来说，DBSynchronizer将标识符转换为INSERT操作的大写字符（但不是DELETE操作），因此后端数据库必须符合不区分大小写的标识符。

## 配置和使用DBSynchronizer ##
您可以使用CREATE ASYNCEVENTLISTENER语句安装和配置DBSynchronizer，类似于安装任何其他AsyncEventListener实现的方式。但是，DBSynchronizer需要特定的初始化参数来建立与第三方数据库的连接。<br/>

### 先决条件 ###

使用与RowStore数据库中使用的名称相同的名称，使用模式和表定义来预配置第三方数据库。部署DBSynchronizer时，可以指定第三方数据库（实现java.sql.Driver的类）的JDBC驱动程序的名称以及用于连接数据库的完整JDBC连接字符串URL。您可以在URL中包含任何所需的连接参数，例如用户凭据，数据库名称等，或作为单独的初始化参数。DBSynchronizer使用驱动程序，URL和参数来获取到第三方RDBMS的java.sql.Connection。DBSynchronizer初始化参数提供了对所有DBSynchronizer初始化参数的完整引用。<br/>

### 程序 ###

以下过程介绍如何使用DBSynchronizer配置RowStore以将表DML操作应用于第三方数据库。<br/>

> 注意：此过程中的示例命令使用[MySQL社区服务器5.5.18](http://www.mysql.com/downloads/mysql/)作为具有Connector / J 5.1.18驱动程序的第三方数据库。但是，无论您使用哪个数据库，一般步骤都相同。有关详细信息，请参阅第三方数据库和JDBC驱动程序文档。

> 注意：如果要在MySQL数据库中同步表，则必须将MySQL配置为使用不区分大小写的标识符。请参阅限制和限制。

1. **配置第三方数据库**<br/>
在RowStore中配置DBSynchronizer之前，请安装并配置第三方数据库。为您要在RowStore中使用的所有表创建数据库模式和表定义。<br/>
为了确保表与RowStore中的表保持同步，只能在第三方数据库中创建表定义。仅在RowStore中插入和更新行，以便DBSynchronizer可以将更改应用于数据库。<br/>
例如，如果尚未运行，则启动MySQL：<br/>

		$ sudo mysqld_safe
在单独的终端提示符下，启动mysql客户端并创建一个新的数据库和表定义。使用具有创建新数据库权限的帐户登录。例如：<br/>

		$ mysql -u my_username -p
		mysql> create database snappystoredb;
		mysql> use snappystoredb;
		mysql> create table snappystoretest
		    -> (id int not null, name varchar(10));

	您将在RowStore中创建一个名称为“snappystoretest”的表，并将其与DBSynchronizer相关联。

1. **配置JDBC连接到第三方数据库**<br/>
DBSynchronizer需要JDBC驱动程序和连接URL才能连接到第三方数据库并应用DML操作。配置DBSynchronizer之前，请使用Java客户端应用程序（如SQuirreL SQL）来验证是否可以连接到数据库。<br/>
在此过程中显示的MySQL示例中，建立JDBC连接所需的组件有：

		-   **Driver JAR file:** mysql-connector-java-5.1.18-bin.jar
		-   **Driver class:** com.mysql.jdbc.Driver
		-   **Connection URL:** jdbc:mysql://localhost:3306/snappystoredb?user=*snappystoreuser*&password=*snappystorepassword* <p class="note"><strong>Note:</strong> Although you can include the username and password directly in the JDBC connection URL (as shown above), doing so runs the risk of having the plain-text password appear in exception messages. To avoid recording plain-text passwords, this example will use an encrypted secret generated using the <a href="../reference/store_commands/store-encrypt-password.html#reference_13F8B5AFCD9049E380715D2EF0E33BDC" class="xref noPageCitation" title="Generates an encrypted password string for use in the gemfirexd.properties file when configuring BUILTIN authentication, or when accessing an external data source with an AsyncEventListener implementation or DBsynchronizer configuration.">encrypt-password</a> command.</p>
		
		To ensure that RowStore can access the JDBC driver class, add the JAR location to your CLASSPATH. For example, open a new command prompt and enter:
		
		``` pre
		$ export CLASSPATH=$CLASSPATH:/path/mysql-connector-java-5.1.18-bin.jar
		```

1. **在自定义服务器组中启动RowStore数据存储**<br/> 
创建新的AsyncEventListener（如DBSynchronizer）时，必须将侦听器分配给可用的服务器组。例如，要启动RowStore数据存储并将其添加到命名服务器组：

		$ mkdir snappystoredbsync
		$ cd snappystoredbsync
		$ export CLASSPATH=$CLASSPATH:/path/mysql-connector-java-5.1.18-bin.jar
		$ snappy-shell rowstore server start -server-groups=dbsync -mcast-port=10334
	这将启动一个新的RowStore数据存储服务器并将其分配给“dbsync”服务器组。

1.  **加密数据库凭据**<br/>
使用`snappy-shell encrypt-password`带有该`external`选项的命令来加密数据库凭据：

		$ snappy-shell encrypt-password external -mcast-port=10334
		Enter User Name: snappystoreuser
		Enter password: snappystorepassword
		Re-enter password: snappystorepassword
		Connecting to distributed system: mcast=/239.192.81.1:10334
		Encrypted to 25325ffc3345be8888eda8156bd1c313

	> 注意：执行`snappy-shell encrypt-password`命令时，指定RowStore成员用于连接到分布式系统的相同连接属性。例如，指定相同的定位器或多播连接属性以及成员加入分布式系统所需的任何授权凭据。

	返回的加密密码特定于这个特定的RowStore分布式系统，因为系统使用唯一的私钥来生成秘密。（私钥的混淆版本存储在持久性数据字典中。）配置DBSynchronizer初始化参数时，将使用加密密码，以使明文密码永远不会出现在异常消息中。上述示例使用默认AES转换和128位密钥大小进行加密。有关更改这些选项的信息，请参阅**encrypt-password**。<br/>
	如果您需要将DBSynchronizer配置移动到另一个RowStore系统，或者如果现有数据字典被删除和重新创建，那么您必须生成并使用新的加密密钥来与新的分布式系统一起使用。

1. **创建AsyncEventListener配置**<br/>
首先创建参数文件来保存访问数据库所需的连接信息。使用文本编辑器创建文件/ usr / local / dbsync-params。将以下参数添加到文件中，每行上都有一个参数定义：

		Driver=com.mysql.jdbc.Driver
		URL=jdbc:mysql://localhost:3306/snappystoredb
		User=snappystoreuser
		secret=25325ffc3345be8888eda8156bd1c313
		SkipIdentityColumns=true
有关所有DBSynchronizer初始化参数的完整引用，请参阅DBSynchronizer初始化参数。
这`SkipIdentityColumns`是一个可选参数，默认为“true”。如果要将RowStore生成的标识值传递给底层数据库，请将此参数设置为具有自动生成的标识列的表。请参阅**身份列**。

	> 注意：作为最佳做法，将所有初始化参数放在单独的文件中。在RowStore中配置DBSynchronizer实现后，可以更改参数（例如，RDBMS凭据）。

	> 注意：为了确保系统的安全，请保护包含加密密钥的DBSynchronizer参数文件以及RowStore分布式系统的持久数据字典文件。虽然数据字典模糊了用于生成加密秘密的私钥，但如果DBSynchronizer配置文件或数据字典遭到破坏，您应该考虑任何密码进行泄密。

	保存参数文件后，连接到RowStore并使用DBSynchronizer创建一个新的AsyncEventListener。确保已将第三方JDBC驱动程序JAR文件添加到类路径中，然后启动`snappy-shell`并创建侦听器定义：<br/>

		$ export CLASSPATH=$CLASSPATH:/path/mysql-connector-java-5.1.18-bin.jar
		$ snappy
		snappy> connect client 'localhost:1527';
		snappy> create asynceventlistener testlistener
		> (
		> listenerclass 'com.pivotal.snappydata.callbacks.DBSynchronizer' 
		> initparams 
		>   'file=/usr/local/dbsync-params' 
		> )
		> server groups (dbsync);

	> 注意：您可以选择安装并启动AsyncEventListener配置*后*将表与监听器名称相关联。请确保使用与CREATE ASYNCEVENTLISTNER命令和CREATE TABLE命令相同的侦听器名称。

	LISTENERCLASS参数必须指定内置的RowStore DBSynchronizer实现。<br/>
	INITPARAMS参数仅指定包含DBSynchronizer初始化参数的文件路径。使用参数文件是最佳做法，因为您可以在配置DBSynchronizer实现后更改参数。或者，您可以直接在INITPARAMS字符串中指定所有参数，如下例所示：<br/>

		snappy> create asynceventlistener testlistener
		> (
		> listenerclass 'com.pivotal.snappydata.callbacks.DBSynchronizer' 
		> initparams 
		>   'com.mysql.jdbc.Driver,jdbc:mysql://localhost:3306/snappystoredb,SkipIdentityColumns=true,user=snappystoreuser,secret=25325ffc3345be8888eda8156bd1c313' 
		> )
		> server groups (dbsync);

	> 注意：如果直接在INITPARAMS字符串中指定参数，则必须删除并重新创建DBSynchronizer配置才能更改或添加参数（例如，更改第三方数据库凭据）。

	RowStore在指定的SERVER GROUPS中的每个数据存储上安装DBSychronizer实例。确保服务器组至少有两个数据存储，以便为DBSynchronizer操作提供冗余。作为最佳做法，由于性能和内存原因，安装不超过两个备用DBSynchronizer实例（至多两个冗余）。<br/>
	当多个数据存储器托管事件监听器时，没有直接的方法可以确保特定的RowStore数据存储器托管活动实例。如果需要特定成员托管活动实例，请确保在部署监听器并附加表时只有该成员正在目标服务器组中运行。然后，您可以在服务器组中启动剩余的数据存储，并且它们将托管冗余的备用实例。<br/>

1. **启动AsyncEventListener**<br/>
使用SYS.START_ASYNC_EVENT_LISTENER过程启动新的DBSynchronizer实现：

		snappy> call sys.start_async_event_listener('TESTLISTENER');


	> 注意： AsyncEventListener名称是一个SQL标识符，应使用此过程以大写字母输入。

1. **在RowStore中创建相同的模式和表**<br/>
在RowStore中创建DBSynchronizer侦听器后，创建表并将其与侦听器相关联，以使DBSynchronizer将后续表DML操作传播到第三方数据库。您创建的表应使用与第三方数据库中使用的表相同的模式，表名和表定义。例如：

		snappy> create schema snappystoredb;
		snappy> set schema snappystoredb;
		snappy> create table snappystoretest
		> (id int not null, name varchar(10))
		> asynceventlistener(testlistener);
请注意，表定义与MySQL中使用的定义相同，但包括ASYNCEVENTLISTENER子句将表与DBSynchronizer相关联。

	> 注意：由于关联的侦听器配置在创建表时不必可用，如果指定的侦听器名称不存在，则RowStore不会显示错误消息。请确保使用与CREATE ASYNCEVENTLISTNER命令和CREATE TABLE命令相同的侦听器名称。

1. **执行DML并验证同步**<br/>
将表与DBSynchronizer实现关联后，RowStore会将针对表执行的DML操作排队到DBSynchronizer INITPARAMS参数中指定的第三方数据库。例如，仍然在gfxd，执行：

		snappy> insert into snappystoretest values (1, '1st Entry');
然后返回到mysql客户端并验证DML是否传播到您的数据库：

		mysql> select * from snappystoredb.snappystoretest;

1. **从DBSynchronizer处理中排除DML操作是必要的**<br/>
默认的DBSynchronizer实现将所有DML操作传播到配置的数据库。如果要仅在RowStore上执行某些DML操作（不将这些更改传播到已配置的监听器），则在连接到分布式系统时将skip-listeners属性设置为true。将skip-listeners设置为true会阻止在连接上执行的所有DML语句被排队并传递到配置的监听器实现。

1. **将DBSynchronizer连接信息更改为必要**<br/>
如果将DBSynchronizer初始化参数存储在单独的文件（推荐）中，则可以通过编辑文件然后重新启动DBSynchronizer配置来轻松修改这些参数。例如，如果需要更新与第三方数据库相关联的用户的密码，请先编辑参数文件以包含正确的密码。然后停止并重新启动DBSynchronizer：

		snappy> call sys.stop_async_event_listener('TESTLISTENER');
		snappy> call sys.start_async_event_listener('TESTLISTENER');
有关所有DBSynchronizer初始化参数的完整引用，请参阅DBSynchronizer初始化参数。

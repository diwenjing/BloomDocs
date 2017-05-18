# snappy-shell启动器命令 #
使用gfxd命令行实用程序启动RowStore实用程序。

- 句法

- 描述

### 句法 ###

> 注意：尽管RowStore引入了“snappy-shell”实用程序来替换早期的“sqlf”实用程序，但是为了方便，“sqlf”仍然作为可选语法提供和支持。

显示snappy-shell命令和选项的完整列表：

	snappy-shell --help
显示特定实用程序用法的命令窗体是：

	snappy-shell <utility> --help
没有参数，`snappy-shell`启动**交互式SQL命令shell**：

	snappy-shell
要为交互式`snappy-shell`会话指定系统属性，必须在开始之前定义JAVA_ARGS环境变量`snappy-shell`。例如，`snappy-shell`使用`snappy-shell.historysystem`属性定义存储在交互式会话期间执行的命令列表的文件。要更改此属性，您可以将其定义为JAVA_ARGS变量的一部分：

	$ export JAVA_ARGS="-Dgfxd.history=/Users/yozie/snappystore-history.sql"
	$ snappy
要启动和退出`snappy-shell`实用程序（而不是启动交互式`snappy-shell`shell），请使用以下语法：

	snappy-shell <utility> <arguments for specified utility>
在此命令窗体中，<utility>是以下之一。

|实用程序名称|描述|
|:--|:--|
|服务器|启动并停止RowStore Server成员，并提供正在运行的状态。|
|定位器|启动并停止RowStore定位器成员，并提供正在运行的状态。|
|版|打印RowStore产品版本信息。|
|统计|从统计档案中打印统计值。|
|上合并日志|将多个日志文件合并到一个日志中。|
|执照|打印获取新许可证所需的信息。|
|加密密码|在使用BUILTIN身份验证时加密gemfirexd.properties文件中使用的密码，或者使用AsyncEventListener实现或DBSynchronizer的外部数据源使用密码。|
|验证磁盘店|确认磁盘存储的文件是否有效。|
|版|显示RowStore产品版本信息。|
|紧凑盘店|压缩脱机磁盘存储，以从永久文件中删除所有不需要的记录。|
|紧凑的全磁盘存储|要求所有分布式系统成员压缩其磁盘存储。|
|撤销缺失盘店|要求分布式系统成员停止等待指定的磁盘存储。|
|列表失踪磁盘存储|打印目前从分布式系统中丢失的磁盘存储。|
|关机，所有|请求所有数据存储和访问者系统成员关闭。|
|备用|请求所有分布式系统成员备份其持久数据。|

要在启动`snappy-shell`实用程序时指定系统属性，请使用-JD property_name = property_value参数。


### 描述 ###
除了启动RowStore提供的各种实用程序之外，当没有任何参数`snappy-shell`启动时，启动一个交互式命令shell，您可以使用它来连接到一个RowStore系统并执行各种命令，包括SQL语句。

启动器会对当前的CLASSPATH环境变量进行荣誉，并将其添加到正在启动的实用程序或命令shell的CLASSPATH中。要将其他参数传递给JVM，请`JAVA\_ARGS`在调用gfxd脚本时设置环境变量。

> 注意： `JAVA \ _ARGS`环境变量不适用于启动单独后台进程的`snappy-shell rowstore server`和`snappy-shell rowstore locator`工具。要将Java属性传递给这些工具，请使用“-J”选项，如这些工具的帮助中所述。

启动器使用`javaPATH`中找到的可执行文件。要覆盖此行为，请将`GFXD\_JAVA`环境变量设置为指向所需的Java可执行文件。（请注意**支持的配置和系统要求**中支持的JRE版本）。

- **备份** 为在分布式系统中运行的所有成员创建操作磁盘存储的备份。具有持久数据的每个成员创建自己的配置和磁盘存储的备份。

- **compact-all-disk-stores** 执行在线压缩RowStore磁盘存储。

- **compact-disk-store** 执行单个RowStore磁盘存储的离线压缩。

- **encrypt-password**在配置BUILTIN身份验证或使用AsyncEventListener实现或DBsynchronizer配置访问外部数据源时，生成用于 gemfirexd.properties文件的**加密**密码字符串。

- **install-jar** 安装JAR文件并自动将JAR文件类加载到RowStore类加载器中。这使得JAR类可用于分布式系统的所有成员（包括稍后加入系统的成员）。

- **list-missing-disk-stores** 列出其他成员正在等待的最新数据的所有磁盘存储。

- **定位器** 允许分布式系统中的对等体（包括其他定位器）发现对方，而无需对任何其他对等体进行硬编码。

- **日志记录支持** 您可以指定JDBC引导属性`snappy-shell rowstore server`和`snappy-shell rowstore locator`命令，以分别为RowStore服务器和定位器配置日志文件位置和日志严重性级别。

- **merge-logs** 将多个日志文件合并到一个日志中。

- **print-stacks** 打印RowStore成员进程的堆栈转储。

- **remove-jar** 删除JAR文件安装，卸载与JAR关联的所有类。

- **replace-jar **使用新的JAR文件的内容替换已安装的JAR文件。新JAR文件中的类将自动加载到RowStore类加载器中，并替换以前为相同JAR标识符安装的任何类。RowStore还重新编译依赖于JAR文件的对象，例如已安装的监听器实现。

- **revoke-missing-disk-store** 指示RowStore成员停止等待磁盘存储可用。

- **运行** 连接到RowStore分布式系统并执行SQL命令文件的内容。指定文件中的所有命令必须与交互式gfxd shell兼容。

- **服务器** RowStore服务器是RowStore系统中的主要服务器端组件，可提供与集群中其他服务器，对等体和客户端的连接。它可以托管数据。使用 gfxd启动器的服务器实用程序启动服务器。

- **show-disk-store-metadata** 显示指定磁盘存储目录的磁盘存储元数据。

- **shutdown-all** 指示所有RowStore访问器和数据存储成员断开与分布式系统的连接并关闭。

- **统计信息** 显示统计档案中的统计值。

- **upgrade-disk-store** （此版本不支持）将磁盘存储升级到当前版本的RowStore。

- **validate-disk-store** 验证脱机磁盘存储的运行状况，并使用该磁盘存储提供有关表的信息。

- **版本** 打印有关RowStore产品版本的信息。

- **write-data-dtd-to-file** 创建文档类型定义（DTD）文件，该文件指定使用创建的XML数据文件的布局`snappy-shell write-data-to-xml`。

- **write-data-to-db** 使用一个或多个数据XML文件（使用snappy-shell write-data-to-xml创建）将数据插入数据库，并将数据库模式定义在一个或多个模式XML文件（与snappy-shell write-schema-to-xml）。该命令通常与RowStore集群一起用于导出表数据，但也可以与其他JDBC数据源一起使用。

- **write-data-to-xml** 将数据库中的所有表的数据写入XML文件。（您可以使用snappy-shell write-data-dtd-to-file命令创建描述XML文件中数据布局的文档类型定义（DTD）文件。）生成的XML文件可用于重新生成在RowStore或另一个数据库管理系统中将数据加载到表中。该命令通常与RowStore集群一起用于导出表数据，但也可以与其他JDBC数据源一起使用。

- **write-schema-to-db** 通过使用一个或多个模式XML文件的内容（请参阅snappy-shell write-schema-to-xml）在数据库中创建模式。此命令通常与RowStore集群一起用于导出模式，但也可以与其他JDBC数据源一起使用。

- **write-schema-to-sql** 将数据库的模式作为SQL命令写入文件。您可以使用生成的文件在RowStore或另一个数据库管理系统中重新创建模式。此命令通常与RowStore集群一起用于导出模式，但也可以与其他JDBC数据源一起使用。

- **write-schema-to-xml** 将数据库的模式写入XML文件。您可以使用生成的XML文件在RowStore或另一个数据库管理系统中重新创建模式。此命令通常与RowStore集群一起用于导出模式，但也可以与其他JDBC数据源一起使用。


## 备份 ##
为在分布式系统中运行的所有成员创建操作磁盘存储的备份。具有持久数据的每个成员创建自己的配置和磁盘存储的备份。


> 注意： RowStore不支持在具有实时事务的系统上备份磁盘存储，或者正在执行并发DML语句。请参阅备份和恢复磁盘存储。

- **语法** 
- **说明** 
- **先决条件和最佳实践** 
- **指定备份目录** 
- **示例** 
- **从snappy** 
- **shell备份输出消息** 
- **备份目录结构及其内容** 
- **恢复在线备份**

### 句法 ###
在命令行上使用mcast-port和-mcast-address或-locators选项连接到RowStore集群。

	snappy-shell backup [-baseline=<baseline directory>] <target directory> [-J-D<vmprop>=<prop-value>]
	 [-mcast-port=<port>] [-mcast-address=<address>]
	 [-locators=<addresses>] [-bind-address=<address>] [-<prop-name>=<prop-value>]*
或者，您可以在运行命令的目录中可用的gemfirexd.properties文件中指定这些和其他分布式系统属性`snappy-shell`。

该表描述了snappy-shell备份的选项。

|选项|描述|
|:--|:--|
|- 基线|包含用于在增量备份期间进行比较的基准备份的目录。基准目录对应于执行原始备份命令的日期，而不是指定的备份位置（例如，有效的基准目录可能类似于/ export / fileServerDirectory / gemfireXDBackupLocation / 2012-10-01-12-30）。<br/>增量备份操作将备份指定目录中尚未存在的任何数据`-baseline`。如果成员找不到以前备份的数据，或者如果先前备份的数据已损坏，则命令将在该成员上执行完整备份。（如果省略该`-baseline`选项，则该命令还会执行完全备份。|
|<目标目录>|[RowStore存储备份内容的目录。请参阅指定备份目录。](http://rowstore.docs.snappydata.io/docs/reference/store_commands/store-backup.html)|
|-mcast端口|组播端口用于与分布式系统的其他成员进行通信。如果为零，组播不会用于成员发现（请指定“-locators”）。|
||有效值范围为0-65535，默认值为10334。|
|-mcast地址|组播地址用于发现分布式系统的其他成员。仅当未指定`-locators`选项时，才使用此值。|
||默认多播地址为239.192.81.1。|
|-locators|用于发现分布式系统成员的定位器列表。以逗号分隔的主机：端口值提供所有定位器。|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下，如果主机名指向本地环回地址，则RowStore将使用主机名或本地主机。|
|- 丙名|任何其他RowStore分布式系统属性。|

### 描述 ###
在线备份保存以下内容：

- 对于具有持久数据的每个成员，备份包括包含持久表数据的所有存储的磁盘存储文件。

- 来自成员启动的配置文件。

- gemfirexd.properties，具有该成员启动的属性。

- 为会员的操作系统编写的还原脚本将文件复制回原始位置。例如，在Windows中，该文件是restore.bat，而在Linux中，它是restore.sh。

### 先决条件和最佳实践 ###
- 在系统中的低活动期间运行此命令。备份不会阻止系统活动，但它会在分布式系统中的所有主机上使用文件系统资源，并可能影响性能。

- 不要尝试使用文件复制命令从运行的系统创建备份文件。您将收到不完整和不可用的副本。

- 确保目标备份目录目录存在，并具有适当的权限，供成员写入并创建子目录。

- 运行备份之前，您可能需要压缩磁盘存储。请参阅**compact-all-disk-stores**命令。

- 确保托管持久数据的RowStore成员在分布式系统中运行。离线成员无法备份其磁盘存储。（如果所有表数据在运行成员中可用，则仍可执行完整的备份。）

### 指定备份目录 ###
您为备份指定的目录可以多次使用。每个备份首先为备份创建一个顶级目录，在您指定的目录下，标识为分钟。您可以使用以下两种格式之一：

- 使用单个物理位置，例如网络文件服务器。（例如/ export / fileServerDirectory / snappyStoreBackupLocation）。

- 使用系统中所有主机的本地目录。（例如，./snappyStoreBackupLocation）。

### 例 ###
使用系统中所有主机的本地备份目录：

	snappy-shell backup  ./snappyStoreBackupLocation
	  -locators=warsaw.pivotal.com[26340]
另请参阅**备份和恢复磁盘存储**。

要稍后执行增量备份：

	snappy-shell backup -baseline=./snappyStoreBackupLocation/2012-10-01-12-30 ./snappyStoreBackupLocation 
	  -locators=warsaw.pivotal.com[26340] 

> 注意： RowStore不支持在具有实时事务的系统上执行增量备份，或者正在执行并发DML语句。


### 输出来自snappy-shell备份的消息 ###
运行`snappy-shell backup`时，会报告操作的结果。

如果任何成员在运行`snappy-shell backup`时离线，您会收到以下消息：

	The backup may be incomplete. The following disk
	stores are not online:  DiskStore at hostc.pivotal.com
	/home/dsmith/dir3
如果所有表数据在运行成员中可用，仍可执行完整的备份。

该工具报告了操作的成功。如果操作成功，您会看到如下消息：

	Connecting to distributed system: locators=warsaw.pivotal.com26340
	The following disk stores were backed up:
	DiskStore at hosta.pivotal.com /home/dsmith/dir1
	DiskStore at hostb.pivotal.com /home/dsmith/dir2
	Backup successful.
如果在备份所有已知成员时操作不成功，您会看到如下消息：

	Connecting to distributed system: locators=warsaw.pivotal.com26357
	The following disk stores were backed up:
	DiskStore at hosta.pivotal.com /home/dsmith/dir1
	DiskStore at hostb.pivotal.com /home/dsmith/dir2
	The backup may be incomplete. The following disk stores are not online:
	DiskStore at hostc.pivotal.com /home/dsmith/dir3
在该结束状态消息中注意到无法完成其备份的成员，并将文件INCOMPLETE_BACKUP留在其最高级备份目录中。


### 备份目录结构及其内容 ###
以下是在分布式系统中备份的文件和目录的结构：

	 2011-05-02-18-10 /:
	pc15_8933_v10_10761_54522
	2011-05-02-18-10/pc15_8933_v10_10761_54522:
	config diskstores README.txt restore.sh
	2011-05-02-18-10/pc15_8933_v10_10761_54522/config:
	gemfirexd.properties
	2011-05-02-18-10/pc15_8933_v10_10761_54522/diskstores:
	GFXD_DD_DISKSTORE
	2011-05-02-18-10/pc15_8933_v10_10761_54522/diskstores/GFXD_DD_DISKSTORE:
	dir0
	2011-05-02-18-10/pc15_8933_v10_10761_54522/diskstores/GFXD_DD_DISKSTORE/dir0:
	BACKUPGFXD-DD-DISKSTORE_1.crf
	BACKUPGFXD-DD-DISKSTORE_1.drf BACKUPGFXD-DD-DISKSTORE_2.crf
	BACKUPGFXD-DD-DISKSTORE_2.drf BACKUPGFXD-DD-DISKSTORE.if

### 恢复在线备份 ###
还原脚本（restore.sh或restore.bat）将文件复制回原始位置。如果您愿意，您可以手动执行：

1. 当您的会员离线并且系统关闭时，恢复磁盘存储。

1. 阅读还原脚本以查看它们将放置文件的位置，并确保目的地位置已准备就绪。恢复脚本拒绝复制具有相同名称的文件。

1. 运行还原脚本。运行备份所在主机上的每个脚本。
恢复将它们复制回原始位置。


## 紧凑的全磁盘存储 ##
执行RowStore磁盘存储的在线压缩。

### 句法 ###
	snappy-shell compact-all-disk-stores
	 [-mcast-port=<port>] [-mcast-address=<address>]
	 [-locators=<addresses>] [-bind-address=<address>] [-<prop-name>=<prop-value>]*
该表描述了snappy-shell compact-all-disk-stores的选项。如果在命令行中未指定多播或定位器选项，则该命令将使用gemfirexd.properties文件（如果可用）来确定应连接的分布式系统。

|选项|描述|
|:--|:--|
|-mcast端口|组播端口用于与分布式系统的其他成员进行通信。如果为零，组播不会用于成员发现（请指定“-locators”）。有效值范围为0-65535，默认值为10334。|
|-mcast地址|组播地址用于发现分布式系统的其他成员。仅当未指定`-locators`选项时，才使用此值。|
||默认多播地址为239.192.81.1。|
|-locators|用于发现分布式系统成员的定位器列表。以逗号分隔的主机：端口值提供所有定位器。|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下gfxd，如果主机名指向本地环回地址，则使用主机名或本地主机。|
|- <丙名> = <丙值>|任何其他RowStore分布式系统属性。|

### 描述 ###
当在持久/溢出表上执行CRUD操作时，将数据写入日志文件。同一行的任何预先存在的操作记录都将过时，RowStore将其标记为垃圾。它通过将所有非垃圾记录复制到当前日志并丢弃旧文件来压缩旧的操作日志。

在线和离线磁盘存储可以进行手动压缩。对于在线磁盘存储，当前操作日志不可用于压缩，无论它包含多少垃圾。

离线压缩基本上以相同的方式运行，但没有进入的CRUD操作。另外，因为没有当前打开的日志，压缩创建一个新的开始日志。


### 在线压实 ###
要运行手动在线压缩，allow-force-compaction应该是true。您可以随时在系统运行时运行手动在线压缩。基于压实阈值的符合压缩条件的Oplog将压缩到当前的oplog中。

## 紧凑盘店 ##
执行单个RowStore磁盘存储的离线压缩。

### 句法 ###
	snappy-shell compact-disk-store <diskStoreName> <directory>+ [-maxOplogSize=<int>]

### 描述 ###

> 注意：不要在增量备份的基准目录上执行脱机压缩。

当在持久/溢出表上执行CRUD操作时，数据将被写入日志文件。同一行的任何预先存在的操作记录都将过时，RowStore将其标记为垃圾。它通过将所有非垃圾记录复制到当前日志并丢弃旧文件来压缩旧的操作日志。

在线和离线磁盘存储可以进行手动压缩。对于在线磁盘存储，当前操作日志不可用于压缩，无论它包含多少垃圾。

离线压缩基本上以相同的方式运行，但没有进入的CRUD操作。另外，因为没有当前的打开日志，压缩创建一个新的开始日志。

> 注意： 您必须提供磁盘存储中的所有目录。如果没有指定oplog max size，RowStore将使用系统默认值。离线压缩可以消耗大量的内存。如果在运行此命令时遇到java.lang.OutOfMemory错误，则需要通过在JAVA \ _ARGS环境变量中设置“-Xmx”和“-Xms”选项来增加堆大小。snappy -shell Launcher命令提供了有关设置Java选项的更多信息。

### 例 ###
	snappy-shell compact-disk-store myDiskStoreName  /firstDir  /secondDir   
	maxOplogSize=maxMegabytesForOplog
此命令的输出类似于：

	Offline compaction removed 12 records.
	Total number of region entries in this disk store is: 7

## 加密密码 ##
生成加密的密码字符串，以在配置BUILTIN身份验证时使用gemfirexd.properties文件，或在使用AsyncEventListener实现或DBsynchronizer配置访问外部数据源时生成加密密码字符串。

### 句法 ###
	snappy-shell encrypt-password
	  [external]
	  [-transformation=<name>]
	  [-keysize=<size>]
	  [-J-D<vmprop>=<prop-value>]  
	  [-mcast-port=<port>]
	  [-mcast-address=<address>]
	  [-locators=<addresses>]
	  [-bind-address=<addr>]
	  [-<prop-name>=<prop-value>]*
命令提示输入密码，然后在控制台上显示加密的密码（使用选项，如果指定）。如果控制台不可用，则抛出异常。如果external包含该选项，加密密码将存储在DBSynchronizer或AsyncEventListener实现中用于外部使用的数据字典中。

> 注意：执行`snappy-shell encrypt-password`命令时，指定RowStore成员用于连接到分布式系统的相同连接属性。例如，指定相同的定位器或多播连接属性以及成员加入分布式系统所需的任何授权凭据。

|选项|描述|
|:--|:--|
|外部|包括在`external` RowStore分布式系统中加密和存储密码的选项，供DBSynchronizer或自定义AsynchEventListener实现访问的外部资源使用。有关详细信息，请参阅**配置和使用DBSynchronizer**或**配置和使用**AsyncEventListener。<br/><br/>当您指定此选项，则必须提供额外的选项连接到运行RowStore分布式系统（无论是`-locators`选项或`-mcast-port`和`-mcast-address`）。分布式系统在数据字典中生成私钥以加密密码。您可以使用该`AsyncEventHelper.decryptPassword`方法解密您的AsyncEventListener实现中的密码，以便使用外部数据源进行身份验证。<br/><br/>该选项还可以在与该结合使用`-transformation`和`-keysize`选择，下文描述。<br/><br/>注意：每个RowStore分布式系统都会生成自己的私钥，加密值特定于特定的分布式系统。如果例如数据字典被破坏并创建了新的数据字典，则重新生成密钥。在这种情况下，您需要使用新的加密密码`snappy-shell encrypt-password`。|
|-转型|此选项仅与`-external`选项结合使用。用于对称密钥加密的转换（加密算法名称）。RowStore默认使用AES加密密钥工厂。支持以下算法名称：<br/>AES<br/> ARCFOUR<br/>DES<br/>DESede<br/>PBKDF2WithHmacSHA1<br/>PBEWith <消化>和<加密><br/>PBEWith <PRF>和<加密><br/>最后两个算法定义了用于PKCS5加密的工厂。指定加密算法名称以及摘要或伪随机函数（PRF）来配置工厂（例如，PBEWithMD5AndDES）。<br/><br/>有关这些算法的更多 信息，请参阅[Java加密架构Sun提供商](http://docs.oracle.com/javase/6/docs/technotes/guides/security/SunProviders.html)文档。|
|-keysize|此选项仅与`-external`选项结合使用。用于加密密钥的密钥大小。默认值为128位。|
|-mcast端口|组播端口用于与分布式系统的其他成员进行通信。如果为零，组播不会用于成员发现（请指定“-locators”）。<br/><br/>有效值范围为0-65535，默认值为10334。|
|-mcast地址|组播地址用于发现分布式系统的其他成员。仅当未指定`-locators`选项时，才使用此值。<br/><br/>默认多播地址为239.192.81.1。|
|-locators|用于发现分布式系统成员的定位器列表。以逗号分隔的主机：端口值提供所有定位器。|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下`gfxd`，如果主机名指向本地环回地址，则使用主机名或本地主机。|
|- <丙名> = <丙值>|任何其他RowStore分布式系统属性。|

### 描述 ###
#### 例 ####
当没有`external`选项使用时，`snappy-shell`提示输入密码进行加密，然后将密码显示给控制台。返回的加密密码特定于这个特定的RowStore分布式系统，因为系统使用唯一的私钥来生成秘密。私钥的混淆版本存储在持久性数据字典中。如果您需要将DBSynchronizer配置移动到另一个RowStore系统，或者如果现有数据字典被删除和重新创建，那么您必须生成并使用新的加密密钥来与新的分布式系统一起使用。

> 注意：为了确保系统的安全，请保护包含加密密钥的任何AsyncEventListener / DBSynchronizer参数文件以及RowStore分布式系统的持久数据字典文件。虽然数据字典模糊了用于生成加密秘密的私钥，但如果配置文件或数据字典遭到破坏，您应该考虑任何密码被盗用。

	snappy-shell encrypt-password -mcast-port=10334
	Enter User Name: test_user
	Enter password: test_encryption (not echoed to screen)
	Re-enter password: test_encryption (not echoed to screen)
	Encrypted to v23b60032c17ab973929e43d60acc597887a5f3d5658bd
然后，您可以将加密密码添加到gemfirexd.properties文件中指定的BUILTIN用户帐户，如**使用BUILTIN身份验证中所述**。

另请参阅**配置和使用DBSynchronizer**作为**使用DBSynchronizer**参数文件中的共享密钥文件的示例。

## 安装-JAR ##
安装JAR文件并自动将JAR文件类加载到RowStore类加载器中。这使得JAR类可用于分布式系统的所有成员（包括稍后加入系统的成员）。

### 句法 ###
要将本地JAR文件安装到RowStore，请使用以下语法：

	snappy-shell install-jar -file=<path or URL> -name=<identifier>
	     [-auth-provider=<name>]
	     [-bind-address=<address>]
	     [-client-bind-address=<address>]
	     [-client-port=<port>]
	     [-extra-conn-props=<properties>] 
	     [-help] [-locators=<adresses>]
	     [-mcast-address=<address>]
	     [-mcast-port=<port>]
	     [-password[=<password>]]
	     [-user=<username>]
此表描述了gfxd install-jar命令的选项。如果不指定选项，则使用默认值。

|选项|描述|
|:--|:--|
|-文件|要安装的JAR文件的本地路径，或链接到JAR文件的URL。<br/><br/>这个参数是必需的。|
|-名称|JAR安装的唯一标识符。您提供的标识符必须指定一个模式名称分隔符。例如：APP.myjar。<br/><br/>这个参数是必需的。|
|-auth提供商|设置用于对等连接以及客户端 - 服务器连接的身份验证提供程序。有效值为BUILTIN和LDAP。RowStore分布式系统的所有其他成员必须使用相同的身份验证提供程序和用户定义。如果省略此选项，则连接不使用身份验证机制。请参阅[配置安全性](http://rowstore.docs.snappydata.io/docs/deploy_guide/Topics/security/security_chapter.html)。|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下，gfxd使用主机名或localhost，如果主机名指向本地环回地址。|
|-client绑定地址|RowStore定位器侦听客户端连接的主机名或IP地址。默认为“localhost”。<br/><br/>使用此选项与`-client-port`作为瘦客户端附加到RowStore集群并执行该命令。|
|-client端口|RowStore定位器侦听客户端连接的端口。默认值为1527。<br/><br/>使用此选项与`-client-bind-address`作为瘦客户机附加到RowStore集群并执行该命令。|
|-extra-CONN-道具|连接到RowStore分布式系统时要使用的分号分隔列表。|
|-help，-help|显示此snappy-shell命令的帮助信息。|
|-locators|定位器列表，以逗号分隔的主机[port]值，用于发现分布式系统的其他成员。<br/><br/>使用`-locators`创建一个对等体客户端成员来执行snappy-shell命令。|
|-mcast地址|用于发现分布式系统的其他成员的组播地址。该值仅在未指定`-locators`选项时使用。默认多播地址为239.192.81.1。<br/><br/>使用此选项与`-mcast-port`作为对等体客户端附加到RowStore集群并执行该命令。|
|-mcast端口|用于与分布式系统的其他成员进行通信的组播端口。如果为零，组播不会用于成员发现（请指定“-locators”）。仅当未指定`-locators`选项时，才使用此值。<br/><br/>有效值范围为0-65535，默认值为10334。<br/><br/>使用此选项与`-mcast-address`作为对等体客户机附加到RowStore集群并执行该命令。|
|-密码|如果服务器或定位器已配置为使用身份验证，则此选项将指定用户（使用-user选项指定）用于引导服务器并加入分布式系统的密码。<br/><br/>密码值是可选的。如果省略密码，gfxd会提示您从控制台输入密码。|
|-用户|如果服务器或定位器已配置为使用身份验证，则此选项指定用于引导服务器并加入分布式系统的用户名。|

### 描述 ###
使用命令连接到RowStore分布式系统并安装JAR文件来指定这些选项之一：

- 同时使用`-client-bind-address`并`-client-port`连接到作为瘦客户机的RowStore集群，并执行该命令。

- 同时使用`mcast-port`和`-mcast-address`，或使用-locators属性连接到RowStore簇作为对等客户机，并执行该命令。

在`-name`您提供的JAR必须是唯一的标识符。您必须包括一个模式名称才能对该标识符进行限定。您可以使用标识符以后调用**replace-jar**或**remove-jar**。

该`-file`选项指定使用本地路径或URL来安装的JAR文件的位置。

将JAR安装到分布式系统后，RowStore自动将JAR文件类加载到其类加载器中; 安装JAR后，您不需要显式加载类。JAR及其类可用于RowStore分布式系统的所有成员，包括稍后加入群集的成员。

> 注意：安装JAR文件后，不能修改JAR文件中的任何一个类或资源。相反，您必须替换整个JAR文件才能更新类。

请参阅**在RowStore中存储和加载JAR文件**。


### 例子 ###
此命令连接到在localhost：1527上运行的RowStore网络服务器，并将一个名为tours.jar的JAR文件安装到APP模式中：

	snappy-shell install-jar -name=APP.toursjar -file=c:\tours.jar
此命令连接到运行在myserver：1234上的RowStore网络服务器以安装JAR文件：

snappy-shell install-jar -name=APP.toursjar -file=c:\tours.jar 
     -client-bind-address=myserver -client-port=1234
该命令作为对等体客户端连接到组播端口1234上运行的RowStore系统，并安装JAR文件：

	snappy-shell install-jar -name=APP.toursjar -file=c:\tours.jar
	    -mcast-port=1234 -extra-conn-props=host-data=false

## 列表失踪磁盘存储 ##
列出所有磁盘存储与其他成员正在等待的最新数据。

### 句法 ###
	snappy-shell list-missing-disk-stores [-mcast-port=<port>]
	  [-mcast-address=<address>]
	  [-locators=<addresses>] [-bind-address=<address>] [-<prop-name>=<prop-value>]*
如果在命令行中未指定多播或定位器选项，则该命令将使用gemfirexd.properties文件（如果可用）来确定应连接的分布式系统。

该表描述了snappy-shell列表缺少磁盘存储的选项。

|选项|描述|
|:--|:--|
|-mcast端口|组播端口用于与分布式系统的其他成员进行通信。如果为零，组播不会用于成员发现（请指定“-locators”）。<br/><br/>有效值范围为0-65535，默认值为10334。|
|-mcast地址|组播地址用于发现分布式系统的其他成员。仅当未指定`-locators`选项时，才使用此值。<br/><br/>默认多播地址为239.192.81.1。|
|-locators|用于发现分布式系统成员的定位器列表。以逗号分隔的主机：端口值提供所有定位器。|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下，gfxd使用主机名或localhost，如果主机名指向本地环回地址。|
|- 丙名|任何其他RowStore分布式系统属性。|

### 描述 ###
处理缺少的磁盘存储提供有关列出和撤销缺少的磁盘存储的更多详细信息。


### 例 ###
	snappy-shell list-missing-disk-stores -mcast-port=10334
	Connecting to distributed system: mcast=/239.192.81.1:10334
	1f811502-f126-4ce4-9839-9549335b734d [curwen.local:/Users/yozie/snappydata/rowstore/SnappyData_RowStore_13_bNNNNN_platform/server2/./datadictionary]

## 定位器 ##
允许分布式系统中的对等体（包括其他定位器）发现对方，而无需对任何其他对等体进行硬编码。

### 句法 ###
要启动独立的定位器，请使用以下命令：

	snappy-shell rowstore locator start [-J<vmarg>]* [-dir=<workingdir>] 
                   [-classpath=<classpath>]
                   [-distributed-system-id=<id>]
                   [-heap-size=<size>]
                   [-peer-discovery-address=<addr> (default is 0.0.0.0)]
                   [-peer-discovery-port=<port> (default 10334)]
                   [-sync=<true|false> (default false)]
                   [-bind-address=<address> (default is the -peer-discover-address value)]
                   [-run-netserver=<true|false> (default true)]
                   [-client-bind-address=<addr> (default is
                     bind-address or if not set then loopback)]
                   [-client-port=<clientport> (default 1527)]
                   [-locators=<addresses>] 
                   [-log-file=<path> (default gfxdlocator.log)]
                   [-remote-locators=<host[port]>[,<host[port]>]...]
                   [-auth-provider=<provider>]
                   [-server-auth-provider=<provider>]
                   [-user=<username>] [-password[=<password>]]
                   [-<prop-name>=<prop-value>]*

> 注意： 使用`snappy-shell rowstore locator start`启动定位器时，除了使用上面列出的选项之外，您还可以使用任何其他RowStore引导属性（` -=`）在命令行。

警告：

在成员运行时，切勿移动RowStore服务器或定位器的工作目录。

警告：

不要删除或修改RowStore持久性文件，或从成员工作目录移动文件。

启动脚本将定位器的运行状态维护在文件.gfxdlocator.ser中。

显示正在运行的定位器的状态：

	snappy-shell rowstore locator status [ -dir=<workingdir> ]
要停止正在运行的定位器：

	snappy-shell rowstore locator stop [ -dir=<workingdir> ]
如果使用`-sync=false`选项（默认值）从脚本或批处理文件启动定位器，则可以使用以下命令等待定位器完成同步并完成启动：

	snappy-shell rowstore locator wait [-J<vmarg>]* [-dir=<workingdir>]
`wait`在定位器完成启动之前，该命令不会返回控制。

下表介绍了上述所有`snappy-shell rowstore locator`命令的选项。如果不指定选项，则使用默认值。

|选项|描述|
|:--|:--|
|-J|JVM选项传递给生成的RowStore定位器JVM。例如，使用-J-Xmx1024m将JVM堆设置为1GB。|
|-dir|定位器的工作目录将包含RowStore Locator状态文件，并将作为日志文件，持久文件，数据字典等的默认位置。此选项默认为当前目录。|
|-classpath|RowStore定位器所需的用户类的位置。此路径将附加到当前类路径。|
|- 分布式系统-ID|唯一标识此RowStore集群的整数。|
||在多个RowStore群集之间使用WAN复制时设置唯一的值。[为WAN成员发现配置定位器提](http://rowstore.docs.snappydata.io/docs/config_guide/topics/gateway-hubs/wan-locators.html)供更多信息。|
|-heap大小|使用RowStore默认资源管理器设置为Java VM设置固定的堆大小。如果使用该`-heap-size`选项，则默认情况下，RowStore将关键堆百分比设置为堆的80％，并将驱逐堆百分比设置为关键堆的80％。如果JVM支持，RowStore还会为驱逐和垃圾回收设置资源管理属性。<br/><br/>注意：不再支持早期SQLFire产品中使用的参数`-initial-heap`和`-max-heap`参数。使用`-heap-size`来代替。|
|-peer发现地址|定位器绑定到对等体发现的地址（包括服务器以及其他定位器）。|
|-peer发现端口|定位器侦听对等体发现的端口（包括服务器以及其他定位器）。有效值的范围为1-65535，默认值为10334。|
|-同步|确定`snappy-shell rowstore locator`如果定位器达到“等待”状态，该命令是否立即返回。如果成员依赖于另一个服务器或定位器来提供最新的磁盘存储持久性文件，则定位器或服务器在启动时达到“等待”状态。如果您启动的定位器或服务器不是分布式系统中关闭的最后一个成员，并且另一个成员存储系统的最新版本的持久数据，则可能会发生此类依赖关系。<br/><br/>指定`-sync=false`默认值）使`gfxd`成员达到“等待”状态后立即返回控制。使用`-sync=true`（服务器的默认值），`gfxd`在所有依赖成员已启动并且成员已完成同步磁盘存储之后，该命令不会返回控制。`-sync=false`在同一机器上启动多个成员时，始终使用（默认值），特别是在`gfxd`从shell脚本或批处理文件执行命令时，使脚本文件在等待特定RowStore成员启动时不挂起。您可以使用脚本中`snappy-shell rowstore locator wait`和/或`gfxd server wait`稍后的脚本验证每个服务器是否已完成同步并已达到“正在运行”状态。例如：<br/><br/>#!/bin/bash<br/><br/># Start all local RowStore members to waiting state, regardless of which member holds the most recent<br/># disk store files:<br/><br/>snappy-shell rowstore locator start -dir=/locator1 -sync=false <br/>snappy-shell rowstore server start -client-port=1528 -locators=localhost[10334] -dir=/server1 -sync=false<br/>snappy-shell rowstore server start -client-port=1529 -locators=localhost[10334] -dir=/server2 -sync=false<br/><br/># Wait until all members have finished synchronizing and starting:<br/><br/>snappy-shell rowstore locator wait -dir=/locator1<br/>snappy-shell rowstore server wait -dir=/server1<br/>snappy-shell rowstore server wait -dir=/server2<br/><br/># Continue any additional tasks that require access to the RowStore members...<br/><br/>[...]<br/><br/>作为使用的替代方法`snappy-shell rowstore locator wait`，您可以使用[MEMBERS](http://rowstore.docs.snappydata.io/docs/reference/system_tables/members.html)系统表中的STATUS列监视RowStore成员的当前状态。|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下，gfxd使用-peer-discovery-address值。|
|-run-的netserver|如果为true，则启动网络服务器（请参阅“-client-bind-address”和“-client-port”选项以指定服务器应侦听的位置），可以为瘦客户机（默认为true）提供服务。<br/><br/>如果设置为false，那么`-client-bind-address'和`-client-port`选项没有影响。此选项默认为true。|
|-client绑定地址|网络控制器绑定到客户端连接的地址。只有在`-run-netserver`选项未设置为false的情况下，这才会生效。|
|-client端口|网络控制器侦听客户端连接的端口，1-65535，默认值为1527.只有在“-run-netserver”选项未设置为false时，该端口才会生效。|
|-locators|其他定位器的列表，以逗号分隔的主机：端口值用作备份，以防其中一个定位器发生故障。该列表必须包括所有使用的其他定位器，并且必须为分布式系统的每个成员始终配置。(http://rowstore.docs.snappydata.io/docs/reference/system_tables/members.html)(http://rowstore.docs.snappydata.io/docs/reference/system_tables/members.html)必须将服务器配置为使用分布式系统的所有定位器，其中包括此定位器和列表中的所有其他定位器。|
|-remote-定位器|用于远程RowStore集群的定位器的地址和端口号的逗号分隔列表。<br/><br/>配置WAN复制时，使用此选项来标识与远程RowStore集群的连接。[为WAN成员发现配置定位器](http://rowstore.docs.snappydata.io/docs/config_guide/topics/gateway-hubs/wan-locators.html)提供更多信息。|
|-auth提供商|设置用于客户端连接的身份验证机制。有效值为BUILTIN和LDAP。如果省略`-server-auth-provider`，那么这个机制也用于加入集群。如果省略两个选项，则不使用身份验证机制。请参阅[配置安全性](http://rowstore.docs.snappydata.io/docs/deploy_guide/Topics/security/security_chapter.html)。请注意，如果您指定了客户端身份验证机制，则RowStore将默认启用SQL授权`-auth-provider`。|
|-server认证 - 供应商|用于加入集群并与集群中的其他服务器和定位器通话的认证机制。支持的值为BUILTIN和LDAP。默认情况下，如果指定了RowStore，则使用-auth-provider的值，否则不使用任何身份验证。|
|-用户|如果服务器或定位器已配置为使用身份验证，则此选项指定用于引导服务器并加入分布式系统的用户名。|
|-密码|如果服务器或定位器已配置为使用身份验证，则此选项指定用于引导服务器并加入分布式系统的用户的密码（用`-user`选项指定）。<br/><br/>密码值是可选的。如果省略密码，“snappy-shell”会提示您从控制台输入密码。|
|-log文件|此定位器写入日志消息的文件的路径。默认值是gfxdlocator.log在工作目录。|
|- <丙名称>|[任何其他RowStore引导属性，如`log-level`。例如，要作为JMX管理器启动RowStore定位器，请[使用使用Java管理扩展（JMX）](http://rowstore.docs.snappydata.io/docs/manage_guide/jmx/chapter_overview.html)中描述的引导属性。有关引导属性的完整列表，请参[阅配置属性](http://rowstore.docs.snappydata.io/docs/reference/configuration/ConnectionAttributes.html)。|

### 描述 ###
RowStore定位器允许分布式系统中的对等体（包括其他定位器）发现对方，而不必对任何其他对等体进行硬编码。另一种发现方式是使用要求对等体使用相同组播地址和端口进行发现的组播。

定位器可以单独启动，也可以嵌入到具有完整RowStore引擎的服务器中。要启动嵌入式定位器，请对RowStore服务器使用“ -start-locator ”选项。

> 注意：定位器是生产系统的推荐发现方法。运行的独立定位器为整个定位器服务提供了最高的可靠性和可用性。

由于定位器没有数据或元数据可用，因此不能从瘦客户端直接为任何与用户表相关的SQL语句提供服务。它只能服务涉及内置系统表的查询和内置的系统过程调用。客户端使用连接作为控制连接，以最少负载（以其他活动客户端连接方式衡量）查询服务器的定位器，并透明地创建由定位器返回的到服务器的实际数据连接。客户端还通过查询定位器跟踪分布式系统中的所有其他定位器和对等体。这种做法避免为同一个分布式系统创建多个控制连接。如果客户端检测到任何两个单独的用户连接引用同一分布式系统中的定位器或服务器，那么它们将使用相同的控制连接。如果控制连接本身失败，则所有定位器和对等体的列表也用于故障转移。

RowStore定位器对于负载平衡的客户端连接以及瘦客户机驱动程序提供的透明故障切换功能（高可用性或简称HA）的完全可用性是必需的。没有定位器客户端连接不能负载平衡。即使客户端尝试透明地对其他没有定位器的服务器进行故障转移，但是这种行为并不像使用定位器那样可靠，如果多个服务器快速连续失败，故障转移可能无法透明运行。

在引导RowStore分布式系统时，始终在启动数据存储之前启动定位器成员。Converseley在关闭数据存储成员后（最好关闭全部），最后关闭定位器。每个定位器本地保留所需数据字典的副本，并且可能需要等待其他定位器重新联机，以确保它具有数据字典的所有最新更新。


例
启动定位器生成如下所示的输出。XXX是当前工作目录的路径。

	Starting RowStore Locator using peer discovery on: 0.0.0.0[10334]
	Starting network server for RowStore Locator at address localhost/127.0.0.1[1527] 
	RowStore Locator pid: 3722 status:running
	Logs generated in <XXX>/gfxdlocator.log
默认情况下，`snappy-shell rowstore locator start`本地启动服务器进程，并使用当前工作目录来存储日志，定位器状态和数据字典，并使用默认TCP端口10334进行对等发现。

默认情况下，在定位器的当前工作目录中创建名为datadictionary的子目录，并且包含在分布式系统中执行的定位器启动所需的DDL的持久性元数据（例如，认证相关权限）任何客户端或对等体。此目录对于RowStore定位器启动和正常运行是必需的。

如上面的输出所示，默认情况下，网络服务器也启动，该端口绑定到端口1527上的本地主机。该服务允许瘦客户端连接到其中一个服务器，并使用DRDA协议执行SQL命令。

如果要重新启动在分布式系统中使用的定位器，则定位器可能需要等待其他成员（服务器或定位器）加入系统才能同步磁盘存储持久性文件。在这种情况下，定位器达到“等待”状态，直到从属成员加入系统才能完成启动。例如：

	Starting RowStore Locator using peer discovery on: 0.0.0.0[10334]
	Starting network server for RowStore Locator at address localhost/127.0.0.1[1527]
	Logs generated in /Users/yozie/snappydata/rowstore/RowStore_13_b48393_Linux/locator1/gfxdlocator.log
	RowStore Locator pid: 9496 status: waiting
	Region /_DDL_STMTS_META_REGION has potentially stale data. It is waiting for another member to recover the latest data.
	My persistent id:
	
	  DiskStore ID: ff28402d-4fa1-488f-ba47-7bf9dec8248e
	  Name: 
	  Location: /10.0.1.31:/Users/yozie/snappydata/rowstore/RowStore_13_b48393_Linux/locator1/./datadictionary
	
	Members with potentially new data:
	[
	  DiskStore ID: ea249383-b103-43d5-957b-f9789eadd37c
	  Name: 
	  Location: /10.0.1.31:/Users/yozie/snappydata/rowstore/RowStore_13_b48393_Linux/server2/./datadictionary
	, 
	  DiskStore ID: ff7d62c5-4e03-4c74-975f-c8d3639c1cee
	  Name: 
	  Location: /10.0.1.31:/Users/yozie/snappydata/rowstore/RowStore_13_b48393_Linux/server1/./datadictionary
	]
	Use the "snappy-shell list-missing-disk-stores" command to see all disk stores that are being waited on by other members. - See log file for details.
定位器同步持久数据，并在从属成员加入系统后完成启动。

## 日志记录支持 ##
您可以指定JDBC引导属性`snappy-shell rowstore server`和`snappy-shell rowstore locator`命令，以分别配置RowStore服务器和定位器的日志文件位置和日志严重性级别。

**配置和使用RowStore日志文件**提供更多信息。

## 上合并日志 ##
将多个日志文件合并到一个日志中。

### 句法 ###
	snappy-shell merge-logs <logFile>+ [-out=<outFile>]

### 描述 ###
该-out选项指定要在其中写入合并的日志信息的文件。如果不指定文件，则将合并的日志信息写入stdout。

### 例 ###
	snappy-shell merge-logs gfxdserver-01-00.log  gfxdserver-02-00.log   gfxdserver-03-00.log  -out=merged.log
	Completed merge of 3 logs to "merged.log".

## 打印堆栈 ##
打印RowStore成员进程的堆栈转储。

### 句法 ###
	snappy-shell print-stacks
	  [-all-threads] [<filename>] [-J-D<vmprop>=<prop-value>]
	  [-mcast-port=<port>]
	  [-mcast-address=<address>]
	  [-locators=<addresses>]
	  [-bind-address=<addr>]
	  [-<prop-name>=<prop-value>]*

该表描述了选项`snappy-shell print-stacks`。如果在命令行中未指定多播或定位器选项，则该命令将使用gemfirexd.properties文件（如果可用）来确定应连接的分布式系统。

|选项|描述|
|:--|:--|
|-所有线程|默认情况下，此命令尝试从堆栈转储中删除空闲的RowStore线程。包括`-all-threads`在转储中包括空闲线程。|
|[<文件名>]|用于存储堆栈转储的可选文件名。有关将堆栈转储信息附加到RowStore日志文件的信息，请参见[SYS.DUMP \ _STACKS](http://rowstore.docs.snappydata.io/docs/reference/system_procedures/dump-stacks.html)。|
|-mcast端口|组播端口用于与分布式系统的其他成员进行通信。如果为零，组播不会用于成员发现（请指定“-locators”）。<br/><br/>有效值范围为0-65535，默认值为10334。|
|-mcast地址|组播地址用于发现分布式系统的其他成员。仅当未指定`-locators`选项时，才使用此值。<br/>默认多播地址为239.192.81.1。|
|-locators|用于发现分布式系统成员的定位器列表。以逗号分隔的主机：端口值提供所有定位器。|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下`gfxd`，如果主机名指向本地环回地址，则使用主机名或本地主机。|
|- <丙名> = <丙值>|任何其他RowStore分布式系统属性。|

### 例 ###
以下命令将所有RowStore进程的堆栈转储打印为标准：

	$ snappy-shell print-stacks -all-threads
	
	Connecting to distributed system: mcast=/239.192.81.1:10334
	--- dump of stack for member ward(1940)<v0>:48434 ------------------------------------------------------------------------------
	GfxdLocalLockService@2916ab8[gfxd-ddl-lock-service]: PRINTSTACKS
	
	snappy-shell-ddl-lock-service:   0 tokens, 0 locks held
	__VMID_LS:   0 tokens, 0 locks held
	__PRLS:   0 tokens, 0 locks held
	
	
	TX states:
	
	Full Thread Dump:
	
	"Pooled Message Processor 1" Id=73 WAITING
	    at sun.misc.Unsafe.park(Native Method)
	    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:283)
	    at com.gemstone.java.util.concurrent.SynchronousQueueNoSpin$TransferStack.awaitFulfill(SynchronousQueueNoSpin.java:451)
	    at com.gemstone.java.util.concurrent.SynchronousQueueNoSpin$TransferStack.transfer(SynchronousQueueNoSpin.java:352)
	    at com.gemstone.java.util.concurrent.SynchronousQueueNoSpin.take(SynchronousQueueNoSpin.java:886)
	    at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:957)
	    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:917)
	    at com.gemstone.gemfire.distributed.internal.DistributionManager.runUntilShutdown(DistributionManager.java:728)
	    at com.gemstone.gemfire.distributed.internal.DistributionManager$5$1.run(DistributionManager.java:1023)
	    at java.lang.Thread.run(Thread.java:695)
	
	"Pooled High Priority Message Processor 2" Id=72 RUNNABLE
	    at sun.management.ThreadImpl.dumpThreads0(Native Method)
	    at sun.management.ThreadImpl.dumpAllThreads(ThreadImpl.java:433)
	    at com.pivotal.snappydata.internal.engine.locks.GfxdLocalLockService.generateThreadDump(GfxdLocalLockService.java:373)
	    at com.pivotal.snappydata.internal.engine.locks.GfxdLocalLockService.dumpAllRWLocks(GfxdLocalLockService.java:362)
	    at com.pivotal.snappydata.internal.engine.store.RegionEntryUtils$1.printStacks(RegionEntryUtils.java:1360)
	    at com.gemstone.gemfire.internal.OSProcess.zipStacks(OSProcess.java:482)
	    at com.gemstone.gemfire.distributed.internal.HighPriorityAckedMessage.process(HighPriorityAckedMessage.java:174)
	    at com.gemstone.gemfire.distributed.internal.DistributionMessage.scheduleAction(DistributionMessage.java:415)
	    at com.gemstone.gemfire.distributed.internal.DistributionMessage$1.run(DistributionMessage.java:483)
	    at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:895)
	    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:918)
	    at com.gemstone.gemfire.distributed.internal.DistributionManager.runUntilShutdown(DistributionManager.java:728)
	    at com.gemstone.gemfire.distributed.internal.DistributionManager$6$1.run(DistributionManager.java:1060)
	    at java.lang.Thread.run(Thread.java:695)
	
	    Number of locked synchronizers = 1
	    - java.util.concurrent.locks.ReentrantLock$NonfairSync@775c024c
	[...]
包括一个文件名参数，将堆栈转储存储在文件中，而不是写入标准输出：

	$ snappy-shell print-stacks -all-threads gfxd-stack-dump.txt
	
	Connecting to distributed system: mcast=/239.192.81.1:10334
	1 stack dumps written to gfxd-stack-dump.txt
有关将堆栈转储信息附加到RowStore日志文件的信息，请参见**SYS.DUMP_STACKS**。

## 除去-JAR ##
删除JAR文件安装，卸载与JAR关联的所有类。

句法
要删除JAR文件安装并卸载JAR文件类，请使用以下语法：

	snappy-shell remove-jar -name=<identifier>
	     [-auth-provider=<name>]
	     [-bind-address=<address>]
	     [-client-bind-address=<address>]
	     [-client-port=<port>]
	     [-extra-conn-props=<properties>] 
	     [-help] [-locators=<adresses>]
	     [-mcast-address=<address>]
	     [-mcast-port=<port>]
	     [-password[=<password>]]
	     [-user=<username>]
该表描述了snappy-shell remove-jar命令的选项。如果不指定选项，则使用默认值。

|选项|描述|
|:--|:--|
|-名称|要删除的现有JAR安装的唯一标识符。您提供的标识符必须指定模式名称分隔符。例如：APP.myjar。<br/><br/>这个参数是必需的。|
|-auth提供商|设置用于对等连接以及客户端 - 服务器连接的身份验证提供程序。有效值为BUILTIN和LDAP。RowStore分布式系统的所有其他成员必须使用相同的身份验证提供程序和用户定义。如果省略此选项，则连接不使用身份验证机制。请参阅[配置安全。](http://rowstore.docs.snappydata.io/docs/deploy_guide/Topics/security/security_chapter.html)。|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下，gfxd使用主机名或localhost，如果主机名指向本地环回地址。|
|-client绑定地址|RowStore定位器侦听客户端连接的主机名或IP地址。默认为“localhost”。<br/><br/>使用此选项与`-client-port`作为瘦客户端附加到RowStore集群并执行该命令。|
|-client端口|RowStore定位器侦听客户端连接的端口。默认值为1527。<br/><br/>使用此选项与`-client-bind-address`作为瘦客户机附加到RowStore集群并执行该命令。|
|-extra-CONN-道具|连接到RowStore分布式系统时要使用的分号分隔列表。|
|-help，-help|显示此snappy-shell命令的帮助信息。|
|-locators|定位器列表，以逗号分隔的主机[port]值，用于发现分布式系统的其他成员。<br/><br/>使用`-locators`创建一个对等体客户端成员来执行snappy-shell命令。|
|-mcast地址|用于发现分布式系统的其他成员的组播地址。该值仅在未指定`-locators`选项时使用。默认多播地址为239.192.81.1。<br/><br/>使用此选项与`-mcast-port`作为对等体客户端附加到RowStore集群并执行该命令。|
|-mcast端口|用于与分布式系统的其他成员进行通信的组播端口。如果为零，组播不会用于成员发现（请指定“-locators”）。仅当未指定`-locators`选项时，才使用此值。<br/><br/>有效值范围为0-65535，默认值为10334。<br/><br/>使用此选项与`-mcast-address`作为对等体客户机附加到RowStore集群并执行该命令。|
|-密码|如果服务器或定位器已配置为使用身份验证，则此选项将指定用户（使用-user选项指定）用于引导服务器并加入分布式系统的密码。<br/><br/>密码值是可选的。如果省略密码，gfxd会提示您从控制台输入密码。|
|-用户|如果服务器或定位器已配置为使用身份验证，则此选项指定用于引导服务器并加入分布式系统的用户名。|

### 描述 ###
指定这些选项之一，使用命令连接到RowStore分布式系统并删除JAR文件安装：

- 同时使用`-client-bind-address`并`-client-port`连接到作为瘦客户机的RowStore集群，并执行该命令。

- 同时使用`mcast-port`和`-mcast-address`，或使用-locators属性连接到RowStore簇作为对等客户机，并执行该命令。

在`-name`您提供的JAR必须是现有的JAR文件标识符（与创建**安装-JAR**）。您必须包括一个模式名称才能对该标识符进行限定。

RowStore删除JAR文件安装（和唯一标识符），并卸载JAR文件类。

请参阅**在RowStore中存储和加载JAR文件**。


### 例子 ###
此命令连接到在localhost：1527上运行的RowStore网络服务器，并删除JAR安装名称APP.toursjar：

	snappy-shell remove-jar -name=APP.toursjar
此命令连接到运行在myserver：1234上的RowStore网络服务器以删除JAR文件安装：

	snappy-shell remove-jar -name=APP.toursjar
	     -client-bind-address=myserver -client-port=1234
此命令作为对等体客户端连接到组播端口1234上运行的RowStore系统，并删除JAR文件安装：

	snappy-shell remove-jar -name=APP.toursjar
	    -mcast-port=1234 -extra-conn-props=host-data=false

## 更换-JAR ##
使用新的JAR文件的内容替换已安装的JAR文件。新JAR文件中的类将自动加载到RowStore类加载器中，并替换以前为相同JAR标识符安装的任何类。RowStore还重新编译依赖于JAR文件的对象，例如已安装的监听器实现。

### 句法 ###
要使用新的JAR文件的内容替换已安装的JAR文件，请使用以下语法：

	snappy-shell replace-jar -file=<path or URL> -name=<identifier>
	     [-auth-provider=<name>]
	     [-bind-address=<address>]
	     [-client-bind-address=<address>]
	     [-client-port=<port>]
	     [-extra-conn-props=<properties>] 
	     [-help] [-locators=<adresses>]
	     [-mcast-address=<address>]
	     [-mcast-port=<port>]
	     [-password[=<password>]]
	     [-user=<username>]
此表描述了snappy-shell replace-jar命令的选项。如果不指定选项，则使用默认值。

|选项|描述|
|:--|:--|
|-文件|新JAR文件的本地路径，或链接到JAR文件的URL。<br/><br/>这个参数是必需的。|
|-名称|要替换的现有JAR安装的唯一标识符。您提供的标识符必须指定一个模式名称分隔符。例如：APP.myjar。<br/><br/>这个参数是必需的。|
|-auth提供商|设置用于对等连接以及客户端 - 服务器连接的身份验证提供程序。有效值为BUILTIN和LDAP。RowStore分布式系统的所有其他成员必须使用相同的身份验证提供程序和用户定义。如果省略此选项，则连接不使用身份验证机制。请参阅[配置安全性](http://rowstore.docs.snappydata.io/docs/deploy_guide/Topics/security/security_chapter.html)。|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下，gfxd使用主机名或localhost，如果主机名指向本地环回地址。|
|-client绑定地址|RowStore定位器侦听客户端连接的主机名或IP地址。默认为“localhost”。<br/><br/>使用此选项与`-client-port`作为瘦客户端附加到RowStore集群并执行该命令。|
|-client端口|RowStore定位器侦听客户端连接的端口。默认值为1527。<br/><br/>使用此选项与`-client-bind-address`作为瘦客户机附加到RowStore集群并执行该命令。|
|-extra-CONN-道具|连接到RowStore分布式系统时要使用的分号分隔列表。|
|-help，-help|显示此snappy-shell命令的帮助信息。|
|-locators|定位器列表，以逗号分隔的主机[port]值，用于发现分布式系统的其他成员。<br/><br/>使用`-locators`创建一个对等体客户端成员来执行snappy-shell命令。|
|-mcast地址|用于发现分布式系统的其他成员的组播地址。该值仅在未指定`-locators`选项时使用。默认多播地址为239.192.81.1。<br/><br/>使用此选项与`-mcast-port`作为对等体客户端附加到RowStore集群并执行该命令。|
|-mcast端口|用于与分布式系统的其他成员进行通信的组播端口。如果为零，组播不会用于成员发现（请指定“-locators”）。仅当未指定`-locators`选项时，才使用此值。<br/><br/>有效值范围为0-65535，默认值为10334。<br/><br/>使用此选项与`-mcast-address`作为对等体客户机附加到RowStore集群并执行该命令。|
|-密码|如果服务器或定位器已配置为使用身份验证，则此选项将指定用户（使用-user选项指定）用于引导服务器并加入分布式系统的密码。<br/><br/>密码值是可选的。如果省略密码，gfxd会提示您从控制台输入密码。|
|-用户|如果服务器或定位器已配置为使用身份验证，则此选项指定用于引导服务器并加入分布式系统的用户名。|

### 描述 ###
使用命令指定一对选项之一连接到RowStore分布式系统并替换JAR文件安装：

- 同时使用`-client-bind-address`并`-client-port`连接到作为瘦客户机的RowStore集群，并执行该命令。

- 同时使用`mcast-port`和`-mcast-address`，或使用-locators属性连接到RowStore簇作为对等客户机，并执行该命令。

在`-name`您提供的JAR必须是现有的JAR文件标识符（与创建安装-JAR）。您必须包括一个模式名称才能对该标识符进行限定。

该`-file`选项指定用于替换现有JAR安装的新JAR文件的位置。包括文件的本地路径或URL。

> 注意：执行命令来替换JAR文件时，JAR中现有的类可能正在被使用。例如，当您选择替换包含过程实现的JAR时，可能会执行数据感知过程。在这种情况下，`snappy-shell`等待一段时间才能完成数据感知过程。如果该过程在超时期限内未完成，则会收到错误：ERROR 40XL1：在所请求的时间内无法获取锁定。如果发生这种情况，只需重试“snappy-shell replace-jar”命令。

请参阅**在RowStore中存储和加载JAR文件**。


### 例子 ###
此命令连接到在localhost：1527上运行的RowStore网络服务器，并使用名为tours2.jar的新JAR文件替换JAR安装名称APP.toursjar：

	snappy-shell replace-jar -name=APP.toursjar -file=c:\tours2.jar
此命令连接到运行在myserver：1234上的RowStore网络服务器以替换JAR文件：

	snappy-shell replace-jar -name=APP.toursjar -file=c:\tours2.jar 
	     -client-bind-address=myserver -client-port=1234
该命令作为对等体客户端连接到组播端口1234上运行的RowStore系统，并替换JAR文件：
	
	snappy-shell replace-jar -name=APP.toursjar -file=c:\tours2.jar
	    -mcast-port=1234 -extra-conn-props=host-data=false

## 撤销缺失盘店 ##
指示RowStore成员停止等待磁盘存储可用。

### 句法 ###
	snappy-shell revoke-missing-disk-store <disk-store-id>
	  [-mcast-port=<port>]
	  [-mcast-address=<address>]
	  [-locators=<addresses>] 
	        [-bind-address=<address>] 
	  [-<prop-name>=<prop-value>]*
该表描述了snappy-shell revoke-missing-disk-store的选项和参数。如果在命令行中未指定多播或定位器选项，则该命令将使用gemfirexd.properties文件（如果可用）来确定应连接的分布式系统。

|选项|描述|
|:--|:--|
|<磁盘存储-ID>|（必需）指定要撤销的磁盘存储的唯一ID。您可以从[show-disk-store-metadata](http://rowstore.docs.snappydata.io/docs/reference/store_commands/store-show-disk-store-metadata.html)的输出中获取ID]|
|-mcast端口|组播端口用于与分布式系统的其他成员进行通信。如果为零，组播不会用于成员发现（请指定“-locators”）。<br/><br/>有效值范围为0-65535，默认值为10334。|
|-mcast地址|组播地址用于发现分布式系统的其他成员。仅当未指定`-locators`选项时，才使用此值。<br/><br/>默认多播地址为239.192.81.1。|
|-locators|用于发现分布式系统成员的定位器列表。以逗号分隔的主机：端口值提供所有定位器。|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下，gfxd使用主机名或localhost，如果主机名指向本地环回地址。|
|- 丙名|任何其他RowStore分布式系统属性。|

### 描述 ###
处理缺少的磁盘存储提供有关列出和撤销缺少的磁盘存储的更多详细信息。


### 例 ###
此命令首先列出缺少的磁盘存储：

	snappy-shell list-missing-disk-stores -mcast-port=10334
	Connecting to distributed system: mcast=/239.192.81.1:10334
	1f811502-f126-4ce4-9839-9549335b734d [curwen.local:/Users/yozie/snappydata/rowstore/SnappyData_RowStore_13_bNNNNN_platform/server2/./datadictionary]
接下来，snappy-shell如果有更新的数据可用，撤销丢失的磁盘存储：

	snappy-shell revoke-missing-disk-store 1f811502-f126-4ce4-9839-9549335b734d -mcast-port=10334
	Connecting to distributed system: mcast=/239.192.81.1:10334
	revocation was successful and no disk stores are now missing
最后，snappy-shell验证没有磁盘存储丢失：

	snappy-shell list-missing-disk-stores -mcast-port=10334
	Connecting to distributed system: mcast=/239.192.81.1:10334
	The distributed system did not have any missing disk stores

## 运行 ##
连接到RowStore分布式系统，并执行SQL命令文件的内容。指定文件中的所有命令必须与交互式gfxd shell兼容。

### 句法 ###
	snappy-shell run -file=<path or URL>
	     [-auth-provider=<name>]
	     [-bind-address=<address>]
	     [-client-bind-address=<address>]
	     [-client-port=<port>]
	     [-encoding=<charset>]
	     [-extra-conn-props=<properties>] 
	     [-help] 
	     [-ignore-errors]
	     [-J-D<property=value>]
	     [-locators=<adresses>]
	     [-mcast-address=<address>]
	     [-mcast-port=<port>]
	     [-password[=<password>]]
	     [-path=<path>]
	     [-user=<username>]

该表描述了该`snappy-shell run`命令的选项。如果不指定选项，则使用默认值。

|选项|描述|
|:--|:--|
|-文件|要执行的SQL命令文件的本地路径，或链接到SQL命令文件的URL。指定文件中的所有命令必须与交互式gfxd shell兼容。<br/><br/>这个参数是必需的。|
|-auth提供商|设置用于对等连接以及客户端 - 服务器连接的身份验证提供程序。有效值为BUILTIN和LDAP。RowStore分布式系统的所有其他成员必须使用相同的身份验证提供程序和用户定义。如果省略此选项，则连接不使用身份验证机制。请参阅[配置安全性](http://rowstore.docs.snappydata.io/docs/deploy_guide/Topics/security/security_chapter.html)。|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下，gfxd使用主机名或localhost，如果主机名指向本地环回地址。|
|-client绑定地址|RowStore定位器侦听客户端连接的主机名或IP地址。默认为“localhost”。<br/><br/>使用此选项与`-client-port`作为瘦客户端附加到RowStore集群并执行该命令。|
|-client端口|RowStore定位器侦听客户端连接的端口。默认值为1527。<br/><br/>使用此选项与`-client-bind-address`作为瘦客户机附加到RowStore集群并执行该命令。|
|-encoding|SQL脚本文件（`-file`参数）的字符集编码。默认值为UTF-8。其他可能的值有：US-ASCII，ISO-8859-1，UTF-8，UTF-16BE，UTF-16LE，UTF-16。有关详细信息，请参阅[java.nio.charset.Charset](http://docs.oracle.com/javase/7/docs/api/java/nio/charset/Charset.html)参考。|
|-extra-CONN-道具|连接到RowStore分布式系统时要使用的分号分隔列表。|
|-help，-help|显示此snappy-shell命令的帮助信息。|
|-ignore-错误|包括此选项以忽略在执行文件中的语句时可能发生的任何错误，并继续执行剩余的语句。如果省略此选项，则如果发生异常，则gfxd会立即终止脚本的执行。|
|-JD <属性=值>|将Java系统属性设置为指定的值。|
|-locators|定位器列表，以逗号分隔的主机[port]值，用于发现分布式系统的其他成员。<br/><br/>使用`-locators`创建一个对等体客户端成员来执行snappy-shell命令。|
|-mcast地址|用于发现分布式系统的其他成员的组播地址。该值仅在未指定`-locators`选项时使用。默认多播地址为239.192.81.1。<br/><br/>使用此选项与`-mcast-port`作为对等体客户端附加到RowStore集群并执行该命令。|
|-mcast端口|用于与分布式系统的其他成员进行通信的组播端口。如果为零，组播不会用于成员发现（请指定“-locators”）。仅当未指定`-locators`选项时，才使用此值。<br/><br/>有效值范围为0-65535，默认值为10334。<br/><br/>使用此选项与`-mcast-address`作为对等体客户机附加到RowStore集群并执行该命令。|
|-密码|如果服务器或定位器已配置为使用身份验证，则此选项将指定用户（使用-user选项指定）用于引导服务器并加入分布式系统的密码。<br/><br/>密码值是可选的。如果省略密码，gfxd会提示您从控制台输入密码。|
|-路径|配置脚本中执行的任何其他SQL命令文件的工作目录。该`-path`条目是在执行脚本在[运行](http://rowstore.docs.snappydata.io/docs/reference/store_commands/run.html)命令中执行的任何SQL脚本文件名前面的。|
|-用户|如果服务器或定位器已配置为使用身份验证，则此选项指定用于引导服务器并加入分布式系统的用户名。|

### 描述 ###

使用命令连接到RowStore分布式系统并执行SQL命令文件来指定这些选项之一：

- 同时使用`-client-bind-address`并`-client-port`连接到作为瘦客户机的RowStore集群，并执行该命令。

- 同时使用`mcast-port`和`-mcast-address`，或使用`-locators`属性连接到RowStore簇作为对等客户机，并执行该命令。

该`-file`参数指定要执行的SQL脚本文件的位置。如果脚本文件本身调用其他脚本文件`run 'filename'`，还要考虑使用该`-path`选项来指定嵌入脚本文件的位置。如果在执行脚本时发生异常，GFXD将立即停止执行脚本命令，除非您包含该`-ignore-errors`选项。


### 例子 ###
此命令连接到在myserver上运行的RowStore网络服务器：1234，并在ToursDB_schema.sql文件中执行命令：

	snappy-shell run -file=c:\gemfirexd\quickstart\ToursDB_schema.sql
	     -client-bind-address=myserver -client-port=1234
此命令执行loadTables.sql脚本，该脚本调用依赖脚本，如loadCOUNTRIES.sql，loadCITIES.sql等等。如果在依赖脚本所在的目录之外执行此命令，则必须使用该`-path`选项指定工作目录。例如：

	snappy-shell run -file=c:\gemfirexd\quickstart\loadTables.sql
	     -path=c:\gemfirexd\quickstart
	     -client-bind-address=myserver -client-port=1234

## 服务器 ##
RowStore服务器是RowStore系统中的主要服务器端组件，可提供与群集中其他服务器，对等体和客户端的连接。它可以托管数据。使用gfxd启动器的服务器实用程序启动服务器。

### 句法 ###
- **句法**

- **描述**

- **示例：服务器和客户端使用定位器**

- **示例：高可用性的多个定位器**

- **示例：具有定位器的服务器和访问器**

- **示例：使用BUILTIN身份验证的服务器和定位器**

- **示例：具有服务器组和定位器的服务器和访问器**

要启动RowStore服务器：

	snappy-shell rowstore server start [-J<vmarg>]* [-dir=<workingdir>] [-classpath=<classpath>]
	                  [-sync=<false|true> (default false)]
	                  [-heap-size=<size>] [-off-heap-size=<size>]
	                  [-mcast-port=<port> (default 10334)]
	                  [-mcast-address=<address> (default 239.192.81.1)]
	                  [-locators=<addresses>] [-start-locator=<address>]
	                  [-server-groups=<groups>] [-lock-memory]
	                  [-rebalance] [-init-scripts=<sql-files>]
	                  [-run-netserver=<true|false> (default true)]
	                  [-bind-address=<address> (default is hostname or localhost 
	                    if hostname points to a local loopback address)]
	                  [-client-bind-address=<clientaddr> (default is localhost)]
	                  [-client-port=<clientport> (default 1527)]
	                  [-critical-heap-percentage=<percentage>
	                    (default 90% if -heap-size is provided, otherwise 
	                     not configured)]
	                  [-eviction-heap-percentage=<eviction-heap-percentage>
	                    (default 80% of critical-heap-percentage)]
	                  [-critical-off-heap-percentage=<critical-off-heap-percentage>
	                    (default 90% if -off-heap-size is provided, otherwise not
	                     configured)]                     
	                  [-eviction-off-heap-percentage=<eviction-off-heap-percentage>
	                    (default 80% of critical-off-heap-percentage)]
	                  [-host-data=<true|false> (default true)]
	                  [-log-file=<path> (default gfxdserver.log)]
	                  [-auth-provider=<provider>]
	                  [-server-auth-provider=<provider>]
	                  [-user=<username>] [-password[=<password>]]
	                  [-<prop-name>=<prop-value>]*
警告：

在成员运行时，切勿移动RowStore服务器或定位器的工作目录。

警告：

不要删除或修改RowStore持久性文件，或从成员工作目录移动文件。

显示正在运行的服务器的状态：

	snappy-shell rowstore server status [ -dir=<workingdir> ]
要停止正在运行的服务器：

	snappy-shell rowstore server stop [ -dir=<workingdir> ]
如果使用`-sync=false`选项（默认值）从脚本或批处理文件启动服务器，则可以使用以下命令等待服务器完成同步并完成启动：

	snappy-shell rowstore server wait [-J<vmarg>]* [-dir=<workingdir>]
wait在定位器完成启动之前，该命令不会返回控制。

要停止系统中所有正在运行的访问者和数据存储成员，请在服务器工作目录中使用以下命令：

	snappy-shell shut-down-all
该表描述了snappy-shell rowstore服务器命令的选项。如果不指定选项，则使用默认值。


|:--|:--|
|-J|JVM选项传递给生成的RowStore服务器JVM。例如，使用-J-Xmx1024m将JVM堆设置为1GB。|
|-dir|将包含RowStore Server状态文件的服务器的工作目录，将作为日志文件，持久文件，数据字典等的默认位置（默认为当前目录）。|
|-classpath|RowStore Server所需的用户类的位置。此路径将附加到当前类路径。|
|-heap大小|使用RowStore默认资源管理器设置设置Java VM的堆大小。例如，`-heap-size=1024m`。如果使用该`-heap-size`选项，则默认情况下，RowStore将critical-heap-percentage设置为堆大小的90％，并将“migrate-heap-percentage”设置为“critical-heap-percentage”的80％（使用默认值时对应于到堆大小的72％）。如果JVM支持，RowStore还会为驱逐和垃圾回收设置资源管理属性。<br/><br/>注意：不再支持早期SQLFire产品中使用的参数`-initial-heap`和`-max-heap`参数。使用`-heap-size`来代替。|
|-off-堆大小|指定在服务器上分配的非堆内存量。您可以指定以字节（b），千字节（k），兆字节（m）或千兆字节（g）为单位的数量。例如，`-off-heap-size=2g`。|
|-server团|此成员所属的服务器组的逗号分隔列表。用于在特定服务器集中创建表，或用于在特定服务器组中启动数据感知过程。请参见CREATE TABLE和CALL。如果未指定此选项，则服务器仅属于默认服务器组。默认服务器组没有名称，并且包含分布式系统的所有成员。|
||注意：在SYS.MEMBERS表中存储值之前，RowStore将服务器组名转换为全大写字母。DDL语句和过程自动将所提供的服务器组值转换为全大写字母。但是，当您直接查询SYS.MEMBERS表时，必须指定服务器组的大写值。|
|-lock记忆|包括此选项将RowStore堆和非堆内存页锁定到RAM中。这样可以防止操作系统将页面交换到磁盘，从而导致服务器性能下降。作为最佳做法，Pivotal建议您在同一台机器上运行RowStore和Hadoop时设置此选项。<br/><br/>注意：使用此命令时，还可以配置锁定内存的操作系统限制，如支持的[配置和系统要求中所述](http://rowstore.docs.snappydata.io/docs/getting_started/topics/system_requirements.html)。|
|-run-的netserver|如果为真，启动可以为瘦客户机提供服务的网络服务器。请参阅`-client-bind-address`和`-client-port`选项来指定服务器应该在哪里听。默认值为true。<br/>如果设置为false，则“-client-bind-address”和“-client-port”选项不起作用。|
|-同步|确定`snappy-shell rowstore server`如果服务器达到“等待”状态，该命令是否立即返回。如果成员依赖于另一个服务器或定位器来提供最新的磁盘存储持久性文件，则定位器或服务器在启动时达到“等待”状态。如果您启动的定位器或服务器不是分布式系统中关闭的最后一个成员，并且另一个成员存储系统的最新版本的持久数据，则可能会发生此类依赖关系。<br/><br/>指定`-sync=false`（默认值）使`gfxd`成员达到“等待”状态后立即返回控制。使用`-sync=true`该`gfxd`命令，直到所有从属成员启动并且成员已经完成同步磁盘存储之后才返回控制。`-sync=false`在同一机器上启动多个成员时，始终使用（默认值），特别是在`gfxd`从shell脚本或批处理文件执行命令时，使脚本文件在等待特定RowStore成员启动时不挂起。您可以使用脚本中`snappy-shell rowstore locator wait`和/或`snappy-shell rowstore server wait`稍后的脚本验证每个服务器是否已完成同步并已达到“正在运行”状态。例如：<br/><br/>#!/bin/bash<br/><br/># Start all local RowStore members to waiting state, regardless of which member holds the most recent<br/># disk store files:<br/><br/>snappy-shell rowstore locator start -dir=/locator1 -sync=false<br/>snappy-shell rowstore server start -client-port=1528 -locators=localhost[10334] -dir=/server1 -sync=false<br/>snappy-shell rowstore server start -client-port=1529 -locators=localhost[10334] -dir=/server2 -sync=false<br/><br/># Wait until all members have finished synchronizing and starting:<br/><br/>snappy-shell rowstore locator wait -dir=/locator1<br/>snappy-shell rowstore server wait -dir=/server1<br/>snappy-shell rowstore server wait -dir=/server2<br/><br/># Continue any additional tasks that require access to the RowStore members...<br/><br/>[...]<br/><br/>作为使用的替代方法`snappy-shell rowstore server wait`，您可以使用[MEMBERS](http://rowstore.docs.snappydata.io/docs/reference/system_tables/members.html)系统表中的STATUS列监视RowStore成员的当前状态。|
|-rebalance|导致新成员触发系统中所有分区表的重新平衡操作。系统始终尝试满足新成员启动时所有分区表的冗余，而不管此选项。|
|-init的脚本|指定以逗号分隔的文件列表，其中包含要以与RowStore命令shell所需的格式相同的格式执行的初始SQL命令。这些命令是在从该成员或现有成员的持久数据完成初始DDL重播后执行的。这会在执行脚本之前将元数据与集群保持一致状态。<br/><br/>在脚本文件中，可以包含需要存在表和其他对象的命令。例如，您可能会包含将初始数据加载到现有表中的DML语句。|
|-client地址|该对等体绑定以接收对等消息的地址。默认情况下，gfxd使用主机名或localhost，如果主机名指向本地环回地址。|
|-client绑定地址|网络控制器绑定到客户端连接的地址。仅当“ -run-netserver ”选项未设置为false时，此操作才会生效。|
|-client端口|网络控制器侦听客户端连接的端口，1-65535，默认值为1527.只有在“ -run-netserver ”选项未设置为false时，该端口才会生效。|
|- 临界堆百分比|将资源管理器的关键堆阈值设置为0-100的旧代堆的百分比。如果设置`-heap-size`，默认值`critical-heap-percentage`设置为堆大小的90％。使用此开关来覆盖默认值。<br/><br/>当此限制被破坏时，系统将开始取消内存密集型查询，为新的SQL语句引发低内存异常等，以避免内存不足。|
|-eviction堆百分比|设置资源管理器将用于开始从堆中逐出数据的内存使用百分比阈值（0-100）。默认情况下，驱逐阈值是设置的80％`-critical-heap-percentage`。使用此开关来覆盖默认值。|
|- 临界-离堆百分比|以0-100的百分比设置非堆内存使用的临界阈值。当此限制被破坏时，系统将开始取消内存密集型查询，为新的SQL语句引发低内存异常等等，以避免用尽非堆内存。|
|-eviction，离堆百分比|设置资源管理器将用于开始从堆内存中逐出数据的非堆内存使用率百分比阈值0-100。默认情况下，驱逐阈值是设置的80％`-critical-off-heap-percentage`。使用此开关来覆盖默认值。|
|-locators|定位器列表，以逗号分隔的主机：端口值用于与系统中的运行定位器通信，从而发现分布式系统的其他对等体。该列表必须包括所有正在使用的定位器，并且必须为分布式系统的每个成员始终配置。缺省情况下，没有定位器，所以系统对于snappydata 不支持的对等体发现使用组播。对于生产系统，强烈建议使用定位器。|
|- 启动定位器|指定此服务器中的嵌入式定位器，该服务器将使用此服务器自动启动和停止。定位器被指定为<address> [<port>]（注意方括号），或者只是<port>。当未指定地址时，如果设置了“ -bind-address ”中指定的地址。否则将使用机器的默认地址。|
|-host数据|如果设置为false，则该对等体不会托管表数据并充当访问者成员，仍然具有将查询路由到适当的数据存储并聚合结果的功能。|
|-auth提供商|客户端连接的认证机制。如果没有指定`-server-auth-provider`，那么同样的机制也用于加入集群。支持的值为BUILTIN和LDAP。请注意，如果您指定了客户端身份验证机制，则RowStore将默认启用SQL授权`-auth-provider`。缺省情况下，不使用认证。|
|-server认证 - 供应商|用于加入集群并与集群中的其他服务器和定位器通话的认证机制。支持的值为BUILTIN和LDAP。默认情况下，如果指定了RowStore，则使用-auth-provider的值，否则不使用任何身份验证。|
|-用户|如果服务器或定位器已配置为使用身份验证，则此选项指定用于引导服务器并加入分布式系统的用户名。|
|-密码|如果服务器或定位器配置为进行身份验证，则此选项指定用于引导服务器并加入分布式系统的用户的密码（用-user选项指定）。<br/><br/>密码值是可选的。如果省略密码，“snappy-shell”会提示您从控制台输入密码。|
|-log文件|到这个成员写入日志消息（默认的文件的路径gfxdserver.log在工作目录）|
|- <丙名称>|任何其他RowStore引导属性，如“ log-level ”。例如，要将RowStore服务器作为JMX管理器启动，请[使用使用Java管理扩展（JMX）](http://rowstore.docs.snappydata.io/docs/manage_guide/jmx/chapter_overview.html)中描述的引导属性。有关[引导属性](http://rowstore.docs.snappydata.io/docs/reference/configuration/ConnectionAttributes.html)的完整列表，请参阅**配置属性**。|

### 描述 ###
您可以启动可以托管数据（数据存储）或不使用实用程序托管数据（访问器）的服务器`snappy-shell`，但是任何一种成员都可以通过将所有SQL语句路由到适当的数据存储并聚合结果来服务所有SQL语句。即使对于承载表的数据的成员，也不需要在同一个成员中提供所有数据，例如对于引用分区表（**PARTITION BY Clause**）的DML 。因此可能需要向其他商店进行路由。此外，由于没有常见的`SERVER组件条款`，成员可能是数据存储，但仍然不会托管表的任何数据。

启动服务器生成类似于以下内容的输出（XXX是当前工作目录的路径）：

	 Starting RowStore Server using multicast for peer discovery: 239.192.81.1[10334]
	Starting network server for RowStore Server at address localhost/127.0.0.1[1527]
	RowStore Server pid: 2015 status: running
	Logs generated in <XXX>/gfxdserver.log
这将在本地启动服务器进程，并使用当前工作目录来存储日志，统计信息和数据字典，并使用多播进行对等体发现（地址239.192.81.1，端口10334）。使用默认磁盘存储创建的任何永久性表也将使用此目录来管理数据文件。

数据字典在名为“ datadictionary ”的子目录中进行管理，并在默认情况下保持。该子目录包含来自任何客户机或对等体的在分布式系统中执行的所有DDL的持久元数据，并且对于RowStore服务器或定位器成员来说，启动和正常运行是必需的。

如上面的输出所示，网络服务器也默认启动，该端口绑定到端口1527上的本地主机。该服务允许瘦客户端连接到服务器并使用**DRDA协议**执行SQL命令。

您正在启动的RowStore服务器可能需要其他群集成员（定位器或服务器）引导才能确认其数据与这些成员的数据一致。即使没有持久或溢出表，每个服务器本地仍然存在数据字典的副本，并且可能保持在“等待”状态，直到从属定位器或服务器重新联机，以确保它具有数据字典的所有最新更新：

	Starting RowStore Server using locators for peer discovery: localhost[10334]
	Starting network server for RowStore Server at address localhost/127.0.0.1[1528]
	Logs generated in /Users/yozie/snappydata/rowstore/RowStore_13_b48393_Linux/server1/gfxdserver.log
	RowStore Server pid: 9502 status: waiting
	Region /_DDL_STMTS_META_REGION has potentially stale data. It is waiting for another member to recover the latest data.
	My persistent id:
	
	  DiskStore ID: ff7d62c5-4e03-4c74-975f-c8d3639c1cee
	  Name: 
	  Location: /10.0.1.31:/Users/yozie/snappydata/rowstore/RowStore_13_b48393_Linux/server1/./datadictionary
	
	Members with potentially new data:
	[
	  DiskStore ID: ea249383-b103-43d5-957b-f9789eadd37c
	  Name: 
	  Location: /10.0.1.31:/Users/yozie/snappydata/rowstore/RowStore_13_b48393_Linux/server2/./datadictionary
	]
	Use the "snappy-shell list-missing-disk-stores" command to see all disk stores that are being waited on by other members. - See log file for details.
使用`-sync=false`时开始RowStore成员在shell脚本或批处理文件将控制返回给脚本的部件到达“等待”状态之后。

如果另一个服务器现在使用相同的组播端口（默认端口与上面的第一台服务器）一起加入集群，则启动消息将显示分布式系统中的其他成员：

	 snappy-shell rowstore server start -dir=server2
	-client-port=1528
	Starting RowStore Server using multicast for
	peer discovery: 239.192.81.1[10334]
	Starting network server for RowStore Server at
	address localhost/127.0.0.1[1528]
	 Distributed system now has 2 members.
	 Other members: serv1(2015:data store)<v0>:32013/48225
	RowStore Server pid: 2032 status: running
	Logs generated in <XXX>/gfxdserver.log
斜体表示分布式系统中其他成员的ID的输出。

启动脚本将服务器的运行状态维护在文件.gfxdserver.ser中。

示例：使用默认多播端口进行对等发现的多个服务器

	-- start a server using default mcast-port (10334) for discovery,
	-- with current directory as the working directory, and network server
	–- running on default localhost:1527
	snappy-shell rowstore server start
	–- start a second server talking to the first with dir2 as working directory;
	–- network server is started explicitly on localhost:1528
	snappy-shell rowstore server start -dir=dir2 -client-port=1528
	–- start another server talking to the first two with dir3 as working
	–- directory; network server is disabled explicitly
	snappy-shell rowstore server start -dir=dir3 -run-netserver=false
	–- check from the RowStore command shell
	snappy-shell
	snappy> connect client 'localhost:1527';
	snappy> select ID from sys.members;
	–- output will show three members in the distributed system
	
	–- stop everything
	snappy> quit;
	snappy-shell rowstore server stop -dir=dir3
	snappy-shell rowstore server stop -dir=dir2
	snappy-shell rowstore server stop
示例：服务器和客户端使用定位器

	–- start a locator for peer discovery on port 3241
	-- listening on all addresses
	snappy-shell rowstore locator start -peer-discovery-port=3241
	–- start three servers as before using different client ports 
	-- and using the above started locator
	snappy-shell rowstore server start -dir=dir1 -locators=localhost:3241 
	     -client-port=1528
	snappy-shell rowstore server start -dir=dir2 -locators=localhost:3241 
	     -client-port=1529
	snappy-shell rowstore server start -dir=dir3 -locators=localhost:3241 
	     -client-port=1530
	–- check from the RowStore command shell
	–- connect using the locator's client-port (default 1527) 
	-- for load balanced connection to one of the servers 
	-- transparently and for reliable failover in case the 
	-- server goes down
	snappy-shell
	snappy> connect client 'localhost:1527';
	snappy> select ID, KIND from sys.members;
	–- output will show four members with three as data stores 
	-- and one as locator
	
	–- stop everything
	snappy> quit;
	snappy-shell rowstore server stop -dir=dir3
	snappy-shell rowstore server stop -dir=dir2
	snappy-shell rowstore server stop -dir=dir1
	snappy-shell rowstore locator stop
示例：高可用性的多个定位器

	–- start two locators that configured to talk 
	-- to one another
	snappy-shell rowstore locator start -peer-discovery-port=3241 
	     -locators=localhost:3242
	snappy-shell rowstore locator start -dir=loc2 -peer-discovery-port=3242 
	     -locators=localhost:3241 -client-port=1528
	–- start four servers that can talk to both 
	-- the above locators
	snappy-shell rowstore server start -dir=dir1 
	     -locators=localhost:3241,localhost:3242 
	     -client-port=1529
	snappy-shell rowstore server start -dir=dir2 
	     -locators=localhost:3241,localhost:3242 
	     -client-port=1530
	snappy-shell rowstore server start -dir=dir3 
	     -locators=localhost:3241,localhost:3242 
	     -client-port=1531
	snappy-shell rowstore server start -dir=dir4 
	     -locators=localhost:3241,localhost:3242 
	     -client-port=1532
	–- check all the members in the distributed system
	snappy-shell
	snappy> connect client 'localhost:1527';
	snappy> select ID, KIND from sys.members order by KIND DESC;
	-– output will show six members with two locators 
	-- followed by four data stores
	snappy> quit;
	
	–- now bring down the first locator and check that 
	-- new servers can still join
	snappy-shell rowstore locator stop
	snappy-shell rowstore server start -dir=dir5 
	     -locators=localhost:3241,localhost:3242 
	     -client-port=1533
	–- check all the members in the distributed system again
	snappy-shell
	snappy> connect client 'localhost:1528';
	snappy> select ID, KIND from sys.members order by KIND DESC;
	–- output will show six members with one locator 
	-- followed by five data stores
	
	–- stop everything
	snappy> quit;
	snappy-shell rowstore server stop -dir=dir5
	snappy-shell rowstore server stop -dir=dir4
	snappy-shell rowstore server stop -dir=dir3
	snappy-shell rowstore server stop -dir=dir2
	snappy-shell rowstore server stop -dir=dir1
	snappy-shell rowstore locator stop -dir=loc2
示例：具有定位器的服务器和访问器

	–- start a locator for peer discovery on port 3241
	snappy-shell rowstore locator start -peer-discovery-port=3241
	–- start three servers using different client ports and 
	-- using the above started locator
	snappy-shell rowstore server start -dir=dir1 -locators=localhost:3241 -client-port=1528
	snappy-shell rowstore server start -dir=dir2 -locators=localhost:3241 -client-port=1529
	snappy-shell rowstore server start -dir=dir3 -locators=localhost:3241 -client-port=1530
	–- start a couple of peers that will not host data
	snappy-shell rowstore server start -dir=dir4 -locators=localhost:3241 -client-port=1531 -host-data=false
	snappy-shell rowstore server start -dir=dir5 -locators=localhost:3241 -client-port=1532 -host-data=false
	–- check from the RowStore command shell
	–- connect using the locator's client-port (default 1527) 
	-- for load balanced connection to one of the servers/accessors 
	-- transparently and for reliable failover in case the 
	-- server/accessor goes down
	snappy-shell
	snappy> connect client 'localhost:1527';
	snappy> select ID, KIND from sys.members;
	–- output will show six members with one as locator, three 
	-- data stores and two accessors
示例：使用BUILTIN身份验证的服务器和定位器

	–- start a locator for peer discovery on port 3241 
	-- with authentication; below specifies a builtin 
	-- system user on the command-line itself
	-- (snappydata.user.gem1=gem1) but it is recommended 
	-- to be in gemfirexd.properties in encrypted form 
	-- using the "snappy-shell encrypt-password" tool
	snappy-shell rowstore locator start -peer-discovery-port=3241 -auth-provider=BUILTIN  -snappydata.user.gem1=gem1 -user=gem1 -password=gem1
	–- start three servers using different client 
	-- ports and using the above started locator
	snappy-shell rowstore server start -dir=dir1 -locators=localhost:3241 -client-port=1528 -auth-provider=BUILTIN  -snappydata.user.gem1=gem1 -user=gem1 -password=gem1
	snappy-shell rowstore server start -dir=dir2 -locators=localhost:3241 -client-port=1529 -auth-provider=BUILTIN  -snappydata.user.gem1=gem1 -user=gem1 -password=gem1
	snappy-shell rowstore server start -dir=dir3 -locators=localhost:3241 -client-port=1530 -auth-provider=BUILTIN  -snappydata.user.gem1=gem1 -user=gem1 -password=gem1
	–- check from the RowStore command shell
	–- connect using the locator's client-port 
	-- (default 1527) for load balanced
	–- connection to one of the servers/accessors 
	-- transparently and for reliable
	-- failover in case the server/accessor goes down
	snappy-shell
	snappy> connect client 'localhost:1527;user=gem1;password=gem1';
	–- add a new database user
	snappy> call sys.create_user('snappydata.user.gem2', 'gem2');
	–- check members
	snappy> select ID, KIND from sys.members;
	–- output will show four members with one as 
	-- locator and three data stores
示例：具有服务器组和定位器的服务器和访问器

	–- start a locator for peer discovery on port 3241
	snappy-shell rowstore locator start -peer-discovery-port=3241
	
	–- start three servers using different client 
	-- ports and using the above started locator
	-- using two server groups (ordersdb and 
	-- customers) with one server in both
	-- the groups
	
	snappy-shell rowstore server start -dir=dir1 -locators=localhost:3241 -client-port=1528 -server-groups=ordersdb
	snappy-shell rowstore server start -dir=dir2 -locators=localhost:3241 -client-port=1529 -server-groups=customers
	snappy-shell rowstore server start -dir=dir3 -locators=localhost:3241 -client-port=1530 -server-groups=ordersdb,customers
	
	–- start a couple of peers that will 
	-- not host data but in both server 
	-- groups; using server groups in 
	-- accessors is only useful if 
	-- executing data-aware procedures 
	-- targeted to those members
	snappy-shell rowstore server start -dir=dir4 -locators=localhost:3241 -client-port=1531 -host-data=false -server-groups=ordersdb,customers
	snappy-shell rowstore server start -dir=dir5 -locators=localhost:3241 -client-port=1532 -host-data=false -server-groups=ordersdb,customers
	
	–- check from the RowStore command shell
	–- connect using the locator's client-port 
	-- (default 1527) for load balanced
	–- connection to one of the servers/accessors 
	-- transparently and for reliable
	-- failover in case the server/accessor 
	-- goes down
	snappy-shell
	snappy> connect client 'localhost:1527';
	snappy> select ID, KIND from sys.members;
	–- example output
	ID                             |KIND
	-----------------------------------------------
	pc29(23372)<v0>:28185/36245    |locator(normal) 
	
	pc29(23742)<v3>:19653/33509    |datastore(normal)
	pc29(23880)<v4>:37719/51114    |accessor(normal)
	
	pc29(23611)<v2>:52031/53106    |datastore(normal)
	pc29(24021)<v5>:58510/43678    |accessor(normal)
	
	pc29(23503)<v1>:30307/36105    |datastore(normal)
	
	
	6 rows selected
	
	
	-- also check for server groups
	snappy> select ID, KIND, SERVERGROUPS from sys.members;
	-- example output
	ID                             |KIND              |SERVERGROUPS
	-----------------------------------------------------------------
	pc29(23372)<v0>:28185/36245    |locator(normal)   |
	pc29(23742)<v3>:19653/33509    |datastore(normal) |CUSTOMERS,ORDERSDB
	pc29(23880)<v4>:37719/51114    |accessor(normal)
	  |CUSTOMERS,ORDERSDB
	pc29(23611)<v2>:52031/53106    |datastore(normal) |CUSTOMERS
	pc29(24021)<v5>:58510/43678    |accessor(normal)
	  |CUSTOMERS,ORDERSDB
	pc29(23503)<v1>:30307/36105    |datastore(normal) |ORDERSDB
	
	
	6 rows selected

## 显示盘店的元数据 ##
显示指定磁盘存储目录的磁盘存储元数据。

### 句法 ###
	snappy-shell show-disk-store-metadata <disk-store> <directory>+
	  [-mcast-port=<port>]
	  [-mcast-address=<address>]
	  [-locators=<addresses>] 
	        [-bind-address=<address>] 
	  [-<prop-name>=<prop-value>]*
该表描述了gfxd show-disk-store-metadata的选项和参数。如果在命令行中未指定多播或定位器选项，则该命令将使用gemfirexd.properties文件（如果可用）来确定应连接的分布式系统。

|选项|描述|
|:--|:--|
|<磁盘店内>|（必需）指定要显示元数据的磁盘存储的名称。磁盘存储必须脱机。|
|<目录>|（必需）指定一个或多个磁盘存储文件目录。|
|-mcast端口|组播端口用于与分布式系统的其他成员进行通信。如果为零，组播不会用于成员发现（请指定“-locators”）。<br/><br/>有效值范围为0-65535，默认值为10334。|
|-mcast地址|组播地址用于发现分布式系统的其他成员。仅当未指定`-locators`选项时，才使用此值。<br/><br/>默认多播地址为239.192.81.1。|
|-locators|用于发现分布式系统成员的定位器列表。以逗号分隔的主机：端口值提供所有定位器。|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下，gfxd使用主机名或localhost，如果主机名指向本地环回地址。|
|- 丙名|任何其他RowStore分布式系统属性。|

描述
此命令显示指定目录中所有可用磁盘存储的元数据信息。在Linux RPM安装中，磁盘存储目录和文件由非交互式`gemfirexd`用户拥有。您必须使用它`sudo -u gemfirexd`来执行磁盘存储命令。


### 例 ###
此命令显示单个目录中所有磁盘存储的元数据：

	sudo -u gemfirexd gfxd show-disk-store-metadata GFXD-DEFAULT-DISKSTORE /var/opt/snappydata/rowstore/server
	Disk Store ID: cb70441cf00f43f7-93b3cc5fe1ecd2d9
	Regions in the disk store:
	  /__UUID_PERSIST
	    lru=lru-entry-count
	    lruAction=overflow-to-disk
	    lruLimit=100
	    concurrencyLevel=16
	    initialCapacity=16
	    loadFactor=0.75
	    statisticsEnabled=false
	    drId=9
	    isBucket=false
	    clearEntryId=0
	    MyInitializingID=<null>
	    MyPersistentID=</192.168.125.137:/var/opt/snappydata/rowstore/server/. created at timestamp 1360363313694 version 0 diskStoreId cb70441cf00f43f7-93b3cc5fe1ecd2d9 name null>
	    onlineMembers:
	    offlineMembers:
	    equalsMembers:

## 关机，所有(shut-down-all) ##
指示所有RowStore访问器和数据存储成员断开与分布式系统的连接并关闭。

### 句法 ###
	snappy-shell shut-down-all 
	  [-bind-address=<address>]
	  [-locators=<addresses>] 
	  [-mcast-port=<port>] [-mcast-address=<address>]
	  [-skip-accessors] [-<prop-name>=<prop-value>]*
该表显示gfxd关闭的参数和选项。

|选项|描述|
|:--|:--|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下，gfxd使用主机名或localhost，如果主机名指向本地环回地址。|
|-locators|用于发现分布式系统成员的定位器列表。以逗号分隔的主机：端口值提供所有定位器。|
|-mcast地址|组播地址用于发现分布式系统的其他成员。仅当未指定`-locators`选项时，才使用此值。<br/><br/>默认多播地址为239.192.81.1。|
|-mcast端口|组播端口用于与分布式系统的其他成员进行通信。如果为零，组播不会用于成员发现（请指定“-locators”）。<br/><br/>有效值范围为0-65535，默认值为10334。|
|-skip-存取|包含此选项可以`host-data=false`在`shut-down-all`执行命令后运行RowStore访问器成员（已启动的成员）。|
|- 丙名|任何其他RowStore分布式系统属性。|

如果在命令行中未指定多播或定位器选项，则该命令将使用gemfirexd.properties文件（如果可用）来确定应连接的分布式系统。


### 描述 ###
此命令为所有RowStore数据存储和访问器成员提供有序关闭。定位器保持运行，以确保系统有序关闭; 在数据存储停止后才关闭定位器。在下一次启动时，数据存储成员仍然需要等待具有最新的持久数据副本的成员加入分布式系统。


### 例 ###
此命令将关闭使用多播进行发现的系统中的数据存储和访问器成员：

	snappy-shell shut-down-all -mcast-port=3000
	Connecting to distributed system: mcast=/239.192.81.1:3000
	Successfully shut down 2 members

## 统计 ##
显示统计档案中的统计值。

### 句法 ###
- **句法**

- **描述**

- **示例1：启动统计信息归档的服务器**

- **示例2：查看统计档案文件**

- **示例3：查看指定实例ID的所有统计信息**

- **示例4：查看指定实例ID和统计ID的所有统计信息**

- **示例5：查看指定实例ID和统计ID的统计信息**


		snappy-shell stats ([<instanceId>][:<typeId>][.<statId>])* [-details] [-nofilter|-persec|-persample] [-prunezeros]
		     [-starttime=<time>] [-endtime=<time>] -archive=<statFile>

### 描述 ###
要通过归档启用统计生成，请在启动RowStore服务器时配置以下属性：

- statistic-sampling-enabled=true
- statistic-archive-file=&lt;stats-archive-file-name&gt;

> 注意：您必须包含`statistic-archive-file`并指定有效的文件名才能启用统计信息收集。

要包括enable-time-statistics=true与上述属性一起设置的时间统计信息。请参阅**配置属性**。

默认情况下`snappy-shell stats`打印所有统计信息。您可以使用一个或多个统计规范参数打印单个资源或特定统计信息。统计规范的格式是：可选的组合运算符，可选的instanceId，可选的typeId和可选的statId。

组合运算符可以是加（+）来组合同一文件或双加（++）中的所有匹配，以组合所有文件之间的所有匹配。

instanceId必须是资源的名称或ID。

typeId是一个冒号（:)，后跟资源类型的名称。

statId是一个句点（。），后跟一个统计信息的名称。

一个TYPEID或实例Id没有statId打印出所有匹配的资源，所有的统计数据。一个TYPEID或实例Id与statId打印出的匹配资源刚刚命名的统计数字。

一个statId没有TYPEID或实例Id匹配与该名称的所有统计数据。

该表显示了rowstore统计信息的选项。

|选项|描述|
|:--|:--|
|-细节|打印静态描述。|
|-没有过滤器|结合-archive，将统计数据打印为原始，未过滤，值。|
|-persec|与 - 归档结合，打印统计数据作为原始值的每秒变化率。|
|-persample|结合-archive，打印统计数据作为每个样本的原始值的变化率。|
|-prunezeros|结合-archive，禁止打印所有零值的统计信息。|
|-StartTime = <时间>|结合-archive，忽略在指定时间之前拍摄的统计信息样本。时间参数格式必须匹配字符串yyyy / MM / dd HH：mm：ss.SSS z其中z是时区。|
|-EndTime = <时间>|结合-archive，忽略在指定时间后拍摄的统计样本。时间参数格式必须匹配字符串yyyy / MM / dd HH：mm：ss.SSS z其中z是时区。|
|-archive|指定要使用的存档文件。|

### 示例1：启动统计信息归档的服务器 ###
	snappy-shell rowstore server start -client-port=1528 -statistic-sampling-enabled=true 
	-statistic-archive-file=system_stats.data
### 示例2：查看统计档案文件 ###
此命令打印所有可用的统计信息：

	snappy-shell stats -archive=system_stats.data
	vmStats, 31616, VMStats: "2011/05/20 20:04:12.633 GMT+05:30" samples=27
	  pendingFinalization objects: samples=27 min=0 max=0 average=0 stddev=0 
	  daemonThreads threads: samples=27 min=22 max=34 average=32.85 stddev=2.23 
	  threads threads: samples=27 min=23 max=35 average=33.85 stddev=2.23 
	  peakThreads threads: samples=27 min=23 max=35 average=34.3 stddev=2.32 
	  threadStarts threads/sec: samples=26 min=0 max=10 average=0.46 stddev=1.96 
	
	Here   instanceId  is   vmStats
	            typeId      is   VMStats
	            statId      are   pendingFinalization, daemonThreads, threads,
	                              peakThreads, threadStarts  
	
	GFXD-DD-DISKSTORE_DIR#0, 31616, DiskDirStatistics: "2011/05/20 20:04:13.633 GMT+05:30" samples=26 
	  diskSpace bytes: samples=26 min=182 max=182 average=182 stddev=0 
	  maximumSpace bytes: samples=26 min=2251799812636672 max=2251799812636672   
	  average=2251799812636672 stddev=0
	
	Here  instanceId  is   GFXD-DD-DISKSTORE_DIR#0
	      typeId      is   DiskDirStatistics
	      statId     are   diskSpace, maximumSpace
	
	GFXD-DEFAULT-DISKSTORE_DIR#0, 31616, DiskDirStatistics: "2011/05/20 20:04:13.633 GMT+05:30" samples=232821 
	          diskSpace bytes: samples=232821 min=0 max=0 average=0 stddev=0 
	          maximumSpace bytes: samples=232821 min=2251799812636672 max=2251799812636672 average=2251799812636672 stddev=0 
### 示例3：查看指定实例ID的所有统计信息 ###
	snappy-shell stats   vmStats -archive=system_stats.data
	[info] Found 14 matches :  for "vmStats" vmStats, 31616, VMStats: 
	"2011/05/20 20:04:12.633 GMT+05:30" samples=234093 
	
	  pendingFinalization objects: samples=234093 min=0 max=1 average=0 stddev=0 
	[info] Found 14 matches :  for "vmStats" vmStats, 31616, VMStats: 
	"2011/05/20 20:04:12.633 GMT+05:30" samples=234093vmStats, 31616, VMStats: 
	"2011/05/20 20:04:12.633 GMT+05:30" samples=234093 
	
	  daemonThreads threads: samples=234093 min=22 max=34 average=33 stddev=0.02 
	[info] Found 14 matches :  for "vmStats" vmStats, 31616, VMStats: 
	"2011/05/20 20:04:12.633 GMT+05:30" samples=234093vmStats, 31616, VMStats: 
	"2011/05/20 20:04:12.633 GMT+05:30" samples=234093vmStats, 31616, VMStats: 
	"2011/05/20 20:04:12.633 GMT+05:30" samples=234093 
	
	  threads threads: samples=234093 min=23 max=35 average=34 stddev=0.02 
	[info] Found 14 matches :  for "vmStats" vmStats, 31616, VMStats: 
	"2011/05/20 20:04:12.633 GMT+05:30" samples=234093vmStats, 31616, VMStats: 
	"2011/05/20 20:04:12.633 GMT+05:30" samples=234093vmStats, 31616, VMStats: 
	"2011/05/20 20:04:12.633 GMT+05:30" samples=234093vmStats, 31616, VMStats: 
	"2011/05/20 20:04:12.633 GMT+05:30" samples=234093 
	
	  peakThreads threads: samples=234093 min=23 max=35 average=35 stddev=0.03 
	[info] Found 14 matches :  for "vmStats" vmStats, 31616, VMStats: 
	"2011/05/20 20:04:12.633 GMT+05:30" samples=234093vmStats, 31616, VMStats: 
	"2011/05/20 20:04:12.633 GMT+05:30" samples=234093vmStats, 31616, VMStats: 
	"2011/05/20 20:04:12.633 GMT+05:30" samples=234093vmStats, 31616, VMStats: 
	"2011/05/20 20:04:12.633 GMT+05:30" samples=234093vmStats, 31616, VMStats: 
	"2011/05/20 20:04:12.633 GMT+05:30" samples=234093
### 示例4：查看指定实例ID和统计ID的所有统计信息 ###
	snappy-shell stats   GFXD-DEFAULT-DISKSTORE_DIR#0:DiskDirStatistics -archive=system_stats.data
	[info] Found 2 matches :  for "GFXD-DEFAULT-DISKSTORE_DIR#0:DiskDirStatistics" GFXD-DEFAULT-DISKSTORE_DIR#0, 
	31616, DiskDirStatistics: "2011/05/20 20:04:13.633 GMT+05:30" samples=236469 
	
	  diskSpace bytes: samples=236469 min=0 max=0 average=0 stddev=0 
	[info] Found 2 matches :  for "GFXD-DEFAULT-DISKSTORE_DIR#0:DiskDirStatistics" GFXD-DEFAULT-DISKSTORE_DIR#0, 
	31616, DiskDirStatistics: "2011/05/20 20:04:13.633 GMT+05:30" samples=236469GFXD-DEFAULT-DISKSTORE_DIR#0, 
	31616, DiskDirStatistics: "2011/05/20 20:04:13.633 GMT+05:30" samples=236469 
	
	  maximumSpace bytes: samples=236469 min=2251799812636672 max=2251799812636672 average=2251799812636672 stddev=0
### 示例5：查看指定实例ID和统计ID的统计信息 ###
	snappy-shell stats  GFXD-DEFAULT-DISKSTORE_DIR#0:DiskDirStatistics.diskSpace -archive=system_stats.data
	[info] Found 1 matches :  for "GFXD-DEFAULT-DISKSTORE_DIR#0:DiskDirStatistics.diskSpace" GFXD-DEFAULT-DISKSTORE_DIR#0, 
	31616, DiskDirStatistics: "2011/05/20 20:04:13.633 GMT+05:30" samples=240133 
	
	  diskSpace bytes: samples=240133 min=0 max=0 average=0 stddev=0

## 升级磁盘店 ##

（此版本不支持。）将磁盘存储升级到当前版本的RowStore。

### 句法 ###

> 注意：此版本的RowStore不支持“upgrade-disk-store”命令。RowStore升级过程会自动加载和升级使用SQLFire 1.1.1和1.1.2创建的磁盘存储文件，因此不需要手动磁盘存储升级，不受支持。请参阅**升级RowStore**。

	snappy-shell upgrade-disk-store
	  <diskStoreName> <directory>+ [-maxOpLogSize=<value>] 
	  [--J=<value>(,<value>)*]

|选项|描述|
|:--|:--|
|diskStoreName|需要。要升级的脱机磁盘存储的名称。|
|目录|需要。写入磁盘存储的数据的一个或多个目录。|
|-maxOpLogSize|由升级创建的oplog的最大大小（以兆字节为单位）。默认大小为1GB。|
|-J|传递给执行升级操作的Java虚拟机的参数。|

### 描述 ###
请参阅**升级RowStore**有关如何升级到最新版本RowStore信息。`snappy-shell upgrade-disk-store`仅当升级文档声明需要手动升级磁盘存储文件时，才能使用该命令。

磁盘存储必须脱机才能升级`snappy-shell upgrade-disk-store`。如果磁盘存储器很大，则可能需要使用该--J=-Xmx参数为进程分配额外的内存。

您的分布式系统包含默认磁盘存储，并且还可能包含使用**CREATE DISKSTORE**语句定义的其他磁盘存储。在`snappy-shell upgrade-disk-store`命令的多次调用中指定要升级的每个磁盘存储的完整路径。RowStore在每个RowStore服务器或定位器目录以及每个RowStore服务器或定位器目录的/ datadictionary子目录中创建默认磁盘存储。可以在CREATE DISKSTORE命令中指定的不同目录中创建自定义磁盘存储。


### 例 ###
此命令会将具有两个目录中的文件的单个磁盘存储库升级：

	snappy-shell upgrade-disk-store myDiskStore /firstDir /secondDir

## 验证磁盘店 ##
验证脱机磁盘存储的运行状况，并使用该磁盘存储提供有关表的信息。

### 句法 ###
	snappy-shell validate-disk-store <diskStoreName> <directory>+

### 描述 ###
该**gfxd验证磁盘店**内命令验证您的磁盘离线商店的健康，给你有关使用磁盘存储，总行，如果你在压缩商店将被删除的记录数表的信息。使用此命令如下所示：

- 在压缩离线磁盘存储之前，以帮助决定是否值得做。
- 恢复磁盘存储之前。
- 任何时候想要确保磁盘存储器的形状很好。

要验证的磁盘存储的名称

存储磁盘存储文件的路径。

### 例 ###
	snappy-shell validate-disk-store ds1 hostB/bupDirectory
此命令显示类似于以下内容的输出：

	/partitioned_region entryCount=6 bucketCount=10
	Disk store contains 1 compactable records.
	Total number of region entries in this disk store is: 6

## 版本 ##
打印有关RowStore产品版本的信息。

### 句法 ###
	snappy-shell version
### 例 ###
	snappy-shell version
	RowStore product directory: /Users/yozie/SnappyData_RowStore_140_b50226_Linux
	Java version:   1.4.0 build 50226 12/09/2014 14:26:16 PST javac 1.7.0_72
	Native version: gemfirexd native code unavailable
	Running on: ward.local/10.0.1.7, 8 cpu(s), x86_64 Mac OS X 10.9.1

## 写数据DTD到文件 ##
创建文档类型定义（DTD）文件，指定使用创建的XML数据文件的布局`snappy-shell write-data-to-xml`。

### 句法 ###
要创建DTD文件，请使用以下语法：

	snappy-shell write-data-dtd-to-file -file=<path>
	     [-auth-provider=<name>]
	     [-bind-address=<address>]
	     [-client-bind-address=<address>]
	     [-client-port=<port>]
	     [-driver-class=<class name>]
	     [-extra-conn-props=<properties>] 
	     [-help] [-locators=<adresses>]
	     [-mcast-address=<address>]
	     [-mcast-port=<port>]
	     [-password[=<password>]]
	     [-url=<url>]
	     [-user=<username>]
	     [-verbose=<level>]
该表描述了该`snappy-shell write-data-dtd-to-file`命令的选项。如果不指定选项，则使用默认值。

|选项|描述|
|:--|:--|
|-文件|写入数据XML的DTD的文件的完整路径。这个参数是必需的。|
|-auth提供商|设置用于对等连接以及客户端 - 服务器连接的身份验证提供程序。有效值为BUILTIN和LDAP。RowStore分布式系统的所有其他成员必须使用相同的身份验证提供程序和用户定义。如果省略此选项，则连接不使用身份验证机制。请参阅[配置安全性](http://rowstore.docs.snappydata.io/docs/deploy_guide/Topics/security/security_chapter.html)。|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下，gfxd使用主机名或localhost，如果主机名指向本地环回地址。|
|-client绑定地址|RowStore定位器侦听客户端连接的主机名或IP地址。默认为“localhost”。<br/><br/>使用此选项与`-client-port`作为瘦客户端附加到RowStore集群并执行该命令。|
|-client端口|RowStore定位器侦听客户端连接的端口。默认值为1527。<br/><br/>使用此选项与`-client-bind-address`作为瘦客户机附加到RowStore集群并执行该命令。|
|-driver级|用于连接到数据源的JDBC驱动程序类。将此选项与`-url`连接到JDBC数据源。|
|-extra-CONN-道具|连接到数据源时要使用的分号分隔列表。|
|-help，-help|显示此snappy-shell命令的帮助信息。|
|-locators|定位器列表，以逗号分隔的主机[port]值，用于发现分布式系统的其他成员。<br/><br/>使用`-locators`创建一个对等体客户端成员来执行snappy-shell命令。|
|-mcast地址|用于发现分布式系统的其他成员的组播地址。该值仅在未指定`-locators`选项时使用。默认多播地址为239.192.81.1。<br/><br/>使用此选项与`-mcast-port`作为对等体客户端附加到RowStore集群并执行该命令。|
|-mcast端口|用于与分布式系统的其他成员进行通信的组播端口。如果为零，组播不会用于成员发现（请指定“-locators”）。仅当未指定`-locators`选项时，才使用此值。<br/><br/>有效值范围为0-65535，默认值为10334。<br/><br/>使用此选项与`-mcast-address`作为对等体客户机附加到RowStore集群并执行该命令。|
|-密码|如果服务器或定位器已配置为使用身份验证，则此选项将指定用户（使用-user选项指定）用于引导服务器并加入分布式系统的密码。<br/><br/>密码值是可选的。如果省略密码，gfxd会提示您从控制台输入密码。|
|-url|用于连接到数据源的JDBC URL。将此选项与`-driver-class'连接到JDBC数据源。|
|-用户|如果服务器或定位器已配置为使用身份验证，则此选项指定用于引导服务器并加入分布式系统的用户名。|
|-verbose|以增加的日志顺序将DdlUtils详细级别设置为FATAL，ERROR，WARN，INFO或DEBUG之一。默认级别为INFO。|

### 描述 ###
使用以下命令指定这些选项之一连接到数据源：

- 同时使用`-client-bind-address`并`-client-port`连接到作为瘦客户机的RowStore集群，并执行该命令。

- 同时使用`mcast-port`和`-mcast-address`，或使用-locators属性连接到RowStore簇作为对等客户机，并执行该命令。

- 使用两者`-url`并`-driver-class`使用JDBC URL和驱动程序连接到数据源。您可以使用此选项连接到除RowStore之外的数据源。

### 例子 ###
此命令连接到在localhost：1527上运行的RowStore网络服务器，并将DTD写入名为data.dtd的文件：

	snappy-shell write-data-dtd-to-file -file=data.dtd
此命令连接到在myserver上运行的RowStore网络服务器：1234，并将DTD写入名为data.dtd的文件：

	
该命令作为对等体客户端连接到在组播端口1234上运行的RowStore系统，并将DTD写入名为data.dtd的文件：

	snappy-shell write-data-dtd-to-file -file=data.dtd -mcast-port=1234
	          -extra-conn-props=host-data=false

该命令使用MySQL Connector / J连接到“myserver”主机上运行的MySQL服务器，并将“测试”数据库模式的DTD写入文件名test.dtd：
	
	snappy-shell write-data-dtd-to-file -file=test.dtd
	          -url=jdbc:mysql://myserver/test -driver-class=com.mysql.jdbc.Driver
	
## 写数据到数据库 ##
使用一个或多个数据XML文件（使用snappy-shell write-data-to-xml创建）将数据插入数据库，并将数据库模式定义在一个或多个模式XML文件中（使用snappy-shell write-schema-到XML）。该命令通常与RowStore集群一起用于导出表数据，但也可以与其他JDBC数据源一起使用。

句法
要使用一个或多个模式XML文件中指定的架构从一个或多个数据XML文件中插入所有表数据，请使用以下语法：

	snappy-shell write-data-to-db -files=<path,path,...> -schema-files=<path,path,...>
	     [-alter-identity-columns]
	     [-auth-provider=<name>]
	     [-batch-size=<size>]
	     [-bind-address=<address>]
	     [-catalog-pattern=<pattern>]
	     [-client-bind-address=<address>]
	     [-client-port=<port>]
	     [-database-type=<db type>]
	     [-delimited-identifiers=<true | false>]
	     [-driver-class=<class name>]
	     [-ensure-fk-order=<true | false>]
	     [-extra-conn-props=<properties>] 
	     [-help] [-locators=<addresses>]
	     [-mcast-address=<address>]
	     [-mcast-port=<port>]
	     [-password[=<password>]]
	     [-schema-pattern=<pattern>]
	     [-url=<url>]
	     [-user=<username>]
	     [-verbose=<level>]
如果您正在将数据写入RowStore分布式系统，并且您知道数据不违反表约束，则可以`skip-constraint-checks`在编写数据时选择使用连接属性来放松主键，外键和唯一约束。请参阅**skip-constraint-checking**。

此表描述了snappy-shell write-data-to-db命令的选项。如果不指定选项，则使用默认值。

|选项|描述|
|:--|:--|
|-files|包含要插入的数据的一个或多个数据XML文件的完整路径。使用逗号分隔的列表来指定多个文件。这个参数是必需的。<br/><br/>请参阅[write-data-to-xml](http://rowstore.docs.snappydata.io/docs/reference/store_commands/store-write-data-to-xml.html)。|
|-alter身份柱|为具有GENERATED ALWAYS标识列的表提供此选项，即使在数据导入期间也禁止手动插入标识值。如果一个或多个表包含要保留的现有GENERATED ALWAYS标识值，请使用write-schema-to-db和write-data-to-db命令指定-alter-identity-columns 。<br/><br/>当将此选项与[write-schema-to-db相结](http://rowstore.docs.snappydata.io/docs/reference/store_commands/store-write-schema-to-db.html)合时，RowStore将一个现有的GENERATED ALWAYS标识列更改为非标识列。这使您能够导入列的现有数据值。<br/><br/>当将此选项与[write-data-to-db相结](http://rowstore.docs.snappydata.io/docs/reference/store_commands/store-write-data-to-db.html)合时，RowStore会在导入最终列值之后将该列更改为一个标识列（GENERATED ALWAYS AS IDENTITY）。然后将为新行自动生成身份值，RowStore可确保所有新标识值都大于上一个导入的值。<br/><br/>作为使用这些选项的替代方法，RowStore支持允许插入标识值的GENERATED BY DEFAULT标识列。请参阅**CREATE TABLE**参考页面中的标识列。|
|-auth提供商|设置用于对等连接以及客户端 - 服务器连接的身份验证提供程序。有效值为BUILTIN和LDAP。RowStore分布式系统的所有其他成员必须使用相同的身份验证提供程序和用户定义。如果省略此选项，则连接不使用身份验证机制。请参阅[配置安全性](http://rowstore.docs.snappydata.io/docs/deploy_guide/Topics/security/security_chapter.html)。|
|-schema档案|在插入数据时要使用的一个或多个模式XML文件的完整路径。使用逗号分隔的列表来指定多个文件。这个参数是必需的。请参阅[write-schema-to-xml。](http://rowstore.docs.snappydata.io/docs/reference/store_commands/store-write-schema-to-xml.html)|
|-batch大小|指定在单个批次中<br/><br/>合并的最大插入语句数。默认批量大小为1000个语句。|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下，gfxd使用主机名或localhost，如果主机名指向本地环回地址。|
|- 目录模式|确定snappy-shell写入的数据库目录的字符串模式。gfxd不使用默认目录模式。<br/><br/>要使用目录模式，请指定描述要编写的目录的字符串值。使用“％”字符匹配0个或更多字符的任何子字符串。使用“_”字符匹配任何单个字符。|
|-client绑定地址|RowStore定位器侦听客户端连接的主机名或IP地址。默认为“localhost”。<br/><br/>使用此选项与`-client-port`作为瘦客户端附加到RowStore集群并执行该命令。|
|-client端口|RowStore定位器侦听客户端连接的端口。默认值为1527。<br/><br/>使用此选项与`-client-bind-address`作为瘦客户机附加到RowStore集群并执行该命令。|
|-database型|指定要连接到的数据库的类型。如果gfxd无法从JDBC驱动程序和JDBC连接URL确定数据库的类型，请使用此选项。有效值为：axion，cloudscape，db2，derby，firebird，hsqldb，interbase，maxdb，mckoi，mssql，mysql，mysql5，oracle，oracle9，oracle10，postgresql，sapdb，gemfirexd和sybase。|
|-delimited的标识符|指定是否对表名，列名等使用分隔符（引用）标识符。大多数数据库将未标记的标识符转换为大写字母，并忽略您在SQL命令中指定的任何情况。<br/><br/>对于支持分隔标识符的平台，您可以将此选项设置为“true”。但是，请记住，当您使用分隔标识符时，必须始终使用双引号括起标识符，并且必须在所有后续SQL命令中为标识符指定正确的大小写。<br/><br/>默认情况下，gfxd将此选项设置为“false”。|
|-driver级|用于连接到数据源的JDBC驱动程序类。将此选项与`-url`连接到JDBC数据源。|
|- 确保-FK-顺序|指定在数据导入期间是否兑现外键的排序。如果所有引用的行都在引用数据XML文件中的行之前将此选项设置为“false”。（或者，更改插入行的顺序，否则保留外键顺序。）|
||默认值为“true”，gfxd会延迟插入引用的行，直到首先插入引用行。此过程消耗内存降低导入性能，因此大型数据集应尽可能保留XML文件中的外键排序。|
|-extra-CONN-道具|连接到数据源时要使用的分号分隔列表。|
|-help，-help|显示此snappy-shell命令的帮助信息。|
|-locators|定位器列表，以逗号分隔的主机[port]值，用于发现分布式系统的其他成员。<br/><br/>使用`-locators`创建一个对等体客户端成员来执行snappy-shell命令。|
|-mcast地址|用于发现分布式系统的其他成员的组播地址。该值仅在未指定`-locators`选项时使用。默认多播地址为239.192.81.1。<br/><br/>使用此选项与`-mcast-port`作为对等体客户端附加到RowStore集群并执行该命令。|
|-mcast端口|用于与分布式系统的其他成员进行通信的组播端口。如果为零，组播不会用于成员发现（请指定“-locators”）。仅当未指定`-locators`选项时，才使用此值。<br/><br/>有效值范围为0-65535，默认值为10334。<br/><br/>使用此选项与`-mcast-address`作为对等体客户机附加到RowStore集群并执行该命令。|
|-密码|如果服务器或定位器已配置为使用身份验证，则此选项将指定用户（使用-user选项指定）用于引导服务器并加入分布式系统的密码。<br/><br/>密码值是可选的。如果省略密码，gfxd会提示您从控制台输入密码。|
|-schema模式|确定snappy-shell写入的模式的字符串模式。gfxd不使用默认模式模式。但是，某些数据库可能需要使用模式模式来排除包含与DdlUtils 1.1 API不兼容的数据类型的系统表。<br/><br/>要使用模式模式，请指定描述要编写的目录的字符串值。使用“％”字符匹配0个或更多字符的任何子字符串。使用“_”字符匹配任何单个字符。|
|-url|用于连接到数据源的JDBC URL。将此选项与`-driver-class'连接到JDBC数据源。|
|-用户|如果服务器或定位器已配置为使用身份验证，则此选项指定用于引导服务器并加入分布式系统的用户名。|
|-verbose|以增加的日志顺序将DdlUtils详细级别设置为FATAL，ERROR，WARN，INFO或DEBUG之一。默认级别为INFO。|

### 描述 ###
另请参见write-data-to-xml和write-schema-to-xml。

使用以下命令指定这些选项之一连接到数据源：

- 同时使用-client-bind-address并-client-port连接到作为瘦客户机的RowStore集群，并执行该命令。

- 同时使用mcast-port和-mcast-address，或使用-locators属性连接到RowStore簇作为对等客户机，并执行该命令。

- 使用两者-url并-driver-class使用JDBC URL和驱动程序连接到数据源。您可以使用此选项连接到除RowStore之外的数据源。

### 例子 ###

> 注意：另请参阅导出，更改和导入数据库模式使用GFXD获取将第三方数据库迁移到RowStore的完整示例。

此命令连接到在localhost：1527上运行的RowStore网络服务器，并插入data.xml中的数据：

	snappy-shell write-data-to-db -files=data.xml -schema-files=db-schema.xml
此命令连接到在myserver上运行的RowStore网络服务器：1234，并从data1.xml和data2.xml文件中插入数据：

	snappy-shell write-data-to-db -files=data1.xml,data2.xml -schema-files=db-schema.xml
	     -client-bind-address=myserver -client-port=1234
此命令作为对等客户端连接到运行在组播端口1234上的RowStore系统，并插入data.xml中的数据：

	snappy-shell write-data-to-db -files=data.xml -schema-files=db-schema.xml -mcast-port=1234
	          -extra-conn-props=host-data=false

该命令使用MySQL Connector / J连接到在“myserver”主机上运行的MySQL服务器，并将数据从data.xml插入到“测试”数据库中：

	snappy-shell write-data-to-db -files=data.xml -schema-files=db-schema.xml
	          -url=jdbc:mysql://myserver/test -driver-class=com.mysql.jdbc.Driver

## 写数据到XML ##
将数据库中的所有表的数据写入XML文件。（您可以使用snappy-shell write-data-dtd-to-file命令创建描述XML文件中数据布局的文档类型定义（DTD）文件。）生成的XML文件可用于重新生成在RowStore或另一个数据库管理系统中将数据加载到表中。该命令通常与RowStore集群一起用于导出表数据，但也可以与其他JDBC数据源一起使用。

### 句法 ###
要将数据库中的所有表数据写入XML文件，请使用以下语法：

	snappy-shell write-data-to-xml -file=<path>
	     [-auth-provider=<name>]
	     [-bind-address=<address>]
	     [-catalog-pattern=<pattern>]
	     [-client-bind-address=<address>]
	     [-client-port=<port>]
	     [-database-type=<db type>]
	     [-delimited-identifiers=<true | false>]
	     [-driver-class=<class name>]
	     [-exclude-table-filter=<filter>] 
	     [-exclude-tables=<name,name,...>]
	     [-extra-conn-props=<properties>] 
	     [-help] 
	     [-include-table-filter=<filter>] 
	     [-include-tables=<name,name,...>]
	     [-isolation-level=<level>]
	     [-locators=<adresses>]
	     [-mcast-address=<address>]
	     [-mcast-port=<port>]
	     [-password[=<password>]]
	     [-schema-pattern=<pattern>]
	     [-url=<url>]
	     [-user=<username>]
	     [-verbose=<level>]
该表描述了snappy-shell write-data-to-xml命令的选项。如果不指定选项，则使用默认值。

|选项|描述|
|:--|:--|
|-文件|要写入表数据的XML文件的完整路径。这个参数是必需的。|
|-auth提供商|设置用于对等连接以及客户端 - 服务器连接的身份验证提供程序。有效值为BUILTIN和LDAP。RowStore分布式系统的所有其他成员必须使用相同的身份验证提供程序和用户定义。如果省略此选项，则连接不使用身份验证机制。请参阅[配置安全性](http://rowstore.docs.snappydata.io/docs/deploy_guide/Topics/security/security_chapter.html)。|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下，gfxd使用主机名或localhost，如果主机名指向本地环回地址。|
|- 目录模式|确定snappy-shell写入的数据库目录的字符串模式。gfxd不使用默认目录模式。<br/><br/>要使用目录模式，请指定描述要编写的目录的字符串值。使用“％”字符匹配0个或更多字符的任何子字符串。使用“_”字符匹配任何单个字符。|
|-client绑定地址|RowStore定位器侦听客户端连接的主机名或IP地址。默认为“localhost”。<br/><br/>使用此选项与`-client-port`作为瘦客户端附加到RowStore集群并执行该命令。|
|-client端口|RowStore定位器侦听客户端连接的端口。默认值为1527。<br/><br/>使用此选项与`-client-bind-address`作为瘦客户机附加到RowStore集群并执行该命令。|
|-database型|指定要连接到的数据库的类型。如果gfxd无法从JDBC驱动程序和JDBC连接URL确定数据库的类型，请使用此选项。有效值为：axion，cloudscape，db2，derby，firebird，hsqldb，interbase，maxdb，mckoi，mssql，mysql，mysql5，oracle，oracle9，oracle10，postgresql，sapdb，gemfirexd和sybase。|
|-delimited的标识符|指定是否对表名，列名等使用分隔符（引用）标识符。大多数数据库将未标记的标识符转换为大写字母，并忽略您在SQL命令中指定的任何情况。<br/><br/>对于支持分隔标识符的平台，您可以将此选项设置为“true”。但是，请记住，当您使用分隔标识符时，必须始终使用双引号括起标识符，并且必须在所有后续SQL命令中为标识符指定正确的大小写。<br/><br/>默认情况下，gfxd将此选项设置为“false”。|
|-driver级|用于连接到数据源的JDBC驱动程序类。将此选项与`-url`连接到JDBC数据源。|
|-exclude表滤波器|指定在读取数据库时排除表的正则表达式。与模式匹配的表不包括在输出文件中。对于不区分大小写的匹配（`-delimited-identifiers`选项为false），在模式中指定大写表名。|
|-exclude桌|指定读取数据库时要排除的一个或多个表的名称。|
|-extra-CONN-道具|连接到数据源时要使用的分号分隔列表。|
|-help，-help|显示此snappy-shell命令的帮助信息。|
|-include表滤波器|指定读取数据库时用于包括表的正则表达式。与模式匹配的表格包含在输出文件中。对于不区分大小写的匹配（`-delimited-identifiers`选项为false），在模式中指定大写表名。|
|-include桌|指定读取数据库时要包含的一个或多个表的名称。|
|分离所级|设置数据库操作的ANSI标准事务隔离级别。有效值为READ_UNCOMMITTED，READ_COMMITTED，REPEATABLE_READ和SERIALIZABLE（不区分大小写）。|
|-locators|定位器列表，以逗号分隔的主机[port]值，用于发现分布式系统的其他成员。<br/><br/>使用`-locators`创建一个对等体客户端成员来执行snappy-shell命令。|
|-mcast地址|用于发现分布式系统的其他成员的组播地址。该值仅在未指定`-locators`选项时使用。默认多播地址为239.192.81.1。|
||使用此选项与`-mcast-port`作为对等体客户端附加到RowStore集群并执行该命令。|
|-mcast端口|用于与分布式系统的其他成员进行通信的组播端口。如果为零，组播不会用于成员发现（请指定“-locators”）。仅当未指定`-locators`选项时，才使用此值。<br/><br/>有效值范围为0-65535，默认值为10334。<br/><br/>使用此选项与`-mcast-address`作为对等体客户机附加到RowStore集群并执行该命令。|
|-密码|如果服务器或定位器已配置为使用身份验证，则此选项将指定用户（使用-user选项指定）用于引导服务器并加入分布式系统的密码。<br/><br/>密码值是可选的。如果省略密码，gfxd会提示您从控制台输入密码。|
|-schema模式|确定snappy-shell写入的模式的字符串模式。gfxd不使用默认模式模式。但是，某些数据库可能需要使用模式模式来排除包含与DdlUtils 1.1 API不兼容的数据类型的系统表。<br/><br/>要使用模式模式，请指定描述要编写的目录的字符串值。使用“％”字符匹配0个或更多字符的任何子字符串。使用“_”字符匹配任何单个字符。|
|-url|用于连接到数据源的JDBC URL。将此选项与`-driver-class'连接到JDBC数据源。|
|-用户|如果服务器或定位器已配置为使用身份验证，则此选项指定用于引导服务器并加入分布式系统的用户名。|
|-verbose|以增加的日志顺序将DdlUtils详细级别设置为FATAL，ERROR，WARN，INFO或DEBUG之一。默认级别为INFO。|

### 描述 ###
另请参见**write-data-dtd-to-file**。

使用以下命令指定这些选项之一连接到数据源：

- 同时使用`-client-bind-address`并`-client-port`连接到作为瘦客户机的RowStore集群，并执行该命令。

- 同时使用`mcast-port`和`-mcast-address`，或使用`-locators`属性连接到RowStore簇作为对等客户机，并执行该命令。
- 使用两者`-url`并`-driver-class`使用JDBC URL和驱动程序连接到数据源。您可以使用此选项连接到除RowStore之外的数据源。

### 例子 ###

> 注意：另请参阅**导出，更改和导入数据库模式**使用GFXD获取将第三方数据库迁移到RowStore的完整示例。

此命令连接到在localhost：1527上运行的RowStore网络服务器，并将表数据写入名为data.xml的文件：

	snappy-shell write-data-to-xml -file=data.xml
此命令连接到在myserver上运行的RowStore网络服务器：1234，并将表数据写入名为data.xml的文件：

	snappy-shell write-data-to-xml -file=data.xml -client-bind-address=myserver -client-	port=1234
此命令作为对等客户端连接到运行在组播端口1234上的RowStore系统，并将表数据写入名为data.xml的文件：

	snappy-shell write-data-to-xml -file=data.xml -mcast-port=1234
	          -extra-conn-props=host-data=false

该命令使用MySQL Connector / J连接到在“myserver”主机上运行的MySQL服务器，并将表数据从“测试”数据库模式写入文件名为data.xml：

	snappy-shell write-data-to-xml -file=data.xml
	          -url=jdbc:mysql://myserver/test -driver-class=com.mysql.jdbc.Driver

## 写模式到数据库 ##
通过使用一个或多个模式XML文件的内容（请参阅snappy-shell write-schema-to-xml）在数据库中创建模式。此命令通常与RowStore集群一起用于导出模式，但也可以与其他JDBC数据源一起使用。

句法
要从一个或多个模式XML文件创建数据库模式，请使用以下语法：

	snappy-shell write-schema-to-db -files=<path,path,...>
	     [-alter-identity-columns]
	     [-auth-provider=<name>]
	     [-bind-address=<address>]
	     [-catalog-pattern=<pattern>]
	     [-client-bind-address=<address>]
	     [-client-port=<port>]
	     [-database-type=<db type>]
	     [-delimited-identifiers=<true | false>]
	     [-do-drops]
	     [-driver-class=<class name>]
	     [-extra-conn-props=<properties>] 
	     [-help] [-locators=<adresses>]
	     [-mcast-address=<address>]
	     [-mcast-port=<port>]
	     [-password[=<password>]]
	     [-schema-pattern=<pattern>]
	     [-url=<url>]
	     [-user=<username>]
	     [-verbose=<level>]
此表描述了snappy-shell write-schema-to-db命令的选项。如果不指定选项，则使用默认值。

|选项|描述|
|:--|:--|
|-files|用于创建模式的一个或多个模式XML文件的完整路径。使用逗号分隔的列表来指定多个文件。这个参数是必需的。|
|-alter身份柱|为具有GENERATED ALWAYS标识列的表提供此选项，即使在数据导入期间也禁止手动插入标识值。如果一个或多个表包含要保留的现有GENERATED ALWAYS标识值，请使用write-schema-to-db和write-data-to-db命令指定-alter-identity-columns 。<br/><br/>当将此选项与[write-schema-to-db相结合](http://rowstore.docs.snappydata.io/docs/reference/store_commands/store-write-schema-to-db.html)时，RowStore将一个现有的GENERATED ALWAYS标识列更改为非标识列。这使您能够导入列的现有数据值。<br/><br/>当将此选项与[write-data-to-db相结](http://rowstore.docs.snappydata.io/docs/reference/store_commands/store-write-data-to-db.html)合时，RowStore会在导入最终列值之后将该列更改为一个标识列（GENERATED ALWAYS AS IDENTITY）。然后将为新行自动生成身份值，RowStore可确保所有新标识值都大于上一个导入的值。<br/><br/>作为使用这些选项的替代方法，RowStore支持允许插入标识值的GENERATED BY DEFAULT标识列。请参阅CREATE TABLE参考页面中的标识列。|
|-auth提供商|设置用于对等连接以及客户端 - 服务器连接的身份验证提供程序。有效值为BUILTIN和LDAP。RowStore分布式系统的所有其他成员必须使用相同的身份验证提供程序和用户定义。如果省略此选项，则连接不使用身份验证机制。请参阅[配置安全性。](http://rowstore.docs.snappydata.io/docs/deploy_guide/Topics/security/security_chapter.html)|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下，gfxd使用主机名或localhost，如果主机名指向本地环回地址。|
|- 目录模式|确定snappy-shell写入的数据库目录的字符串模式。gfxd不使用默认目录模式。<br/><br/>要使用目录模式，请指定描述要编写的目录的字符串值。使用“％”字符匹配0个或更多字符的任何子字符串。使用“_”字符匹配任何单个字符。|
|-client绑定地址|RowStore定位器侦听客户端连接的主机名或IP地址。默认为“localhost”。<br/><br/>使用此选项与`-client-port`作为瘦客户端附加到RowStore集群并执行该命令。|
|-client端口|RowStore定位器侦听客户端连接的端口。默认值为1527。<br/><br/>使用此选项与`-client-bind-address`作为瘦客户机附加到RowStore集群并执行该命令。|
|-database型|指定要连接到的数据库的类型。如果gfxd无法从JDBC驱动程序和JDBC连接URL确定数据库的类型，请使用此选项。有效值为：axion，cloudscape，db2，derby，firebird，hsqldb，interbase，maxdb，mckoi，mssql，mysql，mysql5，oracle，oracle9，oracle10，postgresql，sapdb，gemfirexd和sybase。|
|-delimited的标识符|指定是否对表名，列名等使用分隔符（引用）标识符。大多数数据库将未标记的标识符转换为大写字母，并忽略您在SQL命令中指定的任何情况。|
||对于支持分隔标识符的平台，您可以将此选项设置为“true”。但是，请记住，当您使用分隔标识符时，必须始终使用双引号括起标识符，并且必须在所有后续SQL命令中为标识符指定正确的大小写。<br/><br/>默认情况下，gfxd将此选项设置为“false”。|
|-DO滴|包括此选项以在重新创建SQL之前生成用于删除表及其关联的外部约束（如有必要）的SQL。|
|-driver级|用于连接到数据源的JDBC驱动程序类。将此选项与`-url`连接到JDBC数据源。|
|-extra-CONN-道具|连接到数据源时要使用的分号分隔列表。|
|-help，-help|显示此snappy-shell命令的帮助信息。|
|-locators|定位器列表，以逗号分隔的主机[port]值，用于发现分布式系统的其他成员。<br/><br/>使用`-locators`创建一个对等体客户端成员来执行snappy-shell命令。|
|-mcast地址|用于发现分布式系统的其他成员的组播地址。该值仅在未指定`-locators`选项时使用。默认多播地址为239.192.81.1。<br/><br/>使用此选项与`-mcast-port`作为对等体客户端附加到RowStore集群并执行该命令。|
|-mcast端口|用于与分布式系统的其他成员进行通信的组播端口。如果为零，组播不会用于成员发现（请指定“-locators”）。仅当未指定`-locators`选项时，才使用此值。<br/><br/>有效值范围为0-65535，默认值为10334。<br/>使用此选项与`-mcast-address`作为对等体客户机附加到RowStore集群并执行该命令。|
|-密码|如果服务器或定位器已配置为使用身份验证，则此选项将指定用户（使用-user选项指定）用于引导服务器并加入分布式系统的密码。<br/><br/>密码值是可选的。如果省略密码，gfxd会提示您从控制台输入密码。|
|-schema模式|确定snappy-shell写入的模式的字符串模式。gfxd不使用默认模式模式。但是，某些数据库可能需要使用模式模式来排除包含与DdlUtils 1.1 API不兼容的数据类型的系统表。<br/><br/>要使用模式模式，请指定描述要编写的目录的字符串值。使用“％”字符匹配0个或更多字符的任何子字符串。使用“_”字符匹配任何单个字符。|
|-url|用于连接到数据源的JDBC URL。将此选项与`-driver-class'连接到JDBC数据源。|
|-用户|如果服务器或定位器已配置为使用身份验证，则此选项指定用于引导服务器并加入分布式系统的用户名。|
|-verbose|以增加的日志顺序将DdlUtils详细级别设置为FATAL，ERROR，WARN，INFO或DEBUG之一。默认级别为INFO。|

### 描述 ###
另请参见write-schema-to-xml。

使用以下命令指定这些选项之一连接到数据源：

- 同时使用`-client-bind-address`并`-client-port`连接到作为瘦客户机的RowStore集群，并执行该命令。

- 同时使用`mcast-port`和`-mcast-address`，或使用`-locators`属性连接到RowStore簇作为对等客户机，并执行该命令。

- 使用两者`-url`并`-driver-class`使用JDBC URL和驱动程序连接到数据源。您可以使用此选项连接到除RowStore之外的数据源。

### 例子 ###
此命令连接到在localhost：1527上运行的RowStore网络服务器，并使用名为db-schema.xml的模式XML文件创建数据库模式：

	snappy-shell write-schema-to-db -files=db-schema.xml
此命令连接到在myserver上运行的RowStore网络服务器：1234，并使用名为db-schema1.xml和db-schema2.xml的两个模式XML文件创建数据库模式：

	snappy-shell write-schema-to-db -files=db-schema1.xml,db-schema2.xml 
	     -client-bind-address=myserver -client-port=1234
此命令作为对等客户端连接到在多播端口1234上运行的RowStore系统，并使用名为db-schema.xml的模式XML文件创建数据库模式：

	snappy-shell write-schema-to-db -files=db-schema.xml -mcast-port=1234
	          -extra-conn-props=host-data=false

此命令使用MySQL Connector / J连接到运行在“myserver”主机上的MySQL服务器，并使用模式XML文件名db-schema.xml在“测试”数据库中创建数据库模式：
	
	snappy-shell write-schema-to-db -files=db-schema.xml
	          -url=jdbc:mysql://myserver/test -driver-class=com.mysql.jdbc.Driver


## 写模式到SQL ##
将数据库的模式作为SQL命令写入文件。您可以使用生成的文件在RowStore或另一个数据库管理系统中重新创建模式。此命令通常与RowStore集群一起用于导出模式，但也可以与其他JDBC数据源一起使用。

### 句法 ###
要将数据库模式作为SQL命令写入文件，请使用以下语法：

	snappy-shell write-schema-to-sql -file=<path>
	     [-auth-provider=<name>]
	     [-bind-address=<address>]
	     [-catalog-pattern=<pattern>]
	     [-client-bind-address=<address>]
	     [-client-port=<port>]
	     [-database-type=<db type>]
	     [-delimited-identifiers=<true | false>]
	     [-driver-class=<class name>]
	     [-exclude-table-filter=<filter>] 
	     [-exclude-tables=<name,name,...>]
	     [-export-all]
	     [-extra-conn-props=<properties>] 
	     [-generic]
	     [-help] 
	     [-include-table-filter=<filter>] 
	     [-include-tables=<name,name,...>]
	     [-isolation-level=<level>]
	     [-locators=<addresses>]
	     [-mcast-address=<address>]
	     [-mcast-port=<port>]
	     [-password[=<password>]]
	     [-schema-pattern=<pattern>]
	     [-to-database-type=<type>]
	     [-url=<url>]
	     [-user=<username>]
	     [-verbose=<level>]
	     [-xml-schema-files=<path,path,...>]
此表描述了snappy-shell write-schema-to-sql命令的选项。如果不指定选项，则使用默认值。

|选项|描述|
|:--|:--|
|-文件|存储SQL命令的文件的完整路径。这个参数是必需的。|
|-auth提供商|设置用于对等连接以及客户端 - 服务器连接的身份验证提供程序。有效值为BUILTIN和LDAP。RowStore分布式系统的所有其他成员必须使用相同的身份验证提供程序和用户定义。如果省略此选项，则连接不使用身份验证机制。请参阅[配置安全性。](http://rowstore.docs.snappydata.io/docs/deploy_guide/Topics/security/security_chapter.html)|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下，gfxd使用主机名或localhost，如果主机名指向本地环回地址。|
|- 目录模式|确定snappy-shell写入的数据库目录的字符串模式。gfxd不使用默认目录模式。<br/><br/>要使用目录模式，请指定描述要编写的目录的字符串值。使用“％”字符匹配0个或更多字符的任何子字符串。使用“_”字符匹配任何单个字符。|
|-client绑定地址|RowStore定位器侦听客户端连接的主机名或IP地址。默认为“localhost”。<br/><br/>使用此选项与`-client-port`作为瘦客户端附加到RowStore集群并执行该命令。|
|-client端口|RowStore定位器侦听客户端连接的端口。默认值为1527。<br/><br/>使用此选项与`-client-bind-address`作为瘦客户机附加到RowStore集群并执行该命令。|
|-database型|指定要连接到的数据库的类型。如果gfxd无法从JDBC驱动程序和JDBC连接URL确定数据库的类型，请使用此选项。有效值为：axion，cloudscape，db2，derby，firebird，hsqldb，interbase，maxdb，mckoi，mssql，mysql，mysql5，oracle，oracle9，oracle10，postgresql，sapdb，gemfirexd和sybase。|
|-delimited的标识符|指定是否对表名，列名等使用分隔符（引用）标识符。大多数数据库将未标记的标识符转换为大写字母，并忽略您在SQL命令中指定的任何情况。<br/><br/>对于支持分隔标识符的平台，您可以将此选项设置为“true”。但是，请记住，当您使用分隔标识符时，必须始终使用双引号括起标识符，并且必须在所有后续SQL命令中为标识符指定正确的大小写。<br/><br/>默认情况下，gfxd将此选项设置为“false”。|
|-driver级|用于连接到数据源的JDBC驱动程序类。将此选项与`-url`连接到JDBC数据源。|
|-exclude表滤波器|指定在读取数据库时排除表的正则表达式。与模式匹配的表不包括在输出文件中。对于不区分大小写的匹配（`-delimited-identifiers`选项为false），在模式中指定大写表名。|
|-exclude桌|指定读取数据库时要排除的一个或多个表的名称。|
|-export-ALL|将此选项作为SQL语句包括在所有数据库对象中，包括所有必需的RowStore特定扩展，如持久性子句。提供此选项作为执行完整RowStore模式备份的方法，因为它涵盖了所有对象（如已安装的JAR）等。<br/><br/>注意：为了执行此实用程序，您必须以系统用户身份连接到RowStore分布式系统。|
|-extra-CONN-道具|连接到数据源时要使用的分号分隔列表。|
|-generic|默认情况下，当您从RowStore导出模式时，gfxd将所有DDL作为包含RowStore特定扩展（如分区，持久性等）的SQL语句导出。包括`-generic`覆盖此默认行为的选项，并仅导出对JDBC驱动程序可见的通用模式对象。|
|-help，-help|显示此snappy-shell命令的帮助信息。|
|-include表滤波器|指定读取数据库时用于包括表的正则表达式。与模式匹配的表格包含在输出文件中。对于不区分大小写的匹配（`-delimited-identifiers`选项为false），在模式中指定大写表名。|
|-include桌|指定读取数据库时要包含的一个或多个表的名称。|
|分离所级|设置数据库操作的ANSI标准事务隔离级别。有效值为READ_UNCOMMITTED，READ_COMMITTED，REPEATABLE_READ和SERIALIZABLE（不区分大小写）。|
|-locators|定位器列表，以逗号分隔的主机[port]值，用于发现分布式系统的其他成员。<br/><br/>使用`-locators`创建一个对等体客户端成员来执行snappy-shell命令。|
|-mcast地址|用于发现分布式系统的其他成员的组播地址。该值仅在未指定`-locators`选项时使用。默认多播地址为239.192.81.1。<br/><br/>使用此选项与`-mcast-port`作为对等体客户端附加到RowStore集群并执行该命令。|
|-mcast端口|用于与分布式系统的其他成员进行通信的组播端口。如果为零，组播不会用于成员发现（请指定“-locators”）。仅当未指定`-locators`选项时，才使用此值。<br/><br/>有效值范围为0-65535，默认值为10334。<br/><br/>使用此选项与`-mcast-address`作为对等体客户机附加到RowStore集群并执行该命令。|
|-密码|如果服务器或定位器已配置为使用身份验证，则此选项将指定用户（使用-user选项指定）用于引导服务器并加入分布式系统的密码。<br/><br/>密码值是可选的。如果省略密码，gfxd会提示您从控制台输入密码。|
|-schema模式|确定snappy-shell写入的模式的字符串模式。gfxd不使用默认模式模式。但是，某些数据库可能需要使用模式模式来排除包含与DdlUtils 1.1 API不兼容的数据类型的系统表。<br/><br/>要使用模式模式，请指定描述要编写的目录的字符串值。使用“％”字符匹配0个或更多字符的任何子字符串。使用“_”字符匹配任何单个字符。|
|-to-数据库型|指定格式化输出文件的SQL语句时要使用的目标数据库类型。有效值为：axion，cloudscape，db2，derby，firebird，hsqldb，interbase，maxdb，mckoi，mssql，mysql，mysql5，oracle，oracle9，oracle10，postgresql，sapdb，gemfirexd和sybase。<br/><br/>当您想从一个数据库读取模式并以与其他数据库兼容的格式创建SQL文件时，请使用此选项。|
|-url|用于连接到数据源的JDBC URL。将此选项与`-driver-class'连接到JDBC数据源。|
|-用户|如果服务器或定位器已配置为使用身份验证，则此选项指定用于引导服务器并加入分布式系统的用户名。|
|-verbose|以增加的日志顺序将DdlUtils详细级别设置为FATAL，ERROR，WARN，INFO或DEBUG之一。默认级别为INFO。|
|-xml-架构文件|指定一个或多个XML模式文件用作读取模式的源。使用此选项从使用[write-schema-to-xml](http://rowstore.docs.snappydata.io/docs/reference/store_commands/store-write-schema-to-xml.html)而不是从数据库连接创建的XML文件生成SQL文件。|

### 例子 ###

> 注意：另请参阅**导出，更改和导入数据库模式**使用GFXD获取将第三方数据库迁移到RowStore的完整示例。

此命令使用系统用户帐户连接在localhost：1527上运行的RowStore定位器，并将所有数据库对象（使用RowStore特定扩展）备份到文件db-schema.sql：

	snappy-shell write-schema-to-sql -user=serveradmin -password=serverpassword -export-all 
		-file=db-schema.sql -client-bind-address=localhost -client-port=1527
请注意，上述示例中的分布式系统必须已经定义了“serveradmin”系统用户和密码。有关详细信息，请参阅**创建系统用户并启动RowStore成员**。

该命令使用MySQL Connector / J连接到在“myserver”主机上运行的MySQL服务器，并将“测试”数据库模式写入文件名db-schema.sql。SQL文件格式化为与RowStore一起使用：
	
	snappy-shell write-schema-to-sql -file=db-schema.sql -to-database-type=gemfirexd
	          -url=jdbc:mysql://myserver/test -driver-class=com.mysql.jdbc.Driver

## 写模式到XML ##
将数据库的模式写入XML文件。您可以使用生成的XML文件在RowStore或另一个数据库管理系统中重新创建模式。此命令通常与RowStore集群一起用于导出模式，但也可以与其他JDBC数据源一起使用。

### 句法 ###
要将数据库模式写入XML文件，请使用以下语法：

	snappy-shell write-schema-to-xml -file=<path>
	     [-auth-provider=<name>]
	     [-bind-address=<address>]
	     [-catalog-pattern=<pattern>]
	     [-client-bind-address=<address>]
	     [-client-port=<port>]
	     [-database-type=<db type>]
	     [-delimited-identifiers=<true | false>]
	     [-driver-class=<class name>]
	     [-exclude-table-filter=<filter>] 
	     [-exclude-tables=<name,name,...>]
	     [-extra-conn-props=<properties>] 
	     [-help] 
	     [-include-table-filter=<filter>] 
	     [-include-tables=<name,name,...>]
	     [-isolation-level=<level>]
	     [-locators=<adresses>]
	     [-mcast-address=<address>]
	     [-mcast-port=<port>]
	     [-password[=<password>]]
	     [-schema-pattern=<pattern>]
	     [-url=<url>]
	     [-user=<username>]
	     [-verbose=<level>]
此表描述了snappy-shell write-schema-to-xml命令的选项。如果不指定选项，则使用默认值。

|选项|描述|
|:--|:--|
|-文件|要创建的XML文件的完整路径。这个参数是必需的。|
|-auth提供商|设置用于对等连接以及客户端 - 服务器连接的身份验证提供程序。有效值为BUILTIN和LDAP。RowStore分布式系统的所有其他成员必须使用相同的身份验证提供程序和用户定义。如果省略此选项，则连接不使用身份验证机制。请参[阅配置安全性。](http://rowstore.docs.snappydata.io/docs/deploy_guide/Topics/security/security_chapter.html)|
|-bind地址|该对等体绑定以接收对等消息的地址。默认情况下，gfxd使用主机名或localhost，如果主机名指向本地环回地址。|
|- 目录模式|确定snappy-shell写入的数据库目录的字符串模式。gfxd不使用默认目录模式。<br/><br/>要使用目录模式，请指定描述要编写的目录的字符串值。使用“％”字符匹配0个或更多字符的任何子字符串。使用“_”字符匹配任何单个字符。|
|-client绑定地址|RowStore定位器侦听客户端连接的主机名或IP地址。默认为“localhost”。<br/><br/>使用此选项与`-client-port`作为瘦客户端附加到RowStore集群并执行该命令。|
|-client端口|RowStore定位器侦听客户端连接的端口。默认值为1527。<br/><br/>使用此选项与`-client-bind-address`作为瘦客户机附加到RowStore集群并执行该命令。|
|-database型|指定要连接到的数据库的类型。如果gfxd无法从JDBC驱动程序和JDBC连接URL确定数据库的类型，请使用此选项。有效值为：axion，cloudscape，db2，derby，firebird，hsqldb，interbase，maxdb，mckoi，mssql，mysql，mysql5，oracle，oracle9，oracle10，postgresql，sapdb，gemfirexd和sybase。|
|-delimited的标识符|指定是否对表名，列名等使用分隔符（引用）标识符。大多数数据库将未标记的标识符转换为大写字母，并忽略您在SQL命令中指定的任何情况。<br/><br/>对于支持分隔标识符的平台，您可以将此选项设置为“true”。但是，请记住，当您使用分隔标识符时，必须始终使用双引号括起标识符，并且必须在所有后续SQL命令中为标识符指定正确的大小写。<br/><br/>默认情况下，gfxd将此选项设置为“false”。|
|-driver级|用于连接到数据源的JDBC驱动程序类。将此选项与`-url`连接到JDBC数据源。|
|-exclude表滤波器|指定在读取数据库时排除表的正则表达式。与模式匹配的表不包括在输出文件中。对于不区分大小写的匹配（`-delimited-identifiers`选项为false），在模式中指定大写表名。|
|-exclude桌|指定读取数据库时要排除的一个或多个表的名称。|
|-extra-CONN-道具|连接到数据源时要使用的分号分隔列表。|
|-help，-help|显示此snappy-shell命令的帮助信息。|
|-include表滤波器|指定读取数据库时用于包括表的正则表达式。与模式匹配的表格包含在输出文件中。对于不区分大小写的匹配（`-delimited-identifiers`选项为false），在模式中指定大写表名。|
|-include桌|指定读取数据库时要包含的一个或多个表的名称。|
|分离所级|设置数据库操作的ANSI标准事务隔离级别。有效值为READ_UNCOMMITTED，READ_COMMITTED，REPEATABLE_READ和SERIALIZABLE（不区分大小写）。|
|-locators|定位器列表，以逗号分隔的主机[port]值，用于发现分布式系统的其他成员。<br/><br/>使用`-locators`创建一个对等体客户端成员来执行snappy-shell命令。|
|-mcast地址|用于发现分布式系统的其他成员的组播地址。该值仅在未指定`-locators`选项时使用。默认多播地址为239.192.81.1。<br/><br/>使用此选项与`-mcast-port`作为对等体客户端附加到RowStore集群并执行该命令。|
|-mcast端口|用于与分布式系统的其他成员进行通信的组播端口。如果为零，组播不会用于成员发现（请指定“-locators”）。仅当未指定`-locators`选项时，才使用此值。<br/><br/>有效值范围为0-65535，默认值为10334。<br/><br/>使用此选项与`-mcast-address`作为对等体客户机附加到RowStore集群并执行该命令。|
|-密码|如果服务器或定位器已配置为使用身份验证，则此选项将指定用户（使用-user选项指定）用于引导服务器并加入分布式系统的密码。<br/><br/>密码值是可选的。如果省略密码，gfxd会提示您从控制台输入密码。|
|-schema模式|确定snappy-shell写入的模式的字符串模式。gfxd不使用默认模式模式。但是，某些数据库可能需要使用模式模式来排除包含与DdlUtils 1.1 API不兼容的数据类型的系统表。<br/><br/>要使用模式模式，请指定描述要编写的目录的字符串值。使用“％”字符匹配0个或更多字符的任何子字符串。使用“_”字符匹配任何单个字符。|
|-url|用于连接到数据源的JDBC URL。将此选项与`-driver-class'连接到JDBC数据源。|
|-用户|如果服务器或定位器已配置为使用身份验证，则此选项指定用于引导服务器并加入分布式系统的用户名。|
|-verbose|以增加的日志顺序将DdlUtils详细级别设置为FATAL，ERROR，WARN，INFO或DEBUG之一。默认级别为INFO。|

### 描述 ###
使用以下命令指定这些选项之一连接到数据源：

- 同时使用`-client-bind-address`并`-client-port`连接到作为瘦客户机的RowStore集群，并执行该命令。

- 同时使用`mcast-port`和`-mcast-address`，或使用-locators属性连接到RowStore簇作为对等客户机，并执行该命令。

- 使用两者`-url`并`-driver-class`使用JDBC URL和驱动程序连接到数据源。您可以使用此选项连接到除RowStore之外的数据源。

### 例子 ###

> 注意：另请参阅导出，更改和导入数据库模式使用GFXD获取将第三方数据库迁移到RowStore的完整示例。

此命令连接到在localhost：1527上运行的RowStore网络服务器，并将数据库模式写入名为db-schema.xml的文件：

	snappy-shell write-schema-to-xml -file=db-schema.xml
此命令连接到在myserver上运行的RowStore网络服务器：1234，并将数据库模式写入名为db-schema.xml的文件：

	snappy-shell write-schema-to-xml -file=db-schema.xml -client-bind-address=myserver -client-port=1234
此命令作为对等客户端连接到运行在组播端口1234上的RowStore系统，并将数据库模式写入名为db-schema.xml的文件：

	snappy-shell write-schema-to-xml -file=db-schema.xml -mcast-port=1234
	          -extra-conn-props=host-data=false

该命令使用MySQL Connector / J连接到在“myserver”主机上运行的MySQL服务器，并将“测试”数据库模式写入文件名db-schema.xml：

	snappy-shell write-schema-to-xml -file=db-schema.xml
	          -url=jdbc:mysql://myserver/test -driver-class=com.mysql.jdbc.Driver


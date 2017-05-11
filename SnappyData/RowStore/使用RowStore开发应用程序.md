# 使用RowStore开发应用程序 #
使用RowStore开发应用程序解释了使用RowStore API和SQL的RowStore实现编程JDBC，ODBC和ADO.NET应用程序的主要概念。它描述了如何在一个Java应用程序中嵌入一个RowStore实例，以及如何作为一个客户端进行连接。本指南还描述了事务和非事务性行为，并解释了如何创建数据感知存储过程。<br/>

- **使用FabricServer接口启动RowStore服务器** FabricServer接口提供了一种在现有Java应用程序中启动嵌入式RowStore服务器进程的简单方法。

- **连接Java客户端和对等体** Java应用程序可以使用JDBC瘦客户端驱动程序连接到RowStore集群并执行SQL语句。或者，Java应用程序可以使用JDBC对等客户端驱动程序引导RowStore对等体进程，并作为集群的成员加入。

- **将RowStore配置为JDBC数据源** RowStore JDBC实现使您能够将分布式系统用作WebLogic Server等产品中的嵌入式JDBC数据源。

- **在RowStore中存储和加载JAR文件** SQL函数和过程可以使用的应用程序逻辑包括以JAR文件格式存储的Java类文件。在RowStore中存储应用程序JAR文件可简化应用程序部署，因为它减少了用户类路径问题的可能性。

- **了解数据一致性模型** 假定单个分布式系统中的所有对等体都在同一数据中心内共同配置，并且可靠带宽和低延迟可访问。分布式系统中表数据的复制一直是急切和同步的。

- **在应用程序中使用分布式事务** 事务是一组一个或多个SQL语句，构成可以提交或回滚的逻辑工作单元，并且在系统发生故障时将被恢复。RowStore分布式事务的独特设计允许线性缩放，而不会影响原子性，一致性，隔离和耐久性（ACID）属性。

- **编程数据感知过程和结果处理程序** 过程是在数据库服务器中管理的应用程序函数调用或子例程。由于多个RowStore成员在分布式系统中一起运行，所以RowStore中的过程执行也可以并行化，以同时在多个成员上运行。在多个RowStore成员上同时执行的过程称为数据感知过程。

- **编程用户定义的类型** RowStore使您能够创建用户定义的类型。用户定义的类型是一个可序列化的Java类，其实例存储在列中。

- **使用结果集和游标** 结果集维护一个游标，指向其当前的数据行。您可以使用结果集逐个逐步处理这些行。

- **使用RowStore ADO.NET组件** RowStore提供用于Visual Studio 2013或2012开发的ADO.NET客户端，以及Visual Studio 2010的ADO.NET实体框架插件。

- **ODBC驱动程序支持的配置** 本主题列出了RowStore支持的ODBC驱动程序配置。

- **使用RowStore与Hibernate** Pivotal提供了一个Hibernate方言文件，它定义了SQL语言的RowStore变体。您可以使用此文件与RowStore JDBC驱动程序配置RowStore作为开发Hibernate项目的数据库。

## 启动具有FabricServer接口的RowStore服务器 ##
FabricServer接口提供了一种在现有Java应用程序中启动嵌入式RowStore服务器进程的简单方法。<br/>

当您希望为嵌入式RowStore成员提供瘦客户端连接时，通常使用FabricServer接口。通过FabricServer接口，您可以启动多个网络服务来监听不同地址和端口组合的客户端。在启动网络服务之前，使用FabricServer接口还可以在RowStore服务器成员中初始化资源，并使该成员可用于客户端连接。

> 注意：尽管FabricServer接口支持启动嵌入式定位器服务，但生产系统应使用独立定位器作为最佳做法。请参阅**开始和停止定位器**。

**程序**

使用FabricServer接口启动RowStore服务器：

1. 使用`FabricServiceManager`工厂类获取FabricServer的单例实例。例如：

		FabricServer server = FabricServiceManager.getFabricServerInstance();


1. 创建一个Java属性对象，并添加您要在启动服务器时配置的所有引导属性定义。例如：

		Properties bootProps = new Properties();
		bootProps.setProperty("mcast-port", "12444");
或者，您可以将属性定义为系统属性（使用-D选项传递到JVM），在属性文件中配置它们，或依赖于默认属性值。**配置属性**提供更多信息。

1. 使用FabricServer.start（）方法启动服务器使用您的属性对象：
 	 	
		server.start(p);

	> 注意： RowStore在任何给定时间仅支持JVM中的单个FabricServer实例。如果使用相同的属性多次调用start（），则在后续调用中不会采取任何操作。如果您使用不同的属性多次调用start（），则默认情况下，现有的FabricServer实例首先被停止，然后重新启动新的属性。您可以选择使用“start（Properties bootProperties，boolean ignoreIfStarted）”方法使用“true”布尔值来重用上一个实例而不是重新启动它。有关更多信息，请参阅[FabricServer JavaDoc](http://rowstore.docs.snappydata.io/docs/1.4.1/javadocs/com/snappydata/rowstore/FabricServer.html)。

1. 要支持客户端连接，请使用startNetworkServer（）方法在唯一的客户端和端口组合上启动网络服务。您可以指定主机和端口号作为方法的参数。您可以在使用该方法传递的“属性”对象中指定其他网络服务器属性。例如，仅指定没有附加属性的地址和端口：

		server.startNetworkServer("localhost", 1528, null);
	
	> 注意： RowStore网络服务器支持[Derby服务器指南](http://db.apache.org/derby/docs/10.4/adminguide/tadminconfigsettingnetwrokserverproperties.html)和管理指南中记录的Apache Derby网络属性。

1. 根据需要开始附加的网络服务，以监听不同的地址和端口组合。<br/>

 	- **启动网络服务器** 客户端可以使用RowStore JDBC客户端驱动程序（“jdbc：snappydata：// <host>：<port>'形式的URL）连接到NetworkInterface。可以通过在FabricServer实例上调用startNetworkServer方法来获取网络侦听器。RowStore使用分布式关系数据库体系结构（DRDA）协议进行客户端 - 服务器通信。

	- **处理强制断开** 如果一段时间内没有响应，或者网络分区将一个或多个成员分离成太小而不能用作分布式系统的组，则RowStore成员可能会与分布式系统强制断开连接。如果您使用FabricService接口启动RowStore，则可以使用回调方法在重新连接过程中执行操作，或者在必要时取消重新连接过程。

### 启动网络服务器 ###
客户端可以使用RowStore JDBC客户端驱动程序（“jdbc：snappydata：// <host>：<port>'形式的URL）连接到NetworkInterface。可以通过在FabricServer实例上调用startNetworkServer方法来获取网络侦听器。RowStore使用分布式关系数据库体系结构（DRDA）协议进行客户端 - 服务器通信。<br/>

#### 网络服务器属性 ####
在RowStore成员上启动网络服务器时指定网络服务器属性。您必须将这些属性指定为系统属性：

- 使用该snappy-shell实用程序时，在引导RowStore服务器时，在命令行中指定网络服务器属性。

- 使用嵌入式JDBC驱动程序时，请在与RowStore成员的第一个JDBC连接中指定所有网络服务器属性。

- 使用FabricServer API时，可以独立于彼此启动FabricServer实例和NetworkListener。例如，您可以启动FabricServer实例，使用初始或默认数据填充表，然后稍后启动网络侦听器以启用客户端连接。当执行FabricService.startNetworkServer（）方法时，在“属性”对象中包含所有网络服务器属性。<br/>
如果应用程序不覆盖它们，FabricServer API将配置参数中的属性提升为等效的系统属性。<br/>

网络服务器启动属性使用前缀'snappydata.drda'。以下属性可用：

snappydata.drda.host

snappydata.drda.keepAlive

snappydata.drda.logConnections

snappydata.drda.maxThreads

snappydata.drda.minThreads

snappydata.drda.portNumber

snappydata.drda.securityMechanism

snappydata.drda.sslMode

snappydata.drda.streamOutBufferSize

snappydata.drda.timeSlice

snappydata.drda.trace

snappydata.drda.traceAll

snappydata.drda.traceDirectory


#### 例 ####
以下示例代码显示了在指定多个属性后如何启动网络侦听器。

	   import com.pivotal.snappydata.*; 
	
	  // start a fabricserver if not running already.
	   FabricServer fserver = FabricServiceManager.getFabricServerInstance(); 
	   if (fserver.status() != FabricService.State.RUNNING) { 
	     Properties props = new Properties(); 
	     props.setProperty("mcast-port", "23342"); 
	     props.setProperty("conserve-sockets", "false"); 
	     props.setProperty("log-level", "fine"); 
	     props.setProperty("host-data", "true"); 
	
	     fserver.start(props); 
	   } 
	
	  // commonly used network server  properties
	   Properties netprops = new Properties(); 
	   netprops.setProperty("snappydata.drda.minThreads", "10"); 
	   netprops.setProperty("snappydata.drda.maxThreads", "100"); 
	   netprops.setProperty("snappydata.drda.keepAlive", "1000"); 
	   netprops.setProperty("snappydata.drda.sslMode" , "off"); // Other possible values 
	                                                         // are "basic" for encryption
	                                                         // only with no client
	                                                         // authentication, and 
	                                                         // "peerAuthentication"
	                                                         // for encryption with
	                                                         // SSL clients.
	
	   // now start a network server listening on port 4343.
	   NetworkInterface netserver = fserver.startNetworkServer("localhost", 4343, netprops); 
	
	   System.out.println("started network server properties with \n" +
	       netserver.getSysinfo());

### 处理强制断电 ###
如果RowStore成员在一段时间内无响应，或者如果网络分区将一个或多个成员分离成太小而不能用作分布式系统的组，则可能会将其与分布式系统强制断开连接。如果您使用FabricService接口启动RowStore，则可以使用回调方法在重新连接过程中执行操作，或者在必要时取消重新连接过程。<br/>

在与分布式系统断开连接后，RowStore会自动关闭，然后重新启动进入“重新连接”状态，同时它会定期尝试重新加入分布式系统。如果成功重新连接，则成员将从现有成员重建其分布式系统的视图，并接收新的分布式系统ID。当定位器处于重新连接状态时，它不会为分布式系统提供发现服务。RowStore数据存储成员可以使用该`FabricService.startNetworkServer()`方法独立于FabricServer实例启动网络服务器; 作为一种最佳做法，您`FabricService.waitUntilReconnected(long, TimeUnit)`只能在该成员重新连接到分布式系统之后才能启动网络服务器。<br/>

> 注意：以“snappy-shell”实用程序或“FabricService”接口开头的成员支持自动重新连接。由于JDBC对等客户端连接关闭，因此在强制断开连接后，RowStore对等客户端不会自动重新连接到分布式系统。

默认情况下，RowStore成员将尝试重新连接到分布式文件，直到尝试的最大次数（max-num-reconnect-attempts属性）或直到被告知要停止使用该`FabricService.stopReconnecting()`方法为止。您可以使用max-wait-time-reconnect属性配置RowStore在重新连接尝试之间等待的时间。您可以通过将disable-auto-reconnect设置为“true”来完全禁用自动重新连接。<br/>

FabricService API提供了几种可以在成员重新连接到分布式系统时采取措施的方法：<br/>

- `FabricService.isReconnecting()` 如果成员尝试重新连接到分布式系统，则返回true。

- `FabricService.waitUntilReconnected(long, TimeUnit)`等待一段时间，然后返回一个布尔值，以指示该成员是否已重新连接。使用-1秒的值无限期等待直到重新连接compeletes或成员关闭。使用0秒的值作为快速探测器来确定成员是否已重新连接。

- `FabricService.stopReconnecting()` 停止重新连接过程并确保成员处于断开状态。

有关更多信息，请参阅[RowStore API](http://rowstore.docs.snappydata.io/docs/1.4.1/javadocs/index.html)参考。

## 连接Java客户端 ##
Java应用程序可以使用JDBC瘦客户端驱动程序连接到RowStore集群并执行SQL语句。或者，Java应用程序可以使用JDBC对等客户端驱动程序引导RowStore对等体进程，并作为集群的成员加入。<br/>

两个驱动程序都使用JDBC的基本连接URL `jdbc:snappydata:`：

- `jdbc:` 是协议。
- `snappydata:` 是子协议。<br/>
瘦客户机为分布式系统提供轻量级的JDBC连接。默认情况下，瘦客户机连接到现有的RowStore数据存储成员，该成员充当用于执行分布式事务的事务协调器。这提供了执行查询或DML语句的单跳或双跳访问，具体取决于数据位于系统中的位置。对于某些类型的查询，您可以为瘦客户端连接启用单跳访问，以提高查询性能。请参阅**启用单跳数据访问**。

- **瘦客户机JDBC驱动程序连接** 瘦客户端驱动程序类包装在com.pivotal.snappydata.jdbc.ClientDriver中。除了基本的JDBC连接URL之外，还可以指定瘦客户机驱动程序连接到的RowStore服务器或定位器的主机和端口号。

### 瘦客户端JDBC驱动程序连接 ###
瘦客户端驱动程序类包装在com.pivotal.snappydata.jdbc.ClientDriver中。除了基本的JDBC连接URL之外，还可以指定瘦客户机驱动程序连接到的RowStore服务器或定位器的主机和端口号。<br/>

例如：

	jdbc:snappydata://myHostName:1527/
- 代码示例
- 瘦客户端故障切换
- 启用单跳数据访问
- 管理单跳连接资源
- 瘦客户端驱动程序限制
- 配置TCP Keepalive设置

#### 代码示例 ####
此代码示例显示了在Java应用程序中连接到RowStore服务器的更完整示例：

        try {
        // 1527 is the default port that a RowStore server uses to listen for thin client connections
        Connection conn =
        DriverManager.getConnection("jdbc:snappydata://myHostName:1527/");

        // do something with the connection

        } catch (SQLException ex) {
        // handle any errors
        System.out.println("SQLException: " + ex.getMessage());
        System.out.println("SQLState: " + ex.getSQLState());
        System.out.println("VendorError: " + ex.getErrorCode());
        }

#### 瘦客户端故障切换 ####
当您使用瘦客户端连接到RowStore定位器成员（而不是直接到RowStore服务器）时，如果与分布式系统的初始连接丢失，瘦客户端驱动程序可以提供自动故障切换。见定位器。但是请注意，这假定与指定的RowStore定位器的初始连接成功。要提高建立与RowStore系统的初始连接的机会，请使用secondary-locators属性指定一个或多个辅助定位器的地址，如果无法访问主定位器，则尝试尝试。例如：

	jdbc:snappydata://myLocatorAddress:1527/;secondary-locators=mySecondaryLocatorAddress:1527,anotherSecondaryLocatorAddress:1527

#### 启用单跳数据访问 ####
默认情况下，使用瘦客户端驱动程序可以对执行查询或DML语句的表数据提供一跳或两跳访问。如果客户端的SQL语句适用于与瘦客户端连接到的RowStore成员上的数据相冲突，则可以使用单跳访问。所有其他情况导致对数据的一跳或两跳访问：首先在客户端连接的RowStore成员上评估SQL语句，如有必要，该服务器将查询路由到集群中的其他成员，托管语句的实际数据。<br/>

当使用瘦客户机连接时，RowStore提供了对特定SELECT，INSERT，UPDATE和DELETE语句的数据的单跳访问的选项。要使用此选项，当与瘦客户机驱动程序连接时，将启用单跳的连接属性设置为“true”。例如：

	jdbc:snappydata://myHostName:1527/;single-hop-enabled=true
或者，从`snappy-shell`实用程序中：

	connect client 'myHostName:1527/;single-hop-enabled=true'

> 注意：瘦客户端的单跳访问需要分布式系统中的RowStore数据存储成员运行网络服务器以进行直接客户端访问。使用网络服务器功能配置每个数据存储，即使您使用定位器进行成员发现。请参阅启动网络服务器。

单步访问仅用于准备的语句。当您在连接上启用单跳访问，然后准备准备好的语句时，本地RowStore服务器会在准备消息的响应中向其他RowStore服务器成员添加数据分发信息。当执行准备好的语句时，客户端使用添加的参数和从连接的服务器获取的信息来确定在其上可以在本地找到数据的确切的RowStore服务器; 然后它将执行指导到这些服务器。<br/>

以下类型的语句是单跳访问的好候选者：

- 在表的分区列上有一个WHERE子句的查询。
- 主键的SELECT语句，其中主键也是分区键。
- 基于分区列的IN-WHERE子句。


如果客户端无法基于WHERE子句确定数据的位置，则默认为标准的两跳执行。只有当客户端可以绝对确定语句所触及的数据位置时，才执行单跳执行。

有关单跳访问数据的限制的更多信息，请参阅瘦客户端驱动程序限制。


#### 管理单跳连接资源 ####
在内部，瘦客户机的JVM维护一个由所有准备好的语句共享的连接池，可以执行具有单跳访问的语句。对于每个单独的RowStore服务器，客户机将维护一个连接队列，该队列可以增长到由snappydata.client.single-hop-max-connections系统属性指定的最大连接数。在为特定服务器创建此最大数量的连接后，进一步的单跳执行必须等待连接在队列中可用。如果为特定RowStore服务器创建的连接数量尚未达到最大值，则客户机即时创建新连接，使用它，然后将其返回到池中。snappydata.client.single-hop-max-connections每个服务器的默认值为5个连接。

> 注意：为避免降低网络服务器的性能，请使用最小数量的并发单跳线程来满足性能要求。

请记住，您的应用程序为执行结果集创建的连接永远不会自动返回到连接池，即使您完全消耗所有返回的结果。您必须显式关闭结果集，以使此连接返回到池。

#### 瘦客户端驱动程序限制 ####
当为事务启用默认批处理模式时，RowStore可以懒洋洋地检测DML操作中的任何冲突。DML冲突可能会在事务稍后的某个时刻被系统抛出（例如，即使执行查询或在提交时）。您可以通过在分布式gemfire.tx-disable-batching系统的所有数据存储成员上将系统属性设置为“true”，来配置RowStore以立即检测操作时的冲突。<br/>

> 注意：启用`gemfire.tx-disable-batching`可能会显着降低性能。只有在对系统中的设置进行了彻底测试并确定性能折衷是必要的，以便与瘦客户端提供即时冲突检测，才能启用此选项。

如果使用瘦客户端驱动程序对没有主键的表进行插入操作，则由于故障切换（在通过定位器成员进行连接时可用）自动重试插入，可能会导致将重复的行添加到表中。<br/>

当启用单跳连接属性时，瘦客户机驱动程序具有以下限制：

- 使用事务或WAN复制时，不提供单跳访问。

> 注意：在使用事务或WAN配置时不要启用单跳访问，因为它可能导致意外的行为。

- 结合JDBC连接池的选项，使用时RowStore不支持单跳连接`dbcp`和`c3po`。

- 只有分区列具有基本数据类型的分区表才支持单跳访问：integer，long，double，decimal，char，varchar和real。如果一个表在具有任何其他数据类型（如日期，时间，时间戳，布尔值，等等）的列上进行分区，那么该表上的操作不被考虑用于单跳执行。

- 仅针对预准备语句尝试单跳执行，而不是针对简单语句。

- RowStore不支持要求将多个数据存储区的数据合并在一起的查询的单跳访问。通常，具有聚合（如MAX，MIN，AVG，ORDER BY等）的查询就像单跳被禁用一样执行。如果聚合查询可以提供单个数据存储的所有结果，则RowStore为查询提供单跳访问。

- 具有表达式解析器的表不适用于单跳访问。

#### 配置TCP Keepalive设置 ####
默认情况下，RowStore服务器使用TCP keepalive探针来帮助确定客户端何时脱机。RowStore瘦客户端还可以使用这些保持活动设置来准确地确定服务器何时脱机。相关配置属性有：

- 存活数
- 存活空闲
- 保活间隔

> 注意： Windows平台不支持“keepalive-count”的每个套接字配置。或者，您可以在某些版本的Windows中配置系统范围的“keepalive-count”值。请参阅[http://msdn.microsoft.com/en-us/library/windows/desktop/dd877220%28v=vs.85%29.aspx](http://msdn.microsoft.com/en-us/library/windows/desktop/dd877220%28v=vs.85%29.aspx)。Windows Vista和更高版本将此值固定为10。

- 注意：在r10之前的Solaris平台上，为了检测客户端的服务器故障，反之亦然，系统范围的TCP保持活动设置必须更改为较大的值（大约30秒）。请参阅[http://docs.oracle.com/cd/E19082-01/819-2724/fsvdg/index.html](http://docs.oracle.com/cd/E19082-01/819-2724/fsvdg/index.html)。这也适用于其他非Linux，非Windows平台。例如，请参阅[http://www-01.ibm.com/support/docview.wss?uid=swg21231084](http://www-01.ibm.com/support/docview.wss?uid=swg21231084)。

## 将RowStore配置为JDBC数据源 ##
RowStore JDBC实现使您能够将分布式系统用作WebLogic Server等产品中的嵌入式JDBC数据源。

### 程序 ###
在第三方产品中将RowStore设置为数据源时，请遵循以下一般步骤：



1. 将`gemfirexd-client.jar`文件复制到存储产品的其他JDBC驱动程序的位置。


1. 对于提供数据源模板的WebLogic Server等产品，如果没有明确支持RowStore，请选择“Apache Derby”，“User-defined”或“Other”作为数据库类型。


1. 指定“snappydata”作为数据库名称。这代表一个RowStore分布式系统。（RowStore不包含Apache Derby或其他关系数据库系统中的多个数据库。）

1. 对于主机名和端口，请指定RowStore定位器或RowStore服务器的主机名和端口组合。这是您从`snappy-shell`提示中用作客户端的主机名和端口组合。

1. 对于数据库用户名和密码，如果您已在系统中启用身份验证（使用该`-auth-provider`属性），请输入有效的用户名和密码组合。
如果您尚未在RowStore中配置身份验证，请将“app”指定为用户名和密码值，或任何其他临时值。

	> 注意：当您不提供数据库对象的模式名称时，RowStore将使用JDBC连接中指定的用户名作为模式名称。RowStore使用“APP”作为默认模式。如果您的系统未启用身份验证，您可以为用户名和密码指定“APP”，以保持与默认模式行为的一致性。

1. 对于driver类，请指定： `com.pivotal.snappydata.internal.jdbc.ClientConnectionPoolDataSource`

1. 您指定的JDBC URL必须以`jdbc:snappydata://`。删除任何模板属性，如`create=true`它们存在于URL或属性字段中。<br/>
在WebLogic等产品中，不能为嵌入式数据源指定JDBC连接属性，作为JDBC URL的一部分。而是使用属性字段并使用以下格式指定connectionAttributes：

		connectionAttributes=attribute;attribute;...
例如：`connectionAttributes=mcast-address=239.192.81.1;mcast-port=10334`或`connectionAttributes=locators=239.192.81.1;mcast-port=0`
另请参见用于[EmbeddedDataSource](http://db.apache.org/derby/docs/10.4/publishedapi/jdbc3/org/apache/derby/jdbc/EmbeddedDataSource.html)的[Apache Derby](http://db.apache.org/derby/docs/10.4/publishedapi/jdbc3/org/apache/derby/jdbc/EmbeddedDataSource.html) 文档。

1. 一个进程一次只能连接到一个RowStore分布式系统。如果要连接到不同的RowStore分布式系统，请在重新连接不同的数据源之前关闭当前的嵌入式数据源。

## 在RowStore中存储和加载JAR文件 ##
SQL函数和过程可以使用的应用程序逻辑包括以JAR文件格式存储的Java类文件。在RowStore中存储应用程序JAR文件可简化应用程序部署，因为它减少了用户类路径问题的可能性。<br/>

RowStore自动将安装的JAR文件类加载到类加载器中，以便您可以在RowStore应用程序和过程中使用它们。JAR类可用于RowStore分布式系统的所有成员，包括稍后加入系统的JAR类。<br/>


> 注意：本节中的许多主题已从Apache Derby文档源文件中进行了修改，并受Apache许可证的约束：根据一个或多个贡献者许可协议授予Apache Software Foundation（ASF）。有关版权所有权的其他信息，请参阅本工作分发的NOTICE文件。ASF根据Apache许可证版本2.0（“许可证”）向您授予此文件;除了符合许可证之外，您不得使用此文件。您可以通过http://www.apache.org/licenses/LICENSE-2.0除非适用法律或书面同意，根据许可证分发的软件以“按现状”分发， 无明示或暗示的任何形式的担保或条件。有关许可证下的权限和限制的特定语言，请参阅许可证。

- **类加载概述** 通过安装一个或多个JAR文件，可以在RowStore中存储应用程序类，资源和过程实现。安装JAR文件后，应用程序可以访问类，而不必以特定的方式进行编码。

- **管理JAR文件的备用方法** RowStore还提供了可用于从客户端连接交互式安装和管理JAR文件的系统过程。请注意，与使用snappy-shell命令来管理JAR文件相比，这些过程具有一定的限制。

### 班加载概况 ###
您可以通过安装一个或多个JAR文件将应用程序类，资源和过程实现存储在RowStore中。安装JAR文件后，应用程序可以访问类，而不必以特定的方式进行编码。<br/>

这些是在RowStore中存储和使用JAR文件的基本步骤。<br/>

- **为应用程序创建JAR文件** 创建用于安装的JAR文件时，请包含用于RowStore类加载的任何Java类。这可能包括存储过程或结果处理器实现以及支持类。

- **在RowStore中管理JAR文件** 安装JAR文件时，可以指定一个唯一的RowStore JAR名称，它是一个SQL92Identifier。RowStore安装JAR文件并自动将JAR文件类加载到其类加载器中，并且这些类可用于分布式系统的所有成员（包括稍后加入系统的成员）。

- **编写应用程序** 在RowStore应用程序中，您可以通过在代码中直接使用 java.lang.Class.forName间接引用它们来加载已安装的类。

#### 为您的应用程序创建JAR文件 ####
创建用于安装的JAR文件时，请包含用于RowStore类加载的任何Java类。这可能包括存储过程或结果处理器实现以及支持类。<br/>

当您将类添加到JAR文件时，请不要包括：<br/>

- 标准的Java包（java。*，javax。*）<br/>
RowStore不会阻止您存储这些标准JAR文件，但是这些类永远不会从JAR文件加载。

- Java环境提供的类（例如，sun。*）<br/>
RowStore可以从任意数量的模式从任意数量的JAR文件加载类。

创建用于RowStore类加载的JAR文件，方式与创建用户类路径中包含的JAR文件相同。例如：<br/>

	jar cf travelagent.jar travelagent/*.class
不同的IDE具有根据应用程序为JAR文件生成内容列表的工具。如果您的应用程序需要来自其他JAR文件的类，则可以使用以下选项：<br/>

- 从他们的JAR文件中提取所需的第三方类，并在JAR文件中仅包含这些类。<br/>
当您只需要第三方JAR文件中的一小部分类时，请使用此选项。

- 将整个第三方JAR文件存储在数据库中。<br/>
当您需要第三方JAR文件中的大多数或所有类时，可以使用此选项，因为您可以独立于彼此升级应用程序和第三方逻辑。

- 在用户的类路径中部署第三方JAR文件。<br/>
当用户的计算机上已经安装必要的类时，请使用此选项。

#### 在RowStore中管理JAR文件 ####
安装JAR文件时，您可以指定一个唯一的RowStore JAR名称，它是一个`SQL92Identifier`。RowStore安装JAR文件并自动将JAR文件类加载到其类加载器中，并且这些类可用于分布式系统的所有成员（包括稍后加入系统的成员）。<br/>

RowStore提供snappy-shell命令来安装，替换和删除JAR文件，如以下部分所述：<br/>

- **安装JAR文件**
- **在RowStore中管理JAR文件**
- **删除已安装的JAR文件**

##### 安装JAR文件 #####
要将JAR文件安装到已分发的RowStore中，请使用install-jar命令。例如，以下命令将tours.jar文件安装到APP模式中：

	snappy-shell install-jar -name=APP.toursjar -file=c:\tours.jar 
                 -client-bind-address=locator_address
                 -client-port=locator_port
如果RowStore分布式系统使用组播进行发现而不是定位器，请指定多播属性，如：

	snappy-shell install-jar -name=toursjar -file=c:\tours.jar 
                 -mcast-address=multicast_address
                 -mcast-port=multicast_port
在`-name`您提供的JAR必须是唯一的标识符。您必须包括一个模式名称才能对该标识符进行限定。您可以使用标识符以后调用snappy-shell replace-jar或snappy-shell remove-jar。<br/>

该`-file`选项指定使用本地路径或URL来安装的JAR文件的位置。<br/>

将JAR安装到分布式系统后，RowStore自动将JAR文件类加载到其类加载器中; 安装JAR后，您不需要显式加载类。JAR及其类可用于RowStore分布式系统的所有成员，包括稍后加入群集的成员。<br/>

> 注意：安装JAR文件后，不能修改JAR文件中的任何一个类或资源。相反，您必须替换整个JAR文件才能更新类。


##### 替换JAR文件 #####
使用replace-jar命令将已安装的JAR文件替换为新的JAR。使用命令时指定现有JAR安装的标识符，如：<br/>

	snappy-shell replace-jar -name=APP.toursjar -file=c:\tours2.jar 
                 -client-bind-address=locator_address
                 -client-port=locator_port
要替换现有的JAR文件，RowStore等待在所有数据存储区成员上获取锁定，然后加载新类。它还重新编译依赖于JAR文件的对象，例如安装的监听器和过程实现。受影响的监听器或过程的所有后续调用然后使用新的实现。<br/>

> 注意：执行命令来替换JAR文件时，JAR中现有的类可能正在被使用。例如，当您选择替换包含过程实现的JAR时，可能会执行数据感知过程。在这种情况下，`snappy-shell`等待一段时间（snappydata.max-lock-wait）来获取替换JAR文件所需的锁。如果此过程在此超时期限内未完成，则会收到错误：ERROR 40XL1：在所请求的时间内无法获取锁定。如果发生这种情况，只需重试“snappy-shell replace-jar”命令。

##### 删除已安装的JAR文件 #####
使用remove-jar命令指定JAR标识符以删除已安装的JAR文件。例如，以下命令将删除与APP.toursjar JAR安装关联的类文件：<br/>

	snappy-shell remove-jar -name=APP.toursjar 
                 -client-bind-address=locator_address
                 -client-port=locator_port

#### 编写您的应用程序 ####
在您的RowStore应用程序中，您可以通过在代码中直接使用java.lang.Class.forName直接引用它们来加载已安装的类。<br/>

您通常使用标准的java.lang.Class.getResourceAsStream加载资源，这种机制允许应用程序访问类路径中定义的资源，而不知道它们在哪里或如何存储。<br/>

您不需要对代码与RowStore及其JDBC驱动程序进行交互的方式进行任何更改。应用程序可以安全地尝试引导RowStore，即使它已经在运行，没有任何错误。应用程序以通常的方式连接到RowStore。<br/>

> 注意：不支持getResource *方法。

### 管理JAR文件的备用方法 ###
RowStore还提供了可用于从客户端连接交互式安装和管理JAR文件的系统过程。请注意，与使用snappy-shell命令来管理JAR文件相比，这些过程具有一定的限制。<br/>

本主题包含以下部分：

- **使用系统过程管理JAR文件的限制**

- **使用SQLJ.INSTALL_JAR安装JAR文件**

- **将JAR文件直接安装到SYS.JARS中**

- **用SQLJ.REPLACE_JAR替换JAR文件**

- **使用SQLJ.REMOVE_JAR删除已安装的JAR文件**

有关用于管理安装的JAR文件的过程的更多信息，请参阅过程。

#### 使用系统过程管理JAR文件的限制 ####
要使用SQLJ.INSTALL_JAR或SQLJ.REPLACE_JAR过程，JAR文件路径必须在客户端连接到的特定RowStore数据存储上可用。如果客户端直接连接到已知的RowStore服务器，则只有该服务器要求JAR文件在过程执行时在指定的路径上可用。然而，如果客户端使用定位器进行连接，那么它可能连接到分布式系统中的任何可用服务器。在这种情况下，JAR文件路径应该可用于RowStore集群的所有成员（例如，从文件共享位置，如z：\），以确保该过程可以执行。<br/>

> 注意：如果不能提供通用的JAR位置，以确保执行SQLJ.INSTALL \ _JAR或SQLJ.REPLACE \ _JAR，请使用在RowStore中管理JAR文件中描述的snappy -shell命令，或者手动将JAR文件注入RowStore系统中在将JAR文件直接安装到SYS.JARS中描述。


#### 使用SQLJ.INSTALL_JAR安装JAR文件 ####
要将JAR文件安装到RowStore，请连接到RowStore成员并执行install_jar过程。例如，以下过程从文件共享位置将tours.jar文件安装到APP模式：<br/>

	call sqlj.install_jar(
    	'z:\tours.jar', 'APP.Sample1', 0)


> 注意：最后的整型参数指定了JAR文件的别名。但是，RowStore忽略此参数，因此通常将其设置为0。

第二个参数定义了RowStore系统中JAR文件的标识符。您必须包括一个模式名称才能对该标识符进行限定。您可以稍后使用JAR标识符与SQLJ.REPLACE_JAR和SQLJ.REMOVE_JAR过程。<br/>

您可以选择为RowStore JAR名称指定引用的标识符：<br/>

	call sqlj.install_jar(
	    'z:\tours.jar', 'APP."Sample2"', 0)
安装JAR后，RowStore自动将JAR文件类加载到其类加载器中; 安装JAR后，您不需要显式加载类。JAR及其类可用于RowStore分布式系统的所有成员，包括稍后加入群集的成员。<br/>

> 注意：安装JAR文件后，不能修改JAR文件中的任何一个类或资源。相反，您必须替换整个JAR文件才能更新类。

#### 将JAR文件直接安装到SYS.JARS中 ####
如果要安装的JAR文件不适用于RowStore集群中的每个数据存储成员，请使用Java代码将JAR文件直接从客户端注入到RowStore系统中。<br/>

SYS.JARS表存储已安装的JAR文件，您可以在该表中插入一个新的JAR作为byte []值。例如，应用程序可以使用与以下内容相似的代码将JAR添加到SYS.JARS表中：<br/>

	byte[] jarBytes = getJarBytes(myjar);
	String sql = "insert into SYS.JARS values(?, ?)";
	    PreparedStatement ps = conn.prepareStatement(sql);
	    ps.setString(1, "app.sample1");
	    ps.setBytes(2, jarBytes);
	    ps.executeUpdate();
在本示例应用程序中，getJarBytes（）方法将以以下方式实现：<br/>

	public static byte[] getJarBytes(String jar) throws Exception {
	    FileInputStream fis = new FileInputStream(jar);
	    byte[] data = new byte[4096];
	    int len;
	    ByteArrayOutputStream bos = new ByteArrayOutputStream();
	    while ((len = fis.read(data)) != -1) {
	      bos.write(data, 0, len);
	    }
	    return bos.toByteArray();
	  }
RowStore自动将安装的JAR文件类加载到其类加载器中; 安装JAR后，您不需要显式加载类。

#### 用SQLJ.REPLACE_JAR替换JAR文件 ####
在RowStore成员上执行SQLJ.REPLACE_JAR过程来替换JAR文件。例如，以下过程将加载的APP.Sample1 JAR安装的JAR内容替换为newtours.jar文件的内容。

	call sqlj.replace_jar(
	    'c:\myjarfiles\newtours.jar', 'APP.Sample1');
当您替换JAR文件时，RowStore立即加载新类，而无需重新启动。

#### 使用SQLJ.REMOVE_JAR删除已安装的JAR文件 ####
在RowStore成员上执行SQLJ.REMOVE_JAR过程以删除已安装的JAR文件。例如，以下命令将删除与APP.Sample1 JAR安装关联的类文件：

	call sqlj.remove_jar(
	    'APP.Sample1', 0);

> 注意：最后的整型参数指定了JAR文件的别名。但是，RowStore忽略此参数，因此通常将其设置为0。

## 了解数据一致性模型 ##
假设单个分布式系统中的所有对等体都被共同定位在相同的数据中心，并且可靠的带宽和低延迟可以访问。分布式系统中表数据的复制一直是急切和同步的。<br/>

您可以通过在这些系统中配置WAN网关发送器和接收器来支持两个分布式系统之间的复制。<br/>

- **数据一致性概念** 没有事务（事务隔离设置为NONE），RowStore确保表更新的FIFO一致性。所有其他线程按照它们被发布的顺序看到由单个线程执行的写入，但是来自不同线程的写入可能被其他线程以不同的顺序看到。

- **非原子操作的失败方案** 由于事务之外发生的多个操作没有原子性，因此不同的RowStore成员故障情况可能导致难以诊断的数据不一致。本节将介绍一些潜在的故障情况及其后果。

- **独立线程中的DML没有排序保证RowStore** 仅保留单个执行线程应用于分布式系统（并排队到AsyncEventlisteners或远程WAN站点）的DML语句的顺序。来自多个线程的更新以先入先出（FIFO）顺序保存。否则，RowStore不提供“总订购”保证。

- **任何行的更新都是原子和隔离** 当行更新时，成员可能会收到一个部分行。RowStore始终克隆现有的行，应用更新（更改一个或多个字段），然后用更新的行原子替换行。这样可以确保读取或写入该行的所有并发线程始终保证与任何部分行更新的访问隔离。

- **批量更新的** 原子性在应用批量更新（单个更新或插入多行的DML语句）之前，RowStore不会验证所有受影响的行的约束。该设计针对这种违规行为很少的应用进行了优化。


### 数据一致性概念 ###
没有事务（事务隔离设置为NONE），RowStore确保表更新的FIFO一致性。所有其他线程按照它们被发布的顺序看到由单个线程执行的写入，但是来自不同线程的写入可能被其他线程以不同的顺序看到。<br/>

当一个表在分布式系统的成员之间进行分区时，RowStore会在承载该表的成员之间均匀分布数据集，以使单个成员成为可扩展性的瓶颈。RowStore确保在任何给定时间，单个成员拥有特定行（由主键标识）。当拥有特定行的成员失败时，该行的所有权将以一致的方式传输到备用成员，以便所有对等服务器具有新所有者的一致视图。<br/>

拥有成员负责将行更改传播到配置的副本。在将这些操作应用于副本之前，同一行的并发操作将通过拥有成员进行序列化。以这种方式，所有副本以相同的顺序查看行更新。对于分区表，RowStore确保对所有成员的给定行的所有并发修改都是原子的，并且彼此隔离，并且“配置的副本”中保留“总排序”。但是，请记住，副本成员上的行或索引的更新是非原子操作。如果更新事务外部的主行，则将受到可能的故障方案阵列的限制，如**“非原子操作故障方案”中所述**。<br/>

RowStore通过从所有者成员并行和同步传播到每个副本，在对等体之间使用一个热切的复制模型。每个副本负责处理该操作，并且响应一个确认（ACK）。只有在收到所有副本的所有ACK之后，拥有成员将返回控制权给调用方。这种方法有利于传播数据更改的数据可用性和低延迟。通过热切地传播到每个副本，可以将读取数据的客户负载平衡到任何副本。假设网络分区在实践中是罕见的，并且当它们在群集环境中发生时，应用程序生态系统通常处理许多分布式进程和应用程序，其中大部分不是为了处理分区问题而设计的。<br/>

注意：其他乐观和“最终一致”的复制方案使用旨在节省带宽的延迟复制技术。这些系统通过批处理和通过懒惰转发消息来提高吞吐量。发生冲突是通过逐步达成最终内容的协议来解决的。即使在存在网络分区的情况下，这类系统也有利于系统的可用性。但是，它会影响读取的数据一致性，或者通过从每个副本读取，使读取非常昂贵。<br/>

### 非原子操作的失败情况 ###
因为在事务之外发生的多个操作没有原子性，因此不同的RowStore成员故障情形可能会导致难以诊断的数据不一致。本节将介绍一些潜在的故障情况及其后果。<br/>



- **分区表与次级更新**。当客户端向已配置冗余的分区表发送更新操作时，RowStore首先将更新的值放在保存表的主分区的成员上。之后，插入应用于内部全局索引，该索引可能位于不同的节点上。然后将插入应用于为该分区配置的任何辅助节点。没有交易，这些操作中的每一个都会受到可能导致数据不一致的潜在故障情况的影响。<br/>
例如，更新全局索引之后但在更新辅助分区之前可能会发生故障，从而导致表与其索引之间的不一致。（由于RowStore内部全局索引已经包含该条目，因此重新尝试使用EntryExistsException进行更新以应用于次要事件）。在有机会重试该操作之前，客户端本身也可能会与一个或多个服务器失败，从而导致不一致。在任一情况下，表和索引仍然不一致，这可能导致不正确的外键检查，以便将来对表的操作。<br/>
如果此方案中的更新操作涉及到RowStore从其全局索引中删除条目，则故障可导致该条目从索引中删除，但不能从辅助分区中删除。重试该删除将导致找不到任何对象，并且删除失败，而不会出现错误。<br/>
在上述任一故障情况下，更新或删除操作也将无法通过WAN配置进行复制，导致多个RowStore集群之间的不一致以及原始集群中的不一致。<br/>

- **在非分区列上具有唯一约束的表**。对于分区表，如果在不用于分区表的列上创建唯一约束，则RowStore再次使用内部全局索引来管理列的更新。列本身和内部全局索引的更新再次被视为单独的操作，并且失败或并发操作可能导致表保持违反约束的数据。<br/>

- **表不在外键列上并置时，父/子表更新**。当父/子表不在外键列上共处时，RowStore在管理更新时再次使用内部索引。子表上的插入操作可能会在内部全局索引中查找父表的外键。如果在插入时该索引中存在父密钥，则插入到该子对象成功。但是，父表上的并发删除操作可能会在表中的子插入操作完成之前一次扫描子表，因此删除操作也将成功。这可能导致两个表都包含违反两个表之间的父子约束的值的情况。<br/>

- **批量DML操作**。具有重叠排序行的批量DML操作可以更新或删除不正确的行。请参阅**原子性批量更新**。


- 自动重试和一致性。RowStore客户端驱动程序旨在检测内部系统故障，并在任何可用的冗余副本上重试失败的操作。但是，在这些情况下，重试可能导致非幂等行为：
	- 当触发器第二次触发时，对触发器内部完成的数据的任何更改都可能会被应用两次。
	- 语句中的累计表达式（如将“x”设置为值“x + 1”）可导致列值在自动重试后递增两次。
#### 避免数据不一致的最佳做法 ####
以上每种故障情形都可能导致数据不一致性问题，可能难以检测，而且无法通过简单地重试客户端的操作来解决。始终牢记与使用非原子操作相关的潜在陷阱，并在必要时使用交易来保护数据的完整性。为了始终确保数据一致性，请始终使用REPEATABLE_READ隔离方式从事务中执行以下操作：<br/>

- 在具有唯一或外键约束的表格中对行进行插入，更新或删除操作。

- 调用触发器的操作。

- 使用累积表达式的语句（如将“x”列设置为值“x + 1”）。

- 批量DML操作。


此外，始终使用至少一个冗余副本配置分区表。

### 单独线程中DML没有订购保证 ###
RowStore仅保留单个执行线程中应用于分布式系统（并排队到AsyncEventlisteners或远程WAN站点）的DML语句的顺序。来自多个线程的更新以先入先出（FIFO）顺序保存。否则，RowStore不提供“总订购”保证。<br/>

如果多个线程同时更新复制表的相同行，或者线程从父子关系中的复制表并发更新相关行，则可能会发生数据不一致。同时更新复制表中的同一行可能导致一些副本具有与其他副本不同的值。同时删除父表中的行并在子表中插入一行可能会导致孤立行。<br/>

当DML操作排队到AsyncEventListener或远程WAN站点时，并发表访问可能会出现类似的不一致问题。例如，如果两个单独的线程分别同时更新父表和子表上的行，则RowStore将这些更新排队到AsyncEventListener或WAN网关的顺序可能与主分布式系统中更新表的顺序不一致。这可能会导致后端数据库中的外键约束冲突（例如，使用DBSynchronizer时）或在最初更新表时不会发生的远程WAN系统中。<br/>

当多个线程同时更新分区表的相同密钥时，不会出现这些类型的“乱序”更新。但是，应用程序应始终为更新多行的任何操作使用事务。<br/>

### 任何行的更新是原子和孤立的 ###
当更新行时，成员可以接收部分行。RowStore始终克隆现有的行，应用更新（更改一个或多个字段），然后用更新的行原子替换行。这样可以确保读取或写入该行的所有并发线程始终保证与任何部分行更新的访问隔离。<br/>

### 批量更新的原子性 ###
在应用批量更新（更新或插入多行的单个DML语句）之前，RowStore不会验证所有受影响的行的约束。该设计针对这种违规行为很少的应用进行了优化。<br/>

在批量更新操作期间抛出的约束违例异常（或任何其他异常）不会指示批量更新的哪一行导致违规。接收任何此类异常的应用程序无法确定批量操作中的任何行是否已成功更新。<br/>

为了解决在批量更新期间发生的约束违规或异常的可能性，应用程序应该始终在事务范围内应用批量更新。使用事务是确保所有行作为一个单元更新或回滚的唯一方法。作为替代方案，应用程序应根据主键选择要更新的行，并一次应用这些更新。<br/>

此外，不使用事务，并发DML操作（批量更新或其他）可以修改合格的行，给这两个操作提供不正确的结果。在必要时使用交易，以确保批量更新操作完成或整体回滚。<br/>

## 在应用程序中使用分布式事务 ##
事务是一组一个或多个SQL语句，构成可以提交或回滚的逻辑工作单元，并且在系统发生故障时将被恢复。RowStore分布式事务的独特设计允许线性缩放，而不会影响原子性，一致性，隔离和耐久性（ACID）属性。<br/>

- **RowStore事务设计** RowStore实现乐观的事务。事务模型是高度优化的共处理数据，其中由事务更新的所有行都由单个成员拥有。

- **RowStore分布式事务概述事务中的** 所有语句都是原子的。事务与单个连接（和数据库）关联，不能跨连接。除了提供线性缩放之外，RowStore事务设计可最大限度地减少消息传递的需求，从而使短时间的事务更有效率。

- **分布式事务的事件顺序** 以下是事件提交序列之前和期间发生的事件的逐步描述。

- **使用事务** 的最佳做法为获得最佳效果，请注意使用RowStore事务的最佳做法。

- **事务功能和限制** 请注意此版本的RowStore中的事务行为和限制。

### RowStore交易设计 ###
RowStore实现乐观的交易。事务模型是高度优化的共处理数据，其中由事务更新的所有行都由单个成员拥有。<br/>

RowStore避免使用集中式分布式锁管理器和传统的两阶段提交协议。事务状态在受事务影响的每个数据存储上进行管理，仅使用本地锁。这允许集群在应用程序利用事务时进行扩展。<br/>

当使用两阶段提交时，RowStore在后台执行第二阶段提交操作，但确保发起事务的连接只看到已提交的结果。您可以使用sync-commits属性更改此默认行为。<br/>

RowStore使用“渴望锁定，失败快速”算法，其利用更新可靠并同步地传播到所有队列（主要是副本）并行的事实。该算法的主要思路如下：<br/>

- 获得渴望的锁。每个事务写入操作被并行传播到所有副本，其中本地事务协调器获取关键字上的LOCAL写入锁定。

- 失败快 如果不能获取写锁定（大概是由于并发的冲突事务），那么写入将返回并标记事务以进行回滚。交易结果无法逆转。

- 交易状态 在事务状态中，每个事务（在事务中承载一行更改的副本的每个成员）受影响的所有成员都维护事务中的所有更改。这些更改仅在提交时在本地应用于基础表数据。这允许读取器与单个写入程序同时执行，而不需要在READ_COMMITTED隔离级别中任何锁定或阻塞。<br/>

这种设计的重点是“乐观交易”，设计使得这些重要的假设：

- 典型的交易时间短。

- 交易之间的冲突是罕见的。如果并发事务倾向于冲突，应用程序有责任重试失败的事务。

与使用集中式锁管理器的系统相比，使用此设计可提供线性缩放的潜力，并提高事务性能。没有集中的锁管理，事务吞吐量可以轻松地扩展与额外的成员。事务处理涉及数据存储加上协调对等体。如果并发事务工作负载均匀地分布在数据集中，那么增加数据存储的数量也可平衡工作负载并增加总的事务吞吐量。该设计还消除了交易工作集的托管限制，因为事务可能涉及任何数量的数据存储。<br/>

### RowStore分布式事务概述 ###
事务中的所有语句都是原子的。事务与单个连接（和数据库）关联，不能跨连接。除了提供线性缩放之外，RowStore事务设计可最大限度地减少消息传递的需求，从而使短时间的事务更有效率。<br/>

- **RowStore事务模型的主要特点**

- **交易模型如何运作**

#### RowStore事务模型的主要特点 ####
RowStore事务模型使用这些重要功能：

- 参与事务的每个RowStore成员都保持自己的事务状态。数据库上的查询总是看到提交的数据，并且不需要获取任何锁; 因此，读写可以在READ_COMMITTED隔离级别并行发生。

- 在事务写入期间，RowStore会单独锁定正在更新的每个副本的每个成员承载该行。这减轻了分布式锁管理器的需要，并且它允许更大的可扩展性。<br/>
此外，RowStore对REPEATABLE_READ和外键检查使用特殊读锁，以确保在事务持续期间这些行不会更改。

- 如果由于来自其他活动事务的并发写入而无法获取锁，则RowStore锁通常会以冲突异常（SQLState：“X0Z02”）的形式出现故障（故障快速）。<br/>
当发起事务的RowStore成员也托管事务的数据时，会发生此故障快速行为的异常。在这种情况下，由于性能原因，RowStore对本地成员进行批处理，并且在RowStore刷新批处理数据之前，直到提交之前，其他节点才可能检测到冲突。<br/>
RowStore从不批量操作SELECT ... FOR UPDATE语句。

#### 交易模型如何运作 ####
在分区表中管理数据时，每一行都由非事务性操作的单个成员隐式拥有。然而，对于分布式事务，一行的所有副本都被视为等效的，并且更新从一个“所有者”成员并行路由到所有副本。这使得分区表的事务行为与复制表的行为相似。事务管理器与RowStore成员资格管理系统紧密合作，确保无论失败或添加/删除成员，对所有行的更改都将在提交时应用到所有可用的副本，否则它们将被应用到任何一个。

> 注意： RowStore不支持为正在进行的事务向群集添加新成员。如果在事务中间添加新成员到集群，并且新成员将存储涉及事务的数据，则RowStore会隐式地回滚该事务并抛出一个SQLException（SQLState：“X0Z05”）。

RowStore中没有集中式事务协调器。相反，事务启动的成员作为事务持续时间的协调者。如果应用程序更新一行或多行，则事务协调器确定涉及哪个拥有成员，并在行的所有副本上获取本地“写入”锁。在提交时，所有更改都将应用于本地缓存和任何冗余副本。如果另一个并发事务尝试更改其中一个行，则该行的本地“写入”采集失败，该事务将自动回滚。在没有持续表的情况下，不需要向冗余成员发出两阶段的承诺; 在这种情况下，提交是高效的单相操作。<br/>

与传统的分布式数据库不同，RowStore不会使用预先提前的日志来进行事务恢复，以防在复制期间提交失败或者对一个或多个成员进行冗余更新。最可能的失败情况是成员不健康，被迫离开分布式系统，保证数据的一致性。当失败的成员重新联机时，它自动恢复复制/冗余数据集，并与其他成员建立一致性。如果某些数据的所有副本在发出提交之前都会关闭，则使用组成员身份系统检测到此条件，并且所有成员将自动回滚该事务。<br/>

- **支持的事务隔离级别** RowStore支持多个事务隔离级别。它不支持SERIALIZABLE隔离级别，嵌套事务或保存点。

- **事务和DDL语句** RowStore允许单个事务中的模式和数据操作语句（DML）。如果您在一个事务中创建表，那么也可以在同一个事务中插入数据。模式操作语句（DDL）在执行时不会自动提交，而是参与发布的事务。

- **与SELECT FOR UPDATE** 的SELECT FOR UPDATE事务隐式放置锁的语句和其他语句在事务之外不支持（默认隔离级别）。

- **回滚行为和成员失败** 在事务范围内，如果遇到约束冲突，RowStore会自动启动回滚。


### 支持的事务隔离级别 ###
RowStore支持多个事务隔离级别。它不支持SERIALIZABLE隔离级别，嵌套事务或保存点。<br/>

RowStore支持这些事务隔离级别：<br/>

- **没有**。默认情况下，与其他数据库不同，RowStore中的连接不涉及事务（请参见数据一致性概念），这对应于JDBC TRANSACTION_NONE隔离级别（或ADO.NET中的IsolationLevel.Chaos）或“SET ISOLATION RESET”SQL命令）。但是，此默认行为并不意味着没有隔离，并且连接可以从其他进程内事务中访问未提交的状态。不了解事务的默认一致性模型在**了解数据一致性模型中有所描述**。<br/>

- **READ_UNCOMMITTED**。RowStore将此隔离内部升级到READ_COMMITTED。

- **READ_COMMITTED**。RowStore确保正在进行的事务和非事务（隔离级别NONE）操作不会读取未提交（脏）数据。RowStore通过在单独的事务状态中维护事务性更改来实现此功能，仅在提交时将其应用于表的实际数据存储。在READ_COMMITTED隔离级别中，RowStore仅检测写写冲突。<br/>

- **REPEATABLE_READ**。RowStore根据ANSI SQL标准支持REPEATABLE_READ隔离级别。在REPEATABLE_READ隔离级别中，RowStore会检测到Write-Write冲突和Read-Write冲突。一次多次读取同一行的事务总是会看到行的相同列值。REPEATABLE_READ还保证表中的基础提交行在事务中首次读取后不会更改，直到事务完成（例如，它提交或中止）。<br/>
RowStore对所选数据的副本应用读写锁，以确保在事务持续时间内可重复读取。RowStore不使用范围锁，并且仍然可以使用此隔离级别进行幻像读取。<br/>
此外，使用REPEATABLE_READ隔离级别的读者保证看到分布式的原子提交。这意味着如果有一个事务在多个RowStore成员上写入并提交，那么读者可以看到分布式系统的所有成员（提交之后）的所有提交行值，或者他们将看到所有的before-committed所有成员的行值。读者从来没有看到一个成员上的一些提交的行和另一个节点上的before-committed行值（与READ_COMMITTED事务隔离一样）。为了支持此行为，RowStore对所有具有等待写入的REPEATABLE_READ事务使用两阶段提交协议。请注意，使用REAPEATABLE_READ隔离有更高的机会接收死锁，因为读者和作者之间有更大的重叠。<br/>
RowStore检测在事务期间或在为READ_COMMITTED和REPEATABLE_READ事务提交之前写入同一行的两个事务之间的冲突。但是，如果REPEATABLE_READ事务在与另一个事务所读取的同一行上写入，则RowStore会始终在写入程序提交之前（在提交的第一阶段）检测到此冲突。这使系统能够最大限度地减少读写器事务持续时间短并且事务在作者启动提交之前完成的冲突。<br/>

	> 注意：只执行读取的REPEATABLE \ _READ事务从未收到冲突。特别是，即使事务在已被其他事务标记为写入之后读取一行，则如果读写器事务尚未完成，写入程序将在提交时看到冲突。对于写入和写入的情况，如果读取器或写入程序尝试在行上的另一个写入器事务的提交进行时锁定行，则读取器等待提交完成。提交的持续时间通常较短，因此这种行为会减少冲突，并确保等待是有限的。应用程序应避免混合REPEATABLE \ _READ和READ \ _COMMITTED事务以减少潜在的冲突。

	RowStore提供了gemfire.LOCK_MAX_TIMEOUT系统属性，可用于更改READ_COMMITTED和REPEATABLE_READ事务的冲突检测行为。<br/>

	> 注意：您必须在RowStore分布式系统中的每个数据存储上将此系统属性设置为相同的值。

有关详细信息，请参阅：

- **设置隔离**

- **java.sql.Connection接口**

- **snappy -shell命令：autocommit，commit和rollback**。

### 交易和DDL语句 ###
RowStore允许单个事务中的模式和数据操作语句（DML）。如果您在一个事务中创建表，那么也可以在同一个事务中插入数据。模式操作语句（DDL）在执行时不会自动提交，而是参与发布的事务。<br/>

尽管表本身在系统中立即可见，但它会在集群中的所有成员上的系统表和受影响的表上获取独占锁，以便其他事务中的任何DML操作将阻止并等待表的锁。<br/>

例如，如果在事务中的表上创建新索引，则引用该表的所有其他事务将等待事务提交或回滚。由于这种行为，作为最佳做法，您应该将涉及DDL语句的事务保持简短（最好在单个事务中）。<br/>

### 与SELECT FOR UPDATE的事务 ###
在SELECT FOR UPDATE事务之外（默认隔离级别）不支持隐式放置锁定的语句和其他语句。<br/>

SELECT FOR UPDATE通过获取读锁开始，这允许其他事务可能获得相同数据上的读锁。在SELECT FOR UPDATE语句的行符合条件之后，事务的读取锁立即升级为独占写入锁定。此时，获取数据读取锁定的任何其他事务都会收到冲突异常，并可以回滚并释放其锁。<br/>

具有排他锁的事务只有在表上的所有其他读锁定被释放之后才能成功执行。在某些情况下，一个事务可以获取一个RowStore成员上的数据的排他锁，而另一个事务获取在不同成员上的排他锁。在这种情况下，两个事务将在提交期间失败。<br/>

### 回滚行为和成员失败 ###
在事务范围内，如果遇到约束冲突，RowStore会自动启动回滚。<br/>

解析查询时或在SQL语句中绑定参数时发生的任何错误都不会导致回滚。例如，执行SQL语句时发生的语法错误不会导致事务中先前的语句回滚。但是，列约束违规将导致事务中的所有以前的SQL操作都回滚。<br/>

#### 处理会员失败 ####

> 注意：此版本的RowStore不支持事务的透明故障切换。如果参与事务的成员在事务中或在提交期间中断，则提交将引发提交失败异常（SQLState：X0Z16），并且事务被回滚。同样，RowStore不支持将现有事务状态自动复制到加入系统的新成员。如果成员在事务中间连接分布式系统，该成员是事务中正在更新的其中一个表的数据存储，则事务将在提交时失败并回滚（SQLState：X0Z16）。

以下步骤描述可能发生的特定事件，具体取决于哪个成员发生故障，以及在事务期间发生故障时：<br/>



1. 如果交易协调员成员在提交被触发之前失败，则每个队列成员将中止正在执行的事务。


1. 如果参与的成员在提交之前失败，那么它被简单的忽略。如果某些密钥的副本/副本为零，那么这些密钥上的任何后续更新操作都会抛出异常，就像非事务更新一样。如果在此状态下触发提交，则整个事务将中止。


1. 如果交易协调者在完成提交过程（有或没有向所有队列发送提交消息）之前失败，则幸存的队列确定事务的结果。<br/>
如果所有队列处于PREPARED状态，并且成功地将更改应用于高速缓存，而不会发生任何唯一的约束违规，那么该事务将在所有队列中执行。否则，如果任何成员报告失败或最后一个副本，相关行在PREPARED状态期间关闭，则事务将在所有队列中回滚。


1. 如果参与成员在向客户承认之前失败，则交易将继续在其他成员上，不会中断。但是，如果该成员包含表或存储桶的最后一个副本，则事务将回滚。


1. 执行回滚操作时，事务协调器也可能失败。在这种情况下，瘦客户机会看到如SQLState错误那样的故障。如果客户端在事务中执行SELECT语句，则成员失败将导致SQLState错误X0Z01 ::

		ERROR X0Z01: Node 'node-name' went down or data no longer available while iterating the results (method 'rollback()'). Please retry the operation. 

	在事务上下文中执行DML语句的客户端将失败，其中一个SQLState错误：X0Z05，X0Z16，40XD2或40XD0。

	> 注意：在事务范围之外，由于成员失败，DML语句将不会看到异常。相反，该语句将在另一个RowStore成员上自动重试。但是，SELECT语句即使在事务之外也会收到X0Z01语句。

如果发生这种类型的故障，RowStore分布式系统的剩余成员将清除故障节点的打开事务，并且不需要额外的步骤来从故障中恢复。对等客户端连接将不会看到此异常，因为对等客户端本身充当事务协调器。

> 注意：在此版本的RowStore中，如果任何队列异常发生，则交易失败。


#### 其他回滚方案 ####
RowStore可能由于内存不足，超时或手动取消该语句的请求而取消执行语句（请参阅取消长时间运行的语句）。如果由于内存不足或手动取消请求而在事务上下文中执行的语句被取消，则RowStore将回滚关联的事务。请注意，如果由于超时而导致语句被取消，则RowStore不会回滚事务。<br/>

### 分布式事务的事件顺序 ###
以下是事件提交序列之前和期间发生的事件的逐步描述。<br/>

在事务提交之前，会发生以下事件：<br/>

1. 当事务启动时，事务协调器创建全局唯一的ID（TXID），并创建一个工作空间（TXState）来跟踪操作。如果事务从瘦客户端连接启动，则协调发生在与客户端连接的服务器上。请注意，事务在JDBC中隐式启动：一个事务的结束隐式地启动一个事务。

1. 事务范围内的所有更新将立即传播到所有副本并行，并在每个数据存储（队列）上部分协调。拥有事务涉及的每个成员管理本地TXState中的状态。当队列接收到更新时，它会尝试在行上获取本地写入锁。<br/>
如果行已被另一个事务锁定，则队列将立即失败。不能获得锁导致协调器隐式地回滚整个事务并释放数据主机上的所有锁。这些类型的故障可能会被触发器加剧，因为触发器是在不同的成员上执行的，并且可能尝试同时更新相同的数据行。批量更新还可以复合这些故障，因为RowStore可以在事务提交之前的任何时间（而不是在更新时对非批处理事务）进行延迟执行冲突检测。<br/>

1. 在READ_COMMITTED隔离级别中，RowStore仅检测写写冲突。它在REPEATABLE_READ隔离级别中检测写入和写入冲突。检测读写冲突发生在提交时，以减少潜在冲突的窗口。例如，如果读者在作者开始提交之前开始并结束其阅读，那么就没有冲突。此外，在大多数情况下，作者交易将收到冲突异常。<br/>
为了防止在事务提交之前在事务中获取的行被修改，RowStore支持`select for update`在选择的行被锁定之前，结果集可以被返回。<br/>

1. 当事务进行时，更新仅在数据存储上的TXState中进行维护，与其他并发连接完全隔离。

1. 立即应用对行的任何约束检查，并且失败会导致约束冲突异常。

	> 注意：在此版本的RowStore中，约束违规也会隐式回滚事务。

1. 读者通常不会获得任何锁定，除非正在读取的行即将被提交。事务读取操作首先应用于TXState，然后才能应用到已提交状态。<br/>

1. 当使用REPEATABLE_READ隔离级别时，读者也可以在受影响的行上获取读取锁。但是，这并不会立即导致与正在进行的作家的冲突。当作者尝试提交一行时，它尝试将写入锁升级到独占模式，如果保持读取锁定，则可能会导致冲突。在极少数情况下，读者可能会收到由死锁造成的冲突异常。<br/>
使用REPEATABLE_READ隔离级别保证分布式提交的原子性，因为在读取器锁定了一行以进行读取之后，任何更改行的事务都不能在整个集群中提交这些更改（或作为其他事务的一部分的关联更改）。<br/>

这些事件发生在事务提交序列期间：

1. 当协调器收到提交消息时，它将向所有队列分派单个提交消息。因为这些行已经被锁定，并且在事务期间应用了约束，所以确定事务不会由于冲突而失败。

1. 每个队列通过确保没有并发事务可以在数据存储上看到部分提交状态来保证事务原子性。但是，对于READ_COMMITTED隔离级别，提交阶段并不能保证在事务中涉及到的所有队列的情况。提交原子性仅在REPEATABLE_READ隔离级别保证，如在事务提交之前发生的事件的步骤3中所述。但是请记住，使用REAPEATABLE_READ隔离可能会有更多的接收死锁的机会，因为读者和作者之间存在更大的重叠。死锁导致一个或两个事务在超时时间后失败（请参阅gemfire.LOCK_MAX_TIMEOUT）。

1. 每个队列将TXState更改应用于内存中的表，并在确认协调器之前释放所有锁。

> 注意：提交在后台线程中执行，其他线程可能不会立即看到事务的结果。因为在提交时确保事务的结果，协调器不会在将提交的事务返回到启动线程之前等待来自队列的单个提交回复。如果相同的连接立即对相同的数据启动另一个操作，则队列在应用更改之前等待来自上一个事务的待处理响应（如步骤3所述）。此外，即使在执行提交或回滚时启动客户端进程变得不可用，也会发生提交或回滚操作。

### 使用交易的最佳做法 ###
为获得最佳结果，请注意使用RowStore事务的最佳做法。

- 为了实现高性能，可以模拟事务的持续时间以避免与其他并发事务的冲突。如果仅需要单行更新的原子性，则完全避免使用事务，因为RowStore为没有事务的单行提供原子性和隔离。

- 使用交易时，尽可能保持交易持续时间和交易涉及的行数。RowStore热切地获取锁定，持久的事务增加了冲突和事务失败的可能性。

- 与传统数据库不同，RowStore事务可能会在更新而不是提交时出现冲突异常而失败。鉴于交易的结果已经确定失败，这种选择是有道理的。<br/>
触发器可能会加剧冲突，因为触发器在不同的成员上执行，并且可能尝试同时更新相同的数据行。批量更新还可以复合这些故障，因为RowStore可以在事务提交之前的任何时间（而不是在更新时，与非批处理事务一样）可以延迟执行冲突检测。发生这种冲突时，一个或所有并发事务通常会失败并回滚。

- 在可能的范围内，建模您的数据库，以便大多数事务处理共同的数据。当所有事务数据都在单个成员上时，提供更严格的隔离保证。

- 如果您的应用程序产生多个线程或连接以处理提交的数据，请考虑将sync-commits连接属性设置为“true”。默认情况下，RowStore在后台执行第二阶段提交操作，但确保仅发布事务的连接看到完成的结果。但是，其他线程或连接可能会看到不同的结果，直到第二阶段提交动作完成。设置`sync-commits=true`确保当前的瘦客户端或对等客户端连接等待，直到所有第二阶段提交操作完成。

### 交易功能和限制 ###
记录此版本的RowStore中的事务行为和限制。<br/>

在这个版本的RowStore中，事务功能的范围是：<br/>

- 从执行查询获取的结果集应该被完全消耗，或者结果集被明确地关闭。否则，DDL操作等待，直到ResultSet被垃圾回收。

- 持久性表的事务默认启用，但是容错的全部范围尚未实现。假设在成员发生故障的情况下，至少有一行副本始终可用（冗余成员可用）。

- `select for update`在事务之外（默认隔离级别）不支持 隐式放置锁的SQL语句（例如）。

- 支持的隔离级别是“READ COMMITTED”和“READ UNCOMMITTED”，其中两者都表现为“READ COMMITTED”。与其他JDBC驱动程序不同，RowStore中Autocommit默认为OFF。

- 事务总是在操作或提交时执行“写入”冲突检测。`select for update`与其他数据库相比，应用程序不需要使用或显式锁定来获取此行为。（`select for update`在事务之外不支持）

- 不支持嵌套事务和保存点。

- RowStore不支持使用DESTROY逐出操作配置的分区表上的事务。存在这种限制，因为ACID事务的要求可能与破坏被驱逐条目的语义相冲突。例如，事务可能需要更新大于驱逐设置允许的数量的条目数。OVERFLOW evict操作支持事务，因为必要条件可以根据需要加载到内存中以支持事务语义。

- RowStore不会限制并发的非事务性客户端更新可能涉及事务的表。这是设计，当没有交易被使用时，保持非常高的性能。如果应用程序使用表上的事务，请确保应用程序在更新该表时始终使用事务。

- 单个行中的所有DML本质上都是事务内部或外部的原子。

- 当承诺集合应用于基础表时，在提交期间存在一个小窗口，并且并发查询者不会查询任何事务状态，可以看到部分承诺状态。事务越大，窗口越大。而且，事务状态保持在基于存储器的缓冲器中。事务越来越短，事务管理器在内存上运行的可能性就越小。

## 编程数据感知程序和结果处理器 ##
一个过程是在数据库服务器中管理的应用程序函数调用或子例程。由于多个RowStore成员在分布式系统中一起运行，所以RowStore中的过程执行也可以并行化，以同时在多个成员上运行。在多个RowStore成员上同时执行的过程称为数据感知过程。<br/>

> 注意： RowStore不支持在过程或函数正文中执行DDL语句。

数据感知过程使用扩展CALL语法与ON子句来指定过程执行的RowStore成员。当您调用过程时，RowStore语法提供了以下选项来并行化过程执行：

- 所有数据存储在RowStore集群中

- 数据存储的子集（属于一个或多个服务器组的成员）

- 托管表的所有数据存储成员

- 在表中托管数据子集的所有数据存储成员

RowStore将进程中的用户代码执行到数据所在的位置，这为共处理数据提供了非常低的延迟访问。（这与Hadoop中的map-reduce作业执行框架形成对比，Hadoop将数据从进程流传输到任务进程。）过程通常返回一个或多个结果集。RowStore将结果集流式传输到一个可以对结果执行缩减步骤的协调成员（通常这涉及聚合，如map-reduce）。在RowStore中，还原步骤由结果处理器执行。RowStore提供了一个默认的结果处理器，您还可以开发自己的结果处理器实现来定制还原步骤。<br/>

以下部分描述了如何在RowStore中开发，配置和调用过程和结果处理器实现。

- **使用过程提供程序API** RowStore提供了一个API来帮助您开发数据感知过程。ProcedureExecutionContext对象提供了有关用于过滤执行过程，分配数据以及有关执行过程的上下文的其他信息的表的信息。OutgoingResultSet接口使您能够通过向List对象添加行或列来构造结果集。

- **使用自定义结果处理器API** 数据感知过程使用单独的结果处理器来合并来自多个RowStore成员的过程结果。当需要结果的基本级联时，可以使用默认的RowStore结果处理器。

### 使用过程提供程序API ###
RowStore提供了一个API来帮助您开发数据感知程序。ProcedureExecutionContext对象提供了有关用于过滤执行过程，分配数据以及有关执行过程的上下文的其他信息的表的信息。OutgoingResultSet接口使您能够通过向List对象添加行或列来构造结果集。<br/>

使用本节中的信息来开发和编译过程，然后使用RowStore中的“ 存储和加载JAR文件”中的说明将该实现添加到RowStore。然后，您可以使用编程数据感知过程和结果处理器中的信息来配置和执行该过程。

- **过程参数** 使用CREATE PROCEDURE语句配置过程时，RowStore将汇编方法参数并将其传递给使用Java反射的过程实现类。

- **填充OUT和INOUT参数** 如果将其值设置在单元素数组中，则过程实现返回OUT和INOUT参数。

- **填充结果集** RowStore API提供了在过程实现中构造结果集的不同方法。

- **使用具有嵌套查询的<local>和<global> Escape语法** 嵌套在数据感知过程中的查询可以在本地成员数据上执行，也可以重新分配给RowStore集群并在多个RowStore成员上执行。可以使用<local>和<global>转义语法覆盖嵌套查询的默认行为。

- **RowStore过程的高可用性** 如果在过程已将任何行返回给客户端之前发生错误，则RowStore会自动重新执行该过程。

- **示例数据感知过程** RowStore包括可从专用客户端应用程序编译，部署和执行的数据感知过程示例。

- **配置过程** 您可以在RowStore中配置过程实现，然后才能调用该过程。

- **调用过程** RowStore使用扩展CALL语法来调用数据感知过程。

#### 过程参数 ####
当您使用CREATE PROCEDURE语句配置过程时，RowStore将汇编方法参数并将其传递给使用Java反射的过程实现类。<br/>

以下方式处理不同类型的参数：<br/>

- IN和INOUT参数作为单元素数组传递给过程实现。

- 在CREATE PROCEDURE语句中指定的动态结果集导致附加的方法参数，每个动态结果集一个。每个参数被视为一个ResultSet []类型。RowStore在每个参数中传递一个包含空值的单元素数组。<br/>
例如，如果指定DYNAMIC RESULT SETS 2，RowStore将附加两个附加的ResultSet []参数，每个参数具有单个空值。
ResultSet通过CallableStatement返回到应用程序，按照它们在过程体中定义的顺序。请参阅**填充结果集**。

- 过程实现类可以可选地指定一个ProcedureExecutionContext参数作为最后一个参数。然后，RowStore传递过程上下文对象，该对象可以用于确定有关执行上下文的信息。

> 注意： CREATE PROCEDURE和CALL语句不应该引用作为过程参数的过程上下文对象。

例如，如果程序实现的Java方法签名是：

	package com.acme.MyProc; 
	
	import java.sql.*; 
	
	public class MyProc 
	  {public static void myMethod(String inParam1, 
	                               Integer[] outParam2, 
	                               Date[] inoutParam3, 
	                               Widget[] inoutParam4, 
	                               ResultSet[] resultSet1, 
	                               ResultSet[] resultSet2) { 
	   ...
	  }
	}
您将使用类似于以下语句的RowStore中的过程进行配置：

	CREATE PROCEDURE myProc (IN inParam1 VARCHAR(10), 
	                         OUT outParam2 INTEGER, 
	                         INOUT DATE inoutParam3, 
	                         INOUT WidgetType inoutParam4)
	
	LANGUAGE JAVA 
	PARAMETER STYLE JAVA 
	READS SQL DATA 
	DYNAMIC RESULT SETS 2 
	EXTERNAL NAME 'com.acme.MyProc.myMethod'
注意：使用形式参数名称作为示例，实际参数名称不一定匹配。

> 注意：在此示例中，必须在其他地方创建用户定义的类型（WidgetType）实现。

即使Java实现在方法签名中包含一个ProcedureExecutionContext，也可以使用相同的CREATE PROCEDURE语句，如下例所示。

	package com.acme.MyProc; 
	
	import java.sql.*; 
	
	public class MyProc 
	   {public static void myMethod(String inParam1, 
	                               Integer[] outParam2, 
	                               Date[] inoutParam3, 
	                               Widget[] inoutParam4, 
	                               ResultSet[] resultSet1, 
	                               ResultSet[] resultSet2,
	                               ProcedureExecutionContext context) { 
	   ...
	  }
	}


#### 填充OUT和INOUT参数 ####
如果将其值设置在单元素数组中，则您的过程实现将返回OUT和INOUT参数。

例如，以下代码设置一个OUT和一个INOUT参数：

	outParam2[0] = 42; inoutParam3[0] = new
	        java.sql.Date(System.currentTimeMillis());

### 填充结果集 ###
RowStore API提供了在过程实现中构造结果集的不同方式。

过程必须从默认连接（jdbc：default：connection）或从ProcedureExecutionContext对象获取的连接中打开并生成ResultSet。RowStore忽略以任何其他方式生成的结果集。如果您的实现需要从瘦客户端连接或从连接到外部数据库的结果集，则创建OutgoingResultSet以填充连接中的结果。<br/>

有关在RowStore中使用结果集的详细信息，请参阅**使用结果集和游标**。<br/>

RowStore通过CallableStatement将ResultSet返回给应用程序，顺序是它们在过程体中定义。您的过程实现可以返回比DYNAMIC RESULT SETS子句中定义的更少的ResultSet; 仅构造您需要的那些ResultSet。<br/>

直接在需要这些对象的过程方法的正文中创建所有PreparedStatement或其他Statement对象。不要尝试将语句对象缓存为静态变量。与JDBC客户端应用程序方法相反，Java过程方法在完成后无法保存到JDBC对象上。此外，不要关闭生成ResultSet的语句，因为这样做会关闭ResultSet本身。<br/>

RowStore API提供了两种方法来帮助您在过程实现中构造结果集：<br/>

- **执行查询以填充结果集**

- **使用OutgoingResultSet构造结果集**

#### 执行查询以填充结果集 ####
ProcedureExecutionContext提供一个返回嵌套JDBC连接的getConnection（）方法。您使用此连接（或默认连接）使用嵌套查询填充一个或多个ResultSet。在ResultSet上调用next（）之前，不会显示嵌套查询的数据。RowStore根据需要调用ResultSet中的next（），以流式传输所需的行。<br/>

例如：

	Connection cxn = context.getConnection();
	Statement stmt = cxn.createStatement();
	resultSet1[0] = stmt.executeQuery("select * from Bar where foo > 42");
	resultSet2[0] = stmt.executeQuery("select * from Bar where foo <= 42");
RowStore创建容纳返回的ResultSet的单元素ResultSet数组。

> 注意：不要关闭用于创建结果集的连接或语句，因为这样做也会关闭结果集。

请记住，可以使用将执行限制到一个或多个RowStore成员的WHERE子句来调用数据感知过程，并且过程实现本身可能在同一个表上执行嵌套查询。默认情况下，嵌套查询仅在过程调用范围内的那些RowStore成员上执行。使用带有嵌套查询的<local>和<global> Escape语法描述了如何在过程实现需要时覆盖嵌套查询的默认范围。

#### 使用OutgoingResultSet构造结果集 ####
作为替代方法，该过程可以从ProcedureExecutionContext或默认连接获取一个空的OutgoingResultSet对象，然后为结果集的每一列调用addColumn（），后跟每行的addRow（）。如果要使用默认列名称，例如“c1”，“c2”等等，可以跳过对addColumn（）的初始调用。<br/>

当您使用此方法构造一个结果集时，即使过程实现继续添加行，RowStore也可以在调用addRow（）之后立即流输出结果。<br/>

例如：

	OutgoingResultSet rs1 = context.getOutgoingResultSet(1);
	rs1.addColumn("field1");
	rs1.addColumn("field2");
	
	for (int i = 0; i < 10; i++) {
	  rs1.addRow(new Object[i, String.valueOf(i)]);
	}
	rs1.endResults();

> 注意：不要关闭用于创建结果集的连接或语句，因为这样做也会关闭结果集。

### 使用带有嵌套查询的<local>和<global> Escape语法 ###
嵌套在数据感知过程中的查询可以在本地成员数据上执行，也可以重新分配给RowStore集群，并在多个RowStore成员上执行。可以使用<local>和<global>转义语法覆盖嵌套查询的默认行为。<br/>

如果在CALL语句中未指定WHERE子句的情况下调用了数据感知过程，则默认情况下RowStore将过程实现中的嵌套查询视为“全局”查询。这意味着嵌套查询分发到RowStore集群，并从托管表的数据的所有RowStore成员访问表数据。请注意，全局查询可以通过单个过程调用返回重复的结果，因为执行过程体的每个成员调用单独的全局查询。<br/>

如果使用WHERE子句调用过程，则默认情况下，RowStore将过程实现中的嵌套查询视为“本地”查询。这意味着查询仅访问分配给过程执行的RowStore成员上的过程的本地分区表数据。<br/>

您可以通过在过程实现中的嵌套查询字符串的开头指定“<local>”或“<global>”转义语法来覆盖此默认行为。<br/>

例如，以下嵌套查询始终与本地表数据一起运行，即使您调用该过程而不指定WHERE子句：

	Connection cxn = context.getConnection();
	Statement stmt = cxn.createStatement();
	resultSet1[0] = stmt.executeQuery("<local> select * from Bar where foo > 42");
	resultSet2[0] = stmt.executeQuery("<local> select * from Bar where foo <= 42");

> 注意：当使用ON TABLE子句调用过程时，过程主体中的任何<local>查询仅指定本地节点上分区表的主数据。对于复制表，本地查询仅访问单个副本。在这两种情况下，这避免了在过程中返回重复的值。当您使用ON ALL调用过程时，<local>查询仅执行分区表数据的本地成员。但是，针对复制表的<local>查询返回重复的结果。

### RowStore程序的高可用性 ###
如果在过程已将任何行返回给客户端之前发生错误，则RowStore会自动重新执行该过程。<br/>

过程和结果处理器实现可以使用该`isPossibleDuplicate()`方法来确定RowStore在RowStore成员失败后是否重新执行过程。这种类型的检测对于执行可能导致程序重新执行上的重复条目的写入操作的某些实现是必需的。<br/>

如果行已返回到客户端后发生错误，则RowStore会抛出异常，但不会尝试重试此过程。<br/>

### 示例数据感知过程 ###
RowStore包括一个数据感知过程示例，您可以从专用客户端应用程序进行编译，部署和执行。<br/>

如设计**RowStore数据库中所述**，数据感知过程经常用于处理多对多关系，其中分区数据不能被共同定位。在这种情况下可以使用程序执行多个查询，然后以编程方式加入数据。<br/>

在其他情况下，分区列的选择可能导致某些类型的查询执行昂贵的嵌套循环连接而不是哈希连接。使用数据感知过程可以使过程体中的多个查询的结果进行哈希。安装在/ examples / joinquerytodap目录中的示例嵌套循环连接过程显示了一个示例查询，通过使用数据感知过程可以提高性能。此示例过程使用查询结果的散列，并且还显示了如何避免在重复转换列的结果时引起的查询中过多的对象创建。有关编译，安装和运行示例的说明，请参阅本示例的README.txt文件。<br/>

### 配置过程 ###
您必须在RowStore中配置过程实现，然后才能调用该过程。<br/>

在配置过程之前，请确保在RowStore类加载器中可以使用过程实现。请参阅**在RowStore中存储和加载JAR文件**。<br/>

在RowStore中配置过程的语法如下。在调用过程时，您将一个过程指定为数据感知或数据无关。请参阅**调用过程**。<br/>

	CREATE PROCEDURE procedure-name
	([ procedure-parameter [, procedure-parameter] * ])
	LANGUAGE JAVA
	PARAMETER STYLE JAVA
	{ NO SQL | CONTAINS SQL | READS SQL DATA | MODIFIES SQL DATA }
	[ [DYNAMIC] RESULT SETS integer]
	EXTERNAL NAME 'procedure_external_class.method'
该过程的名称是您可以用它来调用RowStore程序执行一个SQL标识符。该procedure_external_name指定实际静态class_name.method_name的在Java程序实现的。**使用过程提供程序API**提供了有关实现过程的更多信息。

一个或多个过程参数条目使用语法：

	[ { IN | OUT | INOUT } ] [parameter_name] DataType
每个参数条目应该与过程Java实现中的相应参数相匹配。RowStore支持数据类型中描述的**数据类型**，包括用户定义的类型（请参阅**编程用户定义的类型**）。

程序的结果（如果有的话）可以作为OUT参数，INOUT参数或动态结果集提供。客户端使用java.sql.CallableStatement中的方法检索OUT参数。<br/>

NO SQL，CONTAINS SQL，READS SQL DATA和MODIFIES SQL DATA选项用于描述该过程使用的SQL语句的类型。选择一个可用选项：<br/>

- NO SQL表示存储过程不执行任何SQL语句。

- CONTAINS SQL表示该过程不执行读取或修改SQL数据的SQL语句。

- READS SQL DATA表示该过程不执行修改SQL数据的SQL语句，但可能会发出其他语句（如SELECT语句）来读取数据。

- MODIFIES SQL DATA指示该过程可以执行除存储过程中特别禁止的任何SQL语句。RowStore使用MODIFIES SQL DATA作为默认值。

如果过程尝试执行与NO SQL，CONTAINS SQL或MODIFIES SQL DATA设置冲突的SQL语句，RowStore将抛出异常。<br/>

“结果集”表示该过程返回结果集合的估计上限。通过调用java.sql.Statement中的getMoreResults（）和getResultSet（）语句访问结果集。<br/>

EXTERNAL NAME指定要创建的过程的完全限定类名称和方法名称。<br/>

#### 例 ####

	CREATE PROCEDURE SALES.TOTAL_REVENUE(IN S_MONTH INTEGER,
	IN S_YEAR INTEGER, OUT TOTAL DECIMAL(10,2))
	LANGUAGE JAVA PARAMETER STYLE JAVA READS SQL DATA EXTERNAL NAME 
	'com.snappydata.funcs.Revenue.calculateRevenueByMonth'

### 调用程序 ###
RowStore使用扩展CALL语法来调用数据感知过程。<br/>

用于调用过程的RowStore语法是：<br/>

	CALL procedure_name
	 ( [ expression [, expression ]* ] )
	 [ WITH RESULT PROCESSOR processor_class ]
	 [ { ON TABLE table_name [ WHERE whereClause ] }
	   |
	   { ON { ALL | SERVER GROUPS (server_group_name [, server_group_name]* ) } }
	 ]
使用可选的ON和WHERE子句为RowStore引擎提供路由提示，以便将过程执行修剪为RowStore成员的子集：<br/>

- ON TABLE执行托管表的数据的RowStore成员上的过程代码。使用分区表，您还可以使用WHERE子句来进一步仅指定表中托管特定数据值的成员。<br/>
请注意，ON TABLE和WHERE子句只用于将过程执行修剪到特定成员; 这些子句不限制数据结果，ON TABLE和WHERE子句限制都不适用于过程主体中的查询。

- ON ALL在所有RowStore成员上执行过程代码，而ON SERVER GROUPS在一个或多个命名服务器组上执行过程代码。<br/>


> 注意：指定ON TABLE，ON ALL和ON SERVER组也会影响过程实现中嵌套查询的范围。有关详细信息，请参阅填充结果集和使用带有嵌套查询的<local>和<global> Escape语法。

如果省略ON子句，RowStore会在本地协调成员上执行与数据无关的过程。<br/>

如果在调用数据感知过程时省略WITH RESULT PROCESSOR子句，则RowStore将使用默认的结果处理器实现。<br/>

如果调用的过程没有指定OUT参数或结果集，则RowStore将异步调用过程，而不等待回复。RowStore记录在过程执行期间发生的任何错误。<br/>

- **默认结果处理器** 如果WITH RESULT PROCESSOR在调用数据感知过程时省略子句，RowStore将使用默认的结果处理器实现。

- **示例JDBC客户端** 此示例显示了JDBC客户端如何调用数据感知过程并使用CallableStatement来处理过程的结果集。

#### 默认结果处理器 ####
如果`WITH RESULT PROCESSOR`在调用数据感知过程时省略子句，RowStore将使用默认的结果处理器实现。<br/>

数据感知过程的默认结果处理器对执行过程代码的每个成员的动态结果集执行无序合并。默认处理器对于声明中声明的JDBC客户端显示相同数量的`CREATE PROCEDUREResultSet`。<br/>

默认处理器处理OUT和INOUT参数，如下所示：

- 如果该类型是基本SQL类型，则默认处理器仅返回该过程返回的第一个值。

- 如果类型是JAVA_OBJECT类型，则将运行该过程的所有服务器的值作为对象数组连接在一起，并作为对象值提供给客户端。数组元素的顺序很重要，因为给定的数组位置对应于每个参数的相同RowStore成员。例如，每个阵列的第一个元素对应于由同一个RowStore成员提供的OUT参数。

#### 示例JDBC客户端 ####
此示例显示JDBC客户端如何调用数据感知过程并使用CallableStatement来处理过程的结果集。

	pre package com.pivotal.snappydata.jdbc;
	
	import java.io.Serializable; import java.sql.CallableStatement; import java.sql.Connection; import java.sql.DriverManager; import java.sql.ResultSet; import java.sql.SQLException; import java.sql.Statement; import java.sql.Types;
	
	import com.pivotal.gemfirexd.procedure.ProcedureExecutionContext;
	
	//公共类
	MyClient {
	
	public static class ExampleObj implements Serializable {private static final long serialVersionUID = 1L; //私人空间
	
	public void setValue(int val) {
	  this.
	  val = val;
	}
	
	public int getValue() {
	  return this.val;
	}
	}
	
	public static void main（String [] args）{try {Connection cxn = DriverManager.getConnection（“jdbc：snappydata：”）; Statement stmt = cxn.createStatement（）;
	
	  stmt.execute("create type ExampleObjType external name '"
	      + ExampleObj.class.getName() + "' language java");
	
	  stmt.execute("CREATE PROCEDURE myProc " + "(IN inParam1 VARCHAR(10), "
	      + " OUT outParam2 INTEGER, "
	      + " INOUT example ExampleObjType, OUT count INTEGER)"
	      + "LANGUAGE JAVA PARAMETER STYLE JAVA " + "READS SQL DATA "
	      + "DYNAMIC RESULT SETS 2 " + "EXTERNAL NAME '"
	      + ProcedureTest.class.getName() + ".myProc'");
	
	  stmt.execute("create table MyTable(x int not null, y int not null)");
	  stmt.execute("insert into MyTable values (1, 10), (2, 20), (3, 30), (4, 40)");
	  CallableStatement callableStmt = cxn
	      .prepareCall("{CALL myProc('abc', ?, ?, ?) ON TABLE MyTable WHERE x BETWEEN 5 AND 10}");
	  callableStmt.registerOutParameter(1, Types.INTEGER);
	  callableStmt.registerOutParameter(2, Types.JAVA_OBJECT);
	  callableStmt.registerOutParameter(3, Types.INTEGER);
	
	  callableStmt.setObject(2, new ExampleObj());
	
	  callableStmt.execute();
	
	  int outParam2 = callableStmt.getInt(1);
	  ExampleObj example = (ExampleObj)callableStmt.getObject(2);
	
	
	  ResultSet thisResultSet;
	  boolean moreResults = true;
	  int cnt = 0;
	  int rowCount = 0;
	  do {
	    thisResultSet = callableStmt.getResultSet();
	    int colCnt = thisResultSet.getMetaData().getColumnCount();
	      System.out.println("Result Set 1 starts");
	      while (thisResultSet.next()) {
	        for (int i = 1; i < colCnt + 1; i++) {
	          System.out.print(thisResultSet.getObject(i));
	            System.out.print(',');
	          }
	        }
	        System.out.println();
	        rowCount++;
	      }
	      System.out.println("ResultSet 1 ends\n");
	      cnt++;
	    }
	    else {
	      thisResultSet.next();
	      System.out.println("ResultSet 2 starts");
	      for (int i = 1; i < colCnt + 1; i++) {
	        cnt = thisResultSet.getInt(1);
	        System.out.print(cnt);
	        System.out.println();
	      }
	      System.out.println("ResultSet 2 ends");
	    }
	    moreResults = callableStmt.getMoreResults();
	  } while (moreResults);
	} catch (SQLException e) {
	  e.printStackTrace();
	}
	}
	
	public static void myProc（String inParam1，int [] outParam2，ExampleObj [] example，int [] count，ResultSet [] resultSet1，ResultSet [] resultSet2，ProcedureExecutionContext ctx）throws SQLException {Connection conn = ctx.getConnection（）; ExampleObj obj = new ExampleObj（）; obj.setValue（100）; example [0] = obj;
	
	  outParam2[0] = 200;
	
	  Statement stmt = conn.createStatement();
	  stmt.execute("select * from mytable");
	  resultSet1[0] = stmt.getResultSet();
	
	  Statement stmt3 = conn.createStatement();
	  stmt3 .execute("select count(*) from mytable");
	  stmt3.getResultSet().next();
	  Integer cnt = stmt3.getResultSet().getInt(1);
	  count[0] = cnt;
	
	  Statement stmt2 = conn.createStatement();
	  stmt2.execute("select count(*) from mytable");
	  resultSet2[0] = stmt2.getResultSet();
	}
	}


### 使用自定义结果处理器API ###
数据感知程序使用单独的结果处理器来合并来自多个RowStore成员的过程结果。当需要结果的基本级联时，可以使用默认的RowStore结果处理器。<br/>

**默认结果处理器**描述默认处理器的工作原理。<br/>

对于更复杂的用例，例如排序，合并或加入不同服务器的结果，您必须实现自己的结果处理器来自定义合并行为。在将结果发送给客户端之前，可以使用自定义处理器修改过程的OUT参数和结果集。<br/>

以下部分介绍如何使用RowStore结果处理器API来实现自己的自定义结果处理器。为执行基于比较的排序（合并排序）的数据感知过程结果的结果处理器提供了示例代码。<br/>

- **实现ProcedureResultProcessor接口** 自定义结果处理器必须实现RowStore ProcedureResultProcessor接口。在客户端使用WITH RESULT PROCESSOR子句调用过程后，RowStore调用处理器实现，并开始检索结果。

- **配置自定义结果处理器** 数据感知过程使用单独的结果处理器来合并来自多个RowStore成员的过程结果。您可以使用默认的RowStore结果处理器，或实现和配置您自己的结果处理器实现来自定义合并行为。

- **示例结果处理器**：MergeSort 此示例结果处理器实现使用合并排序算法将排序的结果返回给客户端。

#### 实现ProcedureResultProcessor接口 ####
自定义结果处理器必须实现RowStore ProcedureResultProcessor接口。在客户端使用WITH RESULT PROCESSOR子句调用过程后，RowStore调用处理器实现，并开始检索结果。<br/>

基本结果处理器API的操作方式如下：<br/>

1. RowStore调用实现的init（）方法来提供ProcedureProcessorContext对象。ProcedureProcessorContext提供有关过程实现和将结果返回给客户端的过程调用的重要细节。ProcedureProcessorContext描述可用于获取上下文信息并获取嵌套JDBC连接的方法。

1. 您的实现使用ProcedureProcessorContext对象来获取过程返回给客户端的OUT参数和结果集，以便它可以根据需要处理结果。OUT参数和结果集分别使用getIncomingOutParameters（）和getIncomingResultSets（）方法获得。这两个方法返回一个IncomingResultSet，您可以使用它来获取关于结果集本身的元数据，并检查结果集的各行。

1. RowStore在实现中调用getNextResultRow（）和getOutParameters（）方法来向客户端提供修改的结果。每次特定结果集的下一行必须返回给客户端时，将调用实现的getNextResultRow（）方法。getOutParameters（）应将所有过程的OUT参数返回给客户机。

1. 当关联的语句关闭并且不再需要输出处理器时，RowStore调用实现的close（）方法。免费资源，并在此方法中执行任何最终清理。


**过程结果处理器接口**提供了在ProcedureResultProcessor，ProcedureProcessorContext和IncomingResultSet中定义的方法的完整参考。

#### 配置自定义结果处理器 ####
数据感知程序使用单独的结果处理器来合并来自多个RowStore成员的过程结果。您可以使用默认的RowStore结果处理器，或实现和配置您自己的结果处理器实现来自定义合并行为。<br/>

使用过程提供程序API描述如何实现自定义结果处理器。<br/>

注意：使用自定义结果处理器时，可能会发现为结果处理器类名创建别名很有帮助。当您调用数据感知过程时，可以使用别名代替完整的结果处理器类名。要创建别名使用SQL语句： `pre CREATE ALIAS processor_alias FOR 'processor_class_name'`

其中，processor_class_name是实现com.pivotal.gemfirexd.ProcedureResultProcessor并具有默认构造函数的类的完整名称。<br/>

### 示例结果处理器：MergeSort ###
此示例结果处理器实现使用合并排序算法将排序的结果返回给客户端。

MergeSort处理器设计用于仅返回单个结果集和没有OUT参数的单个过程实现。该过程的实施示于程序执行，并且过程将在RowStore使用类似于以下内容的语句来配置。

	CREATE PROCEDURE MergeSort ()
	LANGUAGE JAVA
	PARAMETER STYLE JAVA
	READS SQL DATA
	DYNAMIC RESULT SETS 1
	EXTERNAL NAME 'examples.MergeSortProcedure.mergeSort'
您自己的结果处理器实现可能需要使用多个过程的输出，并且相应地考虑多个结果集和OUT参数的可能性。

- **过程实现** 示例结果处理器支持此过程实现。该过程使用单个结果集，而不使用OUT参数。

- **合并排序结果处理器** 此结果处理器实现对MergeSortProcedure进行排序，从而返回单个结果集。

#### 程序实施 ####
示例结果处理器支持此过程实现。该过程使用单个结果集，而不使用OUT参数。

	package examples;
	
	import com.pivotal.snappydata.*;
	import java.sql.*;
	
	public class MergeSortProcedure {
	
	  static final String LOCAL = "<local>";
	
	  public static void mergeSort(ResultSet[] outResults,
	                               ProcedureExecutionContext context)
	  throws SQLException {
	
	    String queryString = LOCAL
	                         + "SELECT * FROM "
	                         + context.getTableName();      
	    Connection cxn = context.getConnection();
	    Statement stmt = cxn.createStatement();
	    ResultSet rs = stmt.executeQuery(queryString);
	    outResults[0] = rs;
	    // Do not close the connection since this would also
	    // close the result set.
	  }
	
	}

#### 合并排序结果处理器 ####
此结果处理器实现对MergeSortProcedure进行排序，该结果返回单个结果集。

	package examples;
	
	import com.pivotal.gemfirexd.*;
	import java.sql.*;
	import java.util.*;
	
	public class MergeSortProcessor implements ProcedureResultProcessor {
	
	  private ProcedureProcessorContext context;
	
	  public void init(ProcedureProcessorContext context) {
	    this.context = context;
	  }
	
	  public Object[] getOutParameters() {
	    throw new AssertionError("this procedure has no out parameters");    
	  }
	
	  public Object[] getNextResultRow(int resultSetNumber)
	  throws InterruptedException {
	    // this procedure deals with only result set number 1
	    IncomingResultSet[] inSets = context.getIncomingResultSets(1);
	    Object[] lesserRow = null;
	    Comparator cmp = getComparator();
	
	    IncomingResultSet setWithLeastRow = null;
	    for (IncomingResultSet inSet : inSets) {
	      Object[] nextRow = inSet.waitPeekRow(); // blocks until row is available
	        // no more rows in this incoming results
	        continue;
	      }
	      // find the least row so far
	        lesserRow = nextRow;
	        setWithLeastRow = inSet;
	      }
	    }
	
	    if (setWithLeastRow != null) {
	      // consume the lesserRow by removing lesserRow from the incoming result set
	      Object[] takeRow = setWithLeastRow.takeRow();
	    }
	
	    // if lesserRow is null, then there are no more rows in any incoming results
	    return lesserRow;
	  }
	
	  public boolean getMoreResults(int nextResultSetNumber) {
	    return false; // only one result set
	  }
	
	  public void close() {
	    this.context = null;
	  }
	
	  /** Return an appropriate Comparator for sorting the rows */
	  private Comparator<Object[]> getComparator() {
	    // return an appropriate comparator for the rows
	    throw new UnsupportedOperationException("comparator not implemented");
	  }
	}

## 编程用户定义的类型 ##
RowStore使您能够创建用户定义的类型。用户定义的类型是一个可序列化的Java类，其实例存储在列中。<br/>

> 注意：本主题已从Apache Derby文档源中进行了修改，并受Apache许可证的约束： “`根据一个或多个贡献者许可协议授予Apache Software Foundation（ASF）。有关版权所有权的其他信息，请参阅本工作分发的NOTICE文件。ASF根据Apache许可证版本2.0（“许可证”）向您授予此文件;除非符合许可证，否则不得使用此文件，您可以获得许可证的副本

	     http://www.apache.org/licenses/LICENSE-2.0
	
	   Unless required by applicable law or agreed to in writing,
	   software distributed under the License is distributed on an
	   "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
	   KIND, either express or implied.  See the License for the
	   specific language governing permissions and limitations
	   under the License.
	
	</p>
	User-defined functions can be defined on columns of user-defined types. You can also use these types can as argument types for stored procedures, as described in <a href="../server-side/data-aware-procedures.html#data-aware-procedures" class="xref" title="A procedure is an application function call or subroutine that is managed in the database server. Because multiple RowStore members operate together in a distributed system, procedure execution in RowStore can also be parallelized to run on multiple members, concurrently. A procedure that executes concurrently on multiple RowStore members is called a data-aware procedure.">Programming Data-Aware Procedures and Result Processors</a>.
	
	<p class="note"><strong>Note:</strong> You cannot register a custom .NET type as a user-defined type in RowStore. </p>
	The class of a user-defined type must implement the *java.io.Serializable* interface, and it must be declared to RowStore by means of a <a href="../../../reference/language_ref/rrefsqljcreatetype.html#rrefsqljcreatetype" class="xref noPageCitation">CREATE TYPE</a> statement. You can install user-defined type classes to RowStore as part of a JAR file installation, as described in <a href="../../../deploy_guide/Topics/cdevdeploy30736.html#cdevdeploy30736" class="xref" title="Application logic, which can be used by SQL functions and procedures, includes Java class files stored in a JAR file format. Storing application JAR files in RowStore simplifies application deployment, because it reduces the potential for problems with a user&#39;s classpath.">Storing and Loading JAR Files in RowStore</a>.
	
	The key to designing a good user-defined type is to remember that data evolves over time, just like code. A good user-defined type has version information built into it. This allows the user-defined data to upgrade itself as the application changes. For this reason, it is a good idea for a user-defined type to implement *java.io.Externalizable* and not just *java.io.Serializable*. Although the SQL standard allows a Java class to implement only *java.io.Serializable*, this is bad practice for the following reasons:
	
	-   **Recompilation** - If the second version of your application is compiled on a different platform from the first version, then your serialized objects may fail to deserialize. This problem and a possible workaround are discussed in the "Version Control" section near the end of this <a href="http://java.sun.com/developer/technicalArticles/Programming/serialization/" class="xref">Serialization Primer</a> and in the last paragraph of the header comment for *java.io.Serializable*.
	-   **Evolution** - Your tools for evolving a class which simply implements *java.io.Serializable* are very limited.
	
	Fortunately, it is easy to write a version-aware UDT which implements *java.io.Serializable* and can evolve itself over time. For example, here is the first version of such a class:
	
	``` pre
	package com.acme.types;
	
	import java.io.*;
	import java.math.*;
	
	public class Price implements Externalizable
	{
	    // initial version id
	    private static final int FIRST_VERSION = 0;
	
	    public String currencyCode;
	    public BigDecimal amount;
	
	    // zero-arg constructor needed by Externalizable machinery
	    public Price() {}
	
	    public Price( String currencyCode, BigDecimal amount )
	    {
	        this.currencyCode = currencyCode;
	        this.amount = amount;
	    }
	
	    // Externalizable implementation
	    public void writeExternal(ObjectOutput out) throws IOException
	    {
	        // first write the version id
	        out.writeInt( FIRST_VERSION );
	
	        // now write the state
	        out.writeObject( currencyCode );
	        out.writeObject( amount );
	    }
	
	    public void readExternal(ObjectInput in) 
	        throws IOException, ClassNotFoundException
	    {
	        // read the version id
	        int oldVersion = in.readInt();
	        if ( oldVersion < FIRST_VERSION ) { 
	            throw new IOException( "Corrupt data stream." ); 
	        }
	        if ( oldVersion > FIRST_VERSION ) { 
	            throw new IOException( "Can't deserialize from the future." );
	        }
	
	        currencyCode = (String) in.readObject();
	        amount = (BigDecimal) in.readObject();
	    }
	}
之后，很容易编写添加新字段的用户定义类型的第二个版本。当Price从数据库读取旧版本的值时，它们会自动升级。更改以粗体显示：

	package com.acme.types;
	
	import java.io.*;
	import java.math.*;
	import java.sql.*;
	
	public class Price implements Externalizable
	{
	    // initial version id
	    private static final int FIRST_VERSION = 0;
	    private static final int TIMESTAMPED_VERSION = FIRST_VERSION + 1;
	
	    private static final Timestamp DEFAULT_TIMESTAMP = new Timestamp( 0L );
	
	    public String currencyCode;
	    public BigDecimal amount;
	    public Timestamp timeInstant;
	
	    // 0-arg constructor needed by Externalizable machinery
	    public Price() {}
	
	    public Price( String currencyCode, BigDecimal amount, 
	                  Timestamp timeInstant )
	    {
	        this.currencyCode = currencyCode;
	        this.amount = amount;
	        this.timeInstant = timeInstant;
	    }
	
	    // Externalizable implementation
	    public void writeExternal(ObjectOutput out) throws IOException
	    {
	        // first write the version id
	        out.writeInt( TIMESTAMPED_VERSION );
	
	        // now write the state
	        out.writeObject( currencyCode );
	        out.writeObject( amount );
	        out.writeObject( timeInstant );
	    }
	
	    public void readExternal(ObjectInput in) 
	        throws IOException, ClassNotFoundException
	    {
	        // read the version id
	        int oldVersion = in.readInt();
	        if ( oldVersion < FIRST_VERSION ) { 
	            throw new IOException( "Corrupt data stream." ); 
	        }
	        if ( oldVersion > TIMESTAMPED_VERSION ) {
	            throw new IOException( "Can't deserialize from the future." ); 
	        }
	
	        currencyCode = (String) in.readObject();
	        amount = (BigDecimal) in.readObject();
	
	        if ( oldVersion >= TIMESTAMPED_VERSION ) {
	            timeInstant = (Timestamp) in.readObject(); 
	        }
	        else { 
	            timeInstant = DEFAULT_TIMESTAMP; 
	        }
	    }
	}
应用程序需要在所有层级保持其代码同步。对于在客户端和服务器中运行的所有Java代码都是如此。这对于以多层次运行的功能和过程是正确的。对于以多层次运行的用户定义的类型也是如此。当客户端和服务器运行不同版本的应用程序代码时，程序员应对防御进行编码。特别地，程序员应该为用户定义的类型编写防御性序列化逻辑，以便应用程序正常地处理客户端/服务器版本不匹配。<br/>

## 使用结果集和光标 ##
结果集保持一个光标，它指向其当前的数据行。您可以使用结果集逐个逐步处理这些行。<br/>

在RowStore中，任何SELECT语句都会生成可以使用java.sql.ResultSet对象进行控制的游标。RowStore不支持SQL-92的DECLARE CURSOR语言结构来创建游标，但它支持使用可更新游标定位删除和定位更新。<br/>

> 注意：本主题已从Apache Derby文档源中进行了修改，并受Apache许可证的约束：“`根据一个或多个贡献者许可协议授予Apache Software Foundation（ASF）。有关版权所有权的其他信息，请参阅本工作分发的NOTICE文件。ASF根据Apache许可证版本2.0（“许可证”）向您授予此文件;除了符合许可证之外，您不得使用此文件。您可以通过http：//www.apache获取许可证的副本.org / licenses / LICENSE-2.0除非适用法律或书面同意，否则根据许可证分发的软件按“原样”分发，不附有任何形式的担保或条件， 明示或暗示。有关许可证下的权限和限制的特定语言，请参阅许可证。“`

- **不可更新，仅转发结果集** RowStore支持的最简单的结果集不能更新，并且具有仅向前移动的一个光标。

- **更新结果集** 通过使用结果集更新方法更新RowStore结果集（`updateRow()`，`deleteRow()`和`insertRow()`），或者通过使用定位更新或删除查询。RowStore支持可滚动和不可滚动的可更新结果集（仅向前）。

- **可滚动的不敏感结果集** JDBC提供两种类型的结果集，允许您沿任一方向滚动或将光标移动到特定行。RowStore支持以下类型之一：可滚动不敏感的结果集（`ResultSet.TYPE_SCROLL_INSENSITIVE`）。

- **结果集和自动** 提交发出提交导致连接上的所有结果集都被关闭。
### 不可更新，前向结果集 ###
RowStore支持的最简单的结果集不能被更新，并且具有仅向前移动的一个光标。<br/>

此示例摘自使用简单的SELECT语句生成结果集的JDBC应用程序，然后处理这些行。 <br/>
	
	pre Connection conn = DriverManager.getConnection( "jdbc:snappydata://myHostName:1527/"); Statement s = conn.createStatement(); s.execute("set schema 'SAMP'"); //note that autocommit is on--it is on by default in JDBC ResultSet rs = s.executeQuery( "SELECT empno, firstnme, lastname, salary, bonus, comm " + "FROM samp.employee"); /** a standard JDBC ResultSet. It maintains a * cursor that points to the current row of data. The cursor * moves down one row each time the method next() is called. * You can scroll one way only--forward--with the next() * method. When auto-commit is on, after you reach the * last row the statement is considered completed * and the transaction is committed. */ System.out.println( "last name" + "," + "first name" + ": earnings"); /* here we are scrolling through the result set with the next() method.*/ while (rs.next()) { // processing the rows String firstnme = rs.getString("FIRSTNME"); String lastName = rs.getString("LASTNAME"); BigDecimal salary = rs.getBigDecimal("SALARY"); BigDecimal bonus = rs.getBigDecimal("BONUS"); BigDecimal comm = rs.getBigDecimal("COMM"); System.out.println( lastName + ", " + firstnme + ": " + (salary.add(bonus.add(comm)))); } rs.close(); // once we've iterated through the last row, // the transaction commits automatically and releases //shared locks s.close();	

### 可更新的结果集 ###
您可以使用结果集更新方法（`updateRow()`，`deleteRow()`和`insertRow()`）或使用定位更新或删除查询来更新RowStore中的结果集。RowStore支持可滚动和不可滚动的可更新结果集（仅向前）。

- **可更新结果集的要求** 只有特定的SELECT语句 单个表的简单访问 允许您在逐步执行时更新或删除行。

- **仅向前可更新的结果集** 前向唯一可更新的结果集维护一个只能在一个方向（向前）移动的游标，还可以更新行。

- **滚动的，可更新结果集** 可滚动的，可更新的结果集维护一个游标，既可以滚动和更新行。

- **使用可更新的结果集** 插入行可以使用可更新的结果集将表插入到表中ResultSet.insertRow()。

- **可更新结果集的扩展示例**


#### 对可更新结果集的要求 ####
只有特定的SELECT语句 - 单个表的简单访问 - 允许您在逐步执行时更新或删除行。<br/>

要创建可更新的结果集，必须使用JDBC对等客户端连接来执行指定FOR UPDATE子句的查询。有关更多信息，请参阅SELECT和FOR UPDATE。<br/>

有关可更新结果集的限制的信息，请参阅产品限制。<br/>

#### 前向可更新结果集 ####
只能可更新的结果集维护可以在一个方向（向前）移动的游标，并且还更新行。

要创建前向唯一可更新的结果集，可以使用并发模式`ResultSet.CONCUR_UPDATABLE`和类型创建一个语句`ResultSet.TYPE_FORWARD_ONLY`。

> 注意：默认类型为“ResultSet.TYPE_FORWARD_ONLY”。

- **前向可更新结果集的示例**

- **变更的可见性**

- **冲突的操作**

##### 前向可更新结果集的示例 #####
使用`ResultSet.updateXXX() + ResultSet.updateRow()`更新行的示例：
	
	  Statement stmt = conn.createStatement(ResultSet.TYPE_FORWARD_ONLY, 
	                                        ResultSet.CONCUR_UPDATABLE);
	  ResultSet uprs = stmt.executeQuery(
	    "SELECT FIRSTNAME, LASTNAME, WORKDEPT, BONUS " +
	    "FROM EMPLOYEE FOR UPDATE");
	
	  while (uprs.next()) {
	      int newBonus = uprs.getInt("BONUS") + 100;
	      uprs.updateInt("BONUS", newBonus);
	      uprs.updateRow();
	  }
使用`ResultSet.deleteRow()`删除行的示例：
	
	  Statement stmt = conn.createStatement(ResultSet.TYPE_FORWARD_ONLY, 
	                                        ResultSet.CONCUR_UPDATABLE);
	  ResultSet uprs = stmt.executeQuery(
	    "SELECT FIRSTNAME, LASTNAME, WORKDEPT, BONUS " +
	    "FROM EMPLOYEE FOR UPDATE");
	
	  while (uprs.next()) {
	         uprs.deleteRow();
	      }
	  }

##### 变更的可见性 #####

- 在仅针对结果集进行更新或删除后，结果集的游标不再位于刚更新或删除的行上，而是立即在结果集中的下一行之前移动。（允许任何进一步行操作之前，必须移动到下一行。）这意味着，所做的更改`ResultSet.updateRow()`和`ResultSet.deleteRow()`从不可见。

- 如果已经插入了一行（例如，使用该行，`ResultSet.insertRow()`),则它可能在仅向前的结果集中可见。

##### 冲突的操作 #####
结果集的当前行不能被其他事务更改，因为它将被锁定与更新锁定。提交后保持打开的结果集必须移动到下一行，然后才允许对行进行任何操作。<br/>

某些冲突可能会阻止结果集进行更新和删除。例如，如果当前行被同一事务中的语句删除，则调用将`ResultSet.updateRow()`导致异常，因为游标不再位于有效的行上。<br/>

#### 可滚动，可更新的结果集 ####
可滚动，可更新的结果集维护可以滚动和更新行的游标。<br/>

RowStore仅支持可滚动不敏感结果集。要创建可更新的可滚动不敏感结果集，您可以使用并发模式`ResultSet.CONCUR_UPDATABLE`和类型创建语句`ResultSet.TYPE_SCROLL_INSENSITIVE`。<br/>


##### 可滚动，可更新的结果集的示例 #####

- 可滚动，可更新的结果集的示例

- 变更的可见性

- 冲突的操作


使用结果集更新方法更新行的示例：

	  Statement stmt = conn.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE, 
	                                        ResultSet.CONCUR_UPDATABLE);
	  ResultSet uprs = stmt.executeQuery(
	    "SELECT FIRSTNAME, LASTNAME, WORKDEPT, BONUS " +
	    "FROM EMPLOYEE FOR UPDATE");
	
	  uprs.absolute(5); // update the fifth row
	  int newBonus = uprs.getInt("BONUS") + 100;
	  uprs.updateInt("BONUS", newBonus);
	  uprs.updateRow();
使用`ResultSet.deleteRow()`删除行的示例：
	
	  Statement stmt = conn.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE, 
	                                        ResultSet.CONCUR_UPDATABLE);
	  ResultSet uprs = stmt.executeQuery(
	    "SELECT FIRSTNAME, LASTNAME, WORKDEPT, BONUS " +
	    "FROM EMPLOYEE FOR UPDATE");
	
	  uprs.last();
	  uprs.relative(-5); // moves to the 5th from the last row
	  uprs.deleteRow(); 

##### 变更的可见性 #####


- 由其他语句，触发器和其他事务（其他）引起的更改被视为其他更改，并且在可滚动不敏感的结果集中不可见。

- 在RowStore可滚动不敏感的结果集中可以看到自己的更新和删除。
	> 注意： RowStore处理使用定位更新进行的更改，并将其作为自己的更改进行删除，因此通过结果集的游标进行更改时，该更改在该结果集中也可见。

- 插入表的行可能会在结果集中显示。

- `ResultSet.rowDeleted()`如果行已使用游标或结果集删除，则返回true。它不会检测其他语句或事务所做的删除。请注意，如果底层结果集为FOR UPDATE，并且使用游标删除该行，该方法也将适用于具有并发CONCUR_READ_ONLY的结果集。

- `ResultSet.rowUpdated()`如果行已使用游标或结果集更新，则返回true。它不会检测其他语句或事务所做的更新。请注意，如果底层结果集为FOR UPDATE，并且使用游标来更新该行，该方法也将适用于具有并发CONCUR_READ_ONLY的结果集。<br/>

> 注意：无论ResultSet.rowUpdated()和ResultSet.rowDeleted()如果该行第一次被更新和删除后返回true。

即使其他人引起的更改在结果集中不可见，那么访问当前行的SQL操作（包括定位更新）也将像数据库一样读取和使用行数据，而不是反映在结果集中。


##### 冲突的操作 #####
如果行被另一个提交的事务更新/删除，或者如果行由同一个事务中的另一个语句更新，则可滚动不敏感的结果集可能会发生冲突。光标所在的行被锁定，但一旦移动到另一行，则可能会根据事务隔离级别释放该锁。这意味着可滚动不敏感结果集中的行可能在其被提取之后被其他事务更新/删除。<br/>

因为结果集不敏感，它将不会检测到他人所做的更改。当使用结果集进行更新时，将更改的列上的冲突更改将被覆盖。<br/>

冲突可能会阻止结果集进行更新/删除。如果一行在读入结果集后被删除，则可滚动不敏感的结果集会给出警告`SQLState 01001`。<br/>

为了避免与其他事务的冲突，您可以将事务隔离级别增加到可重复的读取或可序列化。这使得事务在已经读取的行上保持锁定，直到事务提交。<br/>

#### 用可更新的结果集插入行 ####
可以使用可更新的结果集将表插入到表中`ResultSet.insertRow()`。

**插入一行时，您必须为不允许空值而不具有默认值的新行中的每个列赋予一个值。如果插入的行满足查询谓词，它将在结果集中变为可见**。 使用`ResultSet.insertRow()`插入行的示例：

	  Statement stmt = conn.createStatement(ResultSet.TYPE_FORWARD_ONLY, 
	                                        ResultSet.CONCUR_UPDATABLE);
	  ResultSet uprs = stmt.executeQuery(
	    "SELECT firstname, lastname, workdept, bonus " +
	    "FROM employee FOR UPDATE");
	  uprs.moveToInsertRow();
	  uprs.updateString("FIRSTNAME", "Andreas");
	  uprs.updateString("LASTNAME", "Korneliussen");
	  uprs.updateInt("WORKDEPT", 123);
	  uprs.insertRow();
	  uprs.moveToCurrentRow();


#### 可更新结果集的扩展示例 ####
	Connection conn = DriverManager.getConnection("jdbc:snappydata://myHostName:1527/");
	conn.setAutoCommit(false);
	
	// Create the statement with concurrency mode CONCUR_UPDATABLE
	// to allow result sets to be updatable
	Statement stmt = conn.createStatement(ResultSet.TYPE_FORWARD_ONLY, 
	                                      ResultSet.CONCUR_UPDATABLE,
	                                      ResultSet.CLOSE_CURSORS_AT_COMMIT);
	// Updatable statements have some requirements
	// for example, select must be on a single table
	ResultSet uprs = stmt.executeQuery(
	    "SELECT FIRSTNME, LASTNAME, WORKDEPT, BONUS " +
	    "FROM EMPLOYEE FOR UPDATE of BONUS"); // Only bonus can be updated
	
	String theDept="E21";
	
	while (uprs.next()) {
	    String firstnme = uprs.getString("FIRSTNME");
	    String lastName = uprs.getString("LASTNAME");
	    String workDept = uprs.getString("WORKDEPT");
	    BigDecimal bonus = uprs.getBigDecimal("BONUS");
	    if (workDept.equals(theDept)) {
	        // if the current row meets our criteria,
	        // update the updatable column in the row
	        uprs.updateBigDecimal("BONUS", bonus.add(BigDecimal.valueOf(250L)));
	        uprs.updateRow();
	        System.out.println("Updating bonus for employee:" +
	                           firstnme + lastName);
	    } 
	}
	conn.commit(); // commit the transaction
	// close object 
	uprs.close();
	stmt.close();
	// Close connection if the application does not need it any more
	conn.close();

### 可滚动不敏感结果集 ###
JDBC提供两种类型的结果集，允许您沿任一方向滚动或将光标移动到特定行。RowStore支持以下类型之一：可滚动不敏感的结果集（`ResultSet.TYPE_SCROLL_INSENSITIVE`）。<br/>

当您使用类型类型的结果集时`ResultSet.TYPE_SCROLL_INSENSITIVE`，RowStore会将结果集中的第一行的行实例化为具有最大行数的行，因为请求行。如果需要，物化行将被备份到磁盘，以避免过多的内存使用。<br/>

与敏感结果集相比，不敏感的结果集不能看到其他人对已实现的行进行的更改。RowStore允许更新可滚动的不敏感结果集，如可见性更改中所述。（RowStore还允许更新只向前的结果集。）<br/>

> 注意： RowStore不支持“ResultSet.TYPE_SCROLL_SENSITIVE”类型的结果集。

	//autocommit does not have to be off because even if 
	//we accidentally scroll past the last row, the implicit commit
	//on the the statement will not close the result set because result sets
	//are held over commit by default
	conn.setAutoCommit(false);
	Statement s4 = conn.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE, 
	                                    ResultSet.CONCUR_READ_ONLY);
	s4.execute("set schema 'SAMP'");
	ResultSet scroller=s4.executeQuery(
	    "SELECT sales_person, region, sales FROM sales " +
	    "WHERE sales > 8 ORDER BY sales DESC");
	if (scroller.first()) { // One row is now materialized
	    System.out.println("The sales rep who sold the highest number" +
	                       " of sales is " +
	                       scroller.getString("SALES_PERSON"));
	} else {
	    System.out.println("There are no rows.");
	}
	scroller.beforeFirst();
	scroller.afterLast();   // By calling afterlast(), all rows will be materialized
	scroller.absolute(3);
	if (!scroller.isAfterLast()) {
	    System.out.println("The employee with the third highest number " +
	                       "of sales is " +
	                       scroller.getString("SALES_PERSON") + ", with " +
	                       scroller.getInt("SALES") + " sales");
	}
	if (scroller.isLast()) {
	    System.out.println("There are only three rows.");
	}
	if (scroller.last()) {
	    System.out.println("The least highest number " +
	                       "of sales of the top three sales is: " +
	                       scroller.getInt("SALES"));
	}
	scroller.close();
	s4.close();
	conn.commit()
	conn.close();
	System.out.println("Closed connection");

### 结果集和自动提交 ###
发出提交将导致您的连接上的所有结果集都被关闭。<br/>

在可更新结果集上使用更新方法时，JDBC应用程序不需要自动提交。但是，定位更新和删除不能与自动提交组合使用。<br/>

## 使用RowStore ADO.NET组件 ##
RowStore提供了一个用于Visual Studio 2013或2012开发的ADO.NET客户端，以及用于Visual Studio 2010的ADO.NET实体框架插件。<br/>

- **使用RowStore ADO.NET客户端** RowStore 1.4.1 ADO.NET驱动程序为Windows 32位和64位平台提供了一个本地线路协议数据提供程序。ADO.NET客户端支持使用Visual Studio 2013或Visual Studio 2012进行开发。您可以使用客户端将RowStore数据库作为数据源添加到Visual Studio中的“服务器资源管理器”列表中，以使用查询构建器等进行设计查询。
- **使用RowStore.NET实体框架插件** 使用RowStore.NET实体框架从现有模式生成模型对象，或从模型对象生成模式。您还可以使用语言集成查询（LINQ）发出查询，然后使用C＃或VB.NET检索和操作数据为强类型对象。实体框架插件提供对Visual Studio 2010的支持。

### 使用RowStore ADO.NET客户端 ###
RowStore 1.4.1 ADO.NET驱动程序为Windows 32位和64位平台提供了一个本地线路协议数据提供程序。ADO.NET客户端支持使用Visual Studio 2013或Visual Studio 2012进行开发。您可以使用客户端将RowStore数据库作为数据源添加到Visual Studio中的“服务器资源管理器”列表中，以使用查询构建器等进行设计查询。<br/>


#### 安装ADO.NET客户端软件 ####
RowStore ADO.NET和ODBC驱动程序作为从Pivotal Network的组合下载提供。要安装ADO.NET驱动程序：<br/>



1. 下载并解压缩[RowStore ODBC Distribution 1.4.1](https://network.pivotal.io/products/snappystore)。


1. 查看“自述文件”文件和安装指南，了解ADO.NET驱动程序软件和安装过程：
	- 该path_to_install_location \ SnappyData_RowStore_XX_bNNNNN_ODBC \ ADONET \ dotnetreadme.txt描述限制与ADO.NET驱动程序的当前版本，并提供了安装的文件和文件夹的摘要。
	- 该path_to_install_location \ SnappyData_RowStore_XX_bNNNNN_ODBC \ ADONET \文档\ dotnetig.pdf文件包含安装ADO.NET驱动程序的完整文档。


1. 执行path_to_install_location \ SnappyData_RowStore_XX_bNNNN_ODBC \ adonet \ Setup.exe应用程序来安装该软件。有关进一步说明，请参阅安装指南。

#### 配置和使用ADO.NET客户端 ####
ADO.NET Client安装程序安装使用该软件所需的所有文档和示例代码。完成安装过程后，请参阅以下资源以获取有关使用驱动程序的信息。

|开始菜单项|描述|
|:--|:--|
|开始>所有程序> RowStore for ADO.NET 4.2>用于RowStore帮助的ADO.NET客户端|交互式帮助系统，为Pivotal.RowStore命名空间中的所有对象提供引用信息。|
|开始>所有程序> RowStore for ADO.NET 4.2> ADO.NET客户端的RowStore在线图书|提供有关配置ADO.NET客户端软件以及在Visual Studio中使用客户端的详细信息。|
|开始>所有程序> RowStore for ADO.NET 4.2>用于GemFire自述的ADO.NET客户端|提供安装的文件和文件夹的摘要，并描述与当前版本的ADO.NET驱动程序相关的任何限制。|

：表1. RowStore ADO.NET客户端文档

#### 使用RowStore.NET实体框架插件 ####
使用RowStore.NET实体框架从现有模式生成模型对象，或从模型对象生成模式。您还可以使用语言集成查询（LINQ）发出查询，然后使用C＃或VB.NET检索和操作数据为强类型对象。实体框架插件提供对Visual Studio 2010的支持。<br/>

- **安装RowStore ADO.NET实体框架插件** RowStore安装提供了安装和注册Entity Framework插件的安装程序。

- **从RowStore模式生成新模型** 实体框架插件使您能够从现有的RowStore模式生成身份对象。

- **调用RowStore过程和函数** RowStore.NET实体框架使您能够通过将RowStore存储过程映射到每个操作来插入，更新或删除实体的值。您还可以创建调用RowStore过程或函数的实体框架导入函数，以执行各种操作。调用RowStore代码的导入函数可能会返回实体值，标量值，或者可能没有返回值，而是执行一些其他操作，例如更新或删除列值。

- **从模型对象生成新的RowStore模式** 在模型第一种方法中，您可以使用实体数据模型工具创建一个空的.edmx文件，然后创建模型所需的实体，关系和继承层次结构。从模型生成数据库提供了可以在RowStore中执行以创建实际模式的DDL命令。
- **查询或修改实体** 您可以使用实体框架中的ObjectContext对象来查询，插入，删除和更新RowStore数据。

##### 安装RowStore ADO.NET实体框架插件 #####

RowStore安装提供了一个安装程序来安装和注册Entity Framework插件。


###### 先决条件 ######
在安装或使用RowStore.NET实体框架插件之前，您必须安装：

- .NET Framework 4.0

- 带有Service Pack 1的Microsoft Visual Studio 2010

- 用于KB2561001 的Microsoft Visual Studio 修补程序。

> 注意：修复实体的StoreGeneratedPattern属性值时，需要此修补程序来修复实体设计器中可能发生的潜在的数据损坏问题。有关详细信息，请参阅[Microsoft支持文章](http://support.microsoft.com/kb/2561001) 2561001。应用修补程序后，Visual Studio的版本字符串（在Microsoft.data.entity.design.dll文件的属性表中查看）10.0.40219.335。有关此修补程序的相关信息，请参阅[RowStore发行说明](http://rowstore.docs.snappydata.io/relnotes)。


###### 程序 ######
要安装RowStore.NET实体框架插件：

1. 转到RowStore安装的\ dotnet子目录。（例如，c：\ snappystore-dir \ dotnet）。

1. 双击GemFire_XD_EntityFramework_X.XXmsi文件。（例如，GemFire_XD_EntityFramework_1.4.1.msi。）

1. 单击**下一步**开始安装。

1. 选择安装目录或接受默认值，然后单击**下一步**。

1. 单击**下一步**以确认您的选择。安装程序开始安装文件，并提示您为可用环境安装RowStore设计时支持：


	![](http://rowstore.docs.snappydata.io/docs/common/images/store-net-install.png)

1. 选择Visual Studio 2010环境，然后单击**关闭**。
1. 安装完成后，单击**关闭**退出安装程序。

#### 从RowStore模式生成新模型 ####
实体框架插件使您能够从现有的RowStore模式生成身份对象。

如果您已经在RowStore中创建了一个或多个数据库模式，则可以使用ADO.NET实体框架插件来使用现有模式中可用的表，视图，过程和函数生成模型对象。要生成模型对象，首先创建一个新的Windows窗体应用程序，然后使用与RowStore的连接添加实体数据模型。

> 注意：在Visual Studio中工作时，请记住，Visual Studio工具中的“数据库”一词与RowStore中的术语“模式”相对应。

> 注意：从RowStore模式生成模型后，无法根据RowStore中模式发生的进一步更改来更新或刷新模型。相反，您必须通过重复此过程从更新的模式生成新模型。实体框架插件支持导入现有表和视图定义，但不支持存储过程。

##### 创建Windows窗体应用程序 #####

1. 启动带有Service Pack 1的Microsoft Visual Studio 2010。

1. 从主菜单中选择**文件>新建>项目...**。

1. 选择已安装的名为Windows Forms Application的模板，然后单击“**确定**”创建新的解决方案。

1. 按照下一节中的步骤将实体数据模型添加到解决方案中。
##### 添加实体数据模型 #####
1. 在解决方案资源管理器面板中，右键单击Windows窗体应用程序，然后选择**添加>新建项目...**

1. 从安装的模板列表中选择**ADO.NET实体数据模型**。

1. 单击**添加**以显示Visual Studio显示实体数据模型向导。您将使用此向导生成新模型：
	
	![](http://rowstore.docs.snappydata.io/docs/common/images/store-net-generate-from-db.png)
1. 选择**从数据库生成**从选择模型内容窗口，然后单击**下一步**。

1. 在“选择数据连接”窗口中：

	a.点击**新建连接...**<br/>
	b.单击**更改**...更改数据源。<br/>
	c.在数据源：列表中单击<其他>。<br/>
	d.从Data Provider下拉列表中选择**RowStore的.NET Framework数据提供者**：<br/>

	![](http://rowstore.docs.snappydata.io/docs/common/images/store-net-choose-provider.png)

	e.单击**确定**以选择提供程序。现在在“连接属性”对话框中选择了RowStore提供程序：

	![](http://rowstore.docs.snappydata.io/docs/common/images/store-net-connection.png)

	f.在“ **服务器**”字段中，输入要用于连接到分布式系统的RowStore定位器或服务器的地址和端口号（以冒号分隔）。例如，上面的屏幕截图显示了与localhost的连接：1527。单击测试连接以验证是否可以连接，然后单击确定以关闭测试连接和连接属性对话框。<br/>
	g.**在App.Config as**：对话框中选择**保存实体连接设置**，并提供设置的名称：

	![](http://rowstore.docs.snappydata.io/docs/common/images/store-net-connection-complete.png)

	h.单击**下一步**继续。<br/>
	实体数据模型向导连接到RowStore系统，并以默认模式显示数据库对象的树结构：

	![](http://rowstore.docs.snappydata.io/docs/common/images/store-choose-objects.png)


1. 选择要包含在模型中的数据库表，视图和存储过程，然后单击完成。

	> 注意：实体框架插件支持导入现有的存储过程和函数以及表和视图。为了使用导入模型的现有过程和函数，必须在导入完整模式后将它们映射到特定实体对象。见XREF。

	Visual Studio生成所选对象和关系的模型：

	![](http://rowstore.docs.snappydata.io/docs/common/images/store-generated-model.png)

1. 在开始修改模型之前，从Visual Studio主菜单中选择Build> Build Solution，以确保模型编译没有错误。
1. 如果您选择导入存储过程，则实体数据模型向导会为RowStore模式中的存储过程和函数导入元数据。要使用导入的过程和函数，必须首先将它们映射为使用设计曲面的实体的函数。见XREF。
1. 您可以使用设计表面和/或工具箱修改模型，添加实体等。要使用新模型更新RowStore模式，必须首先删除RowStore中的所有数据库对象，然后使用从Visual Studio 2010 生成的 DDL命令重新创建它们。请参阅**从模型对象生成新的RowStore模式**以获取说明。

> 注意：从RowStore模式生成模型后，无法根据RowStore中模式发生的进一步更改来更新或刷新模型。相反，您必须使用上一个过程从更新的模式生成新模型。

#### 调用RowStore过程和函数 ####
RowStore.NET实体框架允许您通过将RowStore存储过程映射到每个操作来插入，更新或删除实体的值。您还可以创建调用RowStore过程或函数的实体框架导入函数，以执行各种操作。调用RowStore代码的导入函数可能会返回实体值，标量值，或者可能没有返回值，而是执行一些其他操作，例如更新或删除列值。<br/>

> 注意：存储过程和函数都在实体框架中映射为“函数”。

- **程序映射的工作原理**

- **插入，更新和删除操作的映射过程**

- **使用导入功能执行存储过程和功能**

##### 程序映射的工作原理 #####
您可以使用RowStore存储过程对实体执行插入，更新和删除操作。当您映射RowStore过程以执行这些操作时，Entity Framework自动使用过程调用（提供实体值来插入，更新或删除），而不是使用关联的SQL语句。例如，对于实体上的INSERT操作，实体框架通常会在创建新的实体对象并`ObjectContex`t保存之后生成下列代码：<br/>

	INSERT INTO ENTITYNAME(ENTITYFIELD) VALUES('NewValue'); SELECT ID FROM ENTITYNAME WHERE ID = ?;
如果您改为使用RowStore过程来执行这些操作，则生成的INSERT操作代码将与以下类似：

	CALL INSERTProcForENTITYNAME('NewValue');
另请参阅**查询或修改实体**。

> 注意：如果为任何INSERT，OPERATE或DELETE操作指定映射过程，则必须对所有三个操作使用映射过程。即使您只将过程映射到单个操作，实体框架将自动为所有三个操作生成过程调用。


##### 插入，更新和删除操作的映射过程 #####
按照以下步骤将插入，更新和删除映射到RowStore存储过程：

1. 完成**从RowStore Schema生成新模型**以导入RowStore模式的步骤。确保您选择导入存储过程以及数据库表和视图。

1. 右键单击设计曲面中的实体，然后选择“ **存储过程映射**”。这将在窗口底部显示实体的映射细节：

	![](http://rowstore.docs.snappydata.io/docs/common/images/snappystore-dotnet-entity-function.png)

1. 您必须配置每个功能（插入，更新和删除）以使用正确的过程，并将每个过程参数映射到正确的实体属性：

	a.单击参数/列下的每个条目，然后单击相关的下拉菜单。选择要为相关操作调用的导入的RowStore存储过程。<br/>
	b.对于每个过程的“IN”参数，请指定实体框架应提供的相应实体属性作为参数值。确保该过程接受兼容的数据类型。（请参阅配置和使用ADO.NET客户端。）<br/>
	c.对于某些列，例如提供时间戳的列，您可以选择使用原始值复选框。选择此选项确保实体框架与插入或更新实体时使用的过程相同的计算值。如果不选中此选项，框架将在调用配置的过程时计算新值。<br/>

##### 使用导入功能执行存储过程和功能 #####

导入功能使您能够执行RowStore中可执行各种操作的存储过程或函数。可以使用一个过程来返回不属于模型的实体，返回标量值或返回一些其他复杂类型。此外，过程不需要返回任何值，而是可以用于更新或删除数据库中的值或执行某些其他功能。<br/>

要创建新的导入功能：<br/>

1. 完成**从RowStore Schema生成新模型**以导入RowStore模式的步骤。确保您选择导入存储过程以及数据库表和视图。

1. 在“模型浏览器”窗口中，展开Model.Store节点，然后展开“存储过程”节点。右键单击所需的存储过程的名称，然后选择**添加功能导入...**。

	![](http://rowstore.docs.snappydata.io/docs/common/images/ef-add-function-import.png)

	这将显示“添加功能导入...”窗口：

	![](http://rowstore.docs.snappydata.io/docs/common/images/ef-function-import-window.png)



1. 为功能或过程选择适当的返回类型，然后单击**确定**：
	- 如果过程执行某些功能（例如插入或删除），则 选择“ **无**”，但不返回任何值。
	- 如果过程从表中返回一行或多行作为返回类型， 请选择**实体**。然后从与过程输出匹配的下拉列表中选择命名实体。
	- 选择**标量**如果程序返回标值作为返回类型。然后选择匹配的数据类型。请参阅**配置和使用ADO.NET客户端**。

#### 从模型对象生成新的RowStore模式 ####
在模型第一种方法中，您可以使用实体数据模型工具创建一个空的.edmx文件，然后创建模型所需的实体，关系和继承层次结构。从模型生成数据库提供了可以在RowStore中执行以创建实际模式的DDL命令。

##### 程序 #####
1. 在解决方案资源管理器中，右键单击Windows窗体应用程序，然后选择**添加>新建项目...**

1. 从Visual Studio安装的模板中选择**ADO.NET实体数据模型**。

1. 单击**添加**以显示Visual Studio显示实体数据模型向导。您将使用此向导生成新模型。

1. 从“选择模型内容”窗口中选择“ **空模型** ”，然后单击“**完成**”。

1. 使用工具箱或右键单击设计曲面以创建要包含在模型中的实体和关联。例如：

	![](http://rowstore.docs.snappydata.io/docs/common/images/store-new-entities.png)

1. 在为您的模型生成DDL之前，请务必从模型“属性”面板中的DDL生成模板菜单中选择SSDLToRowStore.tt（VS）选项：

	![](http://rowstore.docs.snappydata.io/docs/common/images/store-ddl-gen-template.png)


1. 右键单击设计器以显示上下文菜单，然后从模型...中选择生成数据库。这将打开一个生成数据库向导。

1. 在“选择数据连接”窗口中：

	a.点击新建连接...<br/>
	b.单击更改...更改数据源。<br/>
	c.在数据源：列表中单击<其他>。<br/>
	d.从Data Provider下拉列表中选择RowStore的.NET Framework数据提供者：<br/>

	![](http://rowstore.docs.snappydata.io/docs/common/images/store-net-choose-provider.png)

	e.单击**确定**以选择提供程序。现在在“连接属性”对话框中选择了RowStore提供程序：

	![](http://rowstore.docs.snappydata.io/docs/common/images/store-net-connection.png)

	f.在“ **服务器**”字段中，输入要用于连接到分布式系统的RowStore定位器或服务器的地址和端口号（以冒号分隔）。例如，上面的屏幕截图显示了与localhost的连接：1527。单击**	**以验证是否可以连接，然后单击**确定**以关闭测试连接和连接属性对话框。<br/>
	g.**在App.Config as**：对话框中选择**保存实体连接设置**，并提供设置的名称：

	![](http://rowstore.docs.snappydata.io/docs/common/images/store-net-connection-complete.png)

	h.单击**下一步**继续。生成数据库向导将创建并显示创建完成的模型所需的RowStore DDL命令：
	
	![](http://rowstore.docs.snappydata.io/docs/common/images/store-ddl-gen-ddl.png)



1. 在“ **保存DDL为**”字段中输入DDL命令的文件名，然后单击“**完成**”。

1. 如果要使用已创建的模型替换现有模式，请首先备份现有数据库表中的所有数据，并从模式中删除这些表。请参阅使用**RowStore导出和导入数据**。

1. 在RowStore中执行DDL文件来创建新的模式。例如：

		snappy-shell run -file="c:\Users\yozie\Documents\Visual Studio 2010\Projects\WindowsFormsApplication1\WindowsFormsApplication1\Model1.edmx.sql"
		     -client-bind-address=localhost -client-port=1527

1. 如果使用模型替换现有模式，请使用“ **导出和导入数据与RowStore**”中的**说明**将数据导入到新表中。


#### 查询或修改实体 ####
您可以使用实体框架中的ObjectContext对象来查询，插入，删除和更新RowStore数据。

> 注意：如果使用RowStore存储过程插入，更新或删除实体值，则不能使用LINQ执行相同的操作。请参阅调**用RowStore过程和函数**。

- **查询表和视图**

- **插入行**

- **更新行**

- **删除行**

##### 查询表和视图 #####

要执行使用查询`ObjectContext`创建一个`ObjectQuery`，然后你就可以用它来创建使用查询LINQ to实体。针对RowStore数据源执行查询可返回单个对象或对象集合，可用于访问查询结果。执行查询时发生的任何RowStore错误都将直接传递给客户端。<br/>

以下示例显示了C＃中的简单LINQ to Entities查询：

	var context = new StudentEntities();
	var query = from c in context.STUDENT select c; 
	var students = query.ToList(); 
	
	foreach (STUDENT a in students) 
	{ 
	  Console.WriteLine("Student Name: {0}", a.Name); 
	} 
	
	Console.ReadLine();

##### 插入行 #####
您可以通过代码创建新的实体对象表示来插入行，然后将这些对象添加到ObjectContext。调用SaveChanges()在ObjectContext内部生成，并执行命令来执行使用所述RowStore数据源的等效INSERT语句。确保您设置基础表定义所需的所有对象属性。<br/>

以下C＃示例显示了添加到STUDENT表中的四个新学生行：

	var context = new StudentEntities();
	
	STUDENT student1 = new STUDENT { ID = "101", NAME = "Name1" }; 
	  context.STUDENTS.AddObject(student1); 
	STUDENT student2 = new STUDENT { ID = "102", NAME = "Name2" }; 
	  context.STUDENTS.AddObject(student2); 
	STUDENT student3 = new STUDENT { ID = "103", NAME = "Name3" }; 
	  context.STUDENTS.AddObject(student3); 
	STUDENT student4 = new STUDENT { ID = "104", NAME = "Name4" }; 
	  context.STUDENTS.AddObject(student4); 
	
	context.SaveChanges();

##### 更新行 #####
SaveChanges()根据ObjectContext您在上下文中修改实体的方式，调用基础RowStore表中的插入，更新或删除行。通过首先查询上下文然后在保存上下文之前修改返回的实体的一个或多个属性来执行更新。例如：

	var context = new StudentEntities();
	
	
	var student = students.FirstOrDefault(); 
	student.Name = "Name7"; 
	
	context.SaveChanges();

##### 删除行 #####
要从表中删除一行，请使用该DeleteObject()方法ObjectContext删除实体实例。该对象立即从其类型的集合中删除，但尚未被销毁。呼吁SaveChanges()对ObjectContext从表中删除行。例如：

	var context = new StudentEntities();
	
	
	var student = students.FirstOrDefault(); 
	
	context.DeleteObject(student); 
	
	context.SaveChanges();


## ODBC驱动程序支持的配置 ##
本主题列出了RowStore支持的ODBC驱动程序配置。

RowStore 1.4.1引入了对Progress DataDirect ODBC驱动程序1.7版本的支持。下表列出了驱动程序支持的平台，架构和驱动程序管理器。


表1.驱动程序支持的配置

|操作系统|司机经理|驱动下载|字符类型|
|:--|:--|:--|:--|
|RHEL 4，5，6或7，CentOS 4,5,6或7，SUSE Linux Enterprise Server 10.x和11|Progress DataDirect v1.7|[RowStore ODBC分发1.4.1](https://network.pivotal.io/products/snappystore)|Unicode，ANSII|
|（32位和64位）||||
|Windows Server 2003 Service Pack 2及更高版本，Windows Server 2008，Windows Server 2012，Windows 7，Windows 8.x，Windows Vista，Windows XP Service Pack 2及更高版本|Progress DataDirect v1.7|[RowStore ODBC分发1.4.1](https://network.pivotal.io/products/snappystore)|Unicode，ANSII|
|（32位和64位）||||

：表1.驱动程序支持的配置

- **安装和配置进度DataDirect ODBC驱动程序** 本主题介绍如何在Linux和Windows平台上安装和配置Progress DataDirect ODBC驱动程序。

### 安装和配置Progress DataDirect ODBC驱动程序 ###
本主题介绍如何在Linux和Windows平台上安装和配置Progress DataDirect ODBC驱动程序。

Progress DataDirect ODBC驱动程序二进制文件适用于基于Linux和Windows的平台。二进制安装程序可用于32位和64位体系结构。

- **为Linux安装Progress DataDirect ODBC驱动程序**

- **安装Windows的Progress DataDirect ODBC驱动程序**

- **配置Progress DataDirect ODBC驱动程序**

#### 为Linux安装Progress DataDirect ODBC驱动程序 ####

1. 下载并解压缩RowStore ODBC Distribution 1.4.1。例如：

		$ tar -zxvf SnappyData_RowStore_XX_bNNNNN_ODBC.tar.gz -C path_to_install_location
其中path_to_install_location是已存在的目录。

1. 更改为ODBC分发的/ linux子目录：

		$ cd path_to_install_location/SnappyData_RowStore_XX_bNNNNN_ODBC/linux


1. 显示readme.txt文件的内容，并记下linux系统架构（32位或64位）的IPE密钥。您将需要在安装过程中输入正确的IPE密钥。例如：

		$ cat readme.txt
		[...]
		For OEM installations you may be asked for an IPE key. Please use these keys
		for the appropriate platform. 
		
		Linux:
		1076719616 RowStore All 32-bit Platforms
		1076719872 RowStore All 64-bit Platforms
		[...]

1. 根据您的架构更改到/ 32或/ 64子目录，然后执行ODBC安装程序。例如：

		$ cd 64
		$ ./PROGRESS_DATADIRECT_ODBC_7.1_LINUX_64_INSTALL.bin
	安装程序加载并显示“简介”屏幕：

	![](http://rowstore.docs.snappydata.io/docs/common/images/odbc-inst-intro.png)

1. 单击**下一步**显示许可协议。

1. 阅读并接受许可证，然后单击**下一步**显示安装目录：

	![](http://rowstore.docs.snappydata.io/docs/common/images/odbc-inst-installdir.png)

1. 选择要安装驱动程序的目录，或接受默认目录，然后单击**下一步**。

1. 在安装类型屏幕上，选择**OEM /许可安装**，然后单击**下一步**。

1. 输入您在步骤3中记录的IPE密钥，然后单击验证。然后安装程序显示可用于安装的产品：

	![](http://rowstore.docs.snappydata.io/docs/common/images/odbc-inst-ipe.png)

1. 选择**驱动程序**，然后单击**下一步**以显示预安装摘要。

1. 单击**安装**以将驱动程序**安装**到所选目录中。

1. 安装完成后，单击**完成**退出安装程序。

#### 安装Windows的Progress DataDirect ODBC驱动程序 ####
1. 下载并解压缩[RowStore ODBC Distribution 1.4.1](https://network.pivotal.io/products/snappystore)。

1. 根据操作系统（32位或64位）的体系结构，更改为ODBC分发的\ windows \ 32或\ windows \ 64子目录。例如：

		c:\> cd path_to_install_location\SnappyData_RowStore_XX_bNNNNN_ODBC\windows

1. 在目录中执行ODBC安装程序可执行文件。例如：

		c:\SnappyData_RowStore_XX_bNNNNN_ODBC\windows\64> PROGRESS_DATADIRECT_ODBC_7.1_WIN_64_INSTALL.EXE
安装程序加载并显示“简介”屏幕：

	![](http://rowstore.docs.snappydata.io/docs/common/images/odbc-inst-intro-win.png)

1. 单击**下一步**显示许可协议。

1. 阅读并接受许可证，然后单击**下一步**显示安装目录：

	![](http://rowstore.docs.snappydata.io/docs/common/images/odbc-inst-installdir-win.png)

1. 选择要安装驱动程序的目录，或接受默认目录，然后单击**下一步**。

1. 在安装类型屏幕上，选择**OEM /许可安装**，然后单击**下一步**。

1. 输入您在步骤3中记录的IPE密钥，然后单击验证。然后安装程序显示可用于安装的产品：

	![](http://rowstore.docs.snappydata.io/docs/common/images/odbc-inst-ipe-win.png)

1. 选择驱动程序，然后单击**下一步**。<br/>
安装程序将显示“创建默认数据源”屏幕：

	![](http://rowstore.docs.snappydata.io/docs/common/images/odbc-inst-registry-win.png)

1. 如果要更新注册表项并将Gemfire XD的Progress DataDirect ODBC驱动程序设置为**默认值**，请选择“ **创建默认数据源**”。单击**下一步**以显示预安装摘要。

1. 单击**安装**以将驱动程序**安装**到所选目录中。

1. 安装完成后，单击**完成**退出安装程序。

#### 配置Progress DataDirect ODBC驱动程序 ####
有关配置和使用驱动程序的详细说明，请参阅安装的ODBC驱动程序文档：

1. ODBC驱动程序的主要文档安装在path_to_install_location /help/RowStoreHelp/index.html中
'
1. 快速入门连接部分（path_to_install_location /help/RowStoreHelp/RowStoreHelp/snappystore/quick-start-connect.html）介绍了如何配置和测试新安装的驱动程序。

## 使用RowStore与Hibernate ##
Pivotal提供了一个定义SQL语言的RowStore变体的Hibernate方言文件。您可以使用此文件与RowStore JDBC驱动程序配置RowStore作为开发Hibernate项目的数据库。

见[RowStore的Hibernate方言](https://support.pivotal.io/hc/en-us/articles/201724017-Pivotal-GemFire-XD-Hibernate-Dialect)的枢纽社区站点有关如何获取并使用RowStore方言文件，你的Hibernate项目的信息。
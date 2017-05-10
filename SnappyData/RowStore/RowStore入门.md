# RowStore入门 #
RowStore入门提供了安装，配置和使用RowStore的分步过程。该指南还解释了主要概念，并提供了帮助您快速开始使用RowStore的教程。<br/>

- **RowStore在15分钟** 需要快速介绍RowStore？参加这个15分钟的游览尝试基本功能。

- **RowStore概述** RowStore是一种内存优化的分布式数据存储，专为具有苛刻的可扩展性和可用性要求的应用程序而设计。使用RowStore，您可以使用内存中的表完全管理数据，也可以将非常大的表保存到本地磁盘存储文件。在此初始版本中，RowStore为内存表数据提供了低延迟的SQL接口，同时无缝集成持久化的数据。可以使用商品硬件轻松扩展单个RowStore分布式系统，以支持数千个并发客户端，还可以通过WAN接口在不同RowStore群集之间复制数据。

- **了解RowStore分布式系统** RowStore部署由分布式成员进程组成，它们彼此连接以形成对等网络，也称为分布式系统或RowStore集群。

- **安装RowStore** 您可以从下载的RPM文件（仅限RHEL）或从已下载的ZIP文件（Windows或RHEL）安装用于生产的RowStore。Pivotal还提供Debian软件包和Homebrew软件包，用于与Ubuntu和Mac OSX一起开发。以下部分介绍如
何使用下载的RPM文件，ZIP文件，Debian软件包或Homebrew软件包来安装独立的RowStore产品。

- **将RowStore** 升级从GemFire XD 1.4.x升级到RowStore 1.5。

- **使用JDBC工具连接到RowStore** 第三方JDBC工具可以帮助您浏览表中的数据，发出SQL命令，设计新表等。您可以

- **配置这些工具以使用RowStore** JDBC瘦客户端驱动程序连接到RowStore分布式系统。

- **快速入门教程** 了解如何配置和使用RowStore功能，例如表复制和分区，将数据持久存储到本地磁盘存储，以及动态调整集群大小。	

## RowStore在15分钟 ##
需要快速介绍RowStore？参加这个15分钟的游览尝试基本功能。<br/>

RowStore教程扩展了这些概念，并展示了其他产品功能。请参阅快速入门教程。<br/>



1. 如果您已经安装了RowStore，请跳到步骤6。


1. `.zip`从产品下载页面下载最新的RowStore 文件分发：[https://github.com/SnappyDataInc/snappydata/releases](https://github.com/SnappyDataInc/snappydata/releases)。将下载的文件保存在主目录中。


1. 通过解压缩下载的文件来安装RowStore。例如：


		$ cd ~
		$ unzip snappydata-0.5-bin.zip
替换您下载的确切文件名。
这将RowStore安装在`snappydata-XXX-bin`主目录的新子目录中，其中XXX是RowStore的版本。

1. 如果还没有这样做，请下载并安装Java。有关此版本的RowStore支持的Java版本列表，请参阅**支持的配置和系统要求**。

1. 设置PATH环境变量以包括`bin`与`sbin`该RowStore目录的子目录。例如：

		$ export PATH=$PATH:$HOME/snappydata-XXX-bin/bin:$HOME/snappydata-XXX-bin/sbin

1. 在主目录中创建三个新目录，用于定位器和两个将构成RowStore分布式系统的服务器：

		$ mkdir locator1 server1 server2

1. 创建启动配置文件。该snappy-start-all.sh脚本读取启动从服务器和定位器的配置`locators`和`servers`文件中`conf`的RowStore目录的目录（$ HOME / snappydata-XXX-斌/）。使用以下参数在目录中创建`locators`和`servers`文件`conf`。<br/>

		$ cat /home/user1/snappydata-XXX-bin/conf/locators 
		localhost -dir=/home/user1/locator1 -jmx-manager-start=true -jmx-manager-http-port=7075
		
		$ cat /home/user1/snappydata-XXX-bin/conf/servers
		localhost -dir=/home/user1/server1 -locators=localhost[10334] -bind-address=localhost -client-port=1528
		localhost -dir=/home/user1/server2 -locators=localhost[10334] -bind-address=localhost -client-port=1529
`locators`上面显示的文件启动locahost上的定位器，数据目录为/ home / user1 / locator1。指定`-jmx-manager-start=true`在定位器中启动一个嵌入式JMX管理器。通过启动JMX Manager，您可以通过Pulse图形界面监视和浏览RowStore分布式系统
此配置启动默认定位器，该定位器接受localhost地址上的连接。默认端口10334用于与分布式系统的其他成员进行通信。分布式系统的所有新成员必须指定此定位器的地址和对等体发现端口localhost [10334]才能加入系统。1527的默认端口用于与分布式系统的客户端连接。<br/>
servers上面显示的文件有两个不同的条目，两个条目用于在本地主机上启动。它指定要与服务器接受客户端连接的对等体发现端口（10334）和客户端端口一起使用的数据目录位置，定位器。<br/>

1. 使用snappy-start-all.sh命令启动群集：<br/>

		$ snappy-start-all.sh rowstore
		localhost: Starting SnappyData Locator using peer discovery on: localhost[10334]
		localhost: Starting DRDA server for SnappyData at address localhost/127.0.0.1[1527]
		localhost: Logs generated in /home/user1/locator1/snappylocator.log
		localhost: SnappyData Locator pid: 3564 status: running
		localhost: Starting SnappyData Server using locators for peer discovery: localhost[10334]
		localhost: Starting DRDA server for SnappyData at address localhost/127.0.0.1[1528]
		localhost: Logs generated in /home/user1/server1/snappyserver.log
		localhost: SnappyData Server pid: 3849 status: running
		localhost:   Distributed system now has 2 members.
		localhost:   Other members: localhost(3564:locator)<v0>:28083
		localhost: Starting SnappyData Server using locators for peer discovery: localhost[10334]
		localhost: Starting DRDA server for SnappyData at address localhost/127.0.0.1[1529]
		localhost: Logs generated in /home/user1/server2/snappyserver.log
		localhost: SnappyData Server pid: 4033 status: running
		localhost:   Distributed system now has 3 members.
		localhost:   Other members: localhost(3849:datastore)<v1>:42564, localhost(3564:locator)<v0>:28083
snappy-start-all.sh命令使用前面创建的配置文件启动一个定位器和两个服务器，如上所示。<br/>

1. 打开浏览器并输入以下URL 启动脉冲监控工具：<br/>

		http://localhost:7075/pulse
在“脉冲登录”屏幕上，键入默认用户名`admin`和密码`admin`。在执行此快速入门中的活动时，继续检查Pulse工具以进行相应的更改。<br/>

1. 在进一步之前，请检查以确保您位于RowStore quickstart子目录中。您将需要在本教程的后面的该目录中运行脚本文件，您必须从quickstart目录中执行这些脚本。如果从TAR文件安装RowStore：

		$ cd ~/snappydata-XXX-bin/quickstart/store

1. 以瘦客户端方式连接到分布式系统，并显示系统成员的信息：

		$ snappy-shell
		snappy> connect client 'localhost:1527';

1. 现在您已连接到系统，运行一个简单的查询来显示有关RowStore系统成员的信息：

		snappy> select id, kind, netservers from sys.members;
		ID                            |KIND                  |NETSERVERS                                                                                                
		-------------------------------------------------------------------------------
		localhost(4033)<v2>:59747     |datastore(normal)     |localhost/127.0.0.1[1529]                                                                                 
		localhost(3849)<v1>:42564     |datastore(normal)     |localhost/127.0.0.1[1528]                                                                                 
		localhost(3564)<v0>:28083     |locator(normal)       |localhost/127.0.0.1[1527]                                              
		
		3 rows selected
默认情况下，RowStore服务器作为数据存储启动，从而可以托管数据库模式。在此集群中，您可以通过使用成员的唯一端口号（NETSERVERS列中指定的端口号）指定localhost，将其作为客户端连接到任何成员。但是，连接到定位器通过将连接请求路由到可用的服务器成员来提供基本的负载平衡。<br/>

1. 创建一个简单的表并插入几行：

		snappy> create table quicktable (id int generated always as identity, item char(25));
		0 rows inserted/updated/deleted
		snappy> insert into quicktable values (default, 'widget');
		1 row inserted/updated/deleted
		snappy> insert into quicktable values (default, 'gadget');
		1 row inserted/updated/deleted
		snappy> select * from quicktable;
		ID         |ITEM                     
		-------------------------------------
		2          |gadget                
		1          |widget                   
		
		2 rows selected

1. 默认情况下，RowStore会将创建的新表复制到数据存储成员上。您可以使用查询验证此：

		snappy> select tablename, datapolicy from sys.systables where tablename='QUICKTABLE';
		TABLENAME                                                    |DATAPOLICY     
		-----------------------------------------------------------------------------
		QUICKTABLE                                                   |REPLICATE      
		
		1 row selected


1. 执行两个SQL脚本以生成具有复制和分区表的模式，然后使用数据加载模式：

		snappy> run 'create_colocated_schema.sql';
		snappy> run 'loadTables.sql';
在执行各种SQL命令时，会看到大量消息。第一个脚本创建复制和分区表，您可以使用查询看到：

		snappy> select tablename, datapolicy from sys.systables where tableschemaname='APP';
		TABLENAME                                                     |DATAPOLICY     
		------------------------------------------------------------------------------
		FLIGHTS_HISTORY                                               |PARTITION      
		FLIGHTAVAILABILITY                                            |PARTITION      
		FLIGHTS                                                       |PARTITION      
		MAPS                                                          |REPLICATE      
		CITIES                                                        |REPLICATE      
		COUNTRIES                                                     |REPLICATE      
		AIRLINES                                                      |REPLICATE      
		QUICKTABLE                                                    |REPLICATE      
		
		8 rows selected

1. 要观察表分区的好处，请查看涉及其中一个分区表的查询计划。使用具有查询的EXPLAIN命令来生成查询执行计划：

		snappy> explain select * from flights;
该`EXPLAIN`命令将语句的查询执行计划存储在STATEMENTPLANS系统表中。<br/>

1. 要查看查询计划的详细信息，请从分布式系统断开为瘦客户机，然后重新连接为对等客户端。对等客户机作为RowStore分布式系统的成员进行参与，并且可以协调查询，但不会托管任何实际的数据。执行这些命令：

		snappy> disconnect;
		snappy> connect peer 'host-data=false;locators=localhost[10334]';
您可以看到您的对等客户端连接向分布式系统引入了新的成员：

		snappy> select id, kind, netservers from sys.members;
		ID                             |KIND                  |NETSERVERS                                                                                              
		--------------------------------------------------------------------------------
		localhost(3849)<v1>:42564      |datastore(normal)     |localhost/127.0.0.1[1528]                                                                               
		localhost(4033)<v2>:59747      |datastore(normal)     |localhost/127.0.0.1[1529]                                                                               
		localhost(3564)<v0>:28083      |locator(normal)       |localhost/127.0.0.1[1527]                                                                               
		192.168.43.206(4809)<v5>:55522 |accessor(normal)      |                                                                                                        
		
		4 rows selected
访问者一词表示该成员仅访问数据，但不存储分布式系统的数据。

1. 要查看您之前生成的查询执行计划，请查询SYS.STATEMENTPLANS表以查看语句ID（STMT_ID），然后再次使用EXPLAIN与ID查看计划：

		snappy> select stmt_id, stmt_text from sys.statementplans;
		STMT_ID                             |STMT_TEXT                              
		-------------------------------------------------------------------------------
		00000001-ffff-ffff-ffff-000200000018| select * from flights                   
		
		1 row selected
		snappy> explain '00000001-ffff-ffff-ffff-000200000018';
		member localhost(3849)<v1>:42564 begin_execution 2016-07-04 10:40:25.681 end_execution 2016-07-04 10:40:25.712 (31 milli-seconds elapsed)
		|   |(1)QUERY-RECEIVE (56.53%) execute_time 29.676133ms (serialize/deserialize=0.0ms + process=29.676133ms + throttle=0.0ms) member_node localhost(4033)<v2>:59747
		|   |   |(4)RESULT-SEND (0.17%) execute_time 0.091206ms (serialize/deserialize=0.091206ms + process=0.0ms + throttle=0.0ms) member_node localhost(4033)<v2>:59747
		|   |   |   |(2)RESULT-HOLDER (22.46%) execute_time 11.790009ms (serialize/deserialize=1.13364ms + process=10.656369ms + throttle=0.0ms) returned_rows 282 no_opens 1 member_node NULL
		|   |   |   |   |(3)TABLESCAN (20.84%) execute_time 10.938687ms (construct=0.071656ms + open=1.007879ms + next=2.697055ms + close=7.162097ms) returned_rows 282 no_opens 1 visited_rows 282 scan_qualifiers None next_qualifiers NULL scanned_object APP.FLIGHTS scan_type HEAP
		member localhost(4033)<v2>:59747 begin_execution 2016-07-04 10:40:25.678 end_execution 2016-07-04 10:40:25.805 (127 milli-seconds elapsed)
		|   |(1)QUERY-SCATTER (33.51%) execute_time 33.818489ms (serialize/deserialize=0.0ms + process=33.818489ms + throttle=0.0ms) member_node localhost(3849)<v1>:42564,localhost(4033)<v2>:59747
		|   |   |---QUERY-SEND (0.27%) execute_time 0.270473ms member_node localhost(3849)<v1>:42564
		|   |   |---RESULT-RECEIVE (0.39%) execute_time 0.398331ms member_node localhost(3849)<v1>:42564
		|   |   |(4)QUERY-SEND (11.47%) execute_time 11.573682ms (serialize/deserialize=0.0ms + process=11.573682ms + throttle=0.0ms) member_node localhost(4033)<v2>:59747
		|   |   |---RESULT-RECEIVE (0.02%) execute_time 0.016511ms member_node localhost(4033)<v2>:59747
		|   |   |(2)SEQUENTIAL-ITERATION (31.47%) execute_time 31.767072ms (construct=-1.0E-6ms + open=-1.0E-6ms + next=31.767072ms + close=-1.0E-6ms) returned_rows 542 no_opens 1
		|   |   |   |---RESULT-HOLDER (0.94%) execute_time 0.951177ms returned_rows 260 no_opens 1 member_node localhost(4033)<v2>:59747
		|   |   |   |(5)RESULT-HOLDER (1.01%) execute_time 1.019598ms (serialize/deserialize=0.0ms + process=1.019598ms + throttle=0.0ms) returned_rows 282 no_opens 1 member_node localhost(3849)<v1>:42564
		|   |   |(3)DISTRIBUTION-END (20.92%) execute_time 21.113345ms (construct=0.0ms + open=20.299513ms + next=32.556921ms + close=0.023983ms) returned_rows 542
		Local plan:
		member localhost(4033)<v2>:59747 begin_execution 2016-07-04 10:40:25.69 end_execution 2016-07-04 10:40:25.755 (65 milli-seconds elapsed)
		|   |(1)TABLESCAN (100.00%) execute_time 9.715992ms (construct=0.020652ms + open=0.891605ms + next=3.068815ms + close=5.73492ms) returned_rows 260 no_opens 1 visited_rows 260 scan_qualifiers None next_qualifiers NULL scanned_object APP.FLIGHTS scan_type HEAP


	> 注意：您的系统上生成的语句ID可能不同。从SELECT语句的输出中复制确切的ID，并将其粘贴到第二个EXPLAIN语句中。

	该计划描述了RowStore如何执行查询。注意两个QUERY-SEND条目。这些条目表示查询的结果是从分布式系统中的两个数据存储成员中的每一个获取的。因为FLIGHTS表是作为分区表创建的，所以添加到表中的新行根据分区键唯一地分配给分区（在这种情况下，关键是FLIGHT_ID列）。然后将分区放置在数据存储上，当对表执行查询时，可以独立处理其数据部分。然后将多个数据存储的结果合并到单个查询协调器成员上以提供最终结果集。<br/>


1. 可以继续对示例数据库执行查询，还是关闭RowStore分布式系统。要关闭系统的所有成员，请使用以下`snappy-stop-all.sh`命令：

		snappy> quit;
		$ snappy-stop-all.sh rowstore
		localhost: The SnappyData Server has stopped.
		localhost: The SnappyData Server has stopped.
		localhost: The SnappyData Locator has stopped.


1. 要继续了解RowStore，请阅读或使用其余的QuickStart教程。

## RowStore概述 ##
RowStore是一种内存优化的分布式数据存储，专为具有苛刻的可扩展性和可用性要求的应用程序而设计。使用RowStore，您可以使用内存中的表完全管理数据，也可以将非常大的表保存到本地磁盘存储文件。RowStore为内存中的表数据提供了低延迟的SQL接口。可以使用商品硬件轻松扩展单个RowStore分布式系统，以支持数千个并发客户端，还可以通过WAN接口在不同RowStore群集之间复制数据。<br/>

![](http://rowstore.docs.snappydata.io/docs/common/images/esql_at_a_glance.png)


- **RowStore功能和优点** RowStore提供极高的吞吐量，可预测的延迟，动态和线性可扩展性以及数据的持续可用性。RowStore利用SQL接口和工具，Java和其他广泛实现的技术，使其轻松适应现有的数据库应用程序。

- **Apache Geode，Pivotal HD和Apache Derby Components** RowStore集成了核心Apache Geode技术和Apache Derby RDBMS组件，以提供高性能的分布式数据库管理系统。RowStore在必要时扩展标准SQL语句，用于创建和管理表格并配置RowStore系统。

### RowStore功能和优点 ###
RowStore提供极高的吞吐量，可预测的延迟，动态和线性的可扩展性以及数据的持续可用性。RowStore利用SQL接口和工具，Java和其他广泛实现的技术，使其轻松适应现有的数据库应用程序。<br/>

以下部分总结了主要特征。有关此版本中的新功能和更改的完整列表，请参阅“ **RowStore 1.5发行说明** ”。<br/>

#### 具有优化磁盘持久性的内存数据管理 ####
RowStore使应用程序可以通过使用分区和同步复制在多个RowStore成员中分发数据来完全管理内存中的数据。

#### 内存数据与持久性商店的无缝集成 ####
RowStore提供了一种优化的本地磁盘持久性机制，具有非刷新算法，可在需要稳定长期存储的应用程序中保持高性能。应用程序还可以使用RowStore从传统的基于磁盘的RDBMS中主动缓存表数据。<br/>

#### 持续可用性，弹性缩放，低延迟 ####
灵活的架构使RowStore可以从数百个群集成员中池存储器和磁盘资源。该集群方法提供极高的吞吐量，可预测的延迟，动态和线性的可扩展性以及数据的连续可用性。通过将应用程序逻辑与数据并行并行并行执行应用程序逻辑，RowStore可显着增加应用程序吞吐量。如果服务器发生故障，它也会透明地重新执行应用程序逻辑。<br/>


#### 高度适应现有应用 ####
RowStore完全用Java实现，它可以直接嵌入到Java应用程序中。您也可以将RowStore成员部署为参与集群的独立服务器。Java应用程序可以使用提供的JDBC驱动程序连接到RowStore集群。Microsoft .NET和Mono应用程序可以使用提供的ADO.NET驱动程序进行连接，RowStore还提供符合ODBC的驱动程序。<br/>

使用SQL，JDBC，ODBC和ADO.NET意味着许多现有的数据库应用程序可以轻松地适应于使用RowStore集群。RowStore引入了常见SQL数据定义语言（DDL）语句的多个扩展，以管理数据分区，复制，与数据源的同步以及其他功能。然而，最常见的查询和数据操作语言（DML）语句基于ANSI SQL-92，所以经验丰富的数据库应用程序开发人员可以在使用RowStore时使用他们对SQL的了解。<br/>

### Apache Geode和Apache Derby组件 ###
RowStore集成了Apache Geode核心技术和Apache Derby RDBMS组件，以提供高性能的分布式数据库管理系统。RowStore在必要时扩展标准SQL语句，用于创建和管理表格并配置RowStore系统。<br/>

以下部分介绍了RowStore如何利用Apache Geode和Derby组件功能。<br/>


#### Apache Geode技术 ####
RowStore结合了以下[Apache Geode](http://geode.docs.pivotal.io/docs/getting_started/geode_overview.html "Apache Geode")技术：

- 可靠的数据分发

- 高性能复制和分区

- 缓存框架

- 并行“数据感知”应用程序行为路由
#### Apache Derby RDBMS组件 ####
RowStore将Apache Geode功能与Apache Derby关系数据库管理系统（RDBMS）的多个组件集成：

- JDBC驱动程序 RowStore支持本机，高性能JDBC驱动程序（对等驱动程序）和精简JDBC驱动程序。对等驱动程序基于Derby嵌入式驱动程序和JDBC 4.0接口，但与RowStore服务器的所有通信都通过Apache Geode分发层实现。

- 查询引擎 RowStore使用Derby来解析SQL查询并生成解析树。RowStore注入自己的中间计划创建逻辑，并将该计划分发到集群中的数据存储。RowStore还利用Derby中内置优化器的某些方面来生成查询计划。查询执行本身使用基于内存的索引和自定义存储数据结构。当查询执行需要分发时，RowStore使用自定义算法在多个数据存储上并行执行查询。

- 网络服务器 RowStore服务器将Derby网络服务器嵌入到瘦JDBC，ODBC和.NET客户端的连接中。通信协议基于IBM DB2驱动程序中使用的[DRDA](http://en.wikipedia.org/wiki/DRDA "DRDA")标准。

RowStore修改并扩展查询引擎和SQL接口，以支持分区和复制表，数据感知过程，数据持久性，数据迁移以及分布式RowStore架构独有的其他功能。RowStore还添加了SQL命令，存储过程，系统表和函数，以帮助轻松管理分布式系统的功能，如永久磁盘存储，侦听器和定位器。<br/>

## 了解RowStore分布式系统 ##
RowStore部署由分布式成员进程组成，它们彼此连接以形成对等网络，也称为分布式系统或RowStore集群。<br/>

下面的部分说明主要系统组件和过程的相互作用。QuickStart教程可帮助您开始配置和使用RowStore分布式系统。<br/>


- **RowStore成员** 成员进程形成一个单一的逻辑系统，每个成员具有对任何其他成员的单跳访问，具有单跳或无跳数访问数据。

- **服务器和服务器组** 甲 RowStore服务器是驻留数据，并且是对等网络的分布式系统的成员的过程。RowStore服务器在Java虚拟机（JVM）中运行。

- **发现机制** 对等成员（服务器或对等客户端进程）使用两种机制之一向分布式系统发布自己的内容。

- **群组会员服务** 群组会员服务（GMS）使用自定义系统会员资格。进程可以随时加入或离开分布式系统。GMS将此信息与系统中的每个其他成员进行通信，并具有一定的一致性保证。团体中的每个成员都参与会员决策，确保所有成员都看到一个新成员，或者没有成员看到它。

- **复制表和分区表RowStore中的表** 可以被分区或复制。复制表将其整个数据集的副本保留在其服务器组中的每个RowStore服务器上。分区表通过将其分成可管理的块并将这些块分布在表的服务器组中的所有成员上来管理大量数据。

- **数据感知存储过程的并行执行** 在传统的关系数据库中，存储过程是作为数据字典的一部分存储并在数据库系统本身上执行的应用程序例程。存储过程通常提供高性能，因为它们靠近应用逻辑所需的数据执行。RowStore扩展了这种基本的存储过程功能，以支持在许多对等体上分区的表数据上并行执行应用程序逻辑。

- **外部数据连接的缓存插件** RowStore通常用作嵌入式（对等客户端）或客户端 - 服务器配置中的分布式SQL缓存。它提供了一个插件框架，用于将缓存与外部数据源（如另一个关系数据库）相连接。

### RowStore会员 ###
成员进程形成一个单一的逻辑系统，每个成员具有对任何其他成员的单跳访问，具有单跳或无跳数访问数据。<br/>

RowStore成员是在JVM中运行的RowStore代码的实例。RowStore成员可以选择性地托管数据，为客户端连接提供网络服务器功能，并为分布式系统提供位置服务。<br/>

RowStore分布式系统是动态的，可以随时添加或删除成员。RowStore实现保证了分布式系统的一致视图，以确保数据一致性和数据完整性不受影响。<br/>

大多数RowStore成员被配置为托管数据，并被称为数据存储。被配置为不托管数据的成员被称为访问器。数据存储和访问器都可以执行RowStore支持的DDL和DML命令。数据存储提供对存储在分布式系统的成员上的存储的数据的单跳或无跳数访问。访问者提供对分布式系统中数据存储的单跳访问。<br/>

第三种类型的成员，独立定位器，不托管数据，并且不支持用户定义的表上的DDL和DML语句。您可以使用定位器来发现RowStore集群的成员。<br/>

有关详细信息，请参阅：

- **启动和停止RowStore成员**

- **使用定位器**

### 服务器和服务器组 ###
甲RowStore服务器是驻留数据，并且是对等网络的分布式系统的成员的过程。RowStore服务器在Java虚拟机（JVM）中运行。<br/>

您可以使用`snappy-shell`命令提示符或终端窗口中的工具启动RowStore服务器。`snappy-shell`将服务器作为类似于数据库服务器的独立进程启动。这些服务器可以接受来自瘦客户端的TCP连接，验证凭据，管理会话，将工作委派给线程池进行SQL处理等等。<br/>

服务器进程是分布式系统的对等成员。成员通过作为集群的第一个成员启动的定位器服务动态发现。<br/>

> 注意： RowStore服务器支持来自JDBC和ODBC瘦客户端驱动程序的瘦客户端连接。请参阅**连接Java客户端，使用RowStore ADO.NET客户端**和**ODBC驱动程序支持的配置**。

托管数据的RowStore服务器和对等客户端（host-data属性设置为true时）将自动属于默认服务器组。甲服务器组是定义数据存储应该针对表主机数据RowStore服务器和对等客户端部件的逻辑分组。创建任何RowStore对象（如表）时，可以在CREATE TABLE语句中指定要托管该表的服务器组名称。如果未指定组，表将托管在默认服务器组中。使用服务器组管理数据分发提供了其他信息。<br/>

有关详细信息，请参阅：

- **启动和停止RowStore成员**

- **使用定位器**

- **使用服务器组来管理数据分发**

### 发现机制 ###
对等成员（服务器或对等客户端进程）使用两种机制之一向分布式系统发布自身。<br/>

RowStore提供了这些发现机制：<br/>

- **定位器（TCP / IP）。**定位器服务在任何给定时刻维护分布式系统中所有对等成员的注册表。定位器通常作为单独的进程（具有冗余）启动，但也可以将定位器嵌入到任何对等成员中，例如RowStore服务器。定位器打开一个TCP端口，通过它可以连接所有新成员以获取初始成员信息。

- **UDP / IP组播**。成员可以选择使用组播地址来广播其存在并接收成员资格通知信息。<br/>


**配置发现机**制提供更多信息。

### 团体会员服务 ###
集团会员服务（GMS）使用自行定义的系统会员资格。进程可以随时加入或离开分布式系统。GMS将此信息与系统中的每个其他成员进行通信，并具有一定的一致性保证。团体中的每个成员都参与会员决策，确保所有成员都看到新成员，或者没有成员看到它。<br/>

会员协调员是GMS的关键组成部分，处理“加入”和“离开”请求，并处理怀疑已离开系统的成员。系统自动选择分布式系统中最老的成员作为协调者，如果成员失败或不可达，则选择新的成员。协调员的基本目的是将当前成员资格视图中继到分布式系统的每个成员，并确保视图始终保持一致。<br/>

因为RowStore分布式系统是动态的，您可以在很短的时间内添加或删除成员。这使得轻松重新配置系统来处理添加的需求（负载）.GMS允许分布式系统在静态定义的成员系统不能满足的条件下进行。静态模型通过主机和身份定义成员，这使得在活动的分布式系统中添加或删除成员变得困难。系统必须部分或全部关闭以扩展或收缩参与系统的成员数量。<br/>

有关详细信息，请参阅：

- **使用snappy-shell启动和停止成员**

- **使用定位器连接到分布式系统**

- **对RowStore成员重新平衡分区数据**

### 复制表和分区表 ###
RowStore中的表可以被分区或复制。复制表将其整个数据集的副本保留在其服务器组中的每个RowStore服务器上。分区表通过将其分成可管理的块并将这些块分布在表的服务器组中的所有成员上来管理大量数据。<br/>

默认情况下，除非在CREATE TABLE语句中指定了分区，否则将复制所有表。所有RowStore对象的架构信息始终可见包括对等客户端在内的分布式系统的所有对等成员，但不包括独立定位器。<br/>

**分区表**和**复制表**提供更多信息。

### 并行执行数据感知存储过程 ###
在传统的关系数据库中，存储过程是作为数据字典的一部分存储并在数据库系统本身上执行的应用程序例程。存储过程通常提供高性能，因为它们靠近应用逻辑所需的数据执行。RowStore扩展了这种基本的存储过程功能，以支持在许多对等体上分区的表数据上并行执行应用程序逻辑。<br/>

RowStore应用程序可以对服务器组的所有成员并行执行特定数据主机上的存储过程，也可以根据过程的数据要求定位特定成员。基本上，封装在存储过程中的应用程序行为被移动到承载关联数据集的进程，并在那里执行。如果所需的数据集遍布多个分区，则该过程在分区成员上并行执行。结果流式传输到协调成员，并为客户端调用过程进行聚合。<br/>

例如，考虑由“customer_id”划分的“订单”表，以及希望为多个客户执行昂贵的“信用检查”的应用程序。假设信用测试需要对所有订单历史进行迭代。您可以对管理这些客户的数据的所有成员并行执行，并将结果流式传输到客户端。每个执行所需的所有订单历史都在本地可用。<br/>

	// RowStore data-aware procedure invocation 
	      CallableStatement callableStmt = connection.prepareCall("{CALL order_credit_check() " 
	        + "ON TABLE Orders WHERE customerID IN (?, ?, ?)}"); 
	      callableStmt.setInt(1, 325);
	      callableStmt.setInt(2, 540);
	      callableStmt.setInt(3, 871);
	
	// order_credit_check will be executed in parallel on all members where the orders
	// corresponding to the customerIDs are managed
有关详细信息，请参阅：

- **编程数据感知程序和结果处理器**

- **使用过程提供程序API**

- **使用自定义结果处理器API**

### 用于外部数据连接的缓存插件 ###
RowStore通常用作嵌入式（对等客户端）或客户端 - 服务器配置中的分布式SQL缓存。它提供了一个插件框架，用于将缓存与外部数据源（如另一个关系数据库）相连接。<br/>

支持几个缓存插件：

- **从后台读小姐（“阅读”）**<br/>
当RowStore用作缓存时，应用程序可以配置一个SQL RowLoader，该RowLoader被触发以从RowStore中的一个未完成的后端存储库中加载数据。当分布式缓存不能满足唯一标识行的传入查询请求时，调用加载程序以从外部源检索数据。RowStore锁定相关联的行，并阻止并发读取器尝试从捕获后端数据库获取同一行。

	> 注意：当RowStore用作“纯”缓存（即，部分或全部表配置为LRU逐出时，只有主动使用的数据子集在RowStore中缓存），这些表上的查询才能基于首要的关键。只有基于主键的查询才会调用配置的加载程序。任何其他查询可能会从缓存中产生不一致的结果。

请参阅使用**RowLoader加载现有数据**。<br/>

- **同步写回到后端（“直写”）**<br/>
当内存中严格管理数据时，即使使用复制，整个群集的故障也意味着丢失数据。通过同步直写，所有更改可以在缓存更改之前同步写入后端存储库。当且仅当写通过成功时，数据才能在缓存中显示。您可以使用RowStore“缓存写入程序”配置同步直写。请参阅**同时处理DML事件**。

- **异步写回到后端（“写在后面”）**<br/>
如果对后端的同步写入成本太高，则应用程序可以配置使用“写入缓存侦听器”。RowStore支持几个选项，用于如何将事件排队，分批和写入记录数据库。它设计用于非常高的可靠性并处理许多故障条件。即使后端数据库暂时不可用，持久队列仍可确保写入。您可以根据时间间隔订购活动传递或批量。您还可以将一定数量的行和连续更新混合到单个更新中，以减少“更新重”系统的后端数据库的负载。请参阅**异步处理DML事件**。

## 安装RowStore ##
您可以从下载的RPM文件（仅限RHEL）或从下载的ZIP文件安装用于生产的RowStore。以下部分介绍如何使用下载的ZIP文件安装独立的RowStore产品。<br/>

> 注意：请参阅**升级步骤**升级现有的GemFire XD 1.4.x安装

- **Linux：从ZIP文件安装RowStore** 此过程介绍如何在单个计算机或VM上安装RowStore软件，用于对等客户端或客户端/服务器部署。重复此过程以在要运行RowStore成员的每个物理机或虚拟机上安装RowStore。

### Linux：从ZIP文件安装RowStore ###
此过程介绍如何在单个计算机或VM上安装RowStore软件，用于对等客户端或客户端/服务器部署。重复此过程以在要运行RowStore成员的每个物理机或虚拟机上安装RowStore。<br/>

- 先决条件

- 程序

- ZIP文件安装的安装后配置

- 获取修改的开源代码库

#### 先决条件 ####

1. 确认您的系统满足支持的配置和系统要求中描述的硬件和软件要求。

#### 程序 ####


1. 从RowStore下载页面 [https://github.com/SnappyDataInc/snappydata/releases](https://github.com/SnappyDataInc/snappydata/releases)，下载snappydata-XXX-bin.zip其中file XXX对应的产品版本。


1. 更改到您下载RowStore软件的目录，然后解压ZIP文件：


	- UNIX和Linux（Bourne和Korn shell - sh，ksh，bash）。如果使用命令行，请键入以下命令：

			$ unzip snappydata-XXX-bin.zip -d path_to_product
其中XXX对应于您正在安装的RowStore的产品版本。例如：

			$ unzip snappydata-0.5.0-bin.zip -d /opt/snappydata/rowstore
或者，使用适合您的操作系统的另一个ZIP提取工具解压缩.zip文件。


1. 将RowStore bin和sbin目录添加到您的路径。例如：
	- Linux的：
			
			export PATH=$PATH:/opt/snappydata/rowstore/snappydata-0.5.0-bin/bin:/opt/snappydata/rowstore/snappydata-0.5.0-bin/sbin
该`snappy-shell`脚本自动设置相对于安装目录的类路径。


1. 重复此过程以在要运行RowStore成员的每个不同计算机上安装RowStore。


1. 通过执行`snappy-shell version`命令验证本地安装。您应该收到显示RowStore安装的输出，类似于：

		$ snappy-shell version
		SnappyData Platform Version 0.5.0
		    SnappyData RowStore 1.5.0GA
		    SnappyData Column Store 0.5.0
		    SnappyData AQP 0.5.0

#### ZIP文件安装的安装后配置 ####
虽然RowStore软件安装在每台机器上，但您必须执行其他步骤才能在机器上配置和运行RowStore成员：

1. 按照**步骤中的说明进行计划和配置RowStore部署**，以配置每台机器上的一个或多个RowStore成员，并在新的分布式系统中启动这些成员。


## 升级RowStore ##

> 注意： RowStore 1.5不支持从GemFire XD 1.4.x / SQLFire1.1.x滚动升级，并使用RPM分发进行升级。

- **升级之前** 本节提供开始升级RowStore之前需要了解的信息。

- **从SQLFire 1.1.x手动升级到SnappyData RowStore 1.5** 如果您的操作系统是Linux，请参阅这些说明，从SQLFire 1.1.x手动升级到SnappyData 0.6.1 / RowStore 1.5。

- **从GemFire XD 1.4.x手动升级到SnappyData RowStore 1.5** 如果您的操作系统是Linux，请参阅这些说明，从GemFire XD 1.4.x手动升级到SnappyData 0.6.1 / RowStore 1.5

### 升级之前 ###
本部分提供开始升级到SnappyData RowStore 1.5之前需要了解的信息。<br/>

> 注意：在进入生产之前，使用新的RowStore 1.5版本彻底测试系统。不支持从RowStore 1.5升级到先前版本的RowStore

- 阅读RowStore **发行说明**，了解所有功能更改和升级注意事项。

- 确认您的系统符合运行RowStore的要求。请参阅**支持的配置和系统要求**。

- 如果您现有的RowStore部署使用DBSynchronizer，则在升级期间，如果客户端正在同步的表上执行DML，则可能会发生故障切换相关问题（例如RDBMS中的重复行）。请参阅**DBSynchronizer文档中的故障转移**和**升级如何影响同步**。

- 了解如何为系统配置环境变量。

### 从SQLFire 1.1.x手动升级到SnappyData RowStore 1.5 ###
在本文档中，您可以找到有关如何手动将SQLFire 1.1.x升级到SnappyData 0.6.1 / RowStore 1.5的信息。它是配置和自定义系统的分步指南。本指南假定您对Linux系统有基本的了解。<br/>

- **先决条件**

- **升级过程**
#### 先决条件 ####


- [从SnappyData发行页面（snappydata-0.6.1）](https://github.com/SnappyDataInc/snappydata/releases)下载分发文件

- 确保在开始生产之前，您已经使用新版本彻底测试了您的开发系统
#### 升级过程 ####
- 升级前任务

- 下载和提取产品分发

- 开始升级

- 表CHAR主键的附加步骤

- 重建对象

- 验证升级成功

#### 步骤1：预升级任务 ####
在停止SQLFire群集之前，您必须删除该过程，删除所有表的装入程序，停止，分离和删除侦听器。

- **停止和删除对象**

- **关闭系统**

- **备份所有数据目录**

#### 停止和删除对象 ####
停止并删除依赖于SQLFire软件包的对象（侦听器，过程和加载器等）如果您的系统使用任何存储过程，行加载程序，写入程序，异步事件侦听器或引用SQLFire软件包名称（`com.vmware.sqlfire`或者 `com.pivotal.sqlfire`）的函数，那么您必须先停止并从系统中删除这些对象，然后再开始升级过程。<br/>

例如，如果要从SQLFire进行`built-in com.vmware.sqlfire.callbacks.DBSynchronizer`升级，并且使用该实现配置了DBSynchronizer，则必须删除此侦听器实现。<br/>

对使用较旧的SQLFire（`com.vmware.sqlfire`或 `com.pivotal.sqlfire`）软件包名称的任何数据库对象执行以下步骤：

1. 确保您有必要的DDL命令来重新创建过程，异步事件侦听器，行装载器和写入器。通常，这些命令存储在SQL脚本中，用于创建数据库模式。例如，现有的DBSynchronizer可能是使用语句创建的：

		create asynceventlistener testlistener
		( listenerclass 'com.vmware.sqlfire.callbacks.DBSynchronizer'  initparams     'com.mysql.jdbc.Driver,jdbc:mysql://localhost:3306/gfxddb,SkipIdentityColumns=true,user=sqlfuser,secret=25325ffc3345be8888eda8156bd1c313' 
		 ) server groups (dbsync);
类似地，可以使用命令创建行加载器：

		call sys.attach_loader(‘schema’, ‘table’, ‘com.company.RowLoader’, null);


1. 如果您没有使用DDL命令，请使用sqlf write-schema-to-sql命令将现有的DDL命令写入SQL文件，并验证是否存在必需的命令。


1. 使用SYS.STOP ASYNC EVENT_LISTENER来停止使用旧API的侦听器或DBSynchronizer实现。例如：

		call sys.stop_async_event_listener('TESTLISTENER');
更改使用停止的侦听器或DBSynchronizer的表从表中删除侦听器。例如：

		alter table mytable set asynceventlistener();


1. 使用适当的`DROP`命令（`DROP PROCEDURE`，`DROP ASYNCEVENTLISTENER`，或`DROP FUNCTION`），或系统程序（`SYS.REMOVE_LISTENER`，`SYS.REMOVE_LOADER`，`SYS.REMOVE_WRITER`）以去除使用较旧的API程序和听者的配置。例如：

		drop asynceventlistener testlistener;
类似地，对于行加载器，使用sys.remove_loader系统过程删除它们。例如：

		call  sys.remove_loader(‘schema’, ‘table’)


1. 修改您的过程或监听器代码以使用较新的`com.pivotal.gemfirexd`包前缀，并根据需要重新编译。如果您正在使用内置的DBSynchronizer实现，则只需修改DDL SQL脚本即可在CREATE ASYNCEVENTLISTENER命令中指定较新的实现。例如，一个新的DDL可以是：

		-- Upgraded DBSynchronizer DDL
		create asynceventlistener testlistener
		(
		listenerclass 'com.pivotal.gemfirexd.callbacks.DBSynchronizer' 
		Initparams 'com.mysql.jdbc.Driver,jdbc:mysql://localhost:3306/gfxddb,SkipIdentityColumns=true,user=sqlfuser,secret=25325ffc3345be8888eda8156bd1c313' 
		)
		server groups (dbsync);
上述DDL在升级完成后执行以创建DBSynchronizer。

#### 关闭系统 ####
使用关闭全部和定位器停止命令来停止所有成员。例如：

	$ sqlf shut-down-all -locators=localhost[10101]
	$ sqlf locator stop -dir=$HOME/locator

#### 备份所有数据目录 ####
在此步骤中，您将创建所有数据目录的备份。例如：

	$cp -R node1-server /backup/snappydata/
	$cp -R node2-server /backup/snappydata/
这里`node1-server`和`node2-server`是SQLFire服务器的数据目录。


### 步骤2：下载和提取产品分发 ###


1. 从SnappyData发行页面下载分发二进制文件（snappydata-0.6.1 / RowStore1.5.2）（如果尚未完成），并将文件的内容提取到计算机上的合适位置。例如对于Linux：

		$ unzip snappydata-0.6.1-bin.zip -d <path_to_product>
这里 是您要安装产品的目录。重复此步骤以在要运行SnappyData RowStore成员的每个不同计算机上安装RowStore。或者，SnappyData RowStore也可以安装在所有成员可访问的NFS位置。

1. 如果PATH变量设置为SQLFire的路径，请确保将其更新为`snappydata-0.6.1-bin`。将SnappyData RowStore bin * 和* sbin目录添加到路径中，如下所述：

		$ export PATH=$PATH:/path_to_product/snappydata-0.6.1-bin/bin:/path_to_product/snappydata-0.6.1-bin/sbin

### 步骤3：开始升级 ###
- **设置无密码SSH**

- **为SnappyData服务器和定位器创建配置文件**

- **启动SnappyData RowStore集群**

#### 设置无密码SSH ####

SnappyData提供了用于启动/停止集群的实用程序脚本（位于产品分发的sbin目录中）。这些脚本使用SSH在不同的机器上执行命令，SnappyData服务器将要启动/停止。建议您在执行这些脚本的系统上设置无密码的SSH。要设置无密码SSH，请参阅以下文档：[https://github.com/SnappyDataInc/snappydata#setting-up-passwordless-ssh](https://github.com/SnappyDataInc/snappydata#setting-up-passwordless-ssh)

#### 为SnappyData服务器和定位器创建配置文件 ####
在此步骤中，您将为服务器和定位器创建配置文件，然后使用这些文件使用备份的数据目录启动Snappydata RowStore集群（定位器和服务器）。<br/>

如果所有成员无法访问NFS位置，则需要遵循每台机器的进程。如果设置了无密码SSH，则可以从一个主机启动集群。<br/>

这些配置文件包含要启动定位器/服务器的节点的主机名以及启动属性。<br/>

**要配置定位器：**

1. 在里面 **/snappydata-0.6.1-bin/conf**目录，使副本**locators.template**，并将其重命名为定位器。
编辑定位器文件将其设置为SQLFire定位器的上一个位置（使用该`-dir`选项）

		localhost -dir=<log-dir>/snappydata/locator

1. 如果两个定位器在不同的节点上运行，则添加两行以设置定位器目录：

		node1 -dir=<log-dir>/snappydata/node1-locator
		node2 -dir=<log-dir>/snappydata/node2-locator

**配置服务器：**

1. 在 /snappydata-0.6.1-bin/conf，制作servers.template文件的副本，并将其重命名为服务器。

1. 编辑服务器文件以将其设置为SQLFire服务器的上一个位置（使用该`-dir`选项）。您可以使用`-server-group`选项配置服务器组。

		localhost -dir=<log-dir>/snappydata/server -server-groups=cache

	如果有两台服务器在不同的节点上运行，则添加两行服务器目录的行：

		node1 -dir=<log-dir>/snappydata/node1-server -server-groups=cache
		node2 -dir=<log-dir>/snappydata/node2-server  -server-groups=cache

	> 注意： `-heap-size`属性替换SQLFire`-initial-heap`和/或`-max-heap`属性。SnappyData不再支持“-initial-heap”和“-max-heap”属性。

	在SQLFire中，如果要使用属性文件（sqlfire.properties）来配置各种属性（如身份验证），请将该文件重命名为snappydata.properties。此外，属性前缀已更改为snappydata而不是sqlfire。<br/>
	或者，可以添加这些属性上述CONF /定位器和CONF /服务器中的每个主机名的前面文件。

#### 启动SnappyData RowStore集群 ####
创建配置文件后，使用以下命令启动SnappyData RowStore：

	snappy-start-all rowstore
上述命令使用conf / servers和conf / locators文件中提到的属性启动SnappyData服务器和定位器。该rowstore选项指示SnappyData不启动引导节点，当SnappyData用于SnappyData特定扩展（如列存储，运行spark作业等）时使用该节点）

> 注意：通过执行命令snappy-status-all，确保分布式系统正在运行

例如：

	$snappy-status-all
	SnappyData Locator pid: 13750 status: running
	SnappyData Server pid: 13897 status: running
	SnappyData Server pid: 13905 status: running
	  Distributed system now has 2 members.
	  Other members: localhost(13750:locator)<v0>:10811
	SnappyData Leader pid: 0 status: stopped

### 步骤4：表CHAR主键的附加步骤 ###

> 注意：注意：只有在主键中具有CHAR类型的表格时，才必须执行此步骤。

由于CHAR键的空白填充处理已更改，因此此步骤是必需的。如果在主键（单个或复合）中有CHAR类型的表，请为每个表执行以下操作。<br/>

1. 创建与原始源表具有相同模式的临时表：

	snappy>create table app.member_tmp as select * from app.member with no data;

1. 将原始数据源表中的数据插入到新的临时表中：

	snappy>insert into app.member_tmp select * from app.member;

1. 截断原始源表：

	snappy>truncate table app.member;

1. 将临时表中的数据插入原始源表：

	snappy>insert into app.member_tmp select * from app.member;

1. 删掉临时表：

	snappy>drop table app.member_tmp; 

### 步骤5：重建对象 ###
在升级的初始步骤，你已经删除但依赖于对象`com.vmware.sqlfire`或者 `com.pivotal.sqlfire`，如在- 停止和删除对象。<br/>

在此步骤中，您使用修改的DDL SQL脚本来重新创建存储过程，异步事件监听器实现，使用新的`com.pivotal.gemfirexd`包名称的行装载器/写入器。

1. 替换旧的jar文件：分布式系统运行后，使用sqlj.replace_jar系统过程替换包含侦听器/ rowloader / writer / procedure实现的旧jar。<br/>
例如：

		call sqlj.replace_jar('/home/user1/lib/tours.jar', 'app.sample1')

1. 重新创建对象：例如，要还原DBSynchronizer示例配置：

		gfxd> create asynceventlistener testlistener
		(
		listenerclass 'com.pivotal.gemfirexd.callbacks.DBSynchronizer' 
		initparams   'com.mysql.jdbc.Driver,jdbc:mysql://localhost:3306/gfxddb,SkipIdentityColumns=true,user=gfxduser,secret=25325ffc3345be8888eda8156bd1c313' 
		)
		server groups (dbsync);
		gfxd> alter table mytable set asynceventlistener (testlistener);
		gfxd> call sys.start_async_event_listener('TESTLISTENER');
类似地附加行加载器：

		call sys.attach_loader(‘schema’, ‘table’, ‘com.company.RowLoader’, null’); 
### 步骤6：验证升级成功 ###
系统启动，您可以启动查询`snappy-shell`查看升级状态。所述sqlf壳由替换`snappy-shell`在SnappyData命令。例如：

		snappy-shell
		>SnappyData version 1.5.0-SNAPSHOT
		snappy> connect client '127.0.0.1:1527';
		Using CONNECTION0
		snappy> show tables;
		TABLE_SCHEM           |TABLE_NAME                     |TABLE_TYPE  |REMARKS            
		-------------------------------------------------------------------------------------
		SYS                   |ASYNCEVENTLISTENERS            |SYSTEM TABLE|                   
		SYS                   |GATEWAYRECEIVERS               |SYSTEM TABLE|                   
		SYS                   |GATEWAYSENDERS                 |SYSTEM TABLE|                   
		SYS                   |SYSALIASES                     |SYSTEM TABLE|                   
		SYS                   |SYSCHECKS                      |SYSTEM TABLE|                   
		SYS                   |SYSCOLPERMS                    |SYSTEM TABLE|                   
		SYS                   |SYSCOLUMNS                     |SYSTEM TABLE|                   
		SYS                   |SYSCONGLOMERATES               |SYSTEM TABLE|                   
		SYS                   |SYSCONSTRAINTS                 |SYSTEM TABLE|                   
		SYS                   |SYSDEPENDS                     |SYSTEM TABLE|                   
		SYS                   |SYSDISKSTORES                  |SYSTEM TABLE|                   
		SYS                   |SYSFILES                       |SYSTEM TABLE|                   
		SYS                   |SYSFOREIGNKEYS                 |SYSTEM TABLE|                   
		SYS                   |SYSHDFSSTORES                  |SYSTEM TABLE|                   
		SYS                   |SYSKEYS                        |SYSTEM TABLE|                   
		SYS                   |SYSROLES                       |SYSTEM TABLE|                   
		SYS                   |SYSROUTINEPERMS                |SYSTEM TABLE|                   
		SYS                   |SYSSCHEMAS                     |SYSTEM TABLE|                   
		SYS                   |SYSSTATEMENTS                  |SYSTEM TABLE|                   
		SYS                   |SYSSTATISTICS                  |SYSTEM TABLE|                   
		SYS                   |SYSTABLEPERMS                  |SYSTEM TABLE|                   
		SYS                   |SYSTABLES                      |SYSTEM TABLE|                   
		SYS                   |SYSTRIGGERS                    |SYSTEM TABLE|                   
		SYS                   |SYSVIEWS                       |SYSTEM TABLE|                   
		SYSIBM                |SYSDUMMY1                      |SYSTEM TABLE|
您可以执行`show members`查询查看正在运行的成员：

	snappy> show members;
该show members命令在其输出中显示集群的所有成员（如定位器，数据存储）。您可以对表执行查询（例如从表中选择count（*））来验证内容。

### 升级后任务 ###
升级后，执行以下任务：

- JDBC连接字符串前缀已更改`jdbc:sqlfire`为`jdbc:snappydata`。更新任何自定义或第三方JDBC客户端以使用正确的连接字符串格式。

- 运行所有必需的应用程序，以测试您的开发系统是否正常运行与升级版本的SnappyData。

### 从GemFire XD 1.4.x手动升级到SnappyData RowStore 1.5 ###
在本文档中，您可以找到有关如何将GemFire XD 1.4.x手动升级到SnappyData 0.6.1 / RowStore 1.5的信息。它是配置和自定义系统的分步指南。本指南假定您对Linux系统有基本的了解。<br/>

- **先决条件**

- **升级过程**
#### 先决条件 ####
- 从[SnappyData发行页面（snappydata-0.6.1）](https://github.com/SnappyDataInc/snappydata/releases)下载分发文件

- 确保在开始生产之前，您已经使用新版本彻底测试了您的开发系统
#### 升级过程 ####
1. **升级前任务**
1. **下载和提取产品分发**
1. **开始升级**
1. **验证升级成功**

#### 步骤1：预升级任务 ####
##### 关闭系统 #####
使用`shut-down-all`和`locator stop`命令停止所有成员。例如：

	$ gfxd shut-down-all -locators=localhost[10101]
	$ gfxd locator stop -dir=$HOME/locator
##### 备份所有数据目录 #####
在此步骤中，您将创建所有数据目录的备份。例如：

	$cp -R node1-server /backup/snappydata/
	$cp -R node2-server /backup/snappydata/
这里`node1-server`和`node2-server`是的GemFire XD服务器上的数据目录。

#### 步骤2：下载和提取产品分发 ####

1. 从SnappyData发行页面下载分发二进制文件（snappydata-0.6.1 / RowStore1.5.2）（如果尚未完成），并将文件的内容提取到计算机上的合适位置。例如对于Linux：

		$ unzip snappydata-0.6.1-bin.zip -d <path_to_product>
这里 是您要安装产品的目录。重复此步骤以在要运行SnappyData RowStore成员的每个不同计算机上安装RowStore。或者，SnappyData RowStore也可以安装在所有成员可访问的NFS位置。


1. 如果`PATH`变量设置为GemFire XD的路径，请确保将其更新为`snappydata-0.6.1-bin`。将SnappyData RowStore bin和sbin目录添加到您的路径中，如下所述：

		$ export PATH=$PATH:/path_to_product/snappydata-0.6.1-bin/bin:/path_to_product/snappydata-0.6.1-bin/sbin

### 步骤3：开始升级 ###
- **设置无密码SSH**

- **为SnappyData服务器和定位器创建配置文件**

- **启动SnappyData RowStore集群**

#### 设置无密码SSH ####
SnappyData提供了用于启动/停止集群的实用程序脚本（位于产品分发的sbin目录中）。这些脚本使用SSH在不同的机器上执行命令，SnappyData服务器将要启动/停止。建议您在执行这些脚本的系统上设置无密码的SSH。要设置无密码SSH，请参阅以下文档：[https://github.com/SnappyDataInc/snappydata#setting-up-passwordless-ssh](https://github.com/SnappyDataInc/snappydata#setting-up-passwordless-ssh)


#### 为SnappyData服务器和定位器创建配置文件 ####
在此步骤中，您将为服务器和定位器创建配置文件，然后使用它们使用备份的数据目录启动Snappydata RowStore集群（定位器和服务器）。<br/>

如果所有成员无法访问NFS位置，则需要遵循每台机器的进程。如果设置了无密码SSH，则可以从一个主机启动集群。<br/>

这些配置文件包含要启动定位器/服务器的节点的主机名以及启动属性。<br/>

**要配置定位器：**

1. 在里面 /snappydata-0.6.1-bin/conf目录，使副本locators.template，并将其重命名为定位器。

1. 编辑定位器文件将其设置为GemFire XD定位器的上一个位置（使用-dir选项）

		localhost -dir=<log-dir>/snappydata/locator
如果两个定位器在不同的节点上运行，则添加两行以设置定位器目录：

		node1 -dir=<log-dir>/snappydata/node1-locator
		node2 -dir=<log-dir>/snappydata/node2-locator


**配置服务器：**

1. 在 /snappydata-0.6.1-bin/conf，制作servers.template文件的副本，并将其重命名为服务器。


1. 编辑服务器文件以将其设置为GemFire XD服务器的上一个位置（使用-dir选项）。您可以使用-server-group选项配置服务器组。

		localhost -dir=<log-dir>/snappydata/server -server-groups=cache
如果有两台服务器在不同的节点上运行，则添加两行服务器目录的行：

		node1 -dir=<log-dir>/snappydata/node1-server -server-groups=cache
		node2 -dir=<log-dir>/snappydata/node2-server  -server-groups=cache


	> 注意： `-heap-size`属性替换了GemFire XD`-initial-heap`和/或`-max-heap`属性。SnappyData不再支持“-initial-heap”和“-max-heap”属性。

在GemFire XD中，如果您使用属性文件（gemfirexd.properties）配置各种属性（如身份验证），请将该文件重命名为snappydata.properties。此外，属性前缀已更改为snappydata而不是gemfirexd。
或者，可以添加这些属性上述CONF /定位器和CONF /服务器中的每个主机名的前面文件。

#### 启动SnappyData RowStore集群 ####
创建配置文件后，使用以下命令启动SnappyData RowStore：

	snappy-start-all rowstore
上述命令使用conf / servers和conf / locators文件中提到的属性启动SnappyData服务器和定位器。该rowstore选项指示SnappyData不启动引导节点，当SnappyData用于SnappyData特定扩展（如列存储，运行spark作业等）时使用该节点）



> 注意：通过执行命令snappy-status-all，确保分布式系统正在运行。

例如：

	$snappy-status-all
	SnappyData Locator pid: 13750 status: running
	SnappyData Server pid: 13897 status: running
	SnappyData Server pid: 13905 status: running
	  Distributed system now has 2 members.
	  Other members: localhost(13750:locator)<v0>:10811
	SnappyData Leader pid: 0 status: stopped

### 步骤4：验证升级成功 ###
系统启动，您可以启动查询`snappy-shell`来查看升级的状态。所述gfxd壳由替换`snappy-shell`在SnappyData命令。例如：

	snappy-shell
	>SnappyData version 1.5.0-SNAPSHOT
	snappy> connect client '127.0.0.1:1527';
	Using CONNECTION0
	snappy> show tables;
	TABLE_SCHEM           |TABLE_NAME                     |TABLE_TYPE  |REMARKS            
	-------------------------------------------------------------------------------------
	SYS                   |ASYNCEVENTLISTENERS            |SYSTEM TABLE|                   
	SYS                   |GATEWAYRECEIVERS               |SYSTEM TABLE|                   
	SYS                   |GATEWAYSENDERS                 |SYSTEM TABLE|                   
	SYS                   |SYSALIASES                     |SYSTEM TABLE|                   
	SYS                   |SYSCHECKS                      |SYSTEM TABLE|                   
	SYS                   |SYSCOLPERMS                    |SYSTEM TABLE|                   
	SYS                   |SYSCOLUMNS                     |SYSTEM TABLE|                   
	SYS                   |SYSCONGLOMERATES               |SYSTEM TABLE|                   
	SYS                   |SYSCONSTRAINTS                 |SYSTEM TABLE|                   
	SYS                   |SYSDEPENDS                     |SYSTEM TABLE|                   
	SYS                   |SYSDISKSTORES                  |SYSTEM TABLE|                   
	SYS                   |SYSFILES                       |SYSTEM TABLE|                   
	SYS                   |SYSFOREIGNKEYS                 |SYSTEM TABLE|                   
	SYS                   |SYSHDFSSTORES                  |SYSTEM TABLE|                   
	SYS                   |SYSKEYS                        |SYSTEM TABLE|                   
	SYS                   |SYSROLES                       |SYSTEM TABLE|                   
	SYS                   |SYSROUTINEPERMS                |SYSTEM TABLE|                   
	SYS                   |SYSSCHEMAS                     |SYSTEM TABLE|                   
	SYS                   |SYSSTATEMENTS                  |SYSTEM TABLE|                   
	SYS                   |SYSSTATISTICS                  |SYSTEM TABLE|                   
	SYS                   |SYSTABLEPERMS                  |SYSTEM TABLE|                   
	SYS                   |SYSTABLES                      |SYSTEM TABLE|                   
	SYS                   |SYSTRIGGERS                    |SYSTEM TABLE|                   
	SYS                   |SYSVIEWS                       |SYSTEM TABLE|                   
	SYSIBM                |SYSDUMMY1                      |SYSTEM TABLE|
您可以执行show members查询以查看正在运行的成员：

	snappy> show members;
该show members命令在其输出中显示集群的所有成员（如定位器，数据存储）。您可以对表执行查询（例如从表中选择count（*））来验证内容。

### 升级后任务 ###
升级后，执行以下任务：

- JDBC连接字符串前缀已更改`jdbc:gemfirexd`为`jdbc:snappydata`（旧jdbc:gemfirexd前缀适用于向后兼容性原因）。更新任何自定义或第三方JDBC客户端以使用正确的连接字符串格式。

- 运行所有必需的应用程序，以测试您的开发系统是否正常运行与升级版本的SnappyData。

## 使用JDBC工具连接到RowStore ##
第三方JDBC工具可以帮助您浏览表中的数据，发出SQL命令，设计新表等。您可以配置这些工具以使用RowStore JDBC瘦客户端驱动程序连接到RowStore分布式系统。<br/>

虽然设置每个工具的说明有所不同，但建立连接的一般过程涉及配置JDBC客户端驱动程序和设置JDBC连接URL属性。按照以下基本步骤：<br/>



1. 在第三方工具中，选择配置新的驱动程序，然后选择包含RowStore JDBC客户端驱动程序的gemfirexd-client.jar文件。此文件安装在您的RowStore安装的lib目录中。

1. 如果该工具没有自动选择一个驱动程序类，通常可以选择从JAR文件中选择一个类。对于RowStore，选择com.pivotal.snappydata.jdbc.ClientDriver类。

1. 为了使用客户端驱动程序，您必须为RowStore分布式系统指定JDBC连接URL。客户端驱动程序的基本URL格式为：

		jdbc:snappydata://hostname:port/
其中，主机名和端口对应`-client-bind-address`及`-client-port`一个RowStore服务器或定位的价值在您的分布式系统。<br/>

1. 您的工具可能需要您指定用于连接到系统的用户名和密码。如果RowStore服务器或定位器启用身份验证（使用-auth-provider引导属性），请输入有效的用户名和密码组合以连接到分布式系统。<br/>
如果禁用身份验证，请将“应用程序”指定为用户名和密码值或任何其他临时值。


	> 注意：当您不提供数据库对象的模式名称时，RowStore将使用JDBC连接中指定的用户名作为模式名称。RowStore使用“APP”作为默认模式。如果您的系统未启用身份验证，您可以为用户名和密码指定“APP”，以保持与默认模式行为的一致性。

有关使用第三方JDBC工具配置RowStore的完整示例，请参阅在RowStore社区站点上[使用SQuirreL SQL与RowStore](https://support.pivotal.io/hc/en-us/articles/201569486-Using-SQuirreL-SQL-with-GemFire-XD)。

## 快速入门教程 ##
学习配置和使用RowStore功能，如表复制和分区，将数据持久保存到本地磁盘存储和HDFS，以及动态调整集群大小。<br/>

- **主要步骤** 本教程分为以下步骤，介绍如何在多个Java VM上设置RowStore服务器集群，然后在集群中分发数据。按照所示顺序执行步骤。

- **创建RowStore集群** 在本教程中，您可以设置并启动两个RowStore服务器的集群。

- **连接到群集使用** `snappy-shell` snappy -shell和Pulse实现基于Apache Derby ij工具的命令行工具。您可以使用`snappy-shell`连接到RowStore集群并运行脚本或交互式查询。您使用`snappy-shell`或
- snappy-shell.bat脚本执行gfxd。

- **创建复制表和执行查询** 默认情况下，RowStore将表复制到群集的成员。在本教程中，您将创建复制RowStore集群的新表。

- **实现分区策略** 在本教程中，您可以删除ToursDB模式中的所有表，然后使用新的分区和复制策略重新创建它们。

- **将表保持到磁盘** 默认情况下，RowStore分布式系统只会保留您创建的表和索引的数据字典。这些持久性文件存储在数据字典的每个定位器和数据存储，在加入分布式系统的子目录。但是，表数据默认情况下不会保留; 如果关闭所有RowStore成员，则下次启动时表将为空。在本教程中，您将将表数据保存到RowStore磁盘存储文件。

- **将服务器添加到群集和停止服务器** RowStore以灵活的方式管理数据，使您能够在运行时扩展或收缩群集以支持不同的加载。要向群集动态添加更多容量，请添加新的服务器成员并指定该`-rebalance`选项。
执行其他任务 完成RowStore教程后，您可以执行相关的教程任务来探索产品的其他区域。

### 主要步骤 ###
本教程分为以下步骤，其中介绍了如何在多个Java VM上设置RowStore服务器集群，然后在集群中分发数据。按照所示顺序执行步骤。<br/>

|步|描述|
|:--|:--|
|步骤1|[创建一个RowStore集群](http://rowstore.docs.snappydata.io/docs/getting_started/topics/tutorial_create_cluster.html)|
|第2步|[使用snappy-shell和Pulse连接到群集](http://rowstore.docs.snappydata.io/docs/getting_started/topics/tutorial_connect_thin_client.html)|
|步骤3|[创建复制表和执行查询](http://rowstore.docs.snappydata.io/docs/getting_started/topics/tutorial_create_replicated_tables.html)|
|步骤4|[实施分区策略](http://rowstore.docs.snappydata.io/docs/getting_started/topics/tutorial_partitioning_strategy.html)|
|步骤5|[将表固定到磁盘](http://rowstore.docs.snappydata.io/docs/getting_started/topics/tutorial_persist.html)|
|步骤6|[将服务器添加到群集和停止服务器](http://rowstore.docs.snappydata.io/docs/getting_started/topics/tutorial_rebalance.html)|

按照安装RowStore中所述在本地计算机上安装RowStore。

### 创建一个RowStore集群 ###
在本教程中，您将设置并启动两个RowStore服务器的集群。

程序

1. 首先在安装RowStore的计算机或VM上打开命令提示符或终端窗口。


1. 本教程中的初始RowStore集群包含两个独立的RowStore服务器成员。移动到您的主目录，并为每个服务器创建一个新的目录：

		$ cd ~
		$ mkdir server1 server2
每个服务器将使用其本地目录来写入日志文件，备份磁盘存储文件，持久数据的datadictionary目录以及单个状态文件.snappyserver.ser。

1. 要管理与可用的RowStore服务器成员的客户端连接，本教程中的集群使用RowStore定位器成员。为定位器成员创建一个新目录：

		$ mkdir locator

1. 定位器维护集群中可用服务器的列表，并在服务器加入并离开集群时更新该列表。定位器还可以跨所有可用服务器负载平衡客户端连接。
要启动或停止RowStore定位器或RowStore服务器，请使用该snappy-shell脚本（适用于Linux平台）。<br/>
如果您使用ZIP文件分发安装，则必须首先确保RowStore bin目录的路径是PATH环境变量的一部分。例如，在Linux平台上输入：

		$ export PATH=$PATH:path-to-snappydata-installation/bin


1. 当您启动RowStore分布式系统时，始终从启动定位器成员开始。使用该`snappy-shell locator`命令在指定的目录中启动定位器：

		$ snappy-shell locator start -dir=$HOME/locator -peer-discovery-port=10101 -client-port=1527 -jmx-manager-start=true -jmx-manager-http-port=7075
		Starting RowStore Locator using peer discovery on: 0.0.0.0[10101]
		Starting network server for RowStore Locator at address localhost/127.0.0.1[1527]
		Logs generated in /home/gpadmin/locator/snappylocator.log
		RowStore Locator pid: 41456 status: running
它-peer-discovery-port定义了一个唯一的连接端口，该RowStore分布式系统的所有成员都可以用于彼此进行通信。始终使用唯一的-peer-discovery-port编号，以避免加入已在网络上运行的群集。如果其他人可能会在您的网络上评估RowStore，请选择10101以外的端口号。<br/>
在-client-port定义了客户端应用程序使用连接到该定位器连接端口。在本教程中，所有RowStore成员都运行在本地主机地址上，并使用不同的端口号来定义唯一的连接。在生产环境中，您可以通过使用-peer-discovery-address和-client-bind-address选项为成员发现或客户端访问指定专用网络接口。<br/>
指定-jmx-manager-start=true在定位器中启动一个嵌入式JMX管理器。通过启动JMX Manager，您可以通过Pulse图形界面监视和浏览RowStore分布式系统。
您可以在定位器启动消息中验证对等端和客户端连接。


	> 注意：首先启动定位器成员，当新服务器加入并离开分布式系统时，定位器可以从头开始管理集群成员身份。定位器成员也应该是您要停止RowStore分布式系统时关闭的最后一个进程。

	> 注意：作为替代，RowStore成员可以使用多播消息发现彼此。然而，重要的RowStore功能（如WAN复制和用户身份验证）要求RowStore系统使用定位器而不是多播来进行发现。请参阅配置发现机制。



1. 现在使用该`snappy-shell rowstore server`命令启动RowStore服务器成员并将它们连接到分布式系统：

		$ snappy-shell rowstore server start -dir=$HOME/server1 -locators=localhost[10101] -client-port=1528
		Starting RowStore Server using locators for peer discovery: localhost[10101]
		Starting network server for RowStore Server at address localhost/127.0.0.1[1528]
		Logs generated in /home/gpadmin/server1/snappyserver.log
		RowStore Server pid: 42051 status: running
		  Distributed system now has 2 members.
		  Other members: 192.168.125.140(41456:locator):20431
		$ snappy-shell rowstore server start -dir=$HOME/server2 -locators=localhost[10101] -client-port=1529
		Starting RowStore Server using locators for peer discovery: localhost[10101]
		Starting network server for RowStore Server at address localhost/127.0.0.1[1529]
		Logs generated in /home/gpadmin/server2/snappyserver.log
		RowStore Server pid: 42528 status: running
		  Distributed system now has 3 members.
		  Other members: 192.168.125.140(41456:locator):20431, 192.168.125.140(42051:datastore)<v1>:13160
在每个命令中，该`-locators`选项定义了用于加入RowStore分布式系统的对等体发现地址。生产部署通常使用多个定位器成员，在这种情况下，您可以在启动服务器时指定定位器主机[端口]连接的逗号分隔列表。）
再次`-client-port`指出，每个服务器将在唯一连接（localhost：1528和localhost：1529）上侦听瘦客户机。然而，在随后的教程中，所有客户端将使用定位器进行连接，而不是直接连接到服务器。<br/>
启动消息显示集群成员详细信息。<br/>
默认情况下，新的RowStore服务器将数据托管为数据存储，并自动添加到默认服务器组。您可以选择使用服务器组引导属性在启动时指定服务器组成员资格。<br/>

1. 在将任何数据添加到集群之前，请查看分布式系统已启动的新成员目录的内容：

		$ ls $HOME/locator $HOME/server1 $HOME/server2
		/Users/yozie/locator:
		BACKUPGFXD-DEFAULT-DISKSTORE.if     DRLK_IFGFXD-DEFAULT-DISKSTORE.lk    snappylocator.log                     start_snappylocator.log
		BACKUPGFXD-DEFAULT-DISKSTORE_1.crf  datadictionary/                     locator10101state.dat
		BACKUPGFXD-DEFAULT-DISKSTORE_1.drf  snappylocator-01-01.log               locator10101views.log
		
		/Users/yozie/server1:
		BACKUPGFXD-DEFAULT-DISKSTORE.if     BACKUPGFXD-DEFAULT-DISKSTORE_1.drf  datadictionary/                     start_snappyserver.log
		BACKUPGFXD-DEFAULT-DISKSTORE_1.crf  DRLK_IFGFXD-DEFAULT-DISKSTORE.lk    snappyserver.log
		
		/Users/yozie/server2:
		BACKUPGFXD-DEFAULT-DISKSTORE.if     BACKUPGFXD-DEFAULT-DISKSTORE_1.drf  datadictionary/                     start_snappyserver.log
		BACKUPGFXD-DEFAULT-DISKSTORE_1.crf  DRLK_IFGFXD-DEFAULT-DISKSTORE.lk    snappyserver.log
请注意以BACKUPGFXD-DEFAULT-DISKSTORE开头的文件。这些是RowStore磁盘存储文件，可用于将表数据保存到磁盘。上面列出的文件是为默认服务器组创建的，其中包括加入分布式系统的所有数据存储成员。您还可以创建命名服务器组和磁盘存储，但如果不将持久性表和队列专门分配给命名服务器组，则默认组和磁盘存储将用于持久性。还要注意每个成员的/ datadictionary子目录：

		$ ls $HOME/locator/datadictionary $HOME/server1/datadictionary $HOME/server2/datadictionary
		/Users/yozie/locator/datadictionary:
		BACKUPGFXD-DD-DISKSTORE.if     BACKUPGFXD-DD-DISKSTORE_1.crf  BACKUPGFXD-DD-DISKSTORE_1.drf  DRLK_IFGFXD-DD-DISKSTORE.lk
		
		/Users/yozie/server1/datadictionary:
		BACKUPGFXD-DD-DISKSTORE.if     BACKUPGFXD-DD-DISKSTORE_1.crf  BACKUPGFXD-DD-DISKSTORE_1.drf  DRLK_IFGFXD-DD-DISKSTORE.lk
		
		/Users/yozie/server2/datadictionary/:
		BACKUPGFXD-DD-DISKSTORE.if     BACKUPGFXD-DD-DISKSTORE_1.crf  BACKUPGFXD-DD-DISKSTORE_1.drf  DRLK_IFGFXD-DD-DISKSTORE.lk
创建一组单独的磁盘存储文件来维护分布式系统的数据字典。无论您是否打算将表数据保留到磁盘，RowStore始终保持数据字典，因此，您执行的任何数据定义语言（DDL）即使通过完全关闭分布式系统也可以创建对象。<br/>
还要注意，定位器还创建了磁盘存储文件。与数据存储成员一样，定位器维护集群的持久性文件。这很重要，因为您可能希望停止或重新启动数据存储成员进行维护。保持运行的定位器进程继续维护其持久的数据文件，并且稍后重新加入集群的数据存储可以使用来自一个或多个磁盘的磁盘存储文件来使其持久性文件保持最新（称为“恢复”其持久性文件）定位器或其他数据存储成员。<br/>
持久性教程中提供了有关持久性和磁盘存储文件的更多信息。要记住的重要事情是，您不应该删除，移动或重命名持久性文件或目录，因为在重新启动并加入分布式系统时，RowStore成员可以访问这些文件至关重要。<br/>

### 使用snappy-shell和Pulse连接到群集 ###
`snappy-shell`实现基于Apache Derby ij工具的命令行工具。您可以使用`snappy-shell`连接到RowStore集群并运行脚本或交互式查询。您使用`snappy-shell`脚本执行gfxd 。<br/>

#### 程序 ####

1. 在启动RowStore群集的同一命令提示符或终端窗口中，切换到quickstart目录：

		$ cd path-to-snappydata-installation/quickstart/store

	替代路径到snappydata安装与实际路径snappydata安装。该快速入门目录包含例如SQL脚本文件，您将在本教程以后使用。


1. 开始互动snappy-shell会话：

		$ snappy-shell
		SnappyData version 1.5.0
		snappy> 
此命令启动交互式shell并显示提示符：`snappy>`。


1. 打印可用`snappy-shell`命令的简要列表：

		snappy> help;


1. 要使用JDBC瘦客户机驱动程序连接到RowStore集群，请使用该`connect client`命令并指定RowStore定位器的主机和端口号：

		snappy> connect client 'localhost:1527';
请注意，RowStore不具有“数据库”的概念。连接到RowStore集群时，连接到的分布式系统由JDBC，ODBC或ADO.NET 连接中指定的定位器（或mcast-port）定义。


1. 使用以下命令查看“sys”模式中的表：

		snappy> show tables in sys;
		TABLE_SCHEM      |TABLE_NAME                    |REMARKS            
		--------------------------------------------------------------------
		SYS              |ASYNCEVENTLISTENERS           |                   
		SYS              |GATEWAYRECEIVERS              |                   
		SYS              |GATEWAYSENDERS                |                   
		SYS              |SYSALIASES                    |                   
		SYS              |SYSCHECKS                     |                   
		SYS              |SYSCOLPERMS                   |                   
		SYS              |SYSCOLUMNS                    |                   
		SYS              |SYSCONGLOMERATES              |                   
		SYS              |SYSCONSTRAINTS                |                   
		SYS              |SYSDEPENDS                    |                   
		SYS              |SYSDISKSTORES                 |                   
		SYS              |SYSFILES                      |                   
		SYS              |SYSFOREIGNKEYS                |                   
		SYS              |SYSHDFSSTORES                 |                   
		SYS              |SYSKEYS                       |                   
		SYS              |SYSROLES                      |                   
		SYS              |SYSROUTINEPERMS               |                   
		SYS              |SYSSCHEMAS                    |                   
		SYS              |SYSSTATEMENTS                 |                   
		SYS              |SYSSTATISTICS                 |                   
		SYS              |SYSTABLEPERMS                 |                   
		SYS              |SYSTABLES                     |                   
		SYS              |SYSTRIGGERS                   |                   
		SYS              |SYSVIEWS                      |                   
		
		24 rows selected
您将使用许多这些表中的信息来查看有关RowStore集群的信息和统计信息。

1. sys.members表存储有关RowStore成员的信息。执行以下查询以查看分配给两个RowStore服务器和您启动的定位器的唯一ID：

		snappy> select id from sys.members;
		ID                                                                             
		-------------------------------------------------------------------------------
		192.168.125.147(39598)<v2>:23162                                               
		192.168.125.147(39486)<v1>:40258                                               
		192.168.125.147(39381)<v0>:50344                                               
		
		3 rows selected
输出显示在命令行启动成员时记录的成员进程ID（本教程示例中的42528，41456和42051）。


1. 您还可以连接到在定位器成员的嵌入式Web服务器中运行的RowStore Pulse应用程序。打开浏览器并访问URL [http://localhost:7075/pulse](http://localhost:7075/pulse)。输入用户名和密码的“admin”，然后点击登录：

![](http://rowstore.docs.snappydata.io/docs/common/images/tutorial-pulse-login.png)

1. 在主集群视图窗口中，单击机器节点以显示在localhost上运行的三个RowStore成员：

![](http://rowstore.docs.snappydata.io/docs/common/images/tutorial-pulse-cluster.png)

有关可用脉冲视图的更多信息，请参阅**RowStore Pulse**。

### 创建复制表和执行查询 ###
默认情况下，RowStore将表复制到集群的成员。在本教程中，您将创建复制RowStore集群的新表。

#### 程序 ####

1. 在同一个snappy-shell会话中，运行ToursDB_schema.sql脚本来创建与ToursDB示例数据库相关联的表：

		snappy> run 'ToursDB_schema.sql';
你看到DDL输出如：

		CREATE TABLE AIRLINES
		   (
		      AIRLINE CHAR(2) NOT NULL CONSTRAINT AIRLINES_PK PRIMARY KEY,
		      AIRLINE_FULL VARCHAR(24),
		      BASIC_RATE DOUBLE PRECISION,
		      DISTANCE_DISCOUNT DOUBLE PRECISION,
		      BUSINESS_LEVEL_FACTOR DOUBLE PRECISION,
		      FIRSTCLASS_LEVEL_FACTOR DOUBLE PRECISION,
		      ECONOMY_SEATS INTEGER,
		      BUSINESS_SEATS INTEGER,
		      FIRSTCLASS_SEATS INTEGER
		   );
		0 rows inserted/updated/deleted
		snappy> -- ************************************************************\
		[...]


1. 运行loadTables.sql脚本以使用数据填充表：

		snappy> run 'loadTables.sql';
脚本输出完成：

		snappy> insert into FLIGHTAVAILABILITY values ('US1357',2,'2004-04-18',0,0,3);
		1 row inserted/updated/deleted


1. 输入以下命令以显示您创建的表名（APP模式中的表）：

		snappy> show tables in APP;
		TABLE_SCHEM      |TABLE_NAME                    |REMARKS            
		--------------------------------------------------------------------
		APP              |AIRLINES                      |                   
		APP              |CITIES                        |                   
		APP              |COUNTRIES                     |                   
		APP              |FLIGHTAVAILABILITY            |                   
		APP              |FLIGHTS                       |                   
		APP              |FLIGHTS_HISTORY               |                   
		APP              |MAPS                          |                   
		
		7 rows selected


1. 默认情况下，您创建的新表和加载的数据将在两个RowStore服务器上进行复制。您可以通过查询sys.systables中的信息来检查表是否已分区或复制。使用以下查询来查看RowStore已分配给刚创建的表的数据策略：

		snappy> select tablename, datapolicy from sys.systables where tableschemaname='APP';
		TABLENAME                                                     |DATAPOLICY     
		------------------------------------------------------------------------------
		FLIGHTS_HISTORY                                               |REPLICATE      
		MAPS                                                          |REPLICATE      
		FLIGHTAVAILABILITY                                            |REPLICATE      
		FLIGHTS                                                       |REPLICATE      
		CITIES                                                        |REPLICATE      
		COUNTRIES                                                     |REPLICATE      
		AIRLINES                                                      |REPLICATE      
		
		7 rows selected
输出显示您创建的每个ToursDB表都被复制。如果您不在`PARTITION BY`语句中使用该子句，则`CREATE TABLERowStore` 将默认复制表。
分区表和复制表提供了有关在RowStore中创建表的更多信息。


1. RowStore提供与其他数据管理产品中可用的查询功能相似的查询功能。例如，以下命令执行一个简单的查询：

		snappy> SELECT city_name, country, language FROM cities WHERE language LIKE '%ese';
		CITY_NAME               |COUNTRY                   |LANGUAGE        
		--------------------------------------------------------------------
		Rio de Janeiro          |Brazil                    |Portuguese      
		Sao Paulo               |Brazil                    |Portuguese      
		Hong Kong               |China                     |Chinese         
		Shanghai                |China                     |Chinese         
		Osaka                   |Japan                     |Japanese        
		Tokyo                   |Japan                     |Japanese        
		Lisbon                  |Portugal                  |Portuguese      
		
		7 rows selected
以下查询执行表之间的连接：

		snappy> SELECT city_name, countries.country, region, language
		FROM cities, countries
		WHERE cities.country_iso_code = countries.country_iso_code AND language LIKE '%ese';
		CITY_NAME           |COUNTRY            |REGION               |LANGUAGE     
		----------------------------------------------------------------------------
		Rio de Janeiro      |Brazil             |South America        |Portuguese   
		Sao Paulo           |Brazil             |South America        |Portuguese   
		Hong Kong           |China              |Asia                 |Chinese      
		Shanghai            |China              |Asia                 |Chinese      
		Osaka               |Japan              |Asia                 |Japanese     
		Tokyo               |Japan              |Asia                 |Japanese     
		Lisbon              |Portugal           |Europe               |Portuguese   
		
		7 rows selected


1. 虽然使用脚本创建的表默认是复制的，但它们不是持久的。退出gfxd会话并关闭两个数据存储成员：

		snappy> exit;
		$ snappy-shell rowstore server stop -dir=$HOME/server1
		$ snappy-shell rowstore server stop -dir=$HOME/server2
服务器停止后，重新启动并连接到分布式系统：

		$ snappy-shell rowstore server start -dir=$HOME/server1 -locators=localhost[10101] -client-port=1528
		$ snappy-shell rowstore server start -dir=$HOME/server2 -locators=localhost[10101] -client-port=1529
		$ snappy
		snappy> connect client 'localhost:1527';
重复上一步骤的查询：

		snappy> SELECT city_name, countries.country, region, language
		FROM cities, countries
		WHERE cities.country_iso_code = countries.country_iso_code AND language LIKE '%ese';
		CITY_NAME               |COUNTRY                   |REGION                    |LANGUAGE        
		-----------------------------------------------------------------------------------------------
		
		0 rows selected
默认情况下，数据字典保持不变，因此您可以重复以前的查询; 所有数据库对象仍然通过集群重新启动。但是，除非在创建表时特别启用持久性，否则不会保留表数据。持久性教程中提供了更多信息。<br/>

### 实施分区策略 ###
在本教程中，您可以删除ToursDB模式中的所有表，然后使用新的分区和复制策略重新创建它们。<br/>

本教程中的ToursDB模式与“STAR”模式类似，只有几个事实表和几个维度表。维度表通常很小，更改不常见，但通常用于连接查询。维度表是复制RowStore成员的好候选对象，因为连接查询可以并行执行。<br/>

AIRLINES，CITIES，COUNTRIES和MAPS表被视为维度表，并在RowStore集群中进行复制。在本教程中，假设应用程序经常基于选择为分区列的FLIGHT_ID列来频繁地加入这些相关表。<br/>

FLIGHTS，FLIGHTS_HISTORY和FLIGHTAVAILABILITY是事实表，它们将被分区。您将共同定位这些表，以确保与FLIGHT_ID相关联的所有行都保留在单个分区中。此步骤确保基于所选航班的频繁连接查询被修剪到单个成员并有效执行。<br/>

#### 程序 ####

1. 在单独的终端窗口或GUI编辑器中，打开/ quickstart目录中的create_colocated_schema.sql文件，以检查附带的DDL命令。SQL脚本首先删除模式中的现有表：

		DROP TABLE AIRLINES;
		DROP TABLE CITIES;
		DROP TABLE COUNTRIES;
		DROP TABLE FLIGHTAVAILABILITY;
		DROP TABLE FLIGHTS;
		DROP TABLE MAPS;
		DROP TABLE FLIGHTS_HISTORY;
可以使用本教程前面部分中相同的基本CREATE语句来复制维度表。但是，REPLICATE为了清楚起见，此脚本显式添加了关键字。例如：

		CREATE TABLE AIRLINES
		   (
		      AIRLINE CHAR(2) NOT NULL CONSTRAINT AIRLINES_PK PRIMARY KEY,
		      AIRLINE_FULL VARCHAR(24),
		      BASIC_RATE DOUBLE PRECISION,
		      DISTANCE_DISCOUNT DOUBLE PRECISION,
		      BUSINESS_LEVEL_FACTOR DOUBLE PRECISION,
		      FIRSTCLASS_LEVEL_FACTOR DOUBLE PRECISION,
		      ECONOMY_SEATS INTEGER,
		      BUSINESS_SEATS INTEGER,
		      FIRSTCLASS_SEATS INTEGER
		   ) REPLICATE;
FLIGHTS表根据FLIGHT_ID列进行分区：

		CREATE TABLE FLIGHTS
		   (
		      FLIGHT_ID CHAR(6) NOT NULL ,
		      SEGMENT_NUMBER INTEGER NOT NULL ,
		      ORIG_AIRPORT CHAR(3),
		      DEPART_TIME TIME,
		      DEST_AIRPORT CHAR(3),
		      ARRIVE_TIME TIME,
		      MEAL CHAR(1),
		      FLYING_TIME DOUBLE PRECISION,
		      MILES INTEGER,
		      AIRCRAFT VARCHAR(6),
		      CONSTRAINT FLIGHTS_PK PRIMARY KEY (
		                            FLIGHT_ID,
		                            SEGMENT_NUMBER),
		      CONSTRAINT MEAL_CONSTRAINT
		          CHECK (meal IN ('B', 'L', 'D', 'S'))
		
		   )
		   PARTITION BY COLUMN (FLIGHT_ID);
剩余的事实表也被划分，并与FLIGHTS表共同配置。例如：

		CREATE TABLE FLIGHTAVAILABILITY
		   (
		      FLIGHT_ID CHAR(6) NOT NULL ,
		      SEGMENT_NUMBER INTEGER NOT NULL ,
		      FLIGHT_DATE DATE NOT NULL ,
		      ECONOMY_SEATS_TAKEN INTEGER DEFAULT 0,
		      BUSINESS_SEATS_TAKEN INTEGER DEFAULT 0,
		      FIRSTCLASS_SEATS_TAKEN INTEGER DEFAULT 0,
		      CONSTRAINT FLIGHTAVAIL_PK PRIMARY KEY (
		                                  FLIGHT_ID,
		                                  SEGMENT_NUMBER,
		                                  FLIGHT_DATE),
		      CONSTRAINT FLIGHTS_FK2 Foreign Key (
		            FLIGHT_ID,
		            SEGMENT_NUMBER)
		         REFERENCES FLIGHTS (
		            FLIGHT_ID,
		            SEGMENT_NUMBER)
		
		   )
		   PARTITION BY COLUMN (FLIGHT_ID)
		   COLOCATE WITH (FLIGHTS);


1. 在snappy-shell会话中，执行create_colocated_schema.sql脚本以删除现有表，并使用新的分区和复制策略重新创建它们。执行loadTables.sql以数据填充表：

		snappy> run 'create_colocated_schema.sql';
		snappy> run 'loadTables.sql';

1. 确认表已创建：

		snappy> show tables in APP;
		TABLE_SCHEM      |TABLE_NAME                    |REMARKS            
		--------------------------------------------------------------------
		APP              |AIRLINES                      |                   
		APP              |CITIES                        |                   
		APP              |COUNTRIES                     |                   
		APP              |FLIGHTAVAILABILITY            |                   
		APP              |FLIGHTS                       |                   
		APP              |FLIGHTS_HISTORY               |                   
		APP              |MAPS                          |                   
		
		7 rows selected


1. 验证是否复制或分区各个表：

		snappy> select tablename, datapolicy from sys.systables where tableschemaname='APP';
		TABLENAME                                                     |DATAPOLICY     
		------------------------------------------------------------------------------
		AIRLINES                                                      |REPLICATE      
		COUNTRIES                                                     |REPLICATE      
		CITIES                                                        |REPLICATE      
		FLIGHTS_HISTORY                                               |PARTITION      
		FLIGHTAVAILABILITY                                            |PARTITION      
		FLIGHTS                                                       |PARTITION      
		MAPS                                                          |REPLICATE      
		
		7 rows selected


1. FLIGHTS表和其他表现在RowStore集群分区。使用分区表，您可以使用该dsid()功能显示承载表的成员ID：

		snappy> select distinct dsid() as id from flights;
		ID                                                                             
		-------------------------------------------------------------------------------
		192.168.125.147(39486)<v1>:40258                                               
		192.168.125.147(39598)<v2>:23162                                               
		
		2 rows selected
	> 注意：对于复制表，`dsid（）'只显示承载表（接收查询的成员）的唯一RowStore成员的成员ID。
		
1. 现在使用DSID功能来查看该RowStore服务器上存储了多少行分区FLIGHT表。例如：

		snappy> select count(*) memberRowCount, dsid() from flights group by dsid();
		MEMBERROWCOUNT|2                                                               
		-------------------------------------------------------------------------------
		282           |192.168.125.147(39486)<v1>:40258                                
		260           |192.168.125.147(39598)<v2>:23162                                
		
		2 rows selected


	> 注意： RowStore分布式系统的确切行数可能不同。


1. 并行执行两个分区成员的连接。

		snappy> select flight_date, economy_seats_taken, firstclass_seats_taken from flights f, flightavailability fa 
		      where f.flight_id = fa.flight_id and f.flight_id = 'AA1116';
		FLIGHT_DATE|ECONOMY_SEATS_TAKEN|FIRSTCLASS_SEATS_TAKEN
		------------------------------------------------------
		2004-03-31 |2                  |2                     
		2004-04-11 |1                  |1                     
		2004-04-12 |2                  |2                     
		2004-04-15 |5                  |0                     
		2004-04-20 |10                 |0                     
		2004-04-23 |1                  |1                     
		2004-04-24 |2                  |2                     
		2004-05-03 |11                 |0                     
		2004-05-05 |1                  |1                     
		2004-05-06 |2                  |2                     
		2004-05-17 |1                  |1                     
		2004-05-18 |2                  |2                     
		2004-05-29 |1                  |1                     
		2004-05-30 |2                  |2                     
		
		14 rows selected
返回结合的结果。因为表被FLIGHT_ID分区，所以连接的执行被修剪到存储值“AA1116”的分区。您可以使用查询验证flight_id“AA1116”位于只有一个数据存储上：

		snappy> select count(*), dsid() from flights where flight_id = 'AA1116';
		1          |2                                                                  
		-------------------------------------------------------------------------------
		1          |192.168.125.147(39598)<v2>:23162                                   
		
		1 row selected

### 将表固定到磁盘 ###
默认情况下，RowStore分布式系统只会保留您创建的表和索引的数据字典。这些持久性文件存储在数据字典的每个定位器和数据存储，在加入分布式系统的子目录。但是，表数据默认情况下不会保留; 如果关闭所有RowStore成员，则下次启动时表将为空。在本教程中，您将将表数据保存到RowStore磁盘存储文件。<br/>

#### 程序 ####

1. 在单独的终端窗口或GUI编辑器中，检查create_persistent_schema.sql脚本的内容。请注意，此脚本在每个CREATE TABLE语句中都使用PERSISTENT关键字。例如，在脚本中，创建COUNTRIES表的语句如下：

		CREATE TABLE COUNTRIES
		   (
		      COUNTRY VARCHAR(26) NOT NULL CONSTRAINT COUNTRIES_UNQ_NM Unique,
		      COUNTRY_ISO_CODE CHAR(2) NOT NULL CONSTRAINT COUNTRIES_PK PRIMARY KEY,
		      REGION VARCHAR(26),
		      CONSTRAINT COUNTRIES_UC
		        CHECK (country_ISO_code = upper(country_ISO_code) )
		
		   ) REPLICATE PERSISTENT;
当您在下一步执行脚本时，现有架构将被删除，并替换为RowStore在服务器成员目录中的操作日志文件的架构。

1. 在`snappy-shell`会话中，执行create_persistent_schema.sql脚本，然后加载表数据：

		snappy> run 'create_persistent_schema.sql';
		snappy> run 'loadTables.sql';


1. 退出snappy-shell会话：

		snappy> exit;


1. 因为数据是持久的，所以当您重新启动数据存储时，RowStore会从磁盘恢复数据。首先使用该`shut-down-all`命令关闭集群中的所有数据存储：

		$ snappy-shell shut-down-all -locators=localhost[10101]
		Connecting to distributed system: locators=localhost[10101]
		Successfully shut down 2 members
		
		$ snappy-shell locator stop -dir=$HOME/locator
		The RowStore Locator has stopped.


1. 使用以下命令重新启动数据存储：

		$ snappy-shell locator start -dir=$HOME/locator -peer-discovery-port=10101 -client-port=1527 -jmx-manager-start=true -jmx-manager-http-port=7075 -enable-network-partition-detection=true
		Starting network server for RowStore Locator at address localhost/127.0.0.1[1527]
		Logs generated in /Users/dyozie/locator/snappylocator.log
		RowStore Locator pid: 25723 status: waiting
		Waiting for table GFXD_PdxTypes (DiskId: fbefd3dd-2195-4936-b454-f4bcb6175037, Location: /Users/dyozie/locator/./datadictionary) on:
		 [DiskId: 0c4f0b73-036e-413a-a05d-bb01a521d3c0, Location: /Users/dyozie/server1/./datadictionary]
		$ snappy-shell rowstore server start -dir=$HOME/server1 -locators=localhost[10101] -client-port=1528 -sync=false -enable-network-partition-detection=true
		Starting network server for RowStore Server at address localhost/127.0.0.1[1528]
		Logs generated in /Users/dyozie/server1/snappyserver.log
		RowStore Server pid: 25734 status: waiting
		Waiting for table APP.AIRLINES (DiskId: 9c0ba4d7-d9f8-469b-83d0-f8417977aeb3, Location: /Users/dyozie/server1/.) on:
		 [DiskId: a78adff5-2a96-4ac6-a9cf-09515b6ad385, Location: /Users/dyozie/server2/.]
		$ snappy-shell rowstore server start -dir=$HOME/server2 -locators=localhost[10101] -client-port=1529 -sync=false -enable-network-partition-detection=true
		Starting RowStore Server using locators for peer discovery: localhost[10101]
		Starting network server for RowStore Server at address localhost/127.0.0.1[1529]
		Logs generated in /Users/dyozie/server2/snappyserver.log
		RowStore Server pid: 25743 status: running
		  Distributed system now has 3 members.
		  Other members: 10.84.17.128(25734:datastore)<v1>:31259, 10.84.17.128(25723:locator)<ec><v0>:16099


	> 注意：默认情况下，如果成员依赖于另一个服务器或定位器进行磁盘存储同步，则RowStore服务器和定位器将开始处于“等待”状态。任何从属服务器或定位器启动后，服务器自动继续引导。如果从shell脚本启动服务器，可以使用snappy-shell rowstore locator wait或snappy-shell rowstore server wait命令等待成员达到运行状态，然后再继续。或者，您可以使用MEMBERS系统表中的STATUS列监视RowStore成员的当前状态。

	注意启动消息：

		Waiting for table APP.AIRLINES (DiskId: 9c0ba4d7-d9f8-469b-83d0-f8417977aeb3, Location: /Users/dyozie/server1/.) on: 
		 [DiskId: a78adff5-2a96-4ac6-a9cf-09515b6ad385, Location: /Users/dyozie/server2/.]
这不是错误信息。它表示您正在开始的RowStore成员正在等待另一个成员在线上可用，以确保分布式系统使用任何持久数据的最新版本。<br/>
还要注意，启动命令包括新选项`-enable-network-partition-detection=true`。启用网络分区是任何持续数据的系统的最佳做法，即使只持续数据字典。当禁用网络分区检测时，网络分段可能导致不同的数据存储组在其磁盘存储文件中保留不一致的数据。有关详细信息，请参阅会员启动问题。

1. 现在验证持久性表是否已重新加载：

		$ snappy-shell
		snappy> connect client 'localhost:1527';
		snappy> select distinct dsid() as id from flights;
		ID                                                                             
		-------------------------------------------------------------------------------
		10.84.17.128(25734)<v1>:31259                                                                                                   
		10.84.17.128(25743)<v2>:21552                                                
		
		2 rows selected
		snappy> select count(*) memberRowCount, dsid() from flights group by dsid();
		MEMBERROWCOUNT|2                                                               
		-------------------------------------------------------------------------------
		279           |10.84.17.128(25734)<v1>:31259                                                                                                   
		263           |10.84.17.128(25743)<v2>:21552                                  
		
		2 rows selected


	> 注意： RowStore分布式系统的确切行数可能不同。



1. 由于教程集群不使用服务器组，因此在使用PERSISTENT子句创建表时，无法指定命名服务器组。当您不使用命名服务器组时，RowStore将持久数据存储为默认服务器组。如前面的教程所述，默认的服务器组磁盘存储文件位于每个数据存储和定位器成员的成员目录中。文件以BACKUPGFXD-DEFAULT-DISKSTORE开头：

		$ ls $HOME/locator $HOME/server1 $HOME/server2
		/Users/yozie/locator:
		BACKUPGFXD-DEFAULT-DISKSTORE.if     DRLK_IFGFXD-DEFAULT-DISKSTORE.lk    snappylocator.log                     start_snappylocator.log
		BACKUPGFXD-DEFAULT-DISKSTORE_1.crf  datadictionary/                     locator10101state.dat
		BACKUPGFXD-DEFAULT-DISKSTORE_1.drf  snappylocator-01-01.log               locator10101views.log
		
		/Users/yozie/server1:
		BACKUPGFXD-DEFAULT-DISKSTORE.if     BACKUPGFXD-DEFAULT-DISKSTORE_1.drf  datadictionary/                     start_snappyserver.log
		BACKUPGFXD-DEFAULT-DISKSTORE_1.crf  DRLK_IFGFXD-DEFAULT-DISKSTORE.lk    snappyserver.log
		
		/Users/yozie/server2:
		BACKUPGFXD-DEFAULT-DISKSTORE.if     BACKUPGFXD-DEFAULT-DISKSTORE_1.drf  datadictionary/                     start_snappyserver.log
		BACKUPGFXD-DEFAULT-DISKSTORE_1.crf  DRLK_IFGFXD-DEFAULT-DISKSTORE.lk    snappyserver.log


1. 磁盘存储文件对于维护系统中数据的完整性至关重要。维护所有磁盘存储文件的备份（对于数据字典以及表数据），您可以从磁盘损坏或存储设备故障中恢复。您还可以根据需要使用磁盘存储备份文件将RowStore成员移动到新硬件。为了备份磁盘存储文件，首先创建一个用于存储备份的专用目录：

		$ mkdir $HOME/snappydata-backups
然后使用gfxd backup命令创建完整的磁盘存储备份：

		$ snappy-shell backup $HOME/snappydata-backups -locators=localhost[10101]
		Connecting to distributed system: locators=localhost[10101]
		The following disk stores were backed up:
		    20f8d17e-4f23-4615-bc73-950cd639b6f2 [ward-4.local:/Users/yozie/server1/.]
		    b2912f8e-3fdd-4fa7-a035-9066a98f2590 [ward-4.local:/Users/yozie/server1/./datadictionary]
		    539314c2-e5b5-42fa-a251-9bea69e15849 [ward-4.local:/Users/yozie/locator/.]
		    50f25e1f-6441-40ac-9bed-5148dd56c71f [ward-4.local:/Users/yozie/locator/./datadictionary]
		    74667f34-e800-4313-bb0d-a64415f52beb [ward-4.local:/Users/yozie/server2/./datadictionary]
		    1027151d-33ba-4d71-8ec7-24b6a8cff5e1 [ward-4.local:/Users/yozie/server2/.]
		Backup successful.
此命令在指定备份时间的目录中创建磁盘存储文件的完整备份。为分布式系统中的每个定位器和数据存储创建子目录：

		$ ls $HOME/snappydata-backups
		2014-12-16-16-33-15/
		$ ls $HOME/snappydata-backups/2014-12-16-16-33-15
		10_84_17_128_25723_ec_v0_16099/ 10_84_17_128_25734_v1_31259/    10_84_17_128_25743_v2_21552/

1. 备份目录中的每个成员目录都包含一个还原脚本，您可以使用该脚本将文件还原到给定的目录。恢复脚本执行测试以确保它不会覆盖同名的现有磁盘存储文件。如果没有相同名称的文件，脚本将备份复制到指定的目录。这是为server2创建的还原脚本：

		$ cd $HOME/snappydata-backups/2014-12-16-16-33-15/10_84_17_128_25723_ec_v0_16099/
		$ cat restore.sh 
		#!/bin/bash -e
		cd `dirname $0`
		
		#Restore a backup of gemfire persistent data to the location it was backed up
		#from.
		#This script will refuse to restore if the original data still exists.
		
		#This script was automatically generated by the gemfire backup utility.
		
		#Test for existing originals. If they exist, do not restore the backup.
		test -e '/Users/dyozie/locator/./datadictionary/BACKUPGFXD-DD-DISKSTORE.if' && echo 'Backup not restored. Refusing to overwrite /Users/dyozie/locator/./datadictionary/BACKUPGFXD-DD-DISKSTORE.if' && exit 1
		test -e '/Users/dyozie/locator/./BACKUPGFXD-DEFAULT-DISKSTORE.if' && echo 'Backup not restored. Refusing to overwrite /Users/dyozie/locator/./BACKUPGFXD-DEFAULT-DISKSTORE.if' && exit 1
		
		#Restore data
		mkdir -p '/Users/dyozie/locator/./datadictionary'
		cp -rp 'diskstores/GFXD-DD-DISKSTORE_fbefd3dd-2195-4936-b454-f4bcb6175037/dir0'/* '/Users/dyozie/locator/./datadictionary'
		mkdir -p '/Users/dyozie/locator/.'
		cp -rp 'diskstores/GFXD-DEFAULT-DISKSTORE_1695f17d-b150-4e06-b744-3b24287a10cf/dir0'/* '/Users/dyozie/locator/.'


1. 如果您需要将RowStore成员移动到新硬件，则最新的还原脚本可以帮助您复制该成员所需的磁盘存储文件。例如，您可以模拟移动“server2”成员，首先关闭服务器，然后重命名（或删除）其工作目录：

		$ snappy-shell rowstore server stop -dir=$HOME/server2
		The RowStore Server has stopped.
		$ mv $HOME/server2 $HOME/deleted-server-2
在本教程中需要移动或删除目录，因为还原脚本会在复制文件之前检查现有的服务器目录。要还原server2，请运行其关联的还原脚本：

		$ cd $HOME/snappydata-backups/2014-12-16-16-33-15/10_84_17_128_25743_v2_21552
		$ ./restore.sh 
然后，您可以使用还原的磁盘存储文件重新启动服务器：

		$ snappy-shell rowstore server start -dir=$HOME/server2 -locators=localhost[10101] -client-port=1529 -enable-network-partition-detection=true
		Starting RowStore Server using locators for peer discovery: localhost[10101]
		Starting network server for RowStore Server at address localhost/127.0.0.1[1529]
		Logs generated in /Users/dyozie/server2/snappyserver.log
		RowStore Server pid: 25903 status: running
		  Distributed system now has 3 members.
		  Other members: 10.84.17.128(25723:locator)<ec><v0>:16099, 10.84.17.128(25734:datastore)<v1>:31259
请注意，在生产环境中，您的工作目录可能包含其他工件，如自动启动脚本或`gemfirexd.properties`用于配置引导属性的文件。您需要备份和还原这些文件以及磁盘存储文件以获得完整的备份解决方案。

### 执行其他任务 ###
完成RowStore教程后，您可以执行相关的教程任务来探索产品的其他方面。

#### 浏览toursDB数据库 ####
示例toursDB数据库管理有关航班运输的调度航班的信息。如果您需要快速访问数据库试图RowStore功能，toursDB DDL和数据脚本萨雷在可用时使用快速启动目录。要查看toursDB模式，查看toursdb_readme.html在文件快速入门子目录。

#### 浏览RowStore语言扩展 ####
本教程通过对toursDB DDL的简单修改介绍了分区，复制和全局化的基础知识。请参阅语句开始学习在RowStore DDL中实现的其他功能。您可以复制并修改create_colocated_schema.sql根据需要实现新的功能。

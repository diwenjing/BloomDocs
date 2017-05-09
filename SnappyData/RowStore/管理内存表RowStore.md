# 管理内存表 #
设计RowStore数据库还包括基于实际表值和索引的大小估算数据的内存需求，RowStore对数据所需的开销，数据的总体使用模式。此外，您可以加载具有代表性数据的表，然后查询RowStore SYS.MEMORYANALYTICS表，以获取有关各个表和索引所需的堆内存的详细信息。<br/>



- **内存容量规划概述** 本主题描述了确定在数据存储上分配多少内存的高级过程。


- **估计RowStore堆开销和表内存要求** RowStore需要不同数量的堆内存开销，每个表和索引条目取决于是否持有表数据或配置表以溢出到磁盘。如果您已经有代表性的数据，请查询SYS.MEMORYANALYTICS表以获得更准确的存储数据所需内存的图片。


- **查看SYS.MEMORYANALYTICS中的内存使用** RowStore包括显示RowStore成员中各表和索引使用的内存的仪器。您可以通过从客户端连接查询SYS.MEMORYANALYTICS虚拟表来查看此内存使用情况。

## 内存容量规划概述 ##
本主题描述了确定在数据存储上分配多少内存的高级过程。<br/>

对于将作为数据存储的所有成员，必须确定内存要求和表配置，以便在生产部署的硬件资源时提供最佳性能。在开发过程中，您通常会遍历模式版本，并启用或禁用某些产品功能（例如事件队列，WAN等）的使用，并且数据量通常较小。但是，当您开始使用更接近生产数据集的较大数据集时，建议执行以下步骤：<br/>

1. 使用测试集群创建您的模式，以及您将在生产中使用的索引和所有其他功能，以消耗内存（例如，异步事件队列，WAN等）。
1. 加载代表性生产数据的一小部分。尝试使用尽可能靠近生产数据的数据集。
1. 使用分配的堆创建测试集群。根据您的生产数据集大小的估算，为样本数据集过多分配。请参阅**估算开销估计的RowStore堆开销和表内存要求**。
1. 使用内置的系统程序或您自己的自定义程序加载其余的数据。
1. 检查SYS.MEMORYANALYTICS表。该表应提供有关表，索引等消耗的内存的信息。请参阅**在SYS.MEMORYANALYTICS中查看内存使用情况**
1. 根据您在SYS.MEMORYANALYTICS表中查看的信息根据需要测试和修改内存分配。确定您将在生产中使用的分配和配置。

## 将表存储在非堆存储器中 ##
RowStore支持将表数据直接存储在JVM堆（默认值）中，或者在非堆内存中，它指的是不是JVM分配堆的一部分的内存。将表数据存储在堆栈内存中可以扩展内存中的大型数据库，同时提高CPU利用率，避免由JVM垃圾收集引起的暂停。<br/>

通过将表数据存储在堆内存中，可以看到性能提高30％，这是因为在垃圾收集时花费的时间较少。通常，在完成初始开发，迭代多个模式版本以及开始使用更大的数据集之后，您决定是否将表存储在堆内存中。<br/>

- **在非堆内存中存储表的准则** 本主题提供了有关如何以及何时使用堆内存功能的一些准则。
- **配置和使用非堆内存** 本主题介绍如何分配服务器上的堆内存以及如何配置表以使用堆内存。
- **监控关闭内存使用情况** 本主题介绍如何监视堆内存使用情况和RowStore中可用的统计信息。

### 在非堆存储器中存储表的准则 ###
本主题提供了有关如何以及何时使用堆内存功能的一些指导。<br/>

对于需要在内存中保留大量数据的任何部署，建议使用非堆内存。如果您运行的服务器至少具有150GB的RAM，或者您通常运行具有超过32GB的用户数据的JVM，则可能会将表数据保留在堆内存中。<br/>

您可以使用32位或64位体系结构的堆栈内存。<br/>

另外，将表数据存储在非堆内存中可能最有利于具有大小均匀或具有通用大小值模式的行的部署。如果您的表具有明显不同且不可预测的行值大小，或者如果频繁的表更新更改了行值的大小，则堆内存可能无法执行。这些情况可能导致堆栈外存储空间碎片化，从而导致查询性能较差。<br/>

#### 使用非堆存储的附加指南 ####
使用堆内存时，请注意以下准则：<br/>



- 当使用非堆表时，RowStore仅将表行存储在堆内存中。表行包括BLOB和CLOB数据。存储在非堆内存中的每个表行也会消耗一些堆内存，并在计算JVM堆开销时予以考虑。有关详细信息，请参阅估**计RowStore堆开销和表内存要求**。


- RowStore还需要卸载内存来执行某些DML语句和对堆栈内外的表的查询。除了存储行数据所需的存储器之外，还需要此堆栈内存。有关详细信息，请参阅**计算DML和查询执行的非堆内存要求**。


- RowStore可以存储在堆内存中的最大行大小等于堆中存储的表行的最大大小。实质上，您不能通过使用堆内存来存储较大的行，但不需要使用较小的行。如果将BLOB或CLOBS存储在脱离堆表中，BLOB或CLOB将存储在堆内存中的表行中。存储在表行中的BLOB / CLOB的最大大小为〜2GB，但是BLOB或CLOB的大小不会计入表行的最大总大小。


- RowStore存储JVM堆中所有表的索引; 因此，索引将使用相同数量的堆内存，无论您是否将表行存储在堆或非堆中。


- 当您在RowStore数据存储上分配卸载内存时，您应该在同一集群中的所有数据存储上分配相同数量的非堆内存。


- 为了在生产部署中获得最佳性能，请使用snappy-shell rowstore server -lock-memory选项将内存中的堆内存页（和堆内存页）锁定在内存中，特别是在与Hadoop相同的计算机上运行RowStore时。有关详细信息，请参阅支持的配置和系统要求。


- 您不能修改现有表以使用堆内存。要将存储在JVM堆内存中的表数据移动到堆内存中，您需要删除并重新创建表。请参阅将表数据移动到非堆存储器。



- 您可以将堆和非堆表存储在单个数据存储上，但是每个表必须在托管集群中的表的所有数据存储上完全存储在堆内或堆上。


- 如果您创建了分布在集群中的脱离堆表，则随后加入集群的任何数据存储将需要分配非堆内存。如果不在数据存储上分配堆栈内存，则节点将无法启动类似于以下内容的错误消息：

<br/>

    RowStore Server pid: 8831 status: stopped
    Error starting server process: 
    IllegalStateException: The region /APP/AIRLINES was configured to use off heap memory but no off heap memory was configured. - See log file for details.

- 请记住，为脱离堆表配置的任何驱逐设置也可能会影响（并在某些情况下增加）JVM堆内存使用量。例如，如果您使用破坏的驱逐行为与分区表，您可以释放堆内存和堆内存。如果您使用溢出的驱逐动作，您将按预期释放堆内存，但是溢出驱逐也可能导致堆内存使用量增加。一些堆叠外数据用作表上的键，当通过溢出驱逐从堆内存中删除时，该部分数据必须被复制回堆。


- 默认情况下，在数据存储启动时不会分配堆内存，并且不能在已经运行的数据存储上分配堆内存。要分配堆内存，您需要重新启动数据存储。

- 如果您尝试创建一个表，指定数据存储上的OFFHEAP子句，而不分配的堆内存，则会收到类似于以下内容的错误：

<br/>

    ERROR 38000: SQLSTATE=38000,SEVERITY=-1: (Server=localhost[1529],Thread[DRDAConnThread_20,5,snappydata.daemons]) 
    The exception 'java.lang.IllegalStateException: The region /APP/HOTELAVAILABILITY was configured to use off heap memory 
    but no off heap memory was configured.' was thrown while evaluating an expression.
    

### 配置和使用非堆存储器 ###
本主题介绍如何在服务器上分配非堆内存，以及如何配置表以使用堆内存。<br/>

#### 分配离堆内存并创建离堆表 ####



1.对于分布式系统中的所有RowStore数据存储，请在启动命令或gemfirexd.properties文件中设置脱堆内存大小引导属性。例如：`snappy-shell`<br/>

    snappy-shell rowstore server start -dir=server1 -heap-size=10g -off-heap-memory-size=200g -lock-memory

一些其他有效的示例值：

    -off-heap-memory-size=4096m
    -off-heap-memory-size=10485760k
    -off-heap-memory-size=120g

如果指定的值大于2g，则由于表数据存储在2g内存块中，所以分配的千兆位内存的数量应该是2的倍数。<br/>
有关如何估计堆和堆内存的内存分配的指导，请参阅估计RowStore堆开销和表内存要求。<br/>


> 注意：在堆栈内存中托管同一个表的所有数据存储器都应该为“off-heap-memory-size”指定相同的值。

此属性都可以为数据存储区提供堆栈内存存储，并指定分配给非堆存储的最大内存量。每个成员启动时，即时分配堆栈内存，您可以在日志文件条目中进行验证：

    [config 2013/09/10 18:37:25.157 PDT accessor_hs21d_21724 <vm_0_thr_0_accessor1_hs21d_21724> tid=0x2e] 
    Allocating 367001600 bytes of off-heap memory. The maximum size of a single off-heap object is 367001600 bytes.


2.使用`CREATE TABLE`创建新表时，指定`OFFHEAP`子句。例如：

    CREATE TABLE HOTELAVAILABILITY
	    (HOTEL_ID INT NOT NULL, 
	     BOOKING_DATE DATE NOT NULL,
	     ROOMS_TAKEN INT DEFAULT 0,
	     PRIMARY KEY (HOTEL_ID, BOOKING_DATE))
	     OFFHEAP;

如果您创建一个表，其中指定了OFFHEAP子句，但是在接收到该表的所有数据存储上尚未分配非堆内存，则会收到类似于以下内容的错误消息：

    ERROR 38000: SQLSTATE=38000,SEVERITY=-1: (Server=localhost[1529],Thread[DRDAConnThread_20,5,snappydata.daemons]) 
    The exception 'java.lang.IllegalStateException: The region /APP/HOTELAVAILABILITY was configured to use off heap memory 
    but no off heap memory was configured.' was thrown while evaluating an expression.

### 配置卸载内存中的数据的排除和关键阈值 ###
当使用堆栈内存释放空间（如有必要）时，仍然可以使用包括LRU驱逐在内的数据驱逐。`eviction-off-heap-percentage`在数据存储上设置阈值将触发驱逐已经使用LRU_HEAP配置的任何脱离堆表。<br/>

要为存储在非堆内存中的表条目设置LRU驱逐，您可以：<br/>

- -eviction-off-heap-percentage使用gfxd或。启动服务器时 指定
- 使用**SYS.SET_EVICTION_OFFHEAP_PERCENTAGE**和**SYS.SET_EVICTION_OFFHEAP_PERCENTAGE_SG**系统过程。<br/>


如果在分区表上使用DESTROY逐出操作，则可以释放堆内存和堆内存。<br/>

如果使用OVERFLOW逐出动作，您将按预期的方式释放堆内存，但是溢出驱逐也可能导致堆内存使用量增加。一些堆叠外数据用作表上的键，当通过溢出驱逐从堆内存中删除时，该部分数据必须被复制回堆。<br/>

默认情况下，如果`-off-heap-size`在启动snappy-shell rowstore服务器时指定，RowStore将设置默认临界阈值为90％。要在使用堆内存时设置不同的关键内存阈值，您可以：<br/>

- 指定`-critical-offheap-percentage`启动数据存储时的选项。
- 使用**SYS.SET_CRITICAL_OFFHEAP_PERCENTAGE**和**SYS.SET_CRITICAL_OFFHEAP_PERCENTAGE_SG**系统过程。


以下是启动具有驱逐和关键百分比阈值的服务器的示例：<br/>

    snappy-shell rowstore server start -dir=server1 -off-heap-memory-size=200g -critical-off-heap-percentage=99.99 -eviction-off-heap-percentage=85

有关使用gfxd启动**服务器**的更多信息，请参阅**服务器**。


### 将表数据移动到非堆存储器 ###
在不指定OFFHEAP子句的情况下创建表时，不能将表修改为存储在堆内存中的表。同样，您不能将存储在堆内存中的现有表数据移动到堆内存。如果需要切换存储表数据的位置，则需要删除并重新创建表（以类似方式将复制表修改为分区表）。<br/>

以下是将现有表移动到堆内存的示例过程：

1.使用**SYSCS_UTIL.EXPORT_TABLE**内置存储过程导出表数据。

2.成功导出表数据后，删除现有表。

3.如果尚未在承载脱离堆表的所有节点上分配堆内存，请重新启动这些节点并指定-off-heap memory-size引导属性。例如：

    snappy-shell rowstore server start -dir=server1 -off-heap-memory-size=2g

4.重新创建表并指定OFFHEAP子句。例如：

    CREATE TABLE HOTELAVAILABILITY
	    (HOTEL_ID INT NOT NULL, 
	     BOOKING_DATE DATE NOT NULL,
	     ROOMS_TAKEN INT DEFAULT 0,
	     PRIMARY KEY (HOTEL_ID, BOOKING_DATE))
	     OFFHEAP;

5.通过使用**SYSCS_UTIL.IMPORT_TABLE**内置存储过程将表的数据导入到新的脱离堆表中。

### 监控关闭内存使用情况 ###
本主题介绍如何监视堆内存使用情况和RowStore中可用的统计信息。<br/>

GemFire MemberMXBean接口公开了与堆内存使用情况相关的多个统计信息，包括使用和可用的非堆内存量以及存储在堆内存中的对象数量。您还可以观察RowStore在堆内存中使用压缩对象的时间以及脱堆数据的碎片程度。请参见JMX Manager MBeans。<br/>

您可以在Pulse界面中查看其中的一些统计信息，也可以使用JConsole等标准JMX监视工具查看所有这些统计信息。此外，还有几个API可以获取MemberMXBean的统计信息（MBean属性）。<br/>

#### 可用堆堆统计清单 ####
|统计|描述|
|:--|:--|
|compactions|堆栈内存的总次数已经被压缩。如果此统计信息为非零值，则该成员不得不压缩堆内存。这可能表示该成员已经离开堆栈内存了。|
|compactionTime|[压缩堆内存的总时间。除非通过使用enable-time-statistics boot属性启用时间统计信息，否则此值将为零。](http://rowstore.docs.snappydata.io/docs/reference/configuration/ConnectionAttributes.html)|
|碎片|堆内存碎片的百分比。每次执行压缩时都会更新。分段百分比过高表示堆内存充分碎片，从而潜在地难以填满或利用。|
|片段|免费堆内存的片段数。每次压实完成后更新。|
|freeMemory|堆栈内存量（以字节为单位）未被使用。|
|largestFragment|由最后的压缩堆内存发现的最大的内存片段。每次压实完成后更新。|
|maxMemory|堆内存的最大数量，以字节为单位。这是启动时分配的内存量，不会更改。|
|对象|存储在堆内存中的对象数量。|
|||
||RowStore表中的每一行都将为行本身提供一个offheap Object，并为该行中的每个LOB提供一个额外的非堆处理对象。|
|读|堆内存读取总数。此统计信息仅通过读取完整对象而递增。如果一个对象只被部分读取，统计量不会递增。|
|usedMemory|用于存储数据的堆内存量（以字节为单位）。|

#### MemberMXBean脱钩属性 ####
MemberMXBean具有新属性，可以暴露成员的多个堆内存统计信息。您可以使用以下API来获取非堆统计信息的值： - getOffHeapCompactionTime提供compactionTime统计信息的值。- getOffHeapFragmentation提供碎片统计量的值。- getOffHeapFreeSize提供freeSize（freeMemory）统计量的值。- getOffHeapObjects提供对象统计量的值。- getOffHeapUsedSize提供usedSize（usedMemory）统计信息的值。<br/>

要获取非堆栈maxMemory（分配的非堆内存量）统计量的值，请将freeMemory（getOffHeapFreeSize）和usedMemory（getOffHeapUsedSize）的值相结合。<br/>

> 注意： MemberMXBean操作列表GemFireProperties包含的属性“off-heap-memory-size”，其值与maxMemory的值相同。

#### RegionMXBean非堆归属 ####
RegionMXBean listRegionAttributes API包括一个新的布尔属性，指示表是否配置为使用堆内存或常规Java堆内存。listRegionAttributes操作返回的一个属性是enableOffHeapMemory，如果该表配置为使用非堆内存，则该值将为true<br/>

#### 查看脉冲中的非堆用法和表状态 ####
要查看成员使用多少堆栈内存，请选择群集视图。然后选择列表视图图标，然后双击要查看的成员。有关其他信息，请参阅**使用脉冲视图**。<br/>

以下屏幕截图显示了Pulse如何报告会员的堆内存使用情况：<br/>

![](http://rowstore.docs.snappydata.io/docs/common/images/pulse_member_offheap.png)

要查看表是否将数据存储在堆内存中，请选择“数据视图”，然后选择要检查的表。<br/>

以下屏幕截图显示了在Pulse中表示的堆内表： <br/>

![](http://rowstore.docs.snappydata.io/docs/common/images/pulse_offheap_tableview.png)

#### 错误 ####
以下异常与堆内存相关联：<br/>



- OutOfOffHeapMemoryException。抛出此异常时，缓存将关闭，以便在集群中保持一致性。应将每个成员配置为在ResourceManager上使用脱离堆驱逐和关键的非堆叠百分比阈值，以避免获得此异常并防止数据不一致或成员丢失。


- LowMemoryException。当正在使用的堆栈内存量已超过配置的关键堆栈百分比阈值时，在本地和客户端放置操作期间抛出此异常。您可以使用**gfxd start server**命令或使用**SYS.SET_CRITICAL_OFFHEAP_PERCENTAGE**或**SYS.SET_CRITICAL_OFFHEAP_PERCENTAGE_SG**系统过程来指定此阈值。

## 查看SYS.MEMORYANALYTICS中的内存使用情况 ##
RowStore包括显示RowStore成员中各个表和索引使用的内存的工具。您可以通过从客户端连接查询SYS.MEMORYANALYTICS虚拟表来查看此内存使用情况。<br/>

- **访问SYS.MEMORYANALYTICS** 按照此过程访问SYS.MEMORYANALYTICS虚拟表。


- **了解表和索引值** 查询SYS.MEMORYANALYTICS表提供了有关RowStore中的表，索引和内存使用情况的运行时信息。


- **显示开销**，内存分配和总内存占用 您可以查询SYS.MEMORYANALYTICS虚拟表来估计数据存储中的表的内存使用情况。您还可以使用优化器提示来查询表，以获得表和索引的内存占用的更准确的视图。

### 访问SYS.MEMORYANALYTICS ###
按照此过程访问SYS.MEMORYANALYTICS虚拟表。<br/>

如果您在非生产或测试部署系统中查看内存分析，则必须具有代表生产环境的表数据和索引条目，以便准确评估内存占用。创建必要的SQL脚本以自动创建模式并将数据加载到表中。<br/>


#### 程序 ####
按照以下步骤访问SYS.MEMORYANALYTICS表：<br/>

1.启动您的分布式系统成员。<br/>
2.如果您的模式和数据在RowStore系统中不可用（作为持久化数据），请运行任何必需的SQL脚本来创建模式并加载表数据。例如：<br/>

    cd c:\gpxd-dir\quickstart
    gfxd
    snappy> connect client 'localhost:1527';
    snappy> run 'create_colocated_schema.sql';
    snappy> run 'loadTables.sql';
3.连接到RowStore并查询SYS.MEMORYANALYTICS表以查看内存使用情况：

    gfxd
    snappy> connect client 'localhost:1527';
    snappy> select * from sys.memoryanalytics;
    
    TABLE_NAME|INDEX_NAME  |INDEX_TYPE
    |ID |HOST   |CONSTANT_OVERHEAD|ENTRY_SIZE   |KEY_SIZE 
     |VALUE_SIZE   |VALUE_SIZE_OFFHEAP|TOTAL_SIZE   |NUM_ROWS|NUM_KEYS_IN_MEMORY  |NUM_VALUES_IN_MEMORY
      |NUM_VALUES_IN_OFFHEAP|MEMORY
    ----------------------------------------------------------------------------------------------------------------
    
    APP.FLIGHTS_HISTORY   |NULL|NULL  
    |192.168.129.188(5231)<v1>:41602|192.168.129.188|1.688|0.0  |0.0  
     |0.0  |0.0   |1.688|0   |0   |0   
      |0|- 
    
    APP.FLIGHTAVAILABILITY|NULL|NULL  
    |192.168.129.188(5231)<v1>:41602|192.168.129.188|1.688|33.152   |0.0  
     |4.144|45.584|84.568   |518 |0   |0   
      |518  |- 
    
    APP.FLIGHTAVAILABILITY|6__FLIGHTAVAILABILITY__FLIGHT_ID__SEGMENT_NUMBER|LOCAL 
    |192.168.129.188(5231)<v1>:41602|192.168.129.188|0.152|3.352|0.0  
     |3.296|0.0   |6.8  |53  |0   |0   
      |0|- 
    
    APP.MAPS  |NULL|NULL  
    |192.168.129.188(5231)<v1>:41602|192.168.129.188|1.688|0.0  |0.0  
     |0.0  |0.0   |1.688|0   |0   |0   
      |0|- 
    
    APP.MAPS  |3__MAPS__MAP_ID__MAP_NAME   |LOCAL 
    |192.168.129.188(5231)<v1>:41602|192.168.129.188|0.152|0.088|0.0  
     |0.0  |0.0   |0.24 |0   |0   |0   
      |0|- 
    
    APP.FLIGHTS   |NULL|NULL  
    |192.168.129.188(5231)<v1>:41602|192.168.129.188|1.688|30.352   |0.0  
     |36.314   |0.0   |68.354   |542 |0   |542 
      |0|- 
    
    APP.FLIGHTS   |DESTINDEX   |LOCAL 
    |192.168.129.188(5231)<v1>:41602|192.168.129.188|0.152|5.744|0.0  
     |5.176|0.0   |11.072   |87  |0   |0   
      |0|- 
    
    APP.FLIGHTS   |ORIGINDEX   |LOCAL 
    |192.168.129.188(5231)<v1>:41602|192.168.129.188|0.152|5.976|0.0  
     |5.168|0.0   |11.296   |87  |0   |0   
      |0|- 
    
### 了解表和索引值 ###
查询SYS.MEMORYANALYTICS表提供了有关RowStore中的表，索引和内存使用情况的运行时信息。<br/>


#### 表值 ####
每个表在SYS.MEMORYANALYTICS中的单个行由表名称标识，格式为：schema_name。table_name。<br/>

|SYS.MEMORY ANALYTICS列|描述|
|:--|:--|
|TABLE_NAME|使用格式schema_name的表的名称。table_name。|
|INDEX_NAME|索引名称。|
|INDEX_TYPE|与table-local或全局散列索引相关联的索引类型的描述，以及索引是否被排序。|
|ID|托管表的成员的成员ID。|
|主办|数据存储的主机名。|
|CONSTANT_OVERHEAD|一次性内存开销费用，以千字节为单位，由于创建空白表时产生的工件。|
|ENTRY_SIZE|进入开销，以千字节为单位。仅反映将表行保存在内存中所需的内存量，但不包括内存以保持其键值和值。（不包括KEY_SIZE，CONSTANT_OVERHEAD，VALUE_SIZE和VALUE_SIZE_OFFHEAP）。|
|KEY_SIZE|密钥开销，以千字节为单位。请注意，当表设置为溢出到磁盘并且完整行（换句话说，行值）不再存储在内存中时，此列将仅显示非零值。|
|VALUE_SIZE|存储在JVM堆中的表行的大小（以千字节为单位）。（这包括条目大小开销。）|
|VALUE_SIZE_OFFHEAP|存储在堆内存中的表行的大小（以千字节为单位）。|
|TOTAL_SIZE|总大小是以下列的总和（以千字节为单位）：|
||CONSTANT_OVERHEAD|
||ENTRY_SIZE|
||KEY_SIZE|
||VALUE_SIZE|
||VALUE_SIZE_OFFHEAP|
|||
|NUM_ROWS|存储在本地RowStore成员中的总行数。对于分区表，这包括表的所有桶，以及主副本。|
|NUM_KEYS_IN_MEMORY|存储在表中堆的总键数。请注意，当表设置为溢出到磁盘并且完整行（换句话说，行值）不再存储在内存中时，此列将仅显示非零值。|
|NUM_VALUES_IN_MEMORY|存储在表的堆中的行值的总数。|
|NUM_VALUES_IN_OFFHEAP|存储在堆内存中的行值的总数。|
|记忆|目前不使用 占位符列。|

例如，SYS.MEMORYANALYTICS的以下行显示，APP.FLIGHTAVAILABILITY表（正在使用非堆内存）的总体积开销为38.984千字节（KB），总共有518行（总共为518行）（其值为存储在堆内）：<br/>

    TABLE_NAME|INDEX_NAME|INDEX_TYPE
     |ID |HOST   |CONSTANT_OVERHEAD
      |ENTRY_SIZE   |KEY_SIZE |VALUE_SIZE   |VALUE_SIZE_OFFHEAP
       |TOTAL_SIZE   |NUM_ROWS|NUM_KEYS_IN_MEMORY  
    |NUM_VALUES_IN_MEMORY|NUM_VALUES_IN_OFFHEAP|MEMORY
    ---------------------------------------------------------------------
    [ . . .]
    
    APP.FLIGHTAVAILABILITY|NULL  |NULL  
     |192.168.129.188(5231)<v1>:41602|192.168.129.188|1.688
      |33.152   |0.0  |4.144|45.584
       |84.568   |518 |0   
    |0   |518  |- 
    
    [ . . .]

### 显示开销，内存分配和总内存足迹 ###
您可以查询SYS.MEMORYANALYTICS虚拟表estiamte该表的内存使用量的数据存储。你也可以使用一个优化提示，以获得表和索引的内存占用更准确的视图查询表。<br/>

### 样品sys.memoryanalytics查询 ###
这部分包括一些示例查询，可提供有关您的RowStore系统的内存使用情况的有用信息。<br/>

#### 进入开销进行分配表 ####

要找到每个条目的开销总跨所有成员分区表：<br/>

    snappy> SELECT table_name, SUM(constant_overhead+entry_size) / CASE WHEN SUM(num_rows) > 0 THEN SUM(num_rows) ELSE 1 END FROM sys.memoryanalytics WHERE index_name IS null GROUP BY table_name;
要找到每个条目的开销总额为分区表和所有成员所定义的所有索引：<br/>


    snappy> SELECT table_name, SUM(constant_overhead+entry_size) / CASE WHEN sum(num_rows) > 0 THEN SUM(num_rows) ELSE 1 END FROM sys.memoryanalytics GROUP BY table_name;
#### 指数内存使用 ####

要查找所有成员的特定指数的总内存使用：<br/>

    snappy> SELECT index_name, SUM(total_size) FROM sys.memoryanalytics WHERE index_name LIKE 'IDX%' GROUP BY index_name;
#### 内存使用总量 ####

要看到多少内存由（其中在每个主机上的多个成员的情况下）每个主机的所有表消耗：<br/>

    snappy> SELECT host, sum(total_size) FROM sys.memoryanalytics GROUP BY host;
#### 平均主键大小 ####

看到平均初级密钥大小在内存中，当值溢出到磁盘：<br/>

    snappy> SELECT table_name, index_name, key_size/ CASE WHEN num_keys_in_memory > 0 THEN num_keys_in_memory ELSE 1 END FROM sys.memoryanalytics;
#### 表大小的范围 ####

要找到最大的（最大）和最小（最低）表大小在分布式系统：<br/>

	snappy> SELECT table_name, index_name, MAX(total_size) AS max_, min(total_size) AS min_ FROM sys.memoryanalytics GROUP BY table_name, index_name;  


### 使用sizerHints当查询sys.memoryanalytics ###
如果您使用的`sizerHints=withMemoryFootPrint`选项与您的查询，就可以获取你的表的总内存占用更准确的估计。

> 注意：因为它使用Java代理，需要大量的内存来处理使用`sizerHints = withMemoryFootPrint`谨慎。应当适用于非常小的数据集（通常为几百行）和不应该在生产系统中使用。当您使用以下方式查询sys.memoryanalytics`sizerHints = withMemoryFootPrint` RowStore走过堆和使用反射来提供estiamtes thart更接近什么的VisualVM或其他工具可能会显示。所显示的值可以是40〜50％不同于未经优化器提示显示的值，或从脉冲的监测工具中所示的值。

按照以下步骤开始RowStore成员与Java代理查询内存分析时使用sizerHints：<br/>

1.在开始时，定位系统jar_path属性：如果您使用定位器成员发现，在指定的-javaagent的完整路径和snappydata.jar的文件名。例如：<br/>

    snappy-shell rowstore locator start -dir=locator -peer-discovery-port=10101 -client-port=1527 -J-javaagent:/snappystore-dir/lib/snappydata.jar 
2.使用-javaagent：jar_path Java系统属性来指定snappydata.jar当你开始每个RowStore成员在您的安装文件。例如，如果您使用gfxd启动RowStore服务器：

    snappy-shell rowstore server start -client-port=1528 -J-javaagent:/snappystore-dir/lib/snappydata.jar
请确保您指定snappydata.jar的完整路径和文件名您的系统。<br/>

3.启动gfxd提示并连接到分布式系统。

4.使用sizerHints=withMemoryFootPrint提示与查询，显示内存占用：<br/>

    snappy> SELECT * FROM sys.memoryAnalytics -- GEMFIREXD-PROPERTIES sizerHints=withMemoryFootPrint \n ;
> 注意：当处理一个查询提示，如果它出现在一个SQL注释行的末尾RowStore不承认终止分号。出于这个原因，终止分号之前添加一个新行字符（'\ N`）如果优化提示延伸到语句的结束。


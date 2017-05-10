# 从内存中抽出数据表 #
您可以使用最近最少使用（LRU）逐出和到期配置表来管理RowStore分布式系统中内存中管理的数据量。您可以选择是否从RowStore系统中删除被驱逐的数据（减少总数据集的大小），或者将驱逐的数据保留在磁盘存储操作日志文件中，以便根据需要重新加载数据以满足将来的查询。<br/>

- **LRU Eviction的工作原理** RowStore通过删除最近最少使用的（LRU）数据为新数据释放内存来保持表格数据在指定级别的使用。您可以根据条目计数，可用堆的百分比或表数据的绝对内存使用情况来配置表逐出设置。您还可以配置RowStore应该采取的行为来驱逐数据：销毁数据（仅适用于分区表）或将数据溢出到磁盘存储，创建一个“溢出表”。

- **驱逐限制** LRU驱逐仅对基于主键值操作的操作有效。

- **分区表中的驱逐** 在分区表中，RowStore删除可以在执行新的输入操作的存储桶中找到的最旧的条目。RowStore在逐桶的基础上维护LRU条目信息，因为在整个分区表中维护信息的性能成本太高。

- **创建具有驱逐设置的表** 使用驱逐设置将表保持在指定的限制内，无论是完全删除驱逐的数据，还是创建将驱逐的数据保留到磁盘存储的溢出表。

## LRU驱逐如何运作 ##
RowStore通过删除最近最少使用的（LRU）数据来释放内存以获取新数据，从而将表的数据用于指定级别。您可以根据条目计数，可用堆的百分比或表数据的绝对内存使用情况来配置表逐出设置。您还可以配置RowStore应该采取的行为来驱逐数据：销毁数据（仅适用于分区表）或将数据溢出到磁盘存储，创建一个“溢出表”。<br/>

当RowStore确定添加或更新行将使表超过指定级别时，它会根据需要溢出或删除更多的旧行以腾出空间。对于进入计数驱逐，这意味着较旧行的一对一交易。<br/>

对于内存设置，需要删除以创建空间的较旧行的数量完全取决于较旧和较新条目的相对大小。驱逐控制器监控表和内存使用情况，当达到限制时，删除较旧的条目以使新数据成为可能。对于JVM堆内存百分比，所使用的控制器是RowStore资源管理器，与JVM的垃圾收集器配合使用，以获得最佳性能。<br/>

当启用驱逐时，RowStore管理类似于哈希映射的结构中的表行，其中主键值用作该行中所有其他列数据的键。当行被逐出时，主键值保留在内存中，而其余的列数据被驱逐。<br/>

要配置LRU驱逐，请参阅**使用驱逐设置创建一个表**和**CREATE TABLE**。配置驱逐功能后，您可以选择安装RowLoader和/或同步写入程序插件以访问外部数据源，从而有效地将RowStore用作缓存。有关此功能的限制，请参阅“ **驱逐限制** ”。<br/>

## 驱逐限制 ##
LRU驱逐仅对基于主键值操作的操作有效。<br/>

在您的系统中实施LRU驱逐之前，请考虑这些限制：<br/>

- 复制表不支持使用DESTROY操作进行删除。（与**EXPIRE条款**一样，复制表支持OVERFLOW操作的驱逐）。

- RowStore不支持使用DESTROY逐出操作配置的表上的事务。存在这种限制，因为ACID事务的要求可能与破坏被驱逐条目的语义相冲突。例如，事务可能需要更新大于驱逐设置允许的数量的条目数。OVERFLOW evict操作支持事务，因为必要条件可以根据需要加载到内存中以支持事务语义。

- 与外部数据源同步的功能仅对通过主键查询数据的选择/更新/删除操作有效。通过其他条件访问数据可能会导致不完整的结果，因为仅在主键查询上调用RowLoader。

- 如果使用DESTROY逐出操作配置分区表，则必须确保使用主键值对表过滤器进行的所有查询都会生成。如果行在逐出时被销毁，则不会对主键进行过滤的查询可能会产生部分结果。此限制不适用于使用OVERFLOW逐出动作配置的表。

- 您不能创建使用DESTROY操作配置为逐出或到期的分区表的外键引用。此限制不适用于使用OVERFLOW逐出动作配置的表。有关详细信息，请参阅“ **EVICTION BY条款** ”。

- 某些应用程序可能受益于将行排除到磁盘，以减少内存使用（堆或堆内存时）。但是，启用驱逐会增加RowStore 所需的堆内存的每行开销，以执行表的LRU驱逐。作为一般规则，如果表中的非主键列较大，则表驱逐仅对节省内存有帮助：100字节或更多。

- 在使用DESTROY操作的缓存中被逐出或已经过期的行上不会发生UPDATE。此限制不适用于使用OVERFLOW逐出动作配置的表。

- 如果先前使用DESTROY操作从缓存中驱逐或过期相同的行（基于主键），则INSERT将成功，但该行仍然存在于外部数据存储中。

## 分区表中的驱逐 ##
在分区表中，RowStore删除可以在执行新的输入操作的存储桶中找到的最旧的条目。RowStore在逐桶的基础上维护LRU条目信息，因为在整个分区表中维护信息的性能成本太高。<br/>

RowStore以这些方式选择用于LRU驱逐的桶<br/>



- 对于内存和进入计数驱逐，在添加新行的桶中完成LRU驱逐，直到成员中的组合桶的总大小已经下降到足以执行操作而不超过限制。


- 对于堆驱逐，每个分区表桶被视为单独的表，每个逐出动作只考虑桶的LRU，而不是整个分区表。

分区表中的驱逐可能会在本地数据存储区以及分布式系统中的其他主机中为表格留下较旧的条目。它也可以将主副本中的条目留下，从副副本中删除，反之亦然。但是，当插入或更新具有相同主键的条目时，RowStore会同步冗余副本。<br/>

## 创建具有驱逐设置的表 ##
使用驱逐设置将表保持在指定的限制内，无论是完全删除驱逐的数据，还是创建将驱逐的数据保留到磁盘存储的溢出表。<br/>

**程序**

按照以下步骤配置表逐出设置。有关指定驱逐设置的详细信息，请参阅**CREATE TABLE**。

1. 决定是否驱逐：<br/>

	- 输入计数（如果表格行大小相对均匀，则有用）。
	- 使用总字节数
	- 使用的JVM堆的百分比。这将使用RowStore资源管理器。当经理确定需要驱逐时，经理命令驱逐控制器开始逐出从设定驱逐标准的所有表**LRUHEAPPERCENT**。<br/>
您可以配置所有RowStore数据存储的全局堆驱逐百分比，或者通过使用系统过程或通过指定**-eviction-heap-percentage**何时启动服务器来配置一个或多个服务器组的阈值百分比**snappy-shell**。程序介绍如何使用系统过程配置堆驱逐百分比。有关使用gfxd启动服务器的信息，请参阅服务器。<br/>
直到资源管理器停止运行。RowStore驱逐由表的成员托管的最近最少使用的行。<br/>



1. 决定达到极限时要采取的行动：

	- 本地销毁行（仅限分区表）。


	> 注意： RowStore不会将DESTROY evict操作传播到已配置的回调实现（如DBSynchronizer）。不要在具有依赖表的表（例如，具有外键的子行）上使用DESTROY操作来配置逐出。如果为通过驱逐本地销毁的父表行调用了DELETE语句，则DELETE在RowStore中成功。但是，如果依赖行仍然存在，DBSynchronizer异步发送DELETE命令，则后端数据库中的DELETE操作可能会失败。

	如果依赖表需要执行DESTROY操作，则请考虑使用触发器或写入程序实现来侦听父表上的DELETE事件。如果后端数据库中存在子行，则触发器或写入程序将失败DELETE操作。
复制表不支持DESTROY逐出动作。

		-   Overflow the row data to disk. <p class="note"><strong>Note:</strong> When you configure an overflow table, only the evicted rows are written to disk. If you restart or shut down a member that hosts the overflow table, the table data that was in memory is not restored unless you explicitly configure persistence (or you configure one or more replicas with a partitioned table). See <a href="../disk_storage/chapter_overview.html#disk_storage" class="xref" title="By default, a RowStore distributed system persists only the data dictionary for the tables and indexes you create. These persistence files are stored in the datadictionary subdirectory of each locator and data store that joins the distributed system. In-memory table data, however, is not persisted by default; if you shut down all RowStore members, non-persistent tables are empty on the next startup. You have the option to persist all table data to disk as a backup of the in-memory copy, or to overflow table data to disk when it is evicted from memory.">Persisting Table Data to RowStore Disk Stores</a>. </p>


	1. 如果要将数据溢出到磁盘（或将整个表保存到磁盘），请配置用于溢出数据的命名磁盘存储。如果在创建溢出表时未指定磁盘存储，则RowStore将溢出数据存储在默认磁盘存储中。


	1. 创建具有所需的逐出配置的表。
	例如，要驱逐使用LRU条目计数和溢出驱逐的行到磁盘存储：

			CREATE TABLE Orders(OrderId INT NOT NULL,ItemId INT ) 
	     	EVICTION BY LRUCOUNT 2 EVICTACTION OVERFLOW 'OverflowDiskStore' ASYNCHRONOUS
如果不指定磁盘存储，RowStore将表数据溢出到默认磁盘存储。
要创建一个溢出表并持续整个表，以便整个表可以通过对等重新启动：

			snappy> CREATE TABLE Orders(OrderId INT NOT NULL,ItemId INT ) 
			> EVICTION BY LRUCOUNT 2 EVICTACTION OVERFLOW PERSISTENT 'OverflowDiskStore' ASYNCHRONOUS;
该表对于溢出和持久性都使用相同的命名磁盘存储。
要创建一个简单地从内存中删除被驱逐的数据而不持续被驱逐的数据的表，请使用`DESTROY`逐出动作。例如：

			snappy> CREATE TABLE Orders(OrderId INT NOT NULL,ItemId INT ) 
			> PARTITION BY COLUMN (OrderId) EVICTION BY LRUMEMSIZE 1000 EVICTACTION DESTROY;

		> 注意：如果为分区表配置LRUMEMSIZE，则Pivotal建议您将LRUMEMSIZE和MAXPARTSIZE都设置为相同的值。如果指定的LRUMEMSIZE值大于分区表的MAXPARTSIZE，则RowStore会自动将LRUMEMSIZE设置为等于MAXPARTSIZE。参见EVICTION BY条款。



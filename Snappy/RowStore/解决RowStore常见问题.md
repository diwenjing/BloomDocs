#解决RowStore常见问题
##使用RowStore可能会遇到常见问题。
###ClassNotFoundException升级后
###会员启动问题
###从ConflictingPersistentDataException恢复
###连接问题
###WAN复制问题
###防止磁盘完全错误
###从磁盘完全错误中恢复

###ClassNotFoundException升级后<br/>
升级RowStore成员后，您可能会ClassNotFoundException在启动成员时收到一个引用以com.vmware.sqlfire或开头的包名称com.pivotal.sqlfire。如果您无法停止并删除使用SQLFire产品中的软件包名称的旧版过程或侦听器实现，则会发生这些异常。在升级到RowStore之前停止并删除这些对象。有关详细信息，请参阅使用RPM分发执行手动升级或使用ZIP分发进行手动升级。<br/>



###会员启动问题
当您启动RowStore成员时，如果其他成员上的特定磁盘存储文件不可用，则可能会发生启动延迟。这是正常启动行为的一部分，旨在帮助确保数据的一致性。例如，考虑定位器（“locator2”）的以下启动消息：<br/>
<code>
RowStore Locator pid: 23537 status: waiting<br/>
Waiting for DataDictionary (DiskId: 531fc5bb-1720-4836-a468-3d738a21af63, Location: /pivotal/locator2/./datadictionary) on: <br/>
 [DiskId: aa77785a-0f03-4441-84f7-6eb6547d7833, Location: /pivotal/server1/./datadictionary]<br/>
 [DiskId: f417704b-fff4-4b99-81a2-75576d673547, Location: /pivotal/locator1/./<br/>datadictionary]<br/>
</code>
这里，启动消息指示locator2正在等待locator1和server1上的持久数据字典文件变为可用。即使您没有配置这些表来持久存储数据，RowStore始终会保留您创建的索引和表的数据字典。上面的启动消息表示locator1或locator2可能潜在地存储分布式系统的数据字典的较新副本。<br/>

通过启动server1数据存储继续启动，产生：<br/>
<code>
Starting RowStore Server using locators for peer discovery: localhost[10337],localhost[10338]<br/>
Starting network server for RowStore Server at address localhost/127.0.0.1[1529]<br/>
Logs generated in /pivotal/server1/gfxdserver.log<br/>
The server is still starting. 15 seconds have elapsed since the last log message: <br/>
 Region /_DDL_STMTS_META_REGION has potentially stale data. It is waiting for another member to recover the latest data.<br/>
My persistent id:<br/>

  DiskStore ID: aa77785a-0f03-4441-84f7-6eb6547d7833<br/>
  Name: <br/>
  Location: /10.0.1.31:/pivotal/server1/./datadictionary<br/>
<br/>
Members with potentially new data:<br/>
[<br/>
  DiskStore ID: f417704b-fff4-4b99-81a2-75576d673547<br/>
  Name: <br/>
  Location: /10.0.1.31:/pivotal/locator1/./datadictionary<br/>
]<br/>
Use the "snappy-shell list-missing-disk-stores" command to see all disk stores that are being waited on by other members.<br/>
</code>
数据存储启动消息指示locator1具有数据字典的“潜在新数据”，在这种情况下，locator2和server1都在系统中的locator1之前关闭，因此这些成员正在等待locator1确保它们具有最新的版本的数据字典。<br/>

上述数据存储和定位器的消息可能表示某些成员未启动。如果指示的磁盘存储持久性文件在缺少的成员上可用，只需启动该成员并允许正在运行的成员恢复。例如，在上述系统中，您只需启动locator1，并允许locator2和server1同步其数据。<br/>

为避免这种类型的延迟启动和恢复：<br/>

在系统中已经同步磁盘存储之后，尽可能关闭数据存储成员（全部关闭）。在数据存储停止后关闭剩余的定位器成员。<br/>
确保所有持久成员重新正确重新启动。有关详细信息，请参阅从ConflictingPersistentDataException中恢复。
如果成员无法重新启动，并且阻止其他数据存储启动，请使用revoke-missing-disk-store撤销正在阻止启动的磁盘存储。如果撤销的磁盘存储实际上包含数据字典或表数据的最近更改，则可能会导致数据丢失。撤销的磁盘存储不能稍后添加回系统。如果您撤销成员上的磁盘存储，则需要从该成员中删除相关联的磁盘文件，以便重新启动它。只能使用该revoke-missing-disk-store命令作为最后的手段。<br/>



###从ConflictingPersistentDataException恢复
如果您ConflictingPersistentDataException在启动期间收到，表示您有多个持久性数据的副本，并且RowStore无法确定要使用的副本。通常RowStore使用元数据自动确定要使用的持久性数据的副本。每个成员持续存在数据字典或表格数据，其中包含数据的其他成员及其数据是否为最新的列表。<br/>

一个ConflictingPersistentDataException发生在两个成员比较它们的元数据，并发现这是不一致的，他们要么不知道对方的存在，或者他们都认为，其他成员有过期数据。以下是一些可能导致a的情况ConflictingPersistentDataException。<br/>

独立创建的副本<br/>

尝试将两个独立创建的分布式系统合并到单个分布式系统中会导致ConflictingPersistentDataException。有几种方法可以使用独立创建的系统：<br/>

配置问题可能会导致RowStore成员连接到不知道彼此的不同定位器。为避免此问题，请确保所有定位器和数据存储始终在启动时指定相同的完整的定位器地址列表（例如locators=locator1[10334],locator2[10334],locator3[10334]）。考虑使用单个gemfirexd.properties文件进行常规配置项目，如“规划和配置RowStore部署”的步骤中所述。<br/>
系统中的所有持久成员可能会被关闭，之后一组全新的不同持久成员试图启动。<br/>
尝试通过将所有成员指向同一组定位器来合并独立系统，然后生成一个ConflictingPersistentDataException。
<br/>
RowStore无法合并同一表的独立创建的数据。相反，您需要从其中一个系统导出数据并将其导入到其他系统中。请参阅使用RowStore导出和导入数据。<br/>

先开始新成员<br/>

在启动具有持久性数据的旧成员之前，启动一个没有持久数据的全新成员可能导致一个ConflictingPersistentDataException。<br/>

如果关闭系统，然后在启动脚本中添加一个新成员，最后可以并行启动所有成员，这可能会意外发生。在这种情况下，新成员可以先启动。如果发生这种情况，新成员将在旧成员启动之前创建一个空的独立副本。老年人开始时，情况与“独立制作的副本”中所述情况相似。<br/>

在这种情况下，修复只是简单地移动或删除新成员的（空）持久性文件，关闭新成员，最后重新启动旧成员。老成员完全恢复后，重新启动新成员。<br/>

网络拆分，启用网络分区检测设置为false<br/>

当enable-network-partition-detection设置为true时，RowStore会检测网络分区并关闭成员以防止“分裂大脑”。在这种情况下，系统恢复时不会发生冲突。<br/>

但是，如果enable-network-partition-detection是false，则RowStore不能在网络分区之后阻止“分裂大脑”。相反，网络分区的每一边记录分区的另一侧具有陈旧的数据。当分区被治愈并且持续成员重新启动时，他们会发现冲突，因为每一方都认为对方的成员是陈旧的。<br/>

在某些情况下，可能可以在网络分区的两侧进行选择，并只保留分区一侧的数据。否则，您可能需要挽救数据并将其导入新的系统。<br/>

解决ConflictingPersistentDataException<br/>

如果您收到ConflictingPersistentDataException，您将无法启动所有成员，并使他们加入同一个分布式系统。
<br/>
首先，确定系统是否有一部分可以恢复。例如，如果您刚刚向系统添加了一些新成员，请尝试启动而不包括这些成员。对于剩余的成员，使用数据提取器工具从持久性文件中提取数据并将其导入到正在运行的系统中。请参阅从磁盘存储器恢复数据。<br/>


###连接问题<br/>
这些是连接到RowStore分布式系统时发生的常见问题：<br/>

您收到SQL State 08001错误：“尝试所有可用服务器后失败：[]'<br/>
如果在JDBC连接URL中为用户名和密码连接属性指定了空值，则可能会导致此问题。一些第三方工具指定自动提供空值，但如果不指定用户凭据，则包括连接属性。<br/>
如果在分布式系统中禁用了身份验证，则可以在连接时指定任何临时用户名和密码值。使用JDBC工具连接到RowStore提供了更多的细节。<br/>

###WAN复制问题<br/>
在WAN部署中，如果表彼此不相同，表可能会在两个RowStore分布式系统之间失败同步（请参阅使用Gateway发件人创建表）。如果已经在站点之间配置了WAN复制，但是由于模式差异而导致表无法同步，请按照以下步骤更正以下情况：<br/>

停止每个RowStore分布式系统中的网关发送者和网关接收者。请参阅启动和停止网关发件人。<br/>
使用ALTER TABLE在问题表上添加或删除列，以确保两个表具有相同的列定义。比较每个表的describe命令的输出，以确保表格相同。或者，在每个分布式系统中使用write-schema-to-sql来比较用于创建每个表的DDL语句。
使用SYS.GET_TABLE_VERSION来验证这两个表在每个RowStore集群的数据字典中是否具有相同的版本。如果版本不匹配，请对具有较小版本的表使用SYS.INCREMENT_TABLE_VERSION以使两个表版本相等。<br/>
为分布式系统重新启动网关发送者和网关接收者。请参阅启动和停止网关发件人。<br/>


###防止磁盘完全错误
监视RowStore成员的磁盘使用情况很重要。如果成员缺少磁盘存储的足够的磁盘空间，则成员将尝试关闭磁盘存储及其关联的表，并记录错误消息。为该成员创建足够的磁盘空间后，您可以重新启动该成员。（请参阅成员启动问题）由于会员磁盘空间不足导致的关机可能会导致数据丢失，数据文件损坏，日志文件损坏以及可能对应用程序产生负面影响的其他错误条件。<br/>

您可以使用以下技术来防止磁盘文件错误：<br/>

使用磁盘存储文件和磁盘存储元数据文件的默认预分配。预配置为这些文件预留磁盘空间，并在磁盘存储关闭时保持成员处于健康状态，从而在足够的磁盘空间可用时重新启动该成员。预配置默认配置。<br/>
预分配由以下系统属性控制：<br/>
磁盘存储文件 - 将gemfire.preAllocateDisk系统属性设置为true（默认值）。<br/>
磁盘存储元数据文件 - 将gemfire.preAllocateIF系统属性设置为true（默认值）。<br/>
注意： Pivotal建议在Linux平台上使用ext4文件系统，因为ext4支持预分配，可以提高磁盘启动性能。如果在具有高写入吞吐量的延迟敏感环境中使用ext3文件系统，则可以通过将磁盘存储的MAXLOGSIZE属性设置为低于默认1 GB的值来提高磁盘启动性能。请参阅CREATE DISKSTORE。<br/>

监视RowStore日志以获取较低的磁盘空间警告。在以下情况下，RowStore会记录磁盘空间警告：<br/>
日志文件目录 -如果可用空间小于100 MB，则会发出警告。<br/>
磁盘存储目录 -如果可用空间小于创建新的oplog文件所需空间的1.15倍，则会发出警告。<br/>
数据字典 -如果剩余空间小于50 MB，则会发出警告。<br/>
您可以使用gemfire.DISKSPACE_WARNING_INTERVAL系统属性配置日志消息频率。<br/>

###从磁盘完全错误中恢复
如果您的RowStore分布式系统的成员由于磁盘完全错误的情况而失败，请添加或增加额外的磁盘容量，并尝试正常重新启动该成员。如果成员不重新启动，并且其另一个成员的磁盘存储中的表的冗余副本可以使用以下步骤恢复成员：<br/>

从失败的成员删除或移动磁盘存储文件。<br/>
使用list-missing-disk-stores snappy-shell命令来识别任何丢失的数据。您可能需要手动还原此数据。
撤销使用revoke-disk-store命令的成员。<br/>
重新启动会员<br/>
请参阅处理缺少的磁盘存储。<br/>



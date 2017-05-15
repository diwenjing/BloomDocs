# 分区表 #
水平分区涉及在群集中的成员之间传播大型数据集（表中的许多行）。RowStore使用一致的散列算法的变体来帮助确保数据在目标服务器组的所有成员之间均匀平衡。<br/>



- **表分区的工作原理** 您可以在CREATE TABLE语句的PARTITION BY子句中指定表的分区策略。可用的策略包括对每个行的主键值进行散列分区，对主键，范围分区和列表分区之外的列值进行散列分区。<br/>


- **了解存储数据的** 位置RowStore使用表的分区列值和分区策略来计算路由值（通常为整数值）。它使用路由值来确定存储数据的“桶”。<br/>


- **故障和冗余** 如果您有分区表的冗余副本，您可能丢失服务器而不会丢失数据或中断服务。当服务器发生故障时，RowStore自动将尝试向失败的成员写入的任何操作重新路由到尚存的成员。<br/>


- **创建分区表** 您可以在由命名服务器组标识的一组服务器上创建分区表（如果未指定命名服务器组，则在默认服务器组上）。在条款`CREATE TABLE`声明决定了表中的数据进行分区，同位，和整个服务器组复制。<br/>


- **重新平衡RowStore成员上的分区数据** 您可以使用重新平衡来动态增加或减少RowStore集群容量，或者改进分布式系统中数据的平衡。<br/>

- **管理复制故障** RowStore使用多个故障检测算法来快速检测复制问题。RowStore复制设计侧重于一致性，并且不允许可疑成员或网络分区成员隔离运行。<br/>

## 表分区如何工作 ##
您可以在CREATE TABLE语句的PARTITION BY子句中指定表的分区策略。可用的策略包括对每个行的主键值进行散列分区，对主键，范围分区和列表分区之外的列值进行散列分区。<br/>

RowStore将分区表的每一行映射到逻辑“bucket”。将行映射到桶基于您指定的分区策略。例如，通过主键上的哈希分区，RowStore通过对表的主键进行散列来确定逻辑块。每个桶分配给一个或多个成员，具体取决于您为表配置的份数。配置具有一个或多个数据冗余副本的分区表可确保分区数据仍然可用，即使成员丢失。<br/>

当成员丢失或删除时，桶将根据加载重新分配给新成员。在群集中丢失成员不会导致重新分配行到桶。您可以指定要用于CREATE TABLE语句的BUCKETS子句的桶的总数。默认桶数为113。<br/>

在RowStore中，分布式系统中的所有对等服务器都知道哪些对等体主机哪个桶，所以他们可以有效地访问一个最多一个网络跳到承载数据的成员的行。对分区表的读取或写入将透明地路由到承载作为操作目标的行的服务器。每个对等体为群集中的每个对等体提供持久的通信信道。<br/>

图：分区表数据<br/>

![](http://rowstore.docs.snappydata.io/docs/common/images/PartionedBasics.png)

虽然每个桶分配给一个或多个特定的服务器，您可以使用一个过程来重定位正在运行的系统中的桶，以便提高集群中资源的利用率。请参阅对RowStore成员重新平衡分区数据。<br/>

您还可以在将数据加载到表中之前预先分配桶，以确保导入的数据在表分区之间均匀分布。请参阅预分配桶。<br/>

## 了解数据存储位置 ##
RowStore使用表的分区列值和分区策略来计算路由值（通常为整数值）。它使用路由值来确定存储数据的“桶”。<br/>

然后，如果分区表配置为具有冗余，则将每个桶分配给服务器，或分配给多个服务器。当表启动时，不分配存储桶，而是在数据实际放入存储桶时发生懒惰。这允许您在填充表之前启动一些成员。<br/>

> 注意：如果您打算立即使用数据加载新的分区表，请在加载数据之前使用SYS.CREATE \ _ALL \ _BUCKETS过程创建存储桶。这种最佳做法确保分区是平衡的，即使您使用并发线程快速将数据加载到表中。<br/>

图：桶中的分区数据<br/>

![](http://rowstore.docs.snappydata.io/docs/common/images/PartionBuckets.png)

如果将表的冗余副本设置为大于零，RowStore将其中一个副本指定为主副本。对桶的所有写入都将通过主副本。这样可以确保桶的所有副本一致。<br/>

组成员服务（GMS）和分布式锁定服务确保所有分布式成员在分布式系统的任何时间都具有一致的主要和次要视图，而不管管理员发起的成员变更或失败。<br/>

## 故障和冗余 ##
如果您有分区表的冗余副本，您可能丢失服务器而不会丢失数据或中断服务。当服务器发生故障时，RowStore自动将尝试向失败的成员写入的任何操作重新路由到尚存的成员。<br/>

如果可能，RowStore还会尝试将失败的读取操作重新路由到另一个服务器。如果读取操作只返回一行，则透明的故障切换总是可能的。但是，如果一个操作返回多个行并且应用程序已经消耗了一个或多个行，那么如果查询中涉及的服务器在所有结果都被消耗之前发生脱机，则RowStore不能故障转移; 在这种情况下，应用程序会收到带有SQLState X0Z01的SQLException。所有应用程序都应该考虑到接收到这种异常的可能性，如果出现这种情况，应该手动重试查询。<br/>

如果执行查询时服务器不可用，则还会重试读取操作。在此图中，M1正在读取表格值W和Y.它从本地副本直接读取W，并试图从M3读取当前离线的Y。在这种情况下，读取将自动重试在保存表数据的冗余副本的另一个可用成员中。<br/>

![](http://rowstore.docs.snappydata.io/docs/common/images/PartionHA.png)

## 创建分区表 ##
您可以在由命名服务器组标识的一组服务器上创建分区表（如果未指定命名服务器组，则在默认服务器组上）。在条款`CREATE TABLE`声明决定了表中的数据进行分区，同位，和整个服务器组复制。<br/>

本主题专注于`partitioning\_clauseCREATE` TABLE中。在CREATE TABLE参考页详细描述了所有的选项。<br/>

的`partitioning\_clause`控制在服务器组中的位置和数据的分发。使用服务器组和托管对于优化查询很重要，并且对于跨表连接至关重要。此版本的RowStore不支持非共处理数据的跨表连接，因此您必须仔细选择分区子句以启用应用程序所需的连接。<br/>

分区子句可以指定列分区，范围分区，列表分区或表达式分区：<br/>

    {
      {
       PARTITION BY { PRIMARY KEY | COLUMN ( column-name [ , column-name ]* ) }
       |
       PARTITION BY RANGE ( column-name )
       (
	     VALUES BETWEEN value AND value
	     [ , VALUES BETWEEN value AND value ]*
       )
       |
       PARTITION BY LIST ( column-name )
       (
	     VALUES ( value [ , value ]* )
	     [ , VALUES ( value [ , value ]* ) ]*
       )
       |
       PARTITION BY ( expression )
      }
     [ COLOCATE WITH ( table-name [ , table-name ] * ) ]
    }
    [ REDUNDANCY integer-constant ]
    [ BUCKETS integer-constant ]
    [ MAXPARTSIZE integer-constant ]
    [ RECOVERYDELAY integer-constant ]


> 注意：如果表没有主键，则RowStore会生成用于分区数据的唯一行ID。



> 注意：您不能使用表达式在PARTITION BY RANGE或PARTITION BY LIST子句中提供值。

RowStore支持下面描述的分区策略。<br/>

|分区策略|描述|
|:--|:--|
|列分区|该`PARTITION BY COLUMN`子句定义了一组用作分区基础的列名。作为快捷方式，可以使用PARTITION BY PRIMARY KEY引用表的主键列。RowStore使用内部散列函数，通常使用指定列的底层Java类型的hashCode（）方法。对于多列，内部散列函数使用指定列的序列化字节来计算散列。|
|范围划分|该`PARTITION BY RANG`E条款规定，应协同定位字段的范围。这确保了范围查询和跨表连接的数据的位置。范围的下限是包含值，上限是排他的。范围不需要覆盖该领域的整个可能值的范围。不在范围内的值在服务器组中自动分区，但不保证这些值的位置。您不能使用表达式在此子句中提供值。|
|列表分区|该`PARTITION BY LIST`子句指定应该共同定位以优化查询并支持跨表连接的字段的值集合。没有必要列出字段的所有可能的值。任何不属于列表的值都将自动在服务器组中分区，但不能保证这些值的位置。您不能使用表达式在此子句中提供值。|
|表达式分区|`PARTITION BY ( expression )`包含表达式的子句是一种使用表达式指定哈希值的哈希分区。表达式只能从表中引用字段名称。这允许根据它们的值的函数来共同行。|



- **分区示例** 您可以通过例如列（例如客户名称），表达式，优先级范围和状态来分区表。


- **从多个表中协调相关行当** 两个表在列上进行分区并进行共同定位时，它会强制具有与两个表中的列相同值的分区位于同一个RowStore成员上。例如，对于范围或列表分区，满足范围或列表的任何行都可以在所有共同表的同一RowStore成员上共同配置。如果您经常对该列上加入的表执行查询，那么根据分区列的值来协调两个表的数据是最佳做法。


- **使分区表高度可用** 使用该`REDUNDANCY`子句来指定要维护的每个分区的表的冗余副本数。


- **为分区表配置高可用性** 默认情况下，RowStore在表的数据存储中仅存储分区表数据的一个副本。您可以配置RowStore来维护分区表数据的冗余副本，以实现高可用性。


- **限制成员的内存消耗** 使用CREATE TABLE语句的MAXPARTSIZE子句可以在可用成员之间对分区数据进行负载平衡。


- **预分配桶** 作为最佳选择，在开始加载数据之前，请使用SYS.CREATE_ALL_BUCKETS（' table-name '）过程创建分区表的桶。

### 分区示例 ###
您可以通过列（例如客户名称），表达式，优先级范围和状态来分区表。<br/>


#### 基于列的分区 ####
此语句创建一个由“CustomerName”列分隔的表。具有相同CustomerName的所有行都保证在相同的进程空间中共同配置。这里，该`SERVER GROUPS`子句确定承载分区表数据的对等端和服务器。服务器组是在分布式系统中托管数据的所有对等端和服务器的子集。<br/>

    CREATE TABLE Orders 
    ( 
    OrderId INT NOT NULL, 
    ItemId INT, 
    NumItems INT, 
    CustomerName VARCHAR(100), 
    OrderDate DATE, 
    Priority INT, 
    Status CHAR(10), 
    CONSTRAINT Pk_Orders PRIMARY KEY (OrderId) 
    ) 
    PARTITION BY COLUMN ( CustomerName ) 
    SERVER GROUPS ( OrdersDBServers);

### 基于范围的分区 ###
当您使用PARTITION BY RANGE子句时，指定一个具有多个值范围的列用于分区。<br/>



> 注意：您不能使用表达式在PARTITION BY RANGE或PARTITION BY LIST子句中提供值。

以下示例根据“优先级”列的三个值范围指定分区：<br/>

    CREATE TABLE Orders 
    ( 
    OrderId INT NOT NULL, 
    ItemId INT, 
    NumItems INT, 
    CustomerName VARCHAR(100), 
    OrderDate DATE, 
    Priority INT, 
    Status CHAR(10), 
    CONSTRAINT Pk_Orders PRIMARY KEY (OrderId) 
    ) 
    PARTITION BY RANGE ( Priority ) 
    ( 
    VALUES BETWEEN 1 AND 11, 
    VALUES BETWEEN 11 AND 31, 
    VALUES BETWEEN 31 AND 50 
    );

### 基于列表的分区 ###
当使用PARTITION BY LIST子句时，指定列名和一个或多个用于分区的列值列表。<br/>



> 注意：您不能使用表达式在PARTITION BY RANGE或PARTITION BY LIST子句中提供值。

以下示例根据“状态”列的三个不同的值列表对表进行分区：<br/>

    CREATE TABLE Orders 
    ( 
    OrderId INT NOT NULL, 
    ItemId INT, 
    NumItems INT, 
    CustomerName VARCHAR(100), 
    OrderDate DATE, 
    Priority INT, 
    Status CHAR(10), 
    CONSTRAINT Pk_Orders PRIMARY KEY (OrderId) 
    ) 
    PARTITION BY LIST ( Status ) 
    ( 
    VALUES ( 'pending', 'returned' ), 
    VALUES ( 'shipped', 'received' ), 
    VALUES ( 'hold' ) 
    );

#### 基于表达式的分区 ####
表达式分区通过评估您提供的SQL表达式来分区表。例如，以下语句根据OrderDate列的月份对表进行分区，使用MONTH函数作为SQL表达式：<br/>

    CREATE TABLE Orders 
    ( 
    OrderId INT NOT NULL, 
    ItemId INT, 
    NumItems INT, 
    CustomerName VARCHAR(100), 
    OrderDate DATE, 
    Priority INT, 
    Status CHAR(10), 
    CONSTRAINT Pk_Orders PRIMARY KEY (OrderId) 
    ) 
    PARTITION BY ( MONTH( OrderDate ) );

### 从多个表中并置相关行 ###
当两个表在列上进行分区并且共同定位时，它会强制具有相同值的分区，以使两个表中的这些列位于同一个RowStore成员上。例如，对于范围或列表分区，满足范围或列表的任何行都可以在所有共同表的同一RowStore成员上共同配置。如果您经常对该列上加入的表执行查询，那么根据分区列的值来协调两个表的数据是最佳做法。<br/>

该`COLOCATE WITH`子句指定分区表必须与之共同配置的表。在`COLOCATE WITH`创建分区表时，子句中引用的表必须存在。<br/>

指定该`COLOCATE WITH`子句时，必须使用一个`PARTITION BY`子句来指定目标表（`COLOCATE WITH`子句中引用的表）中使用的相同分区值和分区策略。例如，如果在目标表中使用PARTITION BY PRIMARY KEY或PARTITION BY COLUMN，那么您应该使用相同的子句之一对同一列值分配共同表（如果使用多个列进行分区，请指定列相同的顺序）。<br/>



> 注意：需要在多个表中共同定位的所有行必须携带分区键。虽然每个表中的列名称可能不同，但值必须相同。

例如，如果您创建分区表“国家”，如下所示：<br/>

    CREATE TABLE COUNTRIES 
    ( 
      COUNTRY VARCHAR(26) NOT NULL CONSTRAINT COUNTRIES_UNQ_NM Unique, 
      COUNTRY_ISO_CODE CHAR(2) NOT NULL CONSTRAINT COUNTRIES_PK PRIMARY KEY, 
      REGION VARCHAR(26), 
      CONSTRAINT COUNTRIES_UC
    CHECK (country_ISO_code = upper(country_ISO_code) ) 
    ) PARTITION BY PRIMARY KEY;
您可以使用以下命令将另一个表“城市”同时分配在相同的分区键上：<br/>

    CREATE TABLE CITIES 
    ( 
      CITY_ID INTEGER NOT NULL CONSTRAINT CITIES_PK Primary key,
      CITY_NAME VARCHAR(24) NOT NULL, 
      COUNTRY VARCHAR(26) NOT NULL, 
      AIRPORT VARCHAR(3), 
      LANGUAGE VARCHAR(16), 
      C_COUNTRY_ISO_CODE CHAR(2) CONSTRAINT COUNTRIES_FK 
      REFERENCES COUNTRIES (COUNTRY_ISO_CODE) 
    ) PARTITION BY COLUMN (C_COUNTRY_ISO_CODE) 
      COLOCATE WITH (COUNTRIES);
在此示例中，使用国家/地区ISO代码键对“国家”和“城市”进行分区。具有相同ISO代码值的行共同位于同一RowStore成员上。请注意，虽然“cities”中的分区列名与“countries”中的分区列名称不同，但两列都引用相同的分区键。RowStore确保共享分区键具有相同的类型（不考虑约束），但必须确保列存储相同的键。<br/>

类似地，如果使用`PARTITION BY RANGE`或`PARTITION BY LIST`在目标表中定义值，则必须在共同定位表定义中使用相同的分区子句来指定相同的分区键。当`PARTITION BY EXPRESSION`与同位表中使用，您必须确保表达式解析为相同的值在两个表中，这样的数据可以协同定位。<br/>

在所有情况下，对于colocated表，`REDUNDANCYand`和`BUCKETS`子句必须相同。共位表必须指定相同的服务器组（相同的`SERVER GROUPS`子句），或者一个表的服务器组必须是其他表的服务器组的子集。<br/>

有关详细信息，请参阅CREATE TABLE。<br/>

### 使分区表高度可用 ###
使用该`REDUNDANCY`子句为要维护的每个分区指定一个表的冗余副本。<br/>

因为RowStore主要是基于内存的数据管理系统，所以必要时使用冗余来启用故障切换，如果某个成员关闭或失败。但是，请记住，维护大量冗余副本会对性能，网络使用和内存使用造成不利影响。建议`REDUNDANCY`值为1以维护表数据的副本。例如：<br/>

    CREATE TABLE COUNTRIES
    (
      COUNTRY VARCHAR(26) NOT NULL,
      COUNTRY_ISO_CODE CHAR(2) NOT PRIMARY KEY,
      REGION VARCHAR(26),
    )
    REDUNDANCY 1
如果可能，RowStore会尝试将相同存储桶的副本放置到具有不同IP地址的主机上，以防止机器发生故障。但是，如果只有一台机器可用RowStore在该机器上放置多个副本。设置`enforce-unique-hos`t引导属性可以防止RowStore在同一台机器上放置多个副本。<br/>

设置`redundancy-zone`引导属性以确保RowStore在您定义的特定区域上放置冗余副本。例如，为了确保将冗余副本放置在不同的机架上，请将每台机器的冗余区域设置为机器运行的机架的逻辑名称。<br/>

请参阅**配置属性**。<br/>

如果分区的主要和次要存储桶不可用，则对表的查询可能会失败。对于持久性表，涉及丢失桶的查询失败`PartitionOfflineException`。整个表格上的查询也将失败。对具有缺少分区的非持久性表的查询可以返回空结果。<br/>

### 配置分区表的高可用性 ###
默认情况下，RowStore在表的数据存储中仅存储分区表数据的一个副本。您可以配置RowStore来维护分区表数据的冗余副本，以实现高可用性。



- **了解分区表的** 高可用性使用高可用性，托管分区表数据的每个成员都可以获得一些主副本和一些冗余（副本）副本。


- **为分区表配置高可用性为分区表** 配置内存高可用性。设置其他高可用性选项，如冗余区域和冗余恢复策略。

#### 了解分区表的高可用性 ####
具有高可用性，托管分区表的数据的每个成员将获得一些主副本和一些冗余（副本）副本。<br/>


冗余时，如果一个成员发生故障，分区表上的操作将继续进行，不会中断服务：<br/>



- 如果托管主副本的成员丢失，则RowStore将辅助副本作为主副本。这可能会导致冗余的暂时丢失，但不会导致数据丢失。



- 每当没有足够的辅助副本来满足冗余时，系统将通过将另一个成员分配给辅助来复制数据来恢复冗余。


> 注意：如果足够的成员在足够短的时间内下降，则使用冗余时，仍然可能会丢失缓存数据。

可以配置系统在不满足时恢复冗余的工作原理。您可以将恢复配置为立即进行配置，或者如果要让替换成员有机会启动，则可以配置等待时间。在任何分区数据重新平衡操作期间，也会自动尝试冗余恢复。没有冗余，丢失任何表的数据存储会导致某些表的缓存数据丢失。<br/>

通常，当应用程序可以从另一个数据源直接读取时，或者写入性能比读取性能更重要时，您不应该使用冗余。<br/>


##### 控制你的初级和次级居所 #####
默认情况下，RowStore为您提供主数据副本和辅助数据副本，避免在同一物理机器上放置两个副本。如果没有足够的机器将不同的副本分开，RowStore将副本放在同一物理机上。您可以更改此行为，因此RowStore仅将副本放在不同的计算机上。<br/>

您还可以通过使用**冗余区域**来控制哪些成员存储主数据副本和辅助数据副本。冗余区域在成员级配置。冗余区域可以让成员组或区域分隔主副本。您将每个数据主机分配给一个区域。然后RowStore将冗余副本放置在不同的冗余区域中，与在不同物理机器上放置冗余副本相同。您可以使用它将数据副本分割到不同的机架或网络上。此选项允许您即时添加成员，并使用重新平衡重新分配数据加载，冗余数据保留在单独的区域中。使用冗余区域时，RowStore不会将两个数据副本放在同一个区域中，因此请确保有足够的区域。<br/>


##### 在VMware虚拟机中运行进程 #####
默认情况下，RowStore将冗余副本存储在不同的计算机上。在VMware虚拟机中运行流程时，机器的普通视图将成为VMware VM，而不是物理机。如果在同一物理机上运行多个VMWare虚拟机，则最终可能会将分区表主存储区存储在单独的VMware VM中，而在同一台物理机器上。如果物理机失败，您可能会丢失数据。在VMware VM中运行时，可以配置RowStore以识别物理机器，并将冗余副本存储在不同的物理机器上。<br/>


##### 阅读和写在高可用分区表 #####
RowStore在高度可用的分区表中的读写方式与其他表中的读取和写入不同，因为数据在多个成员中可用：<br/>



- 写操作（如`PUT INTO`）转到主数据键，然后与冗余副本同步分配。
阅读操作将转到持有数据副本的任何成员，本地成员喜欢，所以阅读密集型系统可以更好地扩展并处理更高的负载。


- 在该图中，M1正在读取W，Y和Z.它直接从本地副本获取W。由于它没有Y或Z的本地副本，所以进入缓存，随机挑选源缓存。

![](http://rowstore.docs.snappydata.io/docs/common/images/partitioned_data_HA.png)

#### 配置分区表的高可用性 ####
为分区表配置内存高可用性。设置其他高可用性选项，如冗余区域和冗余恢复策略。<br/>

以下是为分区表配置高可用性的主要步骤。有关详细信息，请参阅后面的部分。<br/>




1. 设置系统应保留表数据的冗余副本数。请参阅**设置冗余副本的数量**。<br/>


1. （可选）如果要将数据存储成员分组到冗余区域，请相应配置它们。请参阅**为成员配置冗余区域**。<br/>


1. （可选）如果您希望RowStore只将冗余副本放在不同的物理机上，请配置。请参阅**设置强制唯一主机**。<br/>


1. 决定如何管理冗余恢复，并根据需要更改RowStore的默认行为。<br/>
a.之后的成员崩溃。如果要自动冗余恢复，请更改其配置。请参阅**为分区表配置成员崩溃冗余恢复**。<br/>
b.会员加入后。如果你不希望快速的自动冗余恢复，更改了配置。请参阅**为分区表配置成员加入冗余恢复**。<br/>


1. 回顾你开始重新平衡的几点。冗余恢复在任何重新平衡开始时自动完成。如果在成员崩溃或连接后没有自动恢复运行，这是最重要的。请参阅**对RowStore成员重新平衡分区数据**。<br/>

在运行时，您可以通过为表添加新成员来添加容量。您还可以启动重新平衡操作，以便在所有成员之间传播表格桶。<br/>

<br/>



- **设置冗余副本** 的数量通过指定要维护的辅助副本数量为分区表配置内存高可用性CREATE TABLE的REDUNDANCY子句。


- **将成员** 组冗余区域配置为冗余区域，以便RowStore将冗余数据副本放在不同的区域中。


- **设置强制唯一主机** 配置RowStore仅对唯一的物理机器使用分区表数据的冗余副本。


- **配置分区表的成员崩溃冗余恢复配置成员崩溃后** 如何在分区表中恢复冗余。


- **配置分区表的成员加入冗余恢复** 配置成员加入分布式系统后，如何为分区表恢复冗余。

##### 设置冗余份数 #####
通过指定要维护的辅助副本数量来为分区表配置内存中的高可用性CREATE TABLE的REDUNDANCY子句。<br/>

因为RowStore主要是基于内存的数据管理系统，所以必要时使用冗余来启用故障切换，如果某个成员关闭或失败。但是，请记住，维护大量冗余副本会对性能，网络使用和内存使用造成不利影响。建议`REDUNDANCY`值为1以维护表数据的副本。例如：<br/>

    CREATE TABLE COUNTRIES
    (
      COUNTRY VARCHAR(26) NOT NULL,
      COUNTRY_ISO_CODE CHAR(2) NOT PRIMARY KEY,
      REGION VARCHAR(26),
    )
    REDUNDANCY 1
如果可能，RowStore会尝试将相同存储桶的副本放置到具有不同IP地址的主机上，以防止机器发生故障。但是，如果只有一台机器可用RowStore在该机器上放置多个副本。设置`enforce-unique-host`引导属性可以防止RowStore在同一台机器上放置多个副本。<br/>

设置`redundancy-zone`引导属性以确保RowStore在您定义的特定区域上放置冗余副本。例如，为了确保将冗余副本放置在不同的机架上，请将每台机器的冗余区域设置为机器运行的机架的逻辑名称。<br/>

如果分区的主要和次要存储桶不可用，则对表的查询可能会失败。对于持久性表，涉及丢失桶的查询失败`PartitionOfflineException`。整个表格上的查询也将失败。对具有缺少分区的非持久性表的查询可以返回空结果。<br/>

##### 为会员配置冗余区域 #####
将成员组成冗余区域，以便RowStore将冗余数据副本放在不同的区域中。

了解如何设置会员的`gemfirexd.properties`设置。请参阅**配置属性**。<br/>

通过使用`gemfirexd.properties`设置的**冗余区域**将分区表的主机数据库分组到冗余区域。<br/>

例如，如果冗余设置为1，那么每个数据条目都有一个主副本，您可以通过为每个机架定义一个冗余区，从而在两个机架之间拆分主副本。为此，您可以将该区域设置`gemfirexd.properties`为在一个机架上运行的所有成员的文件中： `pre redundancy-zone=rack1`

您将为gemfirexd.properties另一个机架上的所有成员设置此区域： `pre redundancy-zone=rack2`

每个辅助副本将托管在托管主副本的机架上的机架上。<br/>

##### 设置强制唯一主机 #####
将RowStore配置为仅使用唯一的物理机器进行分区表数据的冗余副本。<br/>

了解如何设置会员的`gemfirexd.properties`设置。请参阅**配置属性**。<br/>

配置您的成员，因此RowStore始终使用不同的物理机器进行分区表数据的冗余副本，使用`gemfire.properties`设置enforce-unique-host。此设置的默认值为false。<br/>

例： `pre enforce-unique-host=true`<br/>

##### 为分区表配置成员崩溃冗余恢复 #####
配置成员崩溃后如何在分区表中恢复冗余。<br/>

您可以使用default-recovery-delay boot属性为所有表指定崩溃冗余恢复期，或者使用带有CREATE TABLE的RECOVERYDELAY子句来指定特定表的恢复延迟。指定恢复延迟后，RowStore会在成员失败后一段时间内自动恢复冗余。<br/>

|default-recovery-delay启动属性|成员失败后的效果|
|:--|:--|
|-1|成员失败之后不要自动恢复冗余，除非该表是使用RECOVERYDELAY子句创建的。这是默认值。|
|长大于或等于0|在恢复冗余之前成员发生故障后等待的毫秒数。|
<br/>
默认情况下，成员崩溃后，冗余不会恢复，除非新成员加入系统。如果您希望快速重新启动大多数崩溃的成员，请使用此成员连接冗余恢复的默认设置来帮助您避免在成员关闭时进行不必要的数据洗牌。通过等待丢失的成员重新加入，冗余恢复是使用新启动的成员完成的，分区更好的平衡，较少的处理。<br/>

或者，您可以在`default-recovery-delay`为特定表指定RECOVERYDELAY间隔时使用默认设置。见分条款。<br/>

##### 配置成员加入分区表的冗余恢复 #####
配置成员加入分布式系统后如何为分区表恢复冗余。<br/>

使用gemfirexd.default-startup-recovery-delay系统属性指定成员连接冗余恢复。<br/>

|启动恢复延迟分区属性|成员加入后的效果|
|:--|:--|
|-1|新成员上线后，不会自动恢复冗余。如果使用此`default-recovery-delay`设置并将其设置为-1（默认值），则只能通过调用SYS.REBALANCE_ALL_BUCKETS启动重新平衡来恢复冗余。|
|长大于或等于0|在恢复冗余之前，成员加入后等待的毫秒数。默认值为0（零），每当新的分区表主机加入时，都会立即进行冗余恢复。|

设置`gemfirexd.default-startup-recovery-delay`为高于默认值0的值将允许多个新成员在冗余恢复开始之前加入。在复原过程中出现多个成员时，系统会在其间传播冗余恢复。没有延迟，如果多个成员连续启动，系统可以仅选择启动的大多数或所有冗余恢复的第一个成员。<br/>



> 注意：满足冗余与添加容量不一样。如果满足冗余，则新成员在您调用重新平衡之前不会使用桶。
### 限制会员的记忆消耗 ###
使用CREATE TABLE语句的MAXPARTSIZE子句在可用成员之间对分区数据进行负载平衡。<br/>

所述MAXPARTSIZE子句指定要RowStore构件上的任何分区的最大内存。<br/>

### 预先分配桶 ###
作为最佳方式，在开始加载数据之前，请使用SYS.CREATE_ALL_BUCKETS（' table-name '）过程为分区表创建存储桶。<br/>

SYS.CREATE_ALL_BUCKETS会立即在分区表的数据存储上创建必要的桶。（通常，RowStore会将存储桶分配给服务器，因为插入是针对表执行的。）预先分配存储桶可确保分区表数据均匀分布在集群中。如果不使用SYS.CREATE_ALL_BUCKETS（' table-name '），如果使用并发进程快速加载表数据，则数据可能会变得偏斜。<br/>

如果分区表变得偏斜，请使用SYS.REBALANCE_ALL_BUCKETS重新平衡分布式系统中的所有分区表数据。<br/>

## 对RowStore成员重新平衡分区数据 ##
您可以使用重新平衡来动态增加或减少您的RowStore集群容量，或者改善整个分布式系统的数据平衡。<br/>

重新平衡是影响在群集中创建的分区表的RowStore成员操作。重新平衡执行两项任务：<br/>



- 如果分区表的冗余设置不满足，则重新平衡可以恢复冗余。请参阅**使分区表高度可用**。<br/>


- 重新平衡根据需要将分区表的数据桶移动到主机成员之间，以便在分布式系统中建立最佳数据平衡。
为了提高效率，启动多个成员后，在添加所有成员后，会一次触发重新平衡。



> 注意：如果您的系统中有事务运行，请在计划重新平衡操作时小心。重新平衡可能会在成员之间移动数据，这可能导致正在运行的事务以TransactionDataRebalancedException失败。

使用以下选项之一启动重新平衡操作：<br/>



- 在引导RowStore服务器的命令行：

    snappy-shell rowstore server start -rebalance


- 在正在运行的RowStore成员中执行系统过程：

    call sys.rebalance_all_buckets();
此过程在所有分区表的整个RowStore集群中启动重新平衡桶。<br/>

### 分区表重新平衡如何工作 ###
重新平衡操作异步运行。<br/>

RowStore 1.5版本的一个重要变化是，一次重新平衡不会发生在一个分区表上，而是同时处理所有分区。对于具有共位数据的表，重新平衡可以将表作为一组进行操作，从而维护表之间的数据共享。<br/>

您可以继续访问分区表，同时正在进行重新平衡。查询，DML操作和过程执行在数据移动时继续。如果在本地数据集上执行过程，则在过程执行期间，如果数据移动到另一个成员，您可能会看到性能下降。未来的调用将路由到正确的成员。<br/>

对于基于空闲时间配置为过期的表，重新平衡操作将重置移动的桶上的表条目的上次访问时间。<br/>


### 何时重新分配分区表 ###
通常，当通过成员启动，关闭或失败来增加或减少总容量时，通常需要触发重新平衡。<br/>

当您使用分区表冗余来实现高可用性时，还可能需要重新平衡，并且您已将表配置为在RowStore成员失败后（默认`RECOVERYDELAY`设置）不自动恢复冗余。在这种情况下，当您调用重新平衡操作时，RowStore仅恢复冗余。请参阅**使分区表高度可用**。<br/>

## 管理复制失败 ##
RowStore使用多个故障检测算法来快速检测复制问题。RowStore复制设计侧重于一致性，并且不允许可疑成员或网络分区成员隔离运行。<br/>


### 配置可疑成员警报 ###

当分布式系统的任何成员发生故障时，其他服务很重要，可以快速检测到丢失并将应用程序客户端转移给其他成员。集群中的任何对等体或服务器都可以检测集群的另一个成员的问题，该成员将与成员协调器发起“SUSPECT”处理。会员协调员然后确定嫌犯是否应该保留在分布式系统中，或者应该删除。<br/>

使用该`ack-wait-threshold`属性可以配置RowStore对等体或服务器等待从复制表数据的其他成员接收确认的时间。默认值为15秒; 您指定一个从0到2147483647秒的值。在此期间，复制对等体向分布式系统中的其他成员发送严重的警报警告，在群集中提起“suspect_member”警报。

要配置群集等待确认此警报的时间，请设置`ack-severe-alert-threshold`属性。默认值为零，禁用该属性。


### 如何发生复制失败 ###
复制期间的故障可以通过以下方式进行：



- 发送确认之前副本失败。
当复制过程中终止成员进程时，会发生最常见的故障。当这种情况发生时，来自所有成员的TCP连接被终止，并且成员资格视图被快速更新以反映变化。启动复制的成员继续复制到其他成员。<br/>
如果不是终止，则进程保持活动（但无法响应），启动成员等待一段时间，然后与分布式系统成员资格协调器发出警报。会员协调员根据分布式系统中其他成员的心跳和健康报告来评估嫌疑犯成员的健康状况。协调员可以决定将成员从分布式系统驱逐出去，在这种情况下，它将成员资格视图中的这种变更传达给所有成员。此时，启动复制的成员继续使用可用的对等体和服务器完成复制。此外，连接到此成员的客户端会自动重新路由到其他成员。<br/>


- “拥有”成员失败。
如果特定密钥的指定的数据所有者失败，则系统会自动选择另一副本以成为失败成员管理的关键范围的所有者。更新线程在进行此传输时被阻止。如果至少有一个副本可用，则操作始终从应用程序的角度成功。<br/>

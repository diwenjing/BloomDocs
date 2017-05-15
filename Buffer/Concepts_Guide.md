## DFlow关键概念

DFlow 关键概念主要包括应用程序、流程、数据源、数据流、类型、事件、窗口、连续查询(CQ)、缓存、数据存储、数据目标和订阅等。


### 应用程序

DFlow 可让您开发和运行从外部来源获取数据的自定义应用程序，并且对它们进行实时处理，并通过DFlow仪表板或其他应用程序进行消费。在SQL环境中，每个应用程序的核心都是一个或多个查询。如本概念指南的其余部分所述，应用程序还包含汇聚到一个或多个流中的源，目标和其他逻辑组件，以及其使用的任何图表，映射或其他内置可视化的定义。

应用程序可以使用Web客户端或使用TQL查询语言（TQL）进行图形化创建，TQL是可以使用Java扩展的类似SQL语法（请参阅示例应用程序指南）。TQL也由平台的命令行客户端Tungsten控制台使用(借鉴Spark Tungsten)。

### 数据流

流程定义应用程序接收的数据，处理数据的方式及其结果。

流程由几种组件组成：

来自适配器的实时事件数据

定义其他组件之间定义的数据流

窗口通过时间或计数来绑定事件数据

连续查询过滤，聚合，加入，丰富和转换数据

缓存历史或参考数据，以丰富事件数据

数据存储填充内置的报告和可视化，并持久化处理完的数据

数据目标将数据传递到外部应用程序

应用程序可能包含多个流，以将组件组织成逻辑组。以MultiLogApp为例。

应用程序本身就是可以包含其他流的流，所以当应用程序只包含一个流时，它不需要被显式地创建。以PosApp为例。

### 数据源

数据源是流的起点，定义如何从外部数据源获取数据。数据流可能有多个数据来源。

每个数据源指定：

用于从外部数据源（例如日志文件）（适配器是DFlow服务器的组件）收集实时数据的输入适配器（读取器）;有关详细信息，请参阅阅读器）

所选适配器所需的属性，如主机名，目录路径，身份验证凭证等

使用一些适配器，一个解析器定义了如何处理来自数据源的数据（例如，DSVParser解析分隔的文件，或FreeFormTextParser使用正则表达式解析）

输出流将数据传递给其他流量组件

以下是MultiLogApp示例应用程序的其中一个数据源的TQL代码：

<code>
 CREATE SOURCE Log4JSource USING FileReader (
  directory:'Samples/MultiLogApp/appData',
  wildcard:'log4jLog.xml',
  positionByEOF:false
) 
PARSE USING XMLParser(
  rootnode:'/log4j:event',
  columnlist:'log4j:event/@timestamp,
    log4j:event/@level,
    log4j:event/log4j:message,
    log4j:event/log4j:throwable,
    log4j:event/log4j:locationInfo/@class,
    log4j:event/log4j:locationInfo/@method,
    log4j:event/log4j:locationInfo/@file,
    log4j:event/log4j:locationInfo/@line'
)
OUTPUT TO RawXMLStream;
</code>


Log4JSource使用FileReader适配器读取 /Samples/MultiLogapp/appData/log4jLog.xml，使用XMLPaarser进行解析，并将数据输出到RawXMLStream。在UI中，相同的数据源配置如下所示：

图

注意：“概念指南”中的其他示例仅显示在其TQL表单中，但它们都具有与上述类似的UI对应项。


### 数据流

数据流将一个组件的输出传递给一个或多个其他组件。例如，仅写入文件的简单流程可能具有以下顺序：

<code>
source > stream1> queryA> stream2> FileWriter
</code>

这个更复杂的流程在stream2中分支，以便发送警报并填充仪表板：

<code>
source > stream1 > queryA > stream2 > ...
... stream2 > queryB > stream3 > Subscription
... stream2 > queryC > WActionStore
</code>


### Kafka数据流

DFlow 本地集成了Apache Kafka,一个高吞吐量、低延时、大规模可扩展的消息代理。有关技术说明，请参考 <a href="http://kafka.apache.org">kafka.apache.org</a>。

简单来说，Kafka为DFlow用户提供了将DFlow持久化实时流数据到磁盘上，同时加载到内存中，然后再次重放的能力。如果数据速度太快，无法由内置Kafka Broker处理，可以使用外部Kafka系统，并根据需要进行放大。

从 Kafka 系统重放的数据有很多用途。例如：
</p>
在一个应用中如果您将数据源持久化到Kafka流以及相关的CQ、窗口、缓存，数据目标和另一个应用程序中的数据存储，则可以使第二个应用程序更新代码，并在重新启动时将自动继续处理数据源达到断点续传，零数据丢失，和无没有重复数据。
</p>

开发人员可以使用持久流来对各种TQL应用程序选项进行A/B测试，或执行任何其他有用的实验。

您可以对历史数据执行处理，挖掘一个持久化数据流。例如，如果您正在对安全警报进行故障排除，则可以针对持久性流编写新查询，以收集在数据存储中未捕获的其他数据。

通过持久化数据源到外部Kafka Broker，您可以在通常无法恢复的源（例如HTTPReader，TCPReader和UDPReader）的DFlow群集故障后启用零数据丢失恢复（请参阅从群集故障中恢复）。

持续到外部Kafka Broker还可以利用转发代理恢复运行在远程主机上的数据源。

有关其他信息，请参阅：

持久化数据流到Kafka

用KafkaReader读取Kafka数据流


### 数据类型

数据流与DFlow数据模型类型相关联，类型是一组命名的字段，每个字段具有名称和Java数据类型，例如整数或字符串（请参阅完整列表的支持数据类型）。可以导入和使用任何其他Java类型，尽管有一些限制，例如关于可序列化。一个字段可能具有用于生成功能的密钥。

从数据源接收其输入的流将自动分配与数据源中指定的读取器相关联的DFlow类型。对于其他流，您必须创建一个适当的数据类型。通过查询执行任何投射或其他字段操作。

以下是适用于产品订单数据的DFlow类型的样本TQL代码：

<code>
CREATE  TYPE OrderType（
  storeId       String  KEY，
  orderId       String，
  sku           String，
  orderAmount   Double，
  dateTime DateTime
）;
</code>

这种类型的每个事件将具有购买商店的ID，订单ID，产品的SKU，订单的数量和订单的时间戳。


### 事件

数据流由一系列事件组成，很像SQL环境中的表由行组成。每个事件是对应于流的类型的固定数据元素序列。


### 窗口

一个窗口按时间（例如五分钟）、事件计数（例如，10,000个事件）或两者同时界定实时数据。应用程序需要一个窗口来聚合或执行数据计算，填充仪表板，或者当条件偏离正常参数时发送警报。没有窗口来限制数据，应用程序仅限于评估和对单个事件进行操作。

有两种类型的窗口、滑动和跳跃。当窗口内容发生变化（滑动）或到期（跳跃）时，窗口会向下游查询发送数据。

滑动窗口—始终包含数据流中最近的事件。例如，上午8:06，5分钟的滑动窗口将包含从8:01到8:06的数据; 上午8:07它会包含从8:02 am到8:07 am的数据，依此类推。时间值可以取自传入流的属性（参见ON dateTime下面的示例）。

如果将窗口的大小指定为多个事件，则每次接收到新事件时，最旧的事件将被丢弃。

如果大小被指定为时间长度，则在自添加到窗口之后的指定时间过后，每个事件将被丢弃，因此窗口中的事件数可能会有所不同。编写查询进行计算时，请务必牢记这一点。

如果指定了多个事件和一段时间，则每个事件在窗口中指定的时间之后被丢弃，或者如果必要的话可以避免超过指定的数量。


定期更新跳跃窗口，并提供全新的事件集。例如，五分钟的跳转窗口将输出8：00-8：04：59 am，8：05：00-8：09：59 am的数据集，依此类推。10,000个事件跳转窗口将为每10,000个事件输出一个新的数据集。如果同时指定了五分钟和一万个事件，则窗口将在每次累积10000个事件或从输出上一个数据集开始经过五分钟后输出一个新的数据集。

换句话说，跳跃的窗口将数据流切成块。接收事件的查询，数据存储或数据目标将轮流处理每个块。例如，五分钟跳转窗口将每五分钟刷新一次。

为了获得更好的性能，在将数据发送到窗口之前，使用查询过滤掉任何不需要的字段。

此窗口将RetailOrders数据流（上面讨论）分解成Chunk：

<code>
CREATE JUMPING WINDOW ProductData_15MIN 
OVER RetailOrders 
KEEP WITHIN 15 MINUTE ON dateTime;
</code>

每个 Chunk包含15分钟的事件，使用事件dateTime字段（而不是DFlow主机的系统时钟）的时间戳值测量15分钟。

PARTITION BY field_name选项应用KEEP语句分别为指定字段的每个值。例如，此窗口将包含每个商店100个订单：

<code>
CREATE JUMPING WINDOW Orders100PerStore
OVER RetailOrders
KEEP 100 ROWS  
PARTITION BY storeId;
</code>



### 持续查询

应用程序的大部分逻辑是由持续查询指定的。在大多数方面，大多数方面的查询与SQL类似，持续运行并对实时数据进行操作，而非操作关系表。

查询可用于过滤、聚合、关联、增强和转换事件。查询可以具有多个输入流以组合来自多个数据源，窗口，缓存和/或数据存储的数据。

说明常见用例的一些示例查询：

#### 过滤事件


MultiLogApp示例应用的GetErrors查询过滤Log4ErrorWarningStream中的日志文件数据，仅将错误消息传递给ErrorStream：

<code>
CREATE CQ GetErrors 
 INSERT  INTO ErrorStream 
 SELECT log4j 
 FROM Log4ErrorWarningStream log4j WHERE log4j.level = 'ERROR' ;
</code>

警告消息被丢弃。

#### 过滤字段

TrackCompanyApiDetail查询也来自MultiLogApp示例应用，将数据流中字段的一部分插入到数据存储中：

<code>
CREATE CQ TrackCompanyApiDetail
INSERT INTO CompanyApiActivity(company,companyZip,companyLat,companyLong,state,ts)
SELECT company,companyZip,companyLat,companyLong,state,ts
FROM CompanyApiUsageStream;
</code>

TrackCompanyApiDetail未插入字段的值从TrackCompanyApiSummary的最新插入中获取，具有相同的company值。

#### 警报

<code>
CREATE CQ SendErrorAlerts 
INSERT INTO ErrorAlertStream 
SELECT 'ErrorAlert', ''+logTime, 'error', 'raise', 'Error in log ' + message 
FROM ErrorStream;
</code>

MultiLogApp示例应用程序的SendErrorAlerts查询在ErrorStream中出现错误消息时发送警报。


#### 聚合

PosApp示例应用程序的GenerateMerchantTxRateOnly查询的这一部分汇总来自传入的PosData5Minutes流的数据，并将每个商家每五分钟一批交易中的一个事件输出到MerchantTxRateOnlyStream：

<code>
CREATE CQ GenerateMerchantTxRateOnly
 INSERT  INTO MerchantTxRateOnlyStream
 SELECT p.merchantId，
        FIRST（p.zip），
        FIRST（p.dateTime），
        COUNT（p.merchantId），
        SUM（p.amount）...
 FROM PosData5Minutes p ...
 GROUP  BY p .merchantId;
</code>

每个输出事件包括第一个事务的邮政编码和时间戳，批处理中的总数量以及这些事务的总数。

#### 增强(Enrich)

MultiLogApp示例应用的GetUserDetails查询通过从MLogUserLookup缓存中加入相应的用户和公司名称和邮政编码来增强InfoStream中的事件日志消息事件：

<code>
CREATE CQ GetUserDetails 
INSERT INTO ApiEnrichedStream 
SELECT a.userId, a.api, a.sobject, a.logTime, u.userName, u.company, u.userZip, u.companyZip 
FROM InfoStream a, MLogUserLookup u 
WHERE a.userId = u.userId;
</code>

随后的查询进一步增强了与邮政编码对应的纬度和经度值的数据，并使用结果填充仪表板上的地图。

#### 处理null值

当缓存中的连接不匹配时，以下内容将返回流中的值：

<code>
SELECT ...
FROM stream S
LEFT OUTER JOIN cache C
ON S.joinkey=C.joinkey WHERE C.joinkey IS NULL
</code>


### 缓存

从外部源获取的非实时历史或参考数据基于内存的缓存，例如用于在仪表板地图上显示数据的邮政编码和地理数据的静态文件，或包含用于确定的历史平均值的数据库表何时发送警报。如果定期更新源，则可以设置缓存以适当的间隔刷新数据。

查询通常使用缓存数据来丰富实时数据，例如添加详细的用户或公司信息，或添加纬度和经度值，以便可以在地图上绘制数据。例如，来自PosApp示例应用程序的以下查询丰富了以前从两个单独的缓存NameLookup和ZipLookup中过滤并与公司名称和位置信息进行聚合的实时数据：

<code>
CREATE CQ GenerateWactionContext
INSERT INTO MerchantActivity
SELECT  m.merchantId,
  m.startTime,
  n.companyName,
  m.category,
  m.status,
  m.count,
  m.hourlyAve,
  m.upperLimit,
  m.lowerLimit,
  m.zip,
  z.city,
  z.state,
  z.latVal,
  z.longVal
FROM MerchantTxRateWithStatusStream m, NameLookup n, ZipLookup z
WHERE m.merchantId = n.merchantId AND m.zip = z.zip
LINK SOURCE EVENT;
</code>

<table>
<tr>
<td>注意</td>
</tr>
<tr>
<td>缓存在部署时会被加载到内存中，因此部署具有大缓存的应用程序或数据流可能需要一些时间。</td>
</tr>

</table>

### 数据存储

数据存储根据一个或多个查询中定义的条件存储来自一个或多个源的事件数据。这些事件可能与公共密钥字段相关。存储的数据可以由CQ查询（参见CREATE CQ（查询）），或通过仪表板访问（参见定义仪表板查询）或手动使用控制台（请参阅使用特殊查询浏览数据）查询。使用REST API的外部应用程序也可以直接访问此数据（请参阅使用REST API 查询数据存储）。

数据存储可能仅在内存中存在，或者可能会持久化到磁盘（请参阅CREATE WACTIONSTORE）。如果数据存储(WActionStore)仅存在于内存中，则当可用内存已满时，将删除较旧的事件，为新的存储空间腾出空间。如果将数据持久化到磁盘，则较旧的事件仍可用于查询和可视化以及外部应用程序。


数据存储(WAction)通常包括：

一组相关实时事件的详细数据（可选）

这些事件的计算结果

常用的上下文信息

例如，用户的登录WAVE可能包含：

源IP，登录时间戳和用户每次登录的设备类型（详细数据）

登录次数（计算）

用户名和历史平均登录号（上下文信息）


如果登录次数超过历史平均值一定量，应用程序可以向适当的网络管理员发送警报。

包括详细数据（通过LINK SOURCE EVENT在查询中包含该选项）可让您在可视化中深入查看特定事件。如果应用程序不要求，可以省略详细数据，减少内存需求。

参见PosApp讨论示例。


### 数据目标端

数据目标是整个流处理的终点，并定义数据如何传递到外部应用程序进行存储、分析或其他目的。数据流可能有多个目标端。

每个目标指定：

一个输入流

输出适配器（Writer）将数据写入到文件或数据库（有关更多详细信息，请参阅作者）

使用一些适配器，一个定义如何编写数据的格式化程序（例如，JSONFormatter或XMLFormattter）

所选适配器所需的属性，如主机名，目录路径，身份验证凭证等

参见PosApp示例


### 订阅

订阅通过指定的频道向指定的用户发送警报。

每个订阅指定：

一个输入流

警报适配器

所选适配器所需的属性，如SMTP服务器和电子邮件地址

有关详细信息，请参阅发送警报。
































































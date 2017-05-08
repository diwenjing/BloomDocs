#管理和监视RowStore
##管理和监视RowStore描述如何使用日志文件，系统表和统计数据来了解RowStore部署的行为。本指南还提供了调整RowStore成员，应用程序和单个查询的性能的一般准则。
<br/>
###配置和使用RowStore日志文件 默认情况下，当您以编程方式启动服务器或使用该实用程序时， RowStore会在当前目录中创建一个名为 gfxdserver.log的日志文件snappy-shell。当您启动定位器时， RowStore会在当前工作目录中创建一个名为 gfxdlocator.log的日志文件。
<br/>
您可以log-file在启动服务器或定位器时使用JDBC引导属性指定日志文件的名称和位置。例如：<br/>

snappy-shell rowstore server start -log-file=/home/user1/log/mysnappystorelog.log
产品使用记录 每个RowStore定位器创建一个记录RowStore分布式系统的不同成员资格视图的日志文件。您可以使用这些日志文件的内容来验证您的产品使用情况是否符合您的RowStore许可证。<br/>
日志消息格式 服务器或定位器日志文件中的每条消息都包含严重性级别，时间戳记和其他重要信息。<br/>
严重性级别 您可以将日志记录系统配置为仅记录等于或高于指定日志记录级别的消息。默认情况下，日志记录级别设置为“config”，这意味着系统会在config，info，warning，error和严重级别上记录消息。<br/>
使用java.util.logging.Logger应用程序日志消息 使用RowStore JDBC对等体驱动程序的应用程序可以通过java.util.Logger logging API记录消息。您可以通过在应用程序com.pivotal.gemfirexd使用对等驱动程序连接到RowStore群集之后输入来获取java.util.Logger日志记录对象的句柄。<br/>
使用跟踪标志进行高级 调试RowStore提供了调试跟踪标志，用于记录日志文件中有关RowStore功能的附加信息。
<br/>







###从RowStore系统表获取信息 您可以通过使用SQL命令（系统过程和简单查询）来监视RowStore分布式系统的许多常见方面来收集和分析RowStore系统表中的数据。
<br/>



您可以使用snappy-shell命令行工具或任何第三方工具（如SQuirreL）来查询系统表。也可以使用该snappy-shell工具编写SQL命令，以便您可以轻松地重新运行查询或以固定的时间间隔自动运行查询。<br/>

分布式系统成员信息 SYS.MEMBERS表提供有关构成RowStore分布式系统的所有对等体和服务器的信息。您可以使用不同的查询来获取有关各个成员及其在群集中的角色的详细信息。<br/>
表和数据存储信息 SYS.SYSTABLES表提供了有关在RowStore分布式系统中创建的所有表的信息。您可以使用不同的查询来获取有关表和托管这些表的数据的服务器组的详细信息。<br/>
查询执行时间 SYS.QUERYSTATS表提供了有关RowStore分布式系统中查询执行时间的基本信息。<br/>


###评估查询执行计划 RowStore可以捕获查询执行计划，以确定给定的查询是在本地执行，还是跨集群成员并行执行。

您还可以使用查询执行计划来确定系统在执行特定查询时花费时间的位置。您可以在启用查询计划的任何对等或瘦客户机连接上生成查询计划。<br/>

捕获单个语句的查询计划 RowStore提供EXPLAIN命令以显示单个语句的查询执行计划。<br/>
捕获所有语句的查询计划 作为使用EXPLAIN命令的替代方法，您可以使用内置的系统过程来启用和禁用在连接上执行的所有语句的查询执行计划和统计信息捕获。<br/>
示例查询计划分析 本示例使用已安装的ToursDB数据库来解释如何分析RowStore中的基本查询计划。<br/>
查询计划代码 RowStore可以将以下代码添加到生成的查询计划中。计划中报告的所有时间均为毫秒。<br/>
###覆盖优化器选项 您可以通过在SQL语句中包含 - GEMFIREXD-PROPERTIES子句和属性定义来覆盖RowStore查询优化器的默认行为。子句和属性定义都显示在SQL注释的上下文中（以两个破折号“ - ”开头）。
<br/>

因为优化器属性表示为注释，它们必须出现在行尾。如果要在包含属性定义后继续执行SQL语句，请在继续语句之前输入换行符（\ n）。如果优化器提示出现在SQL语句的末尾，则此规则适用于终止分号。<br/>

注意：处理SQL脚本文件时，如果SQL语句出现在SQL注释行的末尾，则RowStore不会识别SQL语句的其他部分。因此，您必须将SQL语句的其余部分放在单独的行上。这包括终止分号，如果优化器提示延伸到语句的末尾。<br/>

RowStore支持两大类属性：<br/>

FROM子句属性 - 这些属性定义必须在查询的FROM子句和第一个表名之间指定：<br/>
<code>
FROM [ -- GEMFIREXD-PROPERTIES fromProperty = value \n ]<br/>
         TableExpression [,TableExpression]*<br/>
</code>
表属性 - 这些属性适用于前面的基表，属性定义必须出现在TableExpression的结尾（紧邻基表名或别名之后，并且在任何添加表名，JOIN子句或逗号之前）：<br/>
<code>
{ table-name | view-name }<br/>
         [ [ AS ] correlation-Name<br/>
         [ ( simple-column-name [ , simple-column-name ]* ) ] ]<br/>
         [ -- GEMFIREXD-PROPERTIES tableProperty = value ]<br/>
</code>
- GEMFIREXD-PROPERTIES之间的空间是可选的。<br/>

注意：使用 - GEMFIREXD-PROPERTIES子句时，请确保遵守正确的语法。否则可能导致解析器将其解释为注释，并忽略它。<br/>

注意： RowStore还提供仅在查询MEMORYANALYTICS时使用的优化器属性。请参阅管理内存中的表。<br/>



###取消长时间运行的语句 管理RowStore部署时，可能需要取消完成时间过长或导致系统瓶颈的语句。RowStore支持使用系统过程或JDBCStatement.cancel()API取消查询。
<br/>
###能力和限制
###使用SYS.CANCEL_STATEMENT（）取消声明
###使用JDBC API取消语句
###数据认知程序支持取消

###能力和限制
RowStore使您能够手动取消已准备或未准备的SELECT，UPDATE，DELETE或INSERT语句。DDL语句无法取消。手动取消的运行语句抛出SQLState错误XCL56.S：由于用户请求，该语句已被取消。<br/>

注意：如果取消的语句在事务的上下文中执行，则RowStore将回滚该事务。<br/>

注意：当您使用瘦客户端连接的JDBC`Statement.cancel（）`API时，您只能取消准备的UPDATE，INSERT或DELETE语句以及准备或未准备的SELECT语句。要取消未准备好的UPDATE，INSERT或DELETE语句，请执行`Statement.cancel（）`或使用`SYS.CANCEL_STATEMENT（）'过程。<br/>

为了使用该SYS.CANCEL_STATEMENT()过程取消语句，您必须从SYS.SESSIONS表中获取该语句的CURRENT_STATEMENT_UUID。因为SYS.SESSIONS仅列出根DML和SELECT语句，所以从触发器，数据感知过程或查询的子分发中启动的任何语句都不符合取消条件。数据感知程序必须实现ProcedureExecutionContext.checkQueryCancelled()API才能支持语句取消。请参阅数据感知程序中的支持取消。<br/>

SYS.SESSIONS还排除了对等连接，因此从对等客户端执行的语句不符合使用该过程的取消资格。<br/>
对于有资格取消的任何声明，只有当语句执行以下操作之一时，RowStore才支持取消：<br/>

将查询分发到查询协调器<br/>
将子查询分发到RowStore成员<br/>
订购结果<br/>
分组结果<br/>
加入结果<br/>
扫描表和索引<br/>
评估谓词表达式<br/>
RowStore不支持在语句中取消语句：<br/>

检查约束<br/>
从触发器或数据感知过程的主体分发子查询<br/>
排队操作到WAN网关<br/>
与二级桶同步操作<br/>
等待从磁盘读取数据<br/>
等待锁<br/>
持有结果（与SELECT FOR UPDATE语句一样）<br/>
监听器，加载程序或作者回调<br/>
安装或刷新JAR文件<br/>
在滚动升级期间，RowStore不支持取消运行较旧版本的RowStore软件的成员的语句。<br/>


无论您使用Statement.cancel()还是SYS.CANCEL_STATEMENT()请记住，发出取消请求不能确保给定的语句实际上被取消。在收到取消请求之前，语句可能会成功完成，或者由于操作的当前状态，RowStore无法取消该语句。成功返回Statement.cancel()或SYS.CANCEL_STATEMENT()简单地意味着RowStore收到取消请求，并将竭尽全力取消该声明。<br/>
###取消使用声明 SYS.CANCEL\_STATEMENT()
按照以下步骤使用该SYS.CANCEL_STATEMENT()过程取消语句：<br/>

查询SYS.SESSIONS表以获取要取消的语句的CURRENT_STATEMENT_UUID。例如：<br/>
<code>
snappy> select id, session_id, current_statement_uuid, current_statement, current_statement_status from sys.sessions;<br/>
ID                          |SESSION_ID |CURRENT_STATEMENT_UUID   |CURRENT_STATEMENT                                                                                                               |CURRENT_STATEMENT_STATUS<br/>
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------<br/>
pnq-rdiyewar(7438)<v3>:27584|2          |12884901905-12884901918-1|SYSLH0001  select id, session_id, current_statement_uuid, current_statement, current_statement_status from sys.sessions          |EXECUTING STATEMENT<br/>     
pnq-rdiyewar(7194)<v2>:37098|2          |8589934609-8589934687-1  |SYSLH0001  select eqp_id, cntxt_id from CONTEXT_HISTORY where eqp_id||cast(cntxt_id as char(100)) in (select eqp_id||c&           |EXECUTING STATEMENT <br/>    
pnq-rdiyewar(6844):27404    |2          |17-19-1                  |SYSLH0001    call SYS.GET_ALLSERVERS_AND_PREFSERVER(?, ?, ?, ?)                                                                    |SENDING RESULTS       <br/>  
pnq-rdiyewar(6844):27404    |3          |20-22-1                  |SYSLH0001    call SYS.GET_ALLSERVERS_AND_PREFSERVER(?, ?, ?, ?)                                                                 |SENDING RESULTS          <br/>  

4 rows selected <br/>  
</code>
注意： UUID具有“connectionID-statementID-executionID”组件，一个语句可能有多个“executionID”。如果您选择一个“executionID”为0的UUID，则RowStore取而代之的是具有匹配的“connectionID”和“statementID”的语句。 <br/>  

执行SYS.CANCEL_STATEMENT()您要取消的语句的UUID： <br/>  
<code>
snappy> call sys.cancel_statement('8589934609-8589934687-1'); <br/> 
Statement executed. <br/> 
</code>
取消后，运行该语句的会话将收到SQLState错误XCL56。例如，在上述情况下，交互式snappy-shell会话将收到类似于以下的异常： <br/> 
<code>
snappy> select eqp_id, cntxt_id from CONTEXT_HISTORY where eqp_id||cast(cntxt_id as char(100)) in (select eqp_id||cast(t.cntxt_id as char(100)) from RECEIVER_LOG t where 1=1 );<br/> 
ERROR XCL56: SQLSTATE=XCL56,SEVERITY=-1: (Server=pnq-rdiyewar[1529],Thread[DRDAConnThread_15,5,snappydata.daemons]) The statement has been cancelled due to a user request. : <br/> 
java.sql.SQLException: SQLSTATE=XCL56,SEVERITY=-1: (Server=pnq-rdiyewar[1529],Thread[DRDAConnThread_15,5,snappydata.daemons]) The statement has been cancelled due to a user request. : <br/> 
    at com.pivotal.snappydata.internal.client.am.SQLExceptionFactory40.getSQLException(SQLExceptionFactory40.java:103)<br/> 
    at com.pivotal.snappydata.internal.client.am.SqlException.getSQLException(SqlException.java:401)<br/> 
    at com.pivotal.snappydata.internal.client.am.Statement.execute(Statement.java:1009)<br/> 
    at com.pivotal.snappydata.internal.impl.tools.ij.ij.executeImmediate(ij.java:347)<br/> 
    at com.pivotal.snappydata.internal.impl.tools.ij.utilMain.doCatch(utilMain.java:697)<br/> 
    at com.pivotal.snappydata.internal.impl.tools.ij.utilMain.runScriptGuts(utilMain.java:425)<br/> 
    at com.pivotal.snappydata.internal.impl.tools.ij.utilMain.go(utilMain.java:263)<br/> 
    at com.pivotal.snappydata.internal.impl.tools.ij.Main.go(Main.java:309)<br/> 
    at com.pivotal.snappydata.internal.impl.tools.ij.Main.mainCore(Main.java:275)<br/> 
    at com.pivotal.snappydata.internal.impl.tools.ij.Main.main(Main.java:87)<br/> 
    at com.pivotal.snappydata.internal.tools.ij.main(ij.java:66)<br/> 
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)<br/> 
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)<br/> 
    at sun.reflect.DelegatingMethodAccessorImpl.invoke<br/> (DelegatingMethodAccessorImpl.java:43)<br/> 
    at java.lang.reflect.Method.invoke(Method.java:606)<br/> 
    at com.gemstone.gemfire.internal.GemFireUtilLauncher.invoke<br/> (GemFireUtilLauncher.java:167)<br/> 
    at com.pivotal.snappydata.tools.GfxdUtilLauncher.invoke(GfxdUtilLauncher.java:228)<br/> 
    at com.pivotal.snappydata.tools.GfxdUtilLauncher.main(GfxdUtilLauncher.java:145)<br/> 
Caused by: ERROR XCL56: SQLSTATE=XCL56,SEVERITY=-1: (Server=pnq-rdiyewar[1529],Thread[DRDAConnThread_15,5,snappydata.daemons]) The statement has been cancelled due to a user request. : <br/> 
    at com.pivotal.snappydata.internal.client.am.Statement.completeSqlca(Statement.java:2034)<br/> 
    at com.pivotal.snappydata.internal.client.net.NetStatementReply.parseOpenQueryError(NetStatementReply.java:695)<br/> 
    at com.pivotal.snappydata.internal.client.net.NetStatementReply.parseOPNQRYreply(NetStatementReply.java:286)<br/> 
    at com.pivotal.snappydata.internal.client.net.NetStatementReply.readOpenQuery(NetStatementReply.java:83)<br/> 
    at com.pivotal.snappydata.internal.client.net.StatementReply.readOpenQuery(StatementReply.java:62)<br/> 
    at com.pivotal.snappydata.internal.client.net.NetStatement.readOpenQuery_(NetStatement.java:174)<br/> 
    at com.pivotal.snappydata.internal.client.am.Statement.readOpenQuery(Statement.java:1604)<br/> 
    at com.pivotal.snappydata.internal.client.am.Statement.flowExecute(Statement.java:2324)  <br/> 
    at com.pivotal.snappydata.internal.client.am.Statement.executeX(Statement.java:1014)<br/> 
    at com.pivotal.snappydata.internal.client.am.Statement.execute(Statement.java:1000)<br/> 
    ... 15 more<br/> 
</code>

###使用JDBC API取消语句
<br/>
您可以使用JDBC Statement.cancel()方法来取消正在由另一个线程执行的语句。如果您使用Statement.cancel()瘦客户端连接的API，那么您只能取消准备的UPDATE，INSERT或DELETE语句以及准备或未准备的SELECT语句。如果您尝试取消无准备的UPDATE，INSERT或DELETE语句，则该语句将引发与SQLState 0A000的异常，以指示该功能不受支持。<br/>

使用查询超时自动取消语句<br/>
除了支持手动声明取消与Statement.cancel()和SYS.CANCEL_STATEMENT()，RowStore支持设置查询超时，以自动取消长时间运行的语句。<br/>

JDBC应用程序可以使用Statement.setQueryTimeout（）方法配置JDBC驱动程序等待查询完成的最大秒数。在这段时间后，该语句使用SQLState XCL52.S接收到异常：由于查询超时，该语句已被取消。RowStore还提供snappydata.query-timeout系统属性，它在RowStore成员级设置查询超时值。<br/>

数据感知程序必须实现ProcedureExecutionContext.checkQueryCancelled()API才能支持查询超时。请参阅数据感知程序中的支持取消。对于数据感知过程中的嵌套语句，如果语句使用Statement.setQueryTimeout（）设置超过父语句超时的超时值，则会忽略新设置，有利于父设置。<br/>

数据认知程序支持取消<br/>
数据感知过程可以使用嵌套连接执行语句，这些连接不符合取消条件，因为它们没有出现在SYS.SESSIONS表中。为了支持语句取消和从过程中查询超时，您必须ProcedureExecutionContext.checkQueryCancelled()在程序正文中执行嵌套语句时定期调用该方法。例如：<br/>
<code>
example.MyProcedure {<br/>
public static void myProc(String inParam1, int[] outParam2,<br/>
        ExampleObj[] example, int[] count, ResultSet[] resultSet1,<br/>
        ResultSet[] resultSet2,<br/>
        ProcedureExecutionContext context) throws SQLException { <br/>
  ...<br/>
  int executionCount = 0;<br/>
  while (rs.next()) {<br/>
    ...<br/>
    ...<br/>

     // After every 100 iterations of the statement, check to see whether the query was cancelled.<br/>
       context.checkQueryCancelled();<br/>
     }<br/>
  } <br/>
} // end of procedure<br/>


</code>
<br/>




###评估系统和应用程序的统计信息 RowStore提供用于分析系统性能的统计信息。分布式系统的任何成员，包括RowStore服务器，定位器和对等客户端，都可以收集和存档此统计数据。
<br/>

RowStore以可配置的间隔采样统计信息并将其写入存档。档案可随时读取，包括在运行时。<br/>

您可以使用以下工具查看和分析运行时或归档的历史数据：<br/>

snappy-shell stats 是RowStore产品附带的命令行工具。<br/>
GemFire视觉统计显示（VSD）是一种图形工具，安装在RowStore安装的tools / vsd子目录中。请参见使用VSD分析统计信息。<br/>
注意： RowStore统计信息使用Java System.nanoTimer进行纳秒时序。该方法提供纳秒精度，但不一定是纳秒精度。有关更多信息，请参阅与RowStore一起使用的JRE的System.nanoTimer的联机Java文档。<br/>

注意：由于文件系统缓冲，统计档案文件的运行时间查看不一定是实时的。<br/>

###收集系统统计 信息使用系统过程，成员引导属性或连接属性启用RowStore系统统计信息。<br/>
###收集应用程序统计信息 您可以收集自己的应用程序定义的统计信息，以记录应用程序感兴趣的任何内容，例如多语句事务的吞吐量和延迟以及特定于应用程序的配置值。<br/>
###使用VSD分析统计 信息视觉统计显示（VSD）从一个或多个档案读取采样的统计信息，并生成用于分析的图形显示。VSD安装有RowStore在工具子目录。
<br/>



###使用Java管理扩展（JMX） 您可以使用Java管理扩展（JMX）与RowStore进行其他管理和监视功能。<br/>

您可以使用Java管理扩展（JMX）与RowStore进行其他管理和监视功能。<br/>

基本的JMX监控可以与RowStore成员进程紧密集成。您可以配置定位器和/或服务器，以便在成员启动时根据需要自动启动JMX管理器进程。<br/>

管理和监控功能 RowStore使用联合的开放MBean策略来管理和监视分布式系统的所有成员。此策略为您提供了分布式系统的统一的单一代理视图。<br/>
架构和组件 RowStore管理和监视系统由一个JMX管理器节点（应该只有一个）和一个或多个受管节点组成。分布式系统中的所有成员都可以通过MBean进行管理。<br/>
使用JMX管理器节点 RowStore的管理和监视系统由分布式系统中的一个JMX管理器节点（应该只有一个）和一个或多个受管节点组成。分布式系统中的所有成员都使用MBean来进行某些管理和监视功能。<br/>
联邦MBean架构 RowStore使用MBean来管理和监视软件的不同部分。RowStore联合的MBean架构是可扩展的，使您能够拥有RowStore分布式系统的单一代理视图。<br/>

###RowStore Pulse RowStore Pulse是一个Web应用程序，可提供用于监视RowStore集群，成员和表的重要，实时健康和性能的图形化仪表板。
<br/>
使用Pulse检查成员使用的总内存，CPU和磁盘空间，正常运行时间统计信息，客户端连接，WAN连接，查询统计信息和关键通知。Pulse与RowStore JMX管理器进行通信，以提供您的RowStore部署的完整视图。您可以从高级别集群视图向下钻取以查看成员中的各个成员和表。<br/>

默认情况下，RowStore Pulse运行在嵌入在RowStore JMX管理器节点中的Tomcat服务器容器中。您可以选择将Pulse部署到您选择的Web应用程序服务器，以便该工具独立于您的RowStore集群运行。在应用服务器上托管脉冲还可以使用SSL访问应用程序。<br/>

请参见RowStore脉冲系统要求。<br/>

脉冲快速启动（嵌入式模式） 在嵌入式模式下使用脉冲直接从RowStore JMX Manager监视RowStore部署。默认情况下，嵌入式脉冲应用程序连接到承载Pulse应用程序的本地JMX Manager。或者，您可以将Pulse配置为连接到您选择的RowStore系统。<br/>
在专用Web应用程序服务器上的Web应用程序服务器主机上的主机脉冲，使脉冲应用程序可以在一致的地址处使用，或使用SSL访问Pulse应用程序。当您以这种方式主持Pulse时，您还可以将Pulse配置为连接到特定的定位器或JMX Manager节点进行监视。<br/>
配置脉冲验证 脉冲要求所有用户在使用Pulse Web应用程序之前进行身份验证。如果在RowStore JMX Manager节点上配置了JMX认证，则Pulse Web应用程序本身也可能需要在启动时向RowStore JMX Manager节点进行身份验证。<br/>
使用脉冲视图 Pulse提供了各种不同的视图来帮助您监视RowStore集群，成员和表。<br/>



###调优性能的最佳做法 您可以通过遵循RowStore服务器和应用程序的最佳做法来帮助确保最佳性能。
<br/>


您可以通过遵循RowStore服务器和应用程序的最佳做法来帮助确保最佳性能。<br/>

调谐应用逻辑 按照以下准则调整应用程序。<br/>
减少分发开销 潜在敏感应用程序必须最小化进程间通信。对于RowStore，这意味着减少客户端和服务器之间以及服务器之间的流量，因为数据本身是分布式的。<br/>
减少对磁盘的驱逐 减少表的内存开销以及性能开销。<br/>
最小化复制表的更新延迟 请记住，当应用程序更新复制表时，RowStore对承载该表副本的每个成员执行更新。复制表的更新延迟随托管表的RowStore成员数而增加。<br/>
调整FabricServers 这些与JVM相关的建议涉及在Linux上运行的64位版本1.6 JVM。<br/>
调整磁盘I / O 访问硬盘的大多数RowStore应用程序与同步磁盘存储和合理快速的磁盘工作良好。如果磁盘I / O成为瓶颈，您可以配置系统以尽量减少寻道时间。<br/>
在虚拟环境中运行RowStore 使用较新的操作系统版本（如Red Hat 6或SLES 11）可以帮助虚拟环境中的性能提升。VMware还提供了直接适用于RowStore等应用程序的最佳做法指南。<br/>


###检测和处理网络分割（“拆分大脑”） 当发生网络分段时，不能正确处理分区条件的分布式系统允许形成多个子组。这种情况可能导致许多问题，包括在不一致的数据上运行的分布式应用程序。
<br/>


当发生网络分段时，不能处理分区条件的分布式系统正确地允许形成多个子组。这种情况可能导致许多问题，包括在不一致的数据上运行的分布式应用程序。<br/>

例如，因为连接到服务器集群的瘦客户机不会绑定到成员资格系统中，客户端可能会与来自多个子组的服务器进行通信。或者，一组客户端可能会看到一个服务器组，而另一组客户端看不到该子组，但可以看到另一个。<br/>

RowStore通过仅允许一个子组来形成和生存来处理这个问题。其他子组的分布式系统和高速缓存将尽可能快地关闭。通过RowStore日志记录系统提出适当的警报，以提醒管理员采取行动。<br/>

RowStore中的网络分区检测基于主要成员和组管理协调器的概念。协调员是管理分布式系统其他成员进入和退出的成员。对于网络分区检测，协调器始终是RowStore定位器。主要成员始终是分布式系统中最旧的成员，它们没有在同一进程中运行定位器。鉴于此，两种情况会导致RowStore声明一个网络分区：<br/>

如果定位器和引导成员在可配置的时间段内异常离开分布式系统，则声明网络分区，并且立即关闭并断开无法查看定位器和引导成员的成员缓存。<br/>
如果定位器或主要成员的分布式系统正常关闭，RowStore会自动选择一个新的分布式系统并继续运行。<br/>
如果成员没有联系定位器，则声明已发生网络分区，自动关闭，并从分布式系统断开连接。<br/>
通过将enable-network-partition-detection分布式系统属性设置为true来启用网络分区检测。在所有定位器和应对网络分区敏感的任何其他进程中启用网络分区检测。没有启用网络分区检测的进程不符合成为主要成员的资格，因此它们的失败不会触发网络分区的声明。<br/>

注意：分布式系统必须包含定位器以启用网络分区检测。<br/>





###从内存异常中恢复 本主题描述当RowStore内存不足时会发生什么。<br/>

本主题描述当RowStore内存不足时会发生什么。<br/>

由于您可以将RowStore表存储在Java堆或非堆内存中，因此存在两种截然不同的情况：内存不足：<br/>

Java堆内存不足。当Java堆内存不足时，线程开始抛出一个内存OutOfMemoryError<br/>
系统耗尽了堆内存。在这种情况下，线程开始投掷OutOfOffHeapMemoryException<br/>
在任一种情况下，当线程抛出内存异常时，RowStore将关闭缓存并断开与分布式系统的连接，以防止读取不一致的数据。如果您的Java堆内存不足，您将需要终止并重新启动JVM。依赖于JVM的任何应用程序或框架也必须关闭并重新启动。<br/>

如果您的内存不足，您可能会重用现有的JVM; 但是，如果需要修改RowStore系统的配置，则可能需要重新启动JVM。<br/>

当您遇到内存异常时，Pivotal建议您：<br/>

检查你的配置。启动数据存储时，可能需要分配额外的堆（使用-heap-size）和/或堆栈内存（使用-off-heap-size）。有关这些命令行选项的更多信息，请参阅服务器。<br/>
分析您的数据 您可以通过查询SYS.MEMORYANALYTICS表来查看特定表的内存使用情况。请参阅在SYS.MEMORYANALYTICS中查看内存使用情况。<br/>
注意：为了帮助防止内存不足错误，您还可以将RowStore配置为在内存使用率达到一定级别时从其表中驱逐数据，或者首先通过设置临界堆百分比或关键关闭级别来抛出“LowMemoryException”启动数据存储或使用内置系统过程时的堆百分比阈值。见程序。<br/>



































#RowStore Pulse

##RowStore脉冲

###RowStore Pulse RowStore Pulse是一个Web应用程序，可提供用于监视RowStore集群，成员和表的重要，实时健康和性能的图形化仪表板。
<br/>
使用Pulse检查成员使用的总内存，CPU和磁盘空间，正常运行时间统计信息，客户端连接，WAN连接，查询统计信息和关键通知。Pulse与RowStore JMX管理器进行通信，以提供您的RowStore部署的完整视图。您可以从高级别集群视图向下钻取以查看成员中的各个成员和表。<br/>

默认情况下，RowStore Pulse运行在嵌入在RowStore JMX管理器节点中的Tomcat服务器容器中。您可以选择将Pulse部署到您选择的Web应用程序服务器，以便该工具独立于您的RowStore集群运行。在应用服务器上托管脉冲还可以使用SSL访问应用程序。<br/>

请参见RowStore脉冲系统要求。<br/>

脉冲快速启动（嵌入式模式） 在嵌入式模式下使用脉冲直接从RowStore JMX Manager监视RowStore部署。默认情况下，嵌入式脉冲应用程序连接到承载Pulse应用程序的本地JMX Manager。或者，您可以将Pulse配置为连接到您选择的RowStore系统。<br/>












在专用Web应用程序服务器上的Web应用程序服务器主机上的主机脉冲，使脉冲应用程序可以在一致的地址处使用，或使用SSL访问Pulse应用程序。当您以这种方式主持Pulse时，您还可以将Pulse配置为连接到特定的定位器或JMX Manager节点进行监视。<br/>





配置脉冲验证 脉冲要求所有用户在使用Pulse Web应用程序之前进行身份验证。如果在RowStore JMX Manager节点上配置了JMX认证，则Pulse Web应用程序本身也可能需要在启动时向RowStore JMX Manager节点进行身份验证。<br/>
使用脉冲视图 Pulse提供了各种不同的视图来帮助您监视RowStore集群，成员和表。<br/>


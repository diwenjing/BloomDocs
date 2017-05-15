# 使用服务器组来管理数据分发 #
使用服务器组来定义哪些RowStore数据存储主机表的数据。服务器组还定义RowStore存储附加资源的位置，例如网关发送者，网关接收器和AsyncEventListener实现。<br/>


- **服务器组概述** 服务器组定义了RowStore数据存储成员的子集。您可以使用服务器组来控制哪些数据存储表中的主机数据或在分布式系统中部署侦听器实现。

- **将成员添加到** 服务器组当您使用server-groups引导属性启动RowStore成员时，可以定义服务器组成员身份和/或创建服务器组。

- **将表分配给服务器组** 创建新表时，该CREATE TABLE语句可以指定表所属的服务器组。

## 服务器组概述 ##
服务器组定义RowStore数据存储成员的子集。您可以使用服务器组来控制哪些数据存储表中的主机数据或在分布式系统中部署侦听器实现。<br/>

托管数据的任何数量的RowStore成员都可以参与一个或多个服务器组。您在启动RowStore数据存储时指定命名服务器组。<br/>

![](http://rowstore.docs.snappydata.io/docs/common/images/ServerGroup1.gif)

引导的RowStore成员`host-data=false`是一个访问器，不会托管表数据，即使您指定了一个或多个服务器组。但是，托管数据的对等客户端也可以参与服务器组。<br/>

![](http://rowstore.docs.snappydata.io/docs/common/images/ServerGroup2.gif)

默认情况下，主机数据的所有服务器都将添加到“默认”服务器组。<br/>

不同的逻辑数据库模式通常在不同的服务器组中进行管理。例如，订单管理系统可以以部署到一个服务器组的“订单”模式来管理所有客户及其订单。相同的系统可能会在不同的服务器组中管理运输和物流数据。单个对等体或服务器可以参与多个服务器组，通常作为共享相关数据或控制复制表中冗余副本数量的方式。<br/>

通过支持动态组成员身份，托管服务器组数据的进程数可以动态更改。然而，服务器组成员身份的这个动态方面是从应用程序开发人员中抽象出来的，他们可以将服务器组视为单个逻辑服务器。<br/>

服务器组仅确定正在管理表的数据的那些对等体和服务器。分布式系统的任何对等成员以及连接到单个服务器的瘦客户端始终可以访问表。不是表的服务器组之一的数据存储器在对表执行操作时用作访问器。<br/>

当您调用服务器端程序时，可以并行执行服务器组中所有成员的过程。这些数据感知过程也可以在属于服务器组的任何对等客户端上执行。<br/>

没有将表与特定成员IP地址相关联，可以动态增加或减少服务器组的容量，而不会对现有服务器或客户端应用程序造成任何影响。RowStore可以自动将服务器组中的表重新平衡到新添加的成员。<br/>

至少有一个托管表的数据的数据存储成员必须可用，才能对表执行语句。如果所有表的服务器组中的所有数据存储都处于脱机状态，那么涉及该表的语句将失败，并显示“分配系统”中的“ 声明 ” 中的错误X0Z08：否数据存储。<br/>

## 将成员添加到服务器组 ##
使用server-groups引导属性启动RowStore成员时，可以定义服务器组成员身份和/或创建服务器组。<br/>

例如，如果从命令行启动RowStore服务器snappy-shell，请使用该server-groups属性来指定服务器应该加入的一个或多个服务器组的名称：<br/>

	snappy-shell rowstore server start -server-groups=OrdersDB,OrdersReplicationGrp,DBProcessInstance1

> 注意：在SYS.MEMBERS表中存储值之前，RowStore将服务器组名转换为全大写字母。DDL语句和过程自动将所提供的服务器组值转换为全大写字母。但是，当您直接查询SYS.MEMBERS表时，必须指定服务器组的大写值。

在此示例中，RowStore服务器参与三个服务器组：ORDERSDB，ORDERSREPLICATIONGRP和DBPROCESSINSTANCE1。如果这是第一个定义一个命名服务器组的RowStore成员，那么RowStore创建该组，并在启动后添加新成员。如果集群中的其他RowStore成员使用相同的服务器组进行引导，则RowStore会将此新成员添加到现有组。如果不指定`-server-groups`属性，则RowStore会自动将数据存储成员添加到默认服务器组。<br/>

如果从Java应用程序中启动RowStore对等客户端，请将该`server-groups`属性指定为JDBC对等客户端连接字符串的一部分。例如，使用连接URL：

	jdbc:snappydata:;mcast-port=33666;host-data=true;server-groups=OrdersDB,OrdersReplicationGrp,DBProcessInstance1
**使用FabricServer接口启动和停止RowStore成员**和**启动RowStore服务器**提供了有关指定引导属性的更多信息。<br/>

## 将表分配给服务器组 ##
创建新表时，该`CREATE TABLE`语句可以指定表所属的服务器组。<br/>

分区表的数据分布在指定服务器组的所有成员上。复制表的数据将复制到服务器组的所有成员。请参见**复制表**。<br/>

例如，以下命令创建一个复制的表的数据托管在两个服务器组上：<br/>

	CREATE TABLE COUNTRIES
	(
	  COUNTRY VARCHAR(26) NOT NULL,
	  COUNTRY_ISO_CODE CHAR(2) NOT PRIMARY KEY,
	  REGION VARCHAR(26),
	) SERVER GROUPS (OrdersDB, OrdersReplicationGrp)
如果不指定分区，则默认情况下将复制RowStore中的表。<br/>

如果不指定一个或多个服务器组名称，则表数据将在该模式的默认服务器组的所有成员上进行分区或复制。在所有情况下，这种行为可能是不可取的。例如，如果复制表中的数据频繁更改，则在默认组中的每个服务器上维护副本的成本可能会过高。在这种情况下，应用程序开发人员或系统管理员可以有几个成员参与一个新的较小的服务器组，以限制副本的数量。<br/>

当两个表被分区和共同分配时，它会强制两个表中那些列具有相同值的分区位于同一个成员上。并置表必须属于至少一个通用服务器组。最佳做法是在完全相同的服务器组上部署共同表。请参阅**分区表**。<br/>




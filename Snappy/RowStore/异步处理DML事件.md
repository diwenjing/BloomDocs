# 异步处理DML事件 #
AsyncEventListener在RowStore中接收关于执行数据操作语言（DML）语句（创建，更新和删除操作）的回调。在缓存部署中，您可以使用AsyncEventListener来分析DML语句中的数据，并采取所需的操作，例如以特定格式持久保留数据。<br/>

- **AsyncEventListener如何工作** AsyncEventListener实例由其自己的专用线程服务，其中调用回调方法。与DML操作相对应的事件被放置在内部队列中，单线程以先进先出（FIFO）顺序将一批事件一次调度到用户实现的回调类。向侦听器发送事件的频率由RowStore中的AsyncEventListener的配置控制。

- **实现要求** 在**实现AsynchEventListener**之前，请注意要求和准则。

- **配置和使用** AsyncEventListener通过定义实现AsyncEventListener接口的类，安装AsyncEventListener配置以及将表与之关联来实现AsyncEventListener。

## AsyncEventListener如何工作 ##
AsyncEventListener实例由其自己的专用线程服务，其中调用回调方法。与DML操作相对应的事件被放置在内部队列中，单线程以先进先出（FIFO）顺序将一批事件一次调度到用户实现的回调类。向侦听器发送事件的频率由RowStore中的AsyncEventListener的配置控制。<br/>

> 注意： RowStore仅为单个DML事件调用监听器; 批量DML或批处理SQL语句不调用监听器。

要获得DML回调，您创建一个实现类AsyncEventListener接口。回调方法`processEvents(List<Event> events)`提供了批处理中的DML事件列表，每个Event对象都提供了可用于获取以下信息的方法：

|事件信息|描述|
|:--|:--|
|事件类型|EVENT.Type枚举的对象。这可以是AFTER_INSERT，AFTER_UPDATE，AFTER_DELETE，BEFORE_DELETE，BEFORE_INSERT，BEFORE_UPDATE，BULK_DML或BULK_INSERT，这取决于CRUD操作的类型。|
|模式和表名称|当前操作的模式和表名称。|
|老行|表示正在更新的旧行的列值的结果集。|
|新行|表示正在插入或更新的新行的列值的结果集。|
|表MetaData|有关返回的列的类型，属性和位置的信息。|
|首要的关键|表示更改行的主键列的结果集。|

使用事件列表中提供的信息来执行监听器实现的工作。`processEvents`如果成功处理所有事件，您的方法应返回TRUE，在这种情况下，RowStore会从队列中删除已处理的事件。如果您无法成功处理事件并且希望RowStore保留并重新发送事件，而不是从队列中删除事件，则返回FALSE。<br/>

> 注意：如果多个线程更新排队到AsyncEventListener实现的数据，则可能会发生某些数据一致性问题。请参阅**单独线程中DML的无订购保证**。

## 实施要求 ##
在实现AsynchEventListener之前，请注意要求和准则。<br/>

- 所有AsyncEventListener实现都应该检查现有数据库连接可能由于较早的异常而关闭。例如，`Connection.isClosed()`在执行进一步操作之前，请检查catch块并根据需要重新创建连接。RowStore中的DBSynchronizer实现在重新使用连接之前自动执行此类型的检查。

- 一个AsyncEventListener实现必须安装在RowStore系统的一个或多个成员上。您只能在数据存储（配置属性host-data设置为“`true`”的对等体）上安装AsyncEventListeners 。

- 如果活动的AsyncEventListener的RowStore成员关闭，您可以在多个成员上安装一个监听器，以提供高可用性并保证事件传递。在任何给定时间，只有一个成员具有调度事件的主动线程。其他成员的线程保持备用状态以进行冗余。

- 出于性能和内存原因，安装不超过一个备用侦听器（最多为一个）。

- 要通过成员关闭保留挂起的事件，请配置RowStore将AsyncEventListener的内部队列保留到可用的**磁盘存储**。默认情况下，如果活动侦听器的RowStore成员关闭，则驻留在AsyncEventListener的内部队列中的任何挂起的事件都将丢失。

- 为了确保高可用性和可靠的事件传递，请将事件队列配置为持久和冗余。

- 如果您的监听器实现必须将用户凭据传递给外部数据源，请考虑使用**加密密码加密**凭证并使用该external选项。然后，您可以通过初始化参数向收听者提供加密的密码（而不是明文密码）。然后，侦听器可以使用该`AsyncEventHelper.decryptPassword`方法来解密密码，以便连接到外部数据源。

## 配置和使用AsyncEventListener ##
通过定义实现AsyncEventListener接口的类，安装AsyncEventListener配置以及将表与之关联来实现AsyncEventListener。<br/>


- 注意： RowStore还包括内置的DBSynchronizer侦听器的示例代码，它使用最新的AsyncEventListener API。此样本侦听器安装在您的RowStore安装目录的examples / com / snappydata / rowstore / callbacks子目录中。这也包括AsyncEventHelper类的源代码。请记住，您不能原样编译AsyncEventHelper.java代码，因为它使用内部RowStore类。您可以扩展安装的AsyncEventHelper实现，也可以将示例中的公共API代码复制到您自己的类中，根据需要修改代码以创建自定义帮助器类。

主要步骤在以下部分中详细介绍。在开始之前，请阅读**实施要求**。

1. **实现AsyncEventListener接口**。

1. **安装AsyncEventListener配置**。

1. **启动AsyncEventListener配置**。

1. **将表与AsyncEventListener配置关联**。

1. **停止或删除AsyncEventListener配置**。

1. **从AsyncEventListener处理中排除DML操作**。


示例过程显示如何在具有一个对等客户端成员和三个数据存储成员的RowStore系统中配置具有冗余1的AsyncEventListener。服务器组“SG1”中有两个数据存储“DataStore1”和“DataStore2”。只有那些属于服务器组“SG1”的成员才会安装AsyncEventListener。单个表“TESTTABLE”与AsyncEventListener相关联。<br/>

测试您自己的AsyncEventListener实现时，您还需要停止侦听器配置。**停止或删除AsyncEventListener配置描**述此过程。<br/>

## 实现AsyncEventListener接口 ##
定义一个实现类AsyncEventListener接口。RowStore在RowStore安装目录的/ examples / storedprocedure / com / snappydata / rowstore / callbacks目录中安装DBSynchronizer AsyncEventListener实现的源代码。<br/>

> 注：该DBSynchronizer.java文件不使用内部类。但是，必须将示例代码复制到您自己的类中，以避免与内置的DBSynchronizer实现冲突。

## 安装AsyncEventListener配置 ##
通过在RowStore系统的任何成员上执行CREATE ASYNCEVENTLISTENER语句来安装AsyncEventListener实现。您必须指定要在其上部署实现的服务器组，以及实现的唯一标识和类名。<br/>

例如：

	CREATE ASYNCEVENTLISTENER MyListener
	(
	   LISTENERCLASSNAME 'com.pivotal.snappydata.callbacks.TestAsyncEventListener'
	   INITPARAMS 'file=/usr/local/my-listener-params.txt'
	)  SERVER GROUPS ( SG1 );


> 注意：即使您的AsyncEventListener实现不定义参数，也必须包含INITPARAMS参数。Pivotal建议您在单独的文本文件中定义参数，然后使用INITPARAMS'file = path-to-file'引用该文件。您可以在RowStore中配置侦听器实现后，更改或添加参数。见CREATE ASYNCEVENTLISTENER。

> 注意：如果您在初始化参数中发送用户凭据，请考虑使用encrypt-password加密密码，其中包含“external”选项。

> 注意：您可以选择安装AsyncEventListener配置*后*将表与侦听器名称相关联。请确保使用与CREATE ASYNCEVENTLISTNER命令和CREATE TABLE命令相同的侦听器名称。

RowStore在指定的SERVER GROUPS中的每个数据存储上安装AsyncEventListener实例。确保服务器组至少有两个数据存储，以便为侦听器操作提供冗余。作为最佳做法，由于性能和内存原因，安装不超过两个备用侦听器实例（最多两个冗余）。<br/>

当多个数据存储器托管事件监听器时，没有直接的方法可以确保特定的RowStore数据存储器托管活动实例。如果需要特定成员托管活动实例，请确保在部署监听器并附加表时只有该成员正在目标服务器组中运行。然后，您可以在服务器组中启动剩余的数据存储，并且它们将托管冗余的备用实例。<br/>

上述示例`TestAsyncEventListener`使用/usr/local/my-listener-params.txt中定义的初始化参数在服务器组“SG1”的成员上安装标识符“MyListener” 。有关此过程的其他参数的更多信息，请参阅CREATE ASYNCEVENTLISTENER。<br/>

## 启动AsyncEventListener配置 ##
安装AsyncEventListener实现后，可以通过在SYS.START_ASYNC_EVENT_LISTENER过程中指定实现的唯一标识符来启动它。例如：<br/>

	java.sql.Connection conn = getConnection();    
	    CallableStatement cs = conn.prepareCall("call SYS.START_ASYNC_EVENT_LISTENER( ? )");
	    cs.setString(1, "MyListener");
	    cs.execute();
执行时，单线程服务于AsyncEventListener的队列，并以FIFO顺序将事件分派给回调类。


## 将表与AsyncEventListener配置关联 ##
您使用`AsyncEventListener`表的CREATE TABLE语句中的子句将表与AsyncEventListener配置相关联。例如，要将“TestTable”表与“MyListener”标识的AsyncEventListenerConfiguration相关联，请使用以下语句：<br/>
	
	 java.sql.Connection conn = getConnection();    
	 Statement stmt = conn.createStatement();
	 stmt.execute("CREATE TABLE TESTTABLE (ID int NOT NULL, DESCRIPTION varchar(1024),
	 ADDRESS varchar(1024) ) AsyncEventListener(MyListener) " ); 

> 注意：由于关联的侦听器配置在创建表时不必可用，如果指定的侦听器名称不存在，则RowStore不会显示错误消息。请确保使用与CREATE ASYNCEVENTLISTNER命令和CREATE TABLE命令相同的侦听器名称。

> 注意：您可以使用RowStore ALTER TABLE语句创建表后，将表与AsyncEventListener配置关联。

## 显示已安装的AsyncEventListeners ##
您可以加入SYSTABLES，MEMBERS和ASYNCEVENTLISTENERS表，以显示与表相关联的侦听器实现以及安装侦听器的数据存储ID：

	select t.*, m.ID DSID from SYS.SYSTABLES t, SYS.MEMBERS m, SYS.ASYNCEVENTLISTENERS a
	       where t.tablename='<table>' and groupsintersect(a.SERVER_GROUPS, m.SERVERGROUPS)
	       and groupsintersect(t.ASYNCLISTENERS, a.ID);
请参见SYSTABLES和ASYNCEVENTLISTENERS。


停止或删除AsyncEventListener配置
要停止运行的AsyncEventListener配置，请首先使用SYS.WAIT_FOR_SENDER_QUEUE_FLUSH过程来确保没有排队的事件丢失。然后使用SYS.STOP_ASYNC_EVENT_LISTENER并指定要停止的配置的标识符：

    java.sql.Connection conn = getConnection();    
    CallableStatement cs1 = conn.prepareCall("call SYS.WAIT_FOR_SENDER_QUEUE_FLUSH(?,?,?);
    cs1.setString(1, "MyListener");
    cs1.setString(2, "TRUE");
    cs1.setString(3, "0");
    cs1.execute();
    CallableStatement cs2 = conn.prepareCall("call SYS.STOP_ASYNC_EVENT_LISTENER( ? )");
    cs2.setString(1, "MyListener");
    cs2.execute();
这等待所有排队的事件刷新，然后停止将调度事件发送到回调侦听器类。此外，`close()`调用Callback AsyncEventListener类的方法，以便实现可以执行任何必要的清理。<br/>

您也可以使用DROP ASYNCEVENTLISTENER从群集中删除侦听器实现。<br/>


## 从AsyncEventListener处理中排除DML操作 ##
一个AsyncEventListener实现可以根据接收到的事件的信息选择对事件执行或忽略。作为替代方案，与RowStore系统的单独连接可以将skip-listeners属性设置为true，以防止通过连接执行的所有DML操作排队并传递到配置的监听器。<br/>
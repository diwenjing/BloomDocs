# 同时处理DML事件 #
RowStore提供了同步缓存插件机制来处理缓存事件。<br/>

- **Writer和Listener缓存插件** 您可以实现两种类型的同步缓存插件：写入程序和监听器。
- **示例编写器实现** RowStore将产品目录中的**示例编写器实现**安装为 /examples/storedprocedure/EventCallbackWriterImpl.java。如果相应的驱动程序在类路径中可用，则此writer实现可用于对任何JDBC数据源执行写入操作。有关详细信息，请参阅示例注释。

## Writer和Listener缓存插件 ##
您可以实现两种类型的同步缓存插件：写入程序和监听器。<br/>

Writer和监听器缓存插件的操作方式如下：<br/>

- 侦听器 - 侦听器使您能够接收事件对表的更改的通知（插入，更新和删除）。可以为同一个表定义任意数量的侦听器。侦听器回调被同步调用，所以如果回调阻止，它们将导致DML操作阻塞。

- 写入器 - 缓存写入器是一个事件处理程序，可以在发生更改之前同步处理对表的更改。缓存写入器的主要用途是使外部数据源与RowStore中的表保持同步，或者在RowStore中执行DML操作之前执行其他检查或工作。<br/>
高速缓存写入器为您的外部数据源提供直写缓存。与监听器不同，只能将一个写入器附加到表。当表由于插入，更新或删除而即将被修改时，RowStore会通过回调通知作者挂起的操作。作者可以选择通过抛出SQLException来禁止操作。编写器回调被同步调用，如果回调阻塞，将阻止该操作。
> 注意： RowStore在调用写回调之前不检查主键或唯一键约束。

> 注意： RowStore仅为单个DML事件调用监听器; 批量DML或批处理SQL语句不调用监听器。

您可以使用SYS.ATTACH_WRITER和SYS.ADD_LISTENER内置系统过程将写入程序和侦听器附加到表。<br/>

监听器和写入器都必须是EventCallback 接口的实现。接口的主要方法是onEvent方法，它在writer和listener实现中都被调用。<br/>

通过回调传递的Event对象提供以下信息：<br/>

- 事件的类型。TYPE.BEFORE_INSERT，TYPE.BEFORE_UPDATE和TYPE.BEFORE_DELETE提供给写入程序，以指示插入，更新或删除是否即将发生。同样的听众也会收到TYPE.AFTER_INSERT，TYPE.AFTER_UPDATE和TYPE.AFTER_DELETE。

- 旧行作为List <Object>（用于更新或删除操作）。

- 作为List <Object>的新行（用于插入或更新操作）。

- ResultSetMetaData（表的元数据）。

- 关于事件的来源是否是本地的信息。

- GetModifiedColumns（） 已更新列的基于1的列位置的int []。

- 行的主键被修改为对象[]。


[RowStore API](http://rowstore.docs.snappydata.io/docs/reference/store_api/ref-snappystore-api.html#concept_93329D7D8F0445C99F743B891C5B3DC2)提供了使用回调的其他信息和示例。

## 作者实例 ##
RowStore将产品目录中的示例编写器实现安装为/examples/storedprocedure/EventCallbackWriterImpl.java。如果相应的驱动程序在类路径中可用，则此writer实现可用于对任何JDBC数据源执行写入操作。有关详细信息，请参阅示例注释。

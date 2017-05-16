# 使用RowStore缓存数据 #
使用RowStore 缓存数据描述了如何使用RowStore作为现有的传统RDBMS系统的高性能缓存。本指南介绍了各种缓存策略，并介绍了如何实现和使用驱逐功能，侦听器和缓存插件来实现缓存。<br/>

- **RowStore缓存策略** 您可以将RowStore部署为传统关系数据库系统中管理的数据的分布式缓存。RowStore提供了将该产品用作数据库缓存的几种策略。

- **使用RowLoader加载现有数据** 当您使用RowStore作为缓存时，您可以配置一个SQL数据加载器，该数据加载器被触发以从RowStore中的一个未完成的后端存储库加载数据。

- **同步处理DML事件RowStore** 提供了同步缓存插件机制来处理缓存事件。

- **异步处理DML事件AsyncEventListener** 在RowStore中接收到执行数据操作语言（DML）语句（创建，更新和删除操作）的回调。在缓存部署中，您可以使用AsyncEventListener来分析DML语句中的数据，并采取所需的操作，例如以特定格式持久保留数据。

- **使用DBSynchronizer将DML应用于RDBMS** DBSynchronizer是一个内置的AsyncEventListener实现，可用于将数据异步地保存到符合JDBC 4.0标准的第三方RDBMS中。

- **抑制RowStore连接的事件回调** 当您使用RowStore管理缓存的RDBMS时，您可能偶尔需要修改表数据，而不会触发已配置的AsyncEventListener（包括DBSynchronizer）或缓存插件实现。RowStore提供了skip-listeners连接属性来禁用与RowStore集群的特定连接的DML事件传播。
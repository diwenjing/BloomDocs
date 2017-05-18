# RowStore参考 #
该RowStore参考描述RowStore命令行工具，API和标准查询语言（SQL）实现。

- **配置属性** 您可以使用JDBC连接属性，连接启动属性和Java系统属性来配置RowStore成员和连接。

- **磁盘存储恢复实用程序** 提供了磁盘存储数据恢复实用程序，以帮助从损坏的diks存储文件中提取可用数据。

- **snappy -shell启动器命令** 使用 gfxd命令行实用程序启动RowStore实用程序。

- **snappy-shell** Interactive Commands snappy-shell实现了基于Apache Derbyij工具的交互式命令行工具。使用snappy-shell于针对RowStore集群中运行的脚本或交互式查询。

- **SQL语言参考** 了解有关SQL关键字，语句，子句，表达式，函数等的RowStore实现和要求。

- **系统表** RowStore用于管理分布式系统及其数据的表。所有系统表都是SYS模式的一部分。

- **RowStore API** 以下是RowStore类和接口的简要说明，其中包含指向JavaDoc和相关用户指南部分的链接。

- **JDBC API** 有关RowStore实现java.sql类，接口和方法的信息。

- **ADO.NET** API RowStore ADO.NET驱动程序实现了Microsoft的ADO.NET接口和抽象类。

- **ODBC API** 本节列出了RowStore 1.4.1的Progress DataDirect ODBC驱动程序支持的API。

- **产品限制** RowStore对SQL语句，子句和表达式具有限制和限制。

- **异常消息和SQL国** JDBC驱动程序返回的SQLException s，从RowStore所有错误。如果异常始发于用户类型，但本身不是 SQLException，则将其包装在 SQLException中。RowStore特定的 SQLException使用以 X开头的 SQLState类代码。返回标准 SQLState值以适用例外。
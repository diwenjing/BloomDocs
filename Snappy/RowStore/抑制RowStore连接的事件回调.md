# 抑制RowStore连接的事件回调 #
当您使用RowStore管理缓存的RDBMS时，您可能偶尔需要修改表数据，而不会触发已配置的AsyncEventListener（包括DBSynchronizer）或缓存插件实现。RowStore提供了`skip-listeners`连接属性来禁用与RowStore集群的特定连接的DML事件传播。<br/>

您可以使用`skip-listeners`属性与对等客户端或瘦客户端连接。将属性设置为“true”时，RowStore不会将连接生成的DML事件发送到已配置的DBSynchronizer，AsyncEventListener或缓存插件（writer或listener）实现。<br/>

> 注意：将此属性设置为true不会影响使用配置的网关发件人的WAN复制。RowStore总是将DML操作传播到启用的WAN站点。

例如：

	Properties props = new Properties();
	   props.put("skip-listeners","true");
	   final Connection conn = DriverManager.getConnection("jdbc:snappydata:", props);
或者，将属性直接添加到连接字符串中：

	final Connection conn = DriverManager.getConnection("jdbc:snappydata:;skip-listeners=true");
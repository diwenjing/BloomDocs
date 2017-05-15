#从RowStore系统表获取信息
###从RowStore系统表获取信息 您可以通过使用SQL命令（系统过程和简单查询）来监视RowStore分布式系统的许多常见方面来收集和分析RowStore系统表中的数据。
<br/>



您可以使用snappy-shell命令行工具或任何第三方工具（如SQuirreL）来查询系统表。也可以使用该snappy-shell工具编写SQL命令，以便您可以轻松地重新运行查询或以固定的时间间隔自动运行查询。<br/>

###分布式系统成员信息 SYS.MEMBERS表提供有关构成RowStore分布式系统的所有对等体和服务器的信息。您可以使用不同的查询来获取有关各个成员及其在群集中的角色的详细信息。<br/>



确定群集成员资格<br/>
确定群集成员资格<br/>
确定服务器组成员资格<br/>
要显示参与给定群集的所有成员的列表，只需查询sys.members中的所有ID条目即可。例如：<br/>
<code>
select ID from SYS.MEMBERS;<br/>
ID<br/>
-------------------------------------<br/>
vmc-ssrc-rh154(21046)<v3>:42445/43015<br/>
vmc-ssrc-rh156(18283)<v2>:28299/41236<br/>
vmc-ssrc-rh154(21045)<v3>:15459/55080<br/>
vmc-ssrc-rh156(18284)<v2>:43070/57131<br/>
vmc-ssrc-rh156(18285)<v1>:23603/44410<br/>
vmc-ssrc-rh154(20904)<v0>:11368/40245<br/>

6 rows selected<br/>
</code>
返回的行数对应于集群中的对等体，服务器和定位器的总数。<br/>

要确定每个成员在系统中的角色，请在查询中包含KIND列：<br/>
<code>
select ID, KIND from SYS.MEMBERS;<br/>
ID                                          |KIND<br/>
-------------------------------------------------------------<br/>
vmc-ssrc-rh154(26615)<v0>:31141/53707       |accessor(normal)<br/>
vmc-ssrc-rh154(26739)<v3>:9287/48842        |datastore(norma&<br/>
vmc-ssrc-rh156(23870)<v1>:23802/60824       |datastore(norma&<br/>
vmc-ssrc-rh154(26751)<v4>:42054/49195       |datastore(norma&<br/>
vmc-ssrc-rh156(23897)<v2>:37163/43747       |datastore(norma&<br/>
vmc-ssrc-rh156(23882)<v2>:26779/35052       |accessor(normal)<br/>

6 rows selected<br/>
</code>
数据存储成员在集群中托管数据，而访问者成员不托管数据。此角色由host-data引导属性决定。如果集群只包含一个数据存储，则其KIND列为“数据存储（loner）”。<br/>


确定服务器组成员资格<br/>
托管数据的RowStore对等体和服务器可以参与一个或多个服务器组。您可以使用服务器组成员资格来指定如何在集群中分割或复制单个表。您还可以使用服务器组来定向事件侦听器和其他RowStore功能。要显示每个集群成员的服务器组成员身份，请查询SERVERGROUPS列。例如：<br/>
<code>
select ID, SERVERGROUPS from SYS.MEMBERS;<br/>
ID                                          |SERVERGROUPS<br/>
----------------------------------------------------------<br/>
vmc-ssrc-rh154(26615)<v0>:31141/53707       |<br/>
vmc-ssrc-rh154(26739)<v3>:9287/48842        |SG2<br/>
vmc-ssrc-rh156(23897)<v2>:37163/43747       |SG1<br/>
vmc-ssrc-rh156(23882)<v2>:26779/35052       |SG1<br/>
vmc-ssrc-rh156(23870)<v1>:23802/60824       |SG2<br/>
vmc-ssrc-rh154(26751)<v4>:42054/49195       |SG1<br/>

6 rows selected<br/>
</code>
您不启动而不指定服务器组名称的数据存储将添加到默认服务器组，并且对于SERVERGROUPS列具有空值。<br/>

或者，您可以使用GROUPSINTERSECT函数确定哪些成员是您在函数中指定的服务器组的一部分。请注意，如果指定逗号分隔列表，则GROUPSINTERSECT需要对列表进行排序。例如：<br/>
<code>
select ID, SERVERGROUPS from SYS.MEMBERS where GROUPSINTERSECT(SERVERGROUPS, 'SG1,SG2');<br/>
ID                                           |SERVERGROUPS<br/>
------------------------------------------------------------------------<br/>
vmc-ssrc-rh156(23897)<v2>:37163/43747        |SG1<br/>
vmc-ssrc-rh154(26751)<v4>:42054/49195        |SG1<br/>
vmc-ssrc-rh156(23882)<v2>:26779/35052        |SG1<br/>
vmc-ssrc-rh156(23870)<v1>:23802/60824        |SG2<br/>
vmc-ssrc-rh154(26739)<v3>:9287/48842         |SG2<br/>

5 rows selected<br/>
</code>



###表和数据存储信息 SYS.SYSTABLES表提供了有关在RowStore分布式系统中创建的所有表的信息。您可以使用不同的查询来获取有关表和托管这些表的数据的服务器组的详细信息。<br/>




显示表的列表<br/>
显示表的列表<br/>
确定存储数据的位置<br/>
确定表是否复制或分区<br/>
确定存储持久性数据的方式<br/>
显示排除设置<br/>
显示索引<br/>
显示已安装的AsyncEventListeners<br/>
要显示集群中所有表的列表：<br/>
<code>
select TABLESCHEMANAME, TABLENAME from SYS.SYSTABLES order by TABLESCHEMANAME;<br/>
TABLESCHEMANAME            |TABLENAME<br/>
-----------------------------------------------<br/>
APP                        |PIZZA_ORDER_PIZZAS<br/>
APP                        |PIZZA<br/>
APP                        |TOPPING<br/>
APP                        |PIZZA_ORDER<br/>
APP                        |HIBERNATE_SEQUENCES<br/>
APP                        |HOTEL<br/>
APP                        |BASE<br/>
APP                        |PIZZA_TOPPINGS<br/>
APP                        |BOOKING<br/>
APP                        |CUSTOMER<br/>
SYS                        |SYSCONSTRAINTS     <br/>                                                                                                             
SYS                        |ASYNCEVENTLISTENERS<br/>                                                                                                             
SYS                        |SYSCOLPERMS     <br/>                                                                                                                
SYS                        |SYSKEYS        <br/>                                                                                                                 
SYS                        |SYSFILES       <br/>                                                                                                                 
SYS                        |SYSCONGLOMERATES<br/>                                                                                                                
SYS                        |SYSDEPENDS       <br/>                                                                                                               
SYS                        |SYSROUTINEPERMS  <br/>                                                                                                               
SYS                        |SYSTRIGGERS     <br/>                                                                                                                
SYS                        |SYSTABLES       <br/>                                                                                                                
SYS                        |SYSALIASES    <br/>                                                                                                                  
SYS                        |SYSROLES      <br/>                                                                                                                  
SYS                        |SYSDISKSTORES  <br/>                                                                                                                 
SYS                        |SYSVIEWS      <br/>                                                                                                                  
SYS                        |SYSSTATISTICS <br/>                                                                                                                  
SYS                        |SYSCOLUMNS    <br/>                                                                                                                  
SYS                        |SYSCHECKS     <br/>                                                                                                                  
SYS                        |GATEWAYSENDERS <br/>                                                                                                                 
SYS                        |SYSFOREIGNKEYS  <br/>                                                                                                                
SYS                        |SYSSCHEMAS      <br/>                                                                                                                
SYS                        |SYSTABLEPERMS    <br/>                                                                                                               
SYS                        |SYSSTATEMENTS    <br/>                                                                                                               
SYSIBM                     |SYSDUMMY1       <br/>                                                                                                                
SYSSTAT                    |SYSXPLAIN_SORT_PROPS <br/>                                                                                                           
SYSSTAT                    |SYSXPLAIN_DIST_PROPS    <br/>                                                                                                        
SYSSTAT                    |SYSXPLAIN_STATEMENTS    <br/>                                                                                                        
SYSSTAT                    |SYSXPLAIN_RESULTSET_TIMINGS <br/>                                                                                                    
SYSSTAT                    |SYSXPLAIN_SCAN_PROPS        <br/>                                                                                                    
SYSSTAT                    |SYSXPLAIN_STATEMENT_TIMINGS<br/>                                                                                                     
SYSSTAT                    |SYSXPLAIN_RESULTSETS       <br/>                                                                                                     

40 rows selected<br/>
</code>
确定存储数据的位置<br/>
确定将哪些表部署到特定的一组服务器组：<br/>
<code>

select TABLESCHEMANAME, TABLENAME from SYS.SYSTABLES <br/>
       where GROUPSINTERSECT(SERVERGROUPS, 'SG1,SG2');<br/>
TABLESCHEMANAME          |TABLENAME<br/>
--------------------------------------------------------------<br/>
APP                      |PIZZA_ORDER_PIZZAS<br/>
APP                      |PIZZA_ORDER<br/>
APP                      |BASE<br/>
APP                      |PIZZA_TOPPINGS<br/>

4 rows selected<br/>
</code>
对于特定的表或一组表，可以列出承载该表的数据的所有RowStore成员：<br/>
<code>
select m.ID from SYS.SYSTABLES t, SYS.MEMBERS m where t.TABLESCHEMANAME='APP' <br/>
     and t.TABLENAME='PIZZA' and  m.HOSTDATA = 1 <br/>
     and (LENGTH(t.SERVERGROUPS) = 0 or GROUPSINTERSECT(t.SERVERGROUPS, m.SERVERGROUPS));<br/>
ID<br/>
-------------------------------------<br/>
vmc-ssrc-rh156(23870)<v1>:23802/60824<br/>
vmc-ssrc-rh154(26751)<v4>:42054/49195<br/>
vmc-ssrc-rh156(23897)<v2>:37163/43747<br/>
vmc-ssrc-rh154(26739)<v3>:9287/48842<br/>

4 rows selected<br/>
</code>

确定表是否复制或分区<br/>
DATAPOLICY列指定表是复制还是分区，以及表是否持久存储到磁盘存储。例如：<br/>

	select TABLENAME, DATAPOLICY from SYS.SYSTABLES where TABLESCHEMANAME = 'APP';
	TABLENAME                   |DATAPOLICY
	--------------------------------------------------
	PIZZA                       |PERSISTENT_REPLICATE
	PIZZA_ORDER_PIZZAS          |PARTITION
	BASE                        |REPLICATE
	TOPPING                     |REPLICATE
	PIZZA_TOPPINGS              |PARTITION
	PIZZA_ORDER                 |PERSISTENT_PARTITION
	
	6 rows selected

确定存储持久性数据的方式<br/>
对于持久性表，还可以显示持久化表的数据的磁盘存储，以及该表是使用同步还是异步持久性：<br/>

	select TABLENAME, DISKATTRS from SYS.SYSTABLES where TABLESCHEMANAME = 'APP';
	TABLENAME                  |DISKATTRS
	----------------------------------------------
	PIZZA                      |DiskStore is GFXD-DEFAULT-DISKSTORE; Synchronous writes to disk
	PIZZA_ORDER_PIZZAS         |DiskStore is OVERFLOWDISKSTORE;Asynchronous writes to disk
	BASE                       |NULL
	TOPPING                    |NULL
	PIZZA_TOPPINGS             |DiskStore is GFXD-DEFAULT-DISKSTORE; Synchronous writes to disk
	PIZZA_ORDER                |DiskStore is GFXD-DEFAULT-DISKSTORE; Synchronous writes to disk
	
	6 rows selected

###显示排除设置<br/>
使用EVICTIONATTRS列来确定表是否使用驱逐设置以及表是否配置为溢出到磁盘。例如：<br/>

	select TABLENAME, EVICTIONATTRS  from SYS.SYSTABLES where TABLESCHEMANAME = 'APP';
	TABLENAME                   |EVICTIONATTRS
	-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
	PIZZA                       |NULL
	PIZZA_ORDER_PIZZAS          | algorithm=lru-entry-count; action=overflow-to-disk; maximum=100
	BASE                        |NULL
	TOPPING                     |NULL
	PIZZA_TOPPINGS              | algorithm=lru-entry-count; action=overflow-to-disk; maximum=100
	PIZZA_ORDER                 |NULL
	
	6 rows selected

###显示索引
使用CONGLOMERATENAME加入SYSTABLES以确定表是否具有索引并显示索引列。例如：<br/>

	select CONGLOMERATENAME from SYS.SYSCONGLOMERATES c, SYS.SYSTABLES t 
	      where c.ISINDEX = 1 and c.TABLEID = t.TABLEID and t.TABLESCHEMANAME = 'APP' 
	      and t.TABLENAME = 'PIZZA';
	CONGLOMERATENAME
	----------------
	2__PIZZA__ID
	6__PIZZA__BASE
	
	2 rows selected

显示已安装的AsyncEventListeners<br/>
如果安装AsyncEventListener实现，则可以加入SYSTABLES，MEMBERS和ASYNCEVENTLISTENERS表，以显示与表关联的侦听器实现以及安装侦听器的数据存储ID：<br/>
	
	select t.*, m.ID DSID from SYS.SYSTABLES t, SYS.MEMBERS m, SYS.ASYNCEVENTLISTENERS a
	       where t.tablename='<table>' and groupsintersect(a.SERVER_GROUPS, m.SERVERGROUPS)
	       and groupsintersect(t.ASYNCLISTENERS, a.ID);
请参见SYSTABLES和ASYNCEVENTLISTENERS。<br/>

###查询执行时间 SYS.QUERYSTATS表提供了有关RowStore分布式系统中查询执行时间的基本信息。<br/>




程序<br/>
QUERYSTATS提供了一种简单的方法来确定在RowStore集群中执行查询所花费的时间。您可以使用QUERYSTATS中的信息来确定一段时间内运行时间最长的查询，然后使用“ 评估查询执行计划”中的技术来关注这些查询以进一步执行性能调整。
<br/>
默认情况下，RowStore不会在SYS.QUERYSTATS中收集统计信息。您必须使用SYS.SET_QUERYSTATS过程启用收集。请注意，启用查询统计信息收集在SYS.QUERYSTATS虚拟表中直接存储分布式系统中所有查询的信息。仅在固定的时间段内以此方式收集查询统计信息，或者为了评估一系列查询，然后禁用收集。要收集关于语句性能的长期统计信息，请使用“ 系统和应用程序评估统计”中的技术来收集外部文件中的统计信息，并使用VSD工具进行分析。
<br/>
要在SYS.QUERYSTATS表中收集和使用查询统计信息：<br/>

启动snappy-shell并执行SYS.SET_QUERYSTATS过程以启用SYS.QUERYSTATS中的统计信息收集：<br/>
	$ snappy
	snappy> connect client 'localhost:1527';
	snappy> call sys.set_querystats(1);
执行要比较的一系列查询，或等待一段时间来收集系统中有效查询的统计信息。<br/>
查询SYS.QUERYSTATS以显示所有查询，按最长总执行时间排序：<br/>
	snappy> select sum(totaltime)/avg(numinvocations) t, query from sys.querystats 
	> where numinvocations > 0 group by query order by t desc; 
作为替代，通过添加FETCH FIRST子句仅显示前10个或更少的查询。例如：<br/>
	snappy> select sum(totaltime)/avg(numinvocations) t, query from sys.querystats 
	> where numinvocations > 0 group by query order by t desc fetch first 10 rows only; 
最后，在SYS.QUERYSTATS中禁用统计信息收集：<br/>
<code>

snappy> call sys.set_querystats(0);
</code>
使用有关长时间运行查询的信息，请考虑使用“评估系统和应用程序的统计信息”中的技术来提高查询性能。
<br/>



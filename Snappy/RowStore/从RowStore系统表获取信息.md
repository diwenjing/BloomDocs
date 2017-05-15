#从RowStore系统表获取信息
## 您可以通过使用SQL命令（系统过程和简单查询）来监视RowStore分布式系统的许多常见方面来收集和分析RowStore系统表中的数据。
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
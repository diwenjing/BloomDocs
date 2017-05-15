## SnappyData 概述
本节介绍几种常见操作，例如启动集群，使用表（加载，查询，更新），处理流和运行近似查询。


您可以通过以下任何方式运行示例：

在本地模式下：通过使用bin/run-example脚本（运行Scala示例）或使用bin/spark-submit脚本（运行Python示例）。这些示例与Spark + SnappyData Store并置在同一个JVM中。

作为一个工作：许多Scala示例也被实现为SnappyData作业。在这种情况下，可以将示例作为作业提交到正在运行的SnappyData集群。有关如何运行作业的详细信息，请参阅作业部分。

注意：SnappyData还支持Java API。请参阅文件针对Java API的更多细节。


### 如何启动SnappyData群集
#### 在单台机器上启动SnappyData集群

启动群集：运行sbin/snappy-start-all.sh脚本以使用默认设置在单台计算机上启动SnappyData群集。这将启动一个引导节点，一个定位器和一个数据服务器。

<code>
$ sbin/snappy-start-all.sh
</code>

在本地机上的整个集群启动可能需要30秒或更长时间来启动。

打印输出：输出显示如下：

<code>
Starting SnappyData Locator using peer discovery on: localhost[10334]
Starting DRDA server for SnappyData at address localhost/127.0.0.1[1527]
Logs generated in /home/user/snappyData/work/localhost-locator-1/snappylocator.log
SnappyData Locator pid: 9368 status: running
Starting SnappyData Server using locators for peer discovery: user1-laptop[10334]
Starting DRDA server for SnappyData at address localhost/127.0.0.1[1527]
Logs generated in /home/user1/snappyData/work/localhost-server-1/snappyserver.log
SnappyData Server pid: 9519 status: running
  Distributed system now has 2 members.
  Other members: localhost(9368:locator)<v0>:16944
Starting SnappyData Leader using locators for peer discovery: user1-laptop[10334]
Logs generated in /home/user1/snappyData/work/localhost-lead-1/snappyleader.log
SnappyData Leader pid: 9699 status: running
  Distributed system now has 3 members.
  Other members: localhost(9368:locator)<v0>:16944, 192.168.63.1(9519:datastore)<v1>:46966
</code>

检查状态：您可以使用以下命令检查正在运行的集群的状态：

<code>
$ sbin/snappy-status-all.sh
SnappyData Locator pid: 9368 status: running
SnappyData Server pid: 9519 status: running
  Distributed system now has 2 members.
  Other members: localhost(9368:locator)<v0>:16944
SnappyData Leader pid: 9699 status: running
  Distributed system now has 3 members.
  Other members: localhost(9368:locator)<v0>:16944, 192.168.63.1(9519:datastore)<v1>:46966

</code>

您可以通过http://<leadHostname>:5050在浏览器中打开来检查SnappyData UI。使用Snappy SQL shell连接到集群并执行各种SQL操作。

关闭群集：您可以使用以下命令关闭群集：

<code>
$ sbin/snappy-stop-all.sh
The SnappyData Leader has stopped.
The SnappyData Server has stopped.
The SnappyData Locator has stopped.
</code> 

### 在多个主机上启动SnappyData集群

在多台主机上启动群集：

1.在多个节点上运行SnappyData的最简单的方法是在所有节点上使用NFS等共享文件系统。
您还可以在集群的每个节点上都安装相关产品。如果所有节点都具有NFS访问权限，在任何一个节点上都安装一份SnappyData。
2.使用conf文件夹中提供的模板创建配置文件。复制模板文件servers.template，locators.template，leads.template，并将其重命名为servers，locators，leads等。 
编辑文件以包含主机名，然后启动server，locator和lead。有关属性的更多信息，请参阅配置部分。

注意：建议您在集群中的所有主机上设置无密码的SSH。有关安装和集群配置的更多详细信息，请参阅文档。


### 如何在群集中运行Spark代码

一个Spark程序，运行在SnappyData集群中，作为一个SnappyData Job来实现。

实现作业：SnappyData作业是实现SnappySQLJob或SnappyStreamingJob（用于流应用程序）trait的类或对象。在runSnappyJob作业的方法中，您可以使用传递给它的SnappySession对象实例来实现Spark程序的逻辑。您可以执行所有操作，如创建表，加载数据，使用SnappySession执行查询。

任何Spark API都可以由SnappyJob调用。

<code>
class CreatePartitionedRowTable extends SnappySQLJob { </p>
  /\** SnappyData uses this as an entry point to execute Snappy jobs. **/</p>
  def runSnappyJob(sc: SnappySession, jobConfig: Config): Any</p>

  /\**
  SnappyData calls this function to validate the job input and reject invalid job requests.
  You can implement custom validations here, for example, validating the configuration parameters
  **/ </p>
  def isValidJob(sc: SnappySession, config: Config): SnappyJobValidation
}
</code>

依赖关系：要编译的 Job，请使用Maven / SBT依赖关系来获取最新发布的SnappyData版本。

示例：Maven依赖关系：

<code>
<\!-- https://mvnrepository.com/artifact/io.snappydata/snappydata-cluster_2.11 -->
<dependency>
    <groupId>io.snappydata</groupId>
    <artifactId>snappydata-cluster_2.11</artifactId>
    <version>0.8</version>
</dependency>
</code>

示例：SBT依赖：
<code>
// https://mvnrepository.com/artifact/io.snappydata/snappydata-cluster_2.11
libraryDependencies += "io.snappydata" % "snappydata-cluster_2.11" % "0.8"
</code>

### 









































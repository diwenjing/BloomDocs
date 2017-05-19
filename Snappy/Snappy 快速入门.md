# 5分钟以内入门 #

欢迎使用入门课！
我们为您开始使用SnappyData提供多种选择。根据您的偏好，您可以尝试以下任一选项：

- **选项1：Spark分布开始**
- **选项2：入门使用Spark Scala API**
- **选项3：比Spark 2.0.2快20X快速缓存**
- **选项4：入门使用SQL**
- **选项5：通过在内部安装SnappyData进行入门**
- **选项6：AWS入门**
- **选项7：Docker Image入门**

> 注意：将来的版本中将提供对Microsoft Azure的支持

# 选项1：Spark分布开始 #

如果您是Spark开发人员，并且已经使用Spark 2.0.0 2.0.1或2.0.2，则使用SnappyData的最快方法是将SnappyData添加为依赖关系。例如，使用Spark Shell的“package”选项。

本节包含说明和示例，您可以在5分钟以内尝试使用SnappyData。我们鼓励您也尝试快速性能基准，以了解Spark的本机缓存性能的20倍优势。

**打开命令终端，并转到spark安装目录的位置：**

	$ cd <Spark_Install_dir>
	# Create a directory for SnappyData artifacts
	$ mkdir quickstartdatadir
	$ ./bin/spark-shell --conf spark.snappydata.store.sys-disk-dir=quickstartdatadir --conf spark.snappydata.store.log-file=quickstartdatadir/quickstart.log --packages "SnappyDataInc:snappydata:0.8-s_2.11"
这将打开一个Spark Shell并将相关的SnappyData文件下载到本地计算机。根据您的网络连接速度，下载文件可能需要一些时间。所有SnappyData元数据以及持久数据都存储在quickstartdatadir **目录中**。

在本文中，我们假设您熟悉Spark或SQL（不一定同时使用）。我们展示基本的数据库功能，如使用Columnar和面向行的表，查询和更新这些表。

SnappyData中的表格具有许多操作功能，如磁盘持久性，HA冗余，驱逐等。有关详细信息，请参阅详**细的文档**。

虽然SnappyData支持Scala，Java，Python，SQL API，但您可以根据自己的喜好选择使用Scala API或SQL。

# 选项2：入门使用Spark Scala API #

**创建SnappySession**：SnappySession扩展SparkSession，以便您可以更改数据，获得更高的性能等。

	scala> val snappy = new org.apache.spark.sql.SnappySession(spark.sparkContext)
	//Import snappy extensions
	scala> import snappy.implicits._

**使用Spark的API创建一个小数据集：**

	scala> val ds = Seq((1,"a"), (2, "b"), (3, "c")).toDS()

**定义表的模式：**

	scala>  import org.apache.spark.sql.types._
	scala>  val tableSchema = StructType(Array(StructField("CustKey", IntegerType, false),
	          StructField("CustName", StringType, false)))

**使用简单模式[String，Int]和默认选项创建列表：**有关详细选项，请参阅“ 行和列表”部分。

	//Column tables manage data is columnar form and offer superier performance for analytic class queries.
	scala>  snappy.createTable(tableName = "colTable",
	          provider = "column", // Create a SnappyData Column table
	          schema = tableSchema,
	          options = Map.empty[String, String], // Map for options.
	          allowExisting = false)

SnappyData（SnappySession）扩展了SparkSession，所以你可以简单地使用所有的Spark的API。

**将创建的DataSet插入列表“colTable”：**

	scala>  ds.write.insertInto("colTable")
	// Check the total row count.
	scala>  snappy.table("colTable").count

**使用Spark的API创建行对象并将行插入到表中：**与Spark DataFrames不同，SnappyData列表是可变的。您可以将新行插入列表。

	// Insert a new record
	scala>  import org.apache.spark.sql.Row
	scala>  snappy.insert("colTable", Row(10, "f"))
	// Check the total row count after inserting the row.
	scala>  snappy.table("colTable").count

**使用简单模式[String，Int]和默认选项创建“行”表：**有关详细选项，请参阅“ 行和列表”部分。

	//Row formatted tables are better when datasets constantly change or access is selective (like based on a key).
	scala>  snappy.createTable(tableName = "rowTable",
	          provider = "row",
	          schema = tableSchema,
	          options = Map.empty[String, String],
	          allowExisting = false)

**将创建的DataSet插入到行表“rowTable”中：**

	scala>  ds.write.insertInto("rowTable")
	//Check the row count
	scala>  snappy.table("rowTable").count

**插入新记录：**

	scala>  snappy.insert("rowTable", Row(4, "d"))
	//Check the row count now
	scala>  snappy.table("rowTable").count

**更改行表中的一些数据：**

	//Updating a row for customer with custKey = 1
	scala>  snappy.update(tableName = "rowTable", filterExpr = "CUSTKEY=1",
	                newColumnValues = Row("d"), updateColumns = "CUSTNAME")
	
	scala>  snappy.table("rowTable").orderBy("CUSTKEY").show
	
	//Delete the row for customer with custKey = 1
	scala>  snappy.delete(tableName = "rowTable", filterExpr = "CUSTKEY=1")
	
	//Drop the existing tables
	scala>  snappy.dropTable("rowTable", ifExists = true)
	scala>  snappy.dropTable("colTable", ifExists = true)

# 选项3：比Spark 2.0.2快20X #

在这里，我们将引导您完成一个简单的基准测试，将SnappyData与Spark 2.0.2的性能进行比较。我们将数百万行加载到缓存的Spark DataFrame中，运行一些测量其性能的分析查询，然后使用SnappyData的列表重复相同的操作。

> 注意：建议您至少要为此测试保留4GB的RAM。

**使用以下提到的任何选项启动Spark Shell：**

**如果您使用自己的Spark安装（2.0.0,2.0.1或2.0.2）：**

	# Create a directory for SnappyData artifacts
	$ mkdir quickstartdatadir
	$ ./bin/spark-shell --driver-memory=4g --conf spark.snappydata.store.sys-disk-dir=quickstartdatadir --conf spark.snappydata.store.log-file=quickstartdatadir/quickstart.log --packages "SnappyDataInc:snappydata:0.8-s_2.11" --driver-java-options="-XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+CMSClassUnloadingEnabled -XX:MaxNewSize=1g"

**如果您已经下载了SnappyData：**

	# Create a directory for SnappyData artifacts
	$ mkdir quickstartdatadir
	$ ./bin/spark-shell --driver-memory=4g --conf spark.snappydata.store.sys-disk-dir=quickstartdatadir --conf spark.snappydata.store.log-file=quickstartdatadir/quickstart.log --driver-java-options="-XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+CMSClassUnloadingEnabled -XX:MaxNewSize=1g"

**如果你使用Docker：**

	$ docker run -it -p 5050:5050 snappydatainc/snappydata bin/spark-shell --driver-memory=4g --driver-java-options="-XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+CMSClassUnloadingEnabled -XX:MaxNewSize=1g"

## 获取演出号码 ##

确保您在Spark Shell中，然后按照以下说明获取性能数字。

**定义一个功能“基准”**，它告诉我们在进行初始预热后平均运行查询的时间。

scala>  def benchmark(name: String, times: Int = 10, warmups: Int = 6)(f: => Unit) {
          for (i <- 1 to warmups) {
            f
          }
          val startTime = System.nanoTime
          for (i <- 1 to times) {
            f
          }
          val endTime = System.nanoTime
          println(s"Average time taken in $name for $times runs: " +
            (endTime - startTime).toDouble / (times * 1000000.0) + " millis")
        }
**使用Spark的范围方法创建DataFrame和临时表**：在Spark中缓存它以获得最佳性能。这创建了一个1亿条记录的DataFrame。您可以根据您的内存可用性更改行数。

	scala>  var testDF = spark.range(100000000).selectExpr("id", "concat('sym', cast((id % 100) as STRING)) as sym")
	scala>  testDF.cache
	scala>  testDF.createOrReplaceTempView("sparkCacheTable")

**运行查询并检查性能**：查询使用一个字段的平均值，而没有任何where子句。这样可以确保在扫描时触及所有记录。

	scala>  benchmark("Spark perf") {spark.sql("select sym, avg(id) from sparkCacheTable group by sym").collect()}

**清理JVM**：这样可以确保所有的Spark内存工具被清理掉。

	scala>  testDF.unpersist()
	scala>  System.gc()
	scala>  System.runFinalization()

**创建SnappyContex**：

	scala>  val snappy = new org.apache.spark.sql.SnappySession(spark.sparkContext)

**创建类似的1亿条记录DataFrame**：

	scala>  testDF = snappy.range(100000000).selectExpr("id", "concat('sym', cast((id % 100) as varchar(10))) as sym")

**创建表**：

	scala>  snappy.sql("drop table if exists snappyTable")
	scala>  snappy.sql("create table snappyTable (id bigint not null, sym varchar(10) not null) using column")

**将创建的DataFrame插入表并测量其性能**：

	scala>  benchmark("Snappy insert perf", 1, 0) {testDF.write.insertInto("snappyTable") }

**现在让我们运行与Spark DataFrame相同的基准：**

	scala>  benchmark("Snappy perf") {snappy.sql("select sym, avg(id) from snappyTable group by sym").collect()}
	scala> :q // Quit the Spark Shell

> 我们已经在系统中测试了4个CPU（Intel（R）Core（TM）i7-5600U CPU @ 2.60GHz）和16GiB系统内存的基准代码。在AWS t2.xlarge（可变ECU，4个vCPU，2.4 GHz，Intel Xeon系列，16 GiB内存，仅限EBS）实例中，SnappyData的速度比Spark 2.0.2快16到18倍。

# 选项4：入门使用SQL #

我们使用Spark API使用Spark API来说明SQL。您还可以使用任何SQL客户端工具（例如，Snappy SQL Shell）。有关示例，请参阅“操作方法”部分。

**使用简单模式[Int，String]和默认选项创建列表。** 有关选项的详细信息，请参阅“ **行和列表**”部分。

	scala>  snappy.sql("create table colTable(CustKey Integer, CustName String) using column options()")
	//Insert couple of records to the column table
	scala>  snappy.sql("insert into colTable values(1, 'a')")
	scala>  snappy.sql("insert into colTable values(2, 'b')")
	scala>  snappy.sql("insert into colTable values(3, '3')")
	// Check the total row count now
	scala>  snappy.sql("select count(*) from colTable").show

**用主键创建一个行表：**

	//Row formatted tables are better when datasets constantly change or access is selective (like based on a key).
	scala>  snappy.sql("create table rowTable(CustKey Integer NOT NULL PRIMARY KEY, " +
	            "CustName String) using row options()")

如果您使用标准SQL（即没有“row options”子句）创建表，则会创建一个复制的Row表。

	//Insert couple of records to the row table
	scala>  snappy.sql("insert into rowTable values(1, 'a')")
	scala>  snappy.sql("insert into rowTable values(2, 'b')")
	scala>  snappy.sql("insert into rowTable values(3, '3')")
	//Update some rows
	scala>  snappy.sql("update rowTable set CustName='d' where custkey = 1")
	scala>  snappy.sql("select * from rowTable order by custkey").show
	//Drop the existing tables
	scala>  snappy.sql("drop table if exists rowTable ")
	scala>  snappy.sql("drop table if exists colTable ")
	scala> :q //Quit the Spark Shell

现在我们已经看到了SnappyData表的基本工作，让我们运行基准代码来查看SnappyData的性能，并将其与Spark的本机缓存性能进行比较。


# 选项5：通过在内部安装SnappyData进行入门 #

从SnappyData发行页面页面中下载SnappyData的最新版本，其中列出了SnappyData的最新版本和以前的版本。

	$ tar -xzf snappydata-0.8-bin.tar.gz
	$ cd snappydata-0.8-bin/
	# Create a directory for SnappyData artifacts
	$ mkdir quickstartdatadir
	$./bin/spark-shell --conf spark.snappydata.store.sys-disk-dir=quickstartdatadir --conf spark.snappydata.store.log-file=quickstartdatadir/quickstart.log

它打开一个Spark Shell。所有SnappyData元数据以及持久数据都存储在quickstartdatadir 目录中。按照这里提到的步骤


**选项6：AWS入门**

您可以通过AWS CloudFormation快速创建单个主机SnappyData集群（即单个EC2实例中的一个引导节点，一个数据节点和定位器）。

## 先决条件 ##

在你开始之前：

- 确保您有一个现有的AWS帐户，具有从CloudFormation启动EC2资源所需的权限
- 使用您的AWS帐户特定URL登录AWS控制台。这样可确保帐户特定的URL作为Cookie存储在浏览器中，然后将其重定向到相应的AWS URL以便后续登录。
- 在要启动SnappyData Cloud群集的区域中创建EC2密钥对

要从EC2启动群集，请点击[此处](https://console.aws.amazon.com/cloudformation/home#/stacks/new?templateURL=https://zeppelindemo.s3.amazonaws.com/quickstart/snappydata-quickstart.json)，并按照以下说明进行操作。

1. 将显示AWS登录屏幕。输入您的AWS登录凭据。
1. 将显示“**选择模板**”页面。预先填充模板的URL（JSON格式）。单击**下一步**继续。

	> 注意：您被放置在默认区域。您可以继续选择所选区域，也可以在控制台中进行更改。 

	![](http://snappydatainc.github.io/snappydata/Images/cluster_selecttemplate.png)


1. 在“ **指定详细信息**”页面上，您可以：
	
	- 提供堆栈名称：输入堆栈的名称。堆栈名称必须只包含字母，数字，破折号，应以alpha字符开头。这是必填字段。
	- 选择实例类型：默认情况下，选择c4.2xlarge实例（8个CPU内核和15 GB RAM）。这是运行此快速启动的建议实例大小。
	- 选择KeyPairName：从可用的密钥对列表中选择一个密钥对。这是必填字段。
	- 搜索VPCID：从下拉列表中选择VPC ID。您的实例在此VPC中启动。这是必填字段。

	![](http://snappydatainc.github.io/snappydata/Images/cluster_specifydetails.png)
1. 单击**下一步**。
1. 在**选项**页面上，单击**下一步**继续使用提供的默认值。
1. 在“ **审阅**”页面上，验证详细信息，然后单击“ 创建”以创建堆栈。
	![](http://snappydatainc.github.io/snappydata/Images/cluster_createstack.png)

1. 下一页列出了现有的堆栈。单击**刷新**以查看更新的列表。选择堆栈以查看其状态。当集群启动时，堆栈的状态更改为**CREATE_COMPLETE**。此过程可能需要4-5分钟才能完成。

	![](http://snappydatainc.github.io/snappydata/Images/cluster_refresh.png)


	> 注意：如果堆栈的状态显示为ROLLBACK_IN_PROGRESS或DELETE_COMPLETE，则堆栈创建可能已失败。失败的一些常见原因是：

	- **权限不足**：验证您是否拥有在AWS上创建堆栈（和其他AWS资源）所需的权限。
	- **密钥对无效**：验证在iSight CloudBuilder创建步骤中选择的区域中是否存在EC2密钥对。
	- **超出限制**：验证您是否没有超出您的资源限制。例如，如果超过了分配的Amazon EC2实例的限制，则资源创建失败，并报告错误。

1. 您的群集正在运行。您可以使用Apache Zeppelin进行探索，该技术为基于Web的笔记本电脑提供数据探索。Apache Zeppelin服务器已经在您的实例上启动。只需从**“ 输出 ” （Outputs）**选项卡中点击其链接（URL）即可。
	![](http://snappydatainc.github.io/snappydata/Images/cluster_links.png)

有关更多信息，请参阅Apache Zeppelin部分或参考Apache Zeppelin文档。

> 注意：
> 
> - 将来版本将支持通过CloudFormation在AWS上设置的多节点集群。但是，用户可以使用EC2脚本进行设置。
> - 要停止为实例收取费用，您可以在完成使用群集操作后终止实例或删除堆栈。但是，终止后，您无法连接到或重新启动一个实例。

# 选项7：Docker Image入门 #

SnappyData配有一个带Docker的预配置容器。容器具有SnappyData的二进制文件。这使您能够轻松地尝试快速启动程序和更多，与SnappyData。

本节假设您已经安装了Docker，并且它已正确配置。有关更多详细信息，请参阅[Docker文档](http://docs.docker.com/installation/)。

**验证Docker是否已安装**：在命令提示符下运行命令：

	$ docker run hello-world

> 注意：确保Docker容器可以访问机器上至少4GB的RAM

**获取Docker映像**：在命令提示符下，键入以下命令以获取docker映像。这将启动容器并带您进入Spark Shell。

	$  docker run -it -p 5050:5050 snappydatainc/snappydata bin/spark-shell
它开始将最新的图像文件下载到本地机器。根据您的网络连接，可能需要一些时间。一旦您在Spark Shell中出现“$ scala>”提示，您可以按照[此处介绍](http://snappydatainc.github.io/snappydata/quickstart/#Start_quickStart)的步骤进行操作

有关SnappyData docker镜像的更多详细信息，请参阅[Snappy Cloud Tools](https://github.com/SnappyDataInc/snappy-cloud-tools/tree/master/docker)

### *更多信息* ###

有关常见操作的更多示例，可以参考“[操作方法](http://snappydatainc.github.io/snappydata/howto/)”部分。

如果您有任何问题或疑问，可以通过我们的[社区渠道](http://snappydatainc.github.io/snappydata/techsupport/#community)与我们联系。
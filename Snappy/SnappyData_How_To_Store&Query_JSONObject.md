## 如何存储和查询JSON对象
您可以在SnappyData表中插入JSON数据，并对表执行查询。

代码示例：将JSON数据从JSON文件加载到列表(column table)中，并执行查询

下面给出的代码片段将JSON数据从JSON文件加载到列表(column table)中，并针对它执行查询。JSON示例的源代码位于WorkingWithJson.scala。创建SnappySession之后，我们使用Spark API读取JSON文件并将其加载到SnappyData表中。

获取SnappySession：

<code>
	
	val spark: SparkSession = SparkSession
        .builder
        .appName("WorkingWithJson")
        .master("local[*]")
        .getOrCreate

    val snSession = new SnappySession(spark.sparkContext)
</code>

从JSON文件创建一个DataFrame：

<code>
	
	val some_people_path = s"quickstart/src/main/resources/some_people.json"
    // Read a JSON file using Spark API
    val people = snSession.read.json(some_people_path)
    people.printSchema()
</code>

创建SnappyData表并使用DataFrame插入JSON数据：

<code>
 	
	//Drop the table if it exists
    snSession.dropTable("people", ifExists = true)
	
	//Create a columnar table with the Json DataFrame schema
    snSession.createTable(tableName = "people",
      provider = "column",
      schema = people.schema,
      options = Map.empty[String,String],
      allowExisting = false)

    // Write the created DataFrame to the columnar table
    people.write.insertInto("people")
</code>

从第二个JSON文件追加更多数据：

<code>
	
	//Append more people to the column table
    val more_people_path = s"quickstart/src/main/resources/more_people.json"

    //Explicitly passing schema to handle record level field mismatch
    // e.g. some records have "district" field while some do not.
    val morePeople = snSession.read.schema(people.schema).json(more_people_path)
    morePeople.write.insertInto("people")

    //print schema of the table
    println("Print Schema of the table\n################")
    println(snSession.table("people").schema)
</code>

执行查询并返回结果

<code>
	
	// Query it like any other table
    val nameAndAddress = snSession.sql("SELECT " +
        "name, " +
        "address.city, " +
        "address.state, " +
        "address.district, " +
        "address.lane " +
        "FROM people")
	nameAndAddress.toJSON.show()
</code>




### 如何存储和查询对象

您可以使用域对象将数据加载到SnappyData表中，并通过对表执行查询来选择数据。

代码示例：将Person对象插入到列表中

下面的代码片段将Person对象插入列表。此示例的源代码位于WorkingWithObjects.scala。创建SnappySession后，使用Spark API插入Person对象并加载到SnappyData表中。

获取SnappySession：

<code>
 
	val spark: SparkSession = SparkSession
        .builder
        .appName("CreateReplicatedRowTable")
        .master("local[4]")
        .getOrCreate

    val snSession = new SnappySession(spark.sparkContext)
</code>

创建DataFrame对象：

<code>
    
    // Import the implicits for automatic conversion between Objects to DataSets.
    import snSession.implicits._

    // Create a Dataset using Spark APIs
    val people = Seq(Person("Tom", Address("Columbus", "Ohio"), Map("frnd1"-> "8998797979", "frnd2" -> "09878786886"))
      , Person("Ned", Address("San Diego", "California"), Map.empty[String,String])).toDS()

</code>

创建SnappyData表并在其中插入数据：

<code>

    //Drop the table if it exists.
    snSession.dropTable("Persons", ifExists = true)

    //Create a columnar table with a Struct to store Address
    snSession.sql("CREATE table Persons(name String, address Struct<city: String, state:String>, " +
         "emergencyContacts Map<String,String>) using column options()")

    // Write the created DataFrame to the columnar table.
    people.write.insertInto("Persons")

    //print schema of the table
    println("Print Schema of the table\n################")
    println(snSession.table("Persons").schema)


    // Append more people to the column table
    val morePeople = Seq(Person("Jon Snow", Address("Columbus", "Ohio"), Map.empty[String,String]),
      Person("Rob Stark", Address("San Diego", "California"), Map.empty[String,String]),
      Person("Michael", Address("Null", "California"), Map.empty[String,String])).toDS()

    morePeople.write.insertInto("Persons")
</code>

对表执行查询并返回结果：

<code>

	// Query it like any other table
    val nameAndAddress = snSession.sql("SELECT name, address, emergencyContacts FROM Persons")

    //Reconstruct the objects from obtained Row
    val allPersons = nameAndAddress.as[Person]
    //allPersons is a Spark Dataset of Person objects. 
    // Use of the Dataset APIs to transform, query this data set. 
</code>













































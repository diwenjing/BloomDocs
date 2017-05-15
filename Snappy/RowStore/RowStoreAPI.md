# RowStore API #
## CredentialInitializer ##
CredentialInitializer指定获取对等体凭据的机制。

在安全模式下运行时，CredentialInitializer是对等体必需的，并且在服务器或定位器上配置了UserAuthenticator的自定义实现。

要在服务器或定位器上配置自定义用户验证器，请在auth-provider属性中包含UserAuthenticator实现的完全限定类名称。

## UserAuthenticator ##
UserAuthenticator界面提供了用于验证用户连接到数据库的凭据的操作。

用户认证方案可以通过该接口实现，并在启动时注册。

如果应用程序需要自己的认证方案，则可以实现该接口并注册为RowStore在向系统发送连接请求时应该调用的认证方案。

一个典型的例子是使用此接口实现LDAP，Sun NIS +或甚至Windows用户域的用户身份验证。


> 注意：您可以指定其他连接属性，可以在数据库连接URL和/或Jdbc连接上的Properties对象中指定。这些属性的值可以通过（专用）认证方案在运行时检索，以进一步帮助用户认证，除非用户，密码和数据库名称需要信息。


## 程序实现接口 ##
这些接口与过程实现本身相关。

- **ProcedureExecutionContext** ProcedureExecutionContext是一个上下文对象，可以通过将其声明为静态方法实现中的参数，将其传递给PROCEDURE。

- **OutgoingResultSet** OutgoingResultSet是一个通过将行添加为List <Object>来构造输出结果集的接口。

### ProcedureExecutionContext ###
ProcedureExecutionContext是一个上下文对象，可以通过在静态方法实现中将其声明为参数，将其传递给PROCEDURE。

这些方法在这个上下文界面中提供：

|方法|描述|
|:--|:--|
|用getFilter（）|将这个执行的where子句作为String返回，否则返回null。|
|getTableName时（）|如果此过程使用ON TABLE子句执行，则以“schemaName.tableName”格式返回表名，否则返回null。|
|getColocatedTableNames（）|以“schemaName.tableName”的格式返回此服务器中的一组共处理表。如果没有使用ON TABLE子句调用此过程，则此方法返回null。|
|getProcedureName（）|返回此过程的名称。|
|的getConnection（）|可以由过程用于执行SQL的“嵌套”JDBC连接。|
|isPossibleDuplicate（）|如果这是在分布式系统成员发生故障后发生的重新尝试，则返回true。对于正在执行写入操作的某些过程实现，如果这是对该数据可能的重复调用，则可能需要进行特殊处理。|
|isPartitioned（String tableName）|如果指定的表是分区表，则返回true，如果是复制表，则返回false。|
|getOutgoingResultSet（int resultSetNumber）|为输出结果集创建并返回一个空容器。resultSetNumber是将此结果集分配为为此过程声明的结果集之一的索引。|

### OutgoingResultSet ###
OutgoingResultSet是一个用于通过将行添加为List <Object>来构造输出结果集的接口。

该界面中的方法是：

|方法|描述|
|:--|:--|
|addColumn（String name）|指定此结果集的下一列的名称。应该在调用addRow之前指定所有列的名称，除非需要默认名称。默认列名将用于通过调用此方法未命名的任何列：“c1”，“c2”等|
|addRow（List <Object> row）|向此结果集添加一行，以将其发送到ResultCollector。在调用此方法之前，列描述应添加addColumn，否则将基于添加有名为“c1”，“c2”等的列的第一行推断默认列描述。|
|endResults（）|调用此方法让结果集知道没有更多的行要添加。|

### 例 ###
	--The procedure retrieveDynamicResultsWithOutgoingResultSet a stored
	--procedure which returns 0, 1, 2, 3 or 4 ResultSet.
	
	public static void retrieveDynamicResultsWithOutgoingResultSet(int number,
	          ResultSet[] rs1, ResultSet[] rs2, ResultSet[] rs3,
	          ResultSet[] rs4, ProcedureExecutionContext pec) 
	          throws SQLException {
	
	      Connection c = pec.getConnection();
	
	      if (number > 0) {
	        rs1[0] = c.createStatement().executeQuery("VALUES(1)");
	      }
	      if (number > 1) {
	        OutgoingResultSet ors=pec.getOutgoingResultSet(2);
	        for(int i=0; i<10; ++i) {
	          List<Object> row=new ArrayList<Object>();
	          row.add(new Integer(i));
	          row.add("String"+ i);        
	          ors.addRow(row);
	        }
	        ors.endResults();      
	      }
	      if (number > 2) {
	        rs3[0] = c.createStatement().executeQuery("VALUES(1)");
	      }
	      if (number > 3) {
	        rs4[0] = c.createStatement().executeQuery("VALUES(1)");
	      }
	  }


## 过程结果处理器接口 ##
过程结果处理器接口与定制结果处理器相关。

- **ProcedureResultProcessor** ProcedureResultProcessor从多个服务器整理结果和OUT参数。

- **ProcedureProcessorContext** ProcedureProcessorContext是在其init方法中传递给ProcedureResultProcessor的Context对象。

- **IncomingResultSet** IncomingResultSet是从传入结果集中将列作为List <Object>检索的接口。

### ProcedureResultProcessor ###
ProcedureResultProcessor从多个服务器中整理结果和OUT参数。

可以实现自定义的ProcedureResultProcessor，以与默认行为不同的方式对来自多个服务器的结果和OUT参数进行整理。它有这些方法：

|方法|描述|
|:--|:--|
|getOutParameters（）|作为Object []向客户端提供此过程的输出参数。|
|getNextResultRow（int resultSetNumber）|为结果集编号resultSetNumber提供下一行。处理器应该对输入的数据进行任何处理，以提供下一行。|
|关（）|当此语句关闭时由RowStore调用。|

### 例 ###
	–  A custom Processor Example
	–  Example code showing how to implement a Procedure and a custom Collector that does a MergeSort.
	
	   CREATE PROCEDURE MergeSort () 
	   LANGUAGE JAVA   PARAMETER STYLE JAVA 
	   READS SQL DATA   DYNAMIC RESULT SETS 1 
	   EXTERNAL NAME 'examples.MergeSortProcedure.mergeSort' 
	
	– MergeSortProcedure class
	
	package examples; 
	import com.pivotal.snappydata.*; 
	import java.sql.*; 
	
	public class MergeSortProcedure { 
	  static final String LOCAL = "<local>"; 
	  public static void mergeSort(ResultSet[] outResults, 
	                                ProcedureExecutionContext context) 
	  throws SQLException { 
	    String queryString = LOCAL 
	                         + "SELECT * FROM " 
	                         + context.getTableName(); 
	    Connection cxn = context.getConnection(); 
	    Statement stmt = cxn.createStatement(); 
	    ResultSet rs = stmt.executeQuery(queryString); 
	    outResults[0] = rs; 
	  } 
	} 
	
	MergeSortProcessor class
	
	package examples; 
	import com.pivotal.snappydata.*; 
	import java.sql.*; 
	import java.util.*; 
	
	public class MergeSortProcessor implements ProcedureResultProcessor { 
	
	  private ProcedureProcessorContext context; 
	
	  public void init(ProcedureProcessorContext context) { 
	    this.context = context; 
	  } 
	
	  public Object[] getOutParameters() { 
	    throw new AssertionError("this procedure has no out parameters"); 
	  } 
	
	  public Object[] getNextResultRow(int resultSetNumber) 
	    throws InterruptedException { 
	    // this procedure deals with only result set number 1 
	    IncomingResultSet[] inSets = context.getIncomingResultSets(1); 
	    Object[] lesserRow = null; 
	    Comparator cmp = getComparator(); 
	    IncomingResultSet setWithLeastRow = null; 
	    for (IncomingResultSet inSet : inSets) { 
	      Object[] nextRow = inSet.waitPeekRow(); 
	        // no more rows in this incoming results 
	        continue; 
	      } 
	      // find the least row so far 
	        lesserRow = nextRow; 
	        setWithLeastRow = inSet; 
	      } 
	    } 
	    if (setWithLeastRow != null) { 
	      // consume the lesserRow by removing lesserRow from the incoming result set 
	      Object[] takeRow = setWithLeastRow.takeRow(); 
	    } 
	    // if lesserRow is null, then there are no more rows in any incoming results 
	    return lesserRow; 
	  } 
	
	  public boolean getMoreResults(int nextResultSetNumber) { 
	    return false; // only one result set 
	  } 
	  public void close() { 
	    this.context = null; 
	  } 
	} 
	

### ProcedureProcessorContext ###
ProcedureProcessorContext是在其init方法中传递给ProcedureResultProcessor的Context对象。

界面有这些方法：

|方法|描述|
|:--|:--|
|getIncomingResultSets（int resultSetNumber）|获取给定结果集编号的传入结果集的数组，其中每个数组元素是由某些服务器上执行的过程提供的结果集。数组的大小等于执行此过程的服务器数。|
|getIncomingOutParameters（）|获取执行过程提供的out参数的传入结果集数组。数组的大小等于执行此过程的服务器数。这些结果集中的每一个将只有一个“行”，一个Object []对应于out参数。|
|用getFilter（）|返回此执行的whereClause，如果没有，则返回null。|
|getTableName时（）|如果此过程使用ON TABLE子句执行，则以“schemaName.tableName”格式返回表名，否则返回null。|
|getColocatedTableNames（）|以“schemaName.tableName”的格式返回此服务器中的一组共处理表。如果没有使用ON TABLE子句调用此过程，则此方法返回null。|
|getProcedureName（）|返回此过程的名称。|
|isPossibleDuplicate（）|如果这是在分布式系统成员发生故障后发生的重新尝试，则返回true。对于正在执行写入操作的某些过程实现，如果这是对该数据可能的重复调用，则可能需要进行特殊处理。|
|isPartitioned（String tableName）|如果指定的表是分区表，则返回true，如果是复制表，则返回false。|

### IncomingResultSet ###
IncomingResultSet是一个从传入结果集中将列作为List <Object>检索的接口。

此接口使用类似于BlockingQueue的语义。它有这些方法：

|方法|描述|
|:--|:--|
|takeRow（）|检索并将此结果集中的头行删除为List <Object>，如有必要，等待元素可用，或者如果此结果集中没有更多行，则返回END_OF_RESULTS。|
|pollRow（）|检索并删除此结果集的头行，如果此结果集中没有行当前可用，则返回null，如果此结果集中没有更多行，则返回END_OF_RESULTS。|
|pollRow（long timeout，TimeUnit单位）|超时参数超载。TimeUnit参数用于解释超时参数。|
|peekRow（）|检索但不删除此结果集的头行，如果此结果集中当前没有行，则返回null，如果此结果集中没有更多行，则返回END_OF_RESULTS。|
|peekRow（long timeout，TimeUnit unit）|超时参数超载。|
|waitPeekRow（）|检索但不删除此结果集的头行，如有必要，等待元素可用，或者如果此结果集中没有更多行，则返回END_OF_RESULTS。|
|的getMetaData（）|返回有关此结果集中的列的元数据信息。可能会阻塞，直到元数据可用。|


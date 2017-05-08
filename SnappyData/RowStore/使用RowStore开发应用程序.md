#使用RowStore开发应用程序
###使用RowStore开发应用程序解释了使用RowStore API和SQL的RowStore实现编程JDBC，ODBC和ADO.NET应用程序的主要概念。它描述了如何在一个Java应用程序中嵌入一个RowStore实例，以及如何作为一个客户端进行连接。本指南还描述了事务和非事务性行为，并解释了如何创建数据感知存储过程。
<br/>
###使用FabricServer接口启动RowStore服务器 FabricServer接口提供了一种在现有Java应用程序中启动嵌入式RowStore服务器进程的简单方法。<br/>


FabricServer接口提供了一种在现有Java应用程序中启动嵌入式RowStore服务器进程的简单方法。<br/>

当您希望为嵌入式RowStore成员提供瘦客户端连接时，通常使用FabricServer接口。通过FabricServer接口，您可以启动多个网络服务来监听不同地址和端口组合的客户端。在启动网络服务之前，使用FabricServer接口还可以在RowStore服务器成员中初始化资源，并使该成员可用于客户端连接。<br/>

###注意：尽管FabricServer接口支持启动嵌入式定位器服务，但生产系统应使用独立定位器作为最佳做法。请参阅开始和停止定位器。**程序**
<br/>
使用FabricServer接口启动RowStore服务器：<br/>

使用FabricServiceManager工厂类获取FabricServer的单例实例。例如：<br/>
<code>
FabricServer server = FabricServiceManager.getFabricServerInstance();
</code>
<br/>
创建一个Java属性对象，并添加您要在启动服务器时配置的所有引导属性定义。例如：<br/>
<code>
Properties bootProps = new Properties();<br/>
bootProps.setProperty("mcast-port", "12444");<br/>
</code>
或者，您可以将属性定义为系统属性（使用-D选项传递到JVM），在属性文件中配置它们，或依赖于默认属性值。配置属性提供更多信息。<br/>
使用FabricServer.start（）方法启动服务器使用您的属性对象：<br/>
<code>server.start(p);</code><br/>
注意： RowStore在任何给定时间仅支持JVM中的单个FabricServer实例。如果使用相同的属性多次调用start（），则在后续调用中不会采取任何操作。如果您使用不同的属性多次调用start（），则默认情况下，现有的FabricServer实例首先被停止，然后重新启动新的属性。您可以选择使用“start（Properties bootProperties，boolean ignoreIfStarted）”方法使用“true”布尔值来重用上一个实例而不是重新启动它。有关更多信息，请参阅FabricServer JavaDoc。<br/>

要支持客户端连接，请使用startNetworkServer（）方法在唯一的客户端和端口组合上启动网络服务。您可以指定主机和端口号作为方法的参数。您可以在使用该方法传递的“属性”对象中指定其他网络服务器属性。例如，仅指定没有附加属性的地址和端口：<br/>
<code>
server.startNetworkServer("localhost", 1528, null);
</code>
<br/>
注意： RowStore网络服务器支持Derby服务器和管理指南中记录的Apache Derby网络属性。<br/>

根据需要开始附加的网络服务，以监听不同的地址和端口组合。<br/>
###启动网络服务器 客户端可以使用RowStore JDBC客户端驱动程序（“jdbc：snappydata：// <host>：<port>'形式的URL）连接到NetworkInterface。可以通过在FabricServer实例上调用startNetworkServer方法来获取网络侦听器。RowStore使用分布式关系数据库体系结构（DRDA）协议进行客户端 - 服务器通信。
###处理强制断开 如果一段时间内没有响应，或者网络分区将一个或多个成员分离成太小而不能用作分布式系统的组，则RowStore成员可能会与分布式系统强制断开连接。如果您使用FabricService接口启动RowStore，则可以使用回调方法在重新连接过程中执行操作，或者在必要时取消重新连接过程。
<br/>

###连接Java客户端和对等体 Java应用程序可以使用JDBC瘦客户端驱动程序连接到RowStore集群并执行SQL语句。或者，Java应用程序可以使用JDBC对等客户机驱动程序引导RowStore对等体进程，并作为集群的成员加入。
<br/>
两个驱动程序都使用JDBC的基本连接URL jdbc:snappydata:：
<br/>
jdbc: 是协议。<br/>
snappydata: 是子协议。<br/>
瘦客户机为分布式系统提供轻量级的JDBC连接。默认情况下，瘦客户机连接到现有的RowStore数据存储成员，该成员充当用于执行分布式事务的事务协调器。这提供了执行查询或DML语句的单跳或双跳访问，具体取决于数据位于系统中的位置。对于某些类型的查询，您可以为瘦客户端连接启用单跳访问，以提高查询性能。请参阅启用单跳数据访问。<br/>

瘦客户机JDBC驱动程序连接 瘦客户端驱动程序类包装在com.pivotal.snappydata.jdbc.ClientDriver中。除了基本的JDBC连接URL之外，还可以指定瘦客户机驱动程序连接到的RowStore服务器或定位器的主机和端口号。<br/>

###将RowStore配置为JDBC数据源 RowStore JDBC实现使您能够将分布式系统用作WebLogic Server等产品中的嵌入式JDBC数据源。<br/>

程序
在第三方产品中将RowStore设置为数据源时，请遵循以下一般步骤：<br/>

将gemfirexd-client.jar文件复制到存储产品的其他JDBC驱动程序的位置。<br/>
对于提供数据源模板的WebLogic Server等产品，如果没有明确支持RowStore，请选择“Apache Derby”，“User-defined”或“Other”作为数据库类型。<br/>
指定“snappydata”作为数据库名称。这代表一个RowStore分布式系统。（RowStore不包含Apache Derby或其他关系数据库系统中的多个数据库。）<br/>
对于主机名和端口，请指定RowStore定位器或RowStore服务器的主机名和端口组合。这是您从snappy-shell提示中用作客户端的主机名和端口组合。<br/>
对于数据库用户名和密码，如果您已在系统中启用身份验证（使用该-auth-provider属性），请输入有效的用户名和密码组合。<br/>
如果您尚未在RowStore中配置身份验证，请将“app”指定为用户名和密码值，或任何其他临时值。<br/>
注意：当您不提供数据库对象的模式名称时，RowStore将使用JDBC连接中指定的用户名作为模式名称。RowStore使用“APP”作为默认模式。如果您的系统未启用身份验证，您可以为用户名和密码指定“APP”，以保持与默认模式行为的一致性。<br/>

对于driver类，请指定： com.pivotal.snappydata.internal.jdbc.ClientConnectionPoolDataSource
您指定的JDBC URL必须以jdbc:snappydata://。删除任何模板属性，如create=true它们存在于URL或属性字段中。<br/>
在WebLogic等产品中，不能为嵌入式数据源指定JDBC连接属性，作为JDBC URL的一部分。而是使用属性字段并使用以下格式指定connectionAttributes：<br/>
<code>
connectionAttributes=attribute;attribute;...

</code>


<br/>
例如：

<code>
connectionAttributes=mcast-address=239.192.81.1;mcast-port=10334或connectionAttributes=locators=239.192.81.1;mcast-port=0
</code>


<br/>
另请参见用于EmbeddedDataSource的Apache Derby 文档。<br/>
一个进程一次只能连接到一个RowStore分布式系统。如果要连接到不同的RowStore分布式系统，请在重新连接不同的数据源之前关闭当前的嵌入式数据源。<br/>

###在RowStore中存储和加载JAR文件 SQL函数和过程可以使用的应用程序逻辑包括以JAR文件格式存储的Java类文件。在RowStore中存储应用程序JAR文件可简化应用程序部署，因为它减少了用户类路径问题的可能性。
<br/>

RowStore自动将安装的JAR文件类加载到类加载器中，以便您可以在RowStore应用程序和过程中使用它们。JAR类可用于RowStore分布式系统的所有成员，包括稍后加入系统的JAR类。<br/>

注意：本节中的许多主题已从Apache Derby文档源文件中进行了修改，并受Apache许可证的约束：“`根据一个或多个贡献者许可协议授予Apache Software Foundation（ASF）。有关版权所有权的其他信息，请参阅本工作分发的NOTICE文件。ASF根据Apache许可证版本2.0（“许可证”）向您授予此文件;除了符合许可证之外，您不得使用此文件。您可以通过http：//www.apache获取许可证的副本.org / licenses / LICENSE-2.0除非适用法律或书面同意，根据许可证分发的软件以“按现状”分发， 无明示或暗示的任何形式的担保或条件。有关许可证下的权限和限制的特定语言，请参阅许可证。“`
<br/>
类加载概述 通过安装一个或多个JAR文件，可以在RowStore中存储应用程序类，资源和过程实现。安装JAR文件后，应用程序可以访问类，而不必以特定的方式进行编码。<br/>
管理JAR文件的备用方法 RowStore还提供了可用于从客户端连接交互式安装和管理JAR文件的系统过程。请注意，与使用snappy-shell命令来管理JAR文件相比，这些过程具有一定的限制。<br/>

###了解数据一致性模型 假定单个分布式系统中的所有对等体都在同一数据中心内共同配置，并且可靠带宽和低延迟可访问。分布式系统中表数据的复制一直是急切和同步的。<br/>


 假设单个分布式系统中的所有对等体都被共同定位在相同的数据中心，并且可靠的带宽和低延迟可以访问。分布式系统中表数据的复制一直是急切和同步的。<br/>

您可以通过在这些系统中配置WAN网关发送器和接收器来支持两个分布式系统之间的复制。<br/>

数据一致性概念 没有事务（事务隔离设置为NONE），RowStore确保表更新的FIFO一致性。所有其他线程按照它们被发布的顺序看到由单个线程执行的写入，但是来自不同线程的写入可能被其他线程以不同的顺序看到。<br/>
非原子操作的失败方案 由于事务之外发生的多个操作没有原子性，因此不同的RowStore成员故障情况可能导致难以诊断的数据不一致。本节将介绍一些潜在的故障情况及其后果。<br/>
独立线程中的DML没有排序保证RowStore 仅保留单个执行线程应用于分布式系统（并排队到AsyncEventlisteners或远程WAN站点）的DML语句的顺序。来自多个线程的更新以先入先出（FIFO）顺序保存。否则，RowStore不提供“总订购”保证。<br/>
任何行的更新都是原子和隔离 当行更新时，成员可能会收到一个部分行。RowStore总是克隆现有行，应用更新（更改一个或多个字段），然后用更新的行原子替换行。这样可以确保读取或写入该行的所有并发线程始终保证与任何部分行更新的访问隔离。<br/>
批量更新的 原子性在应用批量更新（单个更新或插入多行的DML语句）之前，RowStore不会验证所有受影响的行的约束。该设计针对这种违规行为很少的应用进行了优化。<br/>





###在应用程序中使用分布式事务 事务是一组一个或多个SQL语句，构成可以提交或回滚的逻辑工作单元，并且在系统发生故障时将被恢复。RowStore分布式事务的独特设计允许线性缩放，而不会影响原子性，一致性，隔离和耐久性（ACID）属性。<br/>


事务是一组一个或多个SQL语句，构成可以提交或回滚的逻辑工作单元，并且在系统发生故障时将被恢复。RowStore分布式事务的独特设计允许线性缩放，而不会影响原子性，一致性，隔离和耐久性（ACID）属性。<br/>

###RowStore事务设计 RowStore实现乐观的事务。事务模型是高度优化的共处理数据，其中由事务更新的所有行都由单个成员拥有。<br/>
###RowStore分布式事务概述事务中的 所有语句都是原子的。事务与单个连接（和数据库）关联，不能跨连接。除了提供线性缩放之外，RowStore事务设计可最大限度地减少消息传递的需求，从而使短时间的事务更有效率。<br/>
###分布式事务的事件顺序 以下是事件提交序列之前和期间发生的事件的逐步描述。<br/>
###使用事务 的最佳做法为获得最佳效果，请注意使用RowStore事务的最佳做法。<br/>
###事务功能和限制 请注意此版本的RowStore中的事务行为和限制。<br/>






###编程数据感知过程和结果处理程序 过程是在数据库服务器中管理的应用程序函数调用或子例程。由于多个RowStore成员在分布式系统中一起操作，所以RowStore中的过程执行也可以并行化，以同时在多个成员上运行。在多个RowStore成员上同时执行的过程称为数据感知过程。<br/>

###注意： RowStore不支持在过程或函数正文中执行DDL语句。

数据感知过程使用扩展CALL语法与ON子句来指定过程执行的RowStore成员。当您调用过程时，RowStore语法提供了以下选项来并行化过程执行：<br/>

所有数据存储在RowStore集群中<br/>
数据存储的子集（属于一个或多个服务器组的成员）<br/>
托管表的所有数据存储成员<br/>
在表中托管数据子集的所有数据存储成员<br/>
RowStore将进程中的用户代码执行到数据所在的位置，这为共处理数据提供了非常低的延迟访问。（这与Hadoop中的map-reduce作业执行框架形成对比，Hadoop将数据从进程流传输到任务进程。）过程通常返回一个或多个结果集。RowStore将结果集流传输给一个可以对结果执行缩减步骤的协调成员（通常这涉及聚合，如map-reduce）。在RowStore中，还原步骤由结果处理器执行。RowStore提供了一个默认的结果处理器，您还可以开发自己的结果处理器实现来定制还原步骤。<br/>

以下部分描述了如何在RowStore中开发，配置和调用过程和结果处理器实现。<br/>

使用过程提供程序API RowStore提供了一个API来帮助您开发数据感知过程。ProcedureExecutionContext对象提供了有关用于过滤执行过程，分配数据以及有关执行过程的上下文的其他信息的表的信息。OutgoingResultSet接口使您能够通过向List对象添加行或列来构造结果集。<br/>
使用自定义结果处理器API 数据感知过程使用单独的结果处理器来合并来自多个RowStore成员的过程结果。当需要结果的基本级联时，可以使用默认的RowStore结果处理器。<br/>



###编程用户定义的类型 RowStore使您能够创建用户定义的类型。用户定义的类型是一个可序列化的Java类，其实例存储在列中。<br/>




 注意：本主题已从Apache Derby文档源中进行了修改，并受Apache许可证的约束： “`根据一个或多个贡献者许可协议授予Apache Software Foundation（ASF）。有关版权所有权的其他信息，请参阅本工作分发的NOTICE文件。ASF根据Apache许可证版本2.0（“许可证”）向您授予此文件;除非符合许可证，否则不得使用此文件，您可以获得许可证的副本<br/>
<code>

     http://www.apache.org/licenses/LICENSE-2.0<br/>

   Unless required by applicable law or agreed to in writing,
   software distributed under the License is distributed on an
   "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
   KIND, either express or implied.  See the License for the
   specific language governing permissions and limitations
   under the License.
<br/>
</p>
<br/>
User-defined functions can be defined on columns of user-defined types. You can also use these types can as argument types for stored procedures, as described in <a href="../server-side/data-aware-procedures.html#data-aware-procedures" class="xref" title="A procedure is an application function call or subroutine that is managed in the database server. Because multiple RowStore members operate together in a distributed system, procedure execution in RowStore can also be parallelized to run on multiple members, concurrently. A procedure that executes concurrently on multiple RowStore members is called a data-aware procedure.">Programming Data-Aware Procedures and Result Processors</a>.
<br/>
<p class="note"><strong>Note:</strong> You cannot register a custom .NET type as a user-defined type in RowStore. </p><br/>
The class of a user-defined type must implement the *java.io.Serializable* interface, and it must be declared to RowStore by means of a <a href="../../../reference/language_ref/rrefsqljcreatetype.html#rrefsqljcreatetype" class="xref noPageCitation">CREATE TYPE</a> statement. You can install user-defined type classes to RowStore as part of a JAR file installation, as described in <a href="../../../deploy_guide/Topics/cdevdeploy30736.html#cdevdeploy30736" class="xref" title="Application logic, which can be used by SQL functions and procedures, includes Java class files stored in a JAR file format. Storing application JAR files in RowStore simplifies application deployment, because it reduces the potential for problems with a user&#39;s classpath.">Storing and Loading JAR Files in RowStore</a>.<br/>

The key to designing a good user-defined type is to remember that data evolves over time, just like code. A good user-defined type has version information built into it. This allows the user-defined data to upgrade itself as the application changes. For this reason, it is a good idea for a user-defined type to implement *java.io.Externalizable* and not just *java.io.Serializable*. Although the SQL standard allows a Java class to implement only *java.io.Serializable*, this is bad practice for the following reasons:<br/>

-   **Recompilation** - If the second version of your application is compiled on a different platform from the first version, then your serialized objects may fail to deserialize. This problem and a possible workaround are discussed in the "Version Control" section near the end of this <a href="http://java.sun.com/developer/technicalArticles/Programming/serialization/" class="xref">Serialization Primer</a> and in the last paragraph of the header comment for *java.io.Serializable*.<br/>
-   **Evolution** - Your tools for evolving a class which simply implements *java.io.Serializable* are very limited.<br/><br/>

Fortunately, it is easy to write a version-aware UDT which implements *java.io.Serializable* and can evolve itself over time. For example, here is the first version of such a class:<br/>

``` pre<br/>
package com.acme.types;<br/>

import java.io.*;<br/>
import java.math.*;<br/>

public class Price implements Externalizable
{<br/>
    // initial version id<br/>
    private static final int FIRST_VERSION = 0;<br/>

    public String currencyCode;<br/>
    public BigDecimal amount;<br/>

    // zero-arg constructor needed by Externalizable machinery<br/>
    public Price() {}<br/>

    public Price( String currencyCode, BigDecimal amount )<br/>
    {<br/>
        this.currencyCode = currencyCode;<br/>
        this.amount = amount;<br/>
    }<br/>

    // Externalizable implementation<br/>
    public void writeExternal(ObjectOutput out) throws IOException<br/>
    {
        // first write the version id<br/>
        out.writeInt( FIRST_VERSION );<br/>

        // now write the state<br/>
        out.writeObject( currencyCode );<br/>
        out.writeObject( amount );<br/>
    }<br/>

    public void readExternal(ObjectInput in) <br/>
        throws IOException, ClassNotFoundException<br/>
    {<br/>
        // read the version id<br/>
        int oldVersion = in.readInt();<br/>
        if ( oldVersion < FIRST_VERSION ) { <br/>
            throw new IOException( "Corrupt data stream." ); 
        }<br/>
        if ( oldVersion > FIRST_VERSION ) { <br/>
            throw new IOException( "Can't deserialize from the future." );
        }<br/>

        currencyCode = (String) in.readObject();<br/>
        amount = (BigDecimal) in.readObject();<br/>
    }<br/>
}<br/>
之后，很容易编写添加新字段的用户定义类型的第二个版本。当Price从数据库读取旧版本的值时，它们会自动升级。更改以粗体显示：<br/>

package com.acme.types;<br/>

import java.io.*;<br/>
import java.math.*;<br/>
import java.sql.*;<br/>

public class Price implements Externalizable<br/>
{<br/>
    // initial version id<br/>
    private static final int FIRST_VERSION = 0;<br/>
    private static final int TIMESTAMPED_VERSION = FIRST_VERSION + 1;<br/>

    private static final Timestamp DEFAULT_TIMESTAMP = new Timestamp( 0L );

    public String currencyCode;
    public BigDecimal amount;
    public Timestamp timeInstant;

    // 0-arg constructor needed by Externalizable machinery
    public Price() {}

    public Price( String currencyCode, BigDecimal amount, 
                  Timestamp timeInstant )
    {
        this.currencyCode = currencyCode;
        this.amount = amount;
        this.timeInstant = timeInstant;
    }

    // Externalizable implementation
    public void writeExternal(ObjectOutput out) throws IOException
    {
        // first write the version id
        out.writeInt( TIMESTAMPED_VERSION );

        // now write the state
        out.writeObject( currencyCode );
        out.writeObject( amount );
        out.writeObject( timeInstant );
    }

    public void readExternal(ObjectInput in) 
        throws IOException, ClassNotFoundException
    {
        // read the version id
        int oldVersion = in.readInt();
        if ( oldVersion < FIRST_VERSION ) { 
            throw new IOException( "Corrupt data stream." ); 
        }
        if ( oldVersion > TIMESTAMPED_VERSION ) {
            throw new IOException( "Can't deserialize from the future." ); 
        }

        currencyCode = (String) in.readObject();
        amount = (BigDecimal) in.readObject();

        if ( oldVersion >= TIMESTAMPED_VERSION ) {
            timeInstant = (Timestamp) in.readObject(); 
        }
        else { 
            timeInstant = DEFAULT_TIMESTAMP; 
        }
    }
}





<br/>
应用程序需要在所有层级保持其代码同步。对于在客户端和服务器中运行的所有Java代码都是如此。这对于以多层次运行的功能和过程是正确的。对于以多层次运行的用户定义类型也是如此。当客户端和服务器运行应用程序代码的不同版本时，程序员应对防御进行编码。特别地，程序员应该为用户定义的类型编写防御性序列化逻辑，以便应用程序正常地处理客户端/服务器版本不匹配。
<br/>


###使用结果集和游标 结果集维护一个游标，指向其当前的数据行。您可以使用结果集逐个逐步处理这些行。<br/>


在RowStore中，任何SELECT语句都会生成可以使用java.sql.ResultSet对象进行控制的游标。RowStore不支持SQL-92的DECLARE CURSOR语言结构来创建游标，但它支持使用可更新游标定位删除和定位更新。<br/>

注意：本主题已从Apache Derby文档源中进行了修改，并受Apache许可证的约束：“`根据一个或多个贡献者许可协议授予Apache Software Foundation（ASF）。有关版权所有权的其他信息，请参阅本工作分发的NOTICE文件。ASF根据Apache许可证版本2.0（“许可证”）向您授予此文件;除了符合许可证之外，您不得使用此文件。您可以通过http：//www.apache获取许可证的副本.org / licenses / LICENSE-2.0除非适用法律或书面同意，否则根据许可证分发的软件按“原样”分发，不附有任何形式的担保或条件， 明示或暗示。有关许可证下的权限和限制的特定语言，请参阅许可证。"<br/>

不可更新，仅转发结果集 RowStore支持的最简单的结果集不能更新，并且具有仅向前移动的一个光标。<br/>
更新结果集 通过使用结果集更新方法更新RowStore结果集（updateRow()，deleteRow()和insertRow()），或者通过使用定位更新或删除查询。RowStore支持可滚动和不可滚动的可更新结果集（仅向前）。<br/>
可滚动的不敏感结果集 JDBC提供两种类型的结果集，允许您沿任一方向滚动或将光标移动到特定行。RowStore支持以下类型之一：可滚动不敏感的结果集（ResultSet.TYPE_SCROLL_INSENSITIVE）。<br/>
结果集和自动 提交发出提交导致连接上的所有结果集都被关闭。<br/>





使用RowStore ADO.NET组件 RowStore提供用于Visual Studio 2013或2012开发的ADO.NET客户端，以及Visual Studio 2010的ADO.NET实体框架插件。<br/>


使用RowStore ADO.NET客户端 RowStore 1.4.1 ADO.NET驱动程序为Windows 32位和64位平台提供了一个本地线路协议数据提供程序。ADO.NET客户端支持使用Visual Studio 2013或Visual Studio 2012进行开发。您可以使用客户端将RowStore数据库作为数据源添加到Visual Studio中的“服务器资源管理器”列表中，以使用查询构建器等进行设计查询。<br/>
使用RowStore.NET实体框架插件 使用RowStore.NET实体框架从现有模式生成模型对象，或从模型对象生成模式。您还可以使用语言集成查询（LINQ）发出查询，然后使用C＃或VB.NET检索和操作数据为强类型对象。实体框架插件提供对Visual Studio 2010的支持。<br/>



###ODBC驱动程序支持的配置 本主题列出了RowStore支持的ODBC驱动程序配置。<br/>
###使用RowStore与Hibernate Pivotal提供了一个Hibernate方言文件，它定义了SQL语言的RowStore变体。您可以使用此文件与RowStore JDBC驱动程序配置RowStore作为开发Hibernate项目的数据库。<br/>

Pivotal提供了一个定义SQL语言的RowStore变体的Hibernate方言文件。您可以使用此文件与RowStore JDBC驱动程序配置RowStore作为开发Hibernate项目的数据库。<br/>

见RowStore的Hibernate方言的枢纽社区站点有关如何获取并使用RowStore方言文件，你的Hibernate项目的信息。





## CQL编程基础

本节介绍CQL语言，基本编程任务和最佳实践。

### CQL编程基础和最佳实践

本节介绍了编写CQL应用程序时必须遵循的规则，以及 与之相关的最佳实践。如果您不熟悉任何术语，请参阅  概念指南或词汇表。

### 命名空间

命名空间是DFlow环境中包含应用程序、流程及其组件（如源，流等）和仪表板的逻辑域。

每个用户帐户都有一个名称相同的个人命名空间。例如，admin用户具有admin命名空间。

每个DFlow应用程序都存在于命名空间中。例如，如果使用安装程序安装评估版本，应用程序在PosApp命名空间中Samples。

有关详细信息，请参阅使用命名空间。

### 连接

DFlow应用程序由一组组件和它们之间的连接组成。一些组件可以直接连接，其他组件可以使用中间流，如下所示：

source (OUTPUT TO) > stream > CQ (SELECT FROM)

source (OUTPUT TO) > stream > window (OVER)

source (OUTPUT TO) > stream > target (INPUT FROM)

cache > CQ (SELECT FROM)

WActionStore > CQ (SELECT FROM)

window > CQ (SELECT FROM)

window > CQ (SELECT FROM window, INSERT INTO stream) > stream > window (OVER)

CQ (INSERT INTO) > WActionStore

CQ (INSERT INTO) > stream > target (INPUT FROM)


<table>
<tr>
<td>注意</td>
</tr>
<tr>
<td>缓存或窗口的输出必须由CQ的SELECT语句处理，然后才能传递给目标。换句话说，cache> stream> target和window> stream> target是无效序列。</td>
</tr>
</table>


### 关联

Inner Join通过在FROM子句中指定多个源来隐含地执行。例如：

<code>
FROM PosData5Minutes p，HourlyAveLookup l
</code>

其他关联必须被明确声明。请参见CREATE CQ（查询）。

关联必须包括绑定数据，换句话说，至少有一个缓存，数据存储或窗口。例如，假设StoreOrders是源自数据源的数据流，ZipCodeLookup是缓存，以下也是有效的：

<code>
SELECT s.storeId，z.latVal，z.longVal
 FROM StoreOrders s，ZipCodeLookup z
 WHERE s.zip = z.zip
</code>

在加入来自多个数据流的事件时，CQ SELECT FROM应该引用Windows而不是流。有关示例，请参阅MultiLogApp示例应用程序中的LargeRTCheck和ZeroContentCheck流程。定义窗口，使其大到足以包含要加入的事件。如有必要，请使用GRACE PERIOD选项（请参阅CREATE STREAM）和/或分拣机（请参阅CREATE SORTER）以确保事件的时间戳同步。



### 依赖关系

您必须先创建一个组件，然后才能在另一个组件的CREATE语句中引用该组件。

<table>
<tr>
<td>组件类型</td><td>创建之后</td><td>创建之前</td>
</tr>
<tr>
<td>应用</td><td></td><td>任何其他组件</td>
</tr>
<tr>
<td>流*</td><td>包含应用程序</td><td>包含的组件</td>
</tr>
<tr>
<td>资源</td><td></td><td>其输出流是输入的任何窗口(OVER),其输出流是输入的任何查询(SELECT FROM)</td>
</tr>
<tr>
<td>类型</td><td></td><td>任何流是它的类型（OF）**，任何缓存为它的类型（OF），在WActionStore的CONTEXT OF或EVENT TYPES子句中引用它</td>
</tr>
<tr>
<td>数据流</td><td>其类型**</td><td>其输出的任何来源（OUTPUT TO）

任何为它输入的窗口（OVER）

任何查询是它的输入（SELECT FROM）

其输入的任何目标（INPUT FROM）</td>
</tr>
<tr>
<td>缓存</td><td></td><td>任何查询是它的输入（SELECT FROM）</td>
</tr>
<tr>
<td>窗口</td><td>其输入流（OVER）</td><td>任何查询是它的输入（SELECT FROM） </td>
</tr>
<tr>
<td>WActionStore</td><td>其类型（CONTEXT OF和EVENT TYPES）</td><td>任何查询是它的输入（SELECT FROM）

任何查询是输出（INSERT INTO） </td>
</tr>
<tr>
<td>CQ（查询）</td><td>所有输入流（SELECT FROM）

所有输入缓存（SELECT FROM）

所有输入窗口（SELECT FROM）

其输出流（INSERT INTO）</td><td>在目标中引用其输出流 </td>
</tr>
<tr>
<td>数据目标</td><td>其输入流（INPUT FROM）</td><td> </td>
</tr>
</table>

*当应用程序只包含一个流程时，不需要由CREATE FLOW语句显式声明。有关具有单个流的应用程序和MultiLogApp的示例，请参阅PosApp作为具有多个流的示例。

**通过在源的OUTPUT TO子句中引用流自动创建流时，将使用与源适配器关联的内置类型，因此无需首先手动创建该类型。


### 组件名称

应用程序，流程，来源等的名称：

必须只包含字母数字字符和下划线

可能不以数字字符开头

在命名空间中必须是唯一的


### 分组语句和评论

CQL支持SQL风格的注释。例如：


<code>
CREATE APPLICATION PosApp; ...
</code>

为了使您的CQL更容易阅读，我们建议您按照CQ或流程对语句进行分组，并根据需要在每个组前加上解释性注释。有关示例应用程序指南，请参阅CQL源代码。

### 在开发过程中加载和重新加载CQL应用程序

在启动新的CQL应用程序之前，必须先创建并部署它（请参阅管理部署组）。例如，以下系列控制台命令创建，部署和运行simple.tql：

<code>
W (admin): @Samples/simple.tql;
 Processing - CREATE APPLICATION simple
 ...
 Processing - END APPLICATION simple
 Elapsed time: 2473 ms
W (admin): deploy application simple in default;
 Processing - deploy application simple in default
 Elapsed time: 180 m
W (admin): start application simple;
 Processing - start application simple
 Elapsed time: 73 ms
</code>


编辑TQL文件后，要运行新版本的应用程序，您必须：

取消部署并删除旧版本

加载，部署和启动新版本

例如，以下一系列命令将丢弃上述命令加载的应用程序。

<code>
W (admin) > undeploy application simple;
 Processing - undeploy application simple
 Elapsed time: 358 ms
W (admin) > drop application simple cascade;
</code>


现在重复的@，DEPLOY和START 你第一次运行应用程序的命令。

您可以通过将命令添加到应用程序来自动化：

<code>
UNDEPLOY APPLICATION PosApp;
DROP APPLICATION PosApp CASCADE;
CREATE APPLICATION PosApp;
...
END APPLICATION PosApp;
DEPLOY APPLICATION PosApp;
START APPLICATION PosApp;
</code>

### 分析WAEvent的数据字段

WAEvent是许多Reader输出流使用的数据类型。它的data字段是一个包含一个事件的字段值的数组。以下是在从CsvDataSource在输出流WAEvent格式一个示例事件PosApp示例应用：

<code>
WAEvent{
  data: ["COMPANY 1159","IQ6wCy3k7PnAiRAN71ROxcNBavvVoUcwp7y","8229344557372754288","1","20130312173212",
"0614","USD","329.64","2094770823399082","79769","Odessa"]
  metadata: {"RecordStatus":"VALID_RECORD","FileName":"posdata.csv","FileOffset":154173}
  before: null
  dataPresenceBitMap: "AAA="
  beforePresenceBitMap: "AAA="
  typeUUID: null
}; 
</code>

dataPresenceBitMap，beforePresenceBitMap并且typeUUID被保留并且应该被忽略。

要解析 data 数组，PosApp使用以下TQL：

<code>
CREATE CQ CsvToPosData
INSERT INTO PosDataStream
SELECT TO_STRING(data[1]) AS merchantId,
  TO_DATEF(data[4],'yyyyMMddHHmmss') AS dateTime,
  DHOURS(TO_DATEF(data[4],'yyyyMMddHHmmss')) AS hourValue,
  TO_DOUBLE(data[7]) AS amount,
  TO_STRING(data[9]) AS zip
FROM CsvStream;
</code>

PosDataStream在 Select 语句中使用数据类型和AS字符串自动创建。
在 Select 语句中的data[#]函数顺序决定了输出中数据域的顺序。这些可能通过任意顺序来指定：例如，data[1]可以在data[0]之前。
通过 Select 语句的不能被引用的值域将被丢弃。
data[#]函数从0开始在数组中计数值域，因此数组(COMPANY 1159)中的第一个值域将被忽略。
No-String 的值将被转换为通过输出流所需要的相关类型，通过TO_DATEF, DHOURS, 和 TO_DOUBLE 函数(查看函数详情)。

在PosDataStream输出中，上述示例事件的解析版本为：

<code>
merchantId：“IQ6wCy3k7PnAiRAN71ROxcNBavvVoUcwp7y”
 dateTime：1363134732000
 hourValue：17
 amount：329.64
 zip：“79769”
</code>

见PosApp以获取更多信息。另请参见MultiLogApp中ParseAccessLog和ParseLog4J的讨论。


### 使用正则表达式

DFlow支持在您的TQL应用程序中使用正则表达式（regex）。重要的是要记住，正则表达式的DFlow实现是基于Java的（请参阅 java.util.regex.Pattern），因此在开发正则表达式时需要注意以下几点：
— 反斜杠字符（\）在Java字符串中被识别为转义字符，因此如果要定义类似\w正则表达式的内容，请\\w在这种情况下使用。
— 在正则表达式中，\\匹配单个反斜杠字面值。因此，如果要在正则表达式的Striim Java实现中使用反斜杠字符作为字面值，则必须实际使用\\\\。

— 该java.lang.String中类为您提供了这些方法支持正则表达式：matches(), split(), replaceFirst(), replaceAll()。请注意，这些String.replace()方法不支持正则表达式。

CQL支持来自java.util.regex的正则表达式语法和构造。请注意，这与POSIX正则表达式有一些区别。

如果您刚开始使用正则表达式，请参考以下资源来开始使用：

<a href="http://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html">java.util.regex.Pattern</a>

<a href="https://docs.oracle.com/javase/tutorial/essential/regex/">Oracle：Java教程。课程：正则表达式</a>

<a href="http://www.vogella.com/tutorials/JavaRegularExpressions/article.html">Lars Vogel：Java Regex - 教程</a>


你可以使用正则表达式LIKE和NOT LIKE表达式。例如：

WHERE ProcessName NOT LIKE '%.tmp%'：从临时文件中过滤掉数据

WHERE instance_applications LIKE '%Apache%'：在其名称中仅选择具有Apache的应用程序

WHERE MerchantID LIKE '45%'：仅选择ID为45的商家。

MultiLogApp示例Apache访问日志数据中的以下条目包括有关第4行REST API调用的信息：

<code>
0: 206.130.134.68
1: -
2: AWashington
3: 25/Oct/2013:11:28:36.960 -0700
4: GET http://cloud.saas.me/query?type=ChatterMessage&id=01e33d9a-34ee-ccd0-84b9-
   14109fcf2383&jsessionId=01e33d9a-34c9-1c68-84b9-14109fcf2383 HTTP/1.1
5: 200
6: 0
7: -
8: Mozilla/5.0 (X11; Linux i686) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu 
   Chromium/28.0.1500.71 Chrome/28.0.1500.71 Safari/537.36
9: 1506
</code>

该MATCH函数也使用正则表达式。CQ中的MATCH功能ParseAccessLog解析第4行中的信息以提取会话ID：

<code>
MATCH（data [4]， “。* jsessionId =（。*）”）
</code>

解析的输出是：

<code>
sessionId： “01e33d9a-34c9-1c68-84b9-14109fcf2383”
</code>

以下，也来自MultiLogApp，是WAEvent 数组的data[2]元素的示例：RawXMLStreamdata

<code>
"Problem in API call [api=login] [session=01e3928f-e975-ffd4-bdc5-14109fcf2383] 
[user=HGonzalez] [sobject=User]","com.me.saas.SaasMultiApplication$SaasException: 
Problem in API call [api=login] [session=01e3928f-e975-ffd4-bdc5-14109fcf2383] 
[user=HGonzalez] [sobject=User]\n\tat com.me.saas.SaasMultiApplication.login
(SaasMultiApplication.java:1253)\n\tat 
sun.reflect.GeneratedMethodAccessor11.invoke(Unknown Source)\n\tat 
sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
\n\tat java.lang.reflect.Method.invoke(Method.java:606)\n\tat 
com.me.saas.SaasMultiApplication$UserApiCall.invoke(SaasMultiApplication.java:360)\n\tat 
com.me.saas.SaasMultiApplication$Session.login(SaasMultiApplication.java:1447)\n\tat 
com.me.saas.SaasMultiApplication.main(SaasMultiApplication.java:1587)"
</code>

这由ParseLog4JCQ 解析如下：

<code>
MATCH（data [2]，'\\\\ [api =（[a-zA-Z0-9] *）\\\\]'），
MATCH（data [2]，'\\\\ [session =（[a-zA-Z0-9 \\  - ] *）\\\\]'），
MATCH（data [2]，'\\\\ [user =（[a-zA-Z0-9 \\  - ] *）\\\\]'），
MATCH（data [2]，'\\\\ [sobject =（[a-zA-Z0-9] *）\\\\]'）
</code>

解析的输出是：

<code>
api： “login” 
sessionId： “01e3928f-e975-ffd4-bdc5-14109fcf2383” 
userId： “HGonzalez” 
sobject： “User”
</code>

有关其他示例，请参阅使用正则表达式解析源，FreeFormTextParser和MultiFileReader。

### 处理异常

默认情况下，当DFlow遇到非致命异常时，它会忽略它并继续。您可以在EXCEPTIONHANDLER语句中添加一个子句来CREATE APPLICATION记录异常并执行各种操作。语法是：
<code>
CREATE APPLICATION ... EXCEPTIONHANDLER ([<exception>:'<action>',...]);
</code>

支持的异常为:

ArithmeticException

ClassCastException

ConnectionException

InvalidDataException

NullPointerException

NumberFormatException

SystemException

UnknownException

支持的操作为:

IGNORE

STOP









































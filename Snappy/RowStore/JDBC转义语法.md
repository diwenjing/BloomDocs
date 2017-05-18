#JDBC转义语法
###JDBC提供了一种平滑不同DBMS供应商实现SQL的方式的一些差异。这被称为转义语法。转义语法表明由特定供应商提供的JDBC驱动程序扫描任何转义语法，并将其转换为特定数据库所理解的代码。这使得转义语法与DBMS无关。

###JDBC转义子句以花括号开头和结尾。关键字始终遵循开放式大括号：

<code>
{keyword }
</code>
<br/>
###注意： RowStore在* Connection.nativeSQL *调用中返回不变的SQL，因为转义语法是SQL本机的。此外，为此，无需调用* Statement.setEscapeProcessing *。

###RowStore仅支持以下不区分大小写的JDBC转义关键字。

###用于呼叫语句的JDBC Escape关键字
<br/>
支持该语法是一个java.sql.Statement中和java.sql.PreparedStatement中除了CallableStatement中。
<br/>
###句法
	{call statement }
	-- Call a Java procedure
	{ call TOURS.BOOK_TOUR(?, ?) }







###用于LIKE子句的JDBC Escape语法


###百分号％和下划线_是SQL LIKE子句中的元字符。JDBC提供了强制这些字符在字面上解释的语法。紧跟LIKE表达式的JDBC子句允许您指定转义字符：

###句法
	WHERE CharacterExpression [ NOT ] LIKE
	    CharacterExpressionWithWildCard
	    { ESCAPE 'escapeCharacter' }
	-- find all rows in which a begins with the character "%"
	SELECT a FROM tabA WHERE a LIKE '$%%' {escape '$'}
	-- find all rows in which a ends with the character "_"
	SELECT a FROM tabA WHERE a LIKE '%=_' {escape '='}
###注意：？如果LIKE模式也是动态参数（？），则不允许作为转义字符。

###在某些语言中，单个字符由多个排序规则单元（16位字符）组成。所述escapeCharacter escape子句中使用的必须是单个核对单元，以正常工作。

###您也可以使用转义字符序列为LIKE，而不使用JDBC的花括号; 请参阅布尔表达式。















##用于fn关键字的JDBC Escape语法

<br/>
###您可以使用fn关键字指定JDBC转义语法中的函数。
<br/>
###句法
    {fn functionCall}
###其中functionCall是以下标量函数之一的名称：
<br/>
###abs 
返回数字的绝对值。
<br/>
###abs(NumericExpression)
JDBC转义语法{fn abs（NumericExpression）}等同于内置语法。有关更多信息，请参阅ABS或ABSVAL。
<br/>
###acos 
返回指定数字的反余弦。
<br/>
###acos(number)
JDBC转义语法{fn acos（number）}相当于内置语法ACOS（number）。有关更多信息，请参阅ACOS。
<br/>
###asin 
返回指定数字的正弦。
<br/>
###asin(number)
JDBC转义语法{fn asin（number）}等效于内置语法ASIN（number）。有关更多信息，请参阅ASIN。
<br/>
###atan 
返回指定数字的反正切。
<br/>
###atan(number)
JDBC转义语法{fn atan（number）}等同于内置语法ATAN（number）。有关更多信息，请参阅ATAN。
<br/>
###ceiling 
将指定的数字向上舍入，并返回大于或等于指定数字的最小数字。
<br/>
###ceiling(number)
JDBC转义语法{fn ceiling（number）}相当于内置语法CEILING（number）。有关更多信息，请参阅CEIL或CEILING。
<br/>
###concat 
返回字符串的并置。
<br/>
###concat(CharacterExpression, CharacterExpression)
通过将第二个字符串附加到第一个字符串形成的字符串。如果任一字符串为空，则结果为NULL。JDBC转义语法{fn concat（CharacterExpression，CharacterExpression）等效于内置语法{​​CharacterExpression || CharacterExpression}。有关详细信息，请参阅连接运算符。
<br/>
###cos 
返回指定数字的余弦值。

###cos(number)
JDBC转义语法{fn cos（number）}等效于内置语法COS（number）。有关更多信息，请参阅COS。
<br/>
度
从弧度到指定度数转换。
<br/>
###degrees(number)
JDBC转义语法{fn degrees（number）}等同于内置语法DEGREES（number）。有关更多信息，请参阅DEGREES。
<br/>
###exp 
返回e提高到指定数量的幂。
<br/>
###exp(number)
JDBC转义语法{fn exp（number）}等效于内置语法EXP（number）。有关详细信息，请参阅EXP。
<br/>
###floor 
将指定的数字向下舍入，并返回小于或等于指定数字的最大数字。
<br/>
###floor(number)
JDBC转义语法{fn floor（number）}等同于内置语法FLOOR（number）。有关更多信息，请参阅FLOOR。

定位
返回在第二位置CharacterExpression所述第一的第一次出现的CharacterExpression。从第二个CharacterExpression的开始搜索，除非指定了startIndex参数。
<br/>
###locate(CharacterExpression,CharacterExpression [, startIndex] )
JDBC转义语法{fn locate（CharacterExpression，CharacterExpression [，startIndex ]）}等同于内置语法。有关详细信息，请参阅LOCATE。
<br/>
###log 
返回指定数字的自然对数（基数e）。
<br/>
###log(number) 
JDBC转义语法{fn log（number）}相当于内置语法LOG（number）。有关更多信息，请参阅LN或LOG。
<br/>
###log10 
返回指定数字的基10对数。
<br/>
###log10(number)
JDBC转义语法{fn log10（number）}等效于内置语法LOG10（number）。有关更多信息，请参阅LOG10。
<br/>
###mod 
返回参数1的余数（模数）除以参数2.只有当参数1为负数时，结果为负。
<br/>
###mod(integer_type, integer_type)
有关更多信息，请参阅MOD。
<br/>
###pi 
返回一个比pi更接近的值。
<br/>
###pi()
JDBC转义语法{fn pi（）}等同于内置语法PI（）。有关更多信息，请参阅PI。
<br/>
弧度
将指定的数字从度转换为弧度。
<br/>
###radians(number)
JDBC转义语法{fn弧度（数））等同于内置语法RADIANS（number）。有关更多信息，请参阅RADIANS。
<br/>
###sin 
返回指定数字的正弦。
<br/>
###sin(number)
JDBC转义语法{fn sin（number）}等效于内置语法SIN（number）。有关更多信息，请参见SIN。
<br/>
###sqrt 
返回浮点数的平方根。
<br/>
###sqrt(FloatingPointExpression)
JDBC转义语法{fn sqrt（FloatingPointExpression）}等效于内置语法。有关更多信息，请参见SQRT。
<br/>
子
形成由提取字符的字符串长度从字符CharacterExpression在开始的startIndex。CharacterExpression中第一个字符的索引为1。
<br/>
###substring(CharacterExpression, startIndex, length)
tan 
返回指定数字的切线。
<br/>
###tan(number)
JDBC转义语法{fn tan（number）}相当于内置语法TAN（number）。有关更多信息，请参阅TAN。
<br/>
###TIMESTAMPADD 
使用该TIMESTAMPADD函数将间隔的值添加到时间戳。该函数根据间隔类型将整数应用于指定的时间戳，并将总和返回为新的时间戳。您可以使用负整数从时间戳中减去。
<br/>
这TIMESTAMPADD是一个JDBC转义函数，只能通过使用JDBC转义函数语法进行访问。
<br/>
###TIMESTAMPADD( interval, integerExpression, timestampExpression )
要执行TIMESTAMPADD日期和时间，有必要将日期和时间转换为时间戳。在日期栏中输入00：00：00.0，将日期转换为时间戳。通过将当前日期放在日期字段中将时间转换为时间戳。
<br/>
您不应该将一个datetime列放在WHERE子句中的时间戳算术函数中，因为优化器不会在列上使用任何索引。
<br/>
###TIMESTAMPDIFF 
使用该TIMESTAMPDIFF函数以指定的间隔查找两个时间戳值之间的差异。例如，该函数可以返回两个指定时间戳之间的分钟数。
<br/>
这TIMESTAMPDIFF是一个JDBC转义函数，只能通过使用JDBC转义函数语法进行访问。
<br/>
###TIMESTAMPDIFF( interval, timestampExpression1, timestampExpression2 )
要执行TIMESTAMPDIFF日期和时间，有必要将日期和时间转换为时间戳。在日期栏中输入00：00：00.0，将日期转换为时间戳。通过将当前日期放在日期字段中将时间转换为时间戳。
<br/>
您不应该将一个datetime列放在WHERE子句中的时间戳算术函数中，因为优化器不会在列上使用任何索引。
<br/>
TIMESTAMPADD和TIMESTAMPDIFF的有效间隔
该函数TIMESTAMPADD和TIMESTAMPDIFF函数用于执行具有时间戳的算术。这两个函数使用以下有效的间隔进行算术运算： - SQL_TSI_DAY - SQL_TSI_FRAC_SECOND - SQL_TSI_HOUR - SQL_TSI_MINUTE - SQL_TSI_MONTH - SQL_TSI_QUARTER - SQL_TSI_SECOND - SQL_TSI_WEEK - SQL_TSI_YEAR
<br/>
###TIMESTAMPADD和TIMESTAMPDIFF转义函数的示例
要返回比当前时间戳记晚一个月的时间戳值，请使用以下语法：
<br/>
    {fn TIMESTAMPADD( SQL_TSI_MONTH, 1, CURRENT_TIMESTAMP)}
要返回现在和2008年1月1日的指定时间之间的周数，请使用以下语法：
<br/>
    {fn TIMESTAMPDIFF(SQL_TSI_WEEK, CURRENT_TIMESTAMP, 
    timestamp('2008-01-01-12.00.00.000000'))}


<br/>



##外部连接的JDBC转义语法
<br/>
RowStore将外连接（和所有连接操作）的JDBC转义语法解释为与外连接或适当连接操作的正确SQL语法相同。
<br/>
另见查询功能和限制。
<br/>
句法<br/>
    {oj JOIN operations [JOIN operations ]* }
相当于<br/>
JOIN operations [JOIN operations ]* 
-- outer join
SELECT *
FROM
{oj Countries LEFT OUTER JOIN Cities ON 
   (Countries.country_ISO_code=Cities.country_ISO_code)}
-- another join operation
SELECT *
FROM
{oj Countries JOIN Cities ON (Countries.country_ISO_code=Cities.country_ISO_code)}
-- a TableExpression can be a joinOperation. Therefore
-- you can have multiple join operations in a FROM clause
SELECT E.EMPNO, E.LASTNAME, M.EMPNO, M.LASTNAME
FROM {oj EMPLOYEE E INNER JOIN DEPARTMENT
INNER JOIN EMPLOYEE M ON MGRNO = M.EMPNO ON E.WORKDEPT = DEPTNO};



##时间格式的JDBC转义语法
<br/>
RowStore将时间的JDBC转义语法解释为等同于正确的SQL语法的时间。RowStore还支持8个字符（6位数和2位小数）的ISO格式。
<br/>
句法<br/>
	{t 'hh:mm:ss'}
	相当于
	TIME 'hh:mm:ss'
	例
	VALUES {t '20:00:03'}









##日期格式的JDBC转义语法

RowStore将日期的JDBC转义语法解释为等同于日期的正确SQL语法。

句法
	{d 'yyyy-mm-dd'}
	相当于
	DATE 'yyyy-mm-dd'
	例
	VALUES {d '1995-12-19'}








##用于Timestamp格式的JDBC Escape语法

RowStore将时间戳的JDBC转义语法解释为等于时间戳的正确SQL语法。RowStore还支持23个字符（17位数，3个破折号和3个小数点）的ISO格式。

句法
	{ts 'yyyy-mm-dd hh:mm:ss.f...'}
	相当于
	TIMESTAMP 'yyyy-mm-dd hh:mm:ss.f...'
	时间戳常量（.f ...）的小数部分可以省略。
	
	VALUES {ts '1999-01-09 20:11:11.123455'}





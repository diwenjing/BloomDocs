# 配置安全性 #
您可以通过配置用户身份验证和SQL授权来确保RowStore部署，并使用SSL / TLS在成员之间启用加密。<br/>

> 注意：本章中的许多主题已从Apache Derby文档源文件中进行了修改，并受Apache许可证的约束：“根据一个或多个贡献者许可协议授予Apache Software Foundation（ASF）。有关版权所有权的其他信息，请参阅本工作分发的NOTICE文件。ASF根据Apache许可证版本2.0（“许可证”）向您授予此文件;除了符合许可证之外，您不得使用此文件。您可以通过http：//www.apache获取许可证的副本.org / licenses / LICENSE-2.0除非适用法律或书面同意，根据许可证分发的软件以“按现状”分发， **无明示或暗示的任何形式的担保或条件。有关许可证下的权限和限制的特定语言，请参阅许可证。**“

- **认证，授权和成员所有权的用户名** 在使用用户身份验证和用户授权时，您需要了解每个系统如何处理用户名。

- **配置认证** 启用用户认证后，需要有效的用户名和密码才能在分布式系统中启动新的RowStore成员; 加入现有的分布式系统; 并连接到正在运行的RowStore分布式系统。RowStore针对为系统定义的用户的存储库验证名称和密码。默认情况下未启用身份验证。

- **配置用户授权** 当您启用用户授权时，RowStore会验证用户是否被授予访问模式，数据库对象或SQL操作的权限。

- **使用SSL** / TLS配置网络加密和身份验证 默认情况下，所有RowStore网络流量都是未加密的，用户名和用户密码除外，可以单独加密。还没有网络层访问控制机制。对于可能存在安全问题的部署场景，RowStore网络服务器通过安全套接字层/传输层安全性（SSL / TLS）支持网络安全。

## 认证，授权和成员所有权的用户名 ##
当使用用户身份验证和用户授权时，您需要了解每个系统如何处理用户名。<br/>

- **用户名转换为授权标识符**
- **处理用户名中的敏感度和特殊字符**
- **了解RowStore会员所有权**

### 用户名转换为授权标识符 ###
RowStore系统中的用户名称为授权标识符。授权标识符是表示用户名称的字符串，如果在连接请求中提供了该名称。例如，内置函数CURRENT_USER返回当前用户的授权标识符。<br/>

授权标识符传递给RowStore系统后，它将成为一个SQL92Identifier。`A SQL92Identifier`是一种表示数据库对象（如表或列）的标识符。甲SQL92Identifier是不区分大小写的（它被转换为全部大写），除非它被用双引号分隔。甲SQL92Identifier被限制为128个字符，并且具有其它限制。<br/>

即使用户身份验证被关闭，即使所有用户都被允许访问所有数据库，所有用户名称也必须是有效的授权标识符。

有关SQL92标识符的更多信息，请参阅**关键字和标识符**。


### 处理用户名中的敏感度和特殊字符 ###
如果使用外部认证系统，则在验证发生（但在用户授权之前），RowStore不会将用户的名称转换为授权标识符。例如，使用名为Fred的示例用户：<br/>



- 在用户认证系统中，Fred可能被称为FRed。如果外部用户认证服务区分大小写，则Fred必须始终输入其名称：


<br/>

    connect client 'localhost:1527;user=FRed;password=flintstone';



- 在RowStore用户授权系统中，Fred成为不区分大小写的授权标识符。在这里，FRed被称为FRED。


- 指定授权访问系统的用户必须列出Fred的授权标识符FRED（可以键入FRED，FREd或fred，因为系统自动将授权标识符转换为全大写）。<br/>

<br/>

    snappydata.authz-full-access-users=sa,FRED,mary

> 注意：在启动RowStore时，您必须在命令行中将访问列表属性“snappydata.authz-full-access-users”定义为Java系统属性，而不是在gemfirexd.properties文件中。

还要考虑第二个例子，其中Fred在用户认证系统中的名称略有不同：<br/>



- 在用户认证系统中，Fred被称为Fred！。您现在必须在用户名周围放置双引号，因为它不是有效的`SQL92Identifier`。（RowStore知道在将名称传递给外部身份验证系统时删除双引号。）

<br/>

    connect client 'localhost:1527;user="Fred!";password=flintstone';


- 在RowStore用户授权系统中，Fred现在成为一个区分大小写的授权标识符。在这种情况下，Fred被称为Fred！。


- 当指定有权访问系统的用户时，必须使用Fred的授权标识符“Fred！”，它必须始终用双引号分隔：
snappydata.authz-full-access-users=sa,"Fred!",manager


> 注意：在启动RowStore时，您必须在命令行中将访问列表属性“snappydata.authz-full-access-users”定义为Java系统属性，而不是在gemfirexd.properties文件中。

如第一个例子所示，外部认证系统可能是区分大小写的，而RowStore中的授权标识可能不是。如果您的身份验证系统允许名称不同的两个不同用户，请在连接请求中分隔所有用户名，以使所有用户名在RowStore系统中区分大小写。另外，还必须使用双引号对不符合SQL92Identifier规则的用户名进行分隔。<br/>


### 了解RowStore会员所有权 ###
启动RowStore成员的经身份验证的系统用户是该成员JVM的所有者。术语JVM所有者是指引导成员的用户的授权标识符。JVM所有者拥有在分布式系统中创建的任何模式和数据库对象。所有者还拥有由其他JVM所有者创建的JVM中的任何模式和数据库对象。换句话说，RowStore在引导RowStore成员时可以互换地对待所有JVM。<br/>

当启用SQL授权时，JVM所有者具有自动SQL级别权限，并且可以使用**GRANT**命令向任何数据库对象授予访问权限。有关详细信息，请参阅**配置用户**授权。<br/>

如果您启用或计划启用SQL授权，则控制JVM所有者的身份变得非常重要。启动JVM后，RowStore成员的所有者无法更改。但是，可以通过停止成员进程，然后使用其他用户的凭据重新启动成员所有权。<br/>


> 注意：如果成员启动*而不*提供用户（仅当未启用身份验证时才可能），则JVM所有者将设置为默认授权标识符“APP”，该默认授权标识符也是默认模式的名称。

注意：在RowStore成员启动后，JVM所有者无法更改。相反，您必须停止该成员，然后使用不同的用户凭据重新启动该成员。如果计划在启用SQL授权的情况下运行，请以您要作为JVM所有者的用户身份启动新的RowStore成员。<br/>

## 配置认证 ##
启用用户认证时，需要有效的用户名和密码才能在分布式系统中启动新的RowStore成员; 加入现有的分布式系统; 并连接到正在运行的RowStore分布式系统。RowStore针对为系统定义的用户的存储库验证名称和密码。默认情况下未启用身份验证。<br/>

RowStore根据您指定的用户的存储库验证用户凭据。RowStore提供内置的存储库，也可以将系统配置为使用您创建的LDAP目录服务或自定义身份验证服务。<br/>


> 注意： RowStore内置的认证机制仅适用于开发和测试目的。生产系统应使用LDAP存储库或自定义目录服务，并应使用SSL / TLS保护网络连接。

在RowStore验证用户之后，它通过在新的分布式系统中启动RowStore成员，将该成员加入到现有的分布式系统，或简单地将客户端连接到分布式系统，来授予用户对RowStore分布式系统的访问权限。经过身份验证的用户还可以根据SQL授权配置访问数据库对象。（默认情况下未启用SQL授权）。<br/>

> 注意：由于RowStore可以嵌入到Java应用程序中，所以也可以部署一个系统，其中Java应用程序而不是嵌入式RowStore进程处理用户身份验证。

- **启用用户验证** 要使用RowStore启用用户身份验证，必须使用RowStore定位器进行成员发现。RowStore在RowStore定位器和引导并加入分布式系统的后续RowStore成员之间使用相互验证。如果使用组播进行成员发现，则不支持用户身份验证。

- **使用BUILTIN身份验证** RowStore BUILTIN身份验证提供程序仅适用于开发和测试。使用此安全机制时，RowStore系统维护用户名和密码信息的存储库。

- **使用LDAP目录服务** LDAP（轻量级目录访问协议）提供通过TCP / IP运行的开放目录访问协议。RowStore可以根据企业中现有的LDAP目录服务来验证用户。

- **外部目录服务的JNDI特定属性** RowStore允许您设置一些高级JNDI属性，您可以通过任何支持的设置RowStore属性的方式进行设置。通常，您可以在配置外部身份验证服务的系统级别进行设置。

### 启用用户验证 ###
要使用RowStore进行用户认证，您必须使用RowStore定位器进行成员发现。RowStore在RowStore定位器和引导并加入分布式系统的后续RowStore成员之间使用相互验证。如果使用组播进行成员发现，则不支持用户身份验证。<br/>

#### 程序 ####



1. 对于RowStore集群（服务器，定位器和访问器）的每个成员，设置`snappydata.auth-provider`属性以启用用户身份验证，并指定RowStore用于验证用户的机制。<br/>
对于服务器和定位器，在命令行中指定`-auth-provider= provider_name`，或在gemfirexd.properties文件中定义`snappydata.auth-provider= provider_name`属性。<br/>
仅用于开发和测试，请指定BUILTIN作为提供程序名称以使用RowStore内置身份验证机制。出于生产目的，请指定LDAP以使用现有的LDAP存储库，或指定实现UserAuthenticator接口的自定义提供程序类的名称。<br/>

2. 在指定的身份验证提供程序中配置用户凭据。请参阅**使用BUILTIN身份验证**或**使用LDAP目录服务**。

	> 注意：如果启动了启用了用户身份验证但没有定义至少一个用户的RowStore系统，那么您将无法一次关闭系统`snappy-shell shut-down-all`。要创建用户，请参阅使用**BUILTIN身份验证**或**使用LDAP目录服务**。

3. 在启动任何其他RowStore数据存储或访问器之前，使用授权配置启动一个或多个RowStore定位器。
当使用BUILTIN身份验证时，定位器必须定义所有系统用户帐户以及整个分布式系统的身份验证提供程序。当新成员尝试加入分布式系统时，RowStore使用指定的提供者和用户执行相互身份验证。

4. 关闭分布式系统时，请使用已关闭的用户凭据关闭全部。默认情况下，此命令将关闭所有服务器和访问器，但会使独立定位器成员运行。始终至少在分布式系统中运行一个定位器，直到所有数据存储完成关闭。
#### 例 ####

下面gemfirexd.properties条目显示，配置为使用RowStore内置的身份验证RowStore成员：

    snappydata.auth-provider=BUILTIN
    mcast-port=0
该`mcast-port=0`条目表示在RowStore分布式系统中未使用组播。在启动RowStore服务器时需要提供有效的定位器属性，以及**使用BUILTIN身份**验证中所述的RowStore用户的**凭据**。

## 使用BUILTIN身份验证 ##
RowStore BUILTIN身份验证提供程序仅适用于开发和测试。当使用此安全机制时，RowStore系统维护用户名和密码信息的存储库。<br/>

- **BUILTIN用户帐户的类型**

- **创建系统用户和启动RowStore成员** 系统用户在引导时建立，并具有启动RowStore服务器和定位器的权限，并将成员加入现有的分布式系统。

- **更改系统用户密码** 引导RowStore定位器时，将定义内置系统用户。加入系统的其他RowStore成员必须指定在定位器中定义的同一系统用户之一。如果需要更改系统用户的密码，则必须停止分布式系统的所有成员，然后重新启动它们（从定位器开始），在启动时指定新的用户名定义。

- **创建分布式系统用户** 创建用户帐户以保护数据库资源。

## BUILTIN用户帐户的类型 ##
BUILTIN提供商支持两种不同类型的用户帐户：<br/>

- 系统用户帐户对RowStore系统的所有成员都可见，并具有将成员加入集群并关闭集群成员的权限。在引导RowStore服务器或定位器时，使用系统属性建立有效的系统用户列表。您应该仅使用RowStore部署中的几个系统级用户（例如，一个用于独立定位器的系统用户，另一个用于RowStore服务器）。
您可以通过`gemfirexd.user.<UserName>=<password>`使用gemfirexd.properties文件中的属性指定系统用户来创建用户名和密码。请参阅**创建系统用户并启动RowStore成员**。

- 分布式系统用户帐户用于建立与RowStore集群的连接，并使用SQL授权来保护数据库资源。您通过连接到正在运行的RowStore系统并执行内置过程来定义分布式系统用户凭据。然后使用SQL命令授予个别数据库资源的权限。请参阅**创建分布式系统用户**。

> 注意： RowStore内置的认证机制仅适用于开发和测试目的。生产系统应使用LDAP或用户定义的类进行身份验证。生产系统还应使用SSL / TLS来保护网络连接。

创建系统用户并启动RowStore成员

系统用户在启动时建立，具有启动RowStore服务器和定位器的权限，并将成员加入现有的分布式系统。

### 程序 ###



1. 启用BUILTIN身份验证，并使用gemfirexd.user.user-name系统属性在引导时创建一个或多个RowStore系统用户的凭据。RowStore用户名是SQL92标识符，对用户认证是区分大小写的。允许使用分隔标识符。例如，以下属性使用密码“java”定义用户名“FRed”：<br/>


		gemfirexd.auth-provider=BUILTIN
		mcast-port=0
		gemfirexd.user."FRed"=java


	作为最佳做法，将所有系统用户的属性定义包含在单个gemfirexd.properties文件中，可以由分布式系统中的每个RowStore服务器，独立定位器或对等客户端使用。例如，此列表显示了一个gemfirexd.properties文件，定义了“locatoradmin”和“serveradmin”系统用户：<br/>


		gemfirexd.auth-provider=BUILTIN
		mcast-port=0
		gemfirexd.user.locatoradmin=locatorpassword
		gemfirexd.user.serveradmin=serverpassword


	如果RowStore成员不引用共享gemfirexd.properties文件，则必须同时定义并指定在启动该成员时在命令行上加入分布式系统所需的凭据。有关示例，请参阅剩余的步骤。

	> 注意：以上示例使用纯文本密码。作为最佳做法，您应该将密码加密，然后使用下一步中的说明将其存储在gemfirexd.properties中。



1. 为避免在gemfirexd.properties文件中以明文形式存储系统用户密码，请使用该snappy-shell encrypt-password命令为密码生成加密的占位符文本。例如：<br/>


		$ snappy-shell encrypt-password
		Enter User Name: locatoradmin
		Enter Password: locatorpassword
		Re-enter Password: locatorpassword
		Encrypted to v23b60584174091d1d7b3bad3728a55500915424516037
将加密的值存储在gemfirexd.properties文件中，以取代指定用户名的纯文本密码：<br/>

    	gemfirexd.user.locatoradmin=v23b60584174091d1d7b3bad3728a55500915424516037



1. 要引导独立定位器，请使用-user选项指定配置的系统用户名。例如，要使用上一步中所示的示例gemfirexd.properties文件引导定位器，请输入：


		$ snappy-shell locator start -dir=./locator -user=locatoradmin -password
		Enter Password: locatorpassword

	如果将该`-password`选项留空，RowStore将提示您输入密码。<br/>
	定位器在引导时定义内置系统用户（在上述示例中，使用示例gemfirexd.properties文件）。加入系统的其他RowStore成员必须指定为定位器定义的同一系统用户之一。例如，使用相同gemfirexd.properties的RowStore数据存储可以使用以下命令加入系统：

		$ snappy-shell rowstore server start -dir=./server -client-port=1528 -user=serveradmin -password
		Enter Password: serverpassword

	如果需要更改系统用户的密码，则必须停止分布式系统的所有成员，然后重新启动它们（从定位器开始），在启动时指定新的用户名定义。


1. 要使用系统用户帐户将瘦客户端连接到分布式系统，您只需要指定具有`user`和`password`客户端连接属性的有效凭据：


		$ snappy
		snappy> connect client 'localhost:1527;user=serveradmin;password=serverpassword' as server1;
		snappy(SERVER1)>


1. 要显示在当前RowStore成员中配置的BUILTIN用户列表，请使用SYS.SHOW_USERS过程。例如，如果从上一步中启动的访问者成员执行过程，RowStore将显示在连接字符串中定义的单个BUILTIN用户：


		snappy(SERVER1)> call sys.show_users();
		NAME                                                                   |TYPE
		------------------------------------------------------------------------------
		SERVERADMIN                                                            |DBA
		
		1 row selected
如果从上一步中启动的服务器执行该过程，您将看到两个BUILTIN用户都被定义：

		snappy(SERVER1)> call sys.show_users();
		NAME                                                                   |TYPE
		------------------------------------------------------------------------------
		SERVERADMIN                                                            |DBA
		LOCATORADMIN                                                           |USER
		
		2 rows selected
“DBA”类型表示用户是当前RowStore成员的JVM所有者。访问器和服务器均使用SERVERADMIN凭据启动，因此用户是两个成员的JVM所有者。


	> 注意：请注意`gemfirexd.user.username`定义用户凭据，`-user`指定用于引导成员或连接到分布式系统的凭据。当您启动RowStore成员时，这两个属性都是必需的，但作为一个最佳做法，您应该*定义所有BUILTIN用户在一个常见的gemfirexd.properties文件中。

更改系统用户密码<br/>

在引导RowStore定位器时定义内置系统用户。加入系统的其他RowStore成员必须指定在定位器中定义的同一系统用户之一。如果需要更改系统用户的密码，则必须停止分布式系统的所有成员，然后重新启动它们（从定位器开始），在启动时指定新的用户名定义。<br/>

要更改现有系统用户的密码，请在使用`gemfirexd.user.UserName`系统属性启动RowStore成员时指定新密码，或者使用gemfirexd.properties文件生成新的加密密码`snappy-shell encrypt-password`并将新的加密密码置于其中。<br/>

有关详细信息，请参阅**创建系统用户和启动RowStore成员**。<br/>



> 注意：无论使用哪种方法来定义新密码，都必须使用新密码重新启动所有RowStore成员（包括定位器）。您不能仅在RowStore成员的一个子集上更改系统密码用户。此限制仅适用于在BUILTIN认证模式中定义的系统用户。LDAP认证发生在RowStore系统之外，因此可以更改密码，而不影响可用的RowStore成员。

#### 创建分布式系统用户 ####

	$ snappy snappy> connect peer 'locators=localhost[10334];mcast-port=0;user=serveradmin;auth-provider=BUILTIN;gemfirexd.user.serveradmin=serverpassword;password=serverpassword; snappy> call sys.create_user('gemfirexd.user.newuser', 'newpassword'); snappy> disconnect;

**创建一个或多个分布式系统用户帐户后，可以使用这些凭据而不是系统用户凭据连接到RowStore分布式系统：**

	 $ snappy
	snappy> connect client 'localhost:1527;user=newuser;password=newpassword';
**使用带有`GRANT`和`REVOKE`语句的分布式系统用户帐户来管理对数据库资源的访问。请参阅配置用户授权。**

### 使用LDAP目录服务 ###
LDAP（轻量级目录访问协议）提供通过TCP / IP运行的开放目录访问协议。RowStore可以根据企业中现有的LDAP目录服务来验证用户。<br/>

LDAP目录服务可以快速验证用户的名称和密码。LDAP服务提供商的示例包括389目录服务器和OpenLDAP。<br/>

Java开发工具包（JDK）提供的运行时库包括可以访问LDAP目录服务的库。有关Java对Java支持的更多信息，请参阅：<br/>

- **javax.naming.ldap包** 的API文档[http://download.oracle.com/javase/6/docs/api/](http://download.oracle.com/javase/6/docs/api/)

- **JNDI教程的LDAP部分** [http://download.oracle.com/javase/tutorial/jndi/ldap/](http://download.oracle.com/javase/tutorial/jndi/ldap/)

- **JNDI规范的LDAP部分** [http://download.oracle.com/javase/1.5.0/docs/guide/jndi/spec/jndi/jndi.5.html#pgfId=999241](http://download.oracle.com/javase/1.5.0/docs/guide/jndi/spec/jndi/jndi.5.html#pgfId=999241)

- **为LDAP** 配置RowStore将RowStore配置为使用LDAP作为身份验证服务时，必须指定要使用的LDAP服务器。

- **配置RowStore以搜索DN** 默认情况下，RowStore在LDAP中启动搜索以获取简单用户名的完整DN。您可以根据您的LDAP服务器是否支持匿名搜索，根据需要配置RowStore搜索行为。

#### 为LDAP配置RowStore ####
配置RowStore以使用LDAP作为身份验证服务时，必须指定要使用的LDAP服务器。<br/>

-** 限制和效能指南**

- **程序**

##### 限制和效能指南 #####

- RowStore不支持LDAP组。

- 出于性能原因，LDAP目录服务器应与RowStore位于同一LAN中。RowStore不会在本地缓存用户的凭证信息，因此每次用户连接时都必须连接到目录服务器。

- 提供完整DN的连接请求比那些必须搜索完整DN的连接请求更快。

##### 程序 #####


1. 在RowStore分布式系统中启动每个定位器和服务器时，将auth-provider属性设置为“LDAP”。
将auth-provider属性设置为“LDAP”时，RowStore使用LDAP来验证分布式系统的所有分布式系统成员以及客户端。因此，RowStore成员必须在启动时提供用户和密码属性。如果在启动RowStore成员时省略了password属性的值，则该成员将在命令行中提示输入密码。


1. 将snappydata.auth-ldap-server属性设置为LDAP服务器的URL。例如：


		snappydata.auth-ldap-server=ldap://server:port/
您可以指定仅具有服务器名称的LDAP服务器，其端口号由冒号分隔，也可以指定为“ldap”URL。如果没有提供完整的URL，则RowStore默认使用未加密的LDAP。要使用SSL加密的LDAP，您必须提供以“ldaps：//”开头的URL。


	> 注意：您必须将此属性指定为Java系统属性。例如，当您使用`snappy-shell`启动一个新的RowStore服务器时，使用命令行选项`-J-Dsnappydata.auth-ldap-server = ldaps：// server：port /`指定属性。



1. 如果使用SSL加密的LDAP，并且您的LDAP服务器证书无法由有效的证书颁发机构（CA）识别，请为每个RowStore成员创建本地信任存储，并将LDAP服务器证书导入信任存储。看到[http://docs.oracle.com/javase/6/docs/technotes/guides/security/jsse/JSSERefGuide.html#CreateKeystore](http://docs.oracle.com/javase/6/docs/technotes/guides/security/jsse/JSSERefGuide.html#CreateKeystore)了解更多信息。


1. 如果执行步骤3，则在启动各个RowStore成员时指定`javax.net.ssl.trustStore`和`javax.net.ssl.trustStorePassword`系统属性。例如：


		snappy-shell rowstore server start -dir=./server -locators=localhost[10101] -client-port=1528 -auth-provider=LDAP \
		                   -J-Dsnappydata.auth-ldap-server=ldaps://ldapserver:636/ -user=user_name -password=user_pwd \
		                   -J-Dsnappydata.auth-ldap-search-dn=uid=gemfirexd1,ou=ldapExample,dc=gemstone,dc=com  \
		                   -J-Dsnappydata.auth-ldap-search-pw=gemfirexd1 \
		                   -J-Dsnappydata.auth-ldap-search-base=ou=ldapTesting,dc=gemstone,dc=com \
		                   -J-Djavax.net.ssl.trustStore=/Users/yozie/snappy-store/keystore_name \
		                   -J-Djavax.net.ssl.trustStorePassword=keystore_password


	> 注意： `javax.net.ssl.trustStore`和`javax.net.ssl.trustStorePassword`必须指定为Java系统属性（使用`snappy-shell`命令行上的-J选项）。

注意：对于RowStore分布式系统的每个成员，LDAP服务器和搜索属性必须设置为相同的值。但是，可以使用不同的身份验证的用户凭据，信任存储等启动单个RowStore成员。

#### 配置RowStore以搜索DN ####
默认情况下，RowStore在LDAP中启动搜索以获取简单用户名的完整DN。您可以根据您的LDAP服务器是否支持匿名搜索，根据需要配置RowStore搜索行为。<br/>

- **了解LDAP搜索行为**

- **配置LDAP搜索的属性**

- **示例LDAP搜索配置**

##### 了解LDAP搜索行为 #####
在LDAP系统中，用户按目录中的一组条目分层组织。一个条目是由唯一名称（称为DN（可分辨名称））标识的一组名称 - 属性对。一个条目被DN明确地标识，这是从沿着从根向下到命名条目的路径从树中的每个条目选择的属性的级联，从右到左排序。例如，用户的DN可能如下所示：

	cn=mary,ou=People,dc=example,dc=com
	
	uid=mary,ou=People,dc=example,dc=com
名称的允许条目由条目的objectClass定义。<br/>

如果LDAP客户端提供用户ID和密码，则LDAP客户端可以绑定到目录（成功登录）。用户ID必须是DN，名称和属性的完全限定列表。这意味着用户必须提供一个很长的名字。<br/>

通常，用户只知道一个简单的用户名（例如，上面的DN的第一部分，mary）。使用RowStore，用户不需要指定完整的DN，因为LDAP客户端（RowStore本身）可以作为访客或匿名用户首先访问目录，搜索完整的DN，然后使用完整DN重新绑定到目录验证用户。<br/>

默认情况下，RowStore会在绑定到目录并使用完整的DN进行用户身份验证之前启动搜索完整的DN。只有在以下情况下，RowStore才会启动搜索：<br/>

- 当snappydata.auth-ldap-search-filter属性设置为snappydata.user时。

- 当用户DN通过snappydata.user本地缓存到特定用户时。UserName **属性。

##### 配置LDAP搜索的属性 #####
某些系统允许进行匿名搜索，而其他系统则需要用户DN和密码。您可以通过配置下面描述的属性来指定要用于搜索的特定DN和密码。此外，您可以通过使用下面描述的属性来指定过滤器（用户定义的对象类）和基础（从中开始搜索的目录）来限制搜索的范围。<br/>


> 注意：引导RowStore对等体时，必须将以下每个属性指定为系统属性。例如，`snappy-shell`使用命令行选项-J-Dsnappydata.auth-ldap-search-base = searchbase引导新的RowStore服务器时。



- snappydata.auth-ldap-search-dn指定在搜索用户DN时与服务器绑定（认证）的DN。如果您的LDAP提供商不支持匿名绑定，则此属性是必需的。如果您的LDAP提供商支持匿名绑定，那么此参数是可选的。
如果配置此属性，该值必须是目录服务识别的有效DN，并且DN必须具有搜索条目的权限。如果不配置此属性，则默认使用Snappydata.auth-ldap-search-base属性指定的根DN进行匿名搜索。例如：

		uid=guest,dc=example,dc=com


- snappydata.auth-ldap-search-pw指定snappydata.auth-ldap-search-dn中指定的访客用户DN的密码。如果不配置此属性，则RowStore默认使用匿名搜索与snappydata.auth-ldap-search-base属性指定的根DN 。


- snappydata.auth-ldap-search-base指定从层次结构中开始对用户DN的来宾搜索的点的根DN。例如：
ou=people,dc=example,dc=com
要缩小搜索范围，您可以指定用户的objectClass。


	> 注意：使用Netscape Directory Server时，可以将此属性设置为根DN，这是不适用访问控制的特殊条目。

如果您配置snappydata.auth-ldap-search-base，则LDAP服务器必须支持匿名绑定，或者必须配置

- snappydata.auth-ldap-search-dn和snappydata.auth-ldap-search-pw以提供DN用于搜索条目。
snappydata.auth-ldap-search-filter指定描述什么构成LDAP目录服务的用户的逻辑表达式。此属性的默认值为objectClass=inetOrgPerson。如果%USERNAME%在过滤器定义中包含令牌，则RowStore将使用要验证的用户名替换令牌。如果您不提供搜索过滤器，则RowStore将使用默认搜索过滤器：

		(&(objectClass=inetOrgPerson)(uid=%USERNAME%))
如果您提供没有%USERNAME%令牌的搜索过滤器，则RowStore将令牌添加到默认搜索过滤器。例如：

		(&(<provided_filter>)(objectClass=inetOrgPerson)(uid=%USERNAME%))

##### 示例LDAP搜索配置 #####
考虑使用OpenLDAP ldapsearch工具调用以下LDAP搜索：

	ldapsearch -b ou=users,dc=domain,dc=com /* base DN */ 
	     -x /* non-SASL plain-text authentication */ 
	     -D uid=test,ou=ldapTesting,dc=domain,dc=com /* bind DN */ 
	     -w test /* bind password */ 
	     "(&(objectClass=user)(uid=user1))" /* filter */
要使用RowStore配置上述搜索，您将定义属性：

	snappydata.auth-ldap-search-base=ou=users,dc=domain,dc=com
	snappydata.auth-ldap-search-filter=(&(objectClass=user)(uid=%USERNAME%))
	snappydata.auth-ldap-search-dn=uid=test,ou=ldapTesting,dc=domain,dc=com
	snappydata.auth-ldap-search-pw=test

### 外部目录服务的JNDI特定属性 ###
RowStore允许您设置一些高级JNDI属性，您可以通过任何支持的设置RowStore属性的方式进行设置。通常，您可以在配置外部身份验证服务的系统级别进行设置。<br/>

支持的属性列表可以在附录A：Java命名和目录API中的JNDI标准环境属性中找到 [http://download.oracle.com/javase/1.5.0/docs/guide/jndi/spec/jndi/properties.html](http://download.oracle.com/javase/1.5.0/docs/guide/jndi/spec/jndi/properties.html)。外部目录服务必须支持该属性。<br/>

每个JNDI提供程序都具有您可以在RowStore系统中设置的一组属性。<br/>

例如，您可以设置属性java.naming.security.authentication以允许用户凭据在网络上加密，如果提供程序支持它。您还可以指定SSL与LDAP（LDAPS）一起使用。<br/>

## 配置用户授权 ##
当您启用用户授权时，RowStore会验证已授予用户访问模式，数据库对象或SQL操作的权限。

- 连接授权和SQL标准授权

- 用户授权属性

- 更改连接授权设置

### 连接授权和SQL标准授权 ###
RowStore有两种类型的用户授权：连接授权和SQL标准授权。连接授权指定用户在连接到分布式系统时具有的基本访问权限。SQL授权控制用户对数据库对象或SQL操作的权限。您可以在RowStore中将用户授权属性设置为系统级属性，无论是在引导RowStore成员时还是在命令行或连接字符串中，或​​在gemfirexd.properties文件中。<br/>


### 用户授权属性 ###
您可以设置属性来控制RowStore的用户授权。某些属性设置所有用户的默认访问模式。其他属性设置特定用户ID的默认访问级别。<br/>

影响授权的属性有：<br/>



- snappydata.authz-default-connection-mode为所有用户设置访问模式，覆盖您可能使用GRANT语句授予的任何细粒度权限。仅当您要覆盖所有用户的访问模式时才配置此属性。

- `snappydata.authz-full-access-users`和 `- snappydata.authz-read-only-access-users`这些属性指定一个或多个对整个分布式系统具有读写访问权限和只读访问权限的用户标识。

	> 注意：`snappydata.authz-full-access-users`在启动RowStore时，必须在命令行中定义访问列表属性，而不是在gemfirexd.properties文件中。

- `snappydata.sql-authorization` - 启用SQL标准授权。使用`snappydata.sql-authorization`控制对象所有者是否能授予和撤销许可的其他用户执行自己的数据库对象的SQL操作。的默认设置`snappydata.sql-authorization`为FALSE。但是，如果您使用gfxd启动RowStore成员，并且包含`-auth-provider`指定客户端身份验证机制的选项，则默认情况下将启用SQL授权。当SQL授权时，对象所有者可以使用GRANT和REVOKE SQL语句来设置特定数据库对象或特定SQL操作的用户权限。
如果不为特定用户标识配置用户授权，则用户ID将继承任何授予设置为RowStore member（`snappydata.authz-default-connection-mode`）的默认用户权限的权限。

提示：如果将`snappydata.authz-default-connection-mode`属性设置为noAccess或readOnlyAccess，则应至少允许一个用户读写访问。否则，根据您指定的默认连接授权，您的系统可能包含无法访问或更改的数据库对象。您必须通过`snappydata.authz-full-access-users=username`在启动RowStore时在命令行上指定用户具有访问权限; 您无法在gemfirexd.properties中定义属性。


### 用户授权属性如何协同工作 ###
该`snappydata.authz-default-connection-mode`和`snappydata.sql-authorization`性能一起工作。这些属性的默认设置允许任何人访问和删除他们创建的数据库对象。您可以通过为这些属性指定不同的设置来更改默认访问模式。




- 当`snappydata.sql-authorization`属性为FALSE时，从数据库对象读取或写入的能力由`snappydata.authz-default-connection-mode`属性的设置确定。如果`snappydata.authz-default-connection-mode`设置为readOnlyAccess，用户可以访问所有数据库对象，但不能更新或删除这些对象。

- 当`snappydata.sql-authorization`为TRUE时，从数据库对象读取或写入数据库对象的能力最初仅限于这些数据库
对象的所有者。所有者必须明确授予他人访问数据库对象的权限。没有人，但对象或JVM所有者的所有者可以删除该对象。

- 为该`snappydata.authz-default-connection-mode`属性指定的访问模式将覆盖由数据库对象的所有者授予的权限。例如，如果用户被授予对表的INSERT权限，但用户只具有只读连接授权，则用户无法将数据插入到表中。

### 更改连接授权设置 ###
连接授权属性在连接期间是固定的。建立新连接以更改授权属性。<br/>

- **设置SQL标准授权模式** 使用该`snappydata.sql-authorization`属性启用RowStore标准授权。

- **设置默认连接访问模式** 使用该`snappydata.authz-default-connection-mode`属性可指定用户在连接到分布式系统时具有的默认访问类型。

- **设置个人用户的授权** 使用`snappydata.authz-full-access-users`和`snappydata.authz-read-only-access-users`属性可以指定对分布式系统具有读写访问权限和只读访问权限的用户标识。

- **只读和完全访问权限** 用户可以对数据库对象执行的操作由用户具有的访问类型决定。

- **用户授权异常** 当用户授权发生错误时，将返回SQL异常。

- **在SQL标准授权中设置数据库对象的用户权限** 启用S​​QL标准授权模式时，对象所有者可以使用GRANT和REVOKE SQL语句来设置特定数据库对象或特定SQL操作的用户权限。

### 设置SQL标准授权模式 ###
使用该`snappydata.sql-authorization`属性启用RowStore标准授权。<br/>

该`snappydata.sql-authorization`属性控制对象所有者授予和撤销权限的能力，以便用户对其创建的数据库对象执行操作。<br/>

该`snappydata.sql-authorization`属性的有效设置有：<br/>

- 真正

- 假


`snappydata.sql-authorization`属性的默认设置为FALSE。<br/>

将`snappydata.sql-authorization`属性设置为TRUE后，您无法将属性设置为FALSE。<br/>

启用此属性时，所有新的数据库模式都启用了SQL授权。<br/>

要为整个系统启用SQL标准授权，请将该`snappydata.sql-authorization`属性设置为系统属性。例如，将此行添加到gemfirexd.properties中：<br/>

    snappydata.sql-authorization=true

### 设置默认连接访问模式 ###
使用该`snappydata.authz-default-connection-mode`属性指定用户在连接到分布式系统时具有的默认访问类型。<br/>

该`snappydata.authz-default-connection-mode`属性的有效设置有：
<br/>
- noAccess 提示：如果将`snappydata.authz-default-connection-mode`属性设置为noAccess或readOnlyAccess，则应至少允许一个用户读写访问。否则，根据您指定的默认连接授权，您的系统可能包含无法访问或更改的数据库对象。您必须通过`snappydata.authz-full-access-users=username`在启动RowStore时在命令行上指定用户具有访问权限; 您无法在gemfirexd.properties中定义属性。

- readOnlyAccess

- fullAccess

如果不指定`snappydata.authz-default-connection-mode`属性的设置，默认访问设置为fullAccess。<br/>

要设置默认连接访问模式，请在gemfirexd.properties中指定系统属性，或在引导RowStore成员时指定。<br/>

### 设置个人用户的授权 ###
使用`snappydata.authz-full-access-users`和`snappydata.authz-read-only-access-users`属性指定对分布式系统具有读写访问权限和只读访问权限的用户标识。<br/>

您可以使用逗号分隔的列表指定多个用户标识，逗号和下一个用户标识之间没有空格。<br/>

在引导RowStore成员时，在命令行上定义这些属性，而不是在gemfirexd.properties中指定它们。<br/>

### 只读和完全访问权限 ###
用户可以对数据库对象执行的操作由用户具有的访问类型决定。

下表列出了用户可以根据用户授予的访问类型执行的操作。

|行动|只读访问|完全访问|
|:--|:--|:--|
|执行SELECT语句|X|X|
|执行INSERT，UPDATE或DELETE语句||X|
|执行DDL语句||X|
|添加或替换jar文件||X|

：表1.通过访问类型授权的操作

### 用户授权异常 ###
当用户授权发生错误时，将返回SQL异常。<br/>

RowStore在设置属性时验证数据库属性。如果在设置这些属性时指定了无效值，则返回异常。<br/>

将`snappydata.sql-authorization`属性设置为TRUE后，您无法将属性设置为FALSE。<br/>

如果用户尝试连接到分布式系统但未被授权连接，则返回SQLException 04501。<br/>

如果具有只读访问权限的用户尝试写入系统，`- connection refused`则返回SQLException 08004 。<br/>

### 在SQL标准授权中设置数据库对象的用户权限 ###
启用S​​QL标准授权模式时，对象所有者可以使用GRANT和REVOKE SQL语句来设置特定数据库对象或特定SQL操作的用户权限。<br/>

SQL标准授权模式是SQL2003兼容的访问控制系统。通过将`snappydata.sql-authorization`属性设置为，启用SQL标准授权模式`TRUE`。<br/>

虽然RowStore具有更简单的数据库访问模式，可以设置为向用户提供完整，只读或无访问权限，但这种更简单的访问模式对于大多数客户端 - 服务器数据库配置不太适用。当用户或应用程序直接针对数据库发出SQL语句时，SQL授权模式提供了更精确的机制来限制用户可以对数据库执行的操作。<br/>

- GRANT和REVOKE特权

- 设置公共和个人用户权限

- 设置视图，触发器和约束的权限

#### GRANT和REVOKE特权 ####
GRANT语句用于向用户授予特定权限。REVOKE语句用于撤销权限。授予和撤销权限是：<br/>

- 删除
- 执行
- 插
- 选择
- 参考
- 触发
- UPDATE
当创建表，视图，函数或过程时，创建对象的人称为对象的所有者。只有对象所有者和JVM所有者拥有该对象的完整权限。没有其他用户对对象拥有权限，直到对象所有者向其授予权限。<br/>


#### 设置公共和个人用户权限 ####
对象所有者可以授予和撤销特定用户或所有用户的权限。关键字PUBLIC用于指定所有用户。指定PUBLIC时，权限会影响当前和未来的所有用户。向PUBLIC和个人用户授予和撤销的特权是独立的。例如，表t上的SELECT 权限授予PUBLIC和用户harry。SELECT权限随后从用户撤销harry，但用户harry可以t通过PUBLIC 权限访问表。<br/>

> 注意：创建视图，触发器或约束时，RowStore首先检查以确定在用户级别是否具有所需的权限。如果您具有用户级权限，则会创建该对象，并且取决于该用户级别的权限。如果您在用户级别没有必需的权限，RowStore将检查您是否在PUBLIC级别具有所需的权限。如果您具有PUBLIC级别权限，则会创建该对象，并且取决于该PUBLIC级别的权限。创建对象后，如果对象所依赖的权限被撤销，则该对象将被自动删除。RowStore不会尝试确定是否有其他权限可以替换正在被撤销的权限。 示例1用户`zhi`创建表't1'，并向表't1'上的用户`harry`授予SELECT权限。用户`zhi`在TABLE`t1上给PUBLIC授予SELECT权限。用户`harry`使用`zhi.t1`的语句SELECT \ *创建视图`v1`。视图取决于用户`harry`在`t1'上的用户级权限。随后，用户`zhi`从表't1'上撤消用户`harry`的SELECT权限。结果，视图`harry.v1`被丢弃。示例2用户`anita`创建表't1'，并向PUBLIC授予SELECT权限。用户`harry`使用`anita.t1`的语句SELECT \ *创建视图`v1`。视图取决于用户`harry`在`t1'上的PUBLIC级别权限，因为当用户'harry`创建'harry.v1'视图时，``````没有用户级权限。后来，用户`anita`向表`anita.t1`授予用户`harry`的SELECT权限。视图`harry.v1`继续依赖于用户'harry`在`t1'上的PUBLIC级别权限。当用户`anita`从表't1上撤销PUBLIC中的SELECT特权时，视图`harry.v1`将被删除。


#### 设置视图，触发器和约束的权限 ####
视图，触发器和约束使用视图，触发器或约束的所有者的权限操作。例如，用户anita希望使用以下语句创建视图：<br/>

	CREATE VIEW s.v(vc1,vc2,vc3)
	    AS SELECT t1.c1,t1.c2,f(t1.c3)
	  FROM t1 JOIN t2 ON t1.c1 = t2.c1 
	    WHERE t2.c2 = 5
用户`anita`需要以下权限才能创建视图：<br/>

- 模式的所有权`s`，以便她可以在模式中创建一些内容
- 表的所​​有权t1，让她可以让别人看到表中的列
- 列`t2.c1`和列的`SELECT`权限`t2.c2`
- EXECUTE权限功能 `f`
创建视图时，只有用户`anita`对其进行了`SELECT`权限。用户`anita`可以在任何或所有视图`s.v`的列授予`SELECT`权限`s.v`给任何人，甚至没有`SELECT`权限的用户t1或t2，或在EXECUTE权限f。用户`anita`授予对用户的视图的`SELECT` 权限`harry`。当用户`harry`在视图上发出SELECT语句时`s.v`，RowStore会检查以确定用户`harry`是否具有视图的`SELECT`权限`s.v`。RowStore不检查，以确定用户`harry`对具有`SELECT`权限t1，或者`t2`，或者EXECUTE权限f。

触发器和约束的权限与视图权限相同。当创建视图，触发器或约束时，RowStore会检查所有者是否具有所需的权限。其他用户不需要具有这些权限来对视图，触发器或约束执行操作。<br/>

如果从视图，触发器或约束的所有者撤销所需的权限，则该对象将作为REVOKE语句的一部分被删除。<br/>

另见**GRANT**和**REVOKE**。

## 使用SSL / TLS配置网络加密和身份验证 ##
默认情况下，所有RowStore网络流量都是未加密的，用户名和用户密码除外，可以单独加密。还没有网络层访问控制机制。对于可能存在安全问题的部署场景，RowStore网络服务器通过安全套接字层/传输层安全性（SSL / TLS）支持网络安全。<br/>

使用SSL / TLS，客户机/服务器通信协议是加密的，彼此独立，客户端和服务器都可能需要基于证书的认证。<br/>

假设读者熟悉SSL，密钥对和证书。本文档还基于Java开发工具包（JDK）及其应用`keytoo`l程序。<br/>

对于本节的其余部分，术语SSL用于SSL / TLS，术语对等体用于通信的其他部分（服务器的对等体是客户端，反之亦然）。<br/>

RowStore的SSL（用于客户端和服务器）以三种可能的模式运行： off<br/>
默认情况下，不进行SSL加密

#### 基本的 ####
SSL加密，无对等认证<br/>

peerAuthentication<br/>
SSL加密和对等认证<br/>

您可以在服务器或客户端或两者上设置对等体身份验证。对等体验证意味着SSL连接的另一端基于本地安装的受信任证书进行身份验证。<br/>

或者，您可以在本地安装证书颁发机构（CA）证书，并且对等体具有该权限签署的证书。如何实现这一点在本文档中没有描述。有关详细信息，请参阅Java环境文档。<br/>

> 注意： 如果明文客户端尝试与SSL服务器进行通信，或者SSL客户端尝试与明文服务器进行通信，通信的明文方面将看到SSL通信为噪声和报告协议错误。

- **生成键对和证书** 对于SSL操作，服务器始终需要一个密钥对。一般来说，对于通信的一端来验证其伙伴，第一端需要安装合作伙伴生成的证书。

- **使用SSL** / TLS为客户端连接启动服务器 当您启动一个RowStore成员时，您可以使用系统属性`snappydata.drda.sslMode`（默认关闭）为客户端配置提供SSL。

- **使用SSL** / TLS连接客户端使用 属性`sslconnection`属性为客户端连接启用SSL加密。

### 生成键对和证书 ###
对于SSL操作，服务器总是需要一个密钥对。一般来说，对于通信的一端来验证其伙伴，第一端需要安装合作伙伴生成的证书。<br/>

如果服务器以对等体身份验证模式运行（服务器认证客户端），则每个客户端都需要自己的密钥对。密钥对位于称为密钥库的文件中，JDK的SSL提供者需要系统属性`javax.net.ssl.keyStore`并`javax.net.ssl.keyStorePassword`访问密钥存储。<br/>

受信任方的证书安装在称为信任存储的文件中。JDK的SSL提供程序需要系统属性`javax.net.ssl.trustStore`并`javax.net.ssl.trustStorePassword`访问信任存储。<br/>


#### 生成键对 ####
密钥对生成keytool -genkey。生成密钥对的最简单的方法是执行以下操作：<br/>

    keytool -genkey <alias> -keystore <keystore>
`keytool` 提示需要的信息，如身份信息和密码。

例如，生成服务器密钥对：

	keytool -genkey -alias myRowStoreServer -keystore serverKeyStore.key
生成客户端密钥对：

	keytool -genkey -alias aRowStoreClient -keystore clientKeyStore.key
有关更多信息，请参阅JDK文档keytool。


#### 生成证书 ####
生成证书keytool -export如下：

	keytool -export -alias <alias> -keystore <keystore> \
        -rfc -file <certificate file>
例如，生成服务器证书：

	keytool -export -alias myRowStoreServer -keystore serverKeyStore.key \
        -rfc -file myServer.cert
生成客户端证书：

	keytool -export -alias aRowStoreClient -keystore clientKeyStore.key \
        -rfc -file aClient.cert
然后证书文件可以分发给相关方。


#### 在信托商店安装证书 ####
在信任存储中安装证书，keytool -import具体如下：

	keytool -import -alias <alias> -file <certificate file> \
        -keystore <trust store>
在服务器的信任存储中安装客户端证书：

	keytool -import -alias aRowStoreClient -file aClient.cert 
        -keystore serverTrustStore.key
将服务器证书安装在客户端的信任存储中：

	keytool -import -alias myRowStoreServer -file myServer.cert 
        -keystore clientTrustStore.key

### 使用SSL / TLS启动客户端连接的服务器 ###
当您启动一个RowStore成员时，您可以使用系统属`性snappydata.drda.sslMode`（默认关闭）为客户端启用SSL 。<br/>

对于服务器SSL / TLS，需要生成服务器密钥对。如果服务器要进行客户端认证，则客户端证书需要安装在信任库中。这些操作在生成密钥对和证书中进行了说明。

- 使用基本SSL加密启动服务器
- 启动认证客户端的服务器

#### 使用基本SSL加密启动服务器 ####
当SSL模式设置为`basic`，服务器只接受来自客户端的SSL加密连接。<br/>

系统属性`javax.net.ssl.keyStore，javax.net.ssl.keyStorePassword`需要设置服务器的正确值。

#### 例 ####
	snappy-shell rowstore server start -J-Djavax.net.ssl.keyStore=serverKeyStore.key \
     -J-Djavax.net.ssl.keyStorePassword=qwerty \
     -J-Dsnappydata.drda.sslMode=basic

#### 启动认证客户端的服务器 ####
当服务器的SSL模式设置为`peerAuthentication`，服务器会对其客户端的身份进行身份验证，并加密网络流量。在这种情况下，服务器的信任存储必须包含要连接的每个客户端的证书。

在`javax.net.ssl.trustStore`和`javax.net.ssl.trustStorePassword`需要，除了上述的属性进行设置。

#### 例 ####
	snappy-shell rowstore server start -J-Djavax.net.ssl.keyStore=serverKeyStore.key \
     -J-Djavax.net.ssl.keyStorePassword=qwerty \
     -J-Djavax.net.ssl.trustStore=serverTrustStore.key \
     -J-Djavax.net.ssl.trustStorePassword=qwerty \
     -J-Dsnappydata.drda.sslMode=peerAuthentication

### 使用SSL / TLS连接客户端 ###
您可以使用属性`sslconnection`属性为客户端连接启用SSL加密。<br/>

将属性设置为“基本”以加密连接。例如，使用基本的SSL与`snappy-shell`瘦客户机连接进行连接：<br/>

	snappy> connect client 'myhost:1527;ssl=basic';
或者，要在Java应用程序中连接：

	Connection c = 
	   getConnection("jdbc:snappydata://myhost:1527/db;ssl=basic");

#### 运行验证服务器的客户端 ####
要使客户端与服务器进行身份验证，客户端的信任存储必须包含服务器的证书。<br/>

您可以通过设置URL属性ssl或属性ssl来启用具有服务器身份验证的客户端SSL peerAuthentication。此外，系统属性`javax.net.ssl.trustStore`和`javax.net.ssl.trustStorePassword`需要进行设置。<br/>

例
    System.setProperty("javax.net.ssl.trustStore","clientTrustStore.key");
    System.setProperty("javax.net.ssl.trustStorePassword","qwerty");
    Connection c = 
       getConnection("jdbc:snappydata://myhost:1527/db;ssl=peerAuthentication");

当服务器进行客户端身份验证时运行客户端
如果服务器对客户端进行身份验证，则客户端需要一个密钥对和安装在服务器的信任存储中的客户端证书。请参阅**生成密钥对和证书**。

客户端需要设置`javax.net.ssl.keyStore`和`javax.net.ssl.keyStorePassword`。

例
    System.setProperty("javax.net.ssl.keyStore","clientKeyStore.key");
    System.setProperty("javax.net.ssl.keyStorePassword","qwerty");
    Connection c = 
       getConnection("jdbc:snappydata://myhost:1527/db;ssl=basic");

### 为RowStore服务器连接配置SSL ###
除了使用SSL进行客户端/服务器连接外，您还可以选择配置RowStore成员，以便在分布式系统中对对等连接使用SSL加密和/或授权。<br/>

服务器SSL配置使用`javax.net.ssl`系统属性和RowStore引导属性ssl-enabled，ssl-protocols，ssl-ciphers和ssl-require-authentication进行管理。以下部分提供了一个简单的示例，演示如何使用SSL配置和启动RowStore成员。<br/>

#### 要求 ####
为了配置RowStore服务器连接的SSL：

- 您必须使用定位器进行成员发现（mcast-port = 0）。
- 根据需要为每个RowStore成员配置SSL密钥对和证书。请参阅**生成密钥对和证书**。
- 所有RowStore成员必须在启动时使用相同的SSL引导参数。

#### 提供者特定的配置文件 ####
此示例使用Java应用程序创建的keytool密钥库为提供程序提供正确的凭据。要创建定位器的密钥库和证书，我们运行以下操作：

	$ keytool -genkey -alias myRowStoreLocator -keystore locatorKeyStore.key
	$ keytool -export -alias myRowStoreLocator -keystore locatorKeyStore.key -rfc -file myLocator.cert
类似的命令用于服务器成员：

	$ keytool -genkey -alias myRowStoreServer -keystore serverKeyStore.key
	$ keytool -export -alias myRowStoreServer -keystore serverKeyStore.key -rfc -file myServer.cert
然后将每个成员的证书导入到另一个成员的信任商店。对于服务器成员：

	$ keytool -import -alias myRowStoreLocator -file ./myLocator.cert -keystore ./serverKeyStore.key
对于定位器成员：

	keytool -import -alias myRowStoreServer -file ./myServer.cert -keystore ./locatorKeyStore.key

#### gemfirexd.properties文件 ####
您可以为`gemfirexd.properties`文件中的服务器连接启用SSL加密。此示例简单地启用SSL以实现成员之间的通信。定位器和服务器成员都可以使用相同的属性文件：

	ssl-enabled=true
	mcast-port=0
	locators=<hostaddress>[<port>]

#### 定位启动 ####
在启动其他系统成员之前，请使用SSL和提供商特定的配置设置启动定位器：

	$ snappy-shell locator start -J-Djavax.net.ssl.keyStoreType=jks -J-Djavax.net.ssl.keyStore=./locatorKeyStore.key -J-Djavax.net.ssl.keyStorePassword=password \
	-J-Djavax.net.ssl.trustStore=./locatorKeyStore.key -J-Djavax.net.ssl.trustStorePassword=password 

#### 其他会员启动 ####
RowStore服务器可以与定位器启动类似，并`gemfirexd.properties`在当前工作目录中使用相应的文件。例如：

	$ snappy-shell rowstore server start -J-Djavax.net.ssl.keyStoreType=jks -J-Djavax.net.ssl.keyStore=./serverKeyStore.key -J-Djavax.net.ssl.keyStorePassword=password \
	-J-Djavax.net.ssl.trustStore=./serverKeyStore.key -J-Djavax.net.ssl.trustStorePassword=password -client-port=1528

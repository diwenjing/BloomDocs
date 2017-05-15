#RowStore支持的配置和系统要求
##本主题介绍了RowStore支持的配置和系统要求。

###在安装RowStore之前，请确保您的系统满足安装和运行产品的最低系统要求。

###独立产品支持
###支持的配置
###主机要求
###增加Linux平台上的单播缓冲区大小
###在Linux平台上禁用SYN Cookie
###Linux平台的文件系统类型
###客户要求

###独立产品支持
###RowStore 1.5仅经过认证才能独立使用。
###支持的配置
下表显示了RowStore支持的所有配置。<br/>

操作系统   	处理器架构	      JVM	          生产或开发者支持<br/>
红帽  EL 5，6.2，6.4	x86（64bit）	
Java SE 1.7.0_72   生产<br/>
CentOS 6.2,6.4,7	x86（64bit）	
Java SE 1.7.0_72     生产<br/>
Ubuntu 14.04	x86（64位和32位）	
Java SE 1.7.0_72     生产<br/>
SUSE Linux Enterprise Server 11	x86（64位和32位）	
Java SE 1.7.0_72    生产<br/>
Ubuntu 10.11	x86（64位）	
Java SE 1.7.0_72     开发人员<br/>
注意： RowStore产品下载不包括Java; 您必须为系统下载并安装支持的JDK。<br/>


###主机要求
每个主机的要求：<br/>

支持的Java SE安装。<br/>
支持长文件名的文件系统。<br/>
足够的每用户配额的文件句柄（ulimit for Linux）。确保为您的系统设置足够高的ulimit（达到81,920的极限）。<br/>
TCP / IP。<br/>
系统时钟设置为正确的时间。<br/>
对于每个Linux主机，必须正确配置主机名和主机文件。请参阅主机名和主机的系统联机帮助页。<br/>
对于每个Linux主机，将交换卷配置为与安装在计算机中的物理RAM相同的大小。<br/>
时间同步服务，如网络时间协议（NTP）。<br/>
注意：对于故障排除，必须在所有主机上运行时间同步服务。同步时间戳可以让您合并来自不同主机的日志消息，以便分布式运行的精确时间顺序。<br/>

注意：如果将RowStore部署到虚拟化主机，请参阅在虚拟化环境中运行RowStore。<br/>

外部负载平衡，HA解决方案不支持<br/>
RowStore不支持在客户端和RowStore分布式系统之间或分布式系统成员之间使用外部负载平衡技术。这包括软件和硬件负载平衡器，虚拟IP地址（VIP）和代理。同样，不支持外部HA技术，如Linux-HA。<br/>


增加Linux平台上的单播缓冲区大小<br/>
在Linux平台上，以root用户身份执行以下命令来增加单播缓冲区大小：<br/>

编辑/etc/sysctl.conf文件以包括以下行：<br/>
net.core.rmem_max=1048576<br/>
net.core.wmem_max=1048576<br/>
重新加载sysctl.conf：<br/>
sysctl -p<br/>

###在Linux平台上禁用SYN Cookie<br/>
许多默认的Linux安装使用SYN cookie来保护系统免遭洪泛TCP SYN数据包的恶意攻击。使用SYN Cookie可显着降低网络带宽，并可由运行的RowStore分布式系统触发。<br/>

如果您的RowStore分布式系统受到此类攻击的其他保护，则禁用SYN Cookie以确保RowStore网络吞吐量不受影响。<br/>

要永久禁用SYN Cookie：<br/>

编辑/etc/sysctl.conf文件以包括以下行：<br/>
<code>
net.ipv4.tcp_syncookies = 0
</code>
<br/>
将此值设置为零将禁用SYN Cookie。<br/>
重新加载sysctl.conf：<br/>
<code>sysctl -p</code><br/>

###Linux平台的文件系统类型
为了获得最佳磁盘存储性能，Pivotal建议在Linux或Solaris平台上运行时使用ext4文件系统。请参阅使用磁盘存储优化系统。<br/>

###客户要求
RowStore支持两个JDBC驱动程序：瘦客户端JDBC驱动程序和对等JDBC驱动程序。只有Java SE 6才支持RowStore服务器实例和对等驱动程序。您可以从中下载Javahttp://www.oracle.com/technetwork/java/javase/downloads/index.htm。<br/>

RowStore提供了可用于开发非Java客户端应用程序的有线协议ADO.NET驱动程序。使用ADO.NET客户端分发的要求记录在RowStore ODBC分发中。请参阅使用RowStore ADO.NET客户端。<br/>

RowStore提供了ODBC驱动程序，可响应客户端应用程序中的ODBC API函数调用。ODBC驱动程序支持32位和64位版本的Linux和Windows，并且符合ODBC 1级（支持所有ODBC Core和1级功能）。它还支持2级功能的子集。有关驱动程序安装要求的更多信息，请参阅ODBC驱动程序支持的配置。<br/>

RowStore脉冲系统要求 验证您的系统是否满足GemFire Pulse的安装和运行时要求。<br/>


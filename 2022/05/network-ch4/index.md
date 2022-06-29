# ch4 网络层-IP基础


## 1 IPv4 地址

IPv4地址是一个32bit地址，唯一地与通用地定义了一个连接在因特网上的设备。IP地址采用点分十进制记法，首先每8个Bit插入一个空格，然后将每8bit的二进制数转换为十进制数。

{{<admonition warning>}}

不能有先导0，数值均小于等于255且大于等于0

{{</admonition>}}

IP地址现由ICANN（Internet Corporation for Assigned Names and Numbers）进行分配。

IP地址管理机构在分配IP地址时，只分配网络号，而剩下的主机号则由得到该网络号的单位自行分配。

路由器仅根据目的主机所连接的网络号来转发分组（而不考虑其主机号），这样可以使路由表中的项目数大幅度减少，从而减小了路由表所占的存储空间。

IP地址的编址方法总共经历了三个阶段
- 分类的IP地址：这是最基本的编址方法，在1981年就通过了相应的标准协议。
- 子网的划分：这是最基本编址方法的改进，其标准RFC 950在1985年通过。
- 构成超网：这是比较新的无分类编址方法。在1993年提出后很快得到推广应用。


### 1.1 分类的IP地址

#### 1.1.1 IP地址分类

通过`net-id`和`host-id`将IP地址空间被划分为ABCDE五类，其中D类为多播地址，E类为保留地址。

{{< image src="0004-2.png" caption="IP地址的分类" src_s="0004-2.png" src_l="0004-2.png" >}}

{{<admonition info>}}

为什么要分离网络号和主机号？

因为两台计算机要通讯，首先要判断是否在同一个广播域内，即网络地址是否相同。如果网络地址相同，表示接受发在本网络上，那么可以把数据报直接发送到目标主机。路由器寻址工作中，也就是通过这样的方式来找到对应的网络号的，进而把数据包转发给对应的网络内。

{{</admonition>}}

#### 1.1.2 特殊IP地址

在IP地址中有一些并不是来标注主机的，这些地址具有特殊的意义。

这些地址包括**网络地址、直接广播地址、受限广播地址、本网络地址、环回地址**等。

①**网络地址** {\<net-id\>，0 }：用于标识网络

因特网上的每个网络都有一个IP地址，其主机号部分为0。

A类网络的网络地址为：Network-number.0.0.0。例如，120.0.0.0；

B类网络的网络地址为：Network-number.0.0。例如，139.22.0.0；

C类网络的网络地址为：Network-number.0。例如，203.120.16.0。

{{<admonition info>}}

用于标识网络，不能分配给主机，故不能作为数据的源地址和目的地址。
{{</admonition>}}

②**直接广播地址** {\<net-id\>，-1}：向某个网络上所有的主机发送报文。

TCP/IP规定，主机号各位全部为“1”的IP地址用于广播，叫作广播地址。路由器在目标网络处将直接广播地址映射为物理网络的广播地址，例如将其映射为6个字节全为1的以太网广播地址。

A类网络的直接广播地址为：Network-number.255.255.255。例如，120.255.255.255；

B类网络的直接广播地址为：Network-number.255.255。例如，139.22.255.255；

C类网络的直接广播地址为：Network-number.255。例如，203.120.16.255。

{{<admonition info>}}

直接广播地址只能作为目的地址

{{</admonition>}}

③**受限广播地址**{-1，-1}：在本网络内部进行广播。

受限广播地址是在本网络内部进行广播的一种广播地址，TCP/IP规定，32位比特全为“1”的IP地址用于本网络内的广播。

直接广播要求发送方必须知道目标网络的网络号，但有些主机在启动时往往并不知道本网络的网络号，这时候如果想要向本网络广播，只能采用受限广播地址（Limited Broadcast Address）。

{{<admonition info>}}

受限广播地址只能作为目的地址。

路由器隔离受限广播，不对受限广播分组进行转发。也就是说因特网不支持全网络范围的广播。

{{</admonition>}}


<br>

TCP/IP协议规定，网络号各位全部为“0”时表示的是本网络。本网络地址分为两种情况：本网络特定主机地址和本网络本主机地址。

④**本网络特定主机地址**{0，\<host-id\>}

{{<admonition info>}}

本网络特定主机地址只能作为源地址。

{{</admonition>}}


⑤**本网络本主机地址**{0，0}

无盘工作站启动时没有IP地址，此时采用网络号和主机号都为“0”的本网络本主机地址作为源地址。

{{<admonition info>}}

本网络本主机地址只能作为源地址。

{{</admonition>}}

⑥**环回地址**{127, \<any\>}：用于网络软件测试以及本机进程之间通信的特殊地址

习惯上采用127.0.0.1作为环回地址，命名为localhost。当使用环回地址作为目标地址发送数据时，数据将不会被发送到网络上，而是在数据离开网络层时将其回送给本机的有关进程。

环回接口对IP数据报的处理过程，如下图所示：

{{< image src="0004-3.png" caption="环回地址" src_s="0004-3.png" src_l="0004-3.png" >}}

在发送IP数据时，首先要判别该数据报的目的IP地址是否为环回地址，如果是环回地址，则直接将IP数据报放入IP输入队列实现环回。对于直接以本机地址作为目的地址的IP数据包也要会送给本机。对于广播或组播数据报，则在会送给本机的同时还要向网络发送。

在除去特殊IP地址之后，可使用的IP地址如下，

{{< image src="0004-4.png" caption="IP地址可使用范围" src_s="0004-4.png" src_l="0004-4.png" >}}

#### 1.1.3 私有网络

在IP地址空间中，保留了几个用于私有网络的地址，私有网络地址通常应用于公司、组织和个人网络，它们没有置于因特网。

A类：10.0.0.0~10.255.255.255

B类：172.16.0.0~172.31.255.255

C类：192.168.0.0~192.168.255.255


<br>


### 1.2 划分子网

{{<admonition question>}}

对于某些中等规模的网络，一个C类的IP地址不够用，而使用B类的IP地址则过于浪费。如果使用多个C类地址则会造成一个组织有多个网络域，这会导致路由器的路由表扩大。

{{</admonition>}}

在ARPANET早期，IP地址的设计不够合理，使得IP地址空间的利用率很低，而且给每个物理网络分配一个网络号会使路由表变得太大因而使网络性能变坏。从1985年起，在IP地址中又增加了一个子网号字段，使两级的IP地址变成了三级的IP地址。这种做法叫做划分子网，划分子网已经称为因特网的正式标准协议。

#### 1.2.1 子网的划分

划分子网属于一个单位（net-id）内部的事情，该单位（net-id）对外仍然表现为没有划分子网的网络。

划分子网的方式：从主机号中借用若干个比特作为子网号subnet-id，由此ip地址变为 {\<net-id>，\<subnet-id>，\<host-id>}这样的三级形式。

#### 1.2.2 变长网络掩码

为了在一个划分子网的网络中可以同时使用几个不同的子网掩码，提出了变长子网掩码（Variable Length Subnet Mask，VLSM）进一步提高IP地址资源的利用率。

{{< image src="0004-7.png" caption="VLSM" src_s="0004-7.png" src_l="0004-7.png" >}}

假设有一个组织的网络规模可以使用一个C类地址范围，且需要分为4个子网，那么使用定长的掩码，则至少需要将子网掩码设置为255.255.255.192，这样这个组织的C类地址就被分成了4个等大小的子网，每个子网有62个可用地址。但事实上子网A有100台主机和设备，子网B有50台，子网C、D各有20多台，此时就无法满足需求。

若使用VLSM技术，则可以设计192.168.1.0/25（126），192.168.1.0/26（62），192.168.1.0/27（30），192.168.1.0/27（30）的子网掩码。

<br>

### 1.3 构成超网
{{<admonition question>}}

虽然使用子网技术可以是IP地址都得到有效的应用，但还是很难防止IP地址资源耗尽，因为任何一个规模超过256的网络都需要使用B类地址，所以B类地址消耗的速度特别快，而C类地址缺得不到应用。解决这个问题的一个方法是消除IP地址中类别的概念。

{{</admonition>}}

在VLSM的基础上，又进一步研究出无分类编址方法，它的正式名字使无分类域间路由选择CIDR（Classless Inter-Domain Routing)。

CIDR技术将子网标识的两个部分直接合并到一起，使三级的IP地址又变回了两级{\<net-prefix>，\<host-id>}，net-prefix为网络前缀。

CIDR仍使用斜线记法，例如128.14.32.0/20，表示该网络一共能容纳$2^{12}-2$个主机或设备。

原有的IP地址分类方法，使得一个C类地址的网络，最多拥有256个IP地址，而使用CIDR之后致力于消除分类的概念，可以使原先多个C类地址合成为一个规模更大的地址范围，从而提高IP地址的利用率。

具体地，可见如下例子，

{{< image src="0004-8.png" caption="CIDR" src_s="0004-8.png" src_l="0004-8.png" >}}

<br>

### 1.4 寻址

IP地址的网络地址（\<net-id\>，0）用于路由控制中。

在主机和路由器上都会有各自的路由表，如果是windows，可以使用route print命令查看。

在路由表中记录着网络地址与下一步应该发送至路由器的位置。

在发送IP数据包时，首先会确定IP包首部中的目标地址，判断是否在同一网络中，若是则直接转发。否则去路由表中查找与该地址具有相同网络地址的记录，根据该记录将IP包转发给相应的下一个路由器。如果路由表中存在多条相同网络地址的记录，就选择相同位数最多的网络地址，也就是最长匹配。

如下图所示，凡是从其他网络发送到本单位某个主机的IP数据，仍然是根据IP数据报的**net-id**先找到连接在本单位网络上的路由器。然后此路由器在收到IP数据报之后，再按**net-id**和**subnet-id**找到目的子网，最后将IP数据报交付给目的主机。

{{< image src="0004-5.png" caption="子网的划分" src_s="0004-5.png" src_l="0004-5.png" >}}

路由器R1使用存储的子网掩码与目标IP地址相与，如果得到的结果与目的网络地址相同，那么说明该子网就是目标网络。

{{< image src="0004-6.png" caption="路由器寻找路径" src_s="0004-6.png" src_l="0004-6.png" >}}

<br>

## 2 IPv6

持续更新中

<br>

## 3 NAT地址映射

持续更新中

{{< image src="0004-9.png" caption="NAT1" src_s="0004-9.png" src_l="0004-9.png" >}}

{{< image src="0004-10.png" caption="NAT2" src_s="0004-10.png" src_l="0004-10.png" >}}

{{< image src="0004-11.png" caption="NAT3" src_s="0004-11.png" src_l="0004-11.png" >}}

{{< image src="0004-12.png" caption="NAT4" src_s="0004-12.png" src_l="0004-12.png" >}}

<br>


## 4 ISP技术

{{< image src="0004-13.png" caption="ISP" src_s="0004-13.png" src_l="0004-13.png" >}}



## 5 参考资料

1. https://blog.csdn.net/qq_38410730/article/details/80980749
2. https://xiaolincoding.com/network/4_ip/ip_base.html
3. 西电计算机网络课程PPT

本文仅IP的相关知识点进行摘录，文中所有图片著作权归原作者所有，如有侵权请联系删除。
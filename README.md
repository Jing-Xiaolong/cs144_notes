本仓库为斯坦福cs144a的课程学习笔记，原课程视频地址为[点我](<https://lagunita.stanford.edu/courses/Engineering/Networking-SP/SelfPaced/about>)

笔记包括其中的第1、2、4、5共四章的内容，即网络模型、网络层、运输层、应用层、拥塞控制内容



[TOC]



# 网络模型

## 四层网络模型

分层思想：

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/2-1-layering.jpg" width="200px">

链路层(link)，网络层(Network)，传输层(Transport)，应用层(Application)

四层网络模型为 end-hosts（终端-主机）间的信息传输提供保障



#### 链路层/网络接口层 Link Layer

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/2-2-link_layer.jpg" width="400px">

链路层任务：链路层沿着路径将数据包传递到下一个节点，完成比特组装成帧、单个链路的传输等工作

常见的链路有WiFi、Ethernet等



#### 网络层 Network Layer

网络层的任务：负责将数据从源端src传递到终端dest，以**包packet**为单位传输，数据包又称**数据报文datagram**

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/2-3-datagram.jpg" width="400px">

网络层将数据报文传送给链路层，链路层为网络层提供“服务”，将网络层传来的数据传送到单个链路的末端——假设为一个路由器Router

Router中的Link层接收数据，将数据传输给Network层，Network层检测报文中的dest信息，然后经过相关处理后将报文再次传送给Link，由其将数据传送到下一链路的末端

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/2-4-data_transmitting.jpg" width="500px">

**网络层特殊之处**

- 传输数据包时，必须使用Internet Protocol (IP)
  - IP只负责传输数据到另一端，但不保证可靠性
  - IP数据报文可能丢失，出现顺序出错、数据损坏等情况

↓ 如何保证传输的可靠性？



#### 运输层Transport Layer

最常见的运输层有

- TCP——Transmission Control Protocol
- UDP——user data protocol

TCP提供服务，保证数据传输的可靠性，在Web、 email等应用中非常实用

UDP只负责捆绑应用层数据并将它发送给网络层进行传输，不保证数据可靠性，在视频传输等中常用

另外，RTP用于实时数据传输，ICMP用于传递错误信息等



#### 应用层Application Layer

应用层的各类数据可以通过良好定义的API对transport传输层进行重用

常见的应用层包括BitTorrent, Skype, http, ftp等



#### 总结

应用层：通过特定的语法(如http, ftp)，两个应用可进行双向的可靠的字节流传输

运输层：提供端到端的可靠性数据报文传递和错误恢复(如TCP, UDP)，保证传输的可靠性；并进行拥塞控制

网络层：进行端到端的数据传输，必须使用IP，不保证可靠性

网络接口层：在单个链路(路由与路由, 或路由与端)上进行数据传输，

四层结构的特点—细腰型：

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/2-5-thin_waist.jpg" width="500px">



#### OSI七层网络模型

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/2-6-OSI.jpg" width="500px">

物理层：通过媒介传输比特，确定机械及电气规范，如RJ45, CLOCK

链路层：将比特组装成帧，在单个链路上进行数据传输， 如PPP, FR, MAC

网络层：进行端到端数据传输， IP

运输层：提供端到端的可靠性数据报文传递和错误恢复，UPD, TCP

会话层：建立、管理和终止会话（会话协议数据单元SPDU)

表示层：对数据进行翻译、加密和压缩（表示协议数据单元PPDU），如JPEG, ASCI

应用层：允许访问OSI环境的首端（应用协议数据单元APDU），如FTP, HTTP, WWW, DNS



# 网络层

## IP 服务模型

事实上，使用互联网即通过IP来发送、接受**包**，IP的工作是将**数据报文**运输到另一端

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/2-7-IP.jpg" width="700px">



### IP服务模型的特征

|      性质      |                       行为                       |
| :------------: | :----------------------------------------------: |
| Datagram为单位 |                  每个包单独路由                  |
| Unreliable传输 |                   包可能被丢弃                   |
|  Best effort   |                ... 如果有必要的话                |
| Connectionless | 不维护与数据报文关联的每个流状态，因此包排序错误 |

- `Datagram`：`IP`是一个数据报文服务，包括报头`Header`和数据`Data`。当请求数据时，`IP`创建一个数据报文并将数据放入其中。数据报文`Datagram`是一个数据包`Packet`，它根据报头中的信息在网络中单独路由
  - 目的地的地址`IP DA`，每个路由通过该地址决定将请求报文发往何处（接收器）
  - 发送源的地址`IP SA`，通过该地址接收器确定将返回的数据包发往何处（发送源）
  - 数据`Data`

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/2-8-datagram_transmitting.jpg" width="500px">

- `Unreliable`：`IP`并不保证包能发往目的地，可能延迟到达，或顺序出错，或根本不会到达。
- `Best effort`：`IP`承诺在必要的时候丢弃数据报文，如
- 路由器中的缓冲区因拥塞而填满，迫使路由器丢弃下一个到达的包，并且**不**告知发送源
- 错误的路由表`Routing table`可能导致包发送到错误的目的地，或导致包被错误的复制
- `Connectionless`：`IP`所做的工作仅仅是将每个包单独、独立的发送到目的地，对请求的其它信息一概不知

##### IP为何如此简单？

- 保持网络简单、最简、更快、更低的构建和维护成本：网络是由分布在整个网络中的大量路由器实现的，保持`IP`足够简单，便于维护升级、可靠性更高
- 端到端准则：特征应该在终端计算机上的软件中实现，而非集成到互联网中。通信可靠性、拥塞控制等功能都应该在端（源端、目标端）完成
- 给运输层提供更多的选择：可以在`IP`上一层建立一系列可靠或不可靠的服务，可让应用自行选择
  - 如果`IP`被实现为复杂的可靠通信时，在视频电话等应用上的效果并不理想，因为重发丢失的数据有延迟，这样重发数据并无多大意义
- 简单性使`IP`可在任何网络接口层上工作，因为`IP`对网络接口层没有任何要求



### IP服务模型的其它特征

- 阻止包循环发送：路由表出错可能导致包沿着同一路径循环发送，为达到阻止循环发送的目的，`IP`在每个包的报头中添加存活时长字段，即`Time To Live(TTL)`，它从一个较大的数（如128）开始，经过每个路由器后递减，如果达到零就判定为陷入循环的包，丢弃
- 分割较长的包：大多数的链路层对它们所能携带的包的最大尺寸有限制，如`Ethernet`不超`1500Bytes`，`IP`将较长的包进行分割，如分割成`1000Bytes`长
- `CheckSum`字段，确保包发往正确的目的地
- 两类`IP`：`IPv4`—`32bit`地址，`IPv6`—`128bit`地址
- 允许向报头中添加自定义字段`OPTIONS`



### IPv4报文结构

报头Header：灰色区域

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/2-9-IPv4.jpg" width="600px">

Version：IP的版本，只能为`IPv4`或者`IPv6`

Header Length：报头长度，某些报头存在自定义的额外字段`OPTIONS`，该字段可以确认其长度

Type of Service：指示该包的重要程度

Total Packet Length：包长度，包括报头和整个数据，可达64KB ($$2^{16}$$)

Packete IP, Flags, Fragment Offset：用来分割较大的包

Time To Live "TTL"：生存时间，阻止循环路径发送位，该字段为0时即丢弃该报文

Protocol IP：传输协议类型，如 6 表示使用`TCP`。IANA组织规定了140余种不同传输协议类型

Checksum：整个报头的校验和，防止报头损坏



### IPv4 Addresses

- IPv4地址：IP允许设备在网络上通过地址交换信息，IPv4的地址为32位，通常写为4个8位的形式`a.b.c.d`，如

  - `171.64.64.64`
  - `10.166.160.1`

- 子网掩码：告知设备哪些IP地址是本地的（在同一子网内），哪些设备需要通过路由器才能传递信息（不在同一子网），由一系列连续的 1 构成，共32位，从最高位开始，如：

  - `255.255.255.0`：前24位相同，表示前24位相同的IP地址在同一个子网，直接收发信息不通过路由器

  将两个地址通过网络掩码按位取AND，如果相同则表示位于同一网络上

- 举例：本机`IPv4`地址为`10.166.241.194`，子网掩码为`255.255.0.0`，则：

  - 往`10.166.241.194`发送信息时直接发送，因为它们在同一个子网内
  - 往`10.165.241.194`发送信息时则将它发送到路由器，由路由器发送到目的地





## 端到端原则 end-to-end principle

网络的工作是尽量高效、灵活地传输数据报文，其它所有工作都应该放在终端主机上完成。为确保数据正确传输，必须在连接两端的主机的帮助下实现，不可能完全在链路层或者传输层实现，因此必须在接收端的主机上对数据进行校验，这需要在应用层实现。

这种设计原则可将底部的网络接口层最简化，只负责数据传输，提高传输性能和稳定性。

同时，这一原则需要为网络上的主机分配了全球唯一地址，这样在传输过程中不需要中间节点或转发节点对包内容进行更改，位于端的应用程序也不需要知道实际传输的具体路线，为实际应用带来方便。





# 运输层

## TCP服务模型

TCP全称transmission control protocol，提供**可靠的、端到端的、双向的**字节流服务

如下图所示，TCP将所有需运输的字节流放入`TCP segment`的数据段中，并将其传入IP，由IP将封装于数据报文。然后，数据报文传入链路层，该层构建`Link frame`（添加`Link address`并封装），然后将其发送到网络。

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-1-TCP1.jpg" width="500px">



#### TCP字节流服务

- 主机A接收上层应用层的字节流，并将其封装进**TCP段**，然后将它传入网络层的IP进行发送，主机B接受TCP段，从中提取并重建字节流，并将字节流发送到自己的上层应用层
- 由于字节段可能被丢弃，或者因为A并未接收到ACK，TCP段可能需要发送多次。
- TCP段可以短至1byte，如通过`ssh`创建会话；也可高达IP数据报文的长度上限

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-3-TCP_trainsmitting.jpg" width="600px">



#### 三次握手——建立连接

TCP连接通过两个主机之间的**三次握手**建立。

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-2-sync.jpg" width="250px">

1. 主机A的传输层向主机B的传输层发出连接请求SYN，表明意图建立连接。
   - 同时，A发送一个初始序列号seq=x，表示A->B字节流中的第一个字节的序列号
   - 注意：SYN报文段不能携带数据，但要消耗掉一个序列号。序列号seq也可以是一个随机数，这样可以防止攻击者轻易提取出序列号，同时可以极大避免与网络中还未清除的旧数据包中的序列号重叠
2. 主机B收到连接请求，若同意连接，则回复SYN+ACK表示确认请求
   - 同时，B发送确认号ack=x+1，表示确认收到了x，下一次发送x+1过来；B发送一个初始序列号seq=y，表示B->A字节流中的第一个字节的序列号
3. 主机A收到B的确认后，向B给出确认，回复ACK表明接受反向通信请求
   - 同时，A发送确认号ack=y+1，表示确认已收到了y，下一次发送y+1过来；A收到了B发来的确认号ack=x+1，因此发送seq=x+1
   - 注意：ACK报文段可以携带数据。如果不携带数据则不消耗序号，这种情况下下一个报文段的的序号仍是seq=x+1。

此时，双向连接建立，已准备好互相发送数据

##### 为什么需要三次握手

1. 信道不可靠，TCP想在不可靠信道上建立可靠连接，三次通信是理论最小值

   UDP不需要可靠连接，因此不需要三次握手

2. 双方都需要确认对方收到了自己发送的序列号，确认过程最少进行三次通信

3. 防止已失效的连接请求报文段又突然传送到了B端，因而产生错误。

4. 至少是三次握手。如果两端同时发出建立连接的请求，则是四次握手：两次SYN，两次SYN+ACK



#### 四次挥手——关闭连接Teardown

两方主机告知彼此关闭连接，并且两端都可以清理和状态机关联的状态

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-4-sync_close.jpg" width="400px">

1. 主机A的发出FIN，请求释放A->B的连接
2. 主机B收到FIN，同意释放连接，表示不再接受来自A的消息，向A发送ACK进行确认
3. 主机A收到ACK，A->B的连接释放
   - B可能仍需向A发送数据，因此B发往A的ACK信息中同样可能存在数据信息，B还能随意向A发送数据
4. 主机B继续发送之前没发送完的数据到A
5. 数据发送完毕，主机B发送FIN+ACK给A，请求释放B->A的连接（就算没收到A的确认过段时间也会自动释放）
6. 主机A收到FIN+ACK，同意释放连接，回复ACK给B，并发送ack=z+1表示已收到B发送的最后一个数据报文
7. 主机B收到ACK，释放B->A的连接。此时，双向连接均已关闭

##### 为什么需要四次挥手

TCP是全双工模式，两个方向的连接需要分别关闭，每次关闭需要请求和确认两次挥手，因此总共需要四次挥手。

##### 为什么握手三次、挥手四次

在连接时，服务器发送ACK和SYN是一起发送（第二次握手），因此TCP连接时只需要三次

在释放时，由于数据双向传递且有快慢，只能一次断开一个方向的连接，不能让服务次将ACK和FIN一起发送



#### TCP服务模型小总结

|     性质     | 行为                                                         |
| :----------: | :----------------------------------------------------------- |
|    字节流    | 提供可靠的字节传输服务                                       |
| 保证传输可靠 | - TCP层收到信息后，回发ACK以告知发送者数据已正确到达<br>- Checksum校验和，检测TCP段是否出错<br>- TCP段中的序列号Seq number，数据段中第一个字节的在整个流中的序号，检测丢失数据<br>- 流控制Flow control，可防止接收过载——防止A发送速度过快于B接收速度<br>  接收器持续告知发送者是否继续发送，即告知其缓冲区还剩多少空间可用 |
|   串联顺次   | TCP层使用序列号Seq number对数据重排，保证主机A和B中应用层的收发数据顺序相同 |
|   拥塞控制   | TCP层为使用网络的所有TCP连接平均分配网络容量，以控制拥塞     |



#### TCP报文格式

由于TCP提供可靠连接，因此其TCP报文较复杂

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-5-TCP_segment.jpg" width="600px">

- Src port：源端口，告知另一端的TCP层应往哪个端口回发数据
- Dest port：目的端口，告知TCP层数据应发送给哪个应用端口，IANA定义了各类应用程序的端口，例如：
  - HTTP，使用port 80
  - FTP，使用port 23
  - HTTPS，使用port 443
- Sequence #：序列号，表明TCP数据中的第一个字节在整个字节流中的位置。比如
  - 第一个TCP段的序列号为1000，段长为500，那么下一个TCP段的序列号就为1500
- Acknowledgement Sequence #：确认号，之前的每一个字节均成功接收，并告知发送端下次接收的序列号
  - 确认号是751，表示750及以前的字节均已成功接收，请你下次发送751
- HLEN: Header length field头部长度，又为OFFSET，指出TCP Data开始的位置
- Flags：表明各种状态的标志位，包括
  - ACK确认位，ACK=1时Ack seq #有效，=0时无效。连接结束前，除了第一次握手，其余时候都为1
  - SYN同步位，告知对方正在发送同步信号，用来三次握手建立连接
  - FIN终止位，确认终止某方向上的连接，用来四次挥手关闭
  - PSH推送位，告知另一端TCP层在数据到达时立即交付给应用层，而非等待更多数据到达。该位对携带时间关键数据的短字段非常有效，如键击数据
  - RST重置位，重置连接（可能连接出错）
  - URG紧急位，告知报文中存在重要数据，此时Urgent Pointer指示该数据在报文中的位置
- Window size：滑动窗大小，用于流控制
- Checksum：16位校验和，使用TCP data和TCP header计算所得，检测数据错误。有时包括部分IP header
- Urgent Pointer：当URG紧急位有效时，该16bit数据告知对应的紧急数据在报文中的位置
- TCP Options：自定义可选段，可通过HLEN确认该段的长度



#### TCP连接的全球唯一性

TCP连接通过TCP header和IP header中的五部分信息确定其全球唯一性

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-6-Unique_TCP.jpg" width="600px">

- IP DA、IP SA确认唯一源、目的地终端的IP地址，Protocol ID=6确认连接类型为TCP
- Src Port、Dest Port确认终端上唯一应用程序：
  - 终端主机为每个新连接递增Src port，共16bit，因此两台主机间可使用最多$2^{16}=64k$的新连接
  - TCP连接以随机序列号initial sequence number ISN对序列号Sequence #进行初始化，即A在建立与B的连接时向B发送随机序列号，B同理

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-7-sync2.jpg" width="250px">

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-8-ISN.jpg" width="600px">





## UDP服务模型

UDP全称User datagram protocol，其服务模型无连接、不保证可靠传输，用于不需要保证传输可靠性的应用中，或应用有自己的处理重发送机制

UDP所作的工作只包括：

- 接收应用层的数据
- 创建UDP数据报文
- 然后递交到网络层进行传输

UDP不需要创建连接，只需要标记数据的源、目的地应用程序端口，然后发送即可

#### UDP报文格式

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-10-UDP_segment.jpg" width="600px">

- Src port：源端口，告知另一端的UDP层应往哪个端口回发数据

- Dest port：目的端口，告知UDP层数据应发送给哪个应用

- Checksum：使用IPv4时可选

  - 如果发送端无校验和，则以0s填充
  - 如果有校验和，则对UDP header和UDP data计算校验和
  - 校验和的计算还包括IP SA、IP DA、Protocol ID=17(UDP)。这种计算跨越了分层，但可以使UDP层检测到会被发往错误地址的报文

- Length: 16bit，确认整个UDP报文长度，包括头部和数据部，且 >= 8 Bytes(头长度)

  

#### UDP服务模型小总结

| 性质                     | 行为                                                         |
| ------------------------ | ------------------------------------------------------------ |
| 无连接数据服务           | 不建立连接<br>所有的包以任意顺序出现（应用层自行管理包顺序） |
| Self contained datagrams |                                                              |
| 不可靠传输               | - 不发送确认信息ACK<br>- 没有检测丢失或顺序出错的包的机制，应用层自行建立<br>- 不进行流控制 |



#### UDP、TCP对比

| TCP                                          | UDP                                      |
| -------------------------------------------- | ---------------------------------------- |
| 面向连接                                     | 无连接                                   |
| 提供可靠服务，数据不丢失且按序到达           | 尽最大努力交付，但不保证可靠性           |
| 逻辑通信信道是全双工可靠信道                 | 不可靠信道                               |
| 每个TCP连接只能是点到点                      | 支持一对一，一对多，多对一，多对多       |
| 面向字节流传输，将数据视为一连串无结构字节流 | 面向报文                                 |
| 存在拥塞控制Congestion control               | 无拥塞控制，网络拥塞时也不会降低发送速率 |
| header复杂，至少20字节长                     | header简单，只有8字节                    |





## ICMP服务模型

#### 简介

ICMP全称Internet control message protocol，用于报告和诊断网络层的错误和问题

保证网络层Internet layer正常工作的三个机制：

- IP
  - 创建并封装IP数据报文
  - 从发送端到接收端进行逐链路传输hop-by-hop
- 路由表
  - 通过算法产生路由转发表
- ICMP
  - ICMP在终端主机和路由器之间传递关于网络层的信息
  - 报告出错状态Error conditions
  - 诊断问题，找出数据包的传送路径等...

ICMP运行于网络层之上，严格来说是一个运输层协议。当终端主机或路由想通过ICMP报告错误或诊断信息时，它将想传送回源的信息放入ICMP有效负载中，并将它传递给到IP层，然后作为数据报文传送

如下图，当路由器发现路由表中没有信息时(别管啥原因，反正没有)，它无法将包发送到B。因此，路由器回向A回发信息”目的地网络无法到达“。

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-11-ICMP_example.jpg" width="400px">



#### ICMP服务模型及报文格式

| 性质     | 行为                                                         |
| -------- | ------------------------------------------------------------ |
| 报告信息 | 自包含Self-contained的消息，用于报告错误                     |
| 不可靠   | 简单的数据报文服务，不可靠，不会尝试重新发送，也不会保留消息状态 |

ICMP的报文格式非常简单，具体如下

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-12-ICMP_segment.jpg" width="500px">

ICMP数据报文包含2个部分：

- 发生错误的IP数据报文的IP header和IP data的前八个字节
- ICMP类型和ICMP码：ICMP部分信息类型（RFC 792可找到）如下：

| ICMP Type | ICMP Code | 备注                                 |
| :-------: | :-------: | ------------------------------------ |
|     0     |     0     | Echo Reply (used by **ping**)        |
|     3     |     0     | Destination Network Unreachable      |
|     3     |     1     | Destination Host Unreachable         |
|     3     |     3     | Destination Port Unreachable         |
|     8     |     0     | Echo Request (used by **ping**)      |
|    11     |     0     | TTL Expired (used by **traceroute**) |



#### ICMP应用举例

##### ICMP Ping

Ping 用来测试另一个主机是否活着，并检查是否连接到该主机，还可用来测试端到端延迟大小

Ping操作直接调用ICMP，并发送一个Echo request，将该请求放入IP，然后发送到对应主机B。主机B收到Echo request后会回发一个Echo reply，其过程如下图

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-13-Ping.jpg" width="600px">

##### ICMP Traceroute

Traceroute用于检测两个终端主机之间的路由器跃点路径，如

- win10 cmd中输入<font color='green'>**`tracert tsinghua.edu`**</font>可查看一个到该终端所经过的所有路由器节点的IP地址

***工作机制***：主机A依次发送TTL=1,2,3...的UDP数据报文，报文在经过第1,2,3...个路由器节点时会因TTL递减到0而返回ICMP消息，报告各个路由的信息。最后，主机A发送一个错误的端口号到主机B，到达时主机B会因找不到对应端口而返回Destination Port Unreachable的ICMP报文信息，以此获得主机B的信息。整个过程如下图

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-14-Traceroute.jpg" width="700px">

#### ICMP小总结

- ICMP可提供网络层内主机和路由器的信息，常用于错误报告和网络诊断中
- ICMP工作于网络层之上，严格来说是一个传输层协议，尽管它是为网络层提供服务的
- Ping和Traceroute指令工具均依赖于ICMP





## 错误探测 error detection

三种错误探测机制——Checksum，CRC - cyclic redundancy codes，MAC - message authentication codes

不同协议将校验值放在不同位置：

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-15-Error_detecting.jpg" width="450px">

- **Checksum校验和**
  - 多用于TCP和IP，将包内所有数据相加
  - 计算快、代价小，鲁棒性差，多个错误可能互相抵消
- **CRC循环冗余码**
  - 多用于Ethernet和各类链路层。在某些情况下，若链路层使用CRC校验，TCP和IP可以不用Checksum
  - 对于一个C-bit长的CRC校验，它可以检测到任何小于C位的误差，以及任意奇数位的误差
  - 计算代价大，但鲁棒性好
- **MAC消息身份验证码**
  - MAC将包和一些私密信息结合起来产生一个检测值
  - 对恶意修改抵抗性更强，因此常用语运输层的安全保障TLS - transport layer security，即https
  - 对错误探测的鲁棒性不高，任意2条信息有$2^{-c}$的几率计算出同样的MAC值



## 有限状态机 Finite state machine

状态机的本质是对具有**逻辑顺序**和**时序规律**的事件的一种描述方法，有**状态、事件、操作**三个要素

有限状态机由有限个状态组成，一个状态即系统的一个特定配置，不同的状态转变的导致事件必然不同，状态之间的边定义了：1. 导致状态转变的事件，2. 状态改变时，系统将采取的操作（可选的，并不一定会有操作）。一个典型的有限状态机如下：



<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-16-FSM.jpg" width="350px">



#### 小实例

设计一对sender-receiver的通讯协议，receiver的有限状态机如下，当receiver收到下列数据值时打印出啥字符？

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-20-FSM_example.jpg" width="500px">

1,2,1,1,2,2,2,2,1,1,2   =>  print: mamajama

1,1,1,2,1,1,2,2,1,1   =>  print: mmamajxm



#### TCP连接的FSM

共12个状态，大部分时间处于ESTABLISHED和CLOSED两个状态：

- ESTABLISHED：连接建立完成阶段，TCP正在发送、接收数据
- CLOSED：连接已关闭（底部）、或连接还未打开（顶部）

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-17-FSM_TCP.jpg" width="800px">

##### 三次握手的FSM

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-18-FSM_TCP3shakehand.jpg" width="500px">

1. 事件：Server(B) 调用`listen()`，状态从CLOSED转变为LISTEN，**被动打开**

2. 事件：Client(A)调用`connect()`，状态从CLOSED转变为SYN SENT

   操作：向Server(B)发送SYN，**主动打开**（第一次握手）

3. 事件：Server(B)接收到SYN，状态从LISTEN转变为SYN RECEIVED

   操作：向Client(A)发送SYN+ACK（第二次握手）

4. 事件：Client(A)接收到SYN+ACK，状态从SYN SENT转变为ESTABLISHED

   操作：向Server(B)发送ACK（第三次握手）

5. 事件：Server(B)接收到ACK，状态从SYN RECEIVED转变为ESTABLISHED



##### 四次挥手的FSM

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/3-19-FSM_TCP4shakehand.jpg" width="500px">

1. 事件：Client（任何一个主动先发关闭请求的主机）调用`Close`**关闭单向连接**，状态由ESTABLISHED转变为FIN WAIT 1

   操作：发送FIN，**主动关闭**

2. 事件：Server收到FIN，状态由ESTABLISHED转变为CLOSE_WAIT

   操作：发送ACK，**被动关闭**

3. 事件：Client收到ACK，状态由FIN WAIT 1转变为FIN WAIT 2，此时只能接受消息不能发送

4. 事件：Server数据发完，调用`Close`**关闭另一方向连接**，状态由CLOSE WAIT转变为LAST ACK

   操作：发送FIN

5. 事件：Client收到FIN，状态由FIN WAIT 2转变为TIME WAIT

   操作：发送ACK

6. 事件：Server收到ACK，状态由LAST Ack转变为CLOSED

7. 事件：Client经过2MSL（最长报文寿命 Maximum segment lifetime）后，此件没有收到Server任何消息，状态由TIME WAIT转变为CLOSED

注意特殊情况：两端几乎同时发送FIN进入FIN WAIT 1状态，然后收到对方发送的FIN，则都进入CLOSING状态，然后进入TIME WAIT状态，等待2MSL后再进入CLOSED状态。

等待2MSL是为了保证对方发送的最后一个ACK报文能够到达，并在这个时间使所有本连接持续时间内产生的所有报文都从网络中消失，使得在端口被立即重用且存在序列号重叠的情况下避免数据损坏。





## 流控制 Flow control

问题：发送端发送数据的速度超过接收端处理能力，导致接收端缓冲区满后直接丢弃后续的包

解决方法：接收端进行反馈，告知发送端不要发送超过接收端处理能力的包数量

两种方案：*Stop and wait*、*Sliding window*



#### 停止并等待 Stop and wait

主要思想：任意时刻只有一个包在传送

具体做法：发送端发送一个包，接收端接收到了之后回复一个确认包表明已成功接收，发送端接收到确认包后再次发送；发送端超时未收到确认包时，认为包丢失，重新发送该包

缺陷：不能充分利用带宽，绝大部分时间都用于等待

**Stop and wait 的 FSM**

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/4-1-Stop_wait_FSM.jpg" width="500px">

**四种情况**：

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/4-2-Stop_wait_exec.jpg" width="500px">

No Loss：正常执行

Data Loss：发送端发送数据，但数据丢失，发送端超时后重新发送

ACK Loss：发送端发送数据，接收端接收到后回复ACK，但回复丢失，发送端超时后重新发送

ACK Delay：发送端发送数据，接收端接收到数据后回复ACK。但ACK回复超时 ，接收端又重新发送，产生多重发送、多重确认的问题。

解决：使用一个一位计数器解决，告诉接收器本次发送的包是新数据包还是重发的包。但本方法必须保证 1.网络本身不会产生重复的包，2. 包延迟不会有多个timeout的延迟



#### 滑动窗 Sliding window

Stop and wait允许一次发送一个包，Sliding window允许一次发送N个包（回复N个ACK）

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/4-2-Sliding_window.jpg" width="500px">

##### 发送端和接收端处理机制

注意：TCP是双工的，当连接建立后两端地位相同，都可以进行发送和接收

- 发送端 Sender

  - 每个发送报文都有一个序列号data sequence number
  - 保存三个变量
    - Send window size SWS （发送窗口大小）
    - Last acknowledgement received LAR （最后一个已接收的确认）
    - Last segment sent LSS （最后一个已发送的报文）
  - 维持 LSS - LAR <= SWS
  - 收到新的确认信息后，根据需要对LAR递增
  - 对SWS个报文进行缓冲，以防突然收到多个确认后还未将需发送的报文准备妥当

- 接收端Receiver

  - 保存三个变量
    - Receive window size RWS （接收窗口大小）
    - Last acceptable segment LAS （最后一个可接收报文）
    - Last segment received （最后一个已接收报文）
  - 维持 LAS - LSR <= RWS，接收到不满足此关系的报文则直接丢弃
  - 接收包序号 < LAS，发送确认报文
  - 使用累计确认，即返回的确认报文对应于收到的连续包的第几个
  - 如果收到了1，2，3，5，6，缺失了4，在收到5及后面的数据时依然返回收到3对应的ACK。如果缓冲区未满，则将5，6放入缓冲区。Sender一直未收到4对应的ACK，超时后重新发送4，此时Receiver收到4后，将4，5，6都进行处理，回复6对应的ACK

  满足RWS >= 1，SWS >= 1，SWS >= RWS（SWS < RWS时部分接受窗始终不会被使用，浪费）



##### TCP通过滑动窗实现flow control

window：指定接收端和发送端的滑动窗大小

data sequence number：报文的序列号

acknowledgement sequence number：确认报文的序列号

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/4-3-sliding_window_TCP.jpg" width="350px">

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/4-4-Sliding_window_TCP_2.jpg" width="650px">

​		

TCP的中包的发送、接收过程

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/4-5-Sliding_window_TCP_3.jpg" width="800px">





## 重传策略 Retransmission strategies

为每个包设定一个计时器，超时未收到对应确认包则进行重传

两种重传机制——Go-back-N、Selective repeat

#### Go-back-N

处理机制：超时未收到包确认，则认为对应包及它后面所有位于窗口内的包均丢失，全部重传

优缺点：窗口内未丢失的包也会重传，浪费带宽；但如果发生连续丢包，则可以快速响应将所有包重传

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/4-6-Go_back_n.jpg" width="300px">

#### Selective repeat

处理机制：超时未收到包确认，则认为只有该包丢失，只重传该包

优缺点：连续多个丢包的重传速度较慢，单个丢包处理更好

#### 滑动窗大小的影响

- 发送窗SWS = N > 1，接收窗 RWS = 1，重传行为是Go-back-N
- 发送窗SWS = 接收窗 RWS > 1，重传行为是Selective repeat







# 拥塞控制



Flow control：针对连接两端进行流量控制，避免发送端发出超过接收端处理能力的大量包

Congestion control：针对整个网络进行流量控制，避免发送端发出过量的包，造成路由器缓冲区填满、链路溢出，进而造成丢包

End-to-end原则：在连接的两端（尤其是发送端）进行拥塞控制，不对网络接口层进行控制。



## 拥塞控制的基本概念（Congestion control）

#### 拥塞控制简介

网络拥塞无法避免，并且一定程度的拥塞有助于提高网络利用率，需要在传输延迟和网络利用率之间进行权衡：

- 缓冲区一直空闲，此时延迟低，但网络利用率低
- 缓冲区一直繁忙，此时网络利用率高，但延迟大

拥塞在多种时间尺度上发生：

- 大时间尺度如用网高峰期大量用户在某个时间段内集中使用某些链路，造成带宽不够
- 中等时间尺度如两个信息流在某个链路中流动，两者带宽之和大于链路带宽时会造成丢包
- 小时间尺度如两个包使用同一个路由器的拥塞，晚到的包需要等待缓冲区中等待前一个包发送完后再发送

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/5-1_congestion_timescales.jpg" width="750px">

拥塞发生后，重发retransmission问题会加剧拥塞

对瓶颈链路，应注意带宽分配的**公平性**和**吞吐量**的权衡

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/5-2-fairness_throughput.jpg" width="500px">

**Max-min公平**：无法在不减小流速更低的流的传输速率的情况下增大一个流的流速，此时网络就达到了Max-min公平。通俗的讲，就是最大限度的使用流量小的链路，使得通过该小流流量链路的流都能分配到一定带宽。上图中分配方案2即Max-min公平分配。注：Max-min公平是针对单个链路。

为达到Max-min公平，在分配时，从需求最小的开始分配，当均分可用带宽后所有的入口都不发满足需求就停止分配。如：对下图的单个链路，首先将链路均分为0.33，C只需要0.2因此分配给它0.2，剩余的再均分为0.4，A、B都不能满足，分配完毕

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/5-3-Max-min-Fairness.jpg" width="300px">

**拥塞控制的目的**：1. 高吞吐量，保持链路繁忙且保持流速快；2. 公平性，主要使用Max-min公平；3. 快速响应网络环境的变化；4. 分布式拥塞控制，不能依靠一个“中央控制器”来调控整个网络拥塞



#### 在何处进行拥塞控制

- ***基于路由器的拥塞控制***

  每个路由器都使用一个Fair Queueing进行拥塞控制，为每一个使用路由器R的流设置一个流队列，为每个流队列平均分配路由器R的带宽。

  缺点：简单的均分带宽，对源没有响应，不会指示源合适的发送速度，并且会浪费大量的上游带宽。

- ***基于网络的拥塞控制***

  数据包经过每个路由器时，路由器向包中添加自己的缓冲区使用率，当包到达目的端后目的端返回ACK时可以通知发送端整个传输路径上每个路由器的拥塞情况，发送端据此自行调整发送率

  缺点：需要整个网络的协助，增加了链路层的复杂性。

- ***<font color="blue">基于主机end-host的拥塞控制</font>***

  主机对网络的状态进行观察（包发送超时、重复接收ACK等），根据观察结果判断网络是否发生拥塞，然后进行拥塞控制：

  <font color="green">优点</font>：不需要每个路由器的协助，不需要网络层的支持，符合端到端原则。这也是TCP拥塞控制的方式。

**TCP拥塞控制**：IP默认不提供路由信息，因此TCP的拥塞控制在主机端实现。通过对网络中可观察的包丢失等事件的观察，计算某一时刻可以安全地向网络中发送的数据包的数量（即拥塞窗大小 congestion window, cwnd），然后根据下列方式调整TCP的滑窗大小来进行流控制

网络拥塞时使用cwnd，网络畅通时使用RWS (receive window size)

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/5-4-TCP_congestion_control_wdsz.jpg" width="500px">



## 拥塞控制基本方法—AIMD

#### AIMD简介

AIMD全名additive increase, multiplicative decrease（加法增加、乘法减少），其cwnd的计算方式为：

- 包成功接收，则 $W←W+1/W$
- 包被丢弃，则 $W←W/2$

每成功接收一个完整窗口内的所有数据包，cwnd增加1。cwnd的变化如下

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/5-5-AIMD_sawtooth.jpg" width="550px">

$$W$$即对整个网络可承受的数据包的数量的估计

自时钟Self-clocking：返回的ACK包触发包的发送

#### 单个数据流的AIMD阻塞控制

[查看动画](<http://guido.appenzeller.net/anims/>)，注意：

- W成锯齿形变化，瓶颈链路保持100%使用率，路由器发送速率一直保持不变，即100%*C
- AIMD未调整发送端的发包率W/RTT，而是在调整整个网络可承受的包的数量
  - 当瓶颈链路使用率达到100%，发送端窗口大小W增加1，则路由器缓冲区就多占一个包，RRT也多一个包
  - 缓冲区满导致丢包，滑动窗减半
  - 滑窗大小成稳定地锯齿变化时是发包率恒定的稳定工作状态

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/5-6-AIMD_Single_flow.jpg" width="800px">

- 在稳定工作状态时，各参数的变化如下：
  - 滑窗大小：成锯齿形变化
  - 包往返时间RTT：与滑窗大小完全同步变化，这一点与多个流的情况完全不同
  - 瓶颈链路使用率：始终保持100%
  - 路由器缓冲区占用率：成锯齿形变化

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/5-7-AIMD_single_flow_paras.jpg" width="750px">

注意：路由器缓冲区要刚好可以容纳足够的包（Buffer Size B = RTT * C），使得W减半缓冲区排空进入链路的包可以填满往返行程中。

**示例1**：每个包长度为1KB，在AIMD达到稳定后，滑窗大小在恒定最小值10KB、最大值20KB之间变化

1. 在滑窗从10KB变化到20KB期间，一共发送了多少个包？

   每个RRT内增加一个包，滑窗最小为10个包，最大为20个包，因此总共

   $$(10+20)/2*(20-10+1)=165$$个

2. RTT固定为10ms，发送一个16,500KB的文件总共需多久?（RTT=W/发包率，不变，说明带宽在变化）

   由1可知，每发送165个包，经历11个RTT

   因此，发送165KB的文件用时$$11\times10ms=110ms$$，则总用时为$$16,500/165*110=11,000ms=11s$$

**示例2**：Alice从远程服务器以10M/s速度下载视频流进行观看，每个包的长度250Bytes。Ping的最小时间为5ms。在AIMD达到稳定后，滑窗大小在恒定最大、最小值之间成锯齿形变化，路由器缓冲区为最佳尺寸。（和例1不同，本例带宽固定不变，因此RTT将会变化）

1. AIMD滑窗最小值是多少Bytes？

   从上图可知，当路由器缓冲区为空（不用缓冲等待）且瓶颈链路使用率100%时，可得到最小Ping时间5ms (即RTT)，因此链路中存在 $$10Mb/s*5ms=50,000bits$$，即最小滑窗大小为 $$50,000/8=6,250Bytes$$

2. AIMD滑窗最大值是多少Bytes?  $$100,000bits=12,500Bytes$$

3. 路由器中的数据包缓冲区有多大？$$B=12,500-6,250=6,250Bytes$$

4. 当某个包被丢弃、滑窗减半后，需要多久可以使滑窗尺寸再次达到最大值？

   包长度为$$250Bytes$$，因此每个RTT滑窗增加$$250Bytes$$大小

   故从$$6,250Bytes$$增加到$$12,500Bytes$$需要时间为$$(12,500-6,250)/250=25$$RTT​

   平均每个RTT的时间为$$(5+10)/2=7.5$$ms，总共需要$$25\times7,5=187.5$$ms

5. 切换远程服务器，带宽仍为10M/s，RTT时间变为250ms，则路由器缓冲区尺寸应为多少？

   与1类似，$$10Mb/s*250ms=2.5\times10^6bits$$，如果用2的幂表示存储大小，则为$$2.4Mbits$$或$$298K$$

6. 远程服务器切换后，包丢弃滑窗减半，需要多久滑窗再次达到最大值？

   与4类似，每个RTT滑窗增加$$250Bytes=2000bits$$

   从$$2.5\times10^6bits$$增加到$$5.0\times10^6bits$$需要$$(5.0-2.5)\times10^6/2000=1250$$RTT

   总共需要$$1250\times(250+500)/2=468.75s$$



#### 多个数据流的AIMD阻塞控制

- 由于多个数据流共用路由器内的缓冲区，因此数据流到达时刚好发现缓冲区已满而丢包是随机事件，每个数据流使用缓冲区很小的一部分，该流的滑动窗减半对缓冲区占用率影响很小，故缓冲区占用率是较为平滑曲线。
- 数据流的数量越多，缓冲区占用率变化越平稳，每个流都经历偶尔的丢包、滑窗减半
- 对其中单个流，滑窗减半可能出现在任意时刻，这取决于流何时恰好遇到缓冲区满而丢包。
- 由于流数量多，因此每个流的cwnd的值都较小，因此可以**认为RTT保持不变**，这点与单个流完全不同。由于滑窗大小可变，因此发包率R=W/RTT也可变。
- 在稳定工作状态时，各个参数的变化为：
  - 滑窗大小：成锯齿形变化
  - **包往返时间RTT：保持不变**
  - 瓶颈链路使用率：始终保持100%
  - 路由器缓冲区占用率：成锯齿形变化

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/5-8-AIMD_multiflows.jpg" width="600px">



在下图中：

​		连续两个丢包间总共发送 $$A=(W_{max}+\frac{W_{max}}{2})*\frac{RTT}{2}=(W_{max}+\frac{W_{max}}{2})*\frac{W_{max}}{2}=\frac{3}{8}W_{max}^2$$

​		丢包率 $$p=\frac{1}{A}$$

​		发包率 $$R=\frac{W(t)}{RTT}=\frac{(\frac{W_{max}}{2}+W_{max})/2}{RTT}=\frac{3W_{max}}{4RTT}=\sqrt{\frac{3}{2}}\frac{1}{RTT\sqrt{p}}(packets/s)$$

​		将 $$R$$ 的单位从 $$packets/s $$ 转化到 $$bits/s$$，只需乘以包大小

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/5-9-AIMD_multiflows.jpg" width="500px">

由于RTT与传输距离有关，远距离传输时传输速率有天然弱势，这也是AIMD的缺点





## TCP阻塞控制 - Tahoe

在TCP刚问世时，并未加入拥塞控制，后来网络频繁因拥塞而出现瘫痪。直到1988，Van Jacobson提出了TCP拥塞控制方法，修正了TCP，一举解决拥塞控制问题，这一版本的TCP代号为TCP Tahoe。1990，一个Tahoe的改进版本被提出——Reno。随后，Reno不断改进，形成现代TCP拥塞控制方案。

TCP有三个最本质的问题，这也是Tahoe所增加的三个重要举措：

- 何时发送新数据  		->  1. **阻塞窗**
- 何时重传数据              ->  2. **超时timeout估计**
- 何时发送确认ACK       ->  3. **自时钟self-clocking**

发送端对发送窗口大小进行控制，控制规则为：**Sender window = $$min$$(flow window, congestion window)**

- flow window：进行流控制的发送端的窗大小，避免发送端发送超过接受端处理能力的数据
- congestion window (cwnd)：进行阻塞控制时的发送端的窗大小，避免发送端发送超过网络运输能力的数据



#### 阻塞窗Congestion window (cwnd)

##### CA和SS两种状态

阻塞窗的大小根据网络所处状态进行变化，共有两种状态：

- Congestion avoidance 拥塞避免状态：在稳定传输时，通过AIMD实现
  - 每收到一个确认段ACK，阻塞窗大小增加 MSS*MSS/cwnd，其中MSS表示Maximum segment size，即一个包的大小
  - 阻塞窗的增加呈线性形式：每过一个RTT，阻塞窗大小增加一个MSS
- slow start 慢启动状态：连接开始时、或包传输超时进入该状态，希望快速找到合适的窗大小，不遵循AIMD
  - 连接开始时，阻塞窗的大小为MSS，即一个包大小，且每收到一个确认段ACK，阻塞窗大小增加一个MSS
  - 阻塞窗的增加呈指数形式：最开始窗为1，发送1个包，收到1个ACK；窗增加到2，发送2个包，收到2个ACK；窗增加到4，发送4个包，收到4个ACK.....
  - 指数增加却称其为slow start，是因为这比不进行阻塞控制（第一次即发送整个窗口大小的数据包）更慢

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/5-10-Tahoe-two_states.jpg" width="400px">

##### CA和SS的状态转换

slow start 和 Congestion avoidance两状态之间的转换，通过有限状态机进行转换

- 转换的目标：
  - slow start阶段快速找到合适的阻塞窗大小cwnd和对应的网络容量
  - 网络负载接近网络容量时，congestion avoidance 精细控制窗大小
- 转换时可参考：
  - ACK数值不断增加，表示传输良好；
  - 重复三次接受确认号ack相同的ACK（即4次同样的ack号），表示有包丢失； 
  - 超时Timeout，表示网络状态极差，超时还未收到任何ACK

**TCP Tahoe 有限状态机**

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/5-11-Tahoe-FSM.jpg" width="500px">

**TCP Tahoe 行为模式**

比AIMD的行为更加保守，一丢包（超时、收到重复确认号）就直接进入 slow start 状态

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/5-12-Tahoe_behaviow.jpg" width="450px">

小例子：某网络可支持发送者的最大窗口长度是16个包长度，ssthresh=8，初始窗大小1，窗大小的变换是：

​		1，2，4，8，9，10，...，16，17，（重复该序列）



#### 超时估计 Timeout estimation

由TCP Tahoe有限状态机可知，当网络发包超时后，会从Congestion avoidance进入slow start状态。超时时间的估计非常重要，超时时间太短，会频繁触发slow start状态，浪费网络容量，超时时间太长则会浪费大量时间等待ACK，因此估计方案为：

- RTT为不变的常数时，超时时间可设为比RTT稍大的值

但是，RTT是高度动态变化的，它随网络负载动态变化。因此需要对超时时间进行鲁棒估计。

##### pre-Tahoe 超时估计

- 估计策略
  $$r$$ 为 RTT 的估计值，初始化为一个合理值，$$m$$ 是最近一次ACK的包的 RTT 测量值

  令RTT的估计值为指数加权移动平均： $$r=\alpha r+(1-\alpha)m$$

  则超时估计为 $$Timeout=\beta r=2r$$

- 存在问题：假定RTT估计值 $$m$$ 的方差是常数，上述估计没考虑到 $$m$$ 的分散程度（方差），比如下列两种情况

  - 绝大多数RTT估计为80-81ms，超时估计一直为160ms，这就造成了容量浪费
  - RTT估计均在为20-100ms，当2r<100ms，就存在一些未丢失的包被判为超时，进而发生不必要的重传

##### Tahoe超时估计

为克服上述问题，Tahoe超时估计将RTT测量值和其方差都考虑在内

- 估计策略

  $$r$$ 为 RTT 的估计值，初始化为一个合理值，$$m$$ 是最近一次ACK的包的 RTT 测量值，RTT估计误差 $$e=m-r$$

  RTT的估计值和方差分别进行指数加权移动平均：$$r=r+ge, v = v +g(|e|-v)$$，$$g$$ 为经验值 $$g=0.25$$

  则超时估计为 $$Timeout=r+\beta v=r+4v$$

**效果对比**

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/5-13-Tahoe_timeout_estimation.jpg" width="790px">

#### 自时钟 self-clocking

自时钟解决接收端何时发送ACK的问题，其思想是发送ACK越快越好，即接收端收到数据后立即返回ACK：

- 接收端接收到数据后立即发送ACK，ACK是一个告知发送端数据包已离开网络的重要信号
- 只有数据包离开网络（发送端收到ACK）时，发送端发送新数据包进入网络
- 发送者收到ACK，用于更新RTT估计值、方差，进而更新Timeout估计
- 有利于充分利用网络容量



#### TCP Tahoe的包发送、接收过程

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/5-14-Tahoe_conparison.jpg" width="690px">





## TCP阻塞控制 - Reno

#### TCP的改进措施

Tahoe对阻塞窗cwnd的控制比AIMD更加保守，丢包就直接重置阻塞窗 cwnd=1 并进入 slow start。Reno对Tahoe这一保守行为进行了改进，加入了 fast retransmit、fast recovery（快速重传和快速恢复机制）

##### Fast retransmit (TCP Tahoe中已使用)

接收方重复收到4个确认号相同的ACK即认定包已丢失，立即重新发送该确认号对应包，将ssthresh=cwnd/2， cwnd =1，进入 slow start 状态，而非等到超时未收到ACK再重传（在TCP Tahoe的有限状态机里有体现，这里重新说一下，加深印象）

##### Fast recovery (Reno和Tahoe的主要区别)

- 对包丢失后处理机制的改进
  - 传送超时：超时未收到ACK，认为丢失，ssthresh=cwnd/2， cwnd =1，进入slow start，与Tahoe相同
  - 包丢失：接收方重复收到3个确认号相同的ACK（4个相同的ACK），进入fast recovery状态，在未收到新确认号的ACK前一直处于该状态，并ssthresh=cwnd/2，**cwnd=cwnd/2**，此时仍遵循遵循AIMD机制
- cwnd膨胀机制：在fast recovery状态时，每收到一个重复的ACK，cwnd增加1，包括最开始重复的3个ACK
  - 在某个包丢失后，TCP需要重发该包，然后等待收到对应的ACK之后再增加cwnd以发送后面的包，但是该ACK需要一个RTT之后才能收到。由于收到的每个重复的ACK都对应着有一个包离开了网络，因此理论上讲此时再往网络中发送一个新的包并不会造成网络阻塞。
  - 假设cwnd在减半前为$$C_{old}$$，减半后为$$C_{new}=C_{old}/2$$，收到新的ACK需要经过一个RTT，这期间收到$$C_{old}$$个重复的ACK，经过一个RTT之后，窗口大小增长到$$C_{new}=3C_{old}/2$$，即**新发送$$C_{old}/2$$个包进入网络**
  - 收到新的ACK后，退出fast recovery状态，此时重新令cwnd=cwnd/2

#### Reno的状态机及状态转换

Reno在Tahoe的基础上，加入了Fast recovery机制加以改进，两种情况：

- 包传送超时，其行为模式和Tahoe完全一样
- 包丢失时，其行为模式为Fast recovery，并加入了cwnd膨胀机制

**Reno的有限状态机**

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/5-16-Reno_FSM.jpg" width="600px">

**Reno的行为机制**

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/5-15-Reno_behavior.jpg" width="500px">



#### TCP Reno的包发送、接收过程

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/5-17-Reno_sending_receiving_mechanism.jpg" width="600px">



#### 小例子

某网络可承受的最大cwnd=16pkts，网络的RTT估计为常数1s。使用Tahoe和Reno在该网络中传输数据，达到稳定状态后，Tahoe和Reno的传输速率分别是多少？（以pkts/s为单位），不考虑超时

**对Tahoe**

网络稳定时，ssthresh=16/2=8pkts，在慢启动SS状态和阻塞避免CA状态循环切换，两状态发包数量分别为：

- SS状态：发送1+2+4+8pkts，并分别持续一个RRT时间，即4*RTT=4s，共发送15pkts
- CA状态：发送9+10+11+12+13+14+15+16+16\*+16pkts，并分别持续一个RTT时间，即10*RTT=10s，共发送132pkts。其中，16\*那一个RTT内发送了17个包，其中一个被丢弃；最后一个16的前三次ACK重复16\*中被丢弃包的ACK，因此四次相同ACK触发SS（该RTT内仍发送出去16pkts），然后立即进入SS状态

综上，两个状态持续时间$$4+10=14s$$，共发送$$15+132=147pkts$$，因此传输速率是

​		$$147pkts/14s = 10.5pkts/s$$

**对Reno**

网络稳定时，ssthresh=16/2=8pkts，始终处于阻塞避免CA状态，因cwnd膨胀机制，存在两个阶段：

- CA状态：发送8+9+10+11+12+13+14+15+16+16\*+16pkts，并分别持续一个RTT时间，即11\*RTT=11s，共发送140pkts。其中，16\*那一个RTT内发送了17个包，其中一个被丢弃，最后一个16的前三次ACK重复16*中被丢弃包的ACK，四次相同ACK触发fast recovery（该RTT内仍发出16pkts），然后进入cwnd膨胀阶段
- cwnd膨胀阶段：触发fast recovery后，需要再过一个RTT才能收到新的ACK后再进入CA状态。在该RTT=1s内，一共收到17个相同的ACK，因此阻塞窗膨胀cwnd=16/2+17=25，进而新发送25-16=9pkts

综上，两个阶段总持续时间$$11+1=12s$$，共发送$$140+9=149pkts$$，因此传输速率是

​		$$149pkts/12s = 12.42pkts/s$$





根据end-to-end的原则，网络的工作是尽量高效、灵活地传输数据报文，其它所有工作都应该放在端完成

# 应用层

## NATs

#### NATs简介

NAT - Network Address Translation 网络地址转换，它是一种通过修改数据包IP ，将一个IP地址空间映射到另一个IP地址空间的方法。它是一种为解决IPv4地址短缺而流行起来的技术。

NATs有两侧，一侧面向公网，一侧面向局域网（本地子网），NATs负责在公网和局域网之间传递信息

- NAT即映射：（私有IP，内部端口）<=>（公共IP，外部端口）

  <img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-1-NAT_example.jpg" width="500px">

- NAT可以将多个终端的私有IP地址转换成一个共享的公共IP地址，使得：

  - 多个终端可共用一个公共IP地址
  - 不同的公共IP地址域内可以使用同一个私有IP地址，因为这些私有IP地址只在该公共IP地址域内有效

- NAT提供为IP提供安全属性，由于IP地址隐藏在NAT后，攻击者很难直接打开IP连接，形成一道”防火墙“

- NAT收到的包时内部向网络中发出时才进行地址转换

**NATs的两个问题**

1. NATs允许哪些包进行映射
2. NATs何时以及如何进行映射

注意：后续描述将使用RFC3489中的术语和分类进行叙述



#### NATs的类型

- NATs有多种实现方式，分为非对称和对称NATs，即
  - 非对称NATs（下列前三种）：同一个客户主机（IP : port），跟不同服务器主机连接，NAT分配相同端口
  - 对称NATs（下列第四种）：同一个客户主机（IP : port），跟不同的服务器主机连接，NAT分配不同端口



##### Full Cone NAT 完全圆锥形NAT

- 非对称NAT
- 内部地址映射到外部地址：(iAddr : iport）<==>（eAddr : eport），i代表内部internal，e代表外部external
- 任意外部主机的任意端口（anyAddr : anyPort）都可以发包到（eAddr : eport），进而发到（iAddr : iport）
- 当内部主机（iAddr : iport）发现收到的数据来自错误主机、或者来自正确主机的错误端口时，会发送ICMP error或其他方式告知对方，并同时丢弃这个包

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-2-fullNAT.jpg" width="400px">

##### Restricted cone NAT 受限圆锥形NAT

- 非对称NAT
- 内部地址映射到外部地址，同时包含目标地址IP： (iAddr : iport, destAddr) <==> (eAddr : eport, destAddr)
- 目标主机的任意端口（destAddr : anyPort）都可以发包到（eAddr : eport），进而发到（iAddr : iport）
- 当错误主机发包到（eAddr : eport），NAT将不予放行，并发送ICMP error或其它方式告知对方，并丢弃包；当（iAddr : iport）发现收到的数据来自目标主机的错误端口时，会发送ICMP error告知，并同时丢弃这个包

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-3-restrictedNAT.jpg" width="400px">

##### Port restricted coned NAT 端口受限圆锥形NAT

- 非对称NAT

- 内部地址映射到外部地址，同时包含目标地址IP和端口

  （iAddr : iport, destAddr : destPort）<==>（eAddr : eport, destAddr : destPort）

- 只有目标主机的目标端口（destAddr : destPort）可以发包到（eAddr : eport），进而发到（iAddr : iport）

- 当错误主机或者错误端口发包到（eAddr : eport）时，NAT将不予放行，并发送ICMP error或其它方式告知，同时丢弃这个包

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-4-port_restrictedNAT.jpg" width="400px">

##### Symmetric NAT 对称形NAT

- 是端口受限圆锥形的对称NAT
- 每一个来自相同的（iAddr : iport），到达不同的（destAddr : destPort），都映射到一个不同的（eAddr : eport）
- 只有曾经收到过内部主机数据的外部主机，才能够把数据包发回

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-5-symmetricNAT.jpg" width="400px">

#### NAT需要对特殊情况进行处理

由于NAT类型多、行为复杂，因此会有很多特殊情况需要进行特殊处理，比如Hari pinning这种情况。

如下图，一个NAT后有两个主机A、B，NAT为A的端口4512建立映射 10.0.0.101:4512 <==> 128.34.22.8:6641，并采用full cone NAT类型，则下列情况需要进行特殊处理：

1. B向A发送包pkt（DA: 128.34.22.8:6641，SA：10.0.0.99:X），包pkt经由交换机传送到NAT，由于包并未离开NAT进入外网，因此pkt中的SA不会被转换。
2. 当NAT收到pkt后，对DA进行转换得到pkt_mapped（DA: 10.0.0.101:4512，SA：10.0.0.99:X），然后根据DA将转换后的包发送到主机A
3. A收到包pkt_mapped后，向B回复包pkt_re（DA：10.0.0.99:X，SA:10.0.0.101:4512），包pkt_re传输到交换机后，根据DA判断目标地址位于局域网内，因此不会发送到NAT，而直接将包发送到主机B
4. B收到包pke_re后发现其源地址为10.0.0.101:4512，与自己发送的包pkt的DA不一致，因此该包被丢弃

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-6_NAT_spcial_case.jpg" width="690px">

NAT类型很多、行为很复杂，也需要考虑很多种特殊情况，因此RFC5382为TCP中的NAT提供了一些建议，RFC4787则为UDP提供了NAT建议



#### 位于NAT后的主机不能有传入连接

如果主机src想与另一位于NAT后的主机dest建立连接，src会向NAT发送连接请求包。但是，NAT中主机dest地址的映射（iAddr : iport）<==>（eAddr : eport）只在dest向外发包时才会建立，因此NAT收到src的请求时该映射还未建立，NAT不知道该如何映射该请求内的地址和端口，即NAT后的主机不能有传入连接

建立传入连接的方法：

##### Connection Reversal 连接反转

适用情况：一个使用公网IP的终端向另一个位于NAT后的私有IP发起连接请求，如下图，A位于NAT后，B直接使用公网。此时，A可以主动建立与B的连接，但是B不能主动建立与A的连接。

解决办法：建立一个中转站R（不能在NAT后），当B想主动建立与A的连接时，将请求发送给R。R收到连接请求后将请求发送给A，A收到请求后再由A主动发出与B的连接。

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-7-NAT_connection_reversal.jpg" width="500px">

##### Relays 中继

适用情况：两个均位于NAT后的私有IP互相发出连接请求，如下图，此时A与B均不能主动建立与对方的连接。

解决方法：建立一个中转站R（不能在NAT后），当A想建立与B连接时，将请求发送给R，由R转送给A告知B的公网地址和端口号；A也通过R告知B自己的公网地址和端口号。

注意：这种方法破坏了端到端的原则，加了一个中继器进行通信

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-8-NAT_relays.jpg" width="500px">

##### NAT Hole-Punching

NAT Hole-Punching是对中继Relays方法的改进

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-9-NAT_hole_punching.jpg" width="300px">

解决方法：

1. A向NAT发送包，此时NAT会为A建立其内网IP和端口与公网IP和端口的映射，NAT将包发送到Server。Server向A回复包，告知A其公网IP和端口。B同样操作获取自身的公网IP和端口。此时双方都已创建映射
2. A向Server发送包，询问B的公网IP和port，服务器Server向A回复包，告知B的公网IP和port。这时A可以向B发送连接请求，B同理。

适用范围：

1. Full cone NAT 可行：可接受任意主机的任意端口发来的包
2. Restricted cone NAT / port-restricted cone NAT 也可行：Server告知A和B对方的公网IP和port，然后A和B同时向对方的公网IP和port发送包。在向对方发送包的同时就会创立映射。
3. symmetric NAT 不可行：因为A在尝试建立与B的连接时，A的NAT会重新分配端口号，B亦如此，因此无法建立两个端口之间的连接



#### NATs的优缺点

**缺点**

1. 违背了分层原则，NATs需要修改IP（网络层首部）和端口号（运输层首部）
2. 使用NATs时，必须知道运输层协议（TCP/UDP/ICMP），才能在包内对应位置找到端口号和IP地址进行修改
3. 阻碍了产生新的运输层协议：NATs不会在协议广泛应用之前去适配协议，而协议不会在NATs适配前得到广泛应用。因此应用层基本只能使用TCP/UDP/ICMP作为运输层协议。现在往往通过对UDP进行修改实现新的协议，因为UDP几乎只提供了一个数据容器，如何实现功能由用户决定

**优点**

1. 解决了IPv4地址资源枯竭的问题，可以实现地址重用
2. 安全性较高，攻击者无法与位于NAT后的主机直接建立连接





## HTTP超文本传输协议 - HyperText Transfer Protocol

#### HTTP请求模型

超文本HyperText是一种文档格式，允许在文档中包含格式化信息和内容信息，它是一种ASCII文本。浏览器加载一个网页即加载一个超文本文档，浏览器基于超文本文档内的特殊格式控制符——标签，来显示它。使用超文本，可以将文档或文件嵌入到其它文件中，如图片、其它页面、样式表、脚本等。

请求过程：客户端通过TCP连接写入请求，服务器读取并处理请求，然后向TCP写入响应，客户端读取响应

##### 请求格式 Request Format

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-10-HTPP_request.jpg" width="400px">

​		method：请求方式，如GET/POST

​		URL：域名、资源名

​		version：HTTP版本，如HTTP/1.1

​		空格space/←/↓：空格/返回本行开头/跳转到下一行

​		headers：零个或多个headers，每个占一行，包含header名和值，所有headers之后空一行

​		body：方法为GET时body为空，方法为POST时body非空

##### 响应格式 Response Format

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-12-HTTP_response.jpg" width="400px">

与Request类似，其中不同处有：

​		status code：状态码

​		phrase：与状态码关联的短语，如200 OK（请求被接受），404 Not Found

例如chrome打开http://www.google.com，通过开发者模式可查看请求和回复（chrome处理过）如下：

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-11-HTTP_request_google.jpg" width="600px">



#### HTTP/1.0

##### HTTP/1.0的请求过程

客户端先建立连接，然后向服务器发出请求，网页可能包含以URL形式显示的图片或文件等，客户端接收完网页所有内容后，服务器断开连接。<font color="green">当客户端想请求网页内的图片或文件所在URL时，再次建立连接并发出请求</font>

- 客户端打开连接
- 请求页面内容 GET
- 服务器响应
- 响应结束后关闭连接
- 请求网页内的其它URL，重复第一步

##### HTTP/1.0请求时间计算

**条件**：1. 客户端和服务器之间的延迟为50ms；2. 请求大小为1个完整TCP报文，响应大小为2个完成TCP报文；3. 一个报文的封包延迟为10ms，故请求的封包延迟10ms，响应的封包延迟为20ms；4. 没有数据的TCP报文没有封包延迟，如SYN、ACK报文；5. 链路为全双工，节点可以同时接收和发送报文，互不影响；6. 最多可打开4个连接

Case1：打开单个网页

​	SYN：50ms，SYN/ACK：50ms，ACK/request：50+10ms，response：50+20ms，Total：230ms

Case2：打开包含2张图片的单个网页

​	Step1 - 请求网页：同Case1，共230ms

​	Step2 - 使用额外两个连接请求图片：SYN50ms，SYN/ACK50ms，ACK/request + response：150ms

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-13-HTTP_speed.jpg" width="450px">



**条件**：客户端服务器延迟20ms，请求为1报文，响应为2报文，封包延迟5ms，全双工，最多可同时打开2个连接

​	Case1：打开单个网页 -> 95ms

​	Case2：打开包含1张图片的单个网页 -> 190ms

​	Case3：打开包含2张图片的单个网页 -> 200ms

​	Case4：打开包含5张图片的单个网页 -> 380ms  ((网页请求 + 响应 共95ms) + 3*(图片请求 + 响应 共95ms))<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-14-HTTP_speed2.jpg" width="800px">



**条件**：RTT延迟为20ms，TCP报文长度1.4Kb，Tahoe协议，初始cwnd=2，忽略封包延迟，无丢包发生

​	Case1：打开单个2.8Kb网页的时间  -> SYN + SYN/ACK：RTT，ACK/req + res：RTT，Total 2RTTs

​	Case2：打开单个15.4Kb网页的时间 ->  SYN + SYN/ACK：RTT，ACK/req + res ：3RTTs，Total 4RTTs

​			解释：假设一直处于slow start状态，每经过一个RTT，cwnd翻倍，因此每个RTT中传输文件大小为

​						2.8Kb(cwnd=2) + 5.6Kb(cwnd=4) + 7Kb(cwnd=8) = 15.4Kb，共三组send+ACK

​	Case3：打开单个大小为15.4Kb网页的时间（处于congestion avoidance状态）-> 1RTT + 4RTTs = 5RTTs

​			解释：处于congestion avoidance状态，每经过一个RTT，cwnd += 1，因此每个RTT中传输文件

​						2.8Kb(cwnd=2) + 4.2Kb(cwnd=3) + 5.6Kb(cwnd=4) + 2.8Kb(cwnd=5) = 15.4，共四组send+ACK



##### HTTP/1.0的缺点

这种请求过程在网页内只有文本，或者包含少量图片/内嵌文件时性能很好。如果网页包含大量图片或嵌入文件时，传输较慢，原因：

1. 当页面内容传输完毕后该连接就直接关闭，再次请求图片/内嵌文件时则重新连接，建立连接用时较多
2. 每个连接都有各自的cwnd，因此cwnd没有机会增长到较大



#### HTTP/1.1

##### 改进措施

- 在请求request中添加Headers：
  - keep-alive：告诉服务器“请保持连接，我还会请求更多内容”
  - close：告诉服务器，回复完成后直接关闭连接
  - 服务器根据情况决定，不完全依照request内header的要求
- 在回复response中添加Headers：
  - keep-alive：告诉客户端“我保持连接不关闭，保持多久”
  - close：告诉客户端”我关闭了连接“

所有的传输均在同一个连接内完成

##### HTTP/1.1请求时间计算

**条件**：RTT为100ms，请求为1报文，响应2报文，cwnd=3报文，封包延迟1ms，全双工，可同时打开2连接

​	Case1：使用HTTP/1.0，打开包含11张图片的网页

​		网页请求 + 响应 共203ms + 6*(图片请求 + 响应 共203ms) = 7 * 203ms = 1421ms

​		视频里是 7*203，但为什么不是 (1+11)\*203？

​	Case1：使用HTTP/1.1，打开包含11张图片的网页

​		网页请求 + 响应 203ms + (图片请求51ms + 响应 72ms) = 326ms，其中

​			图片请求51ms = 1ms封包 + 50ms延迟

​			图片响应72ms = 11*2ms封包 + 50ms延迟

**条件**：客户端和服务器RTT固定110ms，瓶颈链路速度为8Mbps，假定cwnd>110KB时丢包，初始cwnd=10KB，使用TCP Reno

​	Case1：使用HTTP/1.0，请求1个20KB的文件，用时多少，使用了瓶颈链路的多少容量？   330ms  6%

​		SYN+SYN/ACK：110ms，10KB数据传输req+res：110ms，10KB数据传输(ack+res)，总共330ms

​		传输20KB用时330ms，使用了容量占比为 (20KB/330ms) / 1MB/s = 6%

​	Case2：使用HTTP/1.1，请求5个20KB文件，用时多少，使用了瓶颈链路的多少容量？	550ms  19%

​		SYN+SYN/ACK：110ms

​		10KB req+res：110ms		20KB ack+res：110ms，40KB ack+res：110ms		30KB ack+res：110ms

​		传输100KB用时550ms，使用了容量占比 (100KB/550ms) / 1MB/s = 19%

​	Case3：使用HTTP/1.1，请求30个20KB文件（总共60KB），用时多少？

​		由于是单个流，因此在第一次丢包（带*）后，网络就以最大流速传输

​		SYN+SYN/ACK：1RTT

​		文件传输：每个RTT内传输 10KB + 20KB + 40KB + 80KB + *110KB + 110KB + 110KB + 110KB + 10KB

​		总用时 1RTT + 9RTTs = 10RTTs = 1100ms



#### SPDY

SPDY是谷歌提出的一种新的网页传输协议，用于加速网页打开速度

- 它允许服务器回复顺序与请求顺序不一致
  - 如加载一个动态生成的网页，加载整个页面很慢，但是其中大部分内容是简单的图片和文字，则可以先将它们加载并显示，然后再加载动画部分
- 删除了HTTP中冗余的headers
- HTTP/2.0的基础



## BitTorrent

BitTorrent允许客户端之间共享、交换大体积文件。一个客户端可以并行地向另外拥有该文件的客户端请求文件。大体积文件被拆分成块。当客户端A下载了该文件，A会通知其他客户端它有这个文件，其他客户端就可以从A这里下载该文件，这些客户端被称为集群。一个客户端通过下载一个.torrent文件加入到一个集群中。

#### .torrent文件

**普通的.torrent文件**：

- 记录了文件体积、块的大小、如何与集群内其它客户端建立连接、元数据（torrent建立者等）等信息
- 指定了一个tracker，用于记录集群中所有客户端的信息
  - 当有新的客户端new加入时，new会向tracker请求集群内的客户端的信息
  - new得到客户端信息后，通过TCP连接与这些客户端进行数据交换
  - 一次可以建立最多100个TCP连接
- .torrent中记录了tracker的信息，招致不必要的关注，2000s开始使用无追踪trackerless的torrent

**无追踪的.torrent文件**：

- 通过该.torrent文件与一个客户端取得连接，由它告知如何加入DHT（Distributed hash table分布式哈希表）
- DHT提供散列值与节点（客户端）之间的映射关系，与集中式地将信息放置在tracker中不同，DHT将所有映射分布式地存储在各个客户端，即信息存储在集群内的许多客户端中

#### 文件存储策略

##### 以块为单位进行存储

BitTorrent将文件拆分为若干大小为256KB~1MB的块，这个大小可以使阻塞窗口能增长到足够大以提供较高传输速率；同时BitTorrent还会将每个块分为更小的块，以便从更多客户端请求文件，降低下载时间。块是BitTorrent用来检查文件完整性的基本单位。

一个BitTorrent文件包含了每个块的SHA1散列值，这是一种加密哈希功能，它为每个块创建一个独有的哈希值。当块SHA1值不匹配时，即认为文件不完整，此时块被丢弃并尝试重新请求下载。多次提供错误SHA1的客户端会被认为是恶意攻击而加入黑名单。

当存在连接时，客户端之间会互相交换信息告知自己持有哪些块

#### 文件下载策略

##### rarest-first稀有优先策略

新加入的客户端优先下载持有者最少的块；当文件的某一块不被任何客户端持有时，该文件不可被下载；某些块被极少数客户端持有时，这些客户端会成为下载过程的瓶颈端。例外：当文件快下载完成时，会向多个客户端请求剩余部分，并丢弃重复接收的块，避免某个速度较慢的客户端影响整体下载速度。

##### tit-for-tat针锋相对策略

该策略用于挑选集群内速度最快的客户端。在请求下载文件时，为了避免与大量慢速客户端建立连接，BitTorrent会优先向曾给它发送过数据的客户端发送数据（有点绕），这一思想通过阻塞choking的方式实现：

集群中大部分客户端处于被阻塞状态，BitTorrent不会向它们发送数据；每过30s，BitTorrent会阻塞所有客户端，然后向所有客户端发送数据，并测试传输速率，从中选取最快的4或$$\sqrt{n}$$个（$$n$$为客户端总数）个客户端，并与之建立连接然后下载数据。

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-15-bitTorrent_tit_for_tac.jpg" width="500px">

#### BitTorrent总结

1. .torrent文件描述了如何下载文件
2. 文件被分割为块，每个块有一个特定的SHA1哈希值
3. 客户端通过tracker或者分布式哈希表DHT定位集群内的其它客户端
4. 客户端通过TCP/IP建立连接
5. 客户端定期交换信息告知自己持有哪些块
6. 客户端下载文件时遵循rarest-first和tit-for-tat策略

​		

## DNS - domain name system 域名系统

#### DNS简介

一个资源定位符URL(uniform resources locators)主要有3个基本部分

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-16-URL_parsing.jpg" width="450px">

<font color="gree">http://</font>：指定了应用层协议为HTTP，并指定了端口默认为80，也可以在Host后面添加 :portNum 指定另一端口

<font color="blue">cs144/scs/stanford/edu</font>：人类可读的域名，指定了一个主机的域名，它对应一个网络层的IP

<font color="red">/labs/sc.html</font>：指定了位于应用层的想访问的资源名

**存在问题**：输入上述URL时，使用人类可读的域名来替代实际想访问的IP地址，如何将之转换为IP？

**解决**：

- 初期，由Network Information Center维护一个host.txt，内含网络上所有的主机名与对应的IP地址，用户定期从维护host.txt的IP上下载该文件以此来实现主机名与IP的转换，该方法在网络很大时很难应用。
- 域名系统 Domain Name System
  - 可以进行人类可读域名到IP地址的映射，可以很好处理大规模的记录
  - 分布式控制，各个IP对它自己所拥有的人类可读域名有控制权
  - 对单个节点的崩溃具有鲁棒性

**DNS设计时需关注的特征**

1. 只读、或者主读：主机对DNS的读取查找远多于修改
2. 宽松的一致性要求：修改可以有一定的延迟

根据上述特征，DNS可以使用大规模缓存，建立域名（人类可读的主机域名）与IP的映射。客户端请求获得该映射之后再持有一段较长的时间（获取后缓存在本地），当发现该映射失效后再重新请求。

#### DNS架构

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-17-DNS_architecture.jpg" width="500px">

1. DNS采用分级、各层级分区域的体系：根节点—顶级域名—域名—子域名
   - 顶端是DNS的根节点，或者称为 .(dot)
   - 下一级是顶级域名top-level domains (TLD)，如edu、com、org、cn等
   - 再下一级即我们常说的域名，如stanford.edu，cisco.com，acm.org，baidu.cn等
   - 域名的所有者可以提供额外的域名（子域）
2. DNS每个层级、每个区域分别进行管理
   - cn可以将baidu授权给其拥有者，baidu完全自行控制和管理其旗下所有子域并对各子域进行再次授权
   - 所有向baidu.cn的请求均由其所有者自行进行处理
3. 每个区域（edu、google、baidu等）的服务器都有多个备份，单个服务器故障时其余服务器仍可提供服务
4. 根域名服务器具有超高的鲁棒性
   - 全球有13个的根域名服务器（A~M），这些服务器的IP地址储存在DNS服务器上，DNS服务器的IP可以通过DHCP获得
   - 通过任意广播anycast机制高度复制，在全球各地有多个备份，客户端访问root服务器时选择最近一个
   - 针对root根服务器的大规模分布式拒绝服务攻击(DDoS攻击 large-scale distributed denial-of-service)从未成功过，因为root根服务器工作简单、备份多、鲁棒性高。（提一点DDoS，即攻击者利用“肉鸡”对目标网站在较短时间发起大量请求，大规模消耗其主机资源，让它无法正常服务）

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-18-DNS_root_server.jpg" width="700px">



#### DNS query请求

##### DNS query

- DNS请求使用的协议
  - UDP协议，端口port=53，有512B的长度限制
  - TCP协议，端口port=53，没有长度限制，头部中通过一个16位的长度字段告知流的长度
- DNS请求的两种类型
  - 递归式Recursive：客户端设置使用的DNS服务器一般是递归服务器，全权负责处理客户端的DNS查询请求，直到返回最终结果
  - 非递归式Non-recursive：DNS服务器之间一般采用迭代查询

##### DNS域名解析过程

解析 w<font size="0"> </font>ww.stanford.edu的过程：

1. 客户端通过DHCP得到一个DNS服务器（也称为DNS解析器resolver）的IP
2. 发送DNS查询报文 `Q = query www.stanford.edu` 到该DNS服务器
3. DNS服务器检查缓存，存在记录则直接返回；如果记录老化或者不存在，则：
4. DNS服务器向根域名服务器发送查询报文`Q`，根域名服务器返回TLD顶级域.edu的域名服务器IP地址
5. DNS服务器向.edu域名服务器发送查询报文`Q`，得到二级域名服务器.stanford.edu的域名服务器IP地址
6. DNS服务器向.stanford.edu域名服务器发送查询报文，得到w<font size="0"> </font>ww.stanford.edu的IP地址
7. DNS服务器将查询结果（.edu、stanford.edu、w<font size="0"> </font>ww.stanford.edu）存入自身缓存同时，将查询请求的的结果返回给客户端

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-19-DNS_resolve.jpg" width="700px">



#### 资源记录RR (Resource Records)

##### DNS报文的结构

一条典型的DNS报文的数据区中的内容如下图所示，除了Header以外的每一部分都是由一条条资源记录RRs组成

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-20-DNS_msg_structure.jpg" width="400px">

通过dig tool来监测在发送DNS请求时的DNS报文如下图：

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-21-DNS_msg.jpg" width="450px">

Header中包含了DNS报文的一些必要信息

Question记录了DNS请求

Answer记录了DNS响应

Authority记录了所请求域名授权了哪些域名

Additional记录了所授权的域名的IPv4/IPv6地址等信息

***Header的结构***

- RR的Header如下

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-22-DNS_msg_header.jpg" width="500px">

- 其中的每一段表示
  - ID：ID识别号，用于将请求和响应一一对应
  - QR：0=query，1=response
  - OPCODE：0=标准请求standard query
  - RCODE：错误码
  - Flags标志位：
    - AA：authoritative answer
    - TC：truncated
    - RD：recursion desired
    - RA：recursion available

***除了Header以外的其它内容***

- RR中除了Header的其它内容的结构如下，它们每一个都是一条资源记录RRs

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-23-DNS_msg_contents.jpg" width="500px">

##### 资源记录RRs的结构

所有的DNS信息和消息都以资源记录Resource Records (RRs)的形式表示，其结构如下：

<center><font color="red">name [TTL] [class] type rdata</font></center>

- name：域名，比如 stanford.edu，可能经过了名字压缩，长度不定
- TTL：生存时间，以s为单位，长度为4bytes
- class：等级，通常为IN，表示Internet，长度为1byte
- type：资源记录的类型，长度为1byte，通常为：
  - A=1：IPv4地址，此时rdata对应的是IPv4地址
  - AAAA：IPv6地址，此时rdata对应的是IPv6地址
  - NS=2：域名服务器地址，此时rdata是name授权的子域名（可能经过了名字压缩）
  - CNAME：规范名称记录canonical name record，此时rdata是与name相关联的域名（即别名）
  - MX：邮件交换记录Mail eXchange record，此时rdata是与name对应的可以接受邮件的邮件服务器名
  - TXT：任意文本，一般用于扩展
- rdata：与type对应的资源数据

##### 典型的资源记录RR

***DNS A 资源记录***

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-24-DNS_A_record.jpg" width="550px">

***DNS NS 资源记录***

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-25-DNS-NS_record.jpg" width="550px">

##### 域名压缩机制

DNS为了能够尽量使所有信息不超过512Bytes，通常使用域名压缩机制：

- DNS中域名的记录分层级，每一个层级一个标签
  - 比如www. stanford. com，其中www为一个标签，stanford为一个标签，com为一个标签
- 标签的存储格式为长度+内容，长度为一个数字，内容采用ASCII
  - 比如标签www，存储为3www，对应的16进制源码为0x3777 0x7777，其中0x03表示长度为3bytes，后面的三个0x77表示'w'的ASCII=119
- DNS报文中多次出现同一标签，则通过偏置实现
  - 比如第一次出现stanford时从第26bytes开始，那么第二次出现stanford时则即为0xc01a
  - 0xc0 (1100,0000=192)表示该第二次出现标签的位置大于了192，后面的14bits用来表示偏置量0x1a==28
- 必须有相同的后缀才能使用域名压缩机制

例：DNS响应中出现了域名cheese.sandwich.foodie.com，下列可以享受域名压缩带来的好处的有（蓝色部分）

​	[√] <font color="blue">sandwich.foodie.com</font>

​	[√] <font color="blue">foodie.com</font>

​	[√] <font color="blue">cheese.sandwich.foodie.com</font>

​	[√] www.<font color="blue">cheese.sandwich.foodie.com</font>

例：DNS响应中出现了cs.berkeley.edu，下列可以享受域名压缩带来的好处的有（蓝色部分）

​	[√] <font color="blue">cs.berkeley.edu</font>

​	[√] eecs.<font color="blue">berkeley.edu</font>

​	[√] cs.stanford.<font color="blue">edu</font>

​	[×] cs.berkeley.com	没有相同的后缀，因此无法享受好处

例：一条RR的16进制源码为 <font color="blue">c0 2e</font> <font color="red">00 01</font> <font color="green">00 01</font> <font color="gree">00 00 02 6d</font> <font color="pink">00 04</font> <font color="purple">ab 43 d7 c8</font>，下列正确的有

<font color="blue">NAME</font>：c0指示了后面的14bit (2e=46)是一个偏置，用于只是压缩后的NAME的位置

<font color="red">TYPE</font>：00 01 = 1，表示这是一条IPv4地址

<font color="green">CLASS</font>：00 01 = 1，表示类型为IN

<font color="gree">TTL</font>：00 00 02 6d = 621，表示存活时间为621s

<font color="pink">RDLENGTH</font>：00 04 = 4，表示数据长度为4bytes

<font color="purple">rdata</font>：ab 43 d7 c8，对应的IPv4地址为171.67.215.200



## DHCP

#### DHCP简介

DHCP全称为Dynamic Host Configuration Protocol，动态主机配置协议

主机在加入某个网络后，需要使用相关信息——本机的的IP、子网掩码、网关、DNS服务器来进行网络配置，如下图所示，这些也是使用网络的必要配置信息，而DHCP则是用于或者这些配置的协议

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-26-3things_need_to_use_internet.jpg" width="450px">

在初期，主机配置的分配采用固定式的，这样的分配策略的有很多缺点，如：

1. 上网地点更换之后，配置就失效了
2. 配置的使用权限通常以年为单位，一但到期就需要重新分配
3. 难有效回收空置的主机配置

因此，到后来就逐渐更换为DHCP，即每次新加入一个网络后，都动态地自动获取相关信息来对主机进行网络配置

#### DHCP的配置步骤

DHCP通过四（或五）个骤来实现主机配置的动态分配：discover, offer, request, ack, (release)

- discover：当一个主机连上网络后，需要向网络中的DHCP服务器请求主机配置，这时主机向网络中广播发送一条消息，告知网络中的DHCP服务器以索要相关配置信息
- offer：网络中的DHCP服务器（可能不止一台）收到配置请求后，向请求发送者提供可用的主机配置信息
- request：主机收到多条offer之后，从中选择一个，并向对应的DHCP服务器发送正式请求request。（注意，也同时向其它DHCP服务器发送，以便收回所提供的配置）
- ack：对应的DHCP服务器收到正式的配置请求后，回复ack确认该配置生效。
- release：当配置的使用期限到期时，服务器收回并释放该配置。如果客户端还想继续使用则会重新请求该配置，延长使用期。当客户端不再使用配置后，可以提前释放

<img src="C:/Users/xiaol/OneDrive/%E8%8B%A6%E9%80%BC%E7%A0%81%E5%86%9C/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/img/6-27-DCHP-process.jpg" width="700px">

##### 未配置前无IP的处理方法

在主机获取配置之前，它并没有自己的IP地址，那么它是如何与DHCP服务器进行通信的呢？

- 主机使用UDP协议，向网络中广播发送一条UDP报文，源端口src port = 68，源地址src IP = 0.0.0.0，目的IP = 255.255.255.255，目的端口dest port = 67，表示这是一条用于DHCP请求的广播。
- DHCP服务器收到请求后，由于可能服务器和客户端不在同一链路的两端，因此DHCP服务器将响应广播到所有链路，收到该广播的非发出请求的主机或者路由器会将该响应继续广播，直到发出该请求的主机接收到响应，然后选择其中的配置信息来初始化自己的IP

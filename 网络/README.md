# 网络基础
## TCP/IP协议族
TCP/IP协议族分层管理：应用层、传输层、网络层和数据链路层。

**应用层**包括：FTP(File Transfer Protocol, 文件传输协议)，DNS(Domain Name System, 域名系统)和HTTP(HyperText Transfer Protocol, 超文本传输协议)等。

**传输层**包括：TCP(Transmission Control Protocol, 传输控制协议)和UDP(User Data Protocol, 用户数据报协议)。

**网络层**：规定了通过怎样的路径（传输路线）到达对方计算机，并把数据包传输给对方。**数据包**是网络传输的最小数据单位。

**数据链路层**：用来处理网络的硬件部分，网卡，光纤，操作系统等。

网络通信发生时，发送端从应用层往下走，接收端往应用层往上走。发送端每经过一层，就会打上一个对应层的首部信息；接收端每经过一层就会消去对应曾的首部信息。这种把数据消息包装起来的做法称为 **封装**。

### 负责传输的IP协议
IP(Internet Protocol, 网际协议)位于网络层，作用是传送各种数据包。传送需要IP地址和MAC(Media Access Control, 媒体访问控制)地址。IP地址可变，MAC地址一般不变。

ARP(Address Resolution Protocol, 地址解析协议)可以把IP地址解析为MAC地址。如果不在同一个局域网内，需要中转到下一个MAC地址。就像快递公司送货过程。

## 确保可靠性的TCP协议
TCP位于传输层，提供可靠的字节流服务。字节流服务是指把大块数据分割成以 **报文段** 为单位的数据包。可靠是指数据包可以准确的送达对方。为了确保这种准确，TCP采用 **三次握手** 策略。

## 负责域名解析的DNS
DNS协议可以通过域名查找IP,也可以通过IP反查域名。

## 请求网页的过程：
1. 客户端输入域名，DNS返回对应的IP地址
2. HTTP生成请求报文
3. TCP把报文按顺序分割成一个个报文段，并通过三次握手传送给对方
4. IP协议查询MAC地址，并把报文段通过一个个中转站最终传送到目标MAC地址
5. TCP通过三次握手接收到一个个报文段，并把报文段按顺序重组成请求报文
6. HTTP协议处理请求报文并返回响应资源。

## URI、URL和URN
URI(Uniform Resource Identifer, 统一资源标识符)，URL(Uniform Resource Locator, 统一资源标识符)和URN(Uniform Resource Name, 统一资源名称)

URN和URL是URI的子集。URL和URN一定是URI，反之不一定成立。形如`http://abc.om`既是URI也是URL。但是`abc.com`是URI却不是URL，因为`http://abc.com`和`ftp://abc.com`代表了两个不同的locator。

映射到生活中，我说起赵日天你一定知道我说的是谁（URN），但是你要去赵日天家里做客，你必须要知道他家的地址（URL)。

# 三次握手
首先要搞清楚三次握手是为了确保数据准确无误的送达到目标而进行的一种策略。

SYN(Synchronize Sequence Numbers, 同步序列编号)和ACK(Acknowledgement, 确认字符)都是TCP的 **标志(flag)**。
1. 发送端：SYN = m
2. 接收端：SYN = m + 1, ACK = n
3. 发送端：ACK = n + 1
4. 三次握手结束
5. 开始一次请求和响应

# 四次挥手

# 从输入URL到浏览器显示页面发生了什么
## 网络通信
1. **应用层DNS解析域名**。先从本地客户端检查是否有对应的ip，有则返回，没有则请求上级DNS服务器，直到找到或到达根节点
2. **应用层客户端发送HTTP请求**。
3. **传输层TCP传输报文**。传输策略是三次握手。
4. **网络层IP协议查询MAC地址**。
5. **数据到达数据链路层**。
6. **服务器接收数据**。
7. **服务器响应请求**。
8. **服务器返回相应文件**。
## 页面渲染
1. 解析HTML，构建DOM树。
2. 解析CSS，形成CSS对象模型。
3. 用DOM树和CSS对象模型构建渲染树。
4. 布局和绘制。 
## 地址类型

- Unicast: 单播，终端访问互联网最常用的地址，唯一标识一个网络接口
- Anycast: 任播，一类特殊的IP地址，多个网络接口（不同的设备）都配上相同的地址，往这个地址发送数据的时候，路由器会只发往其中的一个接口，一般发往最近的那一个
- Multicast: 多播，代表一类unicast的集合，往这个地址发送数据的时候，会将数据发给属于这个多播组的每个unicast地址，部分地址已经被用于某些协议，如ff0X::101对应NTP服务器（授时）

**特殊地址**

| 类型                 | 前缀           | IPv6 表示方法 |
| :------------------- | :------------- | :------------ |
| Unspecified          | 00…00 (128 位) | ::/128        |
| Loopback             | 00…01 (128 位) | ::1/128       |
| Multicast            | 11111111       | FF00::/8      |
| Link-Local unicast   | 1111111010     | FE80::/10     |
| Unique local address | 1111110        | FC00::/7      |
| Global Unicast       | 所有其它       |               |

- 全0的地址 ::/128 为未定义地址
- 除了最后一位是 1 ，其它都是 0 的地址 ::1/128 为本地环回地址，同IPv4的127.0.0.1
- FF00::/8 这个网段的地址都是多播地址
- FE80::/10 为 Link-Local 的单播地址，这类地址不能穿过路由器
- FC00::/7 为本地的单播地址，可以在局域网内路由
- **全局的单播地址目前只有 2000::/3 开头的可以被申请使用，其它的都被预留了**



## 地址分配方式

### SLAAC

[Stateless Auto Configuration](https://tools.ietf.org/html/rfc2462)(无状态自动分配)，使用邻居发现协议(NDP)对自身进行自动配置，即插即用。

一种基本的情形就是本地的链路地址(Link-Local，fe80::/10)：系统启动后，会为每个网卡生成一个Link-Local的IP地址：fe80:: + INTERFACE_ID，该地址于本地有效，只能用于本地的通信，其中包括了到IPv6网关的通信

对于平时通信使用的IPv6地址的生成方式和上面类似：设备接入网络后，系统会通过广播的方式寻找网络中的路由器（RS，后面有说明），路由器会返回或者定期广播（RA）子网前缀，系统将子网前缀和接口ID组合起来，就构成了一个唯一的IP地址，这个IP地址可以通过路由器路由

> PPPoE的SLAAC地址由来
>
> PPPoE拨号在发起连接的时候采用的MAC地址和设备网卡的地址可能没有关联，如linux使用的pppd工具建立PPPoE连接的时候采用的随机MAC，Windows经过测试应该和拨号的账号有关，故每次拨号所获得的IPv6地址的后缀也可能不一样



### DHCPv6

有状态分配(Stateful Auto Configuration)，使用DHCP服务器向下级设备分配任意的地址。

其中DHCP唯一标识符（DUID）用于客户端从DHCPv6服务器获得IP地址，服务器将DUID与其数据库进行比较，并将配置数据（地址、租期、DNS服务器，等等）发送给客户端

### DHCP-PD

前缀下发(PD，Prefix delegation): **DHCP-PD**会分配一个地址范围（比如一次分一个/60），路由自行分配避免冲突

> 注意区分PD与前缀，PD从内容上来说是前缀，但是前缀不一定是PD，比如64位的前缀



## 邻居发现协议 

Neighbor Discovery Protocol，**NDP**

### NDP消息

5种**ICMPv6报文**类型或者叫**NDP消息**

- 路由器请求(RS)：由主机发起，用来请求RA
- 路由器通告(RA)：由路由器主动发起/响应RA，通告路由器的存在和链路的细节参数(默认路由地址、网络前缀，MTU，跳数限制等)，周期性发送，应答RS
- 邻居请求(NS)：由主机发起，用来请求另一台主机的链路层地址，或实现地址冲突检测、邻居不可达检测
- 邻居通告(NA)：邻居主动发起/响应NS，如果一个节点改变了他的链路层地址，邻居能主动发送一个NA消息通知
- 重定向(Redirect)：当在本地链路上存在一个更好的到达目的网络的路由器时，路由器需要通告节点来进行相应配置改变

### NDP代理

路由器穿透
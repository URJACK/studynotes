# WireShark

世界上应用最广泛的数据包抓取管理软件。

## 前言

### 监听原理

网络监听软件是一种“监视网络状态”、“数据流程”、“网络上的信息传输”的管理工具

network <-> TAP(with sniffer) <-> network 。两个网络之间，可以监听数据包。

### 依赖

windows: winpcap + wireshark

linux: libpcap + wireshark

## 操作

### 数据包界面

抓包的主要界面

![image-20200713225434871](.\a.png)

导出包

![image-20200713225619364](.\b.png)

### 过滤器

#### 显示过滤器

数据包界面中，第一行的filter，会在抓取得到数据包中，按照filter进行挑选

#### 捕获过滤器

从抓包阶段就不再抓取这类数据包

### 常见过滤表达式

![image-20200713230139242](.\c.png)

## 分析常见的基础协议

### ARP

先清空一下arp缓存

```
PS C:\Users\scffz> arp -d
```

查看arp表

```
PS C:\Users\scffz> arp -a

接口: 192.168.31.207 --- 0x5
  Internet 地址         物理地址              类型
  192.168.31.1          ec-41-18-ee-c4-03     动态
  224.0.0.22            01-00-5e-00-00-16     静态

接口: 192.168.43.1 --- 0x10
  Internet 地址         物理地址              类型
  224.0.0.22            01-00-5e-00-00-16     静态

接口: 169.254.54.251 --- 0x14
  Internet 地址         物理地址              类型
  224.0.0.22            01-00-5e-00-00-16     静态
```

以下是ARP报文的交互方式(InterCor是本机电脑，xiaomiCo是我的手机)

wireshark之所以知道这个是小米，另一个是英特尔，是因为它内部有一些MAC地址的映射信息，能够根据MAC地址得出具体的生产厂家。这个软件是真的牛皮。

![image-20200713232333957](.\d.png)

其中，上图中的第一个数据包的详细信息如下

![image-20200713232951696](.\e.png)

### ICMP

Ping测试到远程主机的连通性：

客户端向服务器发送Echo Request

服务器发送Echo Reply返回客户端计算机

客户端通过检查Echo Reply得出通信质量

![image-20200713233944318](.\f.png)

### DNS查询

DNS查询可以一次性查询多条，但是UDP报文最大大小似乎是512字节，所以当DNS查询的报文过大的时候，DNS查询就会以TCP为基础来进行查询了。

![image-20200714001715473](.\g.png)

### TCP三次握手

在实验中，访问我自己的web服务，他建立了两个TCP链接。不过这并不影响理解

Info信息这一列，代表source port -> destination port，可以看到两端究竟是那两个端口建立起了链接

![image-20200714092706110](.\h.png)

实际上，针对TCP，还可以点开数据包仔细核实其内容。

sequence number 和 acknowledge number，wireshark都会帮我们自动转为相对序号。当然绝对序号也是可见的

![image-20200714093051935](.\i.png)

### Telnet

对telnet的包进行追踪

![image-20200714093724933](.\j.png)

追踪流显示

![image-20200714093932933](D:\LearningNotes\net\wireshark\k.png)
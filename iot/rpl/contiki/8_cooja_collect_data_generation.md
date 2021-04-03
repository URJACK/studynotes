# Cooja数据生成

结合上文，我们已经理清数据的来历。也因此得知了如何才能采集到数据。
但显然仅仅这样是不够的，这里里边儿有几个麻烦的地方：

1. 我们采集到数据的方式，仅仅是在Mote Output中，如果我们要使用GA，这意味着我们至少需要提供一个js版本的GA。
2. js版本的GA的运行速率很难得到保证，之前的GA在迭代过程中，lossFunction 是确切的，loss是可以快速计算得出的，但在这里loss却不然，必须完全依赖cooja软件的仿真结果。还需要不断对过往数据进行清理。
3. 同时我们也不清楚，js在Simulation script中，他支持的内存等数据的上限能有多少。或者说会不会有其他不稳定的地方？
4. 退一步说，使用修改CollectView的java源码来做这样的统计有是否可行呢？
   我觉得难度也挺大，主要CollectView按道理说<u>是没有修改节点位置能力</u>的，要修改也只能去修改仿真器Simulation的内容，同时我们需要获得仿真器的诸如能耗等**指标信息**。

第二个想法我不太清楚难度怎么样，我想去溯源这个数据的来历，查看他能耗指标的数值具体是如何生成的。
我相信能耗的计算，并不需要太多的节点之间的交互信息就能够直接计算得出。

## 解析源码（udp-sender.c）

### 预备知识

`UIP_HTONS` 不难从中看出逻辑是将一个2Byte的数据，进行高低Byte互换。

```c
#ifndef UIP_HTONS
#   if UIP_BYTE_ORDER == UIP_BIG_ENDIAN
#      define UIP_HTONS(n) (n)
#      define UIP_HTONL(n) (n)
#   else /* UIP_BYTE_ORDER == UIP_BIG_ENDIAN */
#      define UIP_HTONS(n) (uint16_t)((((uint16_t) (n)) << 8) | (((uint16_t) (n)) >> 8))
#      define UIP_HTONL(n) (((uint32_t)UIP_HTONS(n) << 16) | UIP_HTONS((uint32_t)(n) >> 16))
#   endif /* UIP_BYTE_ORDER == UIP_BIG_ENDIAN */
#else
#error "UIP_HTONS already defined!"
#endif /* UIP_HTONS */
```

`udp_new()`这个函数会创建一个udp客户端`conn`？

```c
struct uip_udp_conn {
  uip_ipaddr_t ripaddr;   /**< The IP address of the remote peer. */
  uint16_t lport;        /**< The local port number in network byte order. */
  uint16_t rport;        /**< The remote port number in network byte order. */
  uint8_t  ttl;          /**< Default time-to-live. */

  /** The application state. */
  uip_udp_appstate_t appstate;
};
```

```c
struct uip_udp_conn *
    udp_new(const uip_ipaddr_t *ripaddr, uint16_t port, void *appstate)
{
    //我们最终需要返回的连接对象
    struct uip_udp_conn *c;
    //应用状态
    uip_udp_appstate_t *s;

    //生成连接对象的实例（传入ip地址与端口号）
    //注意，传入的port是rport，远端port
    //c内部的lport，本地port，会在uip_udp_new()函数中生成
    c = uip_udp_new(ripaddr, port);
    if(c == NULL) {
        //如果连接对象创建失败
        return NULL;
    }

    //设置链接的状态信息
    s = &c->appstate;
    /*
    #define PROCESS_CURRENT() process_current
	CCIF extern struct process *process_current;
    */
    s->p = PROCESS_CURRENT();
    s->state = appstate;

    return c;
}
```

`udp_bind()`会将`conn`对象进行绑定，查看`udp_bind`发现是一个宏定义函数，实际函数是`uip_udp_bind()`。

```
#define udp_bind(conn, port) uip_udp_bind(conn, port)
#define uip_udp_bind(conn, port) (conn)->lport = port
```

`udp_bind()`宏函数的介绍如示：

```c
/**
 * Bind a UDP connection to a local port.
 *
 * This function binds a UDP connection to a specified local port.
 *
 * When a connection is created with udp_new(), it gets a local port
 * number assigned automatically. If the application needs to bind the
 * connection to a specified local port, this function should be used.
 *
 * \note The port number must be provided in network byte order so a
 * conversion with UIP_HTONS() usually is necessary.
 *
 * \param conn A pointer to the UDP connection that is to be bound.
 * \param port The port number in network byte order to which to bind
 * the connection.
 */
```

主要代码

```c
PROCESS_THREAD(udp_client_process, ev, data)
{
    PROCESS_BEGIN();

    PROCESS_PAUSE();

    set_global_address();

    PRINTF("UDP client process started\n");

    print_local_addresses();

    /* new connection with remote host */
    //创建一个连接对象 ripaddr传入的是NULL 在uip.c文件中
    /*创建连接对象的时候，会分配一个新的rip地址内存区域。
      if(ripaddr == NULL) {
        memset(&conn->ripaddr, 0, sizeof(uip_ipaddr_t));
      } else {
        uip_ipaddr_copy(&conn->ripaddr, ripaddr);
      }
    */
    client_conn = udp_new(NULL, UIP_HTONS(UDP_SERVER_PORT), NULL);
    //将连接对象绑定在本地的端口上 #define UDP_CLIENT_PORT 8775
    /*
    其实这里使用这个函数挺让人费解的 因为使用udp_new()的时候
    client_conn->lport就已经生成出来了
    这里又特地单独给它重新绑定成为UDP_CLIENT_PORT 就有点离谱
    */
    udp_bind(client_conn, UIP_HTONS(UDP_CLIENT_PORT));

    PRINTF("Created a connection with the server ");
    PRINT6ADDR(&client_conn->ripaddr);
    PRINTF(" local/remote port %u/%u\n",
           UIP_HTONS(client_conn->lport), UIP_HTONS(client_conn->rport));

    while(1) {
        PROCESS_YIELD();
        if(ev == tcpip_event) {
            tcpip_handler();
        }
    }

    PROCESS_END();
}
```

我们从sender.c的源码中，没有找到发送相关数据的代码。

## 解析源码（uip_process）

在该文件 `~\examples\ipv6\rpl-collect\udp-sink.c` 中的 `tcpip_handler(void)` 函数中，sink节点对`collect-view` <u>发送了（通过Mote Output）</u>信息。

```c
static void
tcpip_handler(void)
{
  uint8_t *appdata;
  rimeaddr_t sender;
  uint8_t seqno;
  uint8_t hops;

  if (uip_newdata())
  {
    appdata = (uint8_t *)uip_appdata;
    sender.u8[0] = UIP_IP_BUF->srcipaddr.u8[15];
    sender.u8[1] = UIP_IP_BUF->srcipaddr.u8[14];
    seqno = *appdata;
    hops = uip_ds6_if.cur_hop_limit - UIP_IP_BUF->ttl + 1;
    collect_common_recv(&sender, seqno, hops,
                        appdata + 2, uip_datalen() - 2);
  }
}
```

我们发现它的数据部分，即 `appdata` ，来自于 `uip_appdata` 。
我们翻看 `uip_appdata` 的定义，发现它来自于三个文件： 
`uip.c` 与 `uip6.c` ，而这两个文件的定义是相同的，均为 `void* uip_appdata;`
`uip.h` 中，只是使用了 `extern void* uip_appdata;` 

```c
/* uip_process(flag):
 *
 * The actual uIP function which does all the work.
 */
void uip_process(uint8_t flag);  //uip.c from line 672 to 1947
```

这里面1276行代码，包含了大量的数据处理。
我们从中筛选出对 `uip_appdata` 有增改的语句和上下文。

### 头文件信息

在line-683行，有这样的一段代码：

```c
uip_sappdata = uip_appdata = &uip_buf[UIP_IPTCPH_LEN + UIP_LLH_LEN];
```

其中`UIP_IPTCPH_LEN`的数值为40，定义如下：

```c
#define UIP_IPTCPH_LEN (UIP_TCPH_LEN + UIP_IPH_LEN)    /* Size of (IP + TCP) header
(20 + 20) == 40 */
```

而`UIP_LLH_LEN` 的数值为14，定义如下

```c
#define UIP_LLH_LEN 14
```

这段语句那么是什么意思呢？我个人的解读如下：
首先`uip_buf`是一个缓存区，它用来接收数据包。一个数据包，必定包含包头信息，我们从上述两个宏定义的名称也可以推出来（诸如"***H_LEN"）。

包头信息长度，在上文中一共取了54位：TCP20位、IP20位、LL14位。
从这里，我们也就能顺带读出了通信的协议栈。

**总结**代码意思是，`uip_appdata`，<u>跳过了包头信息的字段，直指数据字段</u>。
（同时整个协议栈的通信是在**IP层的下面一层（LL层）**进行<u>互相通信</u>的）

### 其他的头文件信息

事实上，与line-683行具有相同功能的代码非常多：

```c
uip_sappdata = uip_appdata = &uip_buf[UIP_LLH_LEN + UIP_IPUDPH_LEN]; //line 815
uip_appdata = &uip_buf[UIP_LLH_LEN + UIP_IPUDPH_LEN];				 //line 1100
uip_sappdata = uip_appdata = &uip_buf[UIP_LLH_LEN + UIP_IPUDPH_LEN]; //line 1176
uip_appdata = &uip_buf[UIP_LLH_LEN + UIP_IPTCPH_LEN];				 //line 1208
```

直到line 1222才开始进入数据处理阶段：

```c
  /* TCP input processing. */
#if UIP_TCP
 tcp_input:
  UIP_STAT(++uip_stat.tcp.recv);
//....
```

### 第一次处理（URG处理）

#### 预备知识

`TCP_URG` 有如此宏定义 `#define TCP_URG 0x20` ：
`0x20` 转为二进制即为 `0010 0000` 。它的作用是为了进行**位与运算**，<u>提取有效位</u>。

`BUF` 有如此宏定义 `#define BUF ((struct uip_tcpip_hdr *)&uip_buf[UIP_LLH_LEN])` ：
含义是`uip_buf` 偏移 `UIP_LLH_LEN` 后的地址，从这个地址开始作为头地址，我们将其作为tcpip头文件信息的首地址。

```c
struct uip_tcpip_hdr {
#if UIP_CONF_IPV6
  /* IPv6 header. */
  uint8_t vtc,
    tcflow;
  uint16_t flow;
  uint8_t len[2];
  uint8_t proto, ttl;
  uip_ip6addr_t srcipaddr, destipaddr;
#else /* UIP_CONF_IPV6 */
  /* IPv4 header. */
  uint8_t vhl,
    tos,
    len[2],
    ipid[2],
    ipoffset[2],
    ttl,
    proto;
  uint16_t ipchksum;
  uip_ipaddr_t srcipaddr, destipaddr;
#endif /* UIP_CONF_IPV6 */
  
  /* TCP header. */
  uint16_t srcport,
    destport;
  uint8_t seqno[4],
    ackno[4],
    tcpoffset,
    flags,
    wnd[2];
  uint16_t tcpchksum;
  uint8_t urgp[2];
  uint8_t optdata[4];
};
```

我们从中可知，flags是属于TCP头信息的内容。它本身是一个单字节变量。

#### 处理过程

我们对uip_buf中的TCP_URG字段进行了校验，如果发现其携带了URG字段，那么我们就会对其做一些额外的处理（指跳过这个字段）。

下图中是一个处理过程的代码，其中需要注意`UIP_URGDATA` 的宏定义是： `#define UIP_URGDATA 0` 我也没有找到其他地方有其他宏定义。所以我们忽略了一部分不满足条件编译的代码。

```c
/* Check the URG flag. If this is set, the segment carries urgent
       data that we must pass to the application. */
if((BUF->flags & TCP_URG) != 0) {
    //
    #if UIP_URGDATA > 0
    //不满足条件编译的代码
    #else /* UIP_URGDATA > 0 */
    uip_appdata = ((char *)uip_appdata) + ((BUF->urgp[0] << 8) | BUF->urgp[1]);
    uip_len -= (BUF->urgp[0] << 8) | BUF->urgp[1];
    #endif /* UIP_URGDATA > 0 */
}
```

从上述代码可知，uip_appdata的组成结构是：

1. [`packet header`] 
2. [`URGDATA` if exist] 
3. [`data`]

## 解析源码（uip）

我们不难发现，很有可能整个的uip相关的源码中，都可能只是与数据相关的**传输部分**，至于其究竟有没有我们想要的数据的生成部分，这很难说。我认为大概率是没有的，所以我们需要找到<u>实际控制发送的API</u>，找到API的调用处。

事实上，除了`uip_process()`之外，还有许多的函数定义都可以在`uip.h`这个文件之中找到。
为了找到我们需要的API，我们将对这个文件进行解读。

### 结构体定义

source code line from 1 to 118之间，针对不同的协议（802.3/802.11）（ipv4/ipv6），定义了不同的结构体。

ipv4的地址结构体。

```c
typedef union uip_ip4addr_t {
  uint8_t  u8[4];			/* Initializer, must come first. */
  uint16_t u16[2];
} uip_ip4addr_t;
```

802.15.4的64bit长地址结构体。

```c
/** \brief 64 bit 802.15.4 address */
typedef struct uip_802154_longaddr {
  uint8_t addr[8];
} uip_802154_longaddr;
```

### 地址相关函数

source code line from 119 to 222之间，定义了诸多ip地址的相关函数。

传入一个**ip地址信息**，通过这些定义好的宏函数获取到相应的数据信息。

```c
#define uip_gethostaddr(addr) uip_ipaddr_copy((addr), &uip_hostaddr)
#define uip_getnetmask(addr) uip_ipaddr_copy((addr), &uip_netmask)
```

### ip设置

source code line from 232 to 307，有一些ip相关的设置函数

```c
void uip_init(void);
void uip_setipid(uint16_t id);
#define uip_input()        uip_process(UIP_DATA)
```

### TCP\UDP处理程序

source code line from 310 to 446，定义了TCP与UDP的处理流程代码。

```c
#if UIP_TCP
#define uip_periodic(conn) do { uip_conn = &uip_conns[conn];    \
    uip_process(UIP_TIMER); } while (0)

#define uip_conn_active(conn) (uip_conns[conn].tcpstateflags != UIP_CLOSED)

#define uip_periodic_conn(conn) do { uip_conn = conn;   \
    uip_process(UIP_TIMER); } while (0)

#define uip_poll_conn(conn) do { uip_conn = conn;       \
    uip_process(UIP_POLL_REQUEST); } while (0)

#endif /* UIP_TCP */
```

在上述代码中，我们看到了`conn`这个参数，
这个参数实际一般是一个整数值，从下方的调用代码中就可以看出来：

```c
//#define UIP_CONF_MAX_CONNECTIONS 10
//#define UIP_CONNS (UIP_CONF_MAX_CONNECTIONS)
for(i = 0; i < UIP_CONNS; ++i) {
    if(uip_conn_active(i)) {
        /* Only restart the timer if there are active
                 connections. */
        etimer_restart(&periodic);
        uip_periodic(i);
        //...
```

而`uip_conns`参数则是一个在 `uip.h` 文件中定义好的一个结构体 `struct uip_conn` 数组。

而`uip_conn`参数则是代表当前，`uip_conns`中被具体选择出的一个connection。

```
CCIF extern struct uip_conn *uip_conn;
CCIF extern struct uip_conn uip_conns[UIP_CONNS];
```

### 监听、发送

source code line from 500 to 862

```c
void uip_listen(uint16_t port);
void uip_unlisten(uint16_t port);
struct uip_conn *uip_connect(uip_ipaddr_t *ripaddr, uint16_t port);
#define uip_outstanding(conn) ((conn)->len)
CCIF void uip_send(const void *data, int len);

#define uip_newdata()   (uip_flags & UIP_NEWDATA)
#define uip_connected() (uip_flags & UIP_CONNECTED)
#define uip_closed()    (uip_flags & UIP_CLOSE)

#define uip_udp_send(len) uip_send((char *)uip_appdata, len)
```

这一部分代码中，包括了传统的监听函数和一些**发送函数**。很有可能就是这里边儿的一些发送函数了。

解析到这里，我们就可以先尝试去搜寻一下了。

`uip.c`的815行有如下代码：

```c
uip_sappdata = uip_appdata = &uip_buf[UIP_LLH_LEN + UIP_IPUDPH_LEN];
```

我们易得，这些data其实来自于`uip_buf`。

翻看定义，在 `~\core\dev\slip.c` 中`#define BUF ((struct uip_tcpip_hdr *)&uip_buf[UIP_LLH_LEN])`

在 `~\core\net\resolv.c` 中 `#define UIP_UDP_BUF ((struct uip_udpip_hdr *)&uip_buf[UIP_LLH_LEN])`

在 `~\core\net\sicslowpan.c` 中，有如下定义

```c
#define UIP_IP_BUF          ((struct uip_ip_hdr *)&uip_buf[UIP_LLH_LEN])
#define UIP_UDP_BUF          ((struct uip_udp_hdr *)&uip_buf[UIP_LLIPH_LEN])
#define UIP_TCP_BUF          ((struct uip_tcp_hdr *)&uip_buf[UIP_LLIPH_LEN])
#define UIP_ICMP_BUF          ((struct uip_icmp_hdr *)&uip_buf[UIP_LLIPH_LEN])
```

那么uip_buf自身本来是什么呢？在`uip.h`中，有如下定义。

```c
#define UIP_BUFSIZE (UIP_CONF_BUFFER_SIZE)

typedef union {
  uint32_t u32[(UIP_BUFSIZE + 3) / 4];
  uint8_t u8[UIP_BUFSIZE];
} uip_buf_t;
```

```c
//uip_buf 的宏定义 其本来指的是uip_aligned_buf
#define uip_buf (uip_aligned_buf.u8)
//定义一个缓存区
uip_buf_t uip_aligned_buf;
```

我们之前发现父节点是通过 `uip_*****` 的方式来获取子节点回传过来的数据的，所以我一直尝试从子结点中找到与父节点通信的函数。但是很无奈的是，我们没能找到一个合适的函数调用时机，也没能找到数据的生成来源。

## *解析源码（shell-collect-view）

### 预备知识

#### 结构体定义

在 `collect-view.h` 中，有如下的结构体定义。

```c
struct collect_view_data_msg {
  uint16_t len;
  uint16_t clock;
  uint16_t timesynch_time;
  uint16_t cpu;
  uint16_t lpm;
  uint16_t transmit;
  uint16_t listen;
  uint16_t parent;
  uint16_t parent_etx;
  uint16_t current_rtmetric;
  uint16_t num_neighbors;
  uint16_t beacon_interval;

  uint16_t sensors[10];
};
```

`energest_t`结构体的内容仅有一个`long`类型的`current`字段

```c
typedef struct {
  /*  unsigned long cumulative[2];*/
  unsigned long current;
} energest_t;
```

`energest_type`在后文中会有大量的使用，注意`ENERGEST_TYPE_MAX`定义在最后，就相当于指定了范围。

```c
enum energest_type {
  ENERGEST_TYPE_CPU,
  ENERGEST_TYPE_LPM,
  ENERGEST_TYPE_IRQ,
  ENERGEST_TYPE_LED_GREEN,
  ENERGEST_TYPE_LED_YELLOW,
  ENERGEST_TYPE_LED_RED,
  ENERGEST_TYPE_TRANSMIT,
  ENERGEST_TYPE_LISTEN,

  ENERGEST_TYPE_FLASH_READ,
  ENERGEST_TYPE_FLASH_WRITE,

  ENERGEST_TYPE_SENSORS,

  ENERGEST_TYPE_SERIAL,

  ENERGEST_TYPE_MAX
};
```

#### 构建函数

我们从这里找到了最相似的一部分代码：`collect-view.c`中的 `collect_view_construct_message()`。

```c
void
    collect_view_construct_message(struct collect_view_data_msg *msg,
                                   const rimeaddr_t *parent,
                                   uint16_t parent_etx,
                                   uint16_t current_rtmetric,
                                   uint16_t num_neighbors,
                                   uint16_t beacon_interval)
{
    //定义四个变量、用来记录最近一次的变量值
    static unsigned long last_cpu, last_lpm, last_transmit, last_listen;
    //显然这个与上面四个static变量对应起来，是当前这次的变量值
    unsigned long cpu, lpm, transmit, listen;
    /*结合之前的定义我们可以得知 collect_view_data_msg 的每个字段都是 2Byte
    所以我们这边用大小去除以 2Byte，就得到了字段的个数，即msg->len
    */
    msg->len = sizeof(struct collect_view_data_msg) / sizeof(uint16_t);
    //这里应该是能通过时钟获得当前的时间
    msg->clock = clock_time();
    //通常情况下TIMESYNCH_CONF_ENABLED是没有定义的，所以timesynch_time字段就为0
    msg->timesynch_time = 0;

    //该函数，会将正在使用的每个模式下的，总量（energest_total_time）进行更新
    energest_flush();
	
    //取得当前cpu\lpm\transmit\listen的能量时长。。与上次的能量时长作差，取得差值，代表间隔。
    cpu = energest_type_time(ENERGEST_TYPE_CPU) - last_cpu;
    lpm = energest_type_time(ENERGEST_TYPE_LPM) - last_lpm;
    transmit = energest_type_time(ENERGEST_TYPE_TRANSMIT) - last_transmit;
    listen = energest_type_time(ENERGEST_TYPE_LISTEN) - last_listen;

    while(cpu >= 65536ul || lpm >= 65536ul ||
          transmit >= 65536ul || listen >= 65536ul) {
    	/*因为我们最终msg中，塞入的数据大小只能是2Byte，
    	但很难保证二者间隔的数值一定能在2Byte以内，
    	所以当任意一个条目超过限制的时候，我们就会将他们的数值统一除以2（统一才能保持量纲）
    	*/
        cpu /= 2;
        lpm /= 2;
        transmit /= 2;
        listen /= 2;
    }
	//将信息塞入msg中 进而能正确返回给调用者
    msg->cpu = cpu;
    msg->lpm = lpm;
    msg->transmit = transmit;
    msg->listen = listen;
	
    //刷新一下记录值 因为在经过上述额外步骤的处理后 很有可能获得的时钟数值又发生改变了。
    last_cpu = energest_type_time(ENERGEST_TYPE_CPU);
    last_lpm = energest_type_time(ENERGEST_TYPE_LPM);
    last_transmit = energest_type_time(ENERGEST_TYPE_TRANSMIT);
    last_listen = energest_type_time(ENERGEST_TYPE_LISTEN);
	
    /*#define RIMEADDR_SIZE 2 
    相当于把parent的信息2Byte，代表地址，直接复制给msg的parent字段。
    */
    memcpy(&msg->parent, &parent->u8[RIMEADDR_SIZE - 2], 2);
    //依次复制相应的字段
    msg->parent_etx = parent_etx;
    msg->current_rtmetric = current_rtmetric;
    msg->num_neighbors = num_neighbors;
    msg->beacon_interval = beacon_interval;
	
    //传感器字段初始化值
    memset(msg->sensors, 0, sizeof(msg->sensors));
    collect_view_arch_read_sensors(msg);
}
```

##### energest_flush()

我们这里解读一下`energest_flush()`函数：

```c
/**
该函数，会将正在使用的每个模式下的，总量（energest_total_time）进行更新
*/
void 
    energest_flush(void)
{
    rtimer_clock_t now;
    int i;
    for(i = 0; i < ENERGEST_TYPE_MAX; i++) {
        //unsigned char energest_current_mode[ENERGEST_TYPE_MAX];
        //意味着，如果这个模式被开启，就会触发下面的这段函数
        if(energest_current_mode[i]) {
            //获取当前计时器的时间 结合后文可以推知这应该是一个 2Byte 变量
            now = RTIMER_NOW();
            /*typedef unsigned short rtimer_clock_t;
            
            energest_t energest_total_time[ENERGEST_TYPE_MAX];
            rtimer_clock_t energest_current_time[ENERGEST_TYPE_MAX];
         	
            这里也不难看出energest_total_time因为自身是long型，足够承受与开始的数值的差值。
            但是energest_current_time自身是short型，仅仅只是用来作为“统计增量”的工具人。
            */
            energest_total_time[i].current += (rtimer_clock_t)
                (now - energest_current_time[i]);
            energest_current_time[i] = now;
        }
    }
}
```

##### energest_type_time()

这里有两个实现，第一个如下，我个人认为不靠谱。

```c
unsigned long energest_type_time(int type) { return 0; }
```

第二个实现，才具有意义：

```c
/**
取得当前类型能量时长(total_time)
*/
unsigned long
    energest_type_time(int type)
{
    /* Note: does not support ENERGEST_CONF_LEVELDEVICE_LEVELS! */
    #ifndef ENERGEST_CONF_LEVELDEVICE_LEVELS
    if(energest_current_mode[type]) {
        //如果当前模式正在工作，那么先对total_time进行更新
        rtimer_clock_t now = RTIMER_NOW();
        energest_total_time[type].current += (rtimer_clock_t)
            (now - energest_current_time[type]);
        energest_current_time[type] = now;
    }
    #endif /* ENERGEST_CONF_LEVELDEVICE_LEVELS */
    //返回更新后的total_time
    return energest_total_time[type].current;
}
```

##### collect_view_arch_read_sensors()

初始化传感器的数值，我们这里查看的是 `~\apps\collect-view\collect-view-sky.c`

`collect-view-sky.c` 的该函数，而不是 `collect-view-z1.c` 的该函数。

这个函数里边儿有三个传感器，再获取他们的数据之前，我们需要先激活这些传感器。

获取到他们的数据之后我们就停止掉这些传感器。

```c
void
    collect_view_arch_read_sensors(struct collect_view_data_msg *msg)
{
    /**激活传感器 需要注意的是这里的几个变量的定义来自于
    #include "dev/light-sensor.h"
    #include "dev/battery-sensor.h"
    #include "dev/sht11-sensor.h"
    */
    SENSORS_ACTIVATE(light_sensor);
    SENSORS_ACTIVATE(battery_sensor);
    SENSORS_ACTIVATE(sht11_sensor);
	
    msg->sensors[BATTERY_VOLTAGE_SENSOR] = battery_sensor.value(0);
    msg->sensors[BATTERY_INDICATOR] = sht11_sensor.value(SHT11_SENSOR_BATTERY_INDICATOR);
    msg->sensors[LIGHT1_SENSOR] = light_sensor.value(LIGHT_SENSOR_PHOTOSYNTHETIC);
    msg->sensors[LIGHT2_SENSOR] = light_sensor.value(LIGHT_SENSOR_TOTAL_SOLAR);
    msg->sensors[TEMP_SENSOR] = sht11_sensor.value(SHT11_SENSOR_TEMP);
    msg->sensors[HUMIDITY_SENSOR] = sht11_sensor.value(SHT11_SENSOR_HUMIDITY);
	
	/**
	停止激活掉这三个传感器。
	*/
    SENSORS_DEACTIVATE(light_sensor);
    SENSORS_DEACTIVATE(battery_sensor);
    SENSORS_DEACTIVATE(sht11_sensor);
}
```


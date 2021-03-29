# Contiki的代码分析

contiki系统的udpsender，本代码分析从udp-sender.c开始。

## 头文件

![img](.\imgs_4\8[62CWZXXZ6SH74$%OCEL2U.png)

从之前仿真添加结点使用逻辑来看，每一个节点都使用了一个独立的代码：
例如 `"~/examples/ipv6/rpl-collect/udp-sender.c"`

显然每个代码又都是实现了rpl的逻辑，那必然意味着每个代码都引用了<u>RPL的相关代码</u>。
经过整理，得出"rpl.h"与"of.h"等等文件的逻辑关系。

## udp-sender.c

### 全局变量与声明进程

```c
static struct uip_udp_conn *client_conn;
static uip_ipaddr_t server_ipaddr;

/*----------------------------------------------------------*/

PROCESS(udp_client_process, "UDP client process");
AUTOSTART_PROCESSES(&udp_client_process, &collect_common_process);

/*----------------------------------------------------------*/
```

---

在该段代码中，

1. "struct uip_udp_conn" 的一个指针变量被定义了。udp_conn，代表的是连接的意思，指的应该是自己与服务器建立的连接实体。（当然，在系统初始化的时候，显然这个系统并没有与其他系统建立起任何的连接，所以这里使用指针类型的变量就是理所当然的了。）

2. 还定义了一个"uip_ipaddr_t"的变量"server_ipaddr"，这个变量显然应该是用来存储服务器地址的一个变量。

3. ```c
   PROCESS(udp_client_process, "UDP client process");
   ```

   PROCESS(process_name, "process description")宏用于声明一个进程；PROCESS_THREAD(process_name, event, data)宏用于定义进程执行主体。

   注意，在声明事件的时候，不需要单独去声明这个变量本身。（我们并没有在文件中去单独定义udp_client_process）
   
4. ```c
   AUTOSTART_PROCESSES(&udp_client_process, &collect_common_process);
   ```

   如果进程需要在系统启动时被自动执行，则可以使用AUTOSTART_PROCESSES(&process_name)宏。该宏可以指定多个进程，如AUTOSTART_PROCESSES(&process_1, &process_2)，表示process_1和process_2都会在系统启动时被启动。

### 进程

结合在cooja仿真过程中，打印日志的结果：

![image-20210115155032482](.\imgs_4\image-20210115155032482.png)

`Fig 1：cooja仿真中，一个mote的打印日志`

```c
PROCESS_THREAD(udp_client_process, ev, data)
{
  PROCESS_BEGIN();

  PROCESS_PAUSE();

  set_global_address();

  PRINTF("UDP client process started\n");

  print_local_addresses();

  /* new connection with remote host */
  client_conn = udp_new(NULL, UIP_HTONS(UDP_SERVER_PORT), NULL);
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
};
```

你会发现在 `UDP client process started` 该行命令执行之前，还有其他几句打印，在这里我分别整理了一下他们出现的位置。

| 编号 | 文件名                                  | 函数名            | 语句截取                                        |
| ---- | --------------------------------------- | ----------------- | ----------------------------------------------- |
| 1    | `~\platform\cooja\contiki-cooja-main.c` | set_rime_addr()   | "Rime started with address "                    |
| 2    | `~\platform\cooja\contiki-cooja-main.c` | contiki_init()    | printf("Node id is set to %u.\n", node_id);     |
| 3    | `~\platform\cooja\contiki-cooja-main.c` | contiki_init()    | printf("%s/%s/%s, channel check rate %lu Hz\n", |
| 4    | `~\platform\cooja\contiki-cooja-main.c` | contiki_init()    | printf("Tentative link-local IPv6 address ");   |
| 5    | `~\platform\cooja\contiki-cooja-main.c` | print_processes() | printf("Starting");...                          |

在上述代码中，4号，说明了，宏定义中的“WITH_UIP6” 是已经被启用了的。

5号，会打印进程的名称，而这个名称是在`udp-sender.c`中，注册进程时候，使用的代码就是

```c
PROCESS(udp_client_process, "UDP client process");
```

综上，我们可以得出每个结点必定会启用的代码就是`contiki-cooja-main.c`这个程序吗？

经过验证后，发现并不是，但最终我们还是找到了最终的文件：`~\platform\sky\contiki-sky-main.c` ，对于<u>不同平台</u>的节点来说，他们一来的platform文件就会有不同，我们这里统一都使用sky平台。

以下是成功修改调试信息的截图。

![image-20210318100526518](.\imgs_4\image-20210318100526518.png)

### Handler

在udp-sender中，handler的代码非常简单：

```c
tcpip_handler(void)
{
  if(uip_newdata()) {
    /* Ignore incoming data */
  }
}
```

就是忽略新来的数据。

## udp-sink.c

### Handler

进程的声明与全局变量的声明都与`udp-sender.c`中如出一辙，所以在这里不做过多的讲述。
因为在仿真过程中，我们发现sink node一直在持续不断的发信息，所以我们在这里对sink node的Handler进行代码的解读。

```c
static void
tcpip_handler(void)
{
  //应用数据 传入的是一个地址
  uint8_t *appdata;
  //发送者信息
  rimeaddr_t sender;
  //seqNumber 这里是指的是tcp 里的sequence Number吗？
  uint8_t seqno;
  //跳数？ 为什么这里会出现跳数？
  uint8_t hops;

  /*下面这一段数据处理挺让人迷惑的*/
  if(uip_newdata()) {
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

被调用的函数 `collect_common_recv()` 

```c
void
collect_common_recv(const rimeaddr_t *originator, uint8_t seqno, uint8_t hops,
                    uint8_t *payload, uint16_t payload_len)
{
  unsigned long time;
  uint16_t data;
  int i;

  //appdata 开头两个字节没有要 就成了 payload
  printf("%u", 8 + payload_len / 2);  
  //Fangzhou Debug Line
  printf(" | ");
  /* Timestamp. Ignore time synch for now. */
  time = get_time();
  //这里可以看出 时间的字节应该是4字节（32位） 这里分布打印出时间的 高2字节（高16位） 和 低2字节（低16位）
  printf(" %lu %lu 0", ((time >> 16) & 0xffff), time & 0xffff);
  //Fangzhou Debug Line
  printf(" | ");
  /* Ignore latency for now */
  printf(" %u %u %u %u",
         originator->u8[0] + (originator->u8[1] << 8), seqno, hops, 0);
  //Fangzhou Debug Line
  printf(" | ");
  for(i = 0; i < payload_len / 2; i++) {
    memcpy(&data, payload, sizeof(data));
    payload += sizeof(data);
    printf(" %u", data);
  }
  printf("\n");
  leds_blink();
}
```

## rpl.h

该文件的开头注释中，清晰的注明：

```
 * This file is part of the Contiki operating system.
 *
 * \file
 *	Public API declarations for ContikiRPL.
```

该文件本身是Contiki系统的一部分，再结合该文件是提供ContikiRPL的API，这显然意味着，该文件可以看成是Contiki系统的RPL实现规范。

### 接口函数

需要注意的是，所有的接口函数，都建立在条件编译：“#if UIP_CONF_IPV6” 这个条件下，必须有该宏的定义，才能有如下函数的定义。

```c
/* Public RPL functions. */
void rpl_init(void);
void uip_rpl_input(void);
rpl_dag_t *rpl_set_root(uint8_t instance_id, uip_ipaddr_t * dag_id);
int rpl_set_prefix(rpl_dag_t *dag, uip_ipaddr_t *prefix, unsigned len);
int rpl_repair_root(uint8_t instance_id);
int rpl_set_default_route(rpl_instance_t *instance, uip_ipaddr_t *from);
rpl_dag_t *rpl_get_any_dag(void);
rpl_instance_t *rpl_get_instance(uint8_t instance_id);
void rpl_update_header_empty(void);
int rpl_update_header_final(uip_ipaddr_t *addr);
int rpl_verify_header(int);
void rpl_insert_header(void);
void rpl_remove_header(void);
uint8_t rpl_invert_header(void);
uip_ipaddr_t *rpl_get_parent_ipaddr(rpl_parent_t *nbr);
rpl_rank_t rpl_get_parent_rank(uip_lladdr_t *addr);
uint16_t rpl_get_parent_link_metric(uip_lladdr_t *addr);
void rpl_dag_init(void);
```

#### rpl_init

```c
void rpl_init(void)
{
  uip_ipaddr_t rplmaddr;
  PRINTF("RPL started\n");
  default_instance = NULL;

  rpl_dag_init();
  rpl_reset_periodic_timer();

  /* add rpl multicast address */
  uip_create_linklocal_rplnodes_mcast(&rplmaddr);
  uip_ds6_maddr_add(&rplmaddr);

#if RPL_CONF_STATS
  memset(&rpl_stats, 0, sizeof(rpl_stats));
#endif
}
```


# Contiki的代码分析

contiki系统的udpsender，本代码分析从udp-sender.c开始。

## 头文件

![img](.\imgs_4\8[62CWZXXZ6SH74$%OCEL2U.png)

从之前仿真添加结点使用逻辑来看，每一个节点都使用了一个独立的代码：
例如 `"~/examples/ipv6/rpl-collect/udp-sender.c"`

显然每个代码又都是实现了rpl的逻辑，那必然意味着每个代码都引用了<u>RPL的相关代码</u>。
经过整理，得出"rpl.h"与"of.h"等等文件的逻辑关系。

下面是文件编译的信息，这体现了文件的先后顺序。

     [java]  INFO [AWT-EventQueue-0] (CompileContiki.java:140) - > make udp-sender.sky TARGET=sky 
     [java] 
     [java] COMPILATION OUTPUT:
     [java] 
     [java] 
     [java] > make udp-sender.sky TARGET=sky 
     [java] mkdir obj_sky
     [java]   CC        ../../../core/net/rime/rimeaddr.c
     [java]   CC        ../../../core/net/rime/timesynch.c
     [java]   CC        ../../../core/net/rime/rimestats.c
     [java]   CC        ../../../core/net/mac/cxmac.c
     [java]   CC        ../../../core/net/mac/xmac.c
     [java] ../../../core/net/mac/xmac.c: In function ‘send_packet’:
     [java] ../../../core/net/mac/xmac.c:457:7: warning: variable ‘ret’ set but not used [-Wunused-but-set-variable]
     [java]   CC        ../../../core/net/mac/nullmac.c
     [java]   CC        ../../../core/net/mac/lpp.c
     [java] ../../../core/net/mac/lpp.c: In function ‘input_packet’:
     [java] ../../../core/net/mac/lpp.c:783:7: warning: variable ‘ret’ set but not used [-Wunused-but-set-variable]
     [java]   CC        ../../../core/net/mac/frame802154.c
     [java]   CC        ../../../core/net/mac/sicslowmac.c
     [java]   CC        ../../../core/net/mac/nullrdc.c
     [java] ../../../core/net/mac/nullrdc.c: In function ‘packet_input’:
     [java] ../../../core/net/mac/nullrdc.c:288:12: warning: variable ‘original_dataptr’ set but not used [-Wunused-but-set-variable]
     [java] ../../../core/net/mac/nullrdc.c:287:7: warning: variable ‘original_datalen’ set but not used [-Wunused-but-set-variable]
     [java]   CC        ../../../core/net/mac/nullrdc-noframer.c
     [java]   CC        ../../../core/net/mac/mac.c
     [java]   CC        ../../../core/net/mac/framer-nullmac.c
     [java]   CC        ../../../core/net/mac/framer-802154.c
     [java]   CC        ../../../core/net/mac/csma.c
     [java]   CC        ../../../core/net/mac/contikimac.c
     [java] ../../../core/net/mac/contikimac.c: In function ‘powercycle’:
     [java] ../../../core/net/mac/contikimac.c:386:27: warning: variable ‘t0’ set but not used [-Wunused-but-set-variable]
     [java] ../../../core/net/mac/contikimac.c: In function ‘send_packet’:
     [java] ../../../core/net/mac/contikimac.c:766:11: warning: variable ‘ret’ set but not used [-Wunused-but-set-variable]
     [java] ../../../core/net/mac/contikimac.c:544:11: warning: variable ‘is_reliable’ set but not used [-Wunused-but-set-variable]
     [java]   CC        ../../../core/net/mac/phase.c
     [java]   CC        ../../../core/net/rpl/rpl.c
     [java]   CC        ../../../core/net/rpl/rpl-dag.c
     [java]   CC        ../../../core/net/rpl/rpl-icmp6.c
     [java]   CC        ../../../core/net/rpl/rpl-timers.c
     [java]   CC        ../../../core/net/rpl/rpl-mrhof.c
     [java]   CC        ../../../core/net/rpl/rpl-ext-header.c
     [java]   CC        ../../../core/sys/process.c
     [java]   CC        ../../../core/sys/procinit.c
     [java]   CC        ../../../core/sys/autostart.c
     [java]   CC        ../../../core/loader/elfloader.c
     [java]   CC        ../../../core/sys/profile.c
     [java]   CC        ../../../core/sys/timetable.c
     [java]   CC        ../../../core/sys/timetable-aggregate.c
     [java]   CC        ../../../core/sys/compower.c
     [java]   CC        ../../../core/dev/serial-line.c
     [java]   CC        ../../../core/lib/memb.c
     [java]   CC        ../../../core/lib/mmem.c
     [java]   CC        ../../../core/sys/timer.c
     [java]   CC        ../../../core/lib/list.c
     [java]   CC        ../../../core/sys/etimer.c
     [java]   CC        ../../../core/sys/ctimer.c
     [java]   CC        ../../../core/sys/energest.c
     [java]   CC        ../../../core/sys/rtimer.c
     [java]   CC        ../../../core/sys/stimer.c
     [java]   CC        ../../../core/lib/trickle-timer.c
     [java]   CC        ../../../core/lib/print-stats.c
     [java]   CC        ../../../core/lib/ifft.c
     [java]   CC        ../../../core/lib/crc16.c
     [java]   CC        ../../../core/lib/random.c
     [java]   CC        ../../../core/lib/checkpoint.c
     [java]   CC        ../../../core/lib/ringbuf.c
     [java]   CC        ../../../core/lib/settings.c
     [java]   CC        ../../../core/net/dhcpc.c
     [java]   CC        ../../../core/net/hc.c
     [java]   CC        ../../../core/net/nbr-table.c
     [java]   CC        ../../../core/net/netstack.c
     [java]   CC        ../../../core/net/packetbuf.c
     [java]   CC        ../../../core/net/packetqueue.c
     [java]   CC        ../../../core/net/psock.c
     [java]   CC        ../../../core/net/queuebuf.c
     [java]   CC        ../../../core/net/resolv.c
     [java] ../../../core/net/resolv.c: In function ‘mdns_write_announce_records’:
     [java] ../../../core/net/resolv.c:509:22: warning: variable ‘ans’ set but not used [-Wunused-but-set-variable]
     [java] ../../../core/net/resolv.c: In function ‘mdns_prep_host_announce_packet’:
     [java] ../../../core/net/resolv.c:606:22: warning: unused variable ‘ans’ [-Wunused-variable]
     [java]   CC        ../../../core/net/sicslowpan.c
     [java]   CC        ../../../core/net/simple-udp.c
     [java]   CC        ../../../core/net/tcpdump.c
     [java]   CC        ../../../core/net/tcpip.c
     [java]   CC        ../../../core/net/uaodv-rt.c
     [java] ../../../core/net/uaodv-rt.c: In function ‘uaodv_rt_lookup_any’:
     [java] ../../../core/net/uaodv-rt.c:98:5: warning: implicit declaration of function ‘memcmp’ [-Wimplicit-function-declaration]
     [java]   CC        ../../../core/net/uaodv.c
     [java] ../../../core/net/uaodv.c: In function ‘fwc_lookup’:
     [java] ../../../core/net/uaodv.c:91:3: warning: implicit declaration of function ‘memcmp’ [-Wimplicit-function-declaration]
     [java]   CC        ../../../core/net/uip-debug.c
     [java]   CC        ../../../core/net/uip-ds6-route.c
     [java]   CC        ../../../core/net/uip-ds6-nbr.c
     [java]   CC        ../../../core/net/uip-ds6.c
     [java]   CC        ../../../core/net/uip-fw-drv.c
     [java]   CC        ../../../core/net/uip-fw.c
     [java]   CC        ../../../core/net/uip-icmp6.c
     [java]   CC        ../../../core/net/uip-nd6.c
     [java]   CC        ../../../core/net/uip-neighbor.c
     [java]   CC        ../../../core/net/uip-over-mesh.c
     [java]   CC        ../../../core/net/uip-packetqueue.c
     [java]   CC        ../../../core/net/uip-split.c
     [java]   CC        ../../../core/net/uip-udp-packet.c
     [java]   CC        ../../../core/net/uip.c
     [java]   CC        ../../../core/net/uip6.c
     [java]   CC        ../../../core/net/uip_arp.c
     [java]   CC        ../../../core/net/uiplib.c
     [java]   CC        ../../../core/sys/mt.c
     [java]   CC        ../../../core/dev/nullradio.c
     [java]   CC        ../../../apps/powertrace/powertrace.c
     [java]   CC        ../../../apps/collect-view/collect-view.c
     [java]   CC        ../../../apps/collect-view/collect-view-sky.c
     [java]   CC        ../../../platform/sky/./contiki-sky-platform.c
     [java]   CC        ../../../core/dev/sht11.c
     [java] ../../../core/dev/sht11.c: In function ‘sht11_init’:
     [java] ../../../core/dev/sht11.c:218:4: warning: #warning SHT11: DISABLING I2C BUS [-Wcpp]
     [java]   CC        ../../../core/dev/sht11-sensor.c
     [java]   CC        ../../../platform/sky/dev/light-sensor.c
     [java]   CC        ../../../platform/sky/dev/battery-sensor.c
     [java]   CC        ../../../platform/sky/dev/button-sensor.c
     [java]   CC        ../../../platform/sky/dev/radio-sensor.c
     [java]   CC        ../../../cpu/msp430/f1xxx/spi.c
     [java]   CC        ../../../core/dev/ds2411.c
     [java]   CC        ../../../platform/sky/dev/xmem.c
     [java]   CC        ../../../platform/sky/dev/i2c.c
     [java]   CC        ../../../platform/sky/./node-id.c
     [java]   CC        ../../../core/lib/sensors.c
     [java]   CC        ../../../core/cfs/cfs-coffee.c
     [java]   CC        ../../../core/dev/cc2420.c
     [java] ../../../core/dev/cc2420.c: In function ‘flushrx’:
     [java] ../../../core/dev/cc2420.c:188:11: warning: variable ‘dummy’ set but not used [-Wunused-but-set-variable]
     [java] ../../../core/dev/cc2420.c: In function ‘cc2420_transmit’:
     [java] ../../../core/dev/cc2420.c:356:11: warning: variable ‘total_len’ set but not used [-Wunused-but-set-variable]
     [java]   CC        ../../../core/dev/cc2420-aes.c
     [java]   CC        ../../../cpu/msp430/./cc2420-arch.c
     [java]   CC        ../../../cpu/msp430/./cc2420-arch-sfd.c
     [java] ../../../cpu/msp430/./cc2420-arch-sfd.c: In function ‘cc2420_timerb1_interrupt’:
     [java] ../../../cpu/msp430/./cc2420-arch-sfd.c:44:7: warning: variable ‘tbiv’ set but not used [-Wunused-but-set-variable]
     [java]   CC        ../../../platform/sky/dev/sky-sensors.c
     [java]   CC        ../../../cpu/msp430/./uip-ipchksum.c
     [java]   CC        ../../../platform/sky/./checkpoint-arch.c
     [java]   CC        ../../../cpu/msp430/f1xxx/uart1.c
     [java]   CC        ../../../cpu/msp430/./slip_uart1.c
     [java]   CC        ../../../cpu/msp430/dev/uart1-putchar.c
     [java]   CC        ../../../core/lib/me.c
     [java]   CC        ../../../core/lib/me_tabs.c
     [java]   CC        ../../../core/dev/slip.c
     [java]   CC        ../../../cpu/msp430/f1xxx/msp430.c
     [java]   CC        ../../../cpu/msp430/./flash.c
     [java]   CC        ../../../cpu/msp430/f1xxx/clock.c
     [java]   CC        ../../../core/dev/leds.c
     [java]   CC        ../../../cpu/msp430/./leds-arch.c
     [java]   CC        ../../../cpu/msp430/./watchdog.c
     [java]   CC        ../../../cpu/msp430/./lpm.c
     [java]   CC        ../../../cpu/msp430/./mtarch.c
     [java]   CC        ../../../cpu/msp430/f1xxx/rtimer-arch.c
     [java]   CC        ../../../core/loader/elfloader-msp430.c
     [java]   CC        ../../../core/loader/symtab.c
     [java]   CC        symbols.c
     [java]   AR        contiki-sky.a
     [java]   CC        udp-sender.c
     [java] udp-sender.c: In function ‘collect_common_send’:
     [java] udp-sender.c:140:9: warning: passing argument 1 of ‘rpl_get_parent_rank’ from incompatible pointer type [enabled by default]
     [java] In file included from udp-sender.c:34:0:
     [java] ../../../core/./net/rpl/rpl.h:247:12: note: expected ‘struct uip_lladdr_t *’ but argument is of type ‘union rimeaddr_t *’
     [java]   CC        collect-common.c
     [java]   CC        ../../../platform/sky/./contiki-sky-main.c
     [java]   LD        udp-sender.sky
     [java] rm udp-sender.co obj_sky/collect-common.o obj_sky/contiki-sky-main.o
     [java] 
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

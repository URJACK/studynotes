# RPL的实现与Cooja仿真

## RPL简介

以下排序，依据作用范围的从大到小。

`NETWORK -> RPL Instance -> RPL DAG -> RPL Node`

## RPL使用到的数据宏定义

### MAC_TX

```c
/* Generic MAC return values. */
enum {
  /**< The MAC layer transmission was OK.  MAC层传输正常。*/
  MAC_TX_OK,

  /**< The MAC layer transmission could not be performed due to a
     collision. 由于冲突，无法执行MAC层传输。*/
  MAC_TX_COLLISION,

  /**< The MAC layer did not get an acknowledgement for the packet. 
  MAC层未收到该数据包的确认。 */
  MAC_TX_NOACK,

  /**< The MAC layer deferred the transmission for a later time.
  MAC层将传输推迟了一段时间。*/
  MAC_TX_DEFERRED,

  /**< The MAC layer transmission could not be performed because of an
     error. The upper layer may try again later. 
     由于发生错误，无法执行MAC层传输。 上层可以稍后再试。*/
  MAC_TX_ERR,

  /**< The MAC layer transmission could not be performed because of a
     fatal error. The upper layer does not need to try again, as the
     error will be fatal then as well.
     由于出现致命错误，无法执行MAC层传输。 上层不需要重试，因为错误也会是致命的。*/
  MAC_TX_ERR_FATAL,
};
```

###  uip_ipaddr_t

该数据结构是一个4字节或者16字节的数据对象。刚好对应Ipv4或者Ipv6的地址存储大小。

```c
typedef union uip_ip4addr_t {
  uint8_t  u8[4];			/* Initializer, must come first. */
  uint16_t u16[2];
} uip_ip4addr_t;

typedef union uip_ip6addr_t {
  uint8_t  u8[16];			/* Initializer, must come first. */
  uint16_t u16[8];
} uip_ip6addr_t;

#if UIP_CONF_IPV6
typedef uip_ip6addr_t uip_ipaddr_t;
#else /* UIP_CONF_IPV6 */
typedef uip_ip4addr_t uip_ipaddr_t;
#endif /* UIP_CONF_IPV6 */
```

### rpl_rank_t

就是一个2字节的整数

```c
typedef uint16_t rpl_rank_t;
```

## RPL使用到的数据结构（PART - A）

### rpl_instance

```c
/* Instance */
struct rpl_instance {
  /* DAG configuration */
  rpl_metric_container_t mc;
  rpl_of_t *of;
  rpl_dag_t *current_dag;
  rpl_dag_t dag_table[RPL_MAX_DAG_PER_INSTANCE];
  /* The current default router - used for routing "upwards" */
  uip_ds6_defrt_t *def_route;
  uint8_t instance_id;
  uint8_t used;
  uint8_t dtsn_out;
  uint8_t mop;
  uint8_t dio_intdoubl;
  uint8_t dio_intmin;
  uint8_t dio_redundancy;
  uint8_t default_lifetime;
  uint8_t dio_intcurrent;
  uint8_t dio_send; /* for keeping track of which mode the timer is in */
  uint8_t dio_counter;
  rpl_rank_t max_rankinc;
  rpl_rank_t min_hoprankinc;
  uint16_t lifetime_unit; /* lifetime in seconds = l_u * d_l */
#if RPL_CONF_STATS
  uint16_t dio_totint;
  uint16_t dio_totsend;
  uint16_t dio_totrecv;
#endif /* RPL_CONF_STATS */
  clock_time_t dio_next_delay; /* delay for completion of dio interval */
  struct ctimer dio_timer;
  struct ctimer dao_timer;
};
```

### rpl_dag

```c
/* Directed Acyclic Graph */
struct rpl_dag {
  uip_ipaddr_t dag_id;
  rpl_rank_t min_rank; /* should be reset per DAG iteration! */
  uint8_t version;
  uint8_t grounded;
  uint8_t preference;
  uint8_t used;
  /* live data for the DAG */
  uint8_t joined;
  rpl_parent_t *preferred_parent;
  rpl_rank_t rank;
  struct rpl_instance *instance;
  LIST_STRUCT(parents);
  rpl_prefix_t prefix_info;
};
typedef struct rpl_dag rpl_dag_t;
```

### rpl_parent

```c
struct rpl_parent {
  struct rpl_parent *next;
  struct rpl_dag *dag;
#if RPL_DAG_MC != RPL_DAG_MC_NONE
  rpl_metric_container_t mc;
#endif /* RPL_DAG_MC != RPL_DAG_MC_NONE */
  rpl_rank_t rank;
  uint16_t link_metric;
  uint8_t dtsn;
  uint8_t updated;
};
typedef struct rpl_parent rpl_parent_t;
```

### 

## rpl-of0.c

### calculate_rank

该函数可以用来计算结点的rank值。

```c
static rpl_rank_t calculate_rank(rpl_parent_t *, rpl_rank_t);
```

传入当前节点的父节点 `p` 以及 `base_rank` ，我们可以得到当前节点 `c` 的 `rank值` 。使用场景如图：

<img src=".\imgs_5\image-20210308093837332.png" alt="image-20210308093837332"  />

```c
static rpl_rank_t
calculate_rank(rpl_parent_t *p, rpl_rank_t base_rank)
{
  rpl_rank_t increment;
  //如果base_rank没有被指定 那么我们会以p的rank值作为我们的base_rank
  if(base_rank == 0) {
    //如果传入的p是一个空结点 那么直接返回无穷RANK 
    //#define INFINITE_RANK  0xffff
    if(p == NULL) {
      return INFINITE_RANK;
    }
    base_rank = p->rank;
  }
  
  //如果p不是空结点 那么返回结点p所属instance中记录的“最小跳跃增量”
  //当然如果p本身就是空结点 那么直接返回一个“默认的增量值” 为 256
  //#define DEFAULT_RANK_INCREMENT RPL_MIN_HOPRANKINC
  //#define RPL_MIN_HOPRANKINC 256
  increment = p != NULL ?
                p->dag->instance->min_hoprankinc :
                DEFAULT_RANK_INCREMENT;
  
  //处理一下 base_rank + increment 溢出后的情况
  if((rpl_rank_t)(base_rank + increment) < base_rank) {
    PRINTF("RPL: OF0 rank %d incremented to infinite rank due to wrapping\n",
        base_rank);
    return INFINITE_RANK;
  }
  return base_rank + increment;
}
```

### best_dag

这个函数用来比较两个DAG，求出最好的DAG

```c
static rpl_dag_t *
best_dag(rpl_dag_t *d1, rpl_dag_t *d2)
{
  //uint8_t grounded;
  //如果d1 和 d2 的grounded 满足：
  //d1->0 d2->1 或者 d1->1 d2->0 的情况
  //那么我们返回grounded为1的dag即可。
  if(d1->grounded) {
    if (!d2->grounded) {
      return d1;
    }
  } else if(d2->grounded) {
    return d2;
  }

  //当两个grounded相同的时候（全为0，或全为1）
  //此时我们需要观察的是preference的数值情况
  //我们返还 “preference 更大” 的 DAG
  if(d1->preference < d2->preference) {
    return d2;
  } else {
    if(d1->preference > d2->preference) {
      return d1;
    }
  }
  
  //当preference 与 grounded 都相同的时候
  //此时我们返回 “rank 更小” 的 DAG
  //当然如果最后rank依然相等，我们返回d1
  if(d2->rank < d1->rank) {
    return d2;
  } else {
    return d1;
  }
}
```

### best_parent

求出最好的 父对象 。（看哪个父对象的rank值更低）

这里两个 父对象 必须属于相同的DAG。

```c
static rpl_parent_t *
best_parent(rpl_parent_t *p1, rpl_parent_t *p2)
{
  rpl_rank_t r1, r2;
  rpl_dag_t *dag;
  
  PRINTF("RPL: Comparing parent ");
  PRINT6ADDR(rpl_get_parent_ipaddr(p1));
  PRINTF(" (confidence %d, rank %d) with parent ",
        p1->link_metric, p1->rank);
  PRINT6ADDR(rpl_get_parent_ipaddr(p2));
  PRINTF(" (confidence %d, rank %d)\n",
        p2->link_metric, p2->rank);

  //#define DAG_RANK(fixpt_rank, instance) ((fixpt_rank) / (instance)->min_hoprankinc)
  //#define RPL_MIN_HOPRANKINC 256
  //注意使用DAG_RANK时，r1与r2都是使用的p1的RPL Instance
  r1 = DAG_RANK(p1->rank, p1->dag->instance) * RPL_MIN_HOPRANKINC  +
         p1->link_metric;
  r2 = DAG_RANK(p2->rank, p1->dag->instance) * RPL_MIN_HOPRANKINC  +
         p2->link_metric;
  /* Compare two parents by looking both and their rank and at the ETX
     for that parent. We choose the parent that has the most
     favourable combination. */
    
  dag = (rpl_dag_t *)p1->dag; /* Both parents must be in the same DAG. */
  //当 | r1 - r2 | < MIN_DIFFERENCE 
  //#define MIN_DIFFERENCE (RPL_MIN_HOPRANKINC + RPL_MIN_HOPRANKINC / 2)
  //#define RPL_MIN_HOPRANKINC 256
  if(r1 < r2 + MIN_DIFFERENCE &&
     r1 > r2 - MIN_DIFFERENCE) {
    //这个条件满足的时候 也就意味着r1 与 r2比较相近的时候 返回p1 的 DAG更青睐的 父节点。
    /*
    这里我认为是有疑问的 因为乍一看 dag->preferred_parent 实际上与 p1 p2 都没有关系
    为啥反而二者的差值比较小的时候，反而最终得出的 parent 是 DAG的preferred_parent ？
    */
    return dag->preferred_parent;
  } else if(r1 < r2) {
    //如果r1 与 r2 差别大的话 我们返回 rank更小的结点
    return p1;
  } else {
    return p2;
  }
}
```

## rpl-mhrof.c

在本文中的一些typedef定义如下

转定义2字节对象 `typedef uint16_t rpl_path_metric_t;`

### calculate_path_metric

返回“父对象”的链路计量（通常就是rank + link_metric）

```c
static rpl_path_metric_t
calculate_path_metric(rpl_parent_t *p)
{
  if(p == NULL) {
    //如果p本身是空结点 那么返回默认的path计量
    //#define MAX_PATH_COST 100
    //#define RPL_DAG_MC_ETX_DIVISOR 128
    return MAX_PATH_COST * RPL_DAG_MC_ETX_DIVISOR;
  }

//link_metric 链路计量 加上 rank 值。
#if RPL_DAG_MC == RPL_DAG_MC_NONE
  return p->rank + (uint16_t)p->link_metric;
#elif RPL_DAG_MC == RPL_DAG_MC_ETX
  return p->mc.obj.etx + (uint16_t)p->link_metric;
#elif RPL_DAG_MC == RPL_DAG_MC_ENERGY
  return p->mc.obj.energy.energy_est + (uint16_t)p->link_metric;
#else
#error "Unsupported RPL_DAG_MC configured. See rpl.h."
#endif /* RPL_DAG_MC */
}
```

### neighbor_link_callback

传入的 **父对象**`p` 的 `link_metric` 指标会因为传入 `status` 和 `numtx` 的取值而改变.

```c
static void
neighbor_link_callback(rpl_parent_t *p, int status, int numtx)
{
  uint16_t recorded_etx = p->link_metric;
  //#define RPL_DAG_MC_ETX_DIVISOR 128
  uint16_t packet_etx = numtx * RPL_DAG_MC_ETX_DIVISOR;
  uint16_t new_etx;

  /* Do not penalize the ETX when collisions or transmission errors occur. */
  if(status == MAC_TX_OK || status == MAC_TX_NOACK) {
    if(status == MAC_TX_NOACK) {
      //如果MAC层未收到该数据包的确认 那么packet_etx 会变为最大的 etx
      //#define MAX_LINK_METRIC 10
      packet_etx = MAX_LINK_METRIC * RPL_DAG_MC_ETX_DIVISOR;
    }
    //new_etx = f( recorded_etx , packet_etx )
    new_etx = ((uint32_t)recorded_etx * ETX_ALPHA +
               (uint32_t)packet_etx * (ETX_SCALE - ETX_ALPHA)) / ETX_SCALE;

    PRINTF("RPL: ETX changed from %u to %u (packet ETX = %u)\n",
        (unsigned)(recorded_etx / RPL_DAG_MC_ETX_DIVISOR),
        (unsigned)(new_etx  / RPL_DAG_MC_ETX_DIVISOR),
        (unsigned)(packet_etx / RPL_DAG_MC_ETX_DIVISOR));
    p->link_metric = new_etx;
  }
}
```

### calculate_rank

计算 **父对象** `p` 的新rank值 `new_rank = f(base_rank, p)`  

该句是核心逻辑 `new_rank = p->link_metric + base_rank;` 

```c
static rpl_rank_t
calculate_rank(rpl_parent_t *p, rpl_rank_t base_rank)
{
  rpl_rank_t new_rank;
  rpl_rank_t rank_increase;

  if(p == NULL) {
    //如果当前的p是一个空结点
    if(base_rank == 0) {
      //如果没有指定base_rank 此时直接返回 无穷rank
      return INFINITE_RANK;
    }
    //如果是空结点 但给定了 base_rank 那么我们给 rank_increase 一个默认值 让其可以完成计算
    rank_increase = RPL_INIT_LINK_METRIC * RPL_DAG_MC_ETX_DIVISOR;
  } else {
    //如果是p是给定的 那么 增量 = f(p->link_metric)
    rank_increase = p->link_metric;
    if(base_rank == 0) {
      //当然 如果当前的基础 base_rank 没有给定， 给出了p结点 那么 base_rank = f(p->rank)
      base_rank = p->rank;
    }
  }

  if(INFINITE_RANK - base_rank < rank_increase) {
    /* Reached the maximum rank. */
    new_rank = INFINITE_RANK;
  } else {
   /* Calculate the rank based on the new rank information from DIO or
      stored otherwise. */
    new_rank = base_rank + rank_increase;
  }

  return new_rank;
}
```

### best_parent

```c
static rpl_parent_t *
best_parent(rpl_parent_t *p1, rpl_parent_t *p2)
{
  rpl_dag_t *dag;
  //typedef uint16_t rpl_path_metric_t;
  rpl_path_metric_t min_diff;
  rpl_path_metric_t p1_metric;
  rpl_path_metric_t p2_metric;

  dag = p1->dag; /* Both parents are in the same DAG. */

  min_diff = RPL_DAG_MC_ETX_DIVISOR /
             PARENT_SWITCH_THRESHOLD_DIV;

  p1_metric = calculate_path_metric(p1);
  p2_metric = calculate_path_metric(p2);

  /* Maintain stability of the preferred parent in case of similar ranks. */
  /* 只要二者有一个，已经是DAG倾向的父对象了 ， 
     那么此时检查二者的差异， 如果仅有较小差异就不改变 */
  if(p1 == dag->preferred_parent || p2 == dag->preferred_parent) {
    if(p1_metric < p2_metric + min_diff &&
       p1_metric > p2_metric - min_diff) {
      PRINTF("RPL: MRHOF hysteresis: %u <= %u <= %u\n",
             p2_metric - min_diff,
             p1_metric,
             p2_metric + min_diff);
      return dag->preferred_parent;
    }
  }
  
  //如果二者的差异并不算小， 那么返回metric更小的 父对象
  return p1_metric < p2_metric ? p1 : p2;
}
```

## 自定义rpl文件

上图显示了可以使用的示例网络拓扑。

要定义一个全新的目标功能文件（不修改现有文件），必须在其中定义以下功能。（同样，应该相应地修改makefile，并应注意不要将新文件运行到编译和链接错误中。）

### RPL的接口

 一些RPL API函数包括：

- reset(dag): Resets the objective function state for a specific DAG. This function is called when doing a global repair on the DAG. 

  重置特定DAG的目标功能状态。 在DAG上进行全局修复时，将调用此函数。

- neighbor_link_callback(parent, status, etx): Receives link-layer neighbor information.

  接收链路层邻居信息。

- best_parent(parent1, parent2): Compares two parents and returns the best one, according to the OF.

  根据OF，比较两个父母并返回最佳父母。

- best_dag(dag1, dag2): Compares two DAGs and returns the best one, according to the OF.

  根据OF，比较两个DAG并返回最佳DAG。

- calculate_rank(parent, base_rank): Calculates a rank value using the parent rank and a base rank.

  使用父等级和基本等级计算等级值。

- update_metric_container(dag): Updates the metric container for outgoing DIOs in a certain DAG. If the objective function of the DAG does not use metric containers, the function should set the object type to RPL_DAG_MC_NONE.

  更新特定DAG中传出DIO的度量标准容器。 如果DAG的目标函数不使用度量标准容器，则该函数应将对象类型设置为RPL_DAG_MC_NONE。

### RPL的上层引用关系

在 `/core/net/rpl/rpl.h` 里有一个结构体定义 rpl_of_t：

```c
struct rpl_of {
  void (*reset)(struct rpl_dag *);
  void (*neighbor_link_callback)(rpl_parent_t *, int, int);
  rpl_parent_t *(*best_parent)(rpl_parent_t *, rpl_parent_t *);
  rpl_dag_t *(*best_dag)(rpl_dag_t *, rpl_dag_t *);
  rpl_rank_t (*calculate_rank)(rpl_parent_t *, rpl_rank_t);
  void (*update_metric_container)( rpl_instance_t *);
  rpl_ocp_t ocp;
};
typedef struct rpl_of rpl_of_t;
```

在 `/core/net/rpl/rpl-mrhof.c` 中，则是对这些函数的具体的实现

```c
rpl_of_t rpl_mrhof = {
  reset,
  neighbor_link_callback,
  best_parent,
  best_dag,
  calculate_rank,
  update_metric_container,
  1
};
```

二者的依赖关系如下:

`rpl-mrhof.c -> rpl-private.h -> rpl.h` 

顺带一提：在

```c
#include "net/rpl/rpl-private.h"

#define DEBUG DEBUG_NONE
#include "net/uip-debug.h"
```

其中"net/uip-debug.h"主要是提供PRINTF这些打印函数。
也就说核心的引用关系其实就是上文提到的rph.h。



## 仿真的步骤

### Run Cooja

首先前往 Contiki 文件夹(contiki-2.7) 

然后前往路径 /tools/cooja 

运行命令sudo ant run来打开一个cooja GUI。

```
$ cd contiki-2.7/tools/cooja
$ sudo ant run
```

### Start a New Simulation

从“文件”下拉菜单中选择一个“新模拟”（或者可以执行Ctrl + N）。

将会弹出以下屏幕。

<img src=".\imgs_5\image-20210308151846553.png" alt="image-20210308151846553" style="zoom: 67%;" />

在 “Simulation name” 字段中为您的 "Simulation" 命名。 从高级设置下的无线电介质的下拉菜单中选择有向图无线电介质（DGRM）。 单击创建按钮。

如图所示，创建的 Simulation 将打开多个窗口。

<img src=".\imgs_5\image-20210308152004976.png" alt="image-20210308152004976" style="zoom:67%;" />

### Add Sink Mote

添加“接收器 `Sink` ”类型的微粒 `Mote` 。 这里使用的是来自rpl-collect示例（`/examples/ipv6/rpl-collect`）的udp-sink代码。

但是，您可以根据您的应用程序上载要实现的任何代码。 单击Compile按钮。 

成功编译后将出现一个Create按钮，该按钮将根据需要在网络中添加节点数量。 这里仅使用一个水槽。

<img src=".\imgs_5\image-20210308152201172.png" alt="image-20210308152201172" style="zoom:67%;" />

### Add Sender Motes

添加“接收器”类型的微粒。 这里使用了来自rpl-collect示例（ `/examples/ipv6/rpl-collect/` ）的udp-sender代码。 但是，您可以根据您的应用程序上载要实现的任何代码。

编译代码并根据拓扑创建许多这种类型的节点。

注意：此处的微粒位置无关紧要。 您可以将微粒放置在图形中的任何位置。 由于这与距离模型不同，因此我们在微粒之间进行显式链接以进行通信，因此它们之间的距离没有区别。

### Add Communication Links

在每组节点之间添加两个通信链接，以便通信可以是双向的。
单击工具-> DGRM链接...，这将打开DGRM配置器对话框。 

点击添加。 选择源和目标节点，然后再次单击添加。 

这将添加从源节点到目标节点的单向链接。 对于双向链接，您需要再添加一个具有切换的源节点和目标节点的链接。 您可以通过这种方式添加多个链接。 添加链接后，关闭对话框。

<img src=".\imgs_5\image-20210308152443408.png" alt="image-20210308152443408" style="zoom:67%;" />

您可以根据应用更改链接的其他参数，例如RX比率，RSSI，LQI和延迟。 这些参数影响单个链路的质量，例如。 接收比率会影响ETX值。

因此，为了在各种链路质量条件下测试您的应用，可以更改这些参数。
您也可以使用remove选项删除现有的一个。 导入选项有助于导入任何已经在其中指定了这些链接连接和参数的数据文件。

### Run Simulation

使用 "Simulation Control" 窗口中的 "Start" 选项运行模拟。 

这将启动Mote并为所有Motes分配一个新的 "Rime地址" 和其他初始化过程。

### Watch Output

Motes输出和调试消息可以在Motes Output窗口中看到。

您可以根据节点ID：node_id过滤输出以监视特定节点。您还可以通过过滤来查看特定的调试消息。 

Motes输出的其他有用功能是文件，编辑和查看：

1. “文件”选项有助于将输出保存到文件中。 
2. 编辑具有复制输出的选项-完整消息或选定的特定消息。 

您也可以使用“清除所有消息”选项清除消息。

您可以根据实验目的使用保存在文件中的这些消息进行观察并绘制图表。
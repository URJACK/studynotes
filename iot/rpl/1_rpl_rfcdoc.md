# RFC6550

RPL的包信息作用：
`数据包与RPL实例相关联` ， `验证RPL路由状态` 。

RPL不依赖特定链路层技术的任何特定功能。

## 术语

**DAG** ： **有向无环图** 。 有向图，具有以下特征：所有边均以不存在环的方式定向。所有边缘都包含在朝向并终止于一个或多个根节点的路径中。

**DAG root** ： **DAG根节点** ，是DAG中没有输出边缘的节点。 根据定义，由于该图是非循环的，因此所有<u>DAG必须具有至少一个DAG根</u>，并且所有 **路径** 都终止于DAG根。

`注意这里的是DAG必须至少有一个DAG root，这里意味着，DAG可以有多个DAG root` 

`注意这里所说的路径，我个人认为路径是节点之间认作父节点的这种依赖关系。所有的路径终止于根，意味着所有的节点的最终的‘父节点’都只能是根节点` 

**Destination-Oriented DAG (DODAG)** ： DAG植根于单个目标，我个人认为是<u>仅有一个根节点（DAG root）的DAG</u>，就可以称为**DODAG**。

`DODAG的原文解释：A DAG rooted at a single destination, i.e., at a single DAG root (the DODAG root) with no outgoing edges. `

**DODAG root** ：DODAG root 是 DODAG的DAG root。 DODAG根可以充当DODAG的边界路由器。 特别地，它可以在DODAG中聚合路由，并且可以将DODAG路由重新分布到其他路由协议中。 

<u>Virtual DODAG root</u> ：虚拟DODAG根是两个或多个RPL路由器（例如6LoWPAN边界路由器（6LBR））的结果，它们协调同步DODAG状态并协调行动，就好像它们是单个DODAG根（具有多个接口） LLN。 协调最有可能发生在可靠的传输链路上的受电设备之间，并且该方案的详细信息超出了本规范的范围（将在以后的伴随规范中进行定义）。 

**Up** ：上指从叶节点到DODAG根的方向，与DODAG边缘方向相同。

**Down** ： 向下是指从DODAG根到叶节点的方向，与DODAG边缘方向相反。 

`这遵循在图形和深度优先搜索中使用的常用术语：其中，离根更远的顶点“更深”或“向下”，而离根更近的顶点“更浅”或“向上”。`

**Rank** ： 节点的Rank定义了该节点相对于DODAG根相对于其他节点的独立位置。
<u>排名严格沿向下方向增加，而严格沿向上方向减少。</u> 
排名的确切计算方式取决于DAG的目标函数（OF）。
等级可以类似地跟踪简单的拓扑距离，可以作为链接度量的函数进行计算，并且可以考虑其他属性（例如约束）。

**Expected Transmission Count（ETX）** ：预期的传输计数，一种在LLN中使用并在[RFC6551]中定义的相当通用的路由指标。  

**Objective Function (OF)** ： OF定义了如何使用路由指标，优化目标和相关功能来计算排Rank。 此外，OF决定了如何选择DODAG中的父节点，从而决定了DODAG的形成。

**Objective Code Point (OCP)** ： OCP是指示DODAG使用哪个目标功能的标识符。

<u>RPLInstanceID</u> ： RPLInstanceID是网络内的唯一标识符。 具有相同RPLInstanceID的DODAG共享相同的目标功能。

**RPL Instance** ： RPL实例是一组共享一个<u>RPLInstanceID</u>的一个或多个DODAG。 一个RPL节点最多可以属于一个RPL实例中的一个DODAG。 每个RPL实例都独立于其他RPL实例运行。 本文档描述了单个RPL实例中的操作。

**DODAGID** ： DODAGID 是 **DODAG root** 的标识符。 DODAGID在LLN中的RPL实例的范围内是唯一的。 元组（RPLInstanceID，DODAGID）唯一标识一个DODAG。

`显然，不同的RPLInstance里的DODAGID可以是相同的，而不必考虑ID冲突。`

<u>DODAG Version</u> ：DODAG版本是具有给定DODAGID的DODAG的特定迭代（“版本”）。 

**DODAGVersionNumber** ： DODAGVersionNumber是一个顺序计数器，由根递增以形成DODAG的新版本。
<u>DODAG Version</u> 由<u>（RPLInstanceID，DODAGID，DODAGVersionNumber）元组</u>唯一标识。

<u>Goal</u> ： 该目标是在RPL范围之外定义的特定于应用程序的目标。 植根DODAG的任何节点都需要了解此目标，以决定是否可以实现该目标。 典型的目标是根据特定的目标函数构造DODAG，并保持与一组主机的连接性（例如，使用最小化指标并连接到特定数据库主机的目标函数来存储收集的数据）。

<u>Grounded</u> ： 当DODAG根目录可以满足目标时，DODAG便会“落地”。

<u>Floating</u> ： 如果DODAG没有接地，则它是浮动的。 浮动DODAG不应具有满足目标所需的属性。 但是，它可以提供与DODAG内其他节点的连接。

**DODAG parent** ： DODAG内节点的父级，是指向DODAG根的路径上该节点的直接后继节点之一。 DODAG父级节点的Rank低于该节点的Rank。 （请参阅第3.5.1节）。

<u>Sub-DODAG</u> ： 节点的子DODAG是其他节点的集合，其其他节点的DODAG根路径通过该节点。 节点的子DODAG中的节点的等级高于该节点的等级。 （请参阅第3.5.1节）。

<u>Local DODAG</u> ： 本地DODAG包含一个且只有一个根节点，它们允许该单个根节点分配和管理由本地RPLInstanceID标识的RPL实例，而无需与其他节点进行协调。 通常，这样做是为了优化到LLN内目的地的路由。 （请参阅第5节）。

`Local DODAG的节点可以直接管理RPL实例，这样看来似乎简化了网络。`

<u>Global DODAG</u> ： 全局DODAG使用可以在其他几个节点之间进行协调的全局RPLInstanceID。 （请参阅第5节）。

**DIO** ：DODAG Information Object (see Section 6.3)

**DAO** ： Destination Advertisement Object (see Section 6.4)

**DIS** ： DODAG Information Solicitation (see Section 6.2)

**CC** ： Consistency Check (see Section 6.6)

与传统的IP网络相比，LLN设备在形成网络时经常混合使用主机和路由器的角色。 在本文档中：
“主机”是指可以生成但不能转发RPL流量的LLN设备。 
“路由器”是指可以转发并生成RPL流量的LLN设备；
 “节点”是指任何RPL设备，可以是主机，也可以是路由器。 

## 笔记

### 协议概览

#### 上行路线与DODAG构建

如何修正Data-Path？
例如，如果一个节点接收到一个标记为向上移动的数据包，并且该数据包记录的发送者的Rank低于（小于）接收者的Rank，则接收者可以得出结论：该数据包向上方向没有进展，并且DODAG不一致。

Distributed Algorithm Operation？
	1·使用一些相关的DODAG配置，可以让一些节点设定成为DODAG root。
	2·每个DODAG 节点如何告知其他节点，自己的“routing cost”，“affiliation with a which DODAG？”这些信息的呢？答案就是通过**本地链路组播（link-local multicast）的DIO**。
	3·节点会去监听DIO的存在。无论是节点们想要加入一个DODAG的时候，又或者是它们想要保持现有的DODAG结构的时候。都会使用到它们**自身的信息**（routing cost 之类的），并根据指定的**OF**和**邻居的Rank**。
	4·DODAG中，节点可以根据它们和父节点的“DODAG Version”来提供路由表项。至于每次数据包的目的地，是通过DIO信息来指定的。

#### 下行路线与目的地

如何建立下行链路？
RPL利用DAO建立下行链路。对于一对多、一对一流量的应用来说，DAO消息只是一个可选的特征。

RPL如何处理下行链路？
RPL对下行链路只有两种处理模式：1·Storing； 2·Non-Storing； （详情见文档section 9）

RPL中P2P的数据包如何传输？
1·数据包先向沿上搜寻，如果搜寻到了目标，目标就接收这个数据包。
2·如果到根结点，都没有找到目标，此时从根结点开始沿下搜寻。

RPL中，不同的下行链路处理模式：
A的父节点记录： “A -> B -> C -> Root”
E的父节点记录： “E -> B -> C -> Root”
A需要发一个数据包给E。
如果使用“Non-Storing”模式，那么“A->B->C->Root->C->B->E”。
如果使用“Storing”模式，那么“A->B->E”就可行了。
`需要注意的是，两种模式是不可以混用的，如果混用，则规范不在该文档的讨论范围之内`

[^研究点]: Non-Storing 与 Storing 模式混用的研究`

#### Rank

Rank是一个DODAG版本内相对于邻居的相对位置的表达，它不一定能很好地表明或恰当地表达到根的距离或路径成本。

Rank每一跳增加的量是有范围的。

A节点的父节点被添加后，A节点在该DODAG中的rank就会被广播。

Rank是一个混点数，小数点的位置由`MinHopRankIncrease`来决定。`MinHopRankIncrease`由``DODAG root`来决定：

- 对于较大的`MinHopRankIncrease`来说，可能会导致<u>小数点的前移</u>：
  这使得Rank可用的<u>整数部分减少</u>，也自然无法支持较多跳数。
  但同时也使得<u>小数部分增多</u>，这使得一些更细微精确的变化，也可以被察觉到。

- 那么 `MinHopRankIncrease` 与 `rank` 之间具体运算关系如下：
  rank整数部分 = DAGRank(rank) = floor(rank/MinHopRankIncrease)
- eg:`rank = 27; MinHopRankIncrease = 16;` 那么我们可以得出 `rank整数 = 1; rank小数 = 11/16`

#### Route 标准

例如，当目标函数最小化的度量是ETX或时延，或以适合DODAG内使用的目标函数的更复杂的方式计算Rank时，Rank的计算方式可以密切跟踪ETX（预期传输数，LLN中使用的一种相当常见的路由度量，在[RFC6551]中定义）。

节点选择父节点，依据的“指标”和“限制”都是在 **DIO Message** 中“广播”出去的。
依此可见**DIO**是构建DODAG的关键部分。

#### 组网过程

摘录自CSCD的Blog `RPL基础知识点与组网过程` ：

1. 节点复位完成，首先发送DIS包，征集邻居节点信息，这点有点像ARP
2. 邻居点接收到DIS开始发送DIO包。
3. 收到DIO包的节点更新自身邻居表，并选择合适的节点发送数据包。
4. 同时节点会向选中的父节点发送DAO包，告知其是子节点。
5. 父节点更新了自身的路由表后，再向父节点的父节点发DAO，最后到达sink点后双向链路最终形成。有人可能会问，一个点可能会收到很多节点的路由包，它如何选择呢？contiki里面两种仲裁机制：最短路径、etx。在rpl-conf.h第63行可以修改。

RPL包括一个可选的机制来确认DAO消息，这可以减轻单个DAO消息被错过的影响。
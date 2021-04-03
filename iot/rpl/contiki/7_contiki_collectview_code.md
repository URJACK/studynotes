# Collectview 源码解析

## 查找发送信息的触发点

我们从图中，不难发现，当我们对sink类型的节点开启CollectView之后，

![image-20210328142309669](.\imgs_7\image-20210328142309669.png)

collectview会向Mote发送一个指令。

![image-20210328142259041](.\imgs_7\image-20210328142259041.png)

此时每隔一段时间（60s，可以自定义这个时间，在Node Control中），我们都会在Mote output 这个对话框中，看到`ID：1`不断去发送一段意义不明的数字，而每发送一段这个数字，CollectView中的数据就会得到更新。

我们找到对应的`udp-sink.c`的代码，发现它的`collect_common_recv`函数就是起到`Mote->CollectView`的作用。

![image-20210328142745862](.\imgs_7\image-20210328142745862.png)

顺带一提，这个"Mote->CollectView"的过程，并不是通过一个什么其他的“通信函数”，而正是最简单的“打印函数”！
我们修改了其打印的内容（因为我想自己加上一些调试信息），发现<u>它便再也无法正确的向CollectView传输信息了</u>。

## 解析源码

为了探寻这一长串字段，每个字段究竟代表了什么意思，也为了探究，每个字段被提取出来后，又是如何被CollectView类所利用的，我们尝试着去寻找它的源码。

结合之前的搜集到的一些碎片信息：CollectView可以独立于Cooja运行 ---> 这意味着CollectView本身有较大概率是一个工具类，所以我判定，其极有可能是存在于`~/tools`路径下。

果不其然，我们找到了 `~/tools/collect-view` 。

### CollectServer

下面文件众多，我从搜索一个选项卡的名字开始：

![image-20210328143516625](.\imgs_7\image-20210328143516625.png)

我们搜索到了这个文件，`~\tools\collect-view\src\se\sics\contiki\collect\CommandConnection.java`。

在该文件中，有许多图表的显示代码，我们将对它们逐个分析。

#### Instantaneous Power

实时能耗的界面如下：

![image-20210328144838922](.\imgs_7\image-20210328144838922.png)

实时能耗的界面代码如下：

```java
new BarChartPanel(this, POWER, "Instantaneous Power",
                  "Instantaneous Power Consumption", "Nodes", "Power (mW)",
                  new String[] { "LPM", "CPU", "Radio listen", "Radio transmit" }) {
    {
        ValueAxis axis = chart.getCategoryPlot().getRangeAxis();
        ((NumberAxis)axis).setAutoRangeIncludesZero(true);
    }
    protected void addSensorData(SensorData data) {
        Node node = data.getNode();
        String nodeName = node.getName();
        dataset.addValue(data.getLPMPower(), categories[0], nodeName);
        dataset.addValue(data.getCPUPower(), categories[1], nodeName);
        dataset.addValue(data.getListenPower(), categories[2], nodeName);
        dataset.addValue(data.getTransmitPower(), categories[3], nodeName);
    }
},
```

在该段代码中，我们以`BarChartPanel`的传入参数作为本次分析的起点。

我们首先传入了this对象（CollectServer），显然，当`BarChartPanel`自身作为子类，如果有需要调用父类的一些接口参数的时候，这个操作显然是必须的。（可以猜出`BarChartPanel`应该是一个**抽象类**）

第二个传入的参数是POWER，查看其宏定义可以得知。

```java
private static final String POWER = "Power";
```

它是CollectView中的大选项卡名称。
这意味着，这个“Instantaneous Power”的子选项卡，是**属于“Power”这个大选项卡**的（可以猜出，使用了HashMap<string,..>）。

第三个参数，显然是“Instantaneous Power”，这代表了这个**子选项卡的名称**。

第四个参数，"Instantaneous Power Consumption"，这是**标题**的文本。

第五个参数、第六个参数，则分别代表了“横轴名称(Nodes)”、“纵轴名称(Power(mW))”

第七个参数是一个数组对象，同时它的实参名称为categories，
在`protected void addSensorData(SensorData data)`这个函数中。

每一个`SensorData`解析自最开始那段`Mote output`输出的那段字符串。
其中就包括仿真节点的编号（这个**节点编号**并不是界面上展示的1，2，3，4。**节点编号**的一组取值更类似于125，342，234，122这样的不连续的随机数）。

```java
Node node = data.getNode();
String nodeName = node.getName();
```

我们通过节点，获取到**节点名称**，节点名称才是界面上展示的1，2，3，4。

```java
dataset.addValue(data.getLPMPower(), categories[0], nodeName);
```

dataset使用addValue函数：

1. 参数1：数值 （eg：0.412）
2. 参数2：对应的数值名称 （eg："Radio listen"）
3. 参数3：属于哪个节点（使用**节点名称**作为Key）（eg：2.2）

从这里我们也不难发现，这里的`addValue`，因为通过`nodeName`进行**key绑定**，所以如果有新的`sensorData`到来，再次调用`addValue`后，对应的数据就获得了刷新，这也正好契合了“Instantaneous Power”的意义。

#### SensorData

这里讲一下刚刚使用到的`SensorData`类的`data`。

```java
public class SensorData implements SensorInfo {

    private final Node node;
    private final int[] values;
    private final long nodeTime;
    private final long systemTime;
    private int seqno;
    private boolean isDuplicate;
    //...
}
```

`node`：代表这条`sensorData`所属的节点。

`values`：包含了传感器数据、能耗、丢包率等各种统计数据。

`nodeTime`：该字段来自于**原始数据**的**时间戳字段**。该字段的作用是防止数据包被重复的添加。
Node每次调用`addSensorData()`都会将`data`存储在自己的`sensorDataList`之中，但这个过程会有一个**检测机制**，那就是判定当前添加的`data`，与`sensorDatalist`中最新的`data`相比<u>不是最新的</u>，该次<u>添加操作就不会生效</u>。

`systemTime`：该字段也来自于原始数据，但是可能是打印框架附带的额外数据，而不是printf打印出来的内容：

```java
//在parseSensorData函数的前半部分的函数如下：
systemTime = Long.parseLong(components[0]);		//从原始数据字段中获取时间戳
components = Arrays.copyOfRange(components, 2, components.length);	//整理原始数据字段（抛开时间戳字段）
```

下面是parseSensorData函数：
我认为，其中的第二个参数`line`，就是**Mote Output**对话框中的**每一行输出**，所以这里最开始有一个复杂的格式校验。
并且在注释中还有一句是：“    // Sensor data line (probably)” 即，可能有一行输出，满足了该格式，就会触发下面的函数，但其实这行输出并不算数据。

```java
public static SensorData parseSensorData(CollectServer server, String line, long systemTime) {
    String[] components = line.trim().split("[ \t]+");
    // Check if COOJA log
    if (components.length == VALUES_COUNT + 2 && components[1].startsWith("ID:")) {
        if (!components[2].equals("" + VALUES_COUNT)) {
            // Ignore non sensor data
            return null;
        }
        try {
            systemTime = Long.parseLong(components[0]);
            components = Arrays.copyOfRange(components, 2, components.length);
        } catch (NumberFormatException e) {
            // First column does not seem to be system time
        }

    } else if (components[0].length() > 8) {
        // Sensor data prefixed with system time
        try {
            systemTime = Long.parseLong(components[0]);
            components = Arrays.copyOfRange(components, 1, components.length);
        } catch (NumberFormatException e) {
            // First column does not seem to be system time
        }
    }
    if (components.length != SensorData.VALUES_COUNT) {
        return null;
    }
    // Sensor data line (probably)
    int[] data = parseToInt(components);
    if (data == null || data[0] != VALUES_COUNT) {
        System.err.println("Failed to parse data line: '" + line + "'");
        return null;
    }
    String nodeID = mapNodeID(data[NODE_ID]);
    Node node = server.addNode(nodeID);
    return new SensorData(node, data, systemTime);
}
```

我们解析SensorData中的一些getter方法，大致推算出各个字段的意义：

![image-20210328160813379](.\imgs_7\image-20210328160813379.png)

<u>需要特别注意id字段</u>：

假设传输了 `30 0 64 0 1285 ....`

`1285`显然就是NODE_ID，那么这个`1285`如何和我们真实的节点相对应起来呢？答案如下：

```java
  public static String mapNodeID(int nodeID) {
    return "" + (nodeID & 0xff) + '.' + ((nodeID >> 8) & 0xff);
  }
```

或许也不太明显，结合这个图就一目了然了：

![image-20210330233909603](.\imgs_7\image-20210330233909603.png)

也就是1285，对应的节点，就是5号节点。使用代码访问，则需要使用 `var node = sim.getMotes()[4]`。

#### Node

尽管，从逻辑上讲，先理完“Instantaneous Power”后，紧接着讲“Average Power”似乎更为合适，但是经过短暂的代码分析后，我还是认为此时，应该先解析Node类。以下是Node类的成员变量。

id（隐式的节点唯一标识，应当是随机数生成）
name（节点的显示编号，也就是在cooja中显示的节点编号）

```java
public class Node implements Comparable<Node> {

    private static final boolean SINGLE_LINK = true;

    private SensorDataAggregator sensorDataAggregator;
    private ArrayList<SensorData> sensorDataList = new ArrayList<SensorData>();
    private ArrayList<Link> links = new ArrayList<Link>();

    private final String id;
    private final String name;

    private Hashtable<String, Object> objectTable;

    private long lastActive;
    //...
}
```

#### SensorDataAggregator

事实上，单次的sensorData获得的数据，很多时候无法构成具有参考价值的信息，所以需要使用一个类将这些信息聚合起来，才能从中攫取有价值的信息。而这，也正是`SensorDataAggregator`类所需要做的事。每个Node，都会有自己专属的一个`Aggregator`。

这里要注意`SensorDataAggregator`这个对象，我们将结合它的用途来对它进行说明。

```java
public Node(String nodeID, String nodeName) {
    this.id = nodeID;
    this.name = nodeName;
    sensorDataAggregator = new SensorDataAggregator(this);
}
```

在单个`Node`创建的时候，`sensorDataAggregator`也会生成一个实例。

也正因一个`Aggregator`对应一个`Node`，所以`Aggregator`自身不再需要存储每一条`sensorData`，在源码实现的过程中，`Aggregator`也仅仅只是存储了<u>*每条数据之和*</u>。

```java
public void removeAllSensorData() {
    sensorDataList.clear();
    sensorDataAggregator.clear();
}
```

`Node`类，在调用`removeAllSensorData()`的时候，不仅`sensorDataList`会被清空，`sensorDataAggregator`也会被清空。

```java
public boolean addSensorData(SensorData data) {
    //....
    sensorDataAggregator.addSensorData(data);
    return true;
}
```

在`Node`类调用`addSensorData`时，除了将data添加进入`sensorDataList`之中，也会添加到`sensorDataAggregator`之中。

在“Average Power”选项卡中，有如下调用Aggregator的代码：

```java
//new BarChartPanel(this, POWER, "Average Power", "Average Power Consumption",
//...
protected void addSensorData(SensorData data) {
    Node node = data.getNode();
    String nodeName = node.getName();
    SensorDataAggregator aggregator = node.getSensorDataAggregator();
    dataset.addValue(aggregator.getLPMPower(), categories[0], nodeName);
    dataset.addValue(aggregator.getCPUPower(), categories[1], nodeName);
    dataset.addValue(aggregator.getListenPower(), categories[2], nodeName);
    dataset.addValue(aggregator.getTransmitPower(), categories[3], nodeName);
}
```

同样的API调用，Aggregator取得的信息，比单次SensorData取得的信息，更具有统计特性。

```java
/**
除了做最简单将data添加进入Aggregator之中
我们需要着重关注于data的seqno
它之前的seqno很有可能是不准确的 我们需要通过一系列的行为 确定这个seqnoDelta
从而获得"准确的seqno"。
*/
public void addSensorData(SensorData data) {
    //取得当前数据的sequence number
    int seqn = data.getValue(SEQNO);
    /*在函数clear()中 
    clear()函数被Node.java 的 removeAllSensorData()所调用
    seqnoDelta = 0;
    */
    int s = seqn + seqnoDelta;

    //我们求得当前节点的最佳邻居的id
    int bestNeighbor = data.getValue(BEST_NEIGHBOR);
    if (lastNextHop != bestNeighbor && lastNextHop >= 0) {
        //lastNextHop 是上次的最佳邻居节点 
        //如果这个记录发生了变动 这会使得nextHopChangeCount的计数增加
        nextHopChangeCount++;
    }
    lastNextHop = bestNeighbor;

    //如果s小于 sequence number 的限制条件
    if (s <= maxSeqno) {
        //节点存储的SensorData中 我们取得最新的5个SensorData
        for (int n = node.getSensorDataCount() - 1, i = n > 5 ? n - 5 : 0; i < n; i++) {
            SensorData sd = node.getSensorData(i);
            if (sd.getValue(SEQNO) != seqn || sd == data || sd.getValueCount() != data.getValueCount()) {
                /* 检测最新的5个SensorData：
                查他们的sequence number是否与新来的data sequence number有所有不同
                查他们之中是否存在相同的对象（通常情况下，不会出现相同的引用吧）
                查他们的数据格式是否不同（通常情况下都是相同的，所以该条件不存在）
                */
            } else if (Math.abs(data.getNodeTime() - sd.getNodeTime()) > 180000) {
                /* 即便是sequence number 有相同的对象，但是也未必是重复的报文
                因为sequence number的取值是有限的，必定会有循环的可能性存在
                所以这里，加上了对时间的判定（这里的单位我估计应该是ms）
                */
            } else {
                /*如果sequence number相同 并且时间并没有超过 阈值
                那么这个新来的数据包 就有可能是一个重复的数据包
                （注意，只是有可能是一个重复的数据包，但是也未必，需要进一步的检验）
                */
                data.setDuplicate(true);

                // Verify that the packet is a duplicate
                for (int j = DATA_LEN2, m = data.getValueCount(); j < m; j++) {
                    if (sd.getValue(j) != data.getValue(j)) {
                        /*检查二者的每个字段值
                        如果至少有一个字段值不同，就可以说明新来的数据包并不算一个重复数据包
                        */
                        data.setDuplicate(false);
                        break;
                    }
                }
                if (data.isDuplicate()) {
                    /* 如果最终该数据包被判定为重复的数据包;
                    那duplicates计数会增加（不会重复增加，因为增加了之后，使用了break）
                    */
                    duplicates++;
                    break;
                }
            }
        }
    }

    if (!data.isDuplicate()) {
        /* 如果该数据包最终不是一个重复的数据包
        VALUES_COUNT 的取值是30 （它的值只有可能比values.length更小，更大的话，不起作用）
        */
        for (int i = 0, n = Math.min(VALUES_COUNT, data.getValueCount()); i < n; i++) {
            //将不重复的数据包的数值的每个字段 添加到values数组中
            //显而易见values[i] 的作用就是记录下该类数据的总值
            values[i] += data.getValue(i);
        }
        // getSensorDataCount()->获得sensorData的个数 如果为2甚至更多
        if (node.getSensorDataCount() > 1) {
            //我们可以求出 第二新的sensorData 与 这个新加入的sensorData的nodeTime作差
            //为什么需要求"第二新"的？
            long timeDiff = data.getNodeTime() - node.getSensorData(node.getSensorDataCount() - 2).getNodeTime();
            //我们把最长的这个timeDiff记录在longestPeriod中
            if (timeDiff > longestPeriod) {
                longestPeriod = timeDiff;
            }
            //把最短的timeDiff会记录在shortestPeriod中
            if (timeDiff < shortestPeriod) {
                shortestPeriod = timeDiff;
            }
        }
        /* 关于dataCount 只在该类的clear()函数中被重置过
        Node.java 中 public void removeAllSensorData() 该方法
        也会调用该类的clear()方法
        */
        if (dataCount == 0) {
            // 显然 如果之前列表中，压根儿就没有sensorData存在 我们的seqno当然是准确的
            // s == seqno 可以直接使用
        } else if (maxSeqno - s > 2) {
            // Handle sequence number overflow.
            seqnoDelta = maxSeqno + 1;
            s = seqnoDelta + seqn;
            if (seqn > 127) {
                // Sequence number restarted at 128 (to separate node restarts
                // from sequence number overflow).
                seqn -= 128;
                seqnoDelta -= 128;
                s -= 128;
            } else {
                // Sequence number restarted at 0. This is usually an indication that
                // the node restarted.
                nodeRestartCount++;
            }
            if (seqn > 0) {
                lost += seqn;
            }
        } else if (s > maxSeqno + 1) {
            lost += s - (maxSeqno + 1);
        }
        /* minSeqno 与 maxSeqno 均来自s迭代后的结果
       	private int minSeqno = Integer.MAX_VALUE;
  		private int maxSeqno = Integer.MIN_VALUE;
        */
        if (s < minSeqno)
            minSeqno = s;
        if (s > maxSeqno)
            maxSeqno = s;
        dataCount++;
    }
    //尽管data在构造函数中 就因序列传值获得了seqno
    //但在经过与之前的数据做对比后 seqno可能会发生变动 这一步也是为了将变动后的 seqno赋值给它
    data.setSeqno(s);
}

```


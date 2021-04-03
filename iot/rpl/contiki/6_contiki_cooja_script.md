# Cooja Using Script

## 运行基础

使用脚本仿真，取代手动拖动物件仿真。

1. 在Cooja软件中，使用 `Tools -> Simulation script editor` 。、

   打开了editor后，你可以看到editor内的脚本代码，这些**脚本代码的执行时机**是：
   当脚本代码被**运行激活**后，再启动模拟。

2. 在 `editor` 中，使用 `run -> activate` ，可以将脚本代码**运行激活**。

### 背景常量

- `se.sics.cooja.Mote mote` The Mote [Mote.java]()
- `int id` The id of the mote
- `long time` The current time
- `String msg` The message
- `se.sics.cooja.Simulation sim` The simulation [Simulation.java]()
- `se.sics.cooja.Gui gui` The GUI [Gui.java]()
- `interface ScriptLog log`
- `log(String log)`
- `public void testOK()`
- `public void testFailed()`
- `public void generateMessage(long delay, String msg)` This generates a message after delay. This can be used to wait without having a mote to output anything.
- `boolean TIMEOUT` Set to true when the script times out
- `boolean SHUTDOWN` Set to true when the script is deactivated
- `java.util.concurrent.Semaphore.Semaphore SEMAPHORE_SCRIPT`
- `java.util.concurrent.Semaphore.Semaphore SEMAPHORE_SIM`

## 代码解析

详细script推荐的文档，参照contiki-os的官方wiki：
`https://github.com/contiki-os/contiki/wiki/Using-Cooja-Test-Scripts-to-Automate-Simulations#activate-the-test`

从官方的文档中，我整理出**非常关键的一个信息**，这个信息也是直接关系到了后面挖掘代码的这个方法：
script editor 使用的 JavaScript 作为脚本语言，但是这些<u>脚本操作的对象，实际却是使用java编写的</u>。

在这里我列出了，我自己整理的一些常见的类。

### Simulation

Simulation类，毫无疑问，这对应的就是我们每次“New simulation”创建的一次模拟活动。

在该类中，有许多的方法，那么该如何查看到这些方法呢？这里参照官方wiki，我们找到它所在的位置：
`~\tools\cooja\java\se\sics\cooja\Simulation.java` 。
在`Simulation.java`中，有如下定义：

```java
public Mote[] getMotes(){
    //....
}
```

这使得我们可以在script editor中，使用如下语句，进行调用：

```javascript
//使用getMotes 完全同名即可（包括大小写） 
motes = sim.getMotes();
log.log(motes.length + "\n")
for(var i = 0 ; i < motes.length ; ++i){
   mt = motes[i]
   /**
   * doing work
   **/
}
```

由此，就可以看出script editor 与 java 源代码之间的内涵关系了。其中Simulation的其他的方法，我们在这里就先不做过多的介绍。

### Mote

`~\tools\cooja\java\se\sics\cooja\Mote.java`

我想着，Mote作为仿真重要的一环，那么仿真中Mote的位置属性，必定也应该存储在该对象中，Mote.java源代码如下：

```java
public interface Mote {
  public int getID();
  public MoteInterfaceHandler getInterfaces();
  public MoteMemory getMemory();
  public MoteType getType();
  public Simulation getSimulation();
  public abstract Collection<Element> getConfigXML();
  public abstract boolean setConfigXML(Simulation simulation,
      Collection<Element> configXML, boolean visAvailable);
  public void removed();
  public void setProperty(String key, Object obj);
  public Object getProperty(String key);
}
```

我们发现，并没有在其中找到与位置相关的信息。但显然它应该是具有这个信息的，所以经过进一步的查找，我们发现它的位置信息存储在 `public MoteInterfaceHandler getInterfaces();` 这里的 `MoteInterfaceHandler` 之中。

`~\tools\cooja\java\se\sics\cooja\MoteInterfaceHandler.java`在该文件中，我们找到了如下代码

```java
public class MoteInterfaceHandler {
  private static Logger logger = Logger.getLogger(MoteInterfaceHandler.class);

  private ArrayList<MoteInterface> moteInterfaces = new ArrayList<MoteInterface>();

  /* Cached interfaces */
  /*....*/
  private Position myPosition;
  /*....*/
}
```

紧接着，我们就找到了 `~\tools\cooja\java\se\sics\cooja\interfaces\Position.java` 这个文件。

```java
@ClassDescription("Position")
public class Position extends MoteInterface {
  private static Logger logger = Logger.getLogger(Position.class);
  private Mote mote = null;
  private double[] coords = new double[3];
  /*...*/
  public void setCoordinates(double x, double y, double z) {
    /*...*/
  }
  /*...*/
  public double getXCoordinate() {
    /*...*/
  }
  /*...*/
  public double getYCoordinate() {
    /*...*/
  }
  /*...*/
  public double getZCoordinate() {
    /*...*/
  }
```

## 测试运行

### eg1：打印所有Mote坐标

```javascript
 motes = sim.getMotes();
 log.log(motes.length + "\n")
 for(var i = 0 ; i < motes.length ; ++i){
    mt = motes[i]
    mt_interface = mt.getInterfaces()
    mt_position = mt_interface.getPosition()
    log.log(mt_position.getXCoordinate())
    log.log("\n")
    log.log(mt_position.getYCoordinate())
    log.log("\n")
    log.log(mt_position.getZCoordinate())
    log.log("\n")
    log.log("-------------\n")
 }
```

---

```
4
10.451066908576557
89.37159229005276
0
-------------
34.5916363263297
56.9842114612881
0
-------------
56.77663769086854
7.396217258943949
0
-------------
3.0244822125656934
86.23492602429121
0
-------------
```

我们在这个基础上，稍作修改，即可让它动态发生变化（因为Z轴未使用，我们在这里不再设置z轴的坐标）：

```javascript
 TIMEOUT(100000);
 log.log("first simulation message at time : " + time + "\n");
 motes = sim.getMotes();
 log.log(motes.length + "\n")
 for(var i = 0 ; i < motes.length ; ++i){
    mt = motes[i]
    mt_interface = mt.getInterfaces()
    mt_position = mt_interface.getPosition()
    mt_position.setCoordinates(i * 2 ,i * 2 ,0);
    log.log(mt_position.getXCoordinate())
    log.log("\n")
    log.log(mt_position.getYCoordinate())
    log.log("\n")
    log.log("-------------\n")
 }
 while (true) {
    YIELD(); /* wait for another mote output */
	mt_interface = mote.getInterfaces()
	mt_position = mt_interface.getPosition()
    log.log("id:" + id + "  msg:" + msg + "  \n")
    x = mt_position.getXCoordinate()
    y = mt_position.getYCoordinate()
	mt_position.setCoordinates(x + 1 ,y + 1 ,0);
 }
```

### eg2：每个固定时间，执行一次操作

```javascript
function getMessage() {
  return "nihao";
}
for (var i = 0; i < 4; ++i) {
  GENERATE_MSG(20000, "sleep"); //Wait for twenty sec
  YIELD_THEN_WAIT_UNTIL(msg.equals("sleep"));
  log.log(getMessage())
}
```

---

更事件化的一种编码结构

```javascript
/**
 * @Member
 * counter = -1
 * @Method
 * increase() : 'counter' will increase after this method called.
 *  And if the new Value of 'counter' is 0, then will return true
 */
eventTimer = {
  counter: -1,
  counterMod: 10,
  increase: function () {
    this.counter++;
    this.counter %= this.counterMod
    return this.counter == 0
  }
}

moteMover = {
  datas: sim.getMotes(),
  setDirection: function () {
    
  },
  moveDirection: function () {

  }
}

function getMessage() {
  return "nihao";
}
while (true) {
  GENERATE_MSG(100, "sleep");
  YIELD_THEN_WAIT_UNTIL(msg.equals("sleep"));
  if (eventTimer.increase()) {
    battery = moteMover.datas[0].getInterfaces().getBattery()
    log.log(battery)
    log.log("\n")
  } else {
    log.log("+")
  }
}
```

### eg3：文件IO，输入输出

```javascript
//import Java Package to JavaScript
importPackage(java.io);
a = new FileWriter("log_2.txt");
a.write("nihao")
a.close()
b = new FileReader("log_1.txt");
log.log("ok?")
while((ch = b.read()) != -1){
    log.log(String.fromCharCode(ch))
}
```

注意这里填入的相对路径，实际的文件都默认存在： `~/tools/cooja/build/...` 路径下
比如 `"~/tools/cooja/build/log_1.txt"` 。

另一个需要注意的是， `b.read()` 读取到的字节，在js中会被转成整型数，这显然与文本中原始存储的数据是有差别的。为此我们需要重新对它使用 `fromCharCode()` 。

### eg4：一个完整的bonnmotion模拟文件

在该文件中，我们需要读取文件，而这个文件显然需要我们进行预先的填入。
这个文件是通过bonnmotion生成的（需要生成后，再使用bonnmotion的转义工具）

 `var fileReader = new FileReader("campus.csv");`

```javascript
importPackage(java.io);
function removeEmptyEle(datas) {
  datas.splice(datas.length - 1, 1);
}
function processCSVData(pq, datas) {
  for (var i = 0; i < datas.length; i++) {
    var data = datas[i].split(' ');
    var dataIndex = parseInt(data[1])
    var newData = {}
    //cooja中是 "1 ~ n" 但是在bonnmotion中是 "0 ~ n-1"
    newData["id"] = parseInt(data[0]) + 1;
    newData["time"] = data[1];
    newData["x"] = data[2];
    newData["y"] = data[3];
    pq.push(new QueueElement(dataIndex, newData))
  }
}
function resetPosition(changeObj) {
  var index = changeObj.id - 1;
  log.log("reset index : " + typeof index + " " + index + "changeObj.id : " + changeObj.id  + "\n")
  var ele = motes[index];
  if (ele.getID() != changeObj.id) {
    log.log("RESET POSITION ID ERROR \n");
  }
  var pos = ele.getInterfaces().getPosition()
  pos.setCoordinates(changeObj.x, changeObj.y, 0)
}
function refreshPositionByMap(pq, nowSecond) {
  while (pq.size() > 0) {
    //pq 不为空的时候
    var key_value = pq.peek();
    var value = key_value.val;
    if (nowSecond < value.time) {
      //不再需要被抛出新元素了
      break
    }
    resetPosition(value)
    pq.pop();
  }
}
function QueueElement(key, val) {
  this.key = key;
  this.val = val;
}
QueueElement.prototype.compare = function (ele) {
  if (this.key < ele.key) {
    return true;
  }
  return false;
}
function PriorityQueue(length) {
  this.data = new Array(length);
  this.limit = 0;
  this.maxLimit = length - 1
}
PriorityQueue.prototype.peek = function () {
  if (this.limit > 0) {
    return this.data[1];
  }
  return null;
}
PriorityQueue.prototype.push = function (val) {
  if (this.limit < this.maxLimit) {
    this.limit++;
    this.data[this.limit] = val
    this.swim()
    return true
  } else {
    return false;
  }
}
PriorityQueue.prototype.pop = function () {
  if (this.limit > 0) {
    this.data[1] = this.data[this.limit];
    this.limit--;
    this.sink()
    return true;
  } else {
    return false;
  }
}
PriorityQueue.prototype.size = function () {
  return this.limit;
}
PriorityQueue.prototype.getFather = function (son) {
  return Math.floor(son / 2)
}
PriorityQueue.prototype.getLeftChild = function (father) {
  return father * 2;
}
PriorityQueue.prototype.getRightChild = function (father) {
  return father * 2 + 1;
}
PriorityQueue.prototype.swim = function () {
  var subIndex = this.limit;
  var fatherIndex = this.getFather(subIndex)
  while (fatherIndex != 0 && this.data[subIndex].compare(this.data[fatherIndex])) {
    /**父子节点的交换 */
    var temp = this.data[subIndex];
    this.data[subIndex] = this.data[fatherIndex];
    this.data[fatherIndex] = temp;
    subIndex = fatherIndex;
    fatherIndex = this.getFather(fatherIndex);
  }
  return this.data
}
PriorityQueue.prototype.sink = function () {
  var sinkIndex = 1
  while (true) {
    var leftIndex = this.getLeftChild(sinkIndex)
    var rightIndex = this.getRightChild(sinkIndex)
    var newSinkIndex;
    if (rightIndex <= this.limit) {
      if (this.data[leftIndex].compare(this.data[rightIndex])) {
        //left 更大
        newSinkIndex = leftIndex;
      } else {
        newSinkIndex = rightIndex;
      }
    } else if (leftIndex <= this.limit) {
      //说明right 已经超过了 limit
      newSinkIndex = leftIndex;
    }
    else {
      //说明两个节点都已经超过了limit 说明已经是叶子节点了 此时可以直接跳出循环
      break;
    }
    if (this.data[newSinkIndex].compare(this.data[sinkIndex])) {
      //如果新节点比原节点更小 此时才需要下沉
      var temp = this.data[sinkIndex];
      this.data[sinkIndex] = this.data[newSinkIndex];
      this.data[newSinkIndex] = temp;
      sinkIndex = newSinkIndex
    } else {
      break
    }
  }
}
var fileReader = new FileReader("campus.csv");
var datas = ""
var motes = sim.getMotes()
while ((ch = fileReader.read()) != -1) {
  datas += (String.fromCharCode(ch))
}
var priorityQue = new PriorityQueue(10000);
datas = datas.split('\n')
removeEmptyEle(datas)
processCSVData(priorityQue, datas)

while (true) {
  GENERATE_MSG(1000, "sleep"); //Wait for twenty sec
  YIELD_THEN_WAIT_UNTIL(msg.equals("sleep"));
  refreshPositionByMap(priorityQue, time / 1000000)
}
```


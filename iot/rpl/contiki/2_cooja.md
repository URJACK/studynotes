# cooja

再官方自带的镜像中，cooja这个软件就在桌面上。双击就可以启动cooja了。

## 基础运行

1. 运行cooja后，点击左上角的<u>File</u>，选中<u>New simulation</u>。

![image-20210114191452135](.\imgs_2\image-20210114191452135.png)

`Fig 1：创建新的simulation`

```
simulation是仿真的一个整体环境，内部包含了所有的仿真对象、事件等参数。

除了创建新的simulation，“Open Simulation”打开一个simulation文件也是可行的。
```

2. 创建新的Simulation

你需要给这次Simulation配置一些更为精确信息。当然，通常情况下，很多属性选择默认的就行了。我在这里只改了一个参数，就是将“New random seed on reload”勾选上了。

![image-20210114191959591](.\imgs_2\image-20210114191959591.png)

`Fig 2：配置新的simulation的参数`

Radio medium（广播媒体）：默认选择为<u>UDGM：Distance Loss</u>

```
UDGM（Unit Disk Graph Medium）：Distance Loss
UDGM（Unit Disk Graph Medium）：Constant Loss
DGRM（Directed Graph Radio Medium）
No radio traffic
Multi-path Ray-tracer Medium（MRM）
```

Mote Startup Delay（微粒启动延迟）：

```
这里我使用了默认值：1000ms。
```

Random seed（随机种子）：

```
自己设置随机数种子，当然，勾选了New random seed on reload后，就不需要设置这个参数了。
```

3. 添加节点

在进入一个Simulation后，就可以进行添加节点的操作了，我们先进入添加节点的界面。

点击<u>Motes</u>-><u>Add motes</u>-><u>create new mote type</u>-><u>sky mote</u>

![image-20210114193346556](.\imgs_2\image-20210114193346556.png)

`Fig 3：添加sky mote`

4. 添加sink节点

执行完上一步后，我们可以尝试添加sink节点了。

首先是Description，我们将其名字更改为 <u>..... #sink</u>

其次我们需要选择一个sink节点的文件，文件里包含了节点的运行逻辑。

1. 在“Contiki process”属性中，这里选择一个c文件，就可以对c文件进行“compile”。
2. “compile”完成后，会生成一个.sky文件（如图就为udp-sink.sky），并且默认会指向这个sky文件，此时，“create”变得可以点击，点击create即可创建节点。

![image-20210114193723076](.\imgs_2\image-20210114193723076.png)

`Fig 4：选中需要的文件，添加Mote`

5. 更丰富的视图信息

这里，我们再更改一下视图的选项，让视图的信息更为丰富。

![image-20210114194448578](.\imgs_2\image-20210114194448578.png)

`Fig 5：更改视图的展示信息`

6. 添加sender结点

同样的，与添加sink节点类似，除了sink节点之外，我们可以添加sender节点。

Contiki process的路径为

```
“/home/user/contiki/examples/ipv6/rpl-collect/udp-sender.c”
```

![image-20210114194819719](.\imgs_2\image-20210114194819719.png)

`Fig 6：添加更多的sender节点`

7. 打开收集窗口、启动收集窗口。

> 尽管不打开收集窗口，依然可以进行仿真，但是没有收集窗口的仿真，没有办法把我们想要的结果以很好的方式展示出来，所以在仿真之前，我们需要打开收集窗口。

![image-20210114195542962](.\imgs_2\image-20210114195542962.png)

`Fig 7：添加collect view`

选中<u>Tools</u> -> <u>Collect View</u> -> <u>Sky 1...</u>

就可以<u>创建一个收集窗口</u>

![image-20210114195800188](.\imgs_2\image-20210114195800188.png)

`Fig 8：启动收集窗口`

如图，图中的小框代表收集窗口。

而**大框**，则可以对这个收集窗口做响应的控制。

在**大框**中，选择<u>Node Control</u> -> <u>Base Station Control</u> -> <u>Start Collect</u>

就可以<u>启动收集窗口</u>。之后再启动仿真就可以了。

8. 启动仿真

![image-20210114201046220](.\imgs_2\image-20210114201046220.png)

`Fig 9：启动仿真`

Start：启动仿真

Pause：暂停仿真

Step：仿真时间推进0.001s

Reload：重新回到0s的时刻

## 详细信息


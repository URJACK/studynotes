# BonnMotion

## 介绍

<u>BonnMotion是一款Java软件</u>，可创建和分析移动性方案，<u>最常用作调查移动自组织网络特征的工具</u>，还可以将方案导出到多个网络模拟器，例如ns-2，ns-3，GloMoSim / QualNet，COOJA，MiXiM和ONE。 BonnMotion由德国波恩大学的通信系统小组，美国科罗拉多州戈尔德的科罗拉多矿业学院的Toilers小组和德国奥斯纳布吕克大学的分布式系统小组联合开发。

## 安装

1. 安装须知

   因为BonnMotion是一款Java软件，这使得BonnMotion的安装脚本，需要进行配置Java的路径。

   ```bash
   @echo off
   
   REM *********************************************
   REM Please configure for your enviroment
   
   set JAVAPATH=D:\softwares\jdk
   set BONNMOTION=D:\softwares\bonnmotion\bonnmotion-3.0.1
   
   REM *********************************************
   ```

   注意这里的路径，JAVAPATH和BONNMOTION的路径中，最好不要包含“空格”。

   例如“Program Files (x86)”这个路径，会引发安装的失败。

2. 安装完成后

   下面这段路径，就是可执行的程序了。

   ```
   bonnmotion-3.0.1\bin\bm.bat
   ```

## 使用

下面描述的所有应用程序都是通过bm包装程序脚本启动的。 语法如下：

```shell
$ bm < parameters > < application > < application parameters >
```

该应用程序可以是移动性模型或例如。 用于分析场景特征的统计应用程序。

### 查看帮助

在不使用命令行参数的情况下启动脚本将提供更多帮助。 使用示例在以下各节中给出：

```bash
PS D:\softwares\bonnmotion\bonnmotion-3.0.1\bin> .\bm.bat -h 
BonnMotion 3.0.1



Help:
  -h                            Print this help

Scenario generation:
  -f <scenario name> [-I <parameter file>] <model name> [model options]
  -hm                           Print available models
  -hm <module name>             Print help to specific model

Application:
  <application name> [Application-Options]
  -ha                           Print available applications
  -ha <application name>        Print help to specific application
PS D:\softwares\bonnmotion\bonnmotion-3.0.1\bin>
PS D:\softwares\bonnmotion\bonnmotion-3.0.1\bin> .\bm.bat   
BonnMotion 3.0.1

OS: Windows 10 10.0
Java: Oracle Corporation 1.8.0_144


Help:
  -h                            Print this help

Scenario generation:
  -f <scenario name> [-I <parameter file>] <model name> [model options]
  -hm                           Print available models
  -hm <module name>             Print help to specific model

Application:
  <application name> [Application-Options]
  -ha                           Print available applications
  -ha <application name>        Print help to specific application
PS D:\softwares\bonnmotion\bonnmotion-3.0.1\bin> 
```

### 生成场景

当前，存在几种可用的移动性模型，稍后将对它们进行介绍，并对实现细节进行一些评论。 有关模型本身的更多详细信息，请参考文献。 在过去的十年中，提出了各种综合模型。已经进行了几项一般性调查[9、6、4、22]，以及一些针对车辆模型的特定调查[13]。

将输入参数输入到场景生成中有两种可能性：第一种是在命令行上输入参数，第二种是使用包含这些参数的文件。 这两种方法也可以结合使用。 在这种情况下，命令行参数将覆盖输入文件中给定的参数。

方案生成器将用于创建特定方案的所有参数写入文件。通过这种方式，可以保存设置，并且可以更改特定方案参数，而无需重新输入所有其他参数。

所有模型使用的重要参数如下：节点号用-n设置，方案持续时间（以秒为单位）用-d设置，而-i参数指定应在方案开始时跳过多少秒。 使用-x和-y设置模拟区域的宽度和高度（以米为单位）。 某些模型（例如RandomWaypoint）支持3D运动。 指定-z设置模拟区域的深度（以米为单位），并生成3D运动。 在这种情况下，输出也默认为3D，否则默认为2D。 但是，请注意，可以通过提供-J <2D | 3D>来独立控制输出尺寸，该值将根据任何情况下提供的参数覆盖输出尺寸。 使用-R，可以手动设置随机种子。

切断初始阶段是一个重要功能，因此-i具有较高的默认值：观察到，使用随机航点模型时，节点在模拟区域中心附近的概率较高，而它们最初在节点上均匀分布 模拟区域。 在我们实现曼哈顿网格模型的过程中，为简单起见，所有节点都从（0,0）开始。

#### 生成场景

```shell
PS D:\softwares\bonnmotion\bonnmotion-3.0.1\bin> .\bm.bat -f scenario1 RandomWaypoint -n 100 -d 900 -i 3600
BonnMotion 3.0.1

OS: Windows 10 10.0
Java: Oracle Corporation 1.8.0_144


Starting RandomWaypoint ...
Next RNG-Seed =1035022552536174885 | #Randoms = 21287
RandomWaypoint done.
Runtime: 0 sec
PS D:\softwares\bonnmotion\bonnmotion-3.0.1\bin> 
```

这将创建一个随机路点场景，其中包含100个节点，持续时间为900秒。初始阶段为3600秒被切断。
场景保存在两个文件中：<u>第一个带有后缀.params，包含用于模拟的完整参数集</u>。 <u>第二个带有后缀.movements.gz包含（压缩后的）运动数据</u>。（<u>这些文件保存在bonnmotion的根目录下</u>）

#### 基于参数生成场景

现在，您可以看一下cenario1.params文件，以查看用于仿真的所有参数，如果您想创建一个类似的场景，只是具有更高的移动性，则可以使用

```shell
PS D:\softwares\bonnmotion\bonnmotion-3.0.1\bin> .\bm.bat -f scenario2 -I scenario1.params RandomWaypoint -h 5.0  
BonnMotion 3.0.1

OS: Windows 10 10.0
Java: Oracle Corporation 1.8.0_144


Starting RandomWaypoint ...
randomSeed (String):1611058104504
randomSeed (Long):1611058104504
Next RNG-Seed =2313430157882850241 | #Randoms = 31717
RandomWaypoint done.
Runtime: 0 sec
PS D:\softwares\bonnmotion\bonnmotion-3.0.1\bin> 
```

scenario1.params

```
model=RandomWaypoint
ignore=3600.0
randomSeed=1611058104504
x=200.0
y=200.0
duration=900.0
nn=100
circular=false
J=2D
dim=4
minspeed=0.5
maxspeed=1.5
maxpause=60.0
```

scenerio2.params

```
model=RandomWaypoint
ignore=3600.0
randomSeed=1611058104504
x=200.0
y=200.0
duration=900.0
nn=100
circular=false
J=2D
dim=4
minspeed=0.5
maxspeed=5.0
maxpause=60.0
```

可以看出仅在maxspeed这个属性处有变化。

#### 无边界模拟区域模型-Boundless

根据[9]创建方案。 将2D矩形模拟区域转换为圆环形模拟区域（不存在边界）的模型。 此外，此模型的节点在先前的行进方向和速度与其当前行进方向和速度之间具有某些特殊关系。 在此模型中，速度矢量和位置均按照以下公式每Δt时间步更新一次：

其中Vmax是模拟中定义的最大速度，∆v是速度的变化，均匀地分布在[Amax * ∆t，Amax ∗ ∆t]之间，Amax是给定MN的最大加速度，∆θ是方向变化 它均匀地分布在[α∗Δt，α∗Δt]和α是MN移动方向上的最大角度变化。

#### 链模型-ChainScenario

Chain模型不是模型本身，而是本文档中描述的已实现模型的串联。在某些情况下，有必要对场景进行建模，在这些场景中，移动节点根据时间和位置以不同的方式运行。使用链模型，移动节点在第N-1个场景的最终位置将链接到第N个场景的初始位置。
Chain模型的工作方式如下：它允许指定一些已知的模型（例如，Random Waypoint，Manhattan，RPGM等），每个模型都有自己的一组参数，如本文档中所定义。因此，可以分别定义每个模型的行为。属于链条的每个模型都必须用引号（“”）括起来，以便与其他模型区分开。
除了各个模型之外，Chain模型还接受两个参数：-m <模式>指定第N个场景的初始位置是否具有2秒的延迟（模式= 1）或不具有2秒的延迟（模式= 0，默认值）。可选参数-P可以为整个链方案中的每个方案生成一组文件。

必须沿所有单个模型协调两个参数。 对于所有方案，移动节点的数量必须相同，否则链式方案生成将失败。 场景之间的模拟区域可能会有所不同，但是如果第N-1个场景的最终位置不在第N个场景的模拟范围之内，则生成也会失败。
下面的示例生成一个名为chainTest的链方案，并将aRandomWaypoint和ManhattanGrid方案连接起来。 完整的链方案的持续时间为400秒（每个型号为200秒），并且具有10个移动节点（请记住，该编号对于所有型号都必须相同）。 由于-m参数定义为1，因此将在两种情况之间引入2秒的延迟：

```shell
PS D:\softwares\bonnmotion\bonnmotion-3.0.1\bin> .\bm.bat  -f chainTest ChainScenario -m 1 -P "RandomWaypoint -d 200 -n 10 -x 500 -y 500 -o 3" "ManhattanGrid -d 200 -n 10 -x 500 -y 500"  
BonnMotion 3.0.1

OS: Windows 10 10.0
Java: Oracle Corporation 1.8.0_144


Starting ChainScenario ...
Mode: 1
Will write parts.
  Starting Model(0): RandomWaypoint
Next RNG-Seed =6768311630887866479 | #Randoms = 547
  done.
  Starting Model(1): ManhattanGrid
Next RNG-Seed =1017579905274511919 | #Randoms = 9788
  done.
ChainScenario done.
Runtime: 0 sec
PS D:\softwares\bonnmotion\bonnmotion-3.0.1\bin> 
```

---

最终会生成三组文件

<img src=".\imgs\image-20210119203415223.png" alt="image-20210119203415223" style="zoom: 67%;" />

连锁模型何时可以帮助从现实中建模场景，考虑具有校园的城市（可以是工厂，大学等）的示例。 对校园内部的流动性进行建模，以及一段时间后，看看学生从校园搬到家中会发生什么，这可能会很有趣。

<img src=".\imgs\image-20210119203606727.png" alt="image-20210119203606727" style="zoom:67%;" />

以下行为此生成了一个链方案：

```shell
PS D:\softwares\bonnmotion\bonnmotion-3.0.1\bin> .\bm.bat -f campus ChainScenario -m 1 "RandomWaypoint -d 200 -n 10 -x 100 -y 100 -o 3" "ManhattanGrid -d 200 -n 10 -x 400 -y 300 -u 4 -v 3"
BonnMotion 3.0.1

OS: Windows 10 10.0
Java: Oracle Corporation 1.8.0_144


Starting ChainScenario ...
Mode: 1
  Starting Model(0): RandomWaypoint
Next RNG-Seed =8269318663040937043 | #Randoms = 1756
  done.
  Starting Model(1): ManhattanGrid
Next RNG-Seed =6283924643490263085 | #Randoms = 8981
  done.
ChainScenario done.
Runtime: 0 sec
PS D:\softwares\bonnmotion\bonnmotion-3.0.1\bin> 
```

#### 列流动性模型-Column

根据[9]创建方案。 组移动性模型，其中的MN组形成一条线，并沿特定方向均匀地向前移动。 在此模型中，用户将指定节点数和组数。 节点数必须可被组数平均除尽，即所有节点组必须完全填充-不能有孤节点。 参考点列选取一个随机的方向角和随机的运动矢量。节点在地图上遵循其各自的参考点。 它们具有-s <maxDist>参数，该参数确定其围绕参考点的随机运动可能有多远-就像Nomadic模型一样。

#### 灾区模型-DisasterArea

根据[2]创建方案。

- 可以使用-b参数在该区域的坐标列表后定义战术区域（以“，”分隔）。 此列表中的最后三个值是类型，所需组和传输组。
- 事故地点的类型可以为0，等待治疗区域的患者为1，待命清除站为2，技术操作命令为3，救护车停放点为4。
- 对于每个指定的事故发生地点，必须有一个指定等待治疗区域的患者，每个等待治疗区域的患者都有一个伤亡清除站，每个救护车停放处都有一个伤亡清除站。

#### 原始高斯马尔可夫模型-OriginalGaussMarkov

Gauss-Markov模型的这种实现遵循出版物[17]。 在该实施方式中，不直接指定平均速度矢量μ； 取而代之的是，使用-a来指定范数，并将具有该范数的随机向量分配给每个站。 当然，范数0仅产生向量（0,0）。 该实现还允许用户指定最大速度。 范数较大的速度矢量将与适当的标量相乘，以将速度减小到最大速度。
该模型已通过以下方式适用于处理场景边界：如果驻站移动到边界，则其速度矢量及其预期速度矢量将被“镜像”。

#### 高斯马尔可夫模型-GaussMarkov

这是Gauss-Markov模型的实现，尽管不尽相同，但仍遵循[9]中的描述。 主要的共同点是，对于每个移动节点，将维护两个单独的值，而不是一个速度矢量：移动站的速度及其移动方向。 同样，处理从模拟区域移出的移动节点的默认方法与[9]密切相关：节点可能会继续走出区域边界，这将导致下一个运动向量更新不是基于先验角度，而是基于角度 将节点带回现场。 因此，字段大小自动适应场景生成后的节点移动。

与[9]的主要区别在于，新的速度和运动方向仅是从正态分布中选择的，而正态分布分别是相应的旧值（标准偏差是在命令行上使用-a和-s指定的）。 速度值被限制为可以使用-m和-h在命令行上指定的某个时间间隔：如果新选择的速度值在此间隔之外，则将其更改为该间隔内的最接近值（最小值或最小值）。 最大值）。
可以使用几个命令行开关来修改上述行为：使用-b，模拟区域的大小是固定的，并且节点仅在区域边界处“反弹”。使用-u，有效速度区间之外的速度值为： 以导致节点速度均匀分布（而不是间隔边界周围的峰值）的方式进行调整。

### 其他类型数据格式

以生成CSV文件格式为例，我们需要先生成原始文件 再使用格式转换。

```
.\bm.bat -f campus RandomWaypoint -n 49 -d 900 -i 3600
```

```
.\bm.bat CSVFile -f campus
```


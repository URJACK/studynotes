# Miniedit

进入mininet/examples目录 执行miniedit.py

## 使用样例

### 例1-基础网络

![image-20210109104026991](D:\LearningNotes\linux\mininet\image-20210109104026991.png)

最简单的一个拓扑结构

我们依次进行如下步骤：

1. 配置控制器的属性：

   更改**ControllerType**字段为<u>Remote Controller</u>。

2. 配置交换机的属性：

   设置**DPID**字段，这个字段必须有16个数：“0000000000000001”诸如此类的

   更改**SwitchType**字段为<u>Open vSwitch Kernel Mode</u>。（全局配置中，也可以进行更改）

3. 配置用户终端的属性：

   设置用户的**IP Address**字段。

4. 配置链路的属性：

5. Mininet的全局配置（左上角）：

   设置**IP Base**字段。

   设置**Start CLI**字段：进而允许终端的操作

   设置**Default Switch**字段：设置默认交换机的类型。

6. 点击（左下角）的RUN：

### 例2-流量随机模型

使用iperf工具，在网络中生成UDP流量。

1. 修改mininet/net.py ##功能代码实现

2. 修改mininet/cli.py ##注册命令

3. 修改bin/mn ##加入到可执行文件中

4. 重新安装Mininet核心文件

   mininet/util/install.sh -n


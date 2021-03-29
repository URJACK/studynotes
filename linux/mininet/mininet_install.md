# Mininet

## 简介

Mininet是一个进程虚拟化网络仿真工具。可以创建一个SDN网络（software defined network，软件定义网络）

包含（主机、交换机、控制器）

交换机支持OpenFlow协议。

![image-20201231084539465](.\mininet_install\image-20201231084539465.png)

Mininet可以做的事：

1. Mininet为openFlow应用程序提供一个简单便宜的网络测试平台。
2. 启用复杂的拓扑测试，无需物理网络
3. 具有拓扑感知的OpenFlow感知的CLI
4. 支持任意自定义拓扑，主机数可达4096
5. 提供扩展的Python API

为什么选择Mininet：

1. 与仿真器相比，启动快、扩展性大、带宽提供多、方便安装。
2. 与模拟器相比，运行真实代码，容易连接真实网络
3. 与硬件测试床相比，快速配置及重新启动

## 安装(ubuntu20，安装失败)（git源码进行安装）

使用git进行安装

### **1·git clone源码**

```shell
kazetoyuki@ubuntu:~$ git clone https://github.com/mininet/mininet.git
```

`进入源码中的“util路径”，在内部查看安装命令的额外参数有哪些。（使用-h）`

```shell
kazetoyuki@ubuntu:~/mininet/util$ ./install.sh -h
Detected Linux distribution: Ubuntu 20.04 focal amd64
sys.version_info(major=3, minor=8, micro=2, releaselevel='final', serial=0)
Detected Python (python3) version 3

Usage: install.sh [-abcdfhikmnprtvVwxy03]

This install script attempts to install useful packages
for Mininet. It should (hopefully) work on Ubuntu 11.10+
If you run into trouble, try
installing one thing at a time, and looking at the 
specific installation function in this script.

options:
 -a: (default) install (A)ll packages - good luck!
 -b: install controller (B)enchmark (oflops)
 -c: (C)lean up after kernel install
 -d: (D)elete some sensitive files from a VM image
 -e: install Mininet d(E)veloper dependencies
 -f: install Open(F)low
 -h: print this (H)elp message
 -i: install (I)ndigo Virtual Switch
 -k: install new (K)ernel
 -m: install Open vSwitch kernel (M)odule from source dir
 -n: install Mini(N)et dependencies + core files
 -p: install (P)OX OpenFlow Controller
 -r: remove existing Open vSwitch packages
 -s <dir>: place dependency (S)ource/build trees in <dir>
 -t: complete o(T)her Mininet VM setup tasks
 -v: install Open (V)switch
 -V <version>: install a particular version of Open (V)switch on Ubuntu
 -w: install OpenFlow (W)ireshark dissector
 -y: install R(y)u Controller
 -x: install NO(X) Classic OpenFlow controller
 -0: (default) -0[fx] installs OpenFlow 1.0 versions
 -3: -3[fx] installs OpenFlow 1.3 versions
```

### **2·使用install的安装脚本**

**2.1·使用install安装minist**

```shell
kazetoyuki@ubuntu:~/mininet/util$ ./install.sh -n3V 2.5.0
```

`在这里出现包的依赖错误`

**2.1.1重新安装python3-pkg-resources**

```powershell
The following packages have unmet dependencies:
 python3-setuptools : Depends: python3-pkg-resources (= 39.0.1-2) but 45.2.0-1 is to be installed
 ssh : Depends: openssh-server (>= 1:7.6p1-4ubuntu0.4)
E: Unable to correct problems, you have held broken packages.
kazetoyuki@ubuntu:~/mininet/util$  sudo apt-get purge python3-pkg-resources
```

**2.1.2重新安装openssh-server**

`在这里出现openssh-server的依赖问题`

```powershell
kazetoyuki@ubuntu:~/mininet/util$ sudo apt-get install openssh-server
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 openssh-server : Depends: openssh-client (= 1:7.6p1-4ubuntu0.4)
                  Depends: openssh-sftp-server but it is not going to be installed
                  Recommends: ssh-import-id but it is not going to be installed
E: Unable to correct problems, you have held broken packages.
```

**2.1.2.1安装openssh-client**

清空一些依赖

```shell
kazetoyuki@ubuntu:~/mininet/util$ sudo apt autoremove
```

重新安装openssh-client

```
kazetoyuki@ubuntu:~/mininet/util$ sudo apt install openssh-client=1:7.6p1-4
```

**2.2解决依赖后重新安装mininet**

```shell
kazetoyuki@ubuntu:~/mininet/util$ ./install.sh -n3V 2.5.0
```

`之后再度报错`

```shell
Setting up cgroup-tools (0.41-8ubuntu2) ...
Processing triggers for libc-bin (2.31-0ubuntu9) ...
Processing triggers for man-db (2.9.1-1) ...
Installing Mininet core
~/mininet ~/mininet/util
cc -Wall -Wextra  -DVERSION=\"`PYTHONPATH=. python3 -B bin/mn --version`\" mnexec.c -o mnexec
2.3.0d6
mnexec.c:17:10: fatal error: stdio.h: No such file or directory
 #include <stdio.h>
          ^~~~~~~~~
compilation terminated.
Makefile:50: recipe for target 'mnexec' failed
make: *** [mnexec] Error 1
kazetoyuki@ubuntu:~/mininet/util$ 
```

这种情况是软件源的问题（本机当时使用的是阿里的ubuntu18的软件源，需要更换为ubuntu20的软件源）

## 安装(ubuntu14.0，安装成功)

配置好相应的阿里源

```
deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse 
deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse 
deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse 
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse 
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse 
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse 
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
```

clone mininet的源码

```shell
git clone git://github.com/mininet/mininet
```

更新后，按照mininet官网上的安装推荐进行安装

```shell
mininet/util/install.sh -a
```

## 安装Ryu

安装Ryu控制器

1. 更新pip，防止安装ryu找不到相应的依赖

```shell
sudo pip install --upgrade pip
```

2. git clone Ryu的源码


```shell
git clone https://github.com/osrg/ryu.git
```

3. <u>ryu路径下</u>，使用如下命令，安装依赖软件

```shell
sudo pip install -r tools/pip-requires 
```

4. <u>ryu路径下</u>，使用如下命令，安装ryu

   ```shell
   sudo python setup.py install
   ```

   此步可能有较多问题：

   1. setuptools 模块未安装

      ```shell
      curl https://bitbucket.org/pypa/setuptools/raw/bootstrap/ez_setup.py | python
      ```

   2. webob>=1.2

      ```shell
      sudo easy_install webob==1.2.3
      ```

   3. the 'routes' distribution was not found ...

      ```shell
      sudo easy_install routes
      ```

   4. The 'oslo.config >= 1.2.0' distribution was not found ...

      ```shell
      sudo easy_install oslo.config==3.0.0
      ```

   5. lxml未安装

      ```shell
      apt-get install libxml2-dev libxslt1-dev python-dev
      
      apt-get install python-lxml
      ```

   6. six版本不足

      ```shell
      pip uninstall six
      pip install six
      ```

      删除不了six时，使用

      ```shell
      sudo pip install thrift --ignore-installed six
      ```

   7. The 'ovs' distribution was not found ....

      ```shell
      sudo pip install -r tools/pip-requires
      ```

   8. Stevedore

      ```shell
      sudo pip install stevedore
      ```

   9. Debtcollector

      ```shell
      sudo pip install debtcollector
      ```

5. 测试安装成功

   ```powershell
   kazetoyuki@ubuntu:~/softwares/ryu$ ryu-manager
   loading app ryu.controller.ofp_handler
   instantiating app ryu.controller.ofp_handler of OFPHandler
   ^Ckazetoyuki@ubuntu:~/softwares/ryu$ 
   ```

## 使用Ryu

先启动ryu管理器，再启用mininet

```powershell
kazetoyuki@ubuntu:~/softwares/ryu/ryu/app$ ryu-manager simple_switch.py
loading app simple_switch.py
loading app ryu.controller.ofp_handler
instantiating app simple_switch.py of SimpleSwitch
instantiating app ryu.controller.ofp_handler of OFPHandler


```

Mininet链接到Ryu

```
sudo mn --controller=remote
```

---

链接到Ryu后，Ryu界面会有很多的数据包

```
kazetoyuki@ubuntu:~/softwares/ryu/ryu/app$ ryu-manager simple_switch.py
loading app simple_switch.py
loading app ryu.controller.ofp_handler
instantiating app simple_switch.py of SimpleSwitch
instantiating app ryu.controller.ofp_handler of OFPHandler
packet in 1 b6:6d:a0:91:66:42 33:33:00:00:00:16 65534
packet in 1 b6:6d:a0:91:66:42 33:33:00:00:00:16 65534
packet in 1 4e:a9:64:21:b7:c9 33:33:ff:21:b7:c9 1
packet in 1 72:69:19:a9:20:33 33:33:ff:a9:20:33 2
packet in 1 b6:6d:a0:91:66:42 33:33:ff:d7:58:59 65534

```



## 使用Mininet

### 文件架构

![image-20210104213042099](.\mininet_install\image-20210104213042099.png)

常用的三个文件夹已经在上图中使用星号进行了标注。

### 网络构建参数

#### 清空拓扑

```shell
sudo mn -c
```

mininet的命令图谱

![image-20210105090924464](.\mininet_install\image-20210105090924464.png)

```shell
sudo mn
```

不加任何命令，直接使用mn，也可以创建一个网络。

#### --topo

```
sudo mn --topo=single,3
```

![image-20210105091449060](.\mininet_install\image-20210105091449060.png)

```
sudo mn --topo=linear,4
```

![image-20210105091646909](.\mininet_install\image-20210105091646909.png)

```
sudo mn --topo=tree,depth=2,fanout=2
```

![image-20210105092044992](.\mininet_install\image-20210105092044992.png)

```
sudo mn --custom file.py --topo mytopo
```

![image-20210105092233312](.\mininet_install\image-20210105092233312.png)

> 使用custom代入file.py时，这里的file.py是一个相对路径，也就是说命令本身需要在custom的路径下执行。
>
> 如果不在custom路径下，必须在使用绝对路径。

#### --switch

![image-20210105092558791](.\mininet_install\image-20210105092558791.png)

用户态和内核态？

#### --controller

定义要使用的控制器，没有指定就使用**默认的控制器**。

```shell
sudo mn --controller=remote,--ip=[controllerIP],--port=[port]
```

#### --mac

自动设置设备的mac地址

```shell
sudo mn --topo=tree,depth=2,fanout=2,--mac
```

### 内部交互命令（查看类）

#### net

查看结点所属的网络

![image-20210106084339870](.\mininet_install\image-20210106084339870.png)

#### intfs

查看网络接口信息

![image-20210106093453217](.\mininet_install\image-20210106093453217.png)

#### links

查看结点的链路结构

![image-20210106084619514](.\mininet_install\image-20210106084619514.png)

#### nodes

查看链路中可以使用的结点

![image-20210106084807997](.\mininet_install\image-20210106084807997.png)

#### ping

单独ping

```
h1 ping h3
```

pingall

查看结点的连通性

![image-20210106084859912](.\mininet_install\image-20210106084859912.png)

与该命令相似的还有"pingpair"：

无论网络中的结点究竟有多少个，都只验证前两个主机的连通性。

#### 其他

```shell
mininet> help
```

在这个命令中，可以查询到许多其他的

### 内部交互命令（使用类）

#### link

开启、禁用某段链路

```
link s1 s2 up
```

#### iperf

两点之间进行tcp的带宽测试

```
iperf h1 h2
```

#### iperfudp

两点之间进行udp的带宽测试（必须跟上 bw）

```
iperfudp bw h1 h2
```

#### dpctl

查看所有交换机上的流表

```
dpctl dump-flows
```

添加流表

（2号端口进入、转发给1号端口）

```
dpctl add-flow in_port=2,actions=output:1
```

（2号端口进入丢弃）

```
dpctl add-flow in_port=2,actions=drop
```

删除全部流表

```
dpctl del-flows
```

删除所有交换机上的部分流表

```
dpctl del-flows in_port=2
```

删除某个交换机上的流表

```
sh ovs-ofctl del-flows s1 in_port=2
```

#### xterm

```
xterm h1
```

开启可视化操作界面

#### py

使用python脚本

```python
py net.addSwitch("s1")
py net.addHost("h1")
py net.addLink("h1","s1")
```

![image-20210106094008639](.\mininet_install\image-20210106094008639.png)


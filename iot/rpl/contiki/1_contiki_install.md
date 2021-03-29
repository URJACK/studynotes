# Contiki

## 介绍 

Contiki 是一个适用于有内存的嵌入式系统的开源的、高可移植的、支持网络的多任务操作系统。包括一个多任务核心、TCP/IP 堆栈、程序集以及低能耗的无线通讯堆栈。Contiki 采用 C 语言开发的非常小型的嵌入式操作系统，运行只需要几K的内存。

## Instant Contiki

<u>Instant contiki其实就是ubuntu</u>，在ubuntu的基础上安装了cooja和msp430的编译环境，可以说instant contiki是一个完善的开发环境，<u>但是并没有安装SDCC，而CC2530正需要SDCC的支持才可以完成编译</u>。

这个Instant Contiki 的镜像的下载路径在：

https://sourceforge.net/projects/contiki/files/Instant%20Contiki/

这里有历代版本的InstantContiki镜像。我下载了**InstantContiki2.7**作为自己的使用系统。

```
需求：
VPN（用来下载镜像）
InstantContiki（本机使用2.7）
Vmware（本机使用16.0）
```

> 这里的下载会比较缓慢，请自行挂梯子

### 基础配置

使用vmware打开镜像后，配置阿里云镜像

```
# 阿里源：
 
deb http://mirrors.aliyun.com/ubuntu/ precise main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ precise-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ precise-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ precise-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ precise-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ precise main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ precise-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ precise-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ precise-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ precise-backports main restricted universe multiverse
```

### SDCC

该镜像中，自带的就安装好了SDCC。

1. 使用命令进行测试sdcc是否安装

```shell
user@instant-contiki:~$ sdcc -v
SDCC : mcs51 3.3.1 #8804 (Aug  6 2013) (Linux)
user@instant-contiki:~$ which sdcc
/usr/local/sdcc/bin/sdcc
```

2. 对CC2530进行编译测试，检测SDCC是否正常工作

```shell
user@instant-contiki:~/contiki/examples/cc2530dk$ make blink-hello
```

---

```shell
Other memory:
   Name             Start    End      Size     Max     
   ---------------- -------- -------- -------- --------
   PAGED EXT. RAM                         0      256   
   EXTERNAL RAM     0x0000   0x0b59    2906     7936   
   ROM/EPROM/FLASH  0x0000   0xafeb   45036    65536   
rm blink-hello.ihx obj_cc2530dk/blink-hello.app.rel
user@instant-contiki:~/contiki/examples/cc2530dk$
```

3. 结果发现运行成功，并在路径下生成了.hex文件

```shell
user@instant-contiki:~/contiki/examples/cc2530dk$ ls
blink-hello.c         blink-hello.mem       Makefile         timer-test.c
blink-hello.cc2530dk  border-router         Makefile.target  udp-ipv6
blink-hello.hex       cc2531-usb-demo       obj_cc2530dk
blink-hello.lk        contiki-cc2530dk.lib  sensors-demo.c
blink-hello.map       hello-world.c         sniffer
```

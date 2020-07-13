# DeepLearning

## 前言

### 预备知识

数学

```
求导、矩阵运算、正态分布
```

技术

```
python
```

###  注意

1. 不要过分追求理论知识

2. 不要过分追求python知识

3. 学习完后及时编码进行反馈

## 概论

### 深度学习的应用

以识别垃圾邮件为例，编程需要给定一个特定的规则，但是深度学习是通过学习，找到一个函数，帮助给定规则。

语音识别、图像识别、天气预报、玩游戏、机器翻译、自动驾驶、黑白图片上色等...

深度学习比机器学习可以具有更少的人为干预。

### 深度学习的框架

封装了函数，方便使用，降低了入门门槛

代码简洁、高效，方便交流

底层被优化，降低运行时间

Tensorflow

```
Google 开源，社区强大
```

PyTorch

```
Facebook 开源，动态图机制、构建灵活的网络结构
```

MxNet

```
Amazon支持，大部分是国人开发，动态图与静态图混合编程
```

## Anaconda

一个可以轻松管理python版本，还可以管理每一个python版本所持有的包的超级管理工具。

安装它之后可以不用安装python，因为任意一个版本的python对它来说都是一个环境。

### 命令

使用之前可以做出一些配置(C:\Users\scffz\\.condarc)

```
channels:
- https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
- https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
- defaults 
show_channel_urls: true
```

以下是常用命令

(不可以在windows默认的命令行程序)，必须使用Anaconda Prompt专属的命令行程序

conda版本管理

```
conda --version  # 查看conda版本
conda update conda # 更新conda版本
```

python环境管理

```
（不用管是3.4.x，conda会为我们自动寻找3.4.x中的最新版本）
conda create --name py34 python=3.4    # 创建环境
conda remove -n py34 --all # 删除环境
conda env list # 查看已经有的环境

# 启用环境
# 激活之前设定的py34环境
activate py34
# 在Linux & Mac中使用source activate激活
source activate py34

# 退出环境
# 在windows环境下使用deactivate
deactivate
# 在Linux & Mac中使用source deactivate
source deactivate
```

包管理(使用包管理命令时，每一个python环境，就有一个独立的包环境)

```
#使用这条命令来查看在当前环境中，已安装的包和对应版本
conda list

#我们可以通过search命令检查pandas这个包是否可以通过conda来安装
conda search pandas 

#通过install安装pandas
conda install pandas 

#通过update更新pandas
conda update pandas

#通过remove卸载pandas
conda remove pandas
```

python环境更新

```
# 例如我们所启用的环境是py34，使用的是python3.4,那么conda会将python升级为3.4.x系列中的最新版本
conda update python 
```

## CUDA、tensorFlow安装

CUDA

```
CUDA 10.0 下载下来的是“安装包”的安装包，我们此处简称“包包”

使用“包包”，选择一个文件夹，作为暂存目录，会在该暂存目录下安装下“安装包”。

使用“安装包”，安装CUDA

CUDA安装时，选择自定义安装。记得取消安装“集成的vs”。
```

cuDNN

```
cuDNN v7.6.5 (November 5th, 2019), for CUDA 10.0
- cuDNN Library for Windows 10

解压完之后，cuDnn路径下的bin,include,lib三个文件夹的所有内容
全部复制到cuda路径下即可，因为cuda具有三个同名目录。复制过程不会有重复文件，因为这是功能拓展
```

pip

```
使用Anaconda进入python环境后，环境有自己的pip。
我们更改pip的国内源（清华）
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

会得到回显结果
Writing to C:\Users\scffz\AppData\Roaming\pip\pip.ini
```

```
# 或：
# 阿里源
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/
# 腾讯源
pip config set global.index-url http://mirrors.cloud.tencent.com/pypi/simple
# 豆瓣源
pip config set global.index-url http://pypi.douban.com/simple/
```

TensorFlow

```
pip install tensorflow-gpu==1.13.1
pip install matplotlib==2.2.2
```

测试代码

```
import tensorflow as tf

A = tf.constant([[1, 2], [3, 4]])
B = tf.constant([[5, 6], [7, 8]])
C = tf.matmul(A, B)

print(C)
```

## Pycharm 安装

安装pycharm才能愉快的玩耍，不然命令行敲代码太难顶了

pycharm的配置过程中，配置过程大概分为：

1. 配置解释器

2. 配置Configure

### 配置解释器

其中配置解释器，需要注意，我们选择

```
settings->Project: demo->Project Interpreter->...
```

在这个Interpreter中，我们需要添加虚拟环境当中的interpreter。

```
点击Interpreter旁边的设置按钮->Add->Virtualenv Environment->Existing environment->...
```

我添加的地址就是D:\software\anaconda\envs\py37\python.exe

一定要选虚拟环境，pycharm在启动的时候，才会用虚拟环境去启动这个python

### 配置Configure

运行python文件，需要自己事先配置一下

与idea一样，找到右上角->添加一个python的运行->右边的Script path 添加上绝对路径(D:\Storage\pycharmProjects\demo\main.py)

如果之前配好了**解释器**，现在应该不用再配解释器了，因为那个信息应该会默认采用过来，但是保险起见，configure中的解释器必须也一致和之前配置的解释器一致

## 额外模块的安装

### slim

slim库，在tensorFlow的高版本中以及tensorflow2中，已经不在官方库中了，使用其中的函数需要单独安装slim

```shell
pip install tf_slim
```

```python
import tf_slim as slim
```

### future

```
from __future__ import absolute_import
from __future__ import division
from __future__ import print_function
```

```
如果你在main.py中写import string,那么在Python 2.4或之前, Python会先查找当前目录下有没有string.py, 若找到了，则引入该模块，然后你在main.py中可以直接用string了。如果你是真的想用同目录下的string.py那就好，但是如果你是想用系统自带的标准string.py呢？那其实没有什么好的简洁的方式可以忽略掉同目录的string.py而引入系统自带的标准string.py。这时候你就需要from __future__ import absolute_import了。这样，你就可以用import string来引入系统的标准string.py, 而用from pkg import string来引入当前目录下的string.py了
```

```
导入python未来支持的语言特征division(精确除法)，当我们没有在程序中导入该特征时，"/"操作符执行的是截断除法(Truncating Division),当我们导入精确除法之后，"/"执行的是精确除法。
>>> 3/4
0
>>> from __future__ import division
>>> 3/4
0.75
导入精确除法后，若要执行截断除法，可以使用"//"操作符：
--------------------------------------------------------------------------------------------
>>> 3//4
0
>>> 
```

```
from __future__ import print_function
在开头加上这句之后，即使在python2.X，使用print就得像python3.X那样加括号使用。python2.X中print不需要括号，而在python3.X中则需要。
```


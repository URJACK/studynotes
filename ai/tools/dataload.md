# 数据读入

## pandas

### 导入依赖

```
import pandas
```

### 读取pandas

```python
if __name__ == '__main__':
    source_file = "D:\\Storage\\datasets\\kdd\\kddcup.data_10_percent_corrected"
    df = pandas.read_csv(source_file)
    print(df)
```

### 常规操作

#### 查看数据集大小

```python
print(df.shape)
```

#### 查看数据集的类型

```python
print(df.dtypes)
```

---

```
0            int64
tcp         object
http        object
SF          object
181          int64
5450         int64
0.1          int64
0.2          int64
0.3          int64
0.4          int64
0.5          int64
1            int64
0.6          int64
0.7          int64
0.8          int64
0.9          int64
0.10         int64
0.11         int64
0.12         int64
0.13         int64
0.14         int64
0.15         int64
8            int64
8.1          int64
0.00       float64
0.00.1     float64
0.00.2     float64
0.00.3     float64
1.00       float64
0.00.4     float64
0.00.5     float64
9            int64
9.1          int64
1.00.1     float64
0.00.6     float64
0.11.1     float64
0.00.7     float64
0.00.8     float64
0.00.9     float64
0.00.10    float64
0.00.11    float64
normal.     object
dtype: object
```

#### 读取前多少行

```python
print(df.head(10))
```

#### 查看是否有缺失值

```python
print(df.isnull().sum())
```

---

```
0          0
tcp        0
http       0
SF         0
181        0
5450       0
0.1        0
0.2        0
0.3        0
0.4        0
0.5        0
1          0
0.6        0
0.7        0
0.8        0
0.9        0
0.10       0
0.11       0
0.12       0
0.13       0
0.14       0
0.15       0
8          0
8.1        0
0.00       0
0.00.1     0
0.00.2     0
0.00.3     0
1.00       0
0.00.4     0
0.00.5     0
9          0
9.1        0
1.00.1     0
0.00.6     0
0.11.1     0
0.00.7     0
0.00.8     0
0.00.9     0
0.00.10    0
0.00.11    0
normal.    0
dtype: int64
```

#### 统计

```python
print(df.describe(include="all"))
```

#### 查看个数和数据类型信息

```python
print(df.info())
```

---

```
 #   Column   Non-Null Count   Dtype  
---  ------   --------------   -----  
 0   0        494020 non-null  int64  
 1   tcp      494020 non-null  object 
 2   http     494020 non-null  object 
 3   SF       494020 non-null  object 
 4   181      494020 non-null  int64  
 5   5450     494020 non-null  int64  
 6   0.1      494020 non-null  int64  
 7   0.2      494020 non-null  int64  
 8   0.3      494020 non-null  int64  
 9   0.4      494020 non-null  int64  
 10  0.5      494020 non-null  int64  
 11  1        494020 non-null  int64  
 12  0.6      494020 non-null  int64  
 13  0.7      494020 non-null  int64  
 14  0.8      494020 non-null  int64  
 15  0.9      494020 non-null  int64  
 16  0.10     494020 non-null  int64  
 17  0.11     494020 non-null  int64  
 18  0.12     494020 non-null  int64  
 19  0.13     494020 non-null  int64  
 20  0.14     494020 non-null  int64  
 21  0.15     494020 non-null  int64  
 22  8        494020 non-null  int64  
 23  8.1      494020 non-null  int64  
 24  0.00     494020 non-null  float64
 25  0.00.1   494020 non-null  float64
 26  0.00.2   494020 non-null  float64
 27  0.00.3   494020 non-null  float64
 28  1.00     494020 non-null  float64
 29  0.00.4   494020 non-null  float64
 30  0.00.5   494020 non-null  float64
 31  9        494020 non-null  int64  
 32  9.1      494020 non-null  int64  
 33  1.00.1   494020 non-null  float64
 34  0.00.6   494020 non-null  float64
 35  0.11.1   494020 non-null  float64
 36  0.00.7   494020 non-null  float64
 37  0.00.8   494020 non-null  float64
 38  0.00.9   494020 non-null  float64
 39  0.00.10  494020 non-null  float64
 40  0.00.11  494020 non-null  float64
 41  normal.  494020 non-null  object 
```

#### 查看分类型特征中的具体分类

可以查看分类型属性的具体取值

```python
print("tcp", df['tcp'].unique())
```

---

```
tcp ['tcp' 'udp' 'icmp']
```

因为最终结果也属于分类，所以最终结果也同样适用

```python
print("normal.", df['normal.'].unique())
```

---

```
normal. ['normal.' 'buffer_overflow.' 'loadmodule.' 'perl.' 'neptune.' 'smurf.'
 'guess_passwd.' 'pod.' 'teardrop.' 'portsweep.' 'ipsweep.' 'land.'
 'ftp_write.' 'back.' 'imap.' 'satan.' 'phf.' 'nmap.' 'multihop.'
 'warezmaster.' 'warezclient.' 'spy.' 'rootkit.']
```

## csv文件

### 使用csv库

```python
import csv

if __name__ == '__main__':
    source_file = "D:\\Storage\\datasets\\kdd\\kddcup.data_10_percent_corrected"
    with open(source_file, 'r') as data_source:
        count = 0
        csv_reader = csv.reader(data_source)
        for row in csv_reader:
            count = count + 1
            print(row)
```

### 使用pandas

```python
if __name__ == '__main__':
    source_file = "D:\\Storage\\datasets\\kdd\\kddcup.data_10_percent_corrected"
    df = pandas.read_csv(source_file)
    print(df)
```

## dpkt

必须是.pcap格式，不可以是.pcapng

此处使用的f打开文件的'rb'，似乎还有其他的参数

```python
import dpkt


def main():
    f = open('D:\\Storage\\wiresharkSave\\ahahaha.pcap', 'rb')
    readBuffer = dpkt.pcap.Reader(f)
    record = 0
    f
    for ts, buf in readBuffer:
        eth = dpkt.ethernet.Ethernet(buf)
        ip = eth.data
        tcp = ip.data
        print(ip)
        print(tcp.dport)
        if record == 0:
            record = ts
        else:
            ts_record = ts - record
            record = ts
            print(ts_record)
        print("==========================================")


if __name__ == '__main__':
    main()
```

从若干个数据包中，读取相同的源IP地址和目的IP地址。

```python
import dpkt
import numpy as np

SPAN_ALPHA = 0.4


def main():
    f = open('D:\\Storage\\wiresharkSave\\demo1.pcap', 'rb')
    readBuffer = dpkt.pcap.Reader(f)
    spanMap = {}
    timeMap = {}
    for ts, buf in readBuffer:
        eth = dpkt.ethernet.Ethernet(buf)
        ip = eth.data
        if not hasattr(ip, 'tcp'):
            continue
        bufferStrA = ""
        bufferStrB = ""
        keyStr = ""
        for data in ip.src:
            bufferStrA += str(data)
        for data in ip.dst:
            bufferStrA += str(data)
        for data in ip.dst:
            bufferStrB += str(data)
        for data in ip.src:
            bufferStrB += str(data)
        # strA 源IP 到 目的IP
        # strB 目的IP 到 源IP

        if bufferStrB > bufferStrA:
            keyStr = bufferStrB
        else:
            keyStr = bufferStrA

        if timeMap.get(keyStr) is None:
            timeMap[keyStr] = ts
            # 第一次，默认产生一个流量
            getSpanList(keyStr, spanMap, True)
        else:
            # 两个 具有相同 源IP地址 和 目的IP地址的 TCP包，的时间差
            span = ts - timeMap.get(keyStr)
            if span > SPAN_ALPHA:
                # 时间差太大了 本次span不作为计算 并且应当产生一个新的流量
                timeMap[keyStr] = ts
                getSpanList(keyStr, spanMap, True)
            else:
                # 时间差在SPAN_ALPHA内 当前“流量集”的最新“流量”需要添加一个新的“时间差”
                spanList = getSpanList(keyStr, spanMap)
                spanList.append(span)
    print(spanMap)


# 通过keyStr从spanMap中取得“流量集”
# 如果adder为True，那么意味着“流量集”需要添加一个新的“流量”
#         为False，那么意味着当前，从“流量集”中取得当前最新的“流量”
def getSpanList(keyStr, spanMap, adder=False):
    spanListList = spanMap.get(keyStr)
    if spanListList is None:
        spanListList = []
        spanMap[keyStr] = spanListList
    spanList: list
    if adder:
        spanListList.append([])
    spanList = spanListList[len(spanListList) - 1]
    return spanList


if __name__ == '__main__':
    main()

```


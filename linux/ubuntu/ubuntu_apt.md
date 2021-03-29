# Ubuntu-apt

## 原理



## 系统源

1`源文件备份

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

2`编辑源文件列表

```shell
sudo gedit /etc/apt/sources.list
```

3`常见的源

阿里云给ubuntu20的源

```
#阿里源
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
```

阿里云的源

这个bionic是可以使用的

```
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```

vivid这个似乎用不了

```
 deb http://mirrors.aliyun.com/ubuntu/ vivid main restricted universe multiverse
 deb http://mirrors.aliyun.com/ubuntu/ vivid-security main restricted universe multiverse
 deb http://mirrors.aliyun.com/ubuntu/ vivid-updates main restricted universe multiverse
 deb http://mirrors.aliyun.com/ubuntu/ vivid-proposed main restricted universe multiverse
 deb http://mirrors.aliyun.com/ubuntu/ vivid-backports main restricted universe multiverse
 deb-src http://mirrors.aliyun.com/ubuntu/ vivid main restricted universe multiverse
 deb-src http://mirrors.aliyun.com/ubuntu/ vivid-security main restricted universe multiverse
 deb-src http://mirrors.aliyun.com/ubuntu/ vivid-updates main restricted universe multiverse
 deb-src http://mirrors.aliyun.com/ubuntu/ vivid-proposed main restricted universe multiverse
 deb-src http://mirrors.aliyun.com/ubuntu/ vivid-backports main restricted universe multiverse
```

网易源

```
deb http://mirrors.163.com/ubuntu/ precise main universe restricted multiverse 
deb-src http://mirrors.163.com/ubuntu/ precise main universe restricted multiverse 
deb http://mirrors.163.com/ubuntu/ precise-security universe main multiverse restricted 
deb-src http://mirrors.163.com/ubuntu/ precise-security universe main multiverse restricted 
deb http://mirrors.163.com/ubuntu/ precise-updates universe main multiverse restricted 
deb http://mirrors.163.com/ubuntu/ precise-proposed universe main multiverse restricted 
deb-src http://mirrors.163.com/ubuntu/ precise-proposed universe main multiverse restricted 
deb http://mirrors.163.com/ubuntu/ precise-backports universe main multiverse restricted 
deb-src http://mirrors.163.com/ubuntu/ precise-backports universe main multiverse restricted 
deb-src http://mirrors.163.com/ubuntu/ precise-updates universe main multiverse restricted
```

ubuntu的源， 最好也加上，避免某些库下载不到

```
 deb http://archive.ubuntu.com/ubuntu/ trusty main restricted universe multiverse
 deb http://archive.ubuntu.com/ubuntu/ trusty-security main restricted universe multiverse
 deb http://archive.ubuntu.com/ubuntu/ trusty-updates main restricted universe multiverse
 deb http://archive.ubuntu.com/ubuntu/ trusty-proposed main restricted universe multiverse
 deb http://archive.ubuntu.com/ubuntu/ trusty-backports main restricted universe multiverse
```

4`sudo apt-get update

遇见问题：E:Could not get lock /var/lib/apt/lists/lock - open (11: Resource temporarily unavailable)

```
sudo rm /var/cache/apt/archives/lock
sudo rm /var/lib/dpkg/lock
```

5`sudo apt-get upgrade


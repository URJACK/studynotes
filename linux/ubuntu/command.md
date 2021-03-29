# 系统命令

## 进程

### 查看所有进程

```shell
kazetoyuki@ubuntu:~/Desktop$ ps
    PID TTY          TIME CMD
   2128 pts/0    00:00:00 bash
   2370 pts/0    00:00:00 ps
kazetoyuki@ubuntu:~/Desktop$ ps -all
F S   UID     PID    PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 R  1000    1443    1441  0  80   0 - 63087 -      tty2     00:00:02 Xorg
0 S  1000    1461    1441  0  80   0 - 49895 poll_s tty2     00:00:00 gnome-ses
0 R  1000    2371    2128  0  80   0 -  5007 -      pts/0    00:00:00 ps
```

### 查看是否有相应进程

```shell
kazetoyuki@ubuntu:~/Desktop$ ps -ef | grep apt-get    
kazetoy+    2373    2128  0 07:49 pts/0    00:00:00 grep --color=auto apt-get
```

### 强制关闭相应进程

```shell
kazetoyuki@ubuntu:~/Desktop$ sudo killall -9 apt-get 
apt-get: no process found
```


# Python Socket

## 使用样例

### Server.py

```python
import socket as sk


def main():
    tcp_server_socket = sk.socket(sk.AF_INET, sk.SOCK_STREAM)
    address = ('', 7788)
    tcp_server_socket.bind(address)
    tcp_server_socket.listen(128)
    client_socket, client_addr = tcp_server_socket.accept()
    recv_data = client_socket.recv(1024)
    print("Received data:", recv_data.decode("utf-8"))
    client_socket.send("thank you!".encode("utf-8"))
    client_socket.close()
    pass


if __name__ == '__main__':
    main()

```

### Client.py

```python
import socket as sk


def main():
    server_ip = input("Please input the server's ip Address:")
    server_port = 7788
    tcp_client_socket = sk.socket(sk.AF_INET, sk.SOCK_STREAM)
    tcp_client_socket.connect((server_ip, server_port))
    tcp_client_socket.send("Hello body".encode("utf-8"))
    tcp_client_socket.close()
    pass


if __name__ == '__main__':
    main()

```

## 语句分析

### A·创建套接字

```python
# tcp的客户端 创建的套接字
tcp_client_socket = sk.socket(sk.AF_INET, sk.SOCK_STREAM)
# tcp的服务端 创建的套接字
tcp_server_socket = sk.socket(sk.AF_INET, sk.SOCK_STREAM)
```

我们发现他们二者创建套接字调用的函数、以及传递的参数，均完全相等。
这说明，python中的socket包，通过这种方式创建一个**插座，socket**，作为<u>程序和TCP/UDP协议连接的接口</u>。

#### 传递参数

这里需要注意，socket函数中的两个参数：

1. Address Family，地址族：
   1. `AF_INET`（用于 Internet 进程间通信） 
   2. `AF_UNIX`（用于同一台机器进程间通信）
2. Type，套接字类型：
   1. `SOCKET_STREAM`（流式套接字，主要用于 TCP 协议）
   2. `SOCKET_DGRAM`（数据报套接字，主要用于 UDP 协议）

#### 可能抛出的异常

```python
try:
    #create an AF_INET, STREAM socket (TCP)
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
except socket.error, msg:
    print 'Failed to create socket. Error code: ' + str(msg[0]) + ' , Error message : ' + msg[1]
    sys.exit();
 
print 'Socket Created'
```

### B·绑定套接字 (server)

```python
# 绑定地址 * ; 绑定端口 7788
address = ('', 7788)
tcp_server_socket.bind(address)
```
绑定套接字，相当于可以让这个插座去监听网口发来的数据。
这样网卡读取到符合要求的数据，就会分发给这个socket。

#### 传递参数

bind函数中，传递的是一个**元组参数**！

该元组中，包含两个元素：

1. HOST，地址：添加我们需要绑定的地址，这样只会接受到来自特定IP的数据报。当然也可以选择**空地址**。显然这样接受全部地址的数据报，更符合我们平时的需求。
2. PORT，端口：添加我们需要绑定的端口。

### C·监听连接 (server)

这个函数可以让socket处于监听模式。

```python
# 创建128个连接个数的监听
tcp_server_socket.listen(128)
```

如果有 128 个连接正在等待处理，此时第 129 个请求过来时将会被拒绝。

### D·接受链接 (server)

该函数一般是通过服务器调用。

```python
# 建立新的连接 重新分配新的socket
client_socket, client_addr = tcp_server_socket.accept()
```

#### 返回参数

返回两个参数：

1. client_socket，新建立的客户端套接字：每个客户端与本文监听的socket相连接后，就会创建一个client_socket。
2. client_addr，新建立的客户端，对方的ip地址和端口信息，是一个**元组数据**，数值类型如右：`('127.0.0.1', 50318)`所示。

### E·接受数据

套接字函数 `recv` 可以用来接收 socket 的数据：

```python
# tcp_server_socket = sk.socket(sk.AF_INET, sk.SOCK_STREAM)
recv_data = client_socket.recv(1024)
```

#### 编码转换

实际上，接受到的数据是有编码的，所以我们在这里也使用编码对其进行转换。

```python
print("Received data:", recv_data.decode("utf-8"))
```

#### 接受为空时候

```python
while True:
    data = client_socket.recv(4096)
    # 如果接受到数据为空 此时必定说明socket已经关闭 并且会一直不断返回
    # 当socket未关闭的时候 recv会是一个阻塞效果
    if len(data) == 0:
        break
        print(data)
```

### F·连接服务器 (client)

连接服务器

```python
# tcp_client_socket = sk.socket(sk.AF_INET, sk.SOCK_STREAM)
tcp_client_socket.connect((server_ip, server_port))
```

### G·发送数据

python的发送数据

```python
client_socket.send("thank you!".encode("utf-8"))
```

可以通过这种方式对原始文本进行编码。

## 用例编写

### 服务器A

这个服务器可以同时监听多个socket 但是却无法做到同时服务多个socket！

比如启动这个服务器后，我同时启动多个client对它进行访问。
它只会优先处理第一个client的内容。

```python
import socket as sk
import _thread
from tools import socket_host

LISTEN_SIZE = 5

def main():
    host_socket = sk.socket(sk.AF_INET, sk.SOCK_STREAM)
    host_socket.bind(('', 6666))
    host_socket.listen(LISTEN_SIZE)
    while True:
        client_socket, client_addr = host_socket.accept()
        while True:
            data = client_socket.recv(4096)
            # 如果接受到数据为空 此时必定说明socket已经关闭 并且会一直不断返回
            # 当socket未关闭的时候 recv会是一个阻塞效果
            if len(data) == 0:
                break
                print(data)
                
                
if __name__ == '__main__':
	main()

```

### 服务器B  (加了线程模块)

使用了线程模块，

```python
import socket as sk
import _thread
from tools import socket_host

LISTEN_SIZE = 5


def main():
    host_socket = sk.socket(sk.AF_INET, sk.SOCK_STREAM)
    host_socket.bind(('', 6666))
    host_socket.listen(LISTEN_SIZE)
    while True:
        client_socket, client_addr = host_socket.accept()
        # 第二个参数必须传元组 如果是“单元素元组” 必须像下面这样，末尾加上一个小逗号
        _thread.start_new_thread(listen_thread, (client_socket,))


def listen_thread(client_socket):
    while True:
        data = client_socket.recv(4096)
        if len(data) == 0:
            break
        print(data)


if __name__ == '__main__':
    main()

```


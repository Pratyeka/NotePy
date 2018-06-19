### Socket介绍
- Python提供可Socket标准库，是非常底层的接口库
- Socket是一种通用的网络编程接口，和网络层次没有一一对应的关系
- 协议族(socket()的第一个参数)：
    - `AF_INET`: IPV4 (默认)
    - `AF_INET6`: IPV6
    - `AF_UNIX`: 
- Socket类型：
    - `SOCK_STREAM`: 面向连接的流套接字, 默认值，TCP协议
    - `SOCK_DGRAM`:  无连接的数据报文套接字，UDP协议

### TCP编程
- TCP的Socket编程，包括服务端和客户端两部分

#### TCP服务器端
- TCP服务端编程步骤：
    1. 创建socket对象（Socket.socket()）
    2. 绑定IP地址和端口号（bind((add,port))）
        - 要求参数是一个元祖
        - 还没有占用端口
    3. 开始监听（listen()）
        - 正式占用端口
    4. 获取用来传送数据的socket对象（accept()）
        - 返回值是一个socket对象*rsocket*和raddr组成的元祖
        - raddr是客户端的ip地址个端口号组成的元祖
        - accept()会一直阻塞到客户端连接
    5. 接受数据（rsocket.recv(1024)）
        - 使用4中新建的用于传送数据的socket对象
        - 必须设置缓冲区大小，建议1024/4096
        - 返回值为data（bytes类型数据）
    6. 发送数据（rsocket.send(bytes)）
        - 发送数据要求是bytes类型
- *注：两次绑定统一端口会抛异常*

- 服务器端基本代码
```Python
    import socket

    servicer = socket.socket()
    servicer.bind(('127.0.0.1', 12345))
    servicer.listen()
    s1,raddr = servicer.accept()

    while True:
        data = s1.recv(1024)
        print(data)

        s1.send(b'I am OK, are u OK')

    s1.close()
    servicer.close()
```
- 因为accept()/recv()都是阻塞函数，会阻塞主线程导致无法继续执行，多线程解决

#### TCP客户端
- TCP客户端编程步骤：
    1. 创建socket对象（socket.socket()）
    2. 与服务器端连接（client.connect((ip,port))）
        - 要求参数是一个元祖
    3. 给服务器端发送数据（client.send(bytes)）
    4. 接受从客户端的数据（(data = client.recv(1024))）
        - 要求必须提供缓冲区大小的参数
        - 返回值是bytes类型的数据
    5. 关闭连接，释放资源（client.close()）

- TCP客户端基本代码
```Python
    import socket
    import logging

    FORMAT = '%(asctime)s %(threadName)s %(message)s'
    logging.basicConfig(format=FORMAT, level=logging.INFO)


    client = socket.socket()
    client.connect(('172.16.101.126', 10008))

    while True:
        data = client.recv(1024)
        logging.info(data)
        client.send(data)
        if data == b'quit':
            break

    logging.info('cs disconnected')
```

### 群聊软件
- 服务器端
```Python
    import threading
    import socket
    import logging

    FORMAT = '%(asctime)s %(threadName)s %(message)s'
    logging.basicConfig(format=FORMAT, level=logging.INFO)

    class User:
        def __init__(self, ip, port):
            self.addr = (ip, port)
            self.socket = socket.socket()
            self.socket.bind(self.addr)
            self.ssR = {}

        def load(self):
            self.socket.listen()
            threading.Thread(target=self._accept, args=(self.socket,), name=self.addr[0]).start()

        def _accept(self, servicer):
            while True:
                ss, addr = servicer.accept()
                self.ssR[addr] = ss
                threading.Thread(target=self._recv, args=(ss,), name=addr[0]).start()

        def _recv(self, servicer):
            while True:
                try:
                    data = servicer.recv(1024)
                    logging.info(data.decode())
                    for ss in self.ssR.values():
                        ss.send(('recv the message {}'.format(data.decode()).encode()))
                except ConnectionAbortedError:
                    self.ssR.pop(servicer.getpeername())
                    servicer.close()
                    logging.error('disconnect from the servicer')
                    break

        def exit(self):
            for ss in self.ssR.values():
                ss.close()
            self.socket.close()

    xm = User('127.0.0.1', 10001)
    xm.load()
```

- 客户端
```Python
    import socket
    import threading
    import logging

    FORMAT = '%(asctime)s %(threadName)s %(message)s'
    logging.basicConfig(format=FORMAT, level=logging.INFO)


    class ClienTcp:
        def __init__(self, rip, rport):
            self.addr = (rip, rport)
            self.client = socket.socket()
            self.event = threading.Event()

        def start(self):
            self.client.connect(self.addr)
            threading.Thread(target=self._recv, name='client').start()

        def _recv(self):
            while not self.event.is_set():
                try:
                    data = self.client.recv(1024)
                    logging.info(data)
                    self.send('ack' + data.decode())
                except ConnectionAbortedError:
                    break

        def send(self, msg):
            self.client.send(msg.encode())

        def stop(self):
            self.client.close()
            self.event.set()

    if __name__ == '__main__':
        user = ClienTcp('172.16.101.126',10008)
        user.start()
        while True:
            msg = input('>>>>')
            if msg == 'quit':
                user.stop()
                break
            user.send(msg)
        print('end connect')
```

### socket对象转文件对象
- socket->文件对象：
    - 基本语法：`socket.makefile(self, mode="r", buffering=None, *,encoding=None, errors=None, newline=None)`
    - 创建一个与套接字关联的文件对象
    - recv等价于读方法， send等价于写方法
    - *注：将bytes字符转成可对应的字符操作，方便*

- socket函数
    - `getpeername()`: 返回套接字的远程地址
        - 是ip地址和端口组成的元祖
    - `getsockname()`: 返回套接字自己的地址
        - 是ip地址和端口组成的元祖
    - `setblocking(flag)`: 设置socket为阻塞与否
        - flag为0：设置为非阻塞模式，
            - recv没有数据或者send无法立即发送，抛出socket.error异常
        - flag为1：设置为阻塞模式（默认模式）
    - `settimeout(value)`: 设置超时时间
    - `setsocketopt()`: 设置socket参数
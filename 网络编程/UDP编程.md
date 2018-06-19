### UDP编程
- UDP的Socket编程分为服务器端和客户端实现两部分

#### UDP服务器端
- UDP服务器端代码实现步骤：
    1. 创建socket对象（socket.socket(type=SOCK.DRGAM)）
        - 默认使用的是TCP协议，修改为UDP协议
    2. 绑定服务器端的IP地址、port端口（servicer.bind((ip,port))）
        - 要求输入参数是ip和port组成的元祖
        - 与TCP协议不同：bind函数已经完成端口的占用
    3. 接受客户端消息（servicer.recv(1024)）
        - 要求必须输入缓冲器的大小参数
        - 返回值是bytes类型数据
        - 使用recvfrom可以得到，bytes数据以及客户端的地址的元祖
    4. 给客户端发送消息（servicer.send(bytes)）
        - 报错，提示没有目的地址（可以使用connect函数添加目的地址属性）
            - 一般不建议在服务器端使用connect函数，会出错
        - 可以使用servicer.sendto(bytes, raddr)的方式完成（建议）
    5. 断开连接，清理资源

- UDP服务器端基本代码实现：
```Python
    import socket
    import logging

    FORMAT = '%(asctime)s %(threadName)s %(message)s'
    logging.basicConfig(format=FORMAT, level=logging.INFO)


    servicer = socket.socket(type=socket.SOCK_DGRAM)
    addr = ('127.0.0.1', 10003)
    servicer.bind(addr)

    data, raddr = servicer.recvfrom(1024)
    servicer.sendto(b'ack' + data, raddr)
    servicer.close()
```

#### UDP客户端
- UDP客户端实现基本步骤：
    1. 创建socket对象（socket.socket(type=SOCK_DRGAM)）
    2. 向服务器端发送数据（user.sendto(bytes, (ip,addr))）
        - 可以使用connect(addr)来绑定远端地址，使用send来发送
    3. 从服务器端接受数据（user.recv(1024)）
        - 返回值为data数据
        - 可以使用recvfrom(1024),返回数据以及服务器端地址
    4. 断开连接，释放资源

```Python
    import socket
    import logging

    FORMAT = '%(asctime)s %(threadName)s %(message)s'
    logging.basicConfig(format=FORMAT, level=logging.INFO)


    user = socket.socket(type=socket.SOCK_DGRAM)
    raddr = ('127.0.0.1', 8888)

    # user.connect(raddr)
    # data = user.recv(1024)
    # logging.info(data)
    # user.send(b'ack' + data)

    user.sendto(b'msg', raddr)
    data = user.recv(1024)
    logging.info(data)
    user.close()
```

### UDP版本群聊实现
- UDP 服务器端
```Python
    import socket
    import threading
    import datetime
    import logging

    FORMAT = '%(asctime)s %(threadName)s %(message)s'
    logging.basicConfig(format=FORMAT, level=logging.INFO)

    class ChatS:
        def __init__(self, ip, port, inval):
            self.addr = (ip, port)
            self.servicer = socket.socket(type=socket.SOCK_DGRAM)
            self.event = threading.Event()
            self.raddrs = {}
            self.inval = inval

        def start(self):
            self.servicer.bind(self.addr)
            threading.Thread(target=self._recv, name='servicer').start()

        def _recv(self):
            while not self.event.is_set():
                current = datetime.datetime.now().timestamp()
                data, raddr = self.servicer.recvfrom(1024)
                if data == 'HA':
                    self.raddrs[raddr] = current
                    continue
                elif data == b'quit':
                    self.raddrs.pop(raddr)
                    continue
                self.raddrs[raddr] = current
                self._send(b'ACK' + data, current)

        def _send(self, msg, current):
            localset = set()
            for raddr,time in self.raddrs.items():
                if current - time > self.inval:
                    localset.add(raddr)
                    continue
                self.servicer.sendto(msg, raddr)
            for raddr in localset:
                self.raddrs.pop(raddr)

        def stop(self):
            self.servicer.close()
            self.event.set()
            self.raddrs.clear()

    if __name__ == '__main__':
        chat_servicer = ChatS('127.0.0.1', 10007)
        chat_servicer.start()
```

- UDP 客户端
```Python
    import threading
    import socket
    import logging

    FORMAT = '%(asctime)s %(threadName)s %(message)s'
    logging.basicConfig(format=FORMAT, level=logging.INFO)

    class ChatC:
        def __init__(self, rip, raddr):
            self.addr = (rip, raddr)
            self.client = socket.socket(type=socket.SOCK_DGRAM)
            self.event = threading.Event()

        def start(self):
            self.client.bind(self.addr)
            threading.Thread(target=self._recv, name='user').start()
            threading.Thread(target=self.hb, name='hb').start()

        def _recv(self):
            while not self.event.is_set():
                data, raddr = self.client.recvfrom(1024)
                logging.info(data)

        def hb(self, interval):
            while not self.event.wait(interval):
                self.client.sendto('HB', self.addr)

        def send(self, msg):
            self.client.sendto(msg, self.addr)

        def stop(self):
            self.client.close()

    if __name__ == '__main__':
        user = ChatC('127.0.0.1', 10008)
        user.start()
```

### 心跳机制
- 心跳：一端向另外一端定时发送信号来确定是否还在提供服务
    - 一般要求心跳数据包越小越好，防止抢占带宽资源
    - 一般由客户端发起
        - 服务器端收到心跳包可以直接确定客户端活动，且不需要返回消息
        - 比服务器端发送心跳包更加高效

### UDP与TCP的对比
- UDP使用假设：网络环境好，不丢包，不乱序（理想态）
- UDP的效率高于TCP，但是TCP的安全性高于UDP
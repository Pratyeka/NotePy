### sockerserver模块
- 是对socket底层库的进一步封装，高级网络编程框架，*加速网络服务器的开发*
- 相比于socket库，更加健壮、方便

### 类的继承关系
- 常用的两个同步类：
    - `TCPServer`：TCP同步类
    - `UCPServer`: UDP同步类
- 常用的两个异步类：
    - `ForkingMixIn`: 进程同步MixIn类
    - `ThreadingMixIn`: 线程同步MixIn类
- 多线程异步类：
    - `ThreadingUDPServer`: 多线程UDP类
    - `ThreadingTCPServer`: 多线程TCP类

### 编程接口
- sockerserver类编程步骤：
    1. 从BaseRequestHandler类派生子类MM，并实现handle方法
    2. 实例化一个服务器子类，并传入服务器地址和MM类
    3. 调用服务器的server.serve_forever()方法
    4. 调用服务器的close()方法来释放资源

- 基本代码格式如下：
```Python
    import socketserver

    class MyTCPHandler(socketserver.BaseRequestHandler):

        # 可以使用的实例属性有：
        # 连接客户端的socket对象：self.request
        # server端的地址：self.server
        # 客户端地址：self.client_address
        def handle(self):
            # self.request is the TCP socket connected to the client
            self.data = self.request.recv(1024).strip()
            print("{} wrote:".format(self.client_address[0]))
            print(self.data)
            # just send back the same data, but upper-cased
            self.request.sendall(self.data.upper())

    if __name__ == "__main__":
        HOST, PORT = "localhost", 9999

        # Create the server, binding to localhost on port 9999
        with socketserver.TCPServer((HOST, PORT), MyTCPHandler) as server:
            # Activate the server; this will keep running until you
            # interrupt the program with Ctrl-C
            server.serve_forever()
```

- **MyTCPHandler类解释**
    - `MyTCPHandler`类必须是`BaseRequestHandler`类的子类
    - `MyTCPHandler`被初始化时，送入三个参数：request，client_address，server自身
    - 上面三个参数都会是`MyTCPHandler`的属性
        - `self.request`: 是和客户端连接的socket对象，与accept返回值一样
            - 每个不同的连接请求过来后，生成这个连接的socket对象：self.request
        - `self.server`: 是TCPServer自身
        - `self.client_address`: 是客户端地址
    - `MyTCPHandler`类有三个方法，在类的初始化时会依次调用：
        - `setup():` 实现一些参数的设置（一般不用）
        - `handle():` 必须实现（接收到的数据如何处理），相当于socket的recv方法
        - `finish()：`setup中参数的清理（一般不用）
- 在serve_forever函数中，会实例化MyTCPHandler对象，每个实例中有一个数据通信的socket对象
- *注：*       
    - `ThreadingTCPServer`是异步的，可以同时处理多个连接
    - `TCPServer`是同步的，可以建立多个连接，但是只能recv一个连接的消息

### 群聊程序
```Python
    import socketserver
    import logging

    FORMAT = '%(asctime)s %(threadName)s %(message)s'
    logging.basicConfig(format=FORMAT, level=logging.INFO)

    class ChatHandler(socketserver.BaseRequestHandler):
        ssr = set()      # 多个连接对象存放在类属性中，不是存放在实例属性中

        def handle(self):
            self.ssr.add(self.request)
            while True:
                data = self.request.recv(1024)    # self.request已经是accept连接后新建的socket对象了
                if data.strip() == b'quit' or data == b'':
                    self.ssr.remove(self.request)
                    break
                logging.info(data)
                for ss in self.ssr:
                    ss.send(b'ack' + data)

    if __name__ == '__main__':
        server = socketserver.ThreadingTCPServer(('127.0.0.1', 10007), ChatHandler)
        server.serve_forever()
```
- *注：关键是多连接的添加，不能使用实例属性，要将其添加到类属性中*

- 过度使用多线程的群聊
```Python
    import socketserver
    import threading

    import logging

    FORMAT = '%(asctime)s %(threadName)s %(message)s'
    logging.basicConfig(format=FORMAT, level=logging.INFO)

    class ChatS(socketserver.BaseRequestHandler):
        ssr = set()

        def handle(self):
            self.ssr.add(self.request)
            # 开线程的本意是担心会产生阻塞
            # 但是模块在创建新连接部分已经有多线程处理
            # 这里的阻塞只是接受消息的阻塞，不会对创建新连接产生阻塞
            # 因此不用开启新线程
            th = threading.Thread(target=self._recv, name='handle')
            th.start()
            th.join()   # 没有这一句会提示request不是socket
            # while True:
            #     _ = 1

        def _recv(self):
            while True:
                logging.info(self.request)
                data = self.request.recv(1024)
                # 验证data为空字符，可以避免在客户端主动断开连接后抛出异常
                if data.strip() == b'quit' or data == b'':
                    self.ssr.remove(self.request)
                    break
                logging.info(data)
                for ss in self.ssr:
                    ss.send(b'ack' + data)

    if __name__ == '__main__':
        server = socketserver.ThreadingTCPServer(('127.0.0.1', 10007), ChatS)
        # __init__函数执行结束，调用了清理函数将request清理
        server.serve_forever()
```
- *注：以上代码没有th.join()函数，会提示request不是一个socket对象，原因：*
    - 在process_request函数中，先__init__实例化，实例化之后就直接close了request

```Python
    def process_request(self, request, client_address):
            """Call finish_request.

            Overridden by ForkingMixIn and ThreadingMixIn.

            """
            self.finish_request(request, client_address)
            self.shutdown_request(request)

    def finish_request(self, request, client_address):
        """Finish one request by instantiating RequestHandlerClass."""
        self.RequestHandlerClass(request, client_address, self)

    def shutdown_request(self, request):
        """Called to shutdown and close an individual request."""
        self.close_request(request)

    def close_request(self, request):
        """Called to clean up an individual request."""
        pass
```

### 客户端断开连接抛异常处理
- 客户端主动断开连接：
    - recv方法会立即返回一个空bytes，并没有立即抛出异常
    - 当再次循环到recv语句时，才会抛出异常
    - 因此可以在recv函数后判断是否为空bytes来判断客户端的连接状态
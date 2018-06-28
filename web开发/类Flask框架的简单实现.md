### environ解析

#### 使用urilib.parse_qs函数解析environ
- parse_qs函数，将同一个名称的多值，保存在字典中，使用列表保存
```Python
    from wsgiref.simple_server import make_server, demo_app
    from urllib import parse


    # 127.0.0.1:10001/?id=2&name=liubin&age=12&commit=1,2,2&age=23&age=34
    def app(environ, start_response):
        qstr = environ.get('QUERY_STRING')
        print(qstr)
        print(parse.parse_qs(qstr)) # 字典类型

        status = '200 OK'
        headers = [('Content-Type', 'text/html;char=utf-8')]
        start_response(status, headers)
        html = '<h1>Are U OK?</h1>'.encode('utf-8')
        return [html]

    ip = '127.0.0.1'
    port = 10001
    server = make_server(ip, port, app)
    server.serve_forever()
```
- commit不是多值、age是多值

#### webob库解析environ
- 环境数据存放在字典中，字典的存取方式没有对象的属相访问方便
- webob库可以将环境数据的解析封装成对象

#### webob.Request对象
- 将环境参数解析封装成request对象
- GET方法：发送的数据是URL中Query string，在Request Header中
    - request.GET是一个MultiDict字典，里面封装着查询字符串
- POST方法：提交的数据是放在Request Body，同时也使用Query String
    - request.POST可以获取Request Body中的数据，也是个字典
- request.params：提交的数据，所有提交数据的封装
```Python
    from wsgiref.simple_server import make_server, demo_app
    from webob import Request

    def app(environ, start_response):
        request = Request(environ)
        print('header:', request.headers)
        print('method:', request.method)
        print('path:', request.path)
        print('query_str:', request.query_string)
        print('GET:', request.GET)
        print('POST:', request.POST)
        print('params = {}'.format(request.params))

        status = '200 OK'
        headers = [('Content-Type', 'text/html;char=utf-8')]
        start_response(status, headers)
        html = '<h1>Are U OK?</h1>'.encode('utf-8')
        return [html]

    ip = '127.0.0.1'
    port = 10001
    server = make_server(ip, port, app)
    server.serve_forever()
```
#### MultiDict
- 允许一个key存好几个值
```Python
    from webob.multidict import MultiDict

    md = MultiDict()
    md.add(1, 'no1')
    md.add(1, 'no2')
    md.add('a', 1)
    md.add('a', 2)
    md.add('b', '3')
    md['b'] = '5'

    print(md.getall(1))   # ['no1', 'no2']
    print(md.get('a'))    # 2
    print(md.get('c'))    # None
```
#### webob.Reponse对象
```Python
    from wsgiref.simple_server import make_server, demo_app
    from webob import Request, Response

    def app(environ, start_response):
        request = Request(environ)
        print('header:', request.headers)
        print('method:', request.method)
        print('path:', request.path)
        print('query_str:', request.query_string)
        print('GET:', request.GET)
        print('POST:', request.POST)
        print('params = {}'.format(request.params))
        
        res = Response()
        res.status = '200 OK'
        html = '<h1>Are U OK?</h1>'.encode('utf-8')
        res.body = html
        # 自动调用start_response(),并返回可迭代列表
        return res(environ, start_response)

    ip = '127.0.0.1'
    port = 10001
    server = make_server(ip, port, app)
    server.serve_forever()
```

### webob.dec装饰器
- wsgify装饰器
    - app 函数接受一个Request类型的参数，是environ实例化后的对象
    - 返回值都会处理成Response实例
        - 直接返回一个Response类实例
        - 返回一个bytes类型，封装到webob.Response类型实例的body属性
        - 返回一个字符串，转成bytes，在处理

```Python
    from wsgiref.simple_server import make_server
    from webob import Request, Response
    from webob.dec import wsgify

    @wsgify
    def app(request:Request):
        res = Response('<h1> Are u Ok </h1>')
        return res

    ip = '127.0.0.1'
    port = 10001
    server = make_server(ip, port, app)
    server.serve_forever()
```

- 将函数封装成类
```Python
    from wsgiref.simple_server import make_server, demo_app
    from webob import Request, Response
    from webob.dec import wsgify

    class App:
        @wsgify
        def __call__(self, request:Request):
            res = Response('<h1> Are u Ok </h1>')
            return res

    ip = '127.0.0.1'
    port = 10001
    server = make_server(ip, port, App())
    server.serve_forever()
```

### 路由
- 路由：按照不同URL路径来分发数据
- 静态网页：不同的URL代表访问不同的资源
- 动态网页：不同的URL代表不同的应用数据处理，返回数据

#### WEB服务器路由
- 无论静态/动态WEB服务器，都需要路径和资源或处理程序的映射，最终返回HTML文本
- 静态WEB服务器，解决路径和文件之间的映射
- 动态WEB服务器，解决路径和应用程序之间的映射

#### 路由功能实现
- 路由功能通过一个路由类实现（完成路径到应用程序之间的映射）

##### 使用字典建立path到handler的映射关系
- Route类完成path到handler函数的映射
- 并且将handler函数移动到App函数外实现，方便函数功能的修改添加
```Python
    from wsgiref.simple_server import make_server
    from webob import Response, Request
    from webob.dec import wsgify
    from webob.exc import HTTPNotFound

    class Router:
        ROUTERTABLE = {}

        @classmethod
        def register(cls, path):
            def inc(handler):
                cls.ROUTERTABLE[path] = handler
                return handler
            return inc


    @Router.register('/')
    def welcome_handler(request):
        return 'welcome'

    @Router.register('/python')
    def python_handler(request):
        return 'welcome to python'

    class App:
        _router = Router     # 使用类作为属性值

        @wsgify
        def __call__(self, request:Request):
            print(request.method)
            print(request.GET)
            print(request.POST)
            print(request.path)
            path = request.path
            try:
                return  self._router.ROUTERTABLE[path](request)
            except:   # 404等异常处理
                raise HTTPNotFound('WEB NOT FOUND')

    if __name__ == '__main__':
        host = '127.0.0.1'
        port = 10002
        server = make_server(host, port, App())
        server.serve_forever()
```
##### 路由正则匹配
- 使用字典作为映射数据关系的问题：路径匹配应该是有顺序的，但是普通字典是无序的
- 使用确定字符串作为匹配值的问题：匹配路径很死板，使用正则可以更灵活
```Python
    from wsgiref.simple_server import make_server
    from webob import Request, Response
    from webob.dec import wsgify
    from webob.exc import HTTPNotFound
    import re

    class Router:
        ROUTERTABLE = []   # 保证注册路径映射对有序

        @classmethod
        def register(cls, path):
            def inc(handler):
                # 在注册时编译，可以避免每次编译的浪费
                cls.ROUTERTABLE.append((re.compile(path), handler))
                return handler
            return inc

    # 使用正则表达式
    @Router.register('^/$')
    def welcome_handler(request):
        return Response('welcome')

    @Router.register('^/python$')
    def python_handler(request):
        return Response('welcome to python')

    class App:
        _router = Router

        @wsgify
        def __call__(self, request:Request):
            for rpath,handler in self._router.ROUTERTABLE:
                if rpath.match(request.path):
                    return handler(request)
            raise HTTPNotFound('WEB NOT FOUND')

    if __name__ == '__main__':
        host = '127.0.0.1'
        port = 10002
        server = make_server(host, port, App())
        server.serve_forever()
```

- 正则表达式分组捕获
    - 用来捕获特定的数字分组
```python
    @Router.register('^/(?P<id>\d+)$')
    def num_handler(request):
        print(request.groups)
        print(request.groupdict)
        return Response('welcome to {}'.format(request.groupdict['id']))

    class App:
        _router = Router

        @wsgify
        def __call__(self, request:Request):
            for rpath,handler in self._router.ROUTERTABLE:
                matcher = rpath.match(request.path)
                if matcher:
                    # 动态添加属性，因为所有的请求是独立的，添加属性不影响
                    request.groups = matcher.groups()
                    request.groupdict = matcher.groupdict()
                    return handler(request)
            raise HTTPNotFound('WEB NOT FOUND')
```
#### Request Method 过滤
- 即使是同一个URL，也应该很据请求方法的不同，采取不同的处理手段
- 因此在路由注册时加入对应的方法关键字
- 映射关系从URL->handler 变为URL+Method->handler
```Python
    from wsgiref.simple_server import make_server
    from webob import Request, Response
    from webob.dec import wsgify
    from webob.exc import HTTPNotFound
    import re

    class Router:
        ROUTERTABLE = []

        # 修改路由注册，可以接受路径以及多个方法（方法为空，默认全部方法可用）
        @classmethod
        def register(cls, path, *method):
            def inc(handler):
                cls.ROUTERTABLE.append((tuple(map(lambda x:x.upper(), method)), re.compile(path), handler))
                return handler
            return inc

        # 特定方法的装饰器实现
        @classmethod
        def get(cls, path):
            return cls.register(path, 'GET')

    # 只注册GET和POST方法
    @Router.register('^/$', 'GET', 'POST')
    def welcome_handler(request):
        return Response('welcome')

    # 默认使用全部的方法
    @Router.register('^/python$')
    def python_handler(request):
        return Response('welcome to python')

    # 使用特定的get方法装饰器
    @Router.get('^/(?P<id>\d+)$')
    def num_handler(request):
        print(request.groups)
        print(request.groupdict)
        return Response('welcome to {}'.format(request.groupdict['id']))

    class App:
        _router = Router

        @wsgify
        def __call__(self, request:Request):
            for methods, rpath, handler in self._router.ROUTERTABLE:
                # 判定执行方法是否注册，或者是默认全部方法可用
                if request.method in methods or not methods:
                    matcher = rpath.match(request.path)
                    if matcher:
                        request.groups = matcher.groups()
                        request.groupdict = matcher.groupdict()
                        return handler(request)
            raise HTTPNotFound('WEB NOT FOUND')

    if __name__ == '__main__':
        host = '127.0.0.1'
        port = 10002
        server = make_server(host, port, App())
        server.serve_forever()
```

### 路由分组
- 路由分组：按照URL前缀进行分组
- 建立前缀与URL之间的隶属关系
    - 一个前缀下有若干个URL
    - 不同的前缀对应不同的Router实例，所有路由注册方法，都成了实例的方法
    - App中保存不同前缀的Router实例，提供注册方法来管理Router实例
    - __call__方法依然是WSGI的回调入口，遍历所有的Router实例
    - 将路径匹配代码挪到Router类中，并新建一个match方法
    - match方法匹配返回Response对象，否则返回None
- 映射关系变为：(prefix, method, path) -> handler

```Python
    from wsgiref.simple_server import make_server
    from webob import Response
    from webob.dec import wsgify
    from webob.exc import HTTPNotFound
    import re

    class Router:
        def __init__(self, prefix=''):
            self._prefix = prefix
            self._routertable = []

        def route(self, path, *method):
            def inc(handler):
                self._routertable.append((tuple(map(lambda x:x.upper(), method)), re.compile(path), handler))
                return handler
            return inc

        def get(self, path):
            return self.route(path,'GET')

        def match(self,request):
            if not request.path.startswith(self._prefix):      # 前缀匹配
                print(request.path, '----------------------')
                return None
            for methods, repath, handler in self._routertable:   
                if not methods or request.method in methods:   # 方法匹配
                    matcher = repath.match(request.path.replace(self._prefix, '', 1))  # 路径匹配
                    if matcher:
                        request.groups = matcher.groups()
                        request.groupdict = matcher.groupdict()
                        return handler(request)

    class App:
        _router = []

        @wsgify
        def __call__(self, request):
            for router in self._router:           # 遍历Route
                response = router.match(request)  # 调用math方法
                if response:
                    return response
            raise HTTPNotFound('NOT FOUND WEB')

        @classmethod
        def register(cls, *routers):
            cls._router.extend(routers)

    idx = Router('/admin')        # /admin前缀实例
    python = Router('/python')    # /python前缀实例

    App.register(idx, python)     # App管理所有Route实例

    @idx.route('^/13245$')        # 注册为/admin管理的URL
    def indexhandler(request):
        return Response('index handler')

    @python.route('^/54321$')     # 注册为/python管理的URL
    def pythonhandler(request):
        return Response('python handler')

    if __name__ == '__main__':
        ip = '127.0.0.1'
        port = 10006
        server = make_server(ip, port, App())
        server.serve_forever()
```

### 字典访问属性化
- 将dict.[key]的访问方式变为使用.属性访问的方式
- 字典转属性类
```Python
    class AttrDict:
        def __init__(self, d:dict):
            self.__dict__.update(d if isinstance(d, dict) else {})

        def __getattr__(self, item):
            print('missing')

        def __setattr__(self, key, value):
            raise NotImplementedError

        def __repr__(self):
            return "<AttrDict {}>".format(self.__dict__)

    d = {'a': 1, 'b': 3}
    d = AttrDict(d)
    print(d.a, d.b)
    print(d)
```

- 将匹配字典中的字典改为属性实现
```Python
    from wsgiref.simple_server import make_server
    from webob import Response
    from webob.dec import wsgify
    from webob.exc import HTTPNotFound
    import re

    class AttrDict:
        def __init__(self, d:dict):
            self.__dict__.update(d if isinstance(d, dict) else {})

        def __getattr__(self, item):
            print('missing')

        def __setattr__(self, key, value):
            raise NotImplementedError

        def __repr__(self):
            return "<AttrDict {}>".format(self.__dict__)

    class Router:
        def __init__(self, prefix=''):
            self._prefix = prefix
            self._routertable = []

        def route(self, path, *method):
            def inc(handler):
                self._routertable.append((tuple(map(lambda x:x.upper(), method)), re.compile(path), handler))
                return handler
            return inc

        def get(self, path):
            return self.route(path,'GET')

        def match(self,request):
            if not request.path.startswith(self._prefix):
                print(request.path, '----------------------')
                return None
            for methods, repath, handler in self._routertable:
                if not methods or request.method in methods:
                    matcher = repath.match(request.path.replace(self._prefix, '', 1))
                    if matcher:
                        request.groups = AttrDict(matcher.groups())
                        request.groupdict = AttrDict(matcher.groupdict())
                        return handler(request)

    class App:
        _router = []

        @wsgify
        def __call__(self, request):
            for router in self._router:
                response = router.match(request)
                if response:
                    return response
            raise HTTPNotFound('NOT FOUND WEB')

        @classmethod
        def register(cls, *routers):
            cls._router.extend(routers)

    idx = Router('/admin')
    python = Router('/python')

    App.register(idx, python)

    @idx.route('^/(?P<ID>\d+)$')
    def indexhandler(request):
        print(request.groupdict.ID)
        return Response('index handler')

    @python.route('^/(?P<ID>\d+)$')
    def pythonhandler(request):
        print(request.groupdict.ID)
        return Response('python handler')

    if __name__ == '__main__':
        ip = '127.0.0.1'
        port = 10006
        server = make_server(ip, port, App())
        server.serve_forever()
```
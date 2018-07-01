### 作用
- 使用内存读写str/bytes代替，代替文件读写str/bytes，可以大大提速
- 在内存足够的情况下，应该使用内存操作代替文件操作，减少IO，加速

### StringIO
- io模块中的类：`from io import StringIO`
- 内存中开辟一个*文本模式*的buffer,可以像文件对象一样操作它
- 当close方法被调用的时候，这个buffer会被释放
```Python
    from io import StringIO

    sio = StringIO()    # 新建文本实例对象
    print(sio.readable(), sio.writable(), sio.seekable()) # T T T
    sio.write("sting io good")   # 写入字符串内容，不可以写入bytes
    sio.seek(0)
    print(sio.read())       # 从指针位置到EOF的全部内容
    print(sio.getvalue())   # 获取全部内容，无视指针位置
    sio.close()
```
- 其中，getvalue()函数获取实例对象的全部内容，跟文件指针所在位置无关

### BytesIO
- io模块中的类：`from io import BytesIO`
- 内存中，开辟一个*二进制模式*的buffer，可以像文件对象一样操作它
- 当close方法被调用的时候，这个buffer会被释放
```Python
    from io import BytesIO

    bio = BytesIO()   # 新建bytes实例对象
    print(bio.readable(), bio.writable(), bio.seekable())  # T T T
    bio.write(b"bytes io good")  # 写入bytes内容
    bio.seek(0)    
    print(bio.readline())   # 打印全部内容
    print(bio.getvalue())   # 获取全部内容，无视指针位置
    bio.close()
```

- 上面两种对象的操作与流对象的操作函数一致

### file-like对象
- 类文件对象，可以像文件对象一样操作
- socket对象、输入输出对象（stdin、stdout）都是类文件对象
```Python
    from sys import stdout

    f = stdout
    print(type(f))       # io类对象
    f.write('A U OK?')   # 终端打印
```
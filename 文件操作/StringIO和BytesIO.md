### 作用
- 使用内存读写str/bytes代替，代替文件读写str/bytes，可以大大提速
- 在内存足够的情况下，应该使用内存操作代替文件操作，减少IO，加速

### StringIO
- io模块中的类：`from io import StringIO`
- 内存中开辟一个*文本模式*的buffer,可以像文件对象一样操作它
- 当close方法被调用的时候，这个buffer会被释放
```Python
    from io import StringIO

    sio = StringIO()
    print(sio.readable(), sio.writable(), sio.seekable())
    sio.write("sting io good")
    sio.seek(0)
    print(sio.read())
    print(sio.getvalue())
    sio.close()
```
- 其中，getvalue()函数获取实例对象的全部内容，跟文件指针所在位置无关

### BytesIO
- io模块中的类：`from io import BytesIO`
- 内存中，开辟一个*二进制模式*的buffer，可以像文件对象一样操作它
- 当close方法被调用的时候，这个buffer会被释放
```Python
    from io import BytesIO

    bio = BytesIO()
    print(bio.readable(), bio.writable(), bio.seekable())
    bio.write(b"bytes io good")
    bio.seek(0)
    print(bio.readline())
    print(bio.getvalue())
    bio.close()
```
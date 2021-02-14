# Python笔记
>wvv 20191216

本笔记为python一些使用经验，默认采用python3代码进行说明。
更多资料请查看
[https://docs.python.org/3/](https://docs.python.org/3/)
[toc]
### 环境配置相关
#### pip换国内源
建立文件夹
`mkdir ~/.pip`
编辑配置文件
`vim ~/.pip/pip.conf`
内容如下
```
[global]
trusted-host=mirrors.aliyun.com
index-url=https://mirrors.aliyun.com/pypi/simple/
```
#### pip使用
最好采用安装到用户目录方式安装
`pip3 install --user xxxx`
注意python3 和python2的安装区别
用户权限安装库xxx
`pip3 install --user xxx`
#### IDE
推荐使用[**pycharm**](https://www.jetbrains.com/pycharm)

#### pycharm 中配置venv
venv是python的虚拟环境，每个工程设置一个虚拟环境，这样就可以使得不同的工程间库不至于冲突，pycharm中很容易的进行venv的设置。
设置位置位于：
File -> Setting -> Project: xxx -> Project Interpreter
左键点击右侧齿轮符号，点击出现的Add，弹出配置窗口
选择New environment, 设置好 Base interpreter
就完成了venv的设置
#### pycharm在venv中安装第三方库
设置位置位于：
File -> Setting -> Project: xxx -> Project Interpreter
点击想要设置的工程，右侧会列出所有安装的第三方库，点击右侧 + 符号，在出现的搜索框中输入自己想要的库名称，找到想要安装的库，点击安装完成安装。
#### pycharm进入venv的命令行
点击pycharm主界面下方的 Terminal栏，自动进入项目终端环境
执行命令
`cd venv`
`cd Scripts`
进入venv\Scripts 文件夹
执行命令
`activate`
激活venv环境
激活后终端路径前面自动会添加 (venv)表示已经进入虚拟环境
在虚拟环境下，执行pip安装命令会自动将库安装到虚拟环境中，一切相关命令使用与非虚拟环境一致，区别是仅会影响虚拟环境
`deactivate`
退出虚拟环境
对于很多库直接采用pip在线方式安装可能会失败，可以去官网
[https://pypi.org/](https://pypi.org/)
自行下载对应whl包，采用pip离线方式进行安装
具体命令形如
`pip install d:\xxxx.whl`
#### pip 安装库出现异常解决办法
##### 安装whl文件，出现is not a supported wheel on this platform
参考 [https://blog.csdn.net/xiuxiuen_michelle/article/details/81080694](https://blog.csdn.net/xiuxiuen_michelle/article/details/81080694)
原因是：下载的whl文件名形如 xxx-cpxx-cpxx-win_xx.whl
其中cpxx 等是其支持的平台，下载的时候首先要选择自己当前平台对应的whl包，比如python3.8 应该对应形如cp38格式，系统是64位windows则最后一段应该是win_amd64
当确认了没下载错之后，还出现这个问题，则是出现在第二段名字上，
首先在命令行输入python，进入python解释器
执行
```python
import pip._internal
print(pip._internal.pep425tags.get_supported())
```
会打印出系统支持的包的类型，格式形如[('cp38','cp38m','win_amd64'),(....)]
将自己下载的包的文件名改成对应的文件名就可以进行pip安装了
如果包确定没下载错，一般是第二段支持有问题，将第二段改成none一般就可以解决问题了
##### pip在线安装，出现 Microsoft Visual C++ xxxx is required.
因为有些库是要进行源码编译的，默认环境采用的是Microsoft Visual C++ 编译器来进行编译的，我们通过安装对应的编译环境再重启就可以解决问题了。
但是Microsoft Visual C++编译环境搭建需要占用数个G bytes的文件，更优的办法是根据报错日志，找出是哪几个包需要进行编译，进而报错，然后到官网[https://pypi.org/](https://pypi.org/) 下载对应库编译好的whl文件，进行离线pip安装，就不需要进行c++编译了，但是要注意两个问题，whl平台不要下载错了，还有库之间存在依赖关系，要把所有需要编译的库都进行whl下载安装了才能正确安装。
例如安装arcade库，除了下载arcade的whl文件后，还需要依赖Pillow,  numpy 都需要进行编译，所以要找到对应的whl都进行离线安装才可以。不过即使这样也还是比下载完整c++编译环境要更科学合理。

### 打包发布
#### pyinstaller
支持Windows和Linux平台
主页地址
[http://www.pyinstaller.org/](http://www.pyinstaller.org/)
##### 减少发布体积
参考 [https://www.zhihu.com/question/281858271](https://www.zhihu.com/question/281858271)
主要是利用虚拟环境，搭建最小库安装依赖，就能够最大减小发布文件大小
#### py2exe
只支持Windows平台

### 源码保护

#### 源码混淆

可以使用在线服务

http://pyob.oxyry.com/

但是源码混淆代码量不大的情况，替换分析起来还是比较容易，意义不是很大。

#### pyc字节文件

pyc是Python脚本编译后形成的字节文件。生成后的pyc文件可以直接**替换**对应的py文件。通过py_compile可以将代码编译为pyc字节文件。编译好后直接替换原文件就可以。

```python
import py_compile  
py_compile.compile(file="xxx.py")
```

字节码文件很容易反编译，通过[uncompyle6](https://github.com/rocky/python-uncompyle6/) 就可以直接得到源码，所以保护性不强。

#### so/pyd 文件

通过cython可以将py转换为c文件，然后再编译为python可以直接调用的动态链接库。so为linux下文件，pyd为windows平台文件。这里以linux平台为例进行使用说明。

1. 安装cython

```shell
pip3 install cython
```

2. 确保gcc已经安装
3. 新建build.py文件

```python
from distutils.core import setup
from Cython.Build import cythonize
setup(ext_modules = cythonize(["m0.py","m1.py"]))
```

其中m0.py m1.py ... 为需要保护，编译为so文件的源文件。在终端执行

```shell
python build.py build_ext
```

会在当前目录生成build/lib.xxxx/文件夹，里面有生成的形如 xx.cpython-yy.so格式文件，将该so文件直接替换原文件即可，python会自动识别。

4. 剩余事项

编译的时候会有很多临时文件，如果觉得不好看，可以自行编写脚本删除，实现自动编译。

### 基础知识
#### 字符串格式化
格式化常见有三种方式
1. "{}{}{}".format(a,b,c)
2. "%d %d %d" % (a,b,c,) #类C风格
3. f"{a}" #fstring 风格，python3.6以后支持
##### 数字类型格式化

| 格式    | 描述                            |
| ------- | ------------------------------- |
| {:.2f}  | 保留小数点两位                  |
| {:+.2f} | 带符号保留小数点两位            |
| {:.2%}  | 百分比，保留小数点两位          |
| {:0>5d} | 整型占位5，右对齐，左侧填充0    |
| {: <4d} | 整型占位4，左对齐，左侧填充空格 |
| {:^3d}  | 整型占位3，中间对齐             |
| {:,}    | 逗号分隔                        |
| {:b}    | 二进制显示                      |
| {:x}    | 小写字母显示十六进制            |
| {:#X}   | 带0x前缀，大写字母显示十六进制  |
| {{  }}  | {  }的转义                      |
#### 类与继承
#### 元类
#### 协作开发
##### 为每个函数、类、模块编写文档
##### 用包来安排模块
##### 自编模块定义根异常
##### 打破循环依赖
##### 虚拟环境隔离项目
#### 参数传递类型
在Python中，所有参数都是通过**引用传递（地址传递）**，但是Python中变量分为可变类型和不可变类型，当给不可变类型变量赋值时，实际上会产生一个同名变量覆盖了以前的变量，这样就会造成很多传参时的困惑。
下面给出几个Python3例程，能够清楚的说明问题。
##### 例程一
```Python
sig=1
print(hex(id(sig)))
class CTest:
    def __init__(self,n):
        self.sig = n
    def run(self):
        print(f"{self.sig} {hex(id(self.sig))}")
a= CTest(sig)
a.run()
sig = 12
a.run()
print(hex(id(sig)))
```
执行结果
```
0x7ff949c896a0
1 0x7ff949c896a0
1 0x7ff949c896a0
0x7ff949c89800
```
**结果分析**
我们发现sig一开始地址为0x7ff949c896a0，传入了类实例后，实例地址与其相同，对sig第二次赋值后，sig地址发生了变化，但是实例内的地址还是以前的地址，所以外部修改变量内容但是实例内部并没有相应修改，这样容易让人误以为Python是按值进行传递的，实际上是因为后面的变量已经并非是之前的变量了。
##### 例程二
```Python
sig=[0]
print(hex(id(sig)))
class CTest:
    def __init__(self,n):
        self.sig = n
    def run(self):
        print(f"{self.sig} {hex(id(self.sig))}")
a= CTest(sig)
a.run()
sig[0] = 1
a.run()
print(hex(id(sig)))
```
执行结果
```
0x1ea8d23eb80
[0] 0x1ea8d23eb80
[1] 0x1ea8d23eb80
0x1ea8d23eb80
```
**结果分析**
例程二与例程一的区别在于sig类型为列表，从头到尾sig地址都没有发生变化，在类实例进行初始化后，外部修改了变量内容，类实例内部值也发生了变化。这充分表明参数是按地址传递的，同时说明了列表是可变类型，单独修改列表内部数值列表地址并不会发生变化。
##### 例程三
```Python
sig=[0]
print(hex(id(sig)))
class CTest:
    def __init__(self,n):
        self.sig = n
    def run(self):
        print(f"{self.sig} {hex(id(self.sig))}")
a= CTest(sig)
a.run()
sig = [1]
a.run()
print(hex(id(sig)))
```
执行结果
```
0x1fd2ff2fe40
[0] 0x1fd2ff2fe40
[0] 0x1fd2ff2fe40
0x1fd3000cc80
```
**结果分析**
例程三与例程二的区别在于sig赋值的时候不是直接将sig内的值进行修改，而是将一个新的列表赋值给了sig，这样就表现出来和例程一 一样的值传递的效果，实际上根据最后打印出的sig的地址，我们发现sig又指向了不同的位置。这提醒了大家要小心操作列表，虽然列表是可变类型，是表示列表内的元素可变，但是直接对整个列表赋值也会产生新的列表。
##### 例程四
``` Python
s = [0]
for i in range(4):
    s[0] = i
    print(f"{hex(id(s))} {hex(id(s[0]))}")
print(f"{s[0]} {hex(id(s))} {hex(id(s[0]))}")
```
执行结果
```
0x2ab65eaeb80 0x7ff949c89680
0x2ab65eaeb80 0x7ff949c896a0
0x2ab65eaeb80 0x7ff949c896c0
0x2ab65eaeb80 0x7ff949c896e0
3 0x2ab65eaeb80 0x7ff949c896e0
```
**结果分析**
例程四很有意思，揭示了可变类型的实现原理，我们发现在每次进行内部元素赋值时，实际上内部元素地址都发生变化了，意思就是每次赋值都产生了一个新的变量，但是可变类型自身地址没有发生变化，这就表示列表还是那个列表，但是列表内部的元素实际上每次赋值都指向了新的地址，列表元素指向了新的地址。我们也可以发现列表和列表元素地址相差非常远，也说明了python虚拟机维护了列表的类地址，列表元素存放位置是随时变化的，变化的同时索引信息也自动更新了。
##### 例程五
###### 1
``` Python
s = [0, 2]
print(f"{s[0]} {hex(id(s))} {hex(id(s[0]))} {hex(id(s[1]))}")
for i in range(3):
    s[0] = i
    print(f"{hex(id(s))} {hex(id(s[0]))}")
print(f"{s[0]} {hex(id(s))} {hex(id(s[0]))} {hex(id(s[1]))}")
```
执行结果
```
0 0x1f00110fe40 0x7ff949c89680 0x7ff949c896c0
0x1f00110fe40 0x7ff949c89680
0x1f00110fe40 0x7ff949c896a0
0x1f00110fe40 0x7ff949c896c0
2 0x1f00110fe40 0x7ff949c896c0 0x7ff949c896c0
```
###### 2
``` Python
s = [5, 2]
print(f"{s[0]} {hex(id(s))} {hex(id(s[0]))} {hex(id(s[1]))}")
for i in range(3):
    s[0] = i
    print(f"{hex(id(s))} {hex(id(s[0]))}")
print(f"{s[0]} {hex(id(s))} {hex(id(s[0]))} {hex(id(s[1]))}")
```
执行结果
```
5 0x29d098cfe40 0x7ff949c89720 0x7ff949c896c0
0x29d098cfe40 0x7ff949c89680
0x29d098cfe40 0x7ff949c896a0
0x29d098cfe40 0x7ff949c896c0
2 0x29d098cfe40 0x7ff949c896c0 0x7ff949c896c0
```
###### 3
``` Python
s = [5, 2]
print(f"{s[0]} {hex(id(s))} {hex(id(s[0]))} {hex(id(s[1]))}")
for i in range(3):
    s[0] = 5-i
    print(f"{hex(id(s))} {hex(id(s[0]))}")
print(f"{s[0]} {hex(id(s))} {hex(id(s[0]))} {hex(id(s[1]))}")
```
执行结果
```
5 0x16bac13fdc0 0x7ff949c89720 0x7ff949c896c0
0x16bac13fdc0 0x7ff949c89720
0x16bac13fdc0 0x7ff949c89700
0x16bac13fdc0 0x7ff949c896e0
3 0x16bac13fdc0 0x7ff949c896e0 0x7ff949c896c0
```
**结果分析**
这三个子例程揭示了可变类型与不可变类型的关系，
1. 例程1开始的时候s[0],s[1]分别指向了不同的地址，随着给s[0]赋值，s[0]指向了不同的地址，最后s[0]和s[1]指向了相同的地址，内容都是2
2. 例程2和例程1的唯一区别在于s[0]初始化时值不一样，我们发现例程1中，s初始化后s[0]地址位于s[1]之前，例程2中，s[0]地址位于s[1]之后，这是因为例程1中初始化值s[0]小于s[1]，例程2中初始化值s[0]大于s[1]
3. 例程3和例程2的唯一区别在于中间对s[0]赋值语句不一样，一个是i，一个是5-i，然后我们发现循环中打印出来的s[0]的地址在例程2中是上升的，例程3中是下降的
4. 可变对象中的每个元素位置都是一个索引，指向一些不可变对象位置，Python内部自动生成了一定的升序排列的常数不可变对象，列表中不同项如果内容相同，实际上会对应同一个内存地址
##### 结论
* 函数参数传递是按引用传递
* 不可变对象赋值会产生新的变量，和原来变量同名的另一个变量
* 列表、字典等可变对象，对其内容赋值其内容存放的位置会发生变化，其自身地址不会变化，但是对其自身赋值会产生新的同名对象指向不同区域
* 推测Python初始化时内存中有一块区域存放了一个不可变量表，预置了一些值（比如多少以内的整数），每次计算过程中如果产生了不可变量表以外的结果，则会添加到不可变量表中，如果发生不可变量赋值行为，实际上是从不可变量表中找到变量地址给不可变量，这样就造成了不可变量的覆盖。
* 可以利用可变对象以及引用传递的特性来实现类全局变量引用绑定
#### Binary Sequence Types
* bytes 不可变单字节序列
    1. Bytes objects are immutable sequences of single bytes. 
    2. Only ASCII characters are permitted in bytes literals (regardless of the declared source code encoding). Any binary values over 127 must be entered into bytes literals using the appropriate escape sequence.
* bytearray 可变单字节序列
    1. Creating an empty instance: bytearray()
    2. Creating a zero-filled instance with a given length: bytearray(10)
    3. From an iterable of integers: bytearray(range(20))
    4. Copying existing binary data via the buffer protocol: bytearray(b'Hi!')
* memoryview 直接访问对象内存，而不产生拷贝
objects allow Python code to access the internal data of an object that supports the buffer protocol without copying.
    * cast(format[,shape]) 可完成类型强制转换
    * ntytes
    * tobytes()
#### 权限修饰符
* _x 
单下划线修饰的成员变量，表示只有类实例或者子类实例可以访问，类似于protect
* __x
双下划线修饰的成员变量，表示只有类对象自己能访问，类似于private
* \_\_x__
系统定义名字，表示特殊方法专用的标识
### 内部库使用
#### re
#### threading
#### multiprocessing
#### queue
#### socket
#### socketserver
#### urllib

#### base64
``` python
import base64

st = 'hello world!'.encode()#默认以utf8编码
res = base64.b64encode(st)
print(res.decode())#默认以utf8解码
res = base64.b64decode(res)
print(res.decode())#默认以utf8解码

aGVsbG8gd29ybGQh
hello world!
```
#### json
```python
import json
data1 = {
    'no' : 1,
    'name' : 'Runoob',
    'url' : 'http://www.runoob.com'
}
json_str = json.dumps(data1)  # json to str
data2 = json.loads(json_str)  # str to json
```
#### struct
#### logging
#### sqlite3
```python
import sqlite3
conn = sqlite3.connect('test.db')
print ('sqlite open')
c = conn.cursor()
c.execute('''CREATE TABLE STUDENT
       (ID INT PRIMARY KEY     NOT NULL,
       NAME           TEXT    NOT NULL,
       AGE            INT     NOT NULL,
       ADDRESS        CHAR(50) NOT NULL);''')
c.execute("INSERT INTO STUDENT (ID,NAME,AGE,ADDRESS) VALUES (1, 'hello', 32, 'Beijing' )");
print ('Table created')
conn.commit()
conn.close()
```
### 外部库使用
#### SQLAlchemy
#### aiohttp
aiohttp是采用asyncio的高性能异步库
`pip install aiohttp`
简易http服务器
``` python
from aiohttp import web

routes = web.RouteTableDef()

@routes.post('/login')
async def hello(request):
    data = await request.json()
    print(data)
    if data["name"] == "guest" and data["pwd"] == "1234":
        return web.json_response({"login": "ok"})
    return web.json_response({"login": "fail"})
    
app = web.Application()
app.add_routes(routes)
web.run_app(app, port=3000)
```

#### stomp
stomp为activemq所支持的一种文本传输协议
`pip install stomp.py`
```python
import time
import sys
import stomp
 
class MyListener(object):
    def on_error(self, headers, message):
        print('received an error %s' % message)
    def on_message(self, headers, message):
        print('received a message %s' % message)
 
conn = stomp.Connection10([('127.0.0.1',61613)])  
conn.set_listener('', MyListener())
conn.start()
conn.connect()
conn.subscribe(destination='queue/test')
conn.send(body='xxxx', destination='queue/test')
```
#### opencv
pip安装，目前最新版为4.1.2
`pip install opencv-python`
基本使用方式
```python
import cv2
import numpy as np
cap = cv2.VideoCapture(0)
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 320)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 240)
while True:    
    ret, frame = cap.read()
    cv2.imshow('frame', frame)
    cv2.waitKey(1)
```
图像转成jpeg，base64，编码解码
```python
ret, frame = cap.read()
img_encode = cv2.imencode('.jpg', frame, [cv2.IMWRITE_JPEG_QUALITY, 80])[1]
str_encode = np.array(img_encode).tostring()  # convert to bytes
b64_str = base64.b64encode(str_encode)  # bytes to base64
image_data = base64.b64decode(message)  # base64 to bytes
np_image = np.frombuffer(image_data, dtype="uint8")  # bytes to nparray
image = cv2.imdecode(np_image, cv2.IMREAD_UNCHANGED)  # decode to jpeg
```
### web框架

#### Django

```
pip install Django
```

### 综合示例

#### 通过网络传输摄像头采集视频
请先pip安装stomp opencv库，安装方法参考前面章节
```python
import stomp
import cv2
import numpy as np
from threading import Thread
from time import sleep
import base64


class Producer(Thread):
    def __init__(self):
        Thread.__init__(self)

    def run(self):
        img_mq_conn = stomp.Connection10([("server.com", 61613)])
        img_mq_conn.start()
        img_mq_conn.connect("admin", "admin")
        img_mq_conn.subscribe("test")
        cap = cv2.VideoCapture(0)
        cap.set(cv2.CAP_PROP_FRAME_WIDTH, 320)
        cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 240)
        while True:
            ret, frame = cap.read()
            _, img_encode = cv2.imencode('.jpg', frame, [cv2.IMWRITE_JPEG_QUALITY, 80])  # decode with quality
            str_encode = np.array(img_encode).tostring()  # convert to bytes
            img_mq_conn.send("test", base64.b64encode(str_encode))  # send base64 to message queue
            sleep(0.03)


class Consumer(object):
    def on_message(self, headers, message):
        image_data = base64.b64decode(message)
        np_image = np.frombuffer(image_data, dtype="uint8")
        image = cv2.imdecode(np_image, cv2.IMREAD_UNCHANGED)  # decode to jpeg
        cv2.imshow('frame', image)
        cv2.waitKey(1)


if __name__ == "__main__":
    img_consumer = stomp.Connection10([("server.com", 61613)])
    img_consumer.set_listener("", Consumer())
    img_consumer.start()
    img_consumer.connect("admin", "admin")
    img_consumer.subscribe("test")
    c = Producer()
    c.setDaemon(True)
    c.start()
    try:
        while True:
            sleep(1)
    except KeyboardInterrupt:
        exit()
```


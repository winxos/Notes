# MicroPython 笔记

> wvv20191229

[toc]
### 安装烧录工具

`pip install esptool`
[ESP8266 MicroPython固件下载](http://www.micropython.org/download#esp8266)

[ESP8266 烧录方式参考](http://docs.micropython.org/en/latest/esp8266/tutorial/intro.html#deploying-the-firmware)

#### ESP8266 Windows 系统下烧录
上面的链接给出了详细固件烧录的过程，但是例子用的是linux系统下，在windows下，命令有一些变化
擦除芯片指令,注意把COM9替换成自己设备串口
`esptool.py --port COM9 erase_flash`
执行结果
``` sh
esptool.py v2.8
Serial port COM9
Connecting....
Detecting chip type... ESP8266
Chip is ESP8266EX
Features: WiFi
Crystal is 26MHz
MAC: dc:4f:22:1c:0d:08
Uploading stub...
Running stub...
Stub running...
Erasing flash (this may take a while)...
Chip erase completed successfully in 3.2s
Hard resetting via RTS pin...
```
烧录固件,注意把COM9替换成自己设备串口，把xxx.bin 替换成自己下载固件文件名
`
esptool.py --port COM9 --baud 460800 write_flash --flash_size=detect 0 esp8266-xxxx.bin`

### ESP32 烧录特殊性
网上教程的很多烧入方式采用的NodeMCU-PyFlasher 来进行，此工具在烧写ESP32后运行会出现
``` sh
rst:0x10 (RTCWDT_RTC_RESET),boot:0x13 (SPI_FAST_FLASH_BOOT)
flash read err, 1000
ets_main.c 371 
ets Jun  8 2016 00:22:57
```
的错误，这是可能因为默认ESP32 固件应该是从0x1000来启动固件，而改烧录工具生成的烧录指令不正确，
所以还是老实按照MicroPython官方的烧录教程来进行。
先下载ESP32固件
[ESP32固件地址](https://micropython.org/download#esp32)
然后类似ESP8266烧录方式，执行清除flash，COM10替换成自己串口号
`esptool.py --port COM10 erase_flash`
执行烧录指令,把xxx.bin 替换成自己下载固件文件名
`esptool.py --chip esp32 --port COM10 write_flash -z 0x1000 esp32-xxxx.bin`
用任何串口工具接入，就能看到正常启动信息了。
``` sh
rst:0x1 (POWERON_RESET),boot:0x13 (SPI_FAST_FLASH_BOOT)
configsip: 0, SPIWP:0xee
clk_drv:0x00,q_drv:0x00,d_drv:0x00,cs0_drv:0x00,hd_drv:0x00,wp_drv:0x00
mode:DIO, clock div:2
load:0x3fff0018,len:4
load:0x3fff001c,len:4928
ho 0 tail 12 room 4
load:0x40078000,len:9332
load:0x40080400,len:6216
entry 0x400806e8
I (433) cpu_start: Pro cpu up.
I (434) cpu_start: Application information:
I (434) cpu_start: Compile time:     00:41:11
I (436) cpu_start: Compile date:     May 14 2019
I (441) cpu_start: ESP-IDF:          v3.3-beta1-268-g5c88c5996
I (448) cpu_start: Starting app cpu, entry point is 0x40082950
I (0) cpu_start: App cpu up.
I (458) heap_init: Initializing. RAM available for dynamic allocation:
I (465) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
I (471) heap_init: At 3FFB9B88 len 00026478 (153 KiB): DRAM
I (477) heap_init: At 3FFE0440 len 00003AE0 (14 KiB): D/IRAM
I (484) heap_init: At 3FFE4350 len 0001BCB0 (111 KiB): D/IRAM
I (490) heap_init: At 40093434 len 0000CBCC (50 KiB): IRAM
I (496) cpu_start: Pro cpu start user code
I (67) cpu_start: Starting scheduler on PRO CPU.
I (0) cpu_start: Starting scheduler on APP CPU.
MicroPython v1.10-342-g7359a9e2f on 2019-05-14; ESP32 module with ESP32
Type "help()" for more information.
>>> 
```
### IDE 选择
MicroPython的开发工具常用的有uPyCraft
目前也流行起Thonny
这里推荐Thonny,该软件也是树莓派默认的Python开发工具，使用时需要在设置里面配置一下对MicroPython设备的支持

### MicroPython编程
MicroPython基本语法和模块使用与python3 差异很小，可以自行阅读官网文档
[ESP32 MicroPython参考](http://docs.micropython.org/en/latest/esp32/quickref.html)
#### 接入WIFI，UDP通信
``` python
import socket
def do_connect():
    import network
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    if not wlan.isconnected():
        print('connecting...')
        wlan.connect('HeLuo', '11111111')
        while not wlan.isconnected():
            pass
    print('network config:', wlan.ifconfig())
do_connect()
port = 10086
s=socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s.bind(('0.0.0.0',port))
print('waiting...')
while True: 
    data,addr=s.recvfrom(1024)
    print('received:',data,'from',addr)
```
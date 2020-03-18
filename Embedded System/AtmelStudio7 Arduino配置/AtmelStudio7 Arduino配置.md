# AtmelStudio7 Arduino 固件开发配置

> 本文讲述了AtmelStudio7下，Arduino C语言开发。
>
> 为什么需要采用C语言开发？在固件编写的时候，需要尽可能少的占用系统资源，Arduino自带的IDE过于简单，默认编译器为C++，编译出来的文件在arduino的固件下运行，所以就不适合用来做自己的固件开发。**AtmelStudio7**为目前avr单片机官方最新版本，采用定制的VisualStudio2015来作为IDE，支持全系列AVR单片机，体验非常的好。但是目前找不到一篇完整的**AtmelStudio7 下进行Arduino 固件**编写的教程，所以就产生了此文。
>
> winxos 2016-12-02

[TOC]

## AtmelStudio7 安装

AtmelStudio7为AVR系列单片机官方开发环境，完全免费，最新版下载地址为：http://www.atmel.com/tools/atmelstudio.aspx

界面如下： ![atmelstudio7](.\pics\atmelstudio7.png)

AtmelStudio7中整合了ASF向导，能够自动化对工程进行配置，而且提供了大量的示例工程，使用起来非常方便。

 ![1](.\pics\1.png)

 新建工程里面有很多选择，也可以打开示例工程。![2](.\pics\2.png)

在箭头处点击后可以修改芯片型号。

 ![3](.\pics\3.png)

按箭头处或者 F7快捷键可以进行工程编译，编译速度非常的快，比Arduino IDE快了几个数量级。



## USBASP烧写器

AtmelStudio7默认支持jtag进行代码烧写、调试，但是AVR单片机正版JTAG非常贵，山寨版只能使用非常低版本的AtmelStudio，所以这种的方案我们可以采用USBASP/USBISP进行代码烧写，但是没法进行代码实时调试。

 ![4](.\pics\4.jpg)

淘宝10块左右，但要注意默认的是10PIN的，Arduino上默认为6PIN，所以还要买个转接头。



## Avrdude参数配置

在windows上烧写AVR单片机需要借助**Avrdude**这个工具，它支持很多协议，所以我们可以在AtmelStudio7中配置用**Avrdude**来将我们编译好的十六进制文件写入Arduino中。Avrdude可以单独在网上下载，如果安装了**Arduino IDE**，里面已经包含了Avrdude，路径为：

> (arduino ide安装文件夹)\hardware\tools\avr\bin\avrdude.exe

Avrdude可以完成配置熔丝位等高级的功能，所以参数非常的复杂，还好搜索到了一篇相关内容，

http://www.avrfreaks.net/comment/1973631

其实我们也可以参考Arduino IDE的实现方式来进行配置，avrdude的配置参数位于：

> (arduino ide安装文件夹)\hardware\arduino\avr\platform.txt

其中我们在倒数几行的位置可以看到：

> tools.avrdude.erase.pattern="{cmd.path}" "-C{config.path}" {program.verbose} {program.verify} -p{build.mcu} -c{protocol} {program.extra_params} "-Uflash:w:{build.path}/{build.project_name}.hex:i"

这样的语句，上面很清楚的表达了各个参数的意思，比如-p后面跟mcu的类型，-c后面跟协议等等，注意参数区分大小写，当然更详细的参数可以到avrdude官网去查看。



**熔丝位配置**是个特别需要注意的地方，一定不能乱写，我们也可以参考Arduino IDE的实现方式，各个Arduino 开发板配置文件位于：

> (arduino ide安装文件夹)\hardware\arduino\avr\boards.txt

比如我们搜索Nano我们可以发现：

> nano.menu.cpu.atmega328.bootloader.low_fuses=0xFF
> nano.menu.cpu.atmega328.bootloader.high_fuses=0xDA
> nano.menu.cpu.atmega328.bootloader.extended_fuses=0x05

这就是Nano的熔丝位的合理配置，熔丝位总共有24位。



另外一个重要参数就是avrdude的配置文件，就是 -C 参数，这个我们可以采用avrdude默认配置文件，位于：

> (arduino ide安装文件夹)\hardware\tools\avr\etc\avrdude.conf



-Uflash 后面添加的就是要写入的HEX文件的路径



## 整合Avrdude到AtmelStudio7



以上我们就搞清了采用命令行利用Avrdude烧写Arduinod 方式，我们可以把其整合到AtmelStudio7界面上，这样可以实现一键烧写代码。
我们打开AtmelStudio7的 **Tools > External Tools** 菜单

 ![7](.\pics\7.png)

如上图

 ![8](.\pics\8.png)

然后添加一个命令，特别注意上图箭头的几个地方，其中：

Command 就是执行工具，这里我们指向Avrdude。

Arguments 就是传入参数，就是我们上一节讲的一大堆东西。

Use Output window选中，这样我们就可以在AtmelStudio7上看到Avrdude的输出信息。

因为我们传入Avrdude的参数中包含我们编译生成的hex文件，而这个文件的路径和名字又是不确定的，所以我们还需要采用AtmelStudio7中的宏定义来完成变量的替换。

## AtmelStudio7 外部工具宏定义

我们可以到Atmel官网找到全部的宏定义，http://www.atmel.com/webdoc/atmelstudio/ch10s05s02.html#external.tools.macros

其中我们可能需要用到的有

| Name             | Argument      | Description                              |
| ---------------- | ------------- | ---------------------------------------- |
| Target Directory | $(TargetDir)  | The directory of the item to be built.   |
| Target Name      | $(TargetName) | The file name of the item to be built.   |
| Binary Directory | $(BinDir)     | The final location of the binary that is being built (defined as drive + path). |



所以我们得到了一个可行的配置方案：

Command：参数设置为

> D:\\(arduino ide安装文件夹)\hardware\tools\avr\bin\avrdude.exe

Arguments: 参数设置为

> -C "D:\\(arduino ide安装文件夹)\hardware\tools\avr\etc\avrdude.conf" -c usbasp -p atmega328p -b 115200 -Uflash:w:\$(TargetDir)\$(TargetName).hex:i

请自行根据自身情况对上面参数进行替换，主要需要替换的位置为：

1. (arduino ide安装文件夹)
2. -p atmega328p

我们再把刚才的工具添加到我们的自定义工具栏，至此我们开发环境就配置好了。

## AtmelStudio7 编写最简单的代码

我们打开示例工程，设置好MCU型号，我们发现主文件如下代码：

###mega_led_example.c
``` c
#include <asf.h>
int main (void)
{
	/* set board io port */
	board_init();
	bool button_state;
	while(1){
		button_state = ioport_get_pin_level(GPIO_PUSH_BUTTON_0);
		if(button_state){
			LED_Off(LED0);
		}else{
			LED_On(LED0);
		}
	}	
}
```

我们发现asf框架给我们封装了大量的函数，Arduino 系列标准都在13号口接了一个LED，我们就可以用来测试。

由于我们用的原生C来进行编程，所以我们就不能像Arduino IDE那样通过13号口来调用了，所以我们要查看Arduino Nano的引脚对应图，还好网上很多，如下：

 ![9](.\pics\9.jpg)

我们发现，D13对应的是PB5，就是B号端口的第6根引脚，

我们查看LED0的定义（ALT+G快捷键）,发现：

```c
#define LED0_GPIO                       IOPORT_CREATE_PIN(PORTB, 5)
#define LED0                            LED0_GPIO
```

正好是PB5，那么我们就不用修改了，我们再看看`GPIO_PUSH_BUTTON_0`的定义，

发现是：

`#define GPIO_PUSH_BUTTON_0              IOPORT_CREATE_PIN(PORTB, 7)`

但是我们的Nano上并不能找到PB7号端口，所以我们把它修改为PB0，就是D8口

`#define GPIO_PUSH_BUTTON_0              IOPORT_CREATE_PIN(PORTB, 0)`

然后按F7进行编译，

我们发现非常的快，大概1秒左右就完成了编译，输出了成功的消息，部分如下：

	Task "RunOutputFileVerifyTask"
				Program Memory Usage 	:	158 bytes   0.5 % Full
				Data Memory Usage 		:	0 bytes   0.0 % Full
	Build succeeded.
	========== Build: 1 succeeded or up-to-date, 0 failed, 0 skipped ==========
然后我们再点击刚才配置好的USBASP按钮，如下图：

 ![10](.\pics\10.png)

完后我们发现瞬间就传完了，因为我们之前把传输速度配置成了115200，部分输出信息如下：

```
avrdude.exe: Device signature = 0x1e950f
avrdude.exe: NOTE: "flash" memory has been specified, an erase cycle will be performed
avrdude.exe: erasing chip
avrdude.exe: warning: cannot set sck period. please check for usbasp firmware update.
avrdude.exe: reading input file "c:\users\winxo\Documents\Atmel Studio\7.0\MEGA_LED_EXAMPLE1\MEGA_LED_EXAMPLE1\Debug\MEGA_LED_EXAMPLE1.hex"
avrdude.exe: writing flash (158 bytes):
Writing | ################################################## | 100% 0.15s
Reading | ################################################## | 100% 0.11s
avrdude.exe: verifying ...
avrdude.exe: 158 bytes of flash verified
avrdude.exe: safemode: Fuses OK (H:05, E:DA, L:FF)
avrdude.exe done.  Thank you.
```

我们发现读写的速度特别的快。烧写完成后并没有发生什么事情，我们回头看代码：

```c
	button_state = ioport_get_pin_level(GPIO_PUSH_BUTTON_0);
	if(button_state){
		LED_Off(LED0);
	}else{
		LED_On(LED0);
	}
```
因为我们代码里面写的是按下8号口上的按钮，灯才亮。

至此，我们就可以欢快的使用C来做自己想做的事了，当然要提高水平的化，还是看看人家ASF库给我们提供了什么，还有自身架构是怎么实现的吧。

## 全文完

winxos 2016-12-02

# 补充内容

## AtmelStudio7 免烧写器方式

今日youtube找到一些教程，里面比较详细的介绍了AtmelStudio7的使用，里面配置了不用USBASP进行Arduino的烧写，比较方便，所以进行一下补充。

原理很简单，因为Avrdude也支持Arduino写入协议，所以只用改一下上文中的avrdude参数就可以了。

youtube上有一系列很好的内容，地址在下方：

https://www.youtube.com/watch?v=8aMsJWpXyE8

https://www.youtube.com/watch?v=zEbSQaQJvHI

根据视频内容，可以进行以下修改：

![atmelstudio7 2](atmelstudio7 2.png)

添加arduino菜单，其中command内容同上文usbasp一致，

D:\arduino-1.6.9\hardware\tools\avr\bin\avrdude.exe

路径替换为你自己的路径，

Arguments内容如下：

-C "D:\arduino-1.6.9\hardware\tools\avr\etc\avrdude.conf" **-c arduino** -p atmega328p -b 115200 **-P\\.\pics\COM14** -Uflash:w:$(TargetDir)$(TargetName).hex:i

其中加粗部分为修改内容，

其一修改协议为 arduino

其二添加串口-P\\.\pics\COM14 此处要将串口改为自己的串口号。

就完成了工具的配置。

我们不采用asf框架直接新建一个gcc空工程，全部代码如下：

``` c
/*
 * GccApplication3.c
 *
 * Created: 2017-1-3 8:43:23
 * Author : winxos
 */ 
#define F_CPU 16000000
#include <avr/io.h>
#include <util/delay.h>

int main(void)
{
    /* Replace with your application code */
	DDRB =0xff;
    while (1) 
    {
		PORTB|=0x20;
		_delay_ms(500);
		PORTB&=~0x20;
		_delay_ms(50);
    }
}
```

其中DDRB为设置PORTB口方向，13号引脚为PB5，编译完成，再用arduino方式下载：

> avrdude.exe: AVR device initialized and ready to accept instructions
>
> Reading | ################################################## | 100% 0.00s
>
> avrdude.exe: Device signature = 0x1e950f
> avrdude.exe: NOTE: "flash" memory has been specified, an erase cycle will be performed
>
> To disable this feature, specify the -D option.
>
> avrdude.exe: erasing chip
> avrdude.exe: reading input file "c:\users\winxo\Documents\Atmel Studio\7.0\GccApplication3\GccApplication3\Debug\GccApplication3.hex"
> avrdude.exe: writing flash (178 bytes):
>
> Writing | ################################################## | 100% 0.04s
>
> avrdude.exe: 178 bytes of flash written
> avrdude.exe: verifying flash memory against c:\users\winxo\Documents\Atmel Studio\7.0\GccApplication3\GccApplication3\Debug\GccApplication3.hex:
> avrdude.exe: load data flash data from input file c:\users\winxo\Documents\Atmel Studio\7.0\GccApplication3\GccApplication3\Debug\GccApplication3.hex:
> avrdude.exe: input file c:\users\winxo\Documents\Atmel Studio\7.0\GccApplication3\GccApplication3\Debug\GccApplication3.hex contains 178 bytes
> avrdude.exe: reading on-chip flash data:
>
> Reading | ################################################## | 100% 0.03s
>
> avrdude.exe: verifying ...
> avrdude.exe: 178 bytes of flash verified
>
> avrdude.exe: safemode: Fuses OK (H:00, E:00, L:00)
>
> avrdude.exe done.  Thank you.

我们可以发现速度更快了，因为总共只有178字节的代码写入。

补充内容结束。

winxos

2017-01-03
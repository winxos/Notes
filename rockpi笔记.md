# rockpi笔记

wvv 20200325

未特殊说明，硬件为rockpi 4b环境, 板载emmc版本，官方帮助参考https://wiki.radxa.com/Rockpi4

### 系统安装

准备工作，双公头usb线一根

**烧录工具**

链接：https://pan.baidu.com/s/1WODN6g3GvHYEQzM2JhCJTA 提取码：ocj7

**bin文件**

链接：https://pan.baidu.com/s/1-cB_G1vqnxaxe7M-ZBMniA 提取码：ydhj

**系统镜像文件**

https://wiki.radxa.com/Rockpi4/downloads

**瑞芯微USB驱动（rk3399）**

链接：https://pan.baidu.com/s/1GjjbHuOY8Kar44z5PtoZ_w 提取码：b9gm

**安装步骤**

1. 电脑装好rk3399usb驱动，重启电脑，经过测试，win11系统驱动不能识别。

2. usb线接入开发板usb3.0端口的上层端口，另一端接入电脑。

3. 按住hdmi接口下方的maskrom按键（靠近白色端子一侧那个），接入电源。

4. 打开烧入工具，会识别到设备，如果未识别到，检查前面3个步骤。第一栏选择下载的bin文件，第二栏选择下载的镜像文件，点击run，开始烧录。

5. 等待烧录完成。

6. 移除usb线，重新上电，系统完成启动。如果安装的是debian系统，系统默认用户名rock 密码rock

### 连接wifi

```sh
sudo su
nmcli r wifi on
nmcli dev wifi
nmcli dev wifi connect "wifiname" password "wifipassword"
```

连上网后，默认ssh已经开启，可以直接接入

### 更换源

确定好自己系统的版本，替换为相应的源

sudo nano /etc/apt/sources.list

buster 版阿里云源如下

```sh
deb http://mirrors.aliyun.com/debian/ buster main non-free contrib
deb http://mirrors.aliyun.com/debian-security buster/updates main
deb http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib
deb http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib
 
deb-src http://mirrors.aliyun.com/debian-security buster/updates main
deb-src http://mirrors.aliyun.com/debian/ buster main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib
```

deb-src源码可以不设置

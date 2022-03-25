# tinkerboard笔记

wvv 20200325

未特殊说明，硬件为rockpi 4b环境, 板载emmc版本，官方帮助参考http://tinkerboard.cn

### 系统安装

参考 http://tinkerboard.cn/thread-137-1-1.html

准备工作，usb type-c线一根

**烧录工具etcher**

链接：https://pan.baidu.com/s/1krfzQGP-1XVXrn7lcn_dQQ 提取码：l6te

**系统镜像文件**

https://www.asus.com.cn/Networking-IoT-Servers/AIoT-Industrial-Solutions/Tinker-Board-Series/Tinker-Board-2S/HelpDesk_Download/

官网提供了android和debian镜像

**瑞芯微USB驱动（rk3399）**

链接：https://pan.baidu.com/s/1GjjbHuOY8Kar44z5PtoZ_w 提取码：b9gm

**安装步骤**

1. 电脑装好rk3399usb驱动，重启电脑，经过测试，win11系统驱动不能识别。

2. usb线接入开发板的typec端口，另一端接入电脑。接入电源。

3. 电脑提示是否需要格式化磁盘，全部取消。

4. 打开etcher，选择镜像，选择设备，开始烧录。等待烧录完成。

5. 移除usb线，重新上电，系统完成启动。如果安装的是debian系统，系统默认用户名linaro密码linaro

### 连接wifi

```sh
sudo su
nmcli r wifi on
nmcli dev wifi
nmcli dev wifi connect "wifiname" password "wifipassword"
```

连上网后，默认ssh已经开启，可以直接接入

# CentOS笔记
> wvv 20191212
### 安装
镜像下载
http://mirrors.aliyun.com/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1908.iso
安装虚拟机软件VMware
新建虚拟机，光驱导入iso镜像文件
默认参数将安装最小系统
自带源已经是国内源，无需再设置

### 常用命令
| 命令                                    | 说明                           |
| --------------------------------------- | ------------------------------ |
| yum -y update                           | 更新系统                       |
| /etc/sudoers                            | 把用户添加到sudoer             |
| yum install vim                         | 安装vim                        |
| yum install python3                     | 安装python3                    |
| yum install net-tools                   | 安装网络工具包                 |
| ip addr                                 | 查看IP                         |
| yum groupinstall -y "GNOME Desktop"     | 安装桌面环境                   |
| systemctl set-default graphical.target  | 由命令行模式更改为图形界面模式 |
| systemctl set-default multi-user.target | 由图形界面模式更改为命令行模式 |

### 安装jdk
```
yum search java|grep jdk
sudo yum install java-1.8.0-openjdk
```
### 安装ActiveMQ
下载最新二进制包
`wget https://mirrors.tuna.tsinghua.edu.cn/apache/activemq/5.15.11/apache-activemq-5.15.11-bin.tar.gz`
解压缩
`tar -zxvf apache-activemq-5.15.11-bin.tar.gz`
进入bin文件夹，启动
`./activemq start`
查看启动情况
`netstat -anp|grep 61616`

### 设置防火墙
查看状态
`firewall-cmd --state`
关闭防火墙
`systemctl stop firewalld.service`
关闭自动启动
`systemctl disable firewalld.service`

### 安装nginx
官网下载最新文件
`wget https://nginx.org/download/nginx-1.16.1.tar.gz`
解压
`tar -zxvf nginx-1.16.1.tar.gz `
安装依赖库
`sudo yum -y install zlib zlib-devel pcre pcre-devel`
进入解压目录，依次sudo执行
```
./configure
make
make install
```
以上操作后Nginx将被安装在/usr/local/nginx/sbin目录下
默认配置文件位于目录
**/usr/local/nginx/conf/nginx.conf**
进入/usr/local/nginx/sbin
启动nginx
`sudo ./nginx`
退出nginx
`./nginx -s quit`
重载配置文件
`./nginx -s reload`
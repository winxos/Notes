### Yolo v5配置（wsl，ubuntu）

1. 安装好pip，配置好国内源

```
sudo pip config --global set global.index-url https://mirrors.aliyun.com/pypi/simple/
sudo pip config --global set install.trusted-host mirrors.aliyun.com
```



2. 配置wsl 代理，（github下载慢，用于git加速）

`export ALL_PROXY="http://172.30.80.1:4780"`

windows上开启代理工具，ip和端口替换成windows上对应参数。

3. 开始下载

`git clone https://github.com/ultralytics/yolov5.git`

4. 进行安装

```
cd yolov5/
pip3 install -U -r requirements.txt
```

安装的时候可能会出现警告，体制 xx不在路径，将其添加到路径就可以了，操作如下

```
vim ~/.bashrc
```

添加 

```
export PATH=/home/dragon/.local/bin/:$PATH
```

到最后一行, 执行下面命令更新

```
source ~/.bashrc  
```

5. 示例代码

在yolov5文件夹根目录下，执行

```
python3 detect.py
```

将会运行测试代码，代码会从github下载数据，所以还需要保持科学上网。

运行完成后，根据提示，可以发现结果存于`runs/detect/exp2`目录

6. 其他

利用mobaxterm连接wsl时，会有自动连接选项，但是通过这种方式连接的没有sftp连接，无法进行文件可视化管理，所以还是建议安装ssh服务，然后通过ssh的方式连接，这样就可以实现文件的可视化操作。


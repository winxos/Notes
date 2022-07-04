# esp模块使用

### esp8266系列

esp-15f

```shell
AT+GMR
AT version:1.7.1.0(Jul 15 2019 16:58:04)
SDK version:3.0.1(78a3e33)
compile time:Feb 14 2020 09:39:03
OK
```

```shell
AT+CWJAP="yfzx-d","11111111"
WIFI CONNECTED
WIFI GOT IP

OK
```

```shell
AT+CIFSR
+CIFSR:STAIP,"192.168.3.48"
+CIFSR:STAMAC,"4c:75:25:02:46:a7"

OK
```

```shell
AT+CIPSTART="TCP","192.168.3.2",999
CONNECT
AT+CIPMODE=1

OK
AT+CIPSEND

OK
>
```
```shell
GET /sample/sign?card_code=100002233444 HTTP/1.1


```
退出透传模式
```shell
+++
```
### esp32系列



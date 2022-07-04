# esp模块使用
https://docs.ai-thinker.com/esp8266/examples/at_demo

https://docs.ai-thinker.com/%E5%9B%BA%E4%BB%B6%E6%B1%87%E6%80%BB

https://docs.ai-thinker.com/%E4%BA%8C%E6%AC%A1%E5%BC%80%E5%8F%91%E8%BD%AF%E4%BB%B6%E8%B5%84%E6%96%99%E5%8F%8A%E5%90%84%E7%B1%BB%E5%BC%80%E5%8F%91%E8%B5%84%E6%96%99

https://docs.ai-thinker.com/%E5%AE%98%E6%96%B9%E6%89%8B%E5%86%8C%E8%B5%84%E6%96%992

### esp8266系列

esp-15f
查看版本信息
```shell
AT+GMR
AT version:1.7.1.0(Jul 15 2019 16:58:04)
SDK version:3.0.1(78a3e33)
compile time:Feb 14 2020 09:39:03
OK
```
连接wifi
```shell
AT+CWJAP="yfzx-d","11111111"
WIFI CONNECTED
WIFI GOT IP

OK
```
查看ip
```shell
AT+CIFSR
+CIFSR:STAIP,"192.168.3.48"
+CIFSR:STAMAC,"4c:75:25:02:46:a7"

OK
```
连接http tcp，开启透传模式
```shell
AT+CIPSTART="TCP","192.168.3.2",999
CONNECT
AT+CIPMODE=1

OK
AT+CIPSEND

OK
>
```
请求http get， 注意需要两个换行。
```shell
GET /sample/sign?card_code=100002233444 HTTP/1.1


```
退出透传模式
```shell
+++
```
### esp32系列



# workerman啟動失敗

## 現象1
啟動後報錯類似如下：
```php
php start.php start
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (Address already in use) in ...workerman/Worker.php on line xxxx

```
**關鍵詞**： ```Address already in use```

**根本原因**: 端口被佔用，無法啟動。

#### 解決方案1

可以通過命令```netstat -anp | grep 端口號```來找出哪個程式佔用了端口。
然後停止對應的程式釋放端口解決。

#### 解決方案2
如果不能停止對應端口的程式，可以通過更換workerman的端口解決。

#### 解決方案3
如果是Workerman佔用的端口，又無法通過stop命令停止(一般是丟失pid文件或者主進程被開發者kill了導致)，可以通過運行以下兩個命令殺死Workerman進程。

```shell
killall php
ps aux|grep -i workerman|awk '{print $2}'|xargs kill -9
```

#### 解決方案4

如果確實沒有程式監聽這個端口，那麼可能是開發者在workerman裡設置了兩個或兩個以上的監聽，並且監聽的端口相同導致，請開發者自行檢查啟動腳本是否監聽了相同的端口。

#### 解決方案5

檢查程式是否開啟了reusePort，關閉reusePort試下。

## 現象2
啟動後報錯類似如下：
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxx (Cannot assign requested address) in ...workerman/Worker.php on line xxxx
```
或者
```shell
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (在其上下文中，該請求的地址無效) in ...workerman/Worker.php on line xxxx
```
**關鍵詞：** `Cannot assign requested address`或者`該請求的地址無效`

**失敗原因：**

啟動腳本監聽ip參數寫錯，不是本機ip，請填寫本機ip機或者填寫 ```0.0.0.0```（表示監聽本機所有ip）即可解決。

**提示**：Linux系統可以通過命令 ```ifconfig```查看本機所有網卡ip。
如果您是雲伺服器(阿里雲/腾訊雲等)用戶，注意您的公網ip實際可能是個代理ip(例如阿里雲的專有網路)，公網ip並不屬於當前的伺服器，所以無法通過公網ip監聽。雖然不能用公網ip監聽，但是仍然可以通過 0.0.0.0 來綁定。

## 現象3
```php
Waring stream_socket_server has been disabled for security reasons in ...
```
**失敗原因：**

stream_socket_server 函數被php.ini禁用

**解決方法**

1、運行```php --ini``` 找到php.ini文件

2、打開php.ini找到disable_functions一項，將stream_socket_server禁用項刪掉

## 現象4
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://0.0.0.0:xxx (Permission denied)
```
**失敗原因**

linux下監聽端口如果小於1024，需要root權限。

**解決辦法**

使用大於1024的端口或者使用root用戶啟動服務。

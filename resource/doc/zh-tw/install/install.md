# 安裝說明
WorkerMan實際上是一個PHP程式包，如果你的PHP環境已經裝好，只需要把WorkerMan源代碼或者demo下載下來即可運行。

**Composer安裝：**
```sh
composer require workerman/workerman
```

> **注意**
> 有些composer代理鏡像不全，使用以上命令`composer config -g --unset repos.packagist` 移除代理

# windows用戶（必讀）

從workerman3.5.3版開始workerman已經可以同時支持windows和linux系統。
windows用戶需要配置下php環境變量。

 ` ===本頁面以下僅適用於Linux環境workerman，windows用戶請忽略=== `

# Linux系統環境檢測
Linux系統可以使用以下腳本測試本機PHP環境是否滿足WorkerMan運行要求。
 `curl -Ss https://www.workerman.net/check | php`

上面腳本如果全部顯示ok，則代表滿足WorkerMan要求，直接到[官網](https://www.workerman.net/)下載例子即可運行。

如果不是全部ok，則參考下面文檔安裝缺失的擴展即可。

（注意：檢測腳本中沒有檢測event擴展，如果業務併發連接數大於1024必須安裝event擴展，並且[優化Linux內核](../appendices/kernel-optimization.md)，擴展安裝方法參照下面說明）

# 已有PHP環境安裝缺失擴展

## 安裝pcntl和posix擴展：

**centos系統**
如果php是通過yum安裝的，則命令列運行 ```yum install php-process```即可安裝pcntl和posix擴展。

如果安裝失敗或者php本身不是用yum安裝的請參考手冊[附錄-安裝擴展](../appendices/install-extension.md)一節中方法三源碼編譯安裝。

**debian/ubuntu/mac os系統**
參考手冊[附錄-安裝擴展](../appendices/install-extension.md)一節中方法三源碼編譯安裝。


## 安裝event擴展：
為了能支持更大的併發連接數，必須安裝event擴展，並且[優化Linux內核](../appendices/kernel-optimization.md)。安裝方法如下:

**centos系統**

1、安裝event擴展依賴的libevent-devel包，命令列運行
```shell
yum install libevent-devel -y
# 如果無法安裝，嘗試使用下面的命令
# yum install libevent2-devel -y
```

2、安裝event擴展，命令列運行
(event擴展要求PHP>=5.4)
```shell
pecl install event
```
注意提示：```Include libevent OpenSSL support [yes] :``` 時輸入```no```回車，其它直接敲回車就行。

3、運行```php --ini```找到並打開php.ini文件，在最後一行加入如下配置
```shell
extension=event.so
```

**debian/ubuntu系統安裝**

1、安裝event擴展依賴的libevent-dev包，命令列運行
```shell
apt-get install libevent-dev -y
# 如果無法安裝，請嘗試以下命令
# apt-get install libevent2-dev -y
```

2、安裝event擴展，命令列運行
```shell
pecl install event
```
注意提示：```Include libevent OpenSSL support [yes] :``` 時輸入```no```回車，其它直接敲回車就行。

3、運行```php --ini```找到並打開php.ini文件，在最後一行加入如下配置
```shell
extension=event.so
```

**mac os 系統安裝教程**

mac 系統一般作為開發機，不必安裝event擴展。

# 全新系統安裝（全新安裝PHP+擴展）

## centos系統安裝教程

1、命令列運行（此步驟包含了安裝php-cli主程序以及pcntl、posix、libevent庫及git程序）
```shell
yum install php-cli php-process git gcc php-devel php-pear libevent-devel -y
```

2、安裝event擴展，命令列運行
(注意：event擴展要求PHP>=5.4)
```shell
pecl install event
```
注意提示：```Include libevent OpenSSL support [yes] :``` 時輸入```no```回車，其它直接敲回車就行。

3、運行```php --ini```找到並打開php.ini文件，在最後一行加入如下配置
```shell
extension=event.so
```

4、命令列運行（此步驟是通過github下載WorkerMan主程序）
```shell
git clone https://github.com/walkor/Workerman
```

5、參考[入門指引--簡單開發實例部分](../getting-started/simple-example.md)寫入口文件運行。
或者從[官網](https://www.workerman.net/)下載打包好的demo運行。


## debian/ubuntu系統安裝教程

1、命令列運行（此步驟包含了安裝php-cli主程序、libevent庫及git程序）
```shell
apt-get install php-cli git gcc php-pear php-dev libevent-dev -y
```

2、安裝event擴展，命令列運行
(注意：event擴展要求PHP>=5.4)
```shell
pecl install event
```
注意提示：```Include libevent OpenSSL support [yes] :``` 時輸入```no```回車，其它直接敲回車就行。

3、運行```php --ini```找到並打開php.ini文件，在最後一行加入如下配置
```shell
extension=event.so
```

4、命令列運行（此步驟是通過github下載WorkerMan主程序）
```shell
git clone https://github.com/walkor/Workerman
```

5、參考[入門指引--簡單開發實例部分](../getting-started/simple-example.md)寫入口文件運行。
或者從[官網](https://www.workerman.net/)下載打包好的demo運行。

## mac os 系統安裝教程
**方法1：** mac系統自帶PHP Cli，但是可能缺少```pcntl```擴展。

1、參考手冊[附錄-安裝擴展](../appendices/install-extension.md)一節中方法三源碼編譯安裝```pcntl```擴展。

2、參考手冊[附錄-安裝擴展](../appendices/install-extension.md)一節中方法四利用phpize安裝```event```擴展（作為開發機此可省略）。

3、通過https://www.workerman.net/download/workermanzip 下載WorkerMan主程序，或者到[官網](https://www.workerman.net/)下載例子運行。

**方法2：** 通過```brew```命令安裝php及對應擴展

1、命令列運行以下命令安裝```brew```工具(如果已經安裝過```brew```可以跳過此步驟)
```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2、命令列運行以下命令安裝```php```
```shell
brew install php
```

3、命令列運行以下命令安裝```event```擴展
```shell
brew install php-event    
```

4、到[官網](https://www.workerman.net/)下載例子運行


# Event擴展說明
[Event擴展](https://php.net/manual/zh/book.event.php)不是必須的，當業務需要支撐大於1000的併發連接時，推薦安裝Event，能夠支持巨大的併發連接。如果業務併發連接比較低，例如1000以下併發連接，則可以不用安裝。

## 常見問題
1、如果出現如下報錯 `checking for include/event2/event.h... not found`，請先嘗試刪除libevent-dev(el)庫安裝libevent2-dev(el)。
centos系統：yum remove libevent-devel && yum install libevent2-devel
debian/ubuntu系統：apt-get remove libevent-dev && apt-get install libevent2-dev

2、如果出現如下報錯`NOTICE: PHP message: PHP Warning: PHP Startup: Unable to load dynamic library '.../event.so' - ..../event.so: undefined symbol: php_sockets_le_socket in Unknown on line 0`。
請更改event.so 和socket.so的加載順序，既在php.ini中將 `extension=socket.so` 寫在 `extension=event.so` 前面，讓socket擴展先加載。

# 環境要求

## Windows用戶
workerman自3.5.3版本開始，已能同時支援Linux系統和Windows系統。

1、需要PHP >= 5.4，並配置好PHP的環境變量。

2、Windows版本的Workerman不依賴任何擴展。

3、安裝使用以及使用限制[**這裡**](https://www.workerman.net/windows)。

4、由於Workerman在Windows下有諸多使用限制，所以正式環境建議使用Linux系統，Windows系統僅建議用於開發環境。

 ```====本頁面以下只適用於Linux用戶，Windows用戶請忽略。====```

## Linux用戶（含Mac OS）
Linux用戶只能使用Linux版本的Workerman。

1、安裝PHP>=5.4，並安裝了pcntl、posix擴展

2、建議安裝event擴展，但不是必須的（注意event擴展需要PHP>=5.4）

### Linux環境檢查腳本
Linux用戶可以運行以下腳本檢查本地環境是否滿足WorkerMan要求

```curl -Ss https://www.workerman.net/check | php```

如果腳本中全部提示ok，則代表滿足WorkerMan運行環境

（注意：檢測腳本中沒有檢測event擴展，如果並發連接數大於1024建議安裝event擴展，安裝方法參見下一節）

## 詳細說明

### 關於PHP-CLI

WorkerMan是基於[PHP命令行(PHP-CLI)](https://php.net/manual/zh/features.commandline.php)模式運行的。PHP-CLI與PHP-FPM或者Apache的MOD-PHP是獨立的可執行程序，它們之間並不衝突也不會有相互依賴，完全獨立。

### 關於WorkerMan依賴的擴展

1、[pcntl擴展](https://cn2.php.net/manual/zh/book.pcntl.php)

pcntl擴展是PHP在Linux環境下進程控制的重要擴展，WorkerMan用到了其[進程創建](https://cn2.php.net/manual/zh/function.pcntl-fork.php)、[信號控制](https://cn2.php.net/manual/zh/function.pcntl-signal.php)、[定時器](https://cn2.php.net/manual/zh/function.pcntl-alarm.php)、[進程狀態監控](https://cn2.php.net/manual/zh/function.pcntl-waitpid.php)等特性。此擴展win平臺不支持。

2、[posix擴展](https://cn2.php.net/manual/zh/book.posix.php)

posix擴展使得PHP在Linux環境可以調用系統通過[POSIX標準](https://baike.baidu.com/view/209573.htm)提供的接口。WorkerMan主要使用了其相關的接口實現了守護進程化、用戶組控制等功能。此擴展win平臺不支持。

3、[Event擴展](https://php.net/manual/zh/book.event.php) 或者 [libevent擴展](https://cn2.php.net/manual/en/book.libevent.php) 

event擴展使得PHP可以使用系統[Epoll](https://baike.baidu.com/view/1385104.htm)、Kqueue等高級事件處理機制，能夠顯著提高WorkerMan在高並發連接時CPU利用率。在高並發長連接相關應用中非常重要。libevent擴展（或者event擴展）不是必須的，如果沒安裝，則默認使用PHP原生Select事件處理機制。

## 如何安裝擴展

參見[安裝擴展](../appendices/install-extension.md)章節

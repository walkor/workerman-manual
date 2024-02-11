# 環境要求

## Windows 使用者
workerman 從 3.5.3 版本開始已經能夠同時支持 Linux 系統和 Windows 系統。

1、需要 PHP>=5.4，并配置好 PHP 的環境變數。

2、Windows 版本的 Workerman 不依賴任何擴展。

3、安裝使用以及使用限制[**這裡**](https://www.workerman.net/windows)。

4、由於 Workerman 在 Windows 下有諸多使用限制，所以正式環境建議用 Linux 系統，Windows 系統僅建議用於開發環境。

``` ====本頁面以下只適用於 Linux 使用者，Windows 使用者請忽略。 ====```

## Linux 使用者(含 Mac OS)
Linux 使用者只能使用 Linux 版本的 Workerman。

1、安裝 PHP>=5.4，並安裝了 pcntl、posix 擴展

2、建議安裝 event 擴展，但不是必須的（注意 event 擴展需要 PHP>=5.4）

### Linux 環境檢查腳本
Linux 使用者可以運行以下腳本檢查本地環境是否滿足 Workerman 要求

```curl -Ss https://www.workerman.net/check | php```

如果腳本中全部提示ok，則代表滿足 Workerman 運行環境

（注意：檢測腳本中沒有檢測 event 擴展，如果並發連接數大於 1024 建議安裝 event 擴展，安裝方法參見下一節）

## 詳細說明

### 關於 PHP-CLI

Workerman 是基於[PHP 命令行(PHP-CLI)](https://php.net/manual/zh/features.commandline.php)模式運行的。PHP-CLI 與 PHP-FPM 或者 Apache 的 MOD-PHP 是獨立的可執行程序，它們之間並不衝突也不會有相互依賴，完全獨立。

### 關於 Workerman 依賴的擴展

1、[pcntl 擴展](https://cn2.php.net/manual/zh/book.pcntl.php)

pcntl 擴展是 PHP 在 Linux 環境下進程控制的重要擴展，Workerman 用到了其[進程創建](https://cn2.php.net/manual/zh/function.pcntl-fork.php)、[信號控制](https://cn2.php.net/manual/zh/function.pcntl-signal.php)、[定時器](https://cn2.php.net/manual/zh/function.pcntl-alarm.php)、[進程狀態監控](https://cn2.php.net/manual/zh/function.pcntl-waitpid.php)等特性。此擴展在 Windows 平臺不支持。

2、[posix 擴展](https://cn2.php.net/manual/zh/book.posix.php)

posix 擴展使得 PHP 在 Linux 環境可以調用系統通過[POSIX 標準](https://baike.baidu.com/view/209573.htm)提供的接口。Workerman 主要使用了其相關的接口實現了守護進程化、使用者組控制等功能。此擴展在 Windows 平臺不支持。

3、 [Event 擴展](https://php.net/manual/zh/book.event.php) 或者 [libevent 擴展](https://cn2.php.net/manual/en/book.libevent.php) 

event 擴展使得 PHP 可以使用系統[Epoll](https://baike.baidu.com/view/1385104.htm)、Kqueue 等高級事件處理機制，能夠顯著提高 Workerman 在高並發連接時 CPU 利用率。在高並發長連接相關應用中非常重要。libevent 擴展(或者 event 擴展)不是必須的，如果沒安裝，則默認使用 PHP 原生 Select 事件處理機制。

## 如何安裝擴展

參見[安裝擴展](../appendices/install-extension.md)章節

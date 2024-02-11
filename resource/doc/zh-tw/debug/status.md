# 查看執行狀態

執行```php start.php status``` 可以查看 Workerman 的執行狀態，類似以下:

```plaintext
----------------------------------------------全域狀態-----------------------------------------------------------
Workerman 版本:3.5.13          PHP 版本:5.5.9-1ubuntu4.24
啟動時間:2018-02-03 11:48:20   已運行 112 天 2 小時  
負載均衡值: 0, 0, 0            事件循環:\Workerman\Events\Event
4 個 worker       11 個進程
worker_name        exit_status      exit_count
ChatBusinessWorker 0                0
ChatGateway        0                0
Register           0                0
WebServer          0                0
----------------------------------------------進程狀態----------------------------------------------------
pid	  記憶體  監聽地址                 worker_name        連線數  發送失敗  定時器  總請求數  qps    狀態
18306	2.25M   無                       ChatBusinessWorker 5       0    0       11         0      [閒置]
18307	2.25M   無                       ChatBusinessWorker 5       0    0       8          0      [閒置]
18308	2.25M   無                       ChatBusinessWorker 5       0    0       3          0      [閒置]
18309	2.25M   無                       ChatBusinessWorker 5       0    0       14         0      [閒置]
18310	2M      websocket://0.0.0.0:7272 ChatGateway        8       0    1       31         0      [閒置]
18311	2M      websocket://0.0.0.0:7272 ChatGateway        7       0    1       26         0      [閒置]
18312	2M      websocket://0.0.0.0:7272 ChatGateway        6       0    1       21         0      [閒置]
18313	1.75M   websocket://0.0.0.0:7272 ChatGateway        5       0    1       16         0      [閒置]
18314	1.75M   text://0.0.0.0:1236      Register           8       0    0       8          0      [閒置]
18315	1.5M    http://0.0.0.0:55151     WebServer          0       0    0       0          0      [閒置]
18316	1.5M    http://0.0.0.0:55151     WebServer          0       0    0       0          0      [閒置]
----------------------------------------------進程狀態----------------------------------------------------
總計	18M     -                      -                  54      0    4       138        0      [總結]
```

## 說明

### 全域狀態

從這個欄中我們可以看到：

Workerman的版本 ```version:3.5.13```。

啟動時間 ```2018-02-03 11:48:20```，已運行了 ```run 112 天 2 小時```。

伺服器負載 ```load average: 0, 0, 0```，分別是最近1分鐘、5分鐘、15分鐘內系統的平均負載。

使用的IO事件庫，```event-loop:\Workerman\Events\Event```。

```4 個 worker```（3 種進程，包括 ChatGateway、ChatBusinessWorker、Register進程、WebServer進程）。

```11 個進程```（共11個進程）。

```worker_name```（worker進程名）。

```exit_status```（worker進程退出狀態碼）。

```exit_count```（該狀態碼的退出次數）。

一般來說，exit_status為0表示正常退出，如果為其它值，代表進程是異常退出的，並產生一條類似 ```WORKER EXIT UNEXPECTED``` 錯誤訊息，錯誤訊息會記錄到 [Worker::logFile](worker/log-file.md) 指定的文件中。

**常見的 exit_status 及其含義如下：**

* 0：表示正常退出，在reload平滑重啟後會出現值為0的退出碼，是正常現象。注意在程式中調用exit或die也會導致退出碼為0，並產生一條 ```WORKER EXIT UNEXPECTED``` 錯誤訊息，workerman中不允許業務程式碼調用exit或者die語句。
* 9：表示進程被SIGKILL信號殺死了。這個退出碼主要發生在stop以及reload平滑重啟時，導致這個退出碼的原因是由於子進程沒有在規定時間內響應主進程reload信號(例如mysql、curl等長時間阻塞等待或者業務死循環等)，被主進程強制使用SIGKILL信號殺死。注意，當在linux命令行中使用kill命令發送SIGKILL信號給子進程也會導致這個退出碼。
* 11：表示php發生了core dump，一般是使用了不穩定的擴展導致，請在php.ini中把對應擴展註釋掉；另外有少數情況是php的bug，這時需要升級php。 
* 65280：導致這個退出碼的原因是業務程式碼有致命錯誤，例如調用了不存在的函數、語法錯誤等，具體錯誤訊息會記錄到 [Worker::logFile](worker/log-file.md) 指定的文件中，也可以在 [php.ini](https://php.net/manual/zh/ini.list.php) 中 [error_log](https://php.net/manual/zh/errorfunc.configuration.php#ini.error-log) 指定的文件中(如果有指定的話)找到。
* 64000：導致這個退出碼的原因是業務程式碼拋出了異常，但業務沒有捕獲這個異常，導致進程退出。如果workerman以debug方式運行時異常調用堆疊會打印到端口，daemon方式運行時異常調用堆疊會記錄到 [Worker::stdoutFile](worker/stdout-file.md) 指定的文件中。

## 進程狀態

pid：進程pid。

memory：該進程目前占用內存（不包括php自身可執行文件的占用的內存）。

listening：傳輸層協議及監聽ip端口。如果不監聽任何端口則顯示無。參見[Worker類構造函數](worker/construct.md)。

worker_name：該進程執行的服務名稱，見[Worker類name屬性](worker/name.md)。

connections：該進程當前有多少個TCP連線實例，連線實例包括TcpConnection和AsyncTcpConnection實例。這個值是實時數值，並非累計值。注意：當連線實例調用close後，如果相應計數沒有相應減少，可能是業務程式碼保存了$connection對象，導致這個連線實例無法銷毀。

total_request：表示該進程從啟動到現在一共接收了多少個請求。這裡的請求數不僅包含客戶端發來的請求，也包含Workerman內部通訊請求，例如GatewayWorker架構裡Gateway與BusinessWorker之間的通訊請求。這個值是累計值。

send_fail：該進程向客戶端發送數據失敗次數，失敗原因一般為客戶端連線斷開，此項不為0一般屬於正常狀態，參見[status裡send_fail的原因](../faq/about-send-fail.md)。這個值是累計值。

timers：該進程活動的定時器數量（不包括被刪除的定時器以及已經運行過的一次性定時器）。注意：此特性需要workerman版本>=3.4.7。這個值是實時數值，並非累計值。

qps：當前進程每秒收到的網絡請求數，注意：只有status時加```-d```才會統計此選項，否則顯示為0。此特性需要workerman版本>=3.5.2。這個值是實時數值，並非累計值。

status：進程狀態，如果是idle代表空閒，如果是busy代表是繁忙。注意：如果進程進入短暫的繁忙是正常情況，如果進程一直是繁忙狀態，則有可能發生了業務阻塞或者業務死循環的情況，需要根據[調試busy進程](busy-process.md)一節排查。注意：此特性需要workerman版本>=3.5.0。

## 原理
status腳本運行後，主進程會向所有worker進程發送一個```SIGUSR2```信號，隨後status腳本進入短暫的睡眠階段，以便等待各個worker進程狀態統計結果。這時空閒的worker進程收到```SIGUSR2```信號後會立刻向特定的磁碟文件寫入自己的運行狀態（連線數、請求數等等），而正在處理業務邏輯的worker進程，則會等待業務邏輯處理完畢才會去寫入自己的狀態訊息。短暫睡眠後，status腳本開始讀取磁碟中的狀態文件，並展示結果到控制台。

## 注意
執行status時可能會發現有些進程顯示busy，原因是由於進程忙於處理業務（例如業務邏輯長時間阻塞在curl或者資料庫請求上，或者運行大的迴圈），無法將狀態上報，導致顯示busy。

出現這種問題需要排查業務代碼，看哪裡導業務致長時間阻塞，並且評估阻塞耗時是否在預期內，如果不符合預期需要根據[調試busy進程](busy-process.md)一節排查業務代碼。

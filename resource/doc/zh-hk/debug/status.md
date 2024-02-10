# 檢視運行狀態

執行 ```php start.php status```，可以檢視WorkerMan的運行狀態，類似下面所示：

``` 
----------------------------------------------GLOBAL STATUS----------------------------------------------------
Workerman版本:3.5.13          PHP版本:5.5.9-1ubuntu4.24
啟動時間:2018-02-03 11:48:20   已運行 112 天 2 小時   
負載均衡: 0, 0, 0            事件循環:\Workerman\Events\Event
4 個 worker       11 個進程
worker名稱        退出狀態      退出次數
ChatBusinessWorker 0                0
ChatGateway        0                0
Register           0                0
WebServer          0                0
----------------------------------------------PROCESS STATUS---------------------------------------------------
pid	內存  監聽                    worker名稱        連接失敗數計時器  總請求 qps    狀態
18306	2.25M   none                     ChatBusinessWorker 5           0         0       11            0      [idle]
18307	2.25M   none                     ChatBusinessWorker 5           0         0       8             0      [idle]
18308	2.25M   none                     ChatBusinessWorker 5           0         0       3             0      [idle]
18309	2.25M   none                     ChatBusinessWorker 5           0         0       14            0      [idle]
18310	2M      websocket://0.0.0.0:7272 ChatGateway        8           0         1       31            0      [idle]
18311	2M      websocket://0.0.0.0:7272 ChatGateway        7           0         1       26            0      [idle]
18312	2M      websocket://0.0.0.0:7272 ChatGateway        6           0         1       21            0      [idle]
18313	1.75M   websocket://0.0.0.0:7272 ChatGateway        5           0         1       16            0      [idle]
18314	1.75M   text://0.0.0.0:1236      Register           8           0         0       8             0      [idle]
18315	1.5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [idle]
18316	1.5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [idle]
----------------------------------------------PROCESS STATUS---------------------------------------------------
總結	18M     -                        -                  54          0         4       138           0      [總結]
```

## 說明

### GLOBAL STATUS

從這部分中，我們可以看到：

WorkerMan版本```version:3.5.13```。

啟動時間```2018-02-03 11:48:20```，已運行```run 112 天 2 小時```。

伺服器負載```load average: 0, 0, 0```，分別是最近1分鐘、5分鐘和15分鐘內系統的平均負載。

使用的IO事件庫，```event-loop:\Workerman\Events\Event```。

```4 個 worker```（3類進程，包括ChatGateway、ChatBusinessWorker、Register進程、WebServer進程）。

```11 個進程```（共11個進程）。

```worker_name```（worker進程名）。

```exit_status```（worker進程退出狀態碼）。

```exit_count```（該狀態碼的退出次數）。

一般情況下，exit_status為0表示正常退出，如果為其他值，代表進程是異常退出的，並且會產生一條類似```WORKER EXIT UNEXPECTED```的錯誤信息，錯誤信息會記錄到[Worker::logFile](worker/log-file.md)指定的文件中。

**常見的exit_status及其含義如下：**

* 0：表示正常退出，運行reload平滑重啟後會出現值為0的退出碼，是正常現象。注意在程式中調用exit或die也會導致退出碼為0，並產生一條```WORKER EXIT UNEXPECTED```的錯誤信息，workerman中不允許業務碼調用exit或者die語句。
* 9：表示進程被SIGKILL信號殺死了。這個退出碼主要發生在stop以及reload平滑重啟時，導致這個退出碼的原因是由於子進程沒有在規定時間內響應主進程reload信號(例如mysql、curl等長時間阻塞等待或者業務死循環等)，被主進程強制使用SIGKILL信號殺死。注意，當在linux命令行中使用kill命令發送SIGKILL信號給子進程也會導致這個退出碼。
* 11：表示php發生了coredump，一般是使用了不穩定的擴展導致，請在php.ini中把對應擴展註釋掉；另外有少數情況是php的bug，這時需要升級php。
* 65280：導致這個退出碼的原因是業務碼有致命錯誤，例如調用了不存在的函數、語法錯誤等，具體錯誤信息會記錄到[Worker::logFile](worker/log-file.md)指定的文件中，也可以在[php.ini](https://php.net/manual/zh/ini.list.php)中[error_log](https://php.net/manual/zh/errorfunc.configuration.php#ini.error-log)指定的文件中(如果有指定的話)找到。
* 64000：導致這個退出碼的原因是業務碼拋出了異常，但業務沒有捕獲這個異常，導致進程退出。如果workerman以debug方式運行時異常調用棧會列印到終端，daemon方式運行時異常調用棧會記錄到[Worker::stdoutFile](worker/stdout-file.md)指定的文件中。

## PROCESS STATUS

pid：進程pid。

memory：該進程當前佔用內存（不包括php自身可執行檔的佔用的內存）。

listening：傳輸層協議及監聽ip端口。如果不監聽任何端口則顯示none。

worker_name：該進程運行的服務服務名，見[Worker類name屬性](worker/name.md)。

connections：該進程**當前**有多少個TCP連接實例，連接實例包括TcpConnection和AsyncTcpConnection實例。這個值是實時數值，並非累計值。注意：當連接實例調用close後，如果相應計數沒有相應減少，可能是業務碼保存了$connection物件，導致這個連接實例無法銷毀。

total_request：表示該進程從啟動到現在一共收到了多少個請求。這裡的請求數不僅包含客戶端發來的請求，也包含Workerman內部通訊請求，例如GatewayWorker架構裡Gateway與BusinessWorker之間的通訊請求。這個值是累計值。

send_fail：該進程向客戶端發送數據失敗次數，失敗原因一般為客戶端連接斷開，此項不為0一般屬於正常狀態，參見[status裡send_fail的原因](../faq/about-send-fail.md)。這個值是累計值。

timers：該進程活動的定時器數量（不包括被刪除的定時器以及已經運行過的一次性定時器）。注意：此特性需要workerman版本>=3.4.7。這個值是實時數值，並非累計值。

qps：當前進程每秒收到的網絡請求數，注意：只有status時加```-d```才會統計此選項，否則顯示0。此特性需要workerman版本>=3.5.2。這個值是實時數值，並非累計值。

status: 進程狀態，如果是idle代表空閒，如果是busy代表是繁忙。注意：如果進程進入短暫的繁忙是正常情況，如果進程一直是繁忙狀態，則有可能發生了業務阻塞或者業務死循環的情況，需要根據[調試busy進程](busy-process.md)一節排查。

## 原理

status腳本運行後，主進程會向所有worker進程發送一個```SIGUSR2```信號，隨後status腳本進入短暫的睡眠階段，以便等待各個worker進程狀態統計結果。這時空閒的worker進程收到```SIGUSR2```信號後會立刻向特定的磁盤文件寫入自己的運行狀態（連接數、請求數等等），而正在處理業務邏輯的worker進程，則會等待業務邏輯處理完畢才會去寫入自己的狀態信息。短暫睡眠後，status腳本開始讀取磁盤中的狀態文件，並展示結果到控制台。

## 注意

status時可能會發現有些進程顯示busy，原因是由於進程忙於處理業務（例如業務邏輯長時間阻塞在curl或者資料庫請求上，或者執行大的迴圈），無法將狀態上報，導致顯示busy。

出現這種問題需要排查業務碼，看哪裡導業務致長時間阻塞，並且評估阻塞耗時是否在預期內，如果不符合預期需要根據[調試busy進程](busy-process.md)一節排查業務碼。

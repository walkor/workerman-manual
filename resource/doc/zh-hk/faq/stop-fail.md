# 停止失敗

## 現象：
執行 ```php start.php stop``` 出現 ```stop fail``` 的提示。

### 第一種可能性
如果是以 debug 方式啟動的 workerman，開發者在終端按了```ctrl z```給 workerman 發送了```SIGSTOP```信號，導致 workerman 進入後台並挂起(暫停)，因此無法回應 stop 命令 (```SIGINT```信號)。

**解決：**
在啟動 workerman 的終端輸入```fg```(發送```SIGCONT```信號)，然後按下回車，將 workerman 切回前台運行，並按```ctrl c```(發送```SIGINT```信號)停止 workerman。

如果仍無法停止，請嘗試運行以下兩條命令：
``` 
killall -9 php
``` 
``` 
ps aux|grep -i workerman|awk '{print $2}'|xargs kill -9
``` 

### 第二種可能性
執行 stop 的使用者與 workerman 啟動使用者不一致，也就是 stop 使用者沒有權限停止 workerman。

**解決：**
切換到啟動 workerman 的使用者，或者使用權限更高的使用者來停止 workerman。

### 第三種可能性
workerman 主進程的 pid 文件被刪除，導致腳本找不到 pid 進程，進而停止失敗。

**解決：**
將 pid 文件保存到安全的位置，參見手冊 [Worker::$pidFile](../worker/pid-file.md)。

### 第四種可能性
workerman 主進程 pid 文件對應的進程並不是 workerman 進程。

**解決：**
打開 workerman 的主進程的 pid 文件查看主進程 pid，pid 文件默認在 Workerman 平行的目錄裡。運行命令```ps aux | grep 主進程 pid``` 查看對應的進程是否是 Workerman 進程。如果不是，可能是伺服器重啟過，導致 workerman 保存的 pid 是過期的 pid，而這個 pid 剛好被其他進程使用，導致停止失敗。如果是這種情況，將 pid 文件刪除即可。

### 第五種可能性
安裝了 grpc 擴展，但沒有給 grpc 擴展設置相應的環境變量，啟動後會多 fork 出一個掛載進程，停止時導致失敗。

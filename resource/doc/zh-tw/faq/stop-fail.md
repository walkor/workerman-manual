# 無法停止

## 現象：
執行```php start.php stop```時顯示 ```stop fail```

### 第一種可能性
假設是以 debug 方式啟動的 workerman，開發者在終端機按下```ctrl z```給 workerman 發送了```SIGSTOP```信號，導致 workerman 進入後台並掛起（暫停），因此無法響應 stop 命令（```SIGINT```信號）。

**解決：**
在啟動 workerman 的終端輸入```fg```（發送```SIGCONT```信號）然後按下 Enter，將 workerman 切回前台運行，按```ctrl c```（發送```SIGINT```信號）停止 workerman。

如果無法停止，請嘗試運行以下兩條命令
```
killall -9 php
```
```
ps aux|grep -i workerman|awk '{print $2}'|xargs kill -9
```
### 第二種可能性
執行 stop 的使用者和 workerman 啟動使用者不一致，即 stop 使用者沒有權限停止 workerman。

**解決：**
切換到啟動 workerman 的使用者，或者使用權限更高的使用者停止 workerman。



### 第三種可能性
保存 workerman 主進程 pid 檔案被刪除，導致腳本找不到 pid 進程，導致停止失敗。

**解決：**
將 pid 檔案保存到安全的位置，參見手冊[Worker::$pidFile](../worker/pid-file.md)。



### 第四種可能性
workerman 主進程 pid 檔案對應的進程不是 workerman 進程。

**解決：**
打開 workerman 的主進程 pid 檔案查看主進程 pid，pid 檔案默認在 Workerman 平行的目錄裡。運行命令```ps aux | grep 主進程pid``` 查看對應的進程是否是 Workerman 進程，如果不是，可能是伺服器重啟過，導致 workerman 保存的 pid 是過期的 pid，而這個 pid 剛好被其它進程使用，導致停止失敗。如果是這種情況，將 pid 檔案刪除即可。

### 第五種可能性
安裝了 grpc 擴展，但是沒有給 grpc 擴展設置相應的環境變數，啟動後會多 fork 出一個挂載進程，停止時導致失敗。

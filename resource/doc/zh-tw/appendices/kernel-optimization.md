# Linux內核調校

為了使系統能夠支持更大的並發，除了必須[安裝event擴展](../install/install.md)之外，優化Linux內核也是**重中之重**，以下優化**每一項**都非常非常重要，請務必逐一完成。

**參數解釋：**

> **max-file**: 表示**系統級別**的能夠打開的文件句柄的數量。是針對整個OS而言，並不是針對使用者的。
> 
> **ulimit -n**: 表示控制**進程級別**能夠打開的文件句柄的數量。針對當前`shell`的當前使用者及其啟動的進程的可用文件句柄控制。

查看**系統級別**能夠打開的文件句柄的數量： `cat /proc/sys/fs/file-max`

打開文件 /etc/sysctl.conf，增加以下設置
```conf
#該參數設置系統的TIME_WAIT的數量，如果超過默認值則會被立即清除
net.ipv4.tcp_max_tw_buckets = 20000
#定義了系統中每一個端口最大的監聽隊列的長度，這是個全局的參數
net.core.somaxconn = 65535
#對於還未獲得對方確認的連接請求，可保存在隊列中的最大數目
net.ipv4.tcp_max_syn_backlog = 262144
#在每個網絡接口接收數據包的速率比內核處理這些包的速率快時，允許送到隊列的數據包的最大數目
net.core.netdev_max_backlog = 30000
#此選項會導致處於NAT網絡的客戶端超時，建議為0。Linux從4.12內核開始移除了 tcp_tw_recycle 配置，如果報錯"No such file or directory"請忽略
net.ipv4.tcp_tw_recycle = 0
#系統所有進程一共可以打開的文件數量
fs.file-max = 6815744
#防火牆跟踪表的大小。注意：如果防火牆沒開則會提示error: "net.netfilter.nf_conntrack_max" is an unknown key，忽略即可
net.netfilter.nf_conntrack_max = 2621440
net.ipv4.ip_local_port_range = 10240 65000
```
運行 `sysctl -p` 即刻生效。

**說明：**

/etc/sysctl.conf 可設置的選項很多，其它選項可以根據自己的環境需要進行設置

## 打開文件數

設置系統打開文件數設置，解決高並發下 ```too many open files``` 問題。此選項直接影響單個進程容納的客戶端連接數。

Soft open files 是Linux系統參數，影響系統單個進程能夠打開最大的文件句柄數量，這個值會影響到長連接應用如聊天中單個進程能夠維持的使用者連接數， 運行```ulimit -n```能看到這個參數值，如果是1024，就是代表單個進程只能同時最多只能維持1024甚至更少（因為有其它文件的句柄被打開）。如果開啟4個進程維持使用者連接，那麼整個應用能夠同時維持的連接數不會超過4*1024個，也就是說最多只能支援4x1024個使用者在線可以增大這個設置以便服務能夠維持更多的TCP連接。

**Soft open files 修改三種方法：**

第一種：在終端直接運行 `ulimit -HSn 102400`，然後重啟workerman。

這只是在當前終端有效，退出之後，open files 又變為默認值。

第二種：在`/etc/profile`文件末尾添加一行 `ulimit -HSn 102400`，這樣每次登錄終端時，都會自動執行。更改後需要重啟workerman。

第三種：令修改open files的數值永久生效，則必須修改配置文件：`/etc/security/limits.conf`. 在這個文件後加上：

```
* soft nofile 1024000
* hard nofile 1024000
root soft nofile 1024000
root hard nofile 1024000
```

這種方法需要重啟伺服器才能生效。

# 客戶端連線失敗原因

通常，連接失敗的客戶端會出現兩種錯誤：```connection refuse``` 和 ```connection timeout```

## connection refuse（連線拒絕）

一般原因如下：
1. 客戶端連線的端口錯誤
2. 客戶端連線的域名或 IP 錯誤
3. 如果客戶端使用了域名連線，域名可能指向了錯誤的伺服器 IP
4. 伺服器使用了 CDN 等加速代理，導致連線的實際 IP 與預期 IP 不一致
5. 伺服器未啟動或未監聽端口
6. 使用了網路代理軟體
7. 服務端監聽 IP 與訪問地址不在同一地址段，例如伺服器監聽 127.0.0.1，那麼客戶端只能通過 127.0.0.1 連接，不能通過區域網路 IP 或外網 IP 連接。建議設置監聽地址為 0.0.0.0，這樣本機、內網、外網都可以連接。

## connection timeout（連線超時）

一般原因如下：
1. 伺服器防火牆阻止了連線，可以暫時關閉防火牆嘗試連接
2. 如果是雲伺服器，安全組也可能會阻止連接建立，需要到管理後台開放對應端口
3. 如果使用了面板，需要在面板中開放對應端口
4. 伺服器不存在或未啟動
5. 如果客戶端使用了域名連線，域名可能指向了錯誤的伺服器 IP
6. 客戶端訪問的 IP 是伺服器內網 IP，且客戶端和服務端不在同一區域網

## cannot assign requested address （無法分配請求地址）

**作為客戶端時**，每發起一個連線需要佔用本地一個臨時端口，一台伺服器默認可用臨時端口大概在 2-3 萬個，如果向特定伺服器發起的連接數超過這個值後將無法分配可用端口，會產生這個錯誤。
可以通過更改內核參數`/etc/sysctl.conf` 中的 `net.ipv4.ip_local_port_range` 來增加本地臨時端口數量，例如設置成`10000 65535`（本地端口範圍設置成 10000 到 65535，也就是本地端口數增加到 55535 個），運行`sysctl -p` 生效。
另外連線斷開後連線變成 TIME_WAIT 狀態，仍然會佔用對應本地端口一段時間，也就是短時間內發起大量（超過 2-3 萬）短連接也會報`Cannot assign requested address`，如果是這種情況可以通過設置內核快速回收 TIME_WAIT 來解決，參考[內核調優](https://www.workerman.net/doc/workerman/appendices/kernel-optimization.html)。

> **注意**
> 本地端口數限制僅限於客戶端，伺服器沒有本地端口限制，只要資源足夠，伺服器維持連接數量可以被視作無限。

## 其他錯誤
如果發生的錯誤不是```connection refuse``` 和 ```connection timeout```，則一般是以下原因：

**1、客戶端使用的通訊協議與服務端不一致。**
例如服務端是 HTTP 通訊協議，客戶端使用 WebSocket 通訊協議訪問是無法連接的。如果客戶端用 WebSocket 協議連接，那麼服務端必須也是 WebSocket 協議。如果服務端是 HTTP 協議的服務，那麼客戶端必須用 HTTP 協議訪問。

這裡的原理類似如果你要和英國人交流，那麼要使用英語。如果要和日本人交流，那麼要使用日語。這裡的語言就類似通訊協議，雙方（客戶端和服務端）必須使用相同的語言才能交流，否則無法通訊。

**通訊協議不一致導致的常見的報錯有：**

> WebSocket connection to 'ws://xxx.com:xx/' failed: Error during WebSocket handshake: Unexpected response code: xxx

> WebSocket connection to 'ws://xxx.com:xx/' failed: Error during WebSocket handshake: net::ERR_INVALID_HTTP_RESPONSE

**解決辦法：**
從上面兩條報錯看出，客戶端使用的是 ws 連接是 WebSocket 協議。服務端也需要是 WebSocket 協議才行，服務端監聽部分代碼需要指定 WebSocket 協議才能通訊，例如下面這樣

如果是 gatewayWorker，監聽部分代碼類似
```php
// WebSocket 協議，這樣客戶端才能用 ws://...來連。xxxx為端口不用改動
$gateway = new Gateway('websocket://0.0.0.0:xxxx');
```
如果是 Workerman 則是
```php
// WebSocket 協議，這樣客戶端才能用 ws://...來連。xxxx為端口不用改動
$worker = new Worker('websocket://0.0.0.0:xxxx');
```

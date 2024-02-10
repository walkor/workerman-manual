# 客戶端連接失敗原因

一般客戶端連接失敗會出現兩種錯誤，即 ```connection refuse``` 和 ```connection timeout```

## connection refuse（連接拒絕）

一般可能原因有：
1、客戶端連接的端口錯誤
2、客戶端連接的域名或IP地址錯誤
3、若客戶端使用域名連接，該域名可能指向了錯誤的伺服器IP
4、伺服器使用了CDN等加速代理，導致實際IP與預期IP不一致
5、服務器未啟動或未監聽該端口
6、使用了網絡代理軟件
7、服務端監聽IP與訪問地址不在同一地址範圍。例如服務端監聽127.0.0.1，客戶端只能通過127.0.0.1連接，無法通過區域網絡IP或者外網IP連接。建議將監聽地址設置為0.0.0.0，這樣本機、內網、外網都可以連接。

## connection timeout（連接超時）

一般可能原因有：
1、伺服器防火牆阻止連接，可以暫時關閉防火牆嘗試
2、若為雲伺服器，安全組也可能阻止連接建立，需要在管理後台開放對應端口
3、若使用了寶塔等面板，需要在寶塔中開放對應端口
4、伺服器不存在或未啟動
5、若客戶端使用域名連接，該域名可能指向了錯誤的伺服器IP
6、客戶端訪問的IP為伺服器內網IP，且客戶端和伺服端不在同一局域網

## cannot assign requested address (無法分配請求地址)

**作為客戶端時**，每發起一個連接需要佔用本地一個臨時端口，一台伺服器默認可用臨時端口大概在2-3萬個，若向特定伺服器發起的連接數超過此值將無法分配可用端口，導致出現此錯誤。可通過修改內核參數`/etc/sysctl.conf`中的`net.ipv4.ip_local_port_range`增加本地臨時端口數量，例如設置為`10000 65535`（本地端口範圍設置成10000 65535，即本地端口數量增至55535個），執行`sysctl -p`生效。另外，連接斷開後轉為TIME_WAIT狀態，仍然會佔用對應本地端口一段時間，短時間內發起大量（超過2-3萬）短連接也會出現`Cannot assign requested address`錯誤。若為這種情況，可通過設置內核快速回收TIME_WAIT來解決，參考[內核調優](https://www.workerman.net/doc/workerman/appendices/kernel-optimization.html)。

> **注意**
> 本地端口數限制僅適用於客戶端，伺服器沒有本地端口限制，只要資源足夠，伺服器維持連接數量可以視為無限。

## 其他報錯
若出現的錯誤不是```connection refuse``` 和 ```connection timeout```，一般可能原因如下：

**1、客戶端使用的通訊協議與伺服器不一致。**
例如伺服器是HTTP通訊協議，客戶端使用WebSocket通訊協議無法連接。若客戶端使用WebSocket協議連接，則伺服器必須也是WebSocket協議。若伺服器是HTTP協議的服務，客戶端必須使用HTTP協議連接。

此處的原理類似於若想與英國人交流，則需使用英語。若想與日本人交流，則需使用日語。此處的語言類似通訊協議，雙方（客戶端和伺服器）必須使用相同的語言才能進行交流，否則無法通訊。

**通訊協議不一致導致的常見錯誤包括：**

> WebSocket connection to 'ws://xxx.com:xx/' failed: Error during WebSocket handshake: Unexpected response code: xxx

> WebSocket connection to 'ws://xxx.com:xx/' failed: Error during WebSocket handshake: net::ERR_INVALID_HTTP_RESPONSE

**解決方法：**
從以上兩條錯誤可以看出，客戶端使用的是ws連接即WebSocket協議。伺服器也需要遵守WebSocket協議，所以伺服器的監聽部分代碼需指定WebSocket協議才能進行通訊，例如以下例子：

若為gatewayWorker，監聽部分代碼如下
```php
// WebSocket協議，這樣客戶端才能用ws://...來連。xxxx為端口不用改動
$gateway = new Gateway('websocket://0.0.0.0:xxxx');
```
若為Workerman則為
```php
// WebSocket協議，這樣客戶端才能用ws://...來連。xxxx為端口不用改動
$worker = new Worker('websocket://0.0.0.0:xxxx');
```

# 序言

**Webman，高性能PHP应用容器**

## Workerman是甚麼？
Workerman是一款純PHP開發的開源高性能的PHP應用容器。

Workerman並非重複造輪子，它並非一個MVC框架，而是一個更底層更通用的服務框架，你可以用它開發tcp代理、梯子代理、做遊戲伺服器、郵件伺服器、ftp伺服器、甚至開發一個php版本的redis、php版本的資料庫、php版本的nginx、php版本的php-fpm等等。Workerman可以說是PHP領域的一次創新，讓開發者徹底擺脫了PHP只能做WEB的束縛。

實際上Workerman類似一個PHP版本的nginx，核心也是多進程+Epoll+非阻塞IO。Workerman每個進程能維持上萬並發連接。由於本身常駐內存，不依賴Apache、nginx、php-fpm這些容器，擁有超高的性能。同時支持TCP、UDP、UNIXSOCKET，支持長連接，支持Websocket、HTTP、WSS、HTTPS等通訊協議以及各種自定義協議。擁有定時器、異步socket客戶端、異步Redis、異步Http、異步消息隊列等眾多高性能組件。

## Workerman的一些應用方向
Workerman不同於傳統MVC框架，Workerman不僅可以用於Web開發，同時還有更廣闊的應用領域，例如即時通訊類、物聯網、遊戲、服務治理、其他伺服器或者中介軟體，這無疑大大提高了PHP開發者的視野。目前這些領域的PHP開發者奇缺，如果想在PHP領域有自己的技術優勢，不滿足於每天的增刪改查工作，或者想向架構師方向或者技術大牛的方向發展，Workerman都是非常值得學習的框架。建議開發者不僅會用，而且能基於Workerman開發出屬於自己的開源項目，提升技能增加自己的影響力，比如[Beanbun多進程網路爬蟲框架](https://github.com/kiddyuchina/Beanbun)就是一個很好的例子，剛剛上線不久就獲得眾多好評。

Workerman的一些應用方向如下：

1、即時通訊類
例如網頁即時聊天、即時消息推送、微信小程序、手機app消息推送、PC軟體消息推送等等
[[示例 workerman-chat聊天室](https://www.workerman.net/workerman-chat) 、 [web消息推送](https://www.workerman.net/web-sender) 、 [小蝌蚪聊天室](https://www.workerman.net/workerman-todpole)]

2、物聯網類
例如Workerman與印表機通訊、與單片機通訊、智能手環、智能家居、共享單車等等。
[客戶案例如 易聯雲、易泊時代等]

3、遊戲伺服器類
例如棋牌遊戲、MMORPG遊戲等等。[[示例 browserquest-php](https://www.workerman.net/browserquest)]

4、HTTP服務
例如 写高性能HTTP接口、高性能網站。如果想要做HTTP相關的服務或者站點強烈推薦 [webman](https://github.com/walkor/webman)

5、SOA服務化
利用Workerman將現有業務不同功能單元封裝起來，以服務的形式對外提供統一的接口，達到系統鬆耦合、易維護、高可用、易伸縮。[[示例 workerman-json-rpc](https://github.com/walkor/workerman-jsonrpc)、 [workerman-thrift](https://github.com/walkor/workerman-thrift)]

6、其他伺服器軟體
例如 [GatewayWorker](https://www.workerman.net/doc/gateway-worker)，[PHPSocket.IO](https://www.workerman.net/phpsocket_io)，[http代理](https://github.com/walkor/php-http-proxy)，[sock5代理](https://github.com/walkor/php-socks5)，[分佈式通訊組件](https://github.com/walkor/Channel)，[分佈式變量共享組件](https://github.com/walkor/GlobalData)，[消息隊列](https://github.com/walkor/workerman-queue)、DNS伺服器、WebServer、CDN伺服器、FTP伺服器等等

7、組件
例如[異步redis](components/workerman-redis.md)，[異步http客戶端](components/workerman-http-client.md)，[物聯網mqtt客戶端](components/workerman-mqtt.md)，消息隊列 [workerman/redis-queue](components/workerman-redis-queue.md) 、 [workerman/stomp](components/workerman-stomp.md)、[workerman/rabbitmq](components/workerman-rabbitmq.md)  ，[文件監控組件](components/file-monitor.md)，還有很多第三方開發的組件框架等等

顯然傳統的mvc框架很難實現以上的功能，所以也就是workerman誕生的原因。

## Workerman理念
極簡、穩定、高性能、分佈式。

### **極簡**
小即是美，Workerman核心極簡，僅有幾個php文件並且只暴露幾個接口，學習起來非常簡單。所有其它功能通過組件的方式擴展。

Workerman擁有完善的文檔+權威的主頁+活躍的社區+數個千人QQ群+眾多的高性能組件+N多的例子，這一切都讓開發者使用起來更得心應手。

### **穩定**
Workerman已經開源數年，被很多上市公司大規模使用，超級穩定。有些服務2年多沒重啟過仍然在飛速運行。沒有coredump、沒有內存洩漏、沒有bug。

### **高性能**
Workerman因為常駐內存，本身不依賴apache/nginx/php-fpm，沒有容器到PHP的通訊開銷，沒有每個請求初始化一切又銷毀一切的開銷，具有超高的性能，比起傳統的MVC框架，性能要高數十倍，PHP7下通過ab壓力測試QPS甚至高於單獨的nginx。

### **分佈式**
現在早已經不是單槍匹馬的時代了，單台伺服器性能再強悍也有極限，分佈式多伺服器部署才是王道。Workerman直接提供了一套長連接分佈式通訊方案[GatewayWorker框架](https://doc2.workerman.net)，加伺服器只需要簡單配置下然後啟動即可，業務代碼零更改，系統承載能力成倍增加。如果你是開發TCP長連接應用，建議直接用[GatewayWorker](https://doc2.workerman.net)，它是對Workerman的一個包裝，針對長連接應用提供了更豐富的接口以及強悍的分佈式處理能力。

## 本手冊作用範圍
Workerman 3.x - 4.x版本

## Windows用戶（必讀）
workerman同時支持linux系統及windows系統。windows版本workerman本身**不依賴任何擴展**，只需要配置好PHP環境變數即可，**windows版本workerman安裝及注意事項參見[windows用戶必看](https://www.workerman.net/windows)。**

## 客戶端
WorkerMan的通訊協議是開放的，又是可定制的，因此，理論上WorkerMan可以與使用任意協議的任意平臺的客戶端進行通訊。當用戶開發客戶端時，可以根據相應的通訊協議完成與服務端的通訊。

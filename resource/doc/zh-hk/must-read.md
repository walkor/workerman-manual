# webman 開發者必須知道的幾個問題
**1、Windows 環境限制**

在 Windows 系統下，workerman 單個進程僅支持 200+ 連接。在 Windows 系統下無法使用 count 參數設定多進程。在 Windows 系統下無法使用 status、stop、reload、restart 等命令。在 Windows 系統下無法守護進程，cmd 窗口關掉後服務即停止。在 Windows 系統下無法在一個文件中初始化多個監聽。Linux 系統無上面的限制，建議正式環境用 Linux 系統，開發環境可以選擇使用 Windows 系統。

**2、workerman 不依賴 Apache 或者 Nginx**

workerman 本身已經是一個類似 Apache/Nginx 的容器，只要 [PHP 環境 OK](315116) workerman 就可以運行。

**3、workerman 是命令行啟動的**

啟動方式類似 Apache 使用命令啟動(一般網頁空間無法使用 workerman)。啟動界面類似下面
![](image/screenshot_1495622774534.png)

**4、長連接必須加心跳**

長連接必須加心跳，長連接必須加心跳，長連接必須加心跳，重要的話說三遍。 
長連接長時間不通訊會被路由節點清理導致連接關閉。
[workerman心跳说明](faq/heartbeat.md)、 [gatewayWorker心跳说明](https://www.workerman.net/doc/gateway-worker/heartbeat.html)

**5、客戶端和服務端協議一定要對應才能通訊**

這個是開發者非常常見的問題。例如客戶端是用 websocket 協議，服務端必須也是 websocket 協議(服務端`new Worker('websocket://0.0.0.0...')`)才能連得上，才能通訊。 
不要嘗試在瀏覽器地址欄訪問 websocket 協議端口，不要嘗試用 webscoket 協議訪問裸 tcp 協議端口，協議一定要對應。

這裡的原理類似如果你要和英國人交流，那麼要使用英語。如果要和日本人交流，那麼要使用日語。這裡的語言就類似與通訊協議，雙方(客戶端和服務端)必須使用相同的語言才能交流，否則無法通訊。 

**6、連接失敗可能的原因**

剛開始使用 workerman 時很常見的一個問題是客戶端連接服務端失敗。 原因一般如下： 
1、伺服器防火牆(包括雲伺服器安全組)阻止了連接 （50%幾率是這個）
2、客戶端和服務端使用的協議不一致 （30%幾率）
3、IP 或者端口寫錯了 (15%的幾率)
4、服務端沒啟動

**7、不要使用 exit die sleep 語句**

業務執行 exit die 語句會導致進程退出，並顯示 WORKER EXIT UNEXPECTED 錯誤。當然，進程退出了會立刻重啟一個新的進程繼續服務。如果需要返回，可以調用 return。sleep 語句會讓進程睡眠，睡眠過程中不會執行任何業務，框架也會停止運行，會導致該進程的所有客戶端請求都無法處理。

**8、不要使用 pcntl_fork 函數**

`pcntl_fork` 用來動態創建新的進程，如果在業務代碼中使用 `pcntl_fork`，它可能會產生無法回收孤兒進程，導致業務出現異常。業務中 `pcntl_fork` 還會影響連接、消息、連接關閉、定時器等事件的處理，導致不可預知的異常。

**9、業務代碼裡不要有死循環**

業務代碼裡不要有死循環，否則會導致控制權無法交還給 workerman 框架，導致無法接收處理其他客戶端消息。

**10、改代碼要重啟**

workerman 是常駐內存的框架，改代碼要重啟 workerman 才能看到新代碼的效果。

**11、長連接應用建議用 GatewayWorker 框架**

很多開發者使用 workerman 是要開發**長連接**應用，例如即時通訊、物聯網等，**長連接**應用建議直接使用 GatewayWorker 框架，它專門在 workerman 的基礎上再次封裝，做起長連接應用後臺更簡單、更易用。

**12、支持更高並發**

如果業務並發連接數超過 1000 同時線上，請務必[優化 Linux 內核](appendices/kernel-optimization.md)，並[安裝 event 擴展](appendices/install-extension.md)。

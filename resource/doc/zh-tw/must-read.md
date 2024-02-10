# workerman開發人員必須知道的幾個問題
**1、Windows環境限制**

在Windows系統下，workerman單個進程僅支持200+個連接。
在Windows系統下無法使用count參數設置多進程。
在Windows系統下無法使用status、stop、reload、restart等命令。
在Windows系統下無法守護進程，cmd視窗關掉後服務即停止。
在Windows系統下無法在一個文件中初始化多個監聽。
Linux系統無上面的限制，建議正式環境用Linux系統，開發環境可以選擇用Windows系統。

**2、workerman不依賴Apache或者Nginx**

workerman本身已經是一個類似Apache/Nginx的容器，只要[PHP環境OK](315116) workerman就可以運行。

**3、workerman是命令行啟動的**

啟動方式類似Apache使用命令啟動(一般網頁空間無法使用workerman)。啟動界面類似下面
![](image/screenshot_1495622774534.png)

**4、長連接必須加心跳**

長連接必須加心跳，長連接必須加心跳，長連接必須加心跳，重要的話說三遍。 
長連接長時間不通訊會被路由節點清理導致連接關閉。
[workerman心跳說明](faq/heartbeat.md)、 [gatewayWorker心跳說明](https://www.workerman.net/doc/gateway-worker/heartbeat.html)

**5、客戶端和服務端協議一定要對應才能通訊**

這個是開發者非常常見的問題。例如客戶端是用websocket協議，服務端必須也是websocket協議(服務端```new Worker('websocket://0.0.0.0...')```)才能連得上，才能通訊。 
不要嘗試在瀏覽器地址欄訪問websocket協議端口，不要嘗試用webscoket協議訪問裸tcp協議端口，協議一定要對應。

這裡的原理類似如果你要和英國人交流，那麼要使用英語。如果要和日本人交流，那麼要使用日語。這裡的語言就類似與通訊協議，雙方(客戶端和服務端)必須使用相同的語言才能交流，否則無法通訊。 

**6、連接失敗可能的原因**

剛開始使用workerman時很常見的一個問題是客戶端連接服務端失敗。 原因一般如下： 
1、伺服器防火牆(包括雲伺服器安全組)阻止了連接 （50%機率是這個）
2、客戶端和服務端使用的協議不一致 （30%機率）
3、IP或者端口寫錯了 (15%的機率)
4、服務端沒啟動 

**7、不要使用exit die sleep語句**

業務執行exit die語句會導致進程退出，並顯示WORKER EXIT UNEXPECTED錯誤。當然，進程退出了會立刻重啟一個新的進程繼續服務。如果需要返回，可以調用return。sleep語句會讓進程睡眠，睡眠過程中不會執行任何業務，框架也會停止運行，會導致該進程的所有客戶端請求都無法處理。

**8、不要使用pcntl_fork函數**

`pcntl_fork`用來動態創建新的進程，如果在業務代碼中使用`pcntl_fork`，它可能會產生無法回收孤兒進程，導致業務出現異常。業務中`pcntl_fork`還會影響連接、消息、連接關閉、定時器等事件的處理，導致不可預知的異常。

**9、業務代碼裡不要有死循環**

業務代碼裡不要有死循環，否則會導致控制權無法交還給workerman框架，導致無法接收處理其他客戶端消息。

**10、改代碼要重啟**

workerman是常駐內存的框架，改代碼要重啟workerman才能看到新代碼的效果。

**11、長連接應用建議用GatewayWorker框架**

很多開發者使用workerman是要開發**長連接**應用，例如即時通訊、物聯網等，**長連接**應用建議直接使用GatewayWorker框架，它專門在workerman的基礎上再次封裝，做起長連接應用後臺更簡單、更易用。

**12、支援更高並發**
如果業務並發連接數超過1000同時線上，請務必[優化Linux內核](appendices/kernel-optimization.md)，並[安裝event擴展](appendices/install-extension.md)。

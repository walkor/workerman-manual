# Worker類
在 Workerman 中有兩個重要的類別，分別是 Worker 與 Connection。

Worker 類別用於實現端口的監聽，並且可以設置客戶端連接事件、連接上消息事件、連接斷開事件的回調函數，從而實現業務處理。

可以設置 Worker 實例的進程數（count 屬性），Worker 主進程會 fork 出 count 個子進程同時監聽相同的端口，並行地接收客戶端連接，處理連接上的事件。

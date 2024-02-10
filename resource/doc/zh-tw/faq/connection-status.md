# 檢視目前客戶端連線數
執行 ```php start.php status``` 可以查看目前伺服器的 WorkerMan 運行狀態，```connections``` 欄位標註了每個進程目前的 TCP 連線數。需要注意的是，這個欄位不僅包括客戶端的 TCP 連線數，也包括 WorkerMan 內部通訊的 TCP 連線數。例如，在 WorkerMan 的 Gateway/Worker 模型中，每個 Gateway 進程目前的客戶端連線數為```connections``` 欄位的數值減去 Worker 進程數。

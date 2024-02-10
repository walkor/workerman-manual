# 查看當前客戶端連接數
運行 ```php start.php status``` 能夠查看當前伺服器的WorkerMan運行狀態，```connections``` 欄位標記了每個進程當前的TCP連接數。需要注意的是這個欄位不僅包括客戶端的TCP連接數，也包括WorkerMan內部通訊的TCP連接數。例如在WorkerMan中的Gateway/Worker模型中，每個Gateway進程當前的客戶端連接數即為```connections``` 欄位值減去Worker進程數。

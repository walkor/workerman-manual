# 終端關閉導致服務關閉
**問：**
為什麼我關閉了終端，Workerman就自己關閉了？

**答：**
Workerman有兩種啟動模式，debug調試模式和daemon守護進程模式。

運行 ```php xxx.php start``` 是進入debug調試模式，用於開發調試問題，當終端關閉後Workerman會隨之關閉。

運行 ```php xxx.php start -d```進入的是daemon守護進程模式，終端關閉不會影響Workerman。

如果想讓Workerman不受終端影響，可以使用daemon模式啟動。

# 終端關閉導致服務關閉
**問：**
為什麼我關閉了終端，Workerman 就自己關閉了？

**答：**
Workerman 有兩種啟動模式，分別是調試模式（debug）和守護進程模式（daemon）。

運行 ```php xxx.php start``` 是進入調試模式，用於開發調試問題，當終端關閉後Workerman 會隨之關閉。

運行 ```php xxx.php start -d``` 進入的是守護進程模式，終端關閉不會影響Workerman。

如果想要讓 Workerman 不受終端影響，可以使用守護進程模式啟動。

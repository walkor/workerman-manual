# 調試busy進程
有時候我們透過```php start.php status``` 命令能看到有```busy```狀態的進程，說明對應進程正在處理業務，正常情況下業務處理完畢對應進程會恢復為```idle```狀態，這一般情況下不會有什麼問題。但是如果一直是busy狀態沒有恢復過```idle```狀態，則說明進程內的業務有阻塞或者無限循環，可以通過以下方法定位。

## 利用strace+lsof命令定位

**1、status裡找到busy進程的pid**
運行```php start.php status``` 後顯示如下
![](../images/d1903ed65ef2f3b0850e84ccbedc52aa.png)
圖中```busy```的進程的```pid```為```11725```和```11748```

**2、strace 跟踪進程**
挑選一個進程```pid```(這裡選擇```11725```)，運行```strace -ttp 11725``` 顯示如下
![](../images/7ce9f36da926f670949609dcdc593ab4.png)
可以看到進程在不斷的循環```poll([{fd=16, events=....```的系統調用，這是在等待fd為16的描述符可讀事件，也就是在等這個描述符返回資料。

如果沒有顯示任何系統調用，保留當前終端，重新再打開一個終端，運行```kill -SIGALRM 11725```(給進程發送一個鬧鐘信號)，然後看strace的終端是否有響應，是否阻塞在某個系統調用上。如果仍然沒有顯示任何系統調用說明程序很可能處於業務死循環中，參考頁面下部引起進程長時間busy的其它原因 第2項 解決。

如果系統阻塞在epoll_wait或者select系統調用是正常情況，這說明進程已經處於idle狀態。

**3、lsof 查看進程描述符**
運行```lsof -nPp 11725``` 顯示如下
![](../images/27bd629c3a1ac93f9f4b535d01df2ac1.png)
述符16對應的是16u的記錄(最後一行)，能看```fd=16```的描述符是一個tcp連接，遠程地址是```101.37.136.135:80```，說明進程應該是在訪問一個http資源，循環```poll([{fd=16, events=....```是一直在等待http服務端返回數據，這解釋了為什麼進程處於```busy```狀態

**解決：**
知道了進程阻塞在哪裡，接下來就容易解決了，例如上面經過定位應該是業務在調用curl，而對應的url長時間沒有返回數據，導致進程一直等待。這時候可以找url提供者定位url返回慢的原因，同時應該在curl調用的時候加上超時參數，比如2秒沒返回就超時，避免長時間阻塞卡死(這樣進程可能會出現2秒左右的busy狀態)。

## 引起進程長時間 busy 的其它原因
除了進程阻塞或導致進程```busy```，還有以下原因會引起進程處於```busy```狀態。

**1、業務有致命錯誤導致進程不斷退出**
**現象：** 這種情況下能看到系統負載比較高，```status```中的```load average```為1或者更高。能看到進程的```exit_count```數字很高，並且不斷增長
**解決：** debug方式運行(```php start.php start```不加```-d```)workerman看下業務報錯，把報錯解決即可

**2、代碼裡無限死循環**
**現象：** top中能看到busy進程佔用cpu很高，```strace -ttp pid```命令沒有打印出任何系統調用信息
**解決：** 參考鳥哥的文章通過[gdb和php源碼定位](https://www.laruence.com/2011/12/06/2381.html)，步驟總結大概如下：
1、```php -v```查看版本
2、[下載對應php版本的源碼](https://www.php.net/releases/)
3、```gdb --pid=busy進程的pid``` 
4、```source php源碼路徑/.gdbinit```，
5、```zbacktrace``` 打印調用棧
最後一步可以看到php代碼當前執行的調用棧，也就是php代碼死循環的位置。
注意：如果```zbacktrace``` 沒有打出調用棧，可能你的php編譯時沒有加入```-g```參數，需要重新編譯php，然後重啟workerman定位。

**3、無限添加定時器**
業務代碼不停的添加定時器又不刪除，導致進程內定時器越來越多，最終造成進程無限運行定時器。例如以下代碼：
```php
$worker = new Worker;
$worker->onConnect = function($con){
    Timer::add(10, function(){});
};
Worker::runAll();
````
以上代碼在有客戶端連接上來後會增加一個定時器，但是整個業務代碼裡沒有刪除定時器的邏輯，這樣隨著時間推移，進程內會不斷增加定時器，最終導致進程無限運行定時器導致busy。
正確的代碼：
```php
$worker = new Worker;
$worker->onConnect = function($con){
    $con->timer_id = Timer::add(10, function(){});
};
$worker->onClose = function($con){
    Timer::del($con->timer_id);
};
Worker::runAll();
````

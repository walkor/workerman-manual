# 檔案監控元件

**背景：**

Workerman是一個常駐記憶體運行的服務器框架，常駐記憶體的特性可避免重複讀取磁碟、重複解釋編譯PHP，從而實現最高性能。因此，在更改業務代碼後，需要手動重新加載或重新啟動才能生效。

同時，Workerman提供了一個監控文件更新的服務，該服務會自動運行重新加載，從而重新載入PHP文件。開發者可將其放入項目中，在項目啟動時即可使用。

**檔案監控服務下載地址：**

1、無依賴版本：https://github.com/walkor/workerman-filemonitor

2、依賴inotify版本：https://github.com/walkor/workerman-filemonitor-inotify (需要安裝[inotify擴展](https://php.net/manual/zh/book.inotify.php))

**兩個版本區別：**

地址1版本使用每秒輪詢文件更新時間的方法來判斷文件是否更新，

地址2利用Linux內核[inotify](https://baike.baidu.com/view/2645027.htm)機制，當文件更新時系統會主動通知workerman。

一般使用第一個無依賴版本即可。

**使用方法**

將Applications/FileMonitor目錄複製到你項目的Applications目錄下即可。

如果你的項目沒有Applications目錄，可以將Applications/FileMonitor/start.php文件複製到你的項目任意位置，在啟動腳本中require到啟動腳本中即可。

監控元件默認監控的是Applications目錄，如果需要更改，可以修改Applications/FileMonitor/start.php中的```$monitor_dir```變數，```$monitor_dir```的值建議是絕對路徑。

**注意：**

* Windows系統不支持reload，無法使用此監控服務。
* 只有在debug模式下才生效，daemon下不會執行文件監控（為何不支持daemon模式見下面說明）。
* 只有在Worker::runAll運行後加載的文件才能熱更新，或者說只有在onXXX回調中加載的文件才能熱更新。

**為何不支持daemon模式?**

daemon模式一般為線上正式環境運行的模式。正式環境發布版本時，一般一次發布多個文件，文件之間也可能有依賴。由於多個文件同步到磁碟需要一定時間，會存在某個時刻磁碟上的文件不全的情況，如果這時候監控到了文件更新並執行了reload，則會有找不到文件導致致命錯誤的風險。

另外正式環境中有時候會線上定位bug，如果直接編輯代碼保存，就會立刻生效，有可能出現語法錯誤導致線上服務不可用。正確的方法應該是保存代碼後，通過```php -l yourfile.php```檢查下是否有語法錯誤，然後再reload熱更新代碼。

如果開發者確實需要daemon模式開啟文件監控及自動更新，可以自行更改Applications/FileMonitor/start.php代碼，將Worker::$daemonize部分的判斷去掉即可。

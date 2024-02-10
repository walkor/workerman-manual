# 檔案監控組件

**背景：**

Workerman 是一個常駐內存運行的服務器，它能夠避免重複讀取磁盤和重複解釋編譯 PHP，從而實現最高性能。因此，當更改業務代碼後，需要手動重新加載或重啟才能生效。同時，Workerman 提供了一個監控文件更新的服務，當檔案更新時會自動執行重新加載，重新載入 PHP 檔案。開發者只需將其放入項目中，隨著項目一起啟動即可。

**檔案監控服務下載地址：**

1. 無依賴版本：https://github.com/walkor/workerman-filemonitor
2. 依賴 inotify 版本：https://github.com/walkor/workerman-filemonitor-inotify（需要安裝 [inotify 擴展](https://php.net/manual/zh/book.inotify.php)）

**兩個版本的區別：**

地址1版本使用每秒輪詢文件更新時間的方法來判斷文件是否更新。
地址2利用 Linux 內核 [inotify](https://baike.baidu.com/view/2645027.htm) 機制，當檔案更新時系統會主動通知 Workerman。一般使用第一個無依賴版本即可。

**使用方法：**

將 Applications/FileMonitor 目錄複製到你項目的 Applications 目錄下即可。如果你的項目沒有 Applications 目錄，可以將 Applications/FileMonitor/start.php 檔案複製到你的項目任意位置，在啟動腳本中 require 進啟動腳本中即可。監控組件默認監控的是 Applications 目錄，如果需要更改，可以修改 Applications/FileMonitor/start.php 中的 `$monitor_dir` 變數，`$monitor_dir` 的值建議是絕對路徑。

**注意：**
- Windows 系統不支持重新加載，無法使用此監控服務。
- 只有在 debug 模式下才生效，daemon 模式下不會執行檔案監控（為何不支持 daemon 模式見下面說明）。
- 只有在 Worker::runAll 運行後加載的檔案才能熱更新，或者說只有在 onXXX 回調中加載的檔案才能熱更新。

**為何不支持 daemon 模式？**

daemon 模式一般為線上正式環境運行的模式。正式環境發布版本時，一般一次發布多個檔案，檔案之間也可能有依賴。由於多個檔案同步到磁盤需要一定時間，會存在某個時刻磁盤上的檔案不全的情況，如果這時候監控到了檔案更新並執行了重新加載，則會有找不到檔案導致致命錯誤的風險。

另外，在正式環境中有時候會線上定位 bug，如果直接編輯代碼保存，則會立刻生效，有可能出現語法錯誤導致線上服務不可用。正確的方法應該是保存代碼後，通過 `php -l yourfile.php` 檢查下是否有語法錯誤，然後再重新加載熱更新代碼。

如果開發者確實需要在 daemon 模式下開啟檔案監控及自動更新，可以自行更改 Applications/FileMonitor/start.php 代碼，將 Worker::$daemonize 部分的判斷去掉即可。

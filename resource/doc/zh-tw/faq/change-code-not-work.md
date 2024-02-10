# 更改程式碼後不生效

**原因：**

Workerman 是在內存中常駐運行的，這能避免重複讀取磁碟、重複解釋編譯 PHP，以達到最佳效能。因此在更改業務程式碼後需要手動 reload 或 restart 才能生效。

同時，Workerman 提供一個檔案監控更新的服務，該服務會自動運行 reload，重新載入 PHP 檔案一旦檔案更新被偵測到。開發者將其放入到專案中並隨專案啟動即可。

請注意：Windows 系統不支持 reload，無法使用監控服務。

**檔案監控服務下載地址：**

1. 無依賴版本：https://github.com/walkor/workerman-filemonitor
2. 依賴 inotify 版本：https://github.com/walkor/workerman-filemonitor-inotify

**兩個版本的區別：**

無依賴版本使用每秒輪詢檔案更新時間的方法來判斷檔案是否更新，
而依賴 inotify 版本利用 Linux 內核的 inotify 機制，當檔案更新時系統會主動通知 Workerman。

一般使用無依賴版本即可。

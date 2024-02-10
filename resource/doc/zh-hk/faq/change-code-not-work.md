# 更改程式碼後不生效

**原因：**

Workerman是常駐內存運行的，常駐內存可以避免重複讀取磁碟、重複解釋編譯PHP，以便達到最高性能。所以更改業務程式碼後需要手動reload或者restart才能生效。

同時workerman提供一個監控檔案更新的服務，該服務檢測到有檔案更新後會自動運行reload，重新載入PHP檔案。開發者將其放入到項目中隨著項目啟能。 

注意：windows系統不支持reload，無法使用監控服務。

**檔案監控服務下載地址：**

1、無依賴版本：https://github.com/walkor/workerman-filemonitor

2、依賴inotify版本:https://github.com/walkor/workerman-filemonitor-inotify

**兩個版本區別：**

地址1版本使用的是每秒輪詢檔案更新時間的方法判斷檔案是否更新，

地址2利用Linux內核[inotify](https://baike.baidu.com/view/2645027.htm)機制，檔案更新時系統會主動通知workerman。

一般使用地址1無依賴版本即可。

# Channel分佈式通訊元件
**```（要求Workerman版本>=3.3.0）```**

源碼地址：https://github.com/walkor/Channel

Channel是一個分佈式通訊元件，用於完成進程間通訊或者伺服器間通訊。

## 特點
1、基於訂閱發布模型
2、非阻塞式IO

## 原理
Channel包含Channel/Server服務端和Channel/Client客戶端
Channel/Client通過connect接口連接Channel/Server並保持長連接
Channel/Client通過調用on接口告訴Channel/Server自己關注哪些事件，並註冊事件回調函數（回調發生在Channel/Client所在進程中）
Channel/Client通過publish接口向Channel/Server發布某個事件及事件相關的數據
Channel/Server接收事件及數據後會分發給關注這個事件的Channel/Client
Channel/Client收到事件及數據後觸發on接口設定的回調
Channel/Client只會收到自己關注事件並觸發回調

## 安裝
`composer require workerman/channel`

## 注意
Channel只能用在workerman環境，php-fpm環境下無法使用。

# 原理

### Worker說明
Worker是Workerman中最基本的容器，Worker可以開啟多個進程監聽端口並使用特定協議進行通訊，類似於nginx監聽某個端口。每個Worker進程獨立運作，採用Epoll（需要裝event擴展）+非阻塞IO，每個Worker進程都能處理上萬個客戶端連接，並處理這些連接上發來的數據。主進程為了保持穩定性，只負責監控子進程，不負責接收數據，也不作任何業務邏輯。

### 客戶端與worker進程的關係
![workerman master woker模型](images/Worker.png)


### 主進程與worker子進程關係
![workerman master woker模型](images/Worker2.png)

**特點：**

從圖上我們可以看出每個Worker維持著各自的客戶端連接，能夠方便地實現客戶端與服務端的實時通訊，基於這種模型我們可以方便實現一些基本的開發需求，例如HTTP伺服器、Rpc伺服器、一些智能硬體實時上報數據、服務端推送數據、遊戲伺服器、微信小程式後台等等。

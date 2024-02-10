# 在其他專案中運用 WorkerMan 給客戶端推送資料

**問：**

我有一個普通的 web 專案，想在這個專案中調用 WorkerMan 的接口，給客戶端推送資料。

**答：**

**基於 Workerman 可以參考以下鏈接**

- [Channel 組件推送例子](../components/channel-examples.md)（支持多進程/伺服器集群，需要下載 Channel 組件）

- [基於 Worker 推送](https://www.workerman.net/q/508)(單進程，最簡單)

**基於 webman 參考以下鏈接**
- [webman 推送插件](https://www.workerman.net/plugin/2)

**基於 GatewayWorker 參考以下鏈接**

- [在其他專案中透過 GatewayWorker 推送](https://www.workerman.net/doc/gateway-worker/push-in-other-project.html)(支持多進程/伺服器集群，支持分組、組播、單個發送)

**基於 PHPSocket.IO 參考以下鏈接**

- [web 消息推送](https://www.workerman.net/web-sender)(默認單進程，基於 socket.io，瀏覽器兼容性最好)

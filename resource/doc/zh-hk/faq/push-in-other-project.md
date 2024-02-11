# 在其他項目中利用Workerman給客戶端推送數據

**問：**

我有一個普通的web項目，想在這個項目中調用Workerman的接口，給客戶端推送數據。

**答：**

**基於Workerman可以參考以下連結**

- [Channel組件推送例子](../components/channel-examples.md)（支持多進程/伺服器集群，需要下載Channel組件）

- [基於Worker推送](https://www.workerman.net/q/508) (單進程，最簡單)

**基於webman參考下面連結**
- [webman push插件](https://www.workerman.net/plugin/2)

**基於GatewayWorker參考下面連結**

- [在其他項目中通過GatewayWorker推送](https://www.workerman.net/doc/gateway-worker/push-in-other-project.html) (支持多進程/伺服器集群，支持分組、組播、單個發送)

**基於PHPSocket.IO參考下面連結**

- [web消息推送](https://www.workerman.net/web-sender) (默認單進程，基於socket.io，瀏覽器兼容性最好)

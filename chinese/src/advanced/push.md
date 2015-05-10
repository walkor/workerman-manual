# 在非WorkerMan项目中推送消息

## 需求
有时候需要在非WorkerMan环境中向客户端推送数据。例如在一个普通的Web项目中通过WorkerMan推送数据（前提是已经部署了WorkerMan，客户端已经连接WorkerMan）。目前有三种比较方便方法推送数据。（本节内容主要是针对Gateway/Worker模型的推送方法的讲解）

## 方法一、使用GatewayClient客户端推送
**客户端地址：**

https://github.com/walkor/GatewayClient

**使用方法：**

*注意:如果项目与WorkerMan不在同一台服务器，需要安装redis，参考8.10 Config/Store配置章节。


1、拷贝WorkerMan项目中的Applications/YourApp/Config目录到GatewayClient下。

拷贝后的GatewayClient目录结构如下
```shell
GatewayClient/
├── Config
│   └── Store.php
└── Gateway.php
```
2、引入GatewayClient/Gateway.php文件开始使用。接口使用方法与\GatewayWorker\Lib\Gateway接口相同，接口说明参见8.5章节。

 **客户端使用示例**
 ```php
require_once '/your/path/GatewayClient/Gateway.php';

Gateway::sendToAll('{"type":"broadcast","content":"hello all"}');

Gateway::sendToClient($client_id,'{"type":"say","content":"hello"}');

Gateway::isOnline($client_id);

...
 ```

 ## 方法二、用一个特殊的账号当做管理客户端，通过这个账号推送数据

 此方法通俗易懂，可以通过现有客户端直接操作，具体代码根据自己的业务实现


 ## 方法三、开启一个内部Gateway端口，用于推送数据
 方法二虽然简单，但是局限于只能通过客户端界面操作，定时推送等需求不好直接操作客户端，而通过PHP模拟客户端可能会受到复杂协议的限制不好操作，这时我们可以开启一个内部文本协议的Gateway端口，通过PHP代码使用文本协议连接WorkerMan作为客户端向其它客户端推送数据。

 ** 示例（workerman-chat为例） **

**服务端：**Applications/Chat/start.php中新增一个文本协议Gateway端口

 ```php
// gateway
$gateway = new Gateway("Websocket://0.0.0.0:7272");
$gateway->name = 'ChatGateway';
$gateway->count = 4;
$gateway->lanIp = '127.0.0.1';
$gateway->startPort = 2500;
$gateway->pingInterval = 10;
$gateway->pingData = '{"type":"ping"}';

// #### 内部推送端口(假设内网ip为192.168.100.100) ####
$internal_gateway = new Gateway("Text://192.168.100.100:7273");
$internal_gateway->name='internalGateway';
$internal_gateway->startPort = 2800;
// #### 内部推送端口设置完毕 ####

// bussinessWorker
$worker = new BusinessWorker();
$worker->name = 'ChatBusinessWorker';
$worker->count = 4;

// WebServer
$web = new WebServer("http://0.0.0.0:55151");
$web->count = 2;
$web->addRoot('www.workerman.net', __DIR__.'/Web');

 ```

 **客户端：**在其它项目中就可以直接用PHP socket 使用文本协议调用，代码类似如下:
 ```php
// 建立连接，@see http://php.net/manual/zh/function.stream-socket-client.php
$client = stream_socket_client('tcp://192.168.100.100:7273');
if(!$client)exit("can not connect");
// 模拟超级用户，以文本协议发送数据，注意文本协议末尾有换行符（发送的数据中最好有能识别超级用户的字段），这样在Event.php中的onMessage方法中便能收到这个数据，然后做相应的处理
fwrite($client, '{"type":"send","content":"hello all", "user":"admin", "pass":"******"}'."\n");
 ```




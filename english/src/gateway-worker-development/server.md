# 超全局数组$_SERVER
### ```$_SERVER```是什么
WorkerMan中的超全局数组```$_SERVER```包含了5个元素，分别是：

  * REMOTE_ADDR // 客户端ip（如果客户端处于局域网，则是客户端所在局域网的出口ip）
  * REMOTE_PORT // 客户端端口（如果客户端处于局域网，则是客户端所在局域网的出口端口）
  * GATEWAY_ADDR // gateway所在服务器的ip
  * GATEWAY_PORT // geteway所在服务器的端口
  * GATEWAY_CLIENT_ID // 全局唯一的客户端id


###```$_SERVER```使用场景

当需要在Event.php中获得客户端的ip及端口信息时，可以使用```$_SERVER['REMOTE_ADDR']```和```$_SERVER['REMOTE_PORT']```获得。当想在某个函数逻辑处理时获得当前客户端的client_id时，可以使用```$_SERVER['GATEWAY_CLIENT_ID']```方便的获得

### ```$_SERVER```使用注意事项
* ```$_SERVER['GATEWAY_ADDR'] ``` 和 ```$_SERVER['GATEWAY_PORT']```开发者一般用不到，可以忽略。

## ```$_SERVER```原理

在WorkerMan的Gateway/BusinessWorker模型中，每个客户端都会连接在gateway进程上，当gateway进程收到客户端的数据时，会将客户端的ip端口及client_id连通消息传递给worker进程，worker进程初始化```$_SERVER```数组便可以使用了。


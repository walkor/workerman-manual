# 超全局数组```$_SESSION```
### ```$_SESSION```是什么
WorkerMan中的超全局数组```$_SESSION```和PHP自身的```$_SESSION```功能基本相同。每个client_id对应一个```$_SESSION```数组，```$_SESSION```数组中可以保存对应客户端的会话数据，对应的client_id的后续请求可以直接使用这个数组中的数据，而不用去反复读取存储。

### ```$_SESSION```使用场景
**(```WorkerMan>=2.1.2，Gateway/Worker模型```)**

例如客户端链接WorkerMan后，需要发送验证数据让服务端验证是否合法，一般要传递一次用户名和密码数据，然后在```Gateway::onMessage($client_id, $message)```中通过查询数据库验证```$message```中的用户名密码是否正确，如果正确就可以将用户的uid写入到```$_SESSION```中如```$_SESSION['uid']=$uid;```，那么当这个client_id再次发来数据时，要判断这个客户端是否是被验证过的，就可以用```$_SESSION['uid']```是否被设置来判断。

### ```$_SESSION```使用注意事项
* 使用```$_SESSION```时无需调用session_start等函数，可直接使用
* ```$_SESSION```中无法保存资源类型的数据
* 当客户端连接断开后，对应的客户端```$_SESSION```将会清除

### ```$_SESSION```实现原理

在WorkerMan的Gateway/Worker模型中，每个客户端的```$_SESSION```数据是存储在Gateway进程内存中的，每次Gateway进程转发消息给BusibuessWorker进程时，都会顺便携带上对应客户端的```$_SESSION```数据给BusibuessWorker进程，这时BusibuessWorker进程就能使用```$_SESSION```了。而当```$_SESSION```数据有更改时，BusibuessWorker会将新的```$_SESSION```数据传递给Gateway进程进行保存。


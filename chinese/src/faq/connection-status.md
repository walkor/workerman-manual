# 查看当前客户端连接数
运行 ```php start.php status```能看到当前服务器的WorkerMan运行的状态，```connections```字段标记了每个进程当前TCP连接数。需要注意的是这个字段不仅包括客户端的TCP连接数，也包括WorkerMan内部通讯的TCP连接数。例如WorkerMan中的Gateway/Worker模型中，每个Gateway进程当前的客户端连接数为```connections```字段的值减去Worker进程数。

# 启动与停止

注意Workerman启动停止等命令都是在命令行中完成的。

要启动Workerman，首先需要有一个启动入口文件，里面定义了服务监听的端口及协议。可以参考[入门指引--简单开发实例部分](../getting-started/simple-example.md)


这里以[workerman-chat](https://www.workerman.net/workerman-chat)为例，它的启动入口为start.php。
### 启动

以debug（调试）方式启动

 ```php start.php start```

以daemon（守护进程）方式启动

 ```php start.php start -d```

### 停止
 ```php start.php stop```

### 重启
 ```php start.php restart```

### 平滑重启
 ```php start.php reload```

### 查看状态
 ```php start.php status```
 
### 查看连接状态（需要Workerman版本>=3.5.0）
```php start.php connections```



## debug和daemon方式区别

1、以debug方式启动，代码中echo、var_dump、print等打印函数会直接输出在终端。

2、以daemon方式启动，代码中echo、var_dump、print等打印会默认重定向到/dev/null文件，可以通过设置```Worker::$stdoutFile = '/your/path/file';```来设置这个文件路径。

3、以debug方式启动，终端关闭后workerman会随之关闭并退出。

4、以daemon方式启动，终端关闭后workerman继续后台正常运行。



## 什么是平滑重启？

参见 [平滑重启原理](../faq/reload-principle.md)



# 停止失败

## 现象：
运行 ```php start.php stop``` 提示 ```stop fail```

## 原因：两种可能性

**第一种可能性：**<br>
前提是以debug方式启动的workerman，开发者在终端按了```ctrl z```给workerman发送了```SIGSTOP```信号，导致workerman进入后台并挂起(暂停)，所以无法响应stop命令(```SIGINT```信号)。

**解决：**<br>
在启动workerman的终端输入```fg```(发送```SIGCONT```信号)然后回车，将workerman切回前台运行，按```ctrl c```(发送```SIGINT```信号)停止workerman。
<br>
<br>
<br>
<br>
**第二种可能性：**<br>
运行stop的用户和workerman启动用户不一致，即stop用户没有权限停止workerman。

**解决：**
切换到启动workerman的用户，或者用权限更高的用户停止workerman。



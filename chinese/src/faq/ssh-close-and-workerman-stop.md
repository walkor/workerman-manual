# 终端关闭导致服务关闭
**问：**

为什么我关闭了终端，Workerman就自己关闭了？

**答：**

Workerman有两种启动模式，debug调试模式和daemon守护进程模式。

运行 ```php xxx.php start``` 是进入debug调试模式，用于开发调试问题，当终端关闭后Workerman会随之关闭。

运行 ```php xxx.php start -d```进入的是daemon守护进程模式，终端关闭不会影响Workerman。

如果想Workerman不受终端影响，可以使用daemon模式启动。

# 不支持的函数

不支持的函数/语句  | 替代方案 | 说明
----|------|----
pcntl_fork | 提前设置好进程数|
php://input | [`$request->rawBody()`](http/request.md)| 用于HTTP协议下的应用获取POST的原始数据
exit | return | 使用exit会导致进程退出，如果要返回请直接用return语句
die | return | 使用die会导致进程退出，如果要返回请直接用return语句
header cookie session相关函数 |参考 [`$request`](http/request.md) 和 [`$response`]([http/response.md) 类 | 
set_time_limit| 无 | 只能设置为0，否则会导致workerman进程在一定时间后退出


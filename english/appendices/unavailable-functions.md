# Unsupported Functions

Unsupported Function/Statement | Alternative | Description
----|------|----
pcntl_fork | Set up the number of processes in advance | 
php://input | [`$request->rawBody()`](http/request.md)| Used for obtaining the raw POST data in HTTP applications
exit | return | Using exit will cause the process to exit, if you want to return, please use the return statement directly
die | return | Using die will cause the process to exit, if you want to return, please use the return statement directly
header cookie session-related functions | Refer to the [`$request`](http/request.md) and [`$response`](http/response.md) classes | 
set_time_limit | None | Can only be set to 0, otherwise it will cause the workerman process to exit after a certain time

# 不支持的函數

不支持的函數/語句  | 替代方案 | 說明
----|------|----
pcntl_fork | 預先設置好進程數| 
php://input | [`$request->rawBody()`](http/request.md)| 用於HTTP協議下的應用獲取POST的原始數據
exit | return | 使用exit會導致進程退出，如果要返回請直接用return語句
die | return | 使用die會導致進程退出，如果要返回請直接用return語句
header cookie session相關函數 |參考 [`$request`](http/request.md) 和 [`$response`]([http/response.md) 類 | 
set_time_limit| 無 | 僅能設置為0，否則會導致workerman進程在一定時間後退出

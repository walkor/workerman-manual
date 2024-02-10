# 不支援的函式

不支援的函式/語句 | 替代方案 | 說明
----|------|----
pcntl_fork | 預先設定好進程數| 
php://input | [`$request->rawBody()`](http/request.md)| 用於HTTP協議應用程式取得POST的原始資料
exit | return | 使用exit會導致進程退出，若要返回請直接使用return語句
die | return | 使用die會導致進程退出，若要返回請直接使用return語句
header cookie session相關函式 |參考 [`$request`](http/request.md) 和 [`$response`]([http/response.md) 類 | 
set_time_limit| 無 | 只能設定為0，否則會導致workerman進程在一定時間後退出

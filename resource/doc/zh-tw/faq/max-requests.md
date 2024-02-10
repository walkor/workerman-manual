# 如何設定workerman處理一定請求後重啟當前進程
為了讓workerman更加精簡，並沒有直接提供這個設定，不過可以通過幾行程式碼實現該功能。
```php
$worker->onMessage = function($connection, $data) {
    static $request_count;
    // 業務處理略
    if(++$request_count > 10000) {
        // 請求數達到10000後退出當前進程，主進程會自動重啟一個新的進程
        Worker::stopAll();
    }
};
```

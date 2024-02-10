# onError
## Giải thích:
```php
callback Connection::$onError
```

Chức năng tương tự như callback [Worker::$onError](../worker/on-error.md), khác biệt là chỉ áp dụng cho kết nối hiện tại, nghĩa là có thể đặt callback onError riêng lẻ cho một kết nối.

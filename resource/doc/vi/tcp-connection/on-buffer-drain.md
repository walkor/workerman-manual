# onBufferDrain
## Giải thích
```php
callback Connection::$onBufferDrain
```

Có tác dụng tương tự như callback [Worker::$onBufferDrain](../worker/on-buffer-drain.md), khác biệt là chỉ áp dụng cho kết nối hiện tại, có thể thiết lập callback onBufferDrain cho một kết nối cụ thể.

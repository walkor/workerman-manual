# onBufferFull
## Giải thích:
```php
callback Connection::$onBufferFull
```

Chức năng tương tự như callback [Worker::$onBufferFull](../worker/on-buffer-full.md), khác biệt là hoạt động chỉ đối với kết nối hiện tại, nghĩa là có thể thiết lập callback onBufferFull cho một kết nối cụ thể.

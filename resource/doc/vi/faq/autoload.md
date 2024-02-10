# Tự động tải

Nếu dự án của bạn chưa triển khai tự động tải, bạn có thể tạo nó bằng composer theo các bước sau.

**1. Thay đổi composer.json**
Thêm `"" : "./"` vào autoload.psr-4 trong composer.json, ví dụ:
```json
{
    "require": {
        "workerman/workerman" : "^v4.0.0"
    },
    "autoload": {
        "psr-4": {
            "" : "./"
        }
    }
}
```

**2. Tạo autoload.php**
Chạy `composer dumpautoload`

**3. Tải autoload.php**
Tại đầu start.php của dự án, tải `vendor/autoload.php` như sau:
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
```

**4. Khởi động lại workerman**
`php start.php restart`

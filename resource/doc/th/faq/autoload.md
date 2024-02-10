# โหลดอัตโนมัติ

หากโปรเจคของคุณยังไม่มีการโหลดอัตโนมัติ คุณสามารถใช้ Composer เพื่อสร้างไฟล์โหลดอัตโนมัติได้ ดังนี้

**1. แก้ไข composer.json**  
ในไฟล์ composer.json ให้เพิ่ม autoload.psr-4 ดังนี้ "": "./" เช่น
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

**2. สร้าง autoload.php**  
รันคำสั่ง `composer dumpautoload`

**3. โหลด autoload.php**  
ในไฟล์ start.php ของโปรเจค ให้โหลด `vendor/autoload.php` ดังนี้
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
```

**4. รีสตาร์ท workerman**  
รันคำสั่ง `php start.php restart`

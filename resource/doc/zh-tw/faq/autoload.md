# 自動加載

如果你的專案尚未實現自動加載，你可以利用 composer 來生成，步驟如下。

**1. 更改 composer.json**  
在 composer.json 中的 autoload.psr-4 添加 `"" : "./"`，例如
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

**2. 生成 autoload.php**  
執行 `composer dumpautoload`

**3. 加載 autoload.php**  
在專案的 start.php 開頭加載 `vendor/autoload.php`，例如
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
```

**4. 重啟 workerman**  
`php start.php restart`

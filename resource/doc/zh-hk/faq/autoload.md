# 自動加載

如果您的項目尚未實現自動加載，您可以使用Composer生成自動加載，步驟如下。

**1. 修改composer.json**
在composer.json中的autoload.psr-4部分加入 `"" : "./"`，例如
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

**2. 生成autoload.php**
執行 `composer dumpautoload`

**3. 載入autoload.php**
在項目的start.php頭部載入`vendor/autoload.php`，例如
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
```

**4. 重新啟動workerman**
`php start.php restart`

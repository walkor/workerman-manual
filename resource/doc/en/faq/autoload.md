# Autoload

If your project does not implement autoload, you can use composer to generate one. Follow the steps below.

**1. Modify composer.json**  
Add `"" : "./"` to autoload.psr-4 in composer.json, for example:
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

**2. Generate autoload.php**  
Run `composer dumpautoload`

**3. Load autoload.php**  
Load `vendor/autoload.php` at the beginning of your project's start.php, for example:
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
```

**4. Restart Workerman**  
`php start.php restart`

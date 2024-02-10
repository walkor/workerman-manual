# Автозагрузка

Если ваш проект не реализует автозагрузку, вы можете сгенерировать ее с помощью composer, следуя следующим шагам.

**1. Измените composer.json**
Добавьте autoload.psr-4 в composer.json `"" : "./"`, например
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

**2. Сгенерируйте autoload.php**
Выполните команду `composer dumpautoload`

**3. Загрузите autoload.php**
В файле start.php вашего проекта, загрузите `vendor/autoload.php`, например
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
```

**4. Перезапустите workerman**
`php start.php restart`

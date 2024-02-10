# Carga automática

Si tu proyecto no tiene implementada la carga automática, puedes generar una usando Composer siguiendo estos pasos.

**1. Modifica composer.json**
Agrega `"": "./"` al autoload.psr-4 en el archivo composer.json, por ejemplo:
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

**2. Genera autoload.php**
Ejecuta `composer dumpautoload`

**3. Carga autoload.php**
En el archivo start.php de tu proyecto, carga `vendor/autoload.php` al inicio, por ejemplo:
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
```

**4. Reinicia Workerman**
`php start.php restart`

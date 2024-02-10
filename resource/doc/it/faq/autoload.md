# Autoload

Se il tuo progetto non ha implementato l'autoload automatico, puoi utilizzare Composer per generarne uno, seguendo i passaggi seguenti.

**1. Modifica composer.json**  
Aggiungi `"" : "./"` a autoload.psr-4 in composer.json, ad esempio:
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
Esegui `composer dumpautoload`

**3. Carica autoload.php**  
Nel file start.php del tuo progetto, carica `vendor/autoload.php`, ad esempio:
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
```

**4. Riavvia workerman**  
`php start.php restart`

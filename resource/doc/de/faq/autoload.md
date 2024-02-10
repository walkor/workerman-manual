# Automatisches Laden

Wenn Ihr Projekt kein automatisches Laden implementiert hat, können Sie mit Composer ein solches generieren. Hier sind die Schritte.

**1. Ändern Sie composer.json**
Fügen Sie unter autoload.psr-4 in der composer.json `"": "./"` hinzu, zum Beispiel:
```json
{
    "require": {
        "workerman/workerman" : "^v4.0.0"
    },
    "autoload": {
        "psr-4": {
            "": "./"
        }
    }
}
```

**2. Generieren Sie autoload.php**
Führen Sie `composer dumpautoload` aus.

**3. Laden Sie autoload.php**
Laden Sie `vendor/autoload.php` am Anfang Ihrer Projektstartdatei, zum Beispiel:
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
```

**4. Workerman neu starten**
`php start.php restart`

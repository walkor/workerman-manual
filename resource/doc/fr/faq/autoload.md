# Chargement automatique

Si votre projet n'a pas implémenté de chargement automatique, vous pouvez en générer un en utilisant Composer, comme suit.

**1. Modifier composer.json**  
Ajoutez `"": "./"` à autoload.psr-4 de votre fichier composer.json, par exemple :
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

**2. Générer autoload.php**  
Exécutez `composer dumpautoload`

**3. Charger autoload.php**  
Chargez `vendor/autoload.php` au début de votre fichier start.php, par exemple :
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
```

**4. Redémarrer workerman**  
`php start.php restart`

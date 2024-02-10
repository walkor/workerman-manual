# Carregamento automático

Se o seu projeto não implementa o carregamento automático, você pode gerar um usando o Composer, seguindo as etapas abaixo.

**1. Modifique o composer.json**
Adicione `"" : "./"` em autoload.psr-4 no arquivo composer.json, por exemplo:
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

**2. Gerar autoload.php**
Execute `composer dumpautoload`

**3. Carregar autoload.php**
Carregue `vendor/autoload.php` no início do arquivo start.php, por exemplo:
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
```

**4. Reinicie o workerman**
`php start.php restart`

# workerman/crontab

# Descripción
`workerman/crontab` es un programa de tareas programadas basado en workerman, similar a crontab en Linux. `workerman/crontab` admite programación a nivel de segundos.

> Para usar `workerman/crontab`, es necesario configurar correctamente la zona horaria de PHP, de lo contrario, los resultados de la ejecución pueden no ser los esperados.

## Descripción del tiempo
```php
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ día de la semana (0 - 6) (Domingo=0)
|   |   |   |   +------ mes (1 - 12)
|   |   |   +-------- día del mes (1 - 31)
|   |   +---------- hora (0 - 23)
|   +------------ minuto (0 - 59)
+-------------- segundo (0-59)[Puede omitirse, si no está presente la posición 0, la unidad de tiempo mínima es el minuto]
```

# Instalación
```php
composer require workerman/crontab
```

# Ejemplo
```php
<?php
use Workerman\Worker;
require __DIR__ . '/vendor/autoload.php';

use Workerman\Crontab\Crontab;
$worker = new Worker();

// Configurar la zona horaria para evitar discrepancias entre los resultados y las expectativas
date_default_timezone_set('PRC');

$worker->onWorkerStart = function () {
    // Ejecutar en el primer segundo de cada minuto.
    new Crontab('1 * * * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
    // Ejecutar a las 7:50 todos los días, se omite la posición de los segundos.
    new Crontab('50 7 * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
};

Worker::runAll();
```

> Nota: Las tareas programadas no se ejecutarán de inmediato, iniciarán en el próximo minuto.

# Interfaz
**Crontab::destroy()**

Destruye el temporizador
```php
$crontab = new Crontab('1 * * * * *', function(){
    echo date('Y-m-d H:i:s')."\n";
});
$crontab->destroy();
```

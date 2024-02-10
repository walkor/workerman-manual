# workerman/crontab

# Descrição
`workerman/crontab` é um programa de tarefas agendadas baseado no workerman, semelhante ao crontab do Linux. O `workerman/crontab` suporta agendamento até o nível de segundos.

> Para usar o `workerman/crontab`, você precisa primeiro configurar o fuso horário do PHP, caso contrário, os resultados da execução podem não corresponder ao esperado.

## Explicação do tempo
```plaintext
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ dia da semana (0 - 6) (Domingo=0)
|   |   |   |   +------ mês (1 - 12)
|   |   |   +-------- dia do mês (1 - 31)
|   |   +---------- hora (0 - 23)
|   +------------ minuto (0 - 59)
+-------------- segundo (0-59)[opcional, se não estiver presente, o menor intervalo de tempo é minuto]
```

# Instalação
```plaintext
composer require workerman/crontab
```

# Exemplo
```php
<?php
use Workerman\Worker;
require __DIR__ . '/vendor/autoload.php';

use Workerman\Crontab\Crontab;
$worker = new Worker();

// Definir fuso horário para evitar resultados inesperados
date_default_timezone_set('PRC');

$worker->onWorkerStart = function () {
    // Executar a cada minuto no primeiro segundo.
    new Crontab('1 * * * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
    // Executar todos os dias às 7:50, observe que o segundo foi omitido aqui.
    new Crontab('50 7 * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
};

Worker::runAll();
```

> Observação: As tarefas agendadas não serão executadas imediatamente; todas as tarefas agendadas começarão a contar a partir do próximo minuto.

# Interface
**Crontab::destroy()**

Destruir o temporizador
```plaintext
$crontab = new Crontab('1 * * * * *', function(){
    echo date('Y-m-d H:i:s')."\n";
});
$crontab->destroy();
```

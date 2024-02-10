# onWorkerReload
Requerido```(workerman >= 3.2.5)```
## Descrição:
```php
callback Worker::$onWorkerReload
```
Este recurso não é frequentemente utilizado.

Configura o callback que será executado quando o Worker receber um sinal de recarga.

O callback onWorkerReload pode ser utilizado para realizar diversas tarefas, como recarregar arquivos de configuração de negócios sem a necessidade de reiniciar o processo.

**Atenção**:

O comportamento padrão do processo filho ao receber o sinal de recarga é sair e reiniciar, para que um novo processo possa recarregar o código de negócios e completar a atualização do código. Portanto, é normal que o processo filho saia imediatamente após a execução do callback onWorkerReload.

Se deseja que o processo filho execute apenas o onWorkerReload após receber o sinal de recarga e não saia, é possível configurar a propriedade reloadable da instância do Worker correspondente como false ao inicializar a instância do Worker.

## Parâmetros da função de callback
``` $worker ```

O objeto Worker

## Exemplo

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Define reloadable como false, ou seja, o processo filho não executa a reinicialização ao receber o sinal de recarga
$worker->reloadable = false;
// Após a recarga, notifica todos os clientes que o servidor executou uma recarga
$worker->onWorkerReload = function(Worker $worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send('worker reloading');
    }
};
// Executa o worker
Worker::runAll();
```

Nota: Além de utilizar uma função anônima como callback, também é possível [consultar aqui](../faq/callback_methods.md) outras formas de escrever callbacks.

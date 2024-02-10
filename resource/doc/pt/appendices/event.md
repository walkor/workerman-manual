# Eventos suportados atualmente pelo workerman

| Nome  | Extensão Dependente | Suporte a Corotinas |  Prioridade  |  Versão do Workerman  |
|-----|------|--|-----|
|  Workerman\Events\Select   |   Nenhum   | Não suportado  |  Suporte padrão do kernel   |  >=3.0  ｜
|  Workerman\Events\Revolt   |   event(opcional)   | Suportado |  Requer a instalação de [revolt/event-loop](https://github.com/revoltphp/event-loop)   |  >=5.0  |
|  Workerman\Events\Event   |   event   | Não suportado |  Suporte padrão do kernel   |  >=3.0  |
|  Workerman\Events\Swoole   |  [swoole](https://github.com/swoole/swoole-src)   | Suportado |  Requer configuração manual   |  >=4.0  |
|  Workerman\Events\Swow   |   [swow](https://github.com/swow/swow)   | Suportado |  Requer configuração manual   |  >=5.0  |

* Cada driver de kernel fornece recursos exclusivos, por exemplo, usar `Revolt` permitirá que o workerman suporte o [Fiber Coroutine](https://www.php.net/manual/zh/language.fibers.php) embutido no PHP, enquanto o uso de `Swoole` permitirá que o workerman suporte corotinas do Swoole.
* Os diferentes drivers de eventos são mutuamente exclusivos, por exemplo, ao usar a corotina de Fibra do `Revolt`, não é possível usar as corotinas de `Swoole` ou `Swow`.
* `Revolt` requer a instalação de `composer require revolt/event-loop ^1.0.0`, após a instalação, o kernel do workerman configurará automaticamente o `Revolt` como o driver de evento preferencial.
* `Swoole` e `Swow` devem ser configurados manualmente usando `Worker::$eventLoopClass` para que tenham efeito (consulte o próximo parágrafo).
* Por padrão, o swoole não tem o [Runtime de Corotinas de Uma Tecla](https://wiki.swoole.com/#/runtime?id=runtime) ativado, o que significa que chamadas bloqueadoras ainda são feitas para Pdo, Redis e leitura/gravação de arquivos embutidos no PHP.
* Se desejar ativar o Runtime de Corotinas de Uma Tecla do swoole, é necessário chamar manualmente `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);`.

> **Nota**
> A extensão swow automaticamente alterará o comportamento de algumas funções embutidas do PHP, o que pode resultar em incapacidade do workerman de responder a solicitações e sinais se a extensão swow estiver ativada, mas não estiver sendo usada como driver de evento. Portanto, se você não estiver usando o swow como driver de evento de alto nível, será necessário comentar o swow no php.ini.

Para mais informações, consulte [corotinas do workerman](../fiber.md).

# Configurando manualmente o driver de evento para o workerman

A seguir está a configuração manual do driver de evento para o workerman.

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// Configurando manualmente como o driver de evento de alto nível
Worker::$eventLoopClass = Workerman\Events\Revolt::class;
//Worker::$eventLoopClass = Workerman\Events\Select::class;
//Worker::$eventLoopClass = Workerman\Events\Event::class;
//Worker::$eventLoopClass = Workerman\Events\Swoole::class;
//Worker::$eventLoopClass = Workerman\Events\Swow::class;
$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    Coroutine::create(function() use ($connection) {
        $cli = new Client('example.com', 80);
        $cli->get('/get');
        $connection->send($cli->body);
    });
};

Worker::runAll();
```

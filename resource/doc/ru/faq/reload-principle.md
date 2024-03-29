# Принцип плавной перезагрузки
## Что такое плавная перезагрузка?

Плавная перезагрузка отличается от обычной перезагрузки тем, что она позволяет перезапустить службу (обычно короткосрочное соединение) без воздействия на пользователей, чтобы перезагрузить программу PHP и обновить бизнес-код.

Плавная перезагрузка обычно используется в процессе обновления бизнеса или выпуска версий, чтобы избежать временной недоступности службы из-за перезагрузки во время публикации кода.

> **Примечание**
> Операционная система Windows не поддерживает перезагрузку.

> **Примечание**
> Для бизнес-подключений (например, websocket) соединение будет разорвано при плавной перезагрузке процесса. Решение состоит в использовании архитектуры, подобной [gatewayWorker](https://www.workerman.net/doc/gateway-worker), где группа процессов специально поддерживает соединение, и свойство [reloadable](../worker/reloadable.md) для этой группы процессов установлено в false. Бизнес-логика запускается в другой группе рабочих процессов, и взаимодействие между шлюзом и рабочими процессами осуществляется по протоколу TCP. Когда бизнес нуждается в изменениях, просто перезагрузите рабочий процесс.

## Ограничения
**Обратите внимание: только файлы, загруженные автоматически через on{...} обратные вызовы, будут автоматически обновлены после плавного перезапуска, файлы, загруженные напрямую в запускаемом сценарии или жестко закодированный код, не будут автоматически обновлены при перезагрузке.**

#### Следующий код после перезагрузки не будет обновлен
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onMessage = function($connection, $request) {
    $connection->send('hi'); // Жестко закодированный код не поддерживает горячее обновление
};
```

```php
$worker = new Worker('http://0.0.0.0:1234');
require_once __DIR__ . '/your/path/MessageHandler.php'; // Файл, загруженный напрямую в запускаемом сценарии, не поддерживает горячее обновление
$messageHandler = new MessageHandler();
$worker->onMessage = [$messageHandler, 'onMessage']; // Предположим, что класс MessageHandler имеет метод onMessage
```

#### Следующий код после перезагрузки будет автоматически обновлен
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onWorkerStart = function($worker) { // onWorkerStart вызывается после запуска процесса
    require_once __DIR__ . '/your/path/MessageHandler.php'; // Файл, загруженный после запуска процессом, поддерживает горячее обновление
    $messageHandler = new MessageHandler();
    $worker->onMessage = [$messageHandler, 'onMessage'];
};
```
После внесения изменений в MessageHandler.php выполните `php start.php reload`, чтобы обновить бизнес-логику.

> **Подсказка**
> В приведенном выше коде для наглядности используется оператор `require_once`. Если ваш проект поддерживает автозагрузку psr4, тогда вызывать оператор `require_once` не нужно.

## Принцип плавной перезагрузки

Workerman состоит из главного процесса и дочерних процессов, главный процесс отвечает за мониторинг дочерних процессов, а дочерние процессы принимают подключения от клиентов и обрабатывают приходящие от них данные, затем возвращают данные клиентам. При обновлении бизнес-кода необходимо обновить только дочерние процессы.

Когда главный процесс Workerman получает сигнал для плавной перезагрузки, он отправляет дочернему процессу сигнал безопасного выхода (чтобы процесс обработал текущий запрос перед выходом). После выхода этого процесса главный процесс создает новый дочерний процесс (в котором загружен новый код PHP), затем главный процесс отправляет стоп-команду другому старому процессу, и таким образом, происходит последовательный перезапуск процесса за процессом, пока все старые процессы не будут заменены новыми.

Мы видим, что плавная перезагрузка фактически заключается в пошаговом выходе старых бизнес-процессов, а затем пошаговом создании новых процессов. Чтобы не влиять на клиентов во время плавной перезагрузки, требуется, чтобы процессы не сохраняли информацию о состоянии пользователя, то есть предпочтительно, чтобы бизнес-процессы были без состояния, чтобы избежать потери информации из-за выхода процесса.

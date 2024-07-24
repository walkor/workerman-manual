# Перед разработкой

Для разработки приложений на Workerman вам необходимо ознакомиться с следующей информацией:

## I. Отличия от разработки на обычном PHP

Помимо того, что нельзя напрямую использовать переменные и функции, связанные с протоколом HTTP, разработка на Workerman не имеет существенных отличий от обычной разработки на PHP.

### 1. Различия в протоколах прикладного уровня
* В обычной разработке PHP обычно используется протокол прикладного уровня HTTP, где сервер веб-приложений уже выполняет разбор протокола.
* Workerman поддерживает различные протоколы, в настоящее время встроены HTTP, WebSocket и другие протоколы. Workerman рекомендует разработчикам использовать более простой пользовательский протокол обмена сообщениями.

*Для разработки приложений, использующих протокол HTTP, см. раздел [HTTP-сервер](../http/request.md).*

### 2. Различия в жизненном цикле запроса
* В PHP после завершения запроса к веб-приложению все переменные и ресурсы освобождаются.
* Приложения, разработанные на Workerman, остаются в памяти после первой загрузки и парсинга, что позволяет сохранить определение классов, глобальные объекты и статические члены класса для повторного использования.

### 3. Осторожно с повторным определением классов и констант
* Поскольку Workerman кеширует скомпилированные файлы PHP, необходимо избегать многократного использования require/include для одного и того же определения классов или констант. Рекомендуется использовать require_once/include_once для загрузки файлов.

### 4. Внимание к освобождению ресурсов в режиме единичного экземпляра
* Поскольку Workerman не освобождает глобальные объекты и статические члены класса после каждого запроса, в случае использования одиночного экземпляра, например, для базы данных, соединение с базой данных (включая сокетное соединение с базой данных) обычно сохраняется в статическом члене класса, что позволяет Workerman повторно использовать это сокетное соединение в течение жизненного цикла процесса. Следует учитывать, что если сервер базы данных обнаруживает неактивное соединение в течение определенного времени, он может закрыть сокетное соединение, что приведет к ошибке при повторном использовании этого экземпляра базы данных (например, подобной ошибке, как mysql gone away). Workerman предоставляет [класс для работы с базой данных](../components/workerman-mysql.md) с возможностью повторного подключения, который может быть использован непосредственно разработчиком.

### 5. Избегайте использования операторов exit и die
* Workerman работает в режиме командной строки PHP, поэтому вызов операторов exit и die приводит к завершению текущего процесса. Хотя дочерний процесс после завершения будет немедленно создан заново для продолжения обслуживания, все равно возможно негативное воздействие на бизнес-процессы.

### 6. Необходимость перезапуска службы после изменения кода
Поскольку Workerman постоянно находится в памяти, определение классов PHP и функций загружается один раз и остается в памяти, не считываясь с диска в каждый раз после изменения бизнес-логики. Поэтому после внесения изменений в код необходимо перезапустить службу, чтобы изменения вступили в силу.

## II. Основные понятия

### 1. Протокол передачи данных через TCP
TCP является надежной основанной на IP протокола уровня передачи данных, который требует установления соединения. Одной из важных характеристик TCP является то, что он основан на потоке данных, при этом запросы клиента постоянно поступают на сервер, сервер может получать данные, которые не являются полным запросом, а также совмещение нескольких запросов. Это требует установления правил для разбора границ каждого запроса в потоке посредством протоколов прикладного уровня.

### 2. Протокол прикладного уровня

Протокол прикладного уровня (application layer protocol) определяет, как процессы прикладных программ на различных конечных системах (клиенты, серверы) взаимодействуют между собой для передачи сообщений, такие как HTTP, WebSocket. Например, простой протокол прикладного уровня может выглядеть следующим образом: `{"module":"user","action":"getInfo","uid":456}\n`. В этом протоколе завершение запроса обозначено символом `\n` (обратите внимание, что `\n` обозначает символ новой строки), а тело сообщения представлено строкой.

### 3. Кратковременное соединение

Кратковременное соединение означает установление соединения между обеими сторонами только в момент обмена данными, после передачи данных соединение закрывается. Например, HTTP-сервисы веб-сайтов обычно используют кратковременное соединение.

*Разработка приложений с использованием кратковременного соединения можно найти в главе основного процесса разработки*

### 4. Долгосрочное соединение

Долгосрочное соединение позволяет передавать несколько пакетов данных через одно соединение. 

Примечание: приложения, использующие долгосрочное соединение, обязательно должны использовать [сигнализацию оживления](../faq/heartbeat.md), в противном случае соединение может быть разорвано межконечным маршрутизатором из-за неактивности в течение длительного времени.

Долгосрочное соединение широко используется в случае частой оперативности и точного взаимодействия. Каждое TCP-соединение требует выполнения трехэтапного рукопожатия, что занимает время. Если каждая операция начинается с установления соединения, то обработка замедлится. Поэтому при долгосрочном соединении после каждой операции соединение не закрывается, а следующий раз передается пакет данных напрямую, без установления TCP-соединения. Например, долгосрочное соединение используется для соединения с базой данных, и если использовать кратковременное соединение для частого общения, то возникнут ошибки в работе с сокетом, и инициирование сокета для каждой операции приведет к излишнему расходу ресурсов.

*При разработке приложений с долгосрочным соединением см. процесс разработки шлюза/воркера*

### 5. Плавная перезагрузка

Обычный процесс перезагрузки заключается в полном останове всех процессов, за которым следует создание полностью новых служб. Во время этого процесса на короткое время никакой из процессов не предоставляет услуг, что может привести к временной недоступности службы, что в условиях высокой нагрузки может привести к сбоям запросов.

Плавная перезагрузка не заключается в одновременном останове всех процессов, а в поочередном останове каждого процесса сразу после этого создается новый процесс, чтобы заменить старый. Таким образом, старые процессы постепенно заменяются новыми до тех пор, пока все старые процессы не будут заменены новыми.

Для плавной перезагрузки Workerman можно использовать команду `php your_file.php reload`, которая обеспечивает обновление программы без ущерба для качества обслуживания.

**Примечание: только файлы, загруженные автоматически в on{...}, обновляются после плавной перезагрузки. Файлы, загруженные непосредственно в запущенном скрипте или жестко закодированный код не обновляются автоматически.**

## III. Различие между главным и дочерним процессами
Важно отметить, что код работает в главном процессе или в дочернем процессе. Обычно код, запускаемый до вызова `Worker::runAll();`, выполняется в главном процессе, а код, выполняемый в колбэках onXXX, относится к дочернему процессу. Следует учитывать, что код, написанный после вызова `Worker::runAll();`, никогда не будет выполнен.

Вот пример:
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Код выполняется в главном процессе
$tcp_worker = new Worker("tcp://0.0.0.0:2347");
// Присваивание выполняется в главном процессе
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Этот код выполняется в дочернем процессе
    $connection->send('hello ' . $data);
};

Worker::runAll();
```

**Примечание:** не инициализируйте соединение с базой данных, memcache, redis и другие ресурсы в главном процессе, так как главный процесс может по умолчанию наследовать их у дочерних процессов (особенно при использовании единственных экземпляров). Это означает, что все процессы используют одно и то же соединение, и сервер возвращает данные через это соединение, которые могут быть прочитаны в нескольких процессах, что приводит к неправильным данным. Таким же образом, если хотя бы один процесс закрывает соединение (например, когда главный процесс завершается в режиме демона), все дочерние процессы будут закрыты и произойдут непредсказуемые ошибки, такие как mysql gone away. Рекомендуется инициализировать ресурсы соединения в обработчике onWorkerStart.


## Как настроить протокол

Фактически, создание собственного протокола - довольно простая задача. Простой протокол обычно состоит из двух частей:
 * Идентификатор, разделяющий границы данных
 * Определение формата данных

## Пример

### Определение протокола
Допустим, идентификатором, разделяющим границы данных, является символ переноса строки "\n" (заметьте, что внутри самих данных запроса не может содержаться символ переноса строки), и формат данных представлен в виде Json. Ниже приведен запрос, соответствующий этим правилам.

<pre>
{"type":"message","content":"hello"}
</pre>

Обратите внимание, что в конце данных запроса находится символ переноса строки (в PHP он представлен как строка в двойных кавычках "\n"), который обозначает завершение запроса.

### Шаги реализации
В Workerman, если вы хотите реализовать описанный выше протокол, предположим, что его название - JsonNL, а проект называется MyApp, вам потребуется выполнить следующие шаги:

1. Разместите файл протокола в папке проекта Protocols, например, файл MyApp/Protocols/JsonNL.php.

2. Реализуйте класс JsonNL с использованием пространства имен `namespace Protocols;`, который должен содержать три статических метода: input, encode и decode.

Примечание: Workerman автоматически вызывает эти три статических метода для реализации фрагментации, сборки и упаковки данных. Конкретный процесс описан в разделе "Работа с классом протокола".

### Взаимодействие между Workerman и классом протокола
1. Предположим, что клиент отправляет пакет данных серверу. После получения данных (возможно, только части данных) сервер сразу вызывает метод `input` протокола для определения длины этого пакета. Метод `input` возвращает значение длины `$length` фрейма для Workerman.
2. После того, как Workerman получает значение `$length`, он проверяет, содержит ли текущий буфер данных уже длину `$length`. Если нет, он продолжает ожидать данных до тех пор, пока длина буфера данных не станет не меньше `$length`.
3. После достаточной длины буфера данных, Workerman извлекает фрагмент данных длиной `$length` (т.е. **фрагментация**) и вызывает метод `decode` протокола для **разбора фрагмента**. Результат разбора сохраняется в переменной `$data`.
4. После разбора данных, Workerman передает данные `$data` в качестве второго аргумента обратного вызова` onMessage($connection, $data)` для обработки в бизнес-логике. Теперь бизнес-логика может использовать переменную `$data`, чтобы получить полные и разобранные данные, отправленные клиентом.
5. Когда бизнес-логике требуется отправить данные клиенту, вызов `$connection->send($buffer)` будет автоматически использовать метод `encode` протокола для **упаковки** `$buffer` перед отправкой клиенту.

### Конкретная реализация

**Реализация в файле MyApp/Protocols/JsonNL.php**
```php
namespace Protocols;
class JsonNL
{
    /**
     * Проверка целостности пакета
     * Если можно получить длину пакета в $recv_buffer, возвращается длина всего пакета
     * В противном случае возвращается 0, что означает, что необходимо дождаться дополнительных данных.
     * Если возникает ошибка в протоколе, можно вернуть false, что приведет к разрыву соединения с текущим клиентом
     * @param string $recv_buffer
     * @return int
     */
    public static function input($recv_buffer)
    {
        // Получение позиции символа переноса строки "\n"
        $pos = strpos($buffer, "\n");
        // Если нет символа переноса строки, невозможно получить длину пакета, вернуть 0 и ожидать дополнительных данных
        if($pos === false)
        {
            return 0;
        }
        // Если символ переноса строки имеется, вернуть длину текущего пакета (включая символ переноса строки)
        return $pos+1;
    }

    /**
     * Упаковка, автоматически вызывается при отправке данных клиенту
     * @param string $buffer
     * @return string
     */
    public static function encode($buffer)
    {
        // JSON сериализация и добавление символа переноса строки в качестве пометки окончания запроса
        return json_encode($buffer)."\n";
    }

    /**
     * Разбор, автоматически вызывается, когда количество полученных байтов равно возвращенному значению input (значение больше 0)
     * и передается как параметр $data в обратный вызов onMessage
     * @param string $buffer
     * @return string
     */
    public static function decode($buffer)
    {
        // Удаление символа переноса строки и восстановление в массив
        return json_decode(trim($buffer), true);
    }
}
```

Теперь протокол JsonNL реализован и может быть использован в проекте MyApp. Пример использования приведен ниже.

Файл: MyApp\start.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$json_worker = new Worker('JsonNL://0.0.0.0:1234');
$json_worker->onMessage = function(TcpConnection $connection, $data) {

    // $data - это данные, отправленные клиентом, которые уже были обработаны методом JsonNL::decode
    echo $data;
    
    // Данные, отправленные через $connection->send, автоматически упаковываются методом JsonNL::encode и затем отправляются клиенту
    $connection->send(array('code'=>0, 'msg'=>'ok'));
    
};
Worker::runAll();
```

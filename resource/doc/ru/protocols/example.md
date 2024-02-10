# Некоторые примеры

## Пример 1

### Определение протокола
  * Первые 10 байтов предназначены для сохранения общей длины пакета, дополнены нулями до нужной длины
  * Формат данных - XML

### Образец данных пакета
```xml
0000000121<?xml version="1.0" encoding="ISO-8859-1"?>
<request>
    <module>user</module>
    <action>getInfo</action>
</request>
```
Где 0000000121 представляет собой общую длину пакета, за которым следует содержание пакета в формате XML.

### Реализация протокола
```php
namespace Protocols;
class XmlProtocol
{
    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer) < 10)
        {
            // Недостаточно 10 байт, возвращаем 0 и ждем дополнительных данных
            return 0;
        }
        $total_len = base_convert(substr($recv_buffer, 0, 10), 10, 10);
        return $total_len;
    }

    public static function decode($recv_buffer)
    {
        // Пакет запроса
        $body = substr($recv_buffer, 10);
        return simplexml_load_string($body);
    }

    public static function encode($xml_string)
    {
        $total_length = strlen($xml_string)+10;
        $total_length_str = str_pad($total_length, 10, '0', STR_PAD_LEFT);
        return $total_length_str . $xml_string;
    }
}
```

## Пример 2

### Определение протокола
  * Первые 4 байта представляют собой беззнаковое целое в сетевом порядке байтов и указывают длину всего пакета
  * Часть данных представлена в виде строки JSON

### Образец данных пакета
<pre>
****{"type":"message","content":"hello all"}
</pre>

Где первые 4 символа **** обозначают беззнаковое целое число в сетевом порядке байтов, затем следует данные в формате JSON.

### Реализация протокола
```php
namespace Protocols;
class JsonInt
{
    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer)<4)
        {
            return 0;
        }
        $unpack_data = unpack('Ntotal_length', $recv_buffer);
        return $unpack_data['total_length'];
    }

    public static function decode($recv_buffer)
    {
        $body_json_str = substr($recv_buffer, 4);
        return json_decode($body_json_str, true);
    }

    public static function encode($data)
    {
        $body_json_str = json_encode($data);
        $total_length = 4 + strlen($body_json_str);
        return pack('N',$total_length) . $body_json_str;
    }
}
```

## Пример 3 (загрузка файла с использованием двоичного протокола)

### Определение протокола
```C
struct
{
  unsigned int total_len;  // Длина всего пакета в сетевом порядке байтов
  char         name_len;   // Длина имени файла
  char         name[name_len]; // Имя файла
  char         file[total_len - BinaryTransfer::PACKAGE_HEAD_LEN - name_len]; // Данные файла
}
```

### Образец данных пакета
<pre> *****logo.png****************** </pre>

Где первые 5 символов ***** представляют собой беззнаковое целое в сетевом порядке байтов, следующий символ хранит длину имени файла, а затем идут имя файла и сырые двоичные данные файла.

### Реализация протокола
```php
namespace Protocols;
class BinaryTransfer
{
    // Длина заголовка протокола
    const PACKAGE_HEAD_LEN = 5;

    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer) < self::PACKAGE_HEAD_LEN)
        {
            return 0;
        }
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        return $package_data['total_len'];
    }


    public static function decode($recv_buffer)
    {
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        $name_len = $package_data['name_len'];
        $file_name = substr($recv_buffer, self::PACKAGE_HEAD_LEN, $name_len);
        $file_data = substr($recv_buffer, self::PACKAGE_HEAD_LEN + $name_len);
         return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        return $data;
    }
}
```

### Пример использования протокола на сервере

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('BinaryTransfer://0.0.0.0:8333');
// Сохранить файл в tmp-директорию
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### Пример использования клиента для файлов client.php (здесь используется PHP для имитации клиента загрузки)

```php
<?php
/** Клиент для загрузки файла **/
// Адрес сервера
$address = "127.0.0.1:8333";
// Проверка параметра пути к загружаемому файлу
if(!isset($argv[1]))
{
   exit("Использование: php client.php \$file_path\n");
}
// Путь к загружаемому файлу
$file_to_transfer = trim($argv[1]);
// Локальный файл для загрузки не существует
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer не существует\n");
}
// Установка соединения через сокет
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
// Установка блокировки
stream_set_blocking($client, 1);
// Имя файла
$file_name = basename($file_to_transfer);
// Длина имени файла
$name_len = strlen($file_name);
// Двоичные данные файла
$file_data = file_get_contents($file_to_transfer);
// Длина заголовка протокола: 4 байта для длины пакета + 1 байт для длины имени файла
$PACKAGE_HEAD_LEN = 5;
// Формирование пакета
$package = pack('NC', $PACKAGE_HEAD_LEN  + strlen($file_name) + strlen($file_data), $name_len) . $file_name . $file_data;
// Загрузка файла
fwrite($client, $package);
// Вывод результата
echo fread($client, 8192),"\n";
``` 

### Пример использования клиента
Запуск в командной строке ```php client.php <путь_к_файлу>```

Например, ```php client.php abc.jpg```
## Пример четыре (загрузка файла с использованием текстового протокола)

### Определение протокола

json+перевод строки, в json содержится имя файла и base64-кодированные данные файла (увеличивает объем на 1/3)

### Образец протокола

{"file_name":"logo.png","file_data":"PD9waHAKLyo......"}\n

Обратите внимание, что в конце стоит символ перевода строки, в PHP он обозначается как ```"\n"```

### Реализация протокола
```php
namespace Protocols;
class TextTransfer
{
    public static function input($recv_buffer)
    {
        $recv_len = strlen($recv_buffer);
        if($recv_buffer[$recv_len-1] !== "\n")
        {
            return 0;
        }
        return strlen($recv_buffer);
    }

    public static function decode($recv_buffer)
    {
        // Распаковка
        $package_data = json_decode(trim($recv_buffer), true);
        // Извлечение имени файла
        $file_name = $package_data['file_name'];
        // Извлечение закодированных в base64 данных файла
        $file_data = $package_data['file_data'];
        // Декодирование base64 для восстановления исходных двоичных данных файла
        $file_data = base64_decode($file_data);
        // Возвращение данных
        return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // Можно кодировать данные, отправляемые клиенту, в соответствии с вашими потребностями, здесь они просто возвращаются как текст
        return $data;
    }
}

```

### Пример использования протокола на сервере
Примечание: этот способ написания практически такой же, как способ загрузки двоичных данных, позволяя почти без изменений переключить протоколы.

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('TextTransfer://0.0.0.0:8333');
// Сохранение файла в папке tmp
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### Пример использования клиента для файла textclient.php (здесь используется PHP для имитации клиента для загрузки)
```php
<?php
/** Клиент для загрузки файлов **/
// Адрес загрузки
$address = "127.0.0.1:8333";
// Проверка параметра пути загружаемого файла
if(!isset($argv[1]))
{
   exit("Использование: php client.php \$file_path\n");
}
// Путь загружаемого файла
$file_to_transfer = trim($argv[1]);
// Локального файла для загрузки не существует
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer не существует\n");
}
// Установка соединения с сокетом
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
stream_set_blocking($client, 1);
// Имя файла
$file_name = basename($file_to_transfer);
// Бинарные данные файла
$file_data = file_get_contents($file_to_transfer);
// Кодирование в base64
$file_data = base64_encode($file_data);
// Пакет данных
$package_data = array(
    'file_name' => $file_name,
    'file_data' => $file_data,
);
// Пакет протокола json+перевод строки
$package = json_encode($package_data)."\n";
// Выполнение загрузки
fwrite($client, $package);
// Вывод результата
echo fread($client, 8192),"\n";
```

### Пример использования клиента
Запустить в командной строке ```php textclient.php <путь_к_файлу>```

Например, ```php textclient.php abc.jpg```

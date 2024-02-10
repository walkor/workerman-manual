# Workerman/MySQL

## Описание
При постоянном пребывании программы в памяти при использовании MySQL часто возникает ошибка "mysql gone away", которая происходит из-за отсутствия связи между программой и MySQL на протяжении длительного времени, что приводит к разрыву соединения со стороны сервера MySQL. Этот класс базы данных может решить эту проблему, автоматически повторяя попытку при возникновении ошибки "mysql gone away".

## Зависимости
Этот класс MySQL зависит от двух расширений: [pdo](https://php.net/manual/ru/book.pdo.php) и [pdo_mysql](https://php.net/manual/ru/ref.pdo-mysql.php). Отсутствие этих расширений приведет к ошибке "Undefined class constant 'MYSQL_ATTR_INIT_COMMAND' in ....".

Выполнение команды `php -m` отобразит список всех установленных расширений для PHP CLI. Если отсутствуют pdo или pdo_mysql, установите их самостоятельно.

**Система centos**
PHP5.x
```
yum install php-pdo
yum install php-mysql
```
PHP7.x
```
yum install php70w-pdo_dblib.x86_64
yum install php70w-mysqlnd.x86_64
```
Если вы не можете найти названия пакетов, попробуйте выполнить поиск с помощью команды `yum search php mysql`.

**Система ubuntu/debian**
PHP5.x
```
apt-get install php5-mysql
```
PHP7.x
```
apt-get install php7.0-mysql
```
Если вы не можете найти названия пакетов, попробуйте выполнить поиск с помощью команды `apt-cache search php mysql`.

**Не удается установить с помощью перечисленных методов?**
Если перечисленные методы установки не сработали, обратитесь к [руководству по Workerman - Приложение - Установка расширений - Метод 3: Установка из исходного кода](../appendices/install-extension.md).

## Установка Workerman/MySQL
**Метод 1:**

Вы можете установить через composer, выполните следующие команды в командной строке (источник composer находится за пределами страны, процесс установки может занять много времени).
````
composer require workerman/mysql
````
После успешного выполнения данной команды будет создан каталог vendor, после чего подключите autoload.php из каталога в проекте.
```php
require_once __DIR__ . '/vendor/autoload.php';
``` 

**Метод 2:**

[Скачайте исходный код](https://github.com/walkor/mysql/archive/master.zip), распакуйте каталог в свой проект (любое местоположение) и подключите исходный файл напрямую.
```php
require_once '/your/path/of/mysql-master/src/Connection.php';
```


## Внимание
Настоятельно рекомендуется инициализировать подключение к базе данных в обратном вызове onWorkerStart, чтобы избежать инициализации подключения до запуска `Worker::runAll();`, так как подключение, инициализированное до запуска `Worker::runAll()`, принадлежит к основному процессу, и дочерние процессы унаследуют это подключение, что может привести к ошибкам из-за одновременного использования одного и того же подключения к базе данных главным и дочерними процессами.

## Пример
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    // Сохранение экземпляра базы данных в глобальной переменной (можно также сохранить в статический член класса)
    global $db;
    $db = new \Workerman\MySQL\Connection('хост', 'порт', 'пользователь', 'пароль', 'имя_бд');
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Получение экземпляра базы данных через глобальную переменную
    global $db;
    // Выполнение SQL-запроса
    $all_tables = $db->query('show tables');
    $connection->send(json_encode($all_tables));
};
// Запуск воркера
Worker::runAll();
``` 

## Особенности использования MySQL/Connection
```php
// Инициализация подключения к базе данных
$db = new \Workerman\MySQL\Connection('хост', 'порт', 'пользователь', 'пароль', 'имя_бд');

// Получение всех данных
$db->select('ID,Sex')->from('Persons')->where('sex= :sex AND ID = :id')->bindValues(array('sex'=>'M', 'id' => 1))->query();
//эквивалентно
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' AND ID = 1")->query();
//эквивалентно
$db->query("SELECT ID,Sex FROM `Persons` WHERE sex='M' AND ID = 1");


// Получение одной строки данных
$db->select('ID,Sex')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->row();
//эквивалентно
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' ")->row();
//эквивалентно
$db->row("SELECT ID,Sex FROM `Persons` WHERE sex='M'");


// Получение одного столбца данных
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->column();
//эквивалентно
$db->select('ID')->from('Persons')->where("sex= 'F' ")->column();
//эквивалентно
$db->column("SELECT `ID` FROM `Persons` WHERE sex='M'");

// Получение одного значения
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->single();
//эквивалентно
$db->select('ID')->from('Persons')->where("sex= 'F' ")->single();
//эквивалентно
$db->single("SELECT ID FROM `Persons` WHERE sex='M'");

// Сложный запрос
$db->select('*')->from('table1')->innerJoin('table2','table1.uid = table2.uid')->where('age > :age')->groupBy(array('aid'))->having('foo="foo"')->orderByASC/*orderByDESC*/(array('did'))
->limit(10)->offset(20)->bindValues(array('age' => 13));
//эквивалентно
$db->query('SELECT * FROM `table1` INNER JOIN `table2` ON `table1`.`uid` = `table2`.`uid`
WHERE age > 13 GROUP BY aid HAVING foo="foo" ORDER BY did LIMIT 10 OFFSET 20');

// Вставка
$insert_id = $db->insert('Persons')->cols(array(
    'Firstname'=>'abc',
    'Lastname'=>'efg',
    'Sex'=>'M',
    'Age'=>13))->query();
эквивалентно
$insert_id = $db->query("INSERT INTO `Persons` ( `Firstname`,`Lastname`,`Sex`,`Age`)
VALUES ( 'abc', 'efg', 'M', 13)");

// Обновление
$row_count = $db->update('Persons')->cols(array('sex'))->where('ID=1')
->bindValue('sex', 'F')->query();
// эквивалентно
$row_count = $db->update('Persons')->cols(array('sex'=>'F'))->where('ID=1')->query();
// эквивалентно
$row_count = $db->query("UPDATE `Persons` SET `sex` = 'F' WHERE ID=1");

// Удаление
$row_count = $db->delete('Persons')->where('ID=9')->query();
// эквивалентно
$row_count = $db->query("DELETE FROM `Persons` WHERE ID=9");

// Транзакция
$db->beginTrans();
....
$db->commitTrans(); // или $db->rollBackTrans();
```

# Webman/MySQL

## Introducción
Los programas de memoria residente a menudo se encuentran con el error "mysql gone away" al usar mysql. Esto se debe a que la conexión entre el programa y mysql no ha estado en comunicación durante mucho tiempo, lo que hace que el servidor mysql la desconecte. Esta clase de base de datos puede resolver este problema y automáticamente reintenta una vez que se produce el error "mysql gone away".

## Extensiones Dependientes
Esta clase de mysql depende de dos extensiones: [pdo](https://php.net/manual/zh/book.pdo.php) y [pdo_mysql](https://php.net/manual/zh/ref.pdo-mysql.php). La falta de estas extensiones resultará en el error "Undefined class constant 'MYSQL_ATTR_INIT_COMMAND' in ....".

Ejecutar el comando `php -m` en la línea de comandos listar todas las extensiones de php cli instaladas. Si no están instaladas las extensiones pdo o pdo_mysql, por favor, instálelas manualmente.

**Sistemas Centos**
PHP5.x
```sh
yum install php-pdo
yum install php-mysql
```
PHP7.x
```sh
yum install php70w-pdo_dblib.x86_64
yum install php70w-mysqlnd.x86_64
```
En caso de no encontrar el nombre del paquete, intente buscar con `yum search php mysql`.

**Sistemas Ubuntu/Debian**
PHP5.x
```sh
apt-get install php5-mysql
```
PHP7.x
```sh
apt-get install php7.0-mysql
```
Si no encuentra el nombre del paquete, intente buscar con `apt-cache search php mysql`.

**¿No puede instalar con estos métodos?**
Si no puede instalar utilizando estos métodos, por favor, consulte el [manual de workerman - apéndice - instalación de extensiones - método de compilación desde código fuente](../appendices/install-extension.md).

## Instalación de Webman/MySQL
**Método 1:**

Puede instalarlo a través de Composer ejecutando el siguiente comando en la línea de comandos (el origen de Composer está en el extranjero, por lo que la instalación puede ser muy lenta).
```sh
composer require workerman/mysql
```
Después de que el comando anterior sea exitoso, se generará el directorio "vendor", luego incluya el archivo autoload.php en su proyecto.
```php
require_once __DIR__ . '/vendor/autoload.php';
```

**Método 2:**

[Descargue el código fuente](https://github.com/walkor/mysql/archive/master.zip), descomprima el directorio y colóquelo en su proyecto (en cualquier ubicación), luego requiera el archivo fuente directamente.
```php
require_once '/su/ruta/de/mysql-master/src/Connection.php';
```

## Nota
Se recomienda encarecidamente inicializar la conexión a la base de datos en la devolución de llamada onWorkerStart para evitar la inicialización de la conexión antes de que se ejecute `Worker::runAll();`. La conexión inicializada antes de `Worker::runAll();` pertenece al proceso principal, los procesos secundarios heredarán esta conexión y el uso compartido de la misma conexión de base de datos entre el proceso principal y los secundarios puede provocar errores.

## Ejemplo
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    // Almacene la instancia de db en una variable global (también se puede almacenar en miembros estáticos de una clase)
    global $db;
    $db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Obtenga la instancia de db a través de una variable global
    global $db;
    // Ejecute SQL
    $all_tables = $db->query('show tables');
    $connection->send(json_encode($all_tables));
};
// Ejecute el worker
Worker::runAll();
```

## Uso específico de MySQL/Connection
```php
// Inicializar la conexión a la base de datos
$db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');

// Obtener todos los datos
$db->select('ID,Sex')->from('Persons')->where('sex = :sex AND ID = :id')->bindValues(array('sex' => 'M', 'id' => 1))->query();
// Equivalente a
$db->select('ID,Sex')->from('Persons')->where("sex = 'M' AND ID = 1")->query();
// Equivalente a
$db->query("SELECT ID,Sex FROM `Persons` WHERE sex='M' AND ID = 1");

// Obtener una fila de datos
$db->select('ID,Sex')->from('Persons')->where('sex = :sex')->bindValues(array('sex' => 'M'))->row();
// Equivalente a
$db->select('ID,Sex')->from('Persons')->where("sex = 'M' ")->row();
// Equivalente a
$db->row("SELECT ID,Sex FROM `Persons` WHERE sex='M'");

// Obtener una columna de datos
$db->select('ID')->from('Persons')->where('sex = :sex')->bindValues(array('sex' => 'M'))->column();
// Equivalente a
$db->select('ID')->from('Persons')->where("sex = 'F' ")->column();
// Equivalente a
$db->column("SELECT `ID` FROM `Persons` WHERE sex='M'");

// Obtener un único valor
$db->select('ID')->from('Persons')->where('sex = :sex')->bindValues(array('sex' => 'M'))->single();
// Equivalente a
$db->select('ID')->from('Persons')->where("sex = 'F' ")->single();
// Equivalente a
$db->single("SELECT ID FROM `Persons` WHERE sex='M'");

// Consultas complejas
$db->select('*')->from('table1')->innerJoin('table2', 'table1.uid = table2.uid')->where('age > :age')->groupBy(array('aid'))->having('foo="foo"')->orderByASC/*orderByDESC*/(array('did'))
->limit(10)->offset(20)->bindValues(array('age' => 13));
// Equivalente a
$db->query('SELECT * FROM `table1` INNER JOIN `table2` ON `table1`.`uid` = `table2`.`uid`
WHERE age > 13 GROUP BY aid HAVING foo="foo" ORDER BY did LIMIT 10 OFFSET 20');

// Insertar
$insert_id = $db->insert('Persons')->cols(array(
    'Firstname' => 'abc',
    'Lastname' => 'efg',
    'Sex' => 'M',
    'Age' => 13))->query();
// Equivalente a
$insert_id = $db->query("INSERT INTO `Persons` ( `Firstname`,`Lastname`,`Sex`,`Age`)
VALUES ( 'abc', 'efg', 'M', 13)");

// Actualizar
$row_count = $db->update('Persons')->cols(array('sex'))->where('ID = 1')
->bindValue('sex', 'F')->query();
// Equivalente a
$row_count = $db->update('Persons')->cols(array('sex' => 'F'))->where('ID = 1')->query();
// Equivalente a
$row_count = $db->query("UPDATE `Persons` SET `sex` = 'F' WHERE ID=1");

// Eliminar
$row_count = $db->delete('Persons')->where('ID = 9')->query();
// Equivalente a
$row_count = $db->query("DELETE FROM `Persons` WHERE ID=9");

// Transacción
$db->beginTrans();
....
$db->commitTrans(); // o $db->rollBackTrans();
```

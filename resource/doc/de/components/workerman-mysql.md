# Workerman/MySQL

## Erklärung
Bei persistenten Speicherprogrammen tritt bei der Verwendung von MySQL häufig der Fehler "mysql gone away" auf. Dies liegt daran, dass die Verbindung zwischen dem Programm und MySQL für eine lange Zeit keine Kommunikation hatte und die Verbindung von dem MySQL-Server getrennt wurde. Diese Datenbankklasse kann dieses Problem lösen, indem sie beim Auftreten des Fehlers "mysql gone away" automatisch einen erneuten Versuch unternimmt.

## Abhängige Erweiterungen
Diese MySQL-Klasse ist abhängig von den Erweiterungen [pdo](https://php.net/manual/zh/book.pdo.php) und [pdo_mysql](https://php.net/manual/zh/ref.pdo-mysql.php). Fehlt eine dieser Erweiterungen, wird der Fehler "Undefined class constant 'MYSQL_ATTR_INIT_COMMAND' in ...." gemeldet.

Führen Sie den Befehl `php -m` in der Befehlszeile aus, um alle installierten Erweiterungen für PHP CLI anzuzeigen. Wenn pdo oder pdo_mysql fehlen, installieren Sie diese bitte eigenständig.

**CentOS-System**

PHP5.x
```bash
yum install php-pdo
yum install php-mysql
```

PHP7.x
```bash
yum install php70w-pdo_dblib.x86_64
yum install php70w-mysqlnd.x86_64
```
Wenn der Paketname nicht gefunden werden kann, versuchen Sie es mit `yum search php mysql`.

**Ubuntu/Debian-System**

PHP5.x
```bash
apt-get install php5-mysql
``` 

PHP7.x
```bash
apt-get install php7.0-mysql
```

Wenn der Paketname nicht gefunden werden kann, versuchen Sie es mit `apt-cache search php mysql`.

**Konnten Sie die Erweiterungen nicht installieren?**

Wenn keiner der oben genannten Methoden funktioniert, beziehen Sie sich bitte auf den [Workerman-Handbuch-Anhang-Installationsmethoden für Erweiterungen - Methode 3: Installation aus dem Quellcode](../appendices/install-extension.md).

## Installation von Workerman/MySQL
**Methode 1:**

Sie können das über Composer installieren, indem Sie den folgenden Befehl in der Befehlszeile ausführen (der Composer-Quellcode befindet sich im Ausland, daher kann der Installationsprozess sehr langsam sein).

```bash
composer require workerman/mysql
```

Nach erfolgreicher Ausführung des obigen Befehls wird das Verzeichnis vendor erstellt. Importieren Sie dann die autoload.php im Vendor-Verzeichnis in Ihr Projekt.

```php
require_once __DIR__ . '/vendor/autoload.php';
```

**Methode 2:**

[Laden Sie den Quellcode herunter](https://github.com/walkor/mysql/archive/master.zip), entpacken Sie das Verzeichnis in Ihr Projekt (beliebige Position) und fordern Sie die Quelldatei direkt an.

```php
require_once '/Ihr/Pfad/zum/mysql-master/src/Connection.php';
```

## Hinweis
Es wird dringend empfohlen, die Datenbankverbindung im onWorkerStart-Rückruf zu initialisieren, um zu vermeiden, dass die Verbindung vor dem Ausführen von `Worker::runAll();` initialisiert wird. Eine Verbindung, die vor dem Ausführen von `Worker::runAll();` initialisiert wurde, gehört zum Hauptprozess, und Unterprozesse erben diese Verbindung. Eine gemeinsame Verwendung der gleichen Datenbankverbindung durch Haupt- und Unterprozesse kann zu Fehlern führen.

## Beispiel
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    // Die db-Instanz in einer globalen Variablen speichern (kann auch in einem statischen Klassenmember gespeichert werden)
    global $db;
    $db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Holen Sie sich die db-Instanz über die globale Variable
    global $db;
    // Führen Sie SQL aus
    $all_tables = $db->query('show tables');
    $connection->send(json_encode($all_tables));
};
// Worker ausführen
Worker::runAll();
```

## Verwendung von MySQL/Connection
```php
// Initialisierung der db-Verbindung
$db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');

// Alle Daten abrufen
$db->select('ID,Sex')->from('Persons')->where('sex= :sex AND ID = :id')->bindValues(array('sex'=>'M', 'id' => 1))->query();
// equals
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' AND ID = 1")->query();
// equals
$db->query("SELECT ID,Sex FROM `Persons` WHERE sex='M' AND ID = 1");


// Eine Zeile Daten abrufen
$db->select('ID,Sex')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->row();
// equals
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' ")->row();
// equals
$db->row("SELECT ID,Sex FROM `Persons` WHERE sex='M'");


// Eine Spalte Daten abrufen
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->column();
// equals
$db->select('ID')->from('Persons')->where("sex= 'F' ")->column();
// equals
$db->column("SELECT `ID` FROM `Persons` WHERE sex='M'");

// Einzelwert abrufen
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->single();
// equals
$db->select('ID')->from('Persons')->where("sex= 'F' ")->single();
// equals
$db->single("SELECT ID FROM `Persons` WHERE sex='M'");

// Komplexe Abfrage
$db->select('*')->from('table1')->innerJoin('table2','table1.uid = table2.uid')->where('age > :age')->groupBy(array('aid'))->having('foo="foo"')->orderByASC/*orderByDESC*/(array('did'))
->limit(10)->offset(20)->bindValues(array('age' => 13));
// equals
$db->query('SELECT * FROM `table1` INNER JOIN `table2` ON `table1`.`uid` = `table2`.`uid`
WHERE age > 13 GROUP BY aid HAVING foo="foo" ORDER BY did LIMIT 10 OFFSET 20');

// Einfügen
$insert_id = $db->insert('Persons')->cols(array(
    'Firstname'=>'abc',
    'Lastname'=>'efg',
    'Sex'=>'M',
    'Age'=>13))->query();
equals
$insert_id = $db->query("INSERT INTO `Persons` ( `Firstname`,`Lastname`,`Sex`,`Age`)
VALUES ( 'abc', 'efg', 'M', 13)");

// Aktualisieren
$row_count = $db->update('Persons')->cols(array('sex'))->where('ID=1')
->bindValue('sex', 'F')->query();
// equals
$row_count = $db->update('Persons')->cols(array('sex'=>'F'))->where('ID=1')->query();
// equals
$row_count = $db->query("UPDATE `Persons` SET `sex` = 'F' WHERE ID=1");

// Löschen
$row_count = $db->delete('Persons')->where('ID=9')->query();
// equals
$row_count = $db->query("DELETE FROM `Persons` WHERE ID=9");

// Transaktion
$db->beginTrans();
....
$db->commitTrans(); // or $db->rollBackTrans();

```

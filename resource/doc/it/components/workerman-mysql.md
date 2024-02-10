# Workerman/MySQL

## Introduzione
Spesso i programmi residenti in memoria incontrano l'errore ```mysql gone away``` quando utilizzano MySQL. Questo è dovuto all'assenza di comunicazione per un lungo periodo tra il programma e MySQL, che porta alla disconnessione da parte del server MySQL. Questa classe per il database può risolvere questo problema, in quanto, se si verifica l'errore ```mysql gone away```, verrà eseguito automaticamente un secondo tentativo.

## Dipendenze dell'estensione
Questa classe MySQL dipende da due estensioni, [pdo](https://php.net/manual/zh/book.pdo.php) e [pdo_mysql](https://php.net/manual/zh/ref.pdo-mysql.php). Se mancano queste estensioni, si otterrà un errore ```Undefined class constant 'MYSQL_ATTR_INIT_COMMAND' in ....```.

Eseguendo il comando ```php -m``` dalla riga di comando, verranno elencate tutte le estensioni PHP CLI installate. Se mancano pdo o pdo_mysql, è necessario installarle manualmente.

**Sistema CentOS**

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
Se non si trovano i nomi dei pacchetti, è possibile provare a cercare con ```yum search php mysql```.

**Sistema Ubuntu/Debian**

PHP5.x
```bash
apt-get install php5-mysql
```

PHP7.x
```bash
apt-get install php7.0-mysql
```

Se non si trovano i nomi dei pacchetti, è possibile provare a cercare con ```apt-cache search php mysql```.

**Impossibile installare usando i metodi sopra menzionati?**

Se i metodi sopra citati non funzionano, si prega di fare riferimento al [manuale di Workerman - Appendici - Installazione dell'estensione - Metodo 3: Installazione da codice sorgente](../appendices/install-extension.md).

## Installazione di Workerman/MySQL
**Metodo 1:**

È possibile installare l'estensione tramite Composer eseguendo il seguente comando da riga di comando (in quanto il repository di Composer si trova all'estero, l'installazione potrebbe essere molto lenta).

```bash
composer require workerman/mysql
```

Dopo il successo del comando sopra, verrà creata la cartella "vendor" e sarà necessario inclusa nel proprio progetto il file "autoload.php" all'interno della cartella "vendor".

```php
require_once __DIR__ . '/vendor/autoload.php';
```

**Metodo 2:**

[Download del codice sorgente](https://github.com/walkor/mysql/archive/master.zip), decomprimere la cartella e posizionarla all'interno del proprio progetto in una posizione qualsiasi, quindi richiamare direttamente il file sorgente.

```php
require_once '/percorso/tuo/di/mysql-master/src/Connection.php';
```

## Nota
Si consiglia vivamente di inizializzare la connessione al database nel callback onWorkerStart, al fine di evitare l'inizializzazione della connessione prima dell'esecuzione di ```Worker::runAll();```. Se la connessione viene inizializzata prima dell'esecuzione di ```Worker::runAll();```, ciò verrà effettuato all'interno del processo principale, e i processi figlio erediteranno questa connessione. L'utilizzo della stessa connessione al database da parte del processo principale e di quello figlio potrebbe causare errori.

## Esempio
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    // Memorizzare l'istanza di db come variabile globale (oppure memorizzarla come membro statico di una classe)
    global $db;
    $db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'nome_db');
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Ottenere l'istanza di db tramite la variabile globale
    global $db;
    // Eseguire SQL
    $all_tables = $db->query('show tables');
    $connection->send(json_encode($all_tables));
};
// Esegui il worker
Worker::runAll();
```

## Utilizzo specifico di MySQL/Connection
```php
// Inizializza la connessione al database
$db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'nome_db');

// Ottieni tutti i dati
$db->select('ID,Sex')->from('Persons')->where('sex= :sex AND ID = :id')->bindValues(array('sex'=>'M', 'id' => 1))->query();
// Equivalente a
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' AND ID = 1")->query();
// Equivalente a
$db->query("SELECT ID,Sex FROM `Persons` WHERE sex='M' AND ID = 1");


// Ottenere una riga di dati
$db->select('ID,Sex')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->row();
// Equivalente a
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' ")->row();
// Equivalente a
$db->row("SELECT ID,Sex FROM `Persons` WHERE sex='M'");


// Ottieni una colonna di dati
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->column();
// Equivalente a
$db->select('ID')->from('Persons')->where("sex= 'F' ")->column();
// Equivalente a
$db->column("SELECT `ID` FROM `Persons` WHERE sex='M'");

// Ottieni un singolo valore
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->single();
// Equivalente a
$db->select('ID')->from('Persons')->where("sex= 'F' ")->single();
// Equivalente a
$db->single("SELECT ID FROM `Persons` WHERE sex='M'");

// Query complessa
$db->select('*')->from('table1')->innerJoin('table2','table1.uid = table2.uid')->where('age > :age')->groupBy(array('aid'))->having('foo="foo"')->orderByASC/*orderByDESC*/(array('did'))
->limit(10)->offset(20)->bindValues(array('age' => 13));
// Equivalente a
$db->query('SELECT * FROM `table1` INNER JOIN `table2` ON `table1`.`uid` = `table2`.`uid`
WHERE age > 13 GROUP BY aid HAVING foo="foo" ORDER BY did LIMIT 10 OFFSET 20');

// Inserimento
$insert_id = $db->insert('Persons')->cols(array(
    'Firstname'=>'abc',
    'Lastname'=>'efg',
    'Sex'=>'M',
    'Age'=>13))->query();
Equivalente a
$insert_id = $db->query("INSERT INTO `Persons` ( `Firstname`,`Lastname`,`Sex`,`Age`)
VALUES ( 'abc', 'efg', 'M', 13)");

// Aggiornamento
$row_count = $db->update('Persons')->cols(array('sex'))->where('ID=1')
->bindValue('sex', 'F')->query();
// Equivalente a
$row_count = $db->update('Persons')->cols(array('sex'=>'F'))->where('ID=1')->query();
// Equivalente a
$row_count = $db->query("UPDATE `Persons` SET `sex` = 'F' WHERE ID=1");

// Cancellazione
$row_count = $db->delete('Persons')->where('ID=9')->query();
// Equivalente a
$row_count = $db->query("DELETE FROM `Persons` WHERE ID=9");

// Transazioni
$db->beginTrans();
....
$db->commitTrans(); // oppure $db->rollBackTrans();

```

# Workerman/MySQL

## Description
Les programmes résidents en mémoire rencontrent souvent l'erreur ```mysql gone away``` lorsqu'ils utilisent MySQL, ce qui est dû au fait que la connexion entre le programme et MySQL n'est pas utilisée pendant une longue période, ce qui fait que le serveur MySQL met fin à la connexion. Cette classe de base de données peut résoudre ce problème en effectuant automatiquement une nouvelle tentative en cas d'erreur ```mysql gone away```.

## Dépendances requises
Cette classe MySQL dépend de deux extensions, [pdo](https://php.net/manual/zh/book.pdo.php) et [pdo_mysql](https://php.net/manual/zh/ref.pdo-mysql.php). Si ces extensions sont manquantes, une erreur ```Undefined class constant 'MYSQL_ATTR_INIT_COMMAND' in ....``` sera signalée.

Exécutez la commande suivante dans le terminal pour lister toutes les extensions PHP CLI installées : ```php -m```. Si pdo ou pdo_mysql ne sont pas présents, veuillez les installer vous-même.

**Système CentOS**
PHP5.x
``` bash
yum install php-pdo
yum install php-mysql
```
PHP7.x
``` bash
yum install php70w-pdo_dblib.x86_64
yum install php70w-mysqlnd.x86_64
```
Si le nom du package n'est pas trouvé, veuillez essayer de le rechercher avec la commande ```yum search php mysql```.

**Système Ubuntu/Debian**
PHP5.x
``` bash
apt-get install php5-mysql
```
PHP7.x
``` bash
apt-get install php7.0-mysql
```
Si le nom du package n'est pas trouvé, veuillez essayer de le rechercher avec la commande ```apt-cache search php mysql```.

**Impossible d'installer en utilisant les méthodes ci-dessus?**
Si les méthodes ci-dessus ne fonctionnent pas, veuillez consulter le [manuel de Workerman - Annexe- Installation des extensions - Méthode 3: Installation à partir du code source](../appendices/install-extension.md).

## Installation de Workerman/MySQL
**Méthode 1:**
Vous pouvez installer via Composer en exécutant la commande suivante dans le terminal (le source de Composer est à l'étranger, le processus d'installation peut être très lent) :
``` bash
composer require workerman/mysql
```
Après le succès de la commande ci-dessus, un répertoire vendor sera généré. Ensuite, incluez le autoload.php sous le répertoire vendor dans votre projet.
```php
require_once __DIR__ . '/vendor/autoload.php';
```

**Méthode 2:**
[Téléchargez le code source](https://github.com/walkor/mysql/archive/master.zip), extrayez le répertoire et placez-le dans votre propre projet (à n'importe quel emplacement) puis incluez directement le fichier source.

```php
require_once '/votre/chemin/vers/mysql-master/src/Connection.php';
```

## Remarque
Il est fortement recommandé d'initialiser la connexion de la base de données dans le callback onWorkerStart, afin d'éviter toute initialisation de connexion avant l'exécution de ```Worker::runAll();```. Une connexion initialisée avant l'exécution de ```Worker::runAll();``` appartiendra au processus principal, et les processus fils hériteront de cette connexion, ce qui entraînera des erreurs dus à la connexion partagée entre le processus principal et les processus fils.

## Exemple
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    // Stocker l'instance de base de données dans une variable globale (ou dans une propriété statique d'une classe)
    global $db;
    $db = new \Workerman\MySQL\Connection('host', 'port', 'utilisateur', 'mot_de_passe', 'nom_de_la_base');
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Obtenir l'instance de la base de données via une variable globale
    global $db;
    // Exécuter une requête SQL
    $all_tables = $db->query('show tables');
    $connection->send(json_encode($all_tables));
};
// Exécuter le worker
Worker::runAll();
```

## Utilisation spécifique de MySQL/Connection
```php
// Initialiser la connexion à la base de données
$db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');

// Obtenir toutes les données
$db->select('ID,Sex')->from('Persons')->where('sex= :sex AND ID = :id')->bindValues(array('sex'=>'M', 'id' => 1))->query();
//équivalent à
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' AND ID = 1")->query();
//équivalent à
$db->query("SELECT ID,Sex FROM `Persons` WHERE sex='M' AND ID = 1");

// Obtenir une ligne de données
$db->select('ID,Sex')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->row();
//équivalent à
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' ")->row();
//équivalent à
$db->row("SELECT ID,Sex FROM `Persons` WHERE sex='M'");

// Obtenir une colonne de données
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->column();
//équivalent à
$db->select('ID')->from('Persons')->where("sex= 'F' ")->column();
//équivalent à
$db->column("SELECT `ID` FROM `Persons` WHERE sex='M'");

// Obtenir une seule valeur
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->single();
//équivalent à
$db->select('ID')->from('Persons')->where("sex= 'F' ")->single();
//équivalent à
$db->single("SELECT ID FROM `Persons` WHERE sex='M'");

// Requête complexe
$db->select('*')->from('table1')->innerJoin('table2','table1.uid = table2.uid')->where('age > :age')->groupBy(array('aid'))->having('foo="foo"')->orderByASC/*orderByDESC*/(array('did'))
->limit(10)->offset(20)->bindValues(array('age' => 13));
// équivalent à
$db->query('SELECT * FROM `table1` INNER JOIN `table2` ON `table1`.`uid` = `table2`.`uid`
WHERE age > 13 GROUP BY aid HAVING foo="foo" ORDER BY did LIMIT 10 OFFSET 20');

// Insertion
$insert_id = $db->insert('Persons')->cols(array(
    'Firstname'=>'abc',
    'Lastname'=>'efg',
    'Sex'=>'M',
    'Age'=>13))->query();
équivalent à
$insert_id = $db->query("INSERT INTO `Persons` ( `Firstname`,`Lastname`,`Sex`,`Age`)
VALUES ( 'abc', 'efg', 'M', 13)");

// Mise à jour
$row_count = $db->update('Persons')->cols(array('sexe'))->where('ID=1')
->bindValue('sexe', 'F')->query();
// équivalent à
$row_count = $db->update('Persons')->cols(array('sex'=>'F'))->where('ID=1')->query();
// équivalent à
$row_count = $db->query("UPDATE `Persons` SET `sex` = 'F' WHERE ID=1");

// Suppression
$row_count = $db->delete('Persons')->where('ID=9')->query();
// équivalent à
$row_count = $db->query("DELETE FROM `Persons` WHERE ID=9");

// Transaction
$db->beginTrans();
....
$db->commitTrans(); // ou $db->rollBackTrans();

```

# Normes de développement

## Répertoire de l'application

Le répertoire de l'application peut être placé n'importe où.

## Fichier d'entrée

Comme les applications PHP sous nginx+PHP-FPM, les applications de Workerman nécessitent également un fichier d'entrée, et ce fichier d'entrée est exécuté en mode CLI PHP.

Le fichier d'entrée contient le code pour créer des processus d'écoute, par exemple le fragment de code de développement basé sur Worker suivant :

test.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Créer un Worker écoutant sur le port 2345, utilisant le protocole http
$http_worker = new Worker("http://0.0.0.0:2345");

// Démarrer 4 processus pour fournir des services
$http_worker->count = 4;

// Répondre "hello world" au navigateur lors de la réception de données envoyées par le navigateur
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Envoyer "hello world" au navigateur
    $connection->send('hello world');
};

Worker::runAll();
```

## Normes de code dans Workerman

1. Les noms de classe utilisent la convention de la première lettre en majuscule en camel case. Le nom du fichier de classe doit être le même que le nom de la classe à l'intérieur du fichier, pour permettre le chargement automatique. Par exemple :
```php
class UserInfo
{
...
```

2. Utilisation d'espaces de noms, le nom de l'espace de noms correspond au chemin du répertoire et est basé sur le répertoire racine du projet du développeur.

Par exemple, dans le projet MyApp/, le fichier de classe MyApp/MyClass.php se trouve dans le répertoire racine du projet, donc le namespace est omis. Le fichier de classe MyApp/Protocols/MyProtocol.php se trouve dans le répertoire Protocols du projet MyApp, donc il doit inclure le namespace ```namespace Protocols;```, comme suit :
```php
namespace Protocols;
class MyProtocol
{
....
```

3. Les noms de fonctions ordinaires et de variables sont en minuscules avec des traits de soulignement. Par exemple :
```php
$connection_list = array();
function get_connection_list()
{
....
```

4. Les membres de classe et les méthodes de classe utilisent la forme camel case avec la première lettre en minuscule. Par exemple :
```php
public $connectionList;
public function getConnectionList();
```

5. Les paramètres de fonctions et de méthodes de classe utilisent la forme minuscule avec des traits de soulignement. Par exemple :
```php
function get_connection_list($one_param, $tow_param)
{
....
```

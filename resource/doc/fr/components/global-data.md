# Composant de partage de variable GlobalData
**``` (Nécessite une version de Workerman >= 3.3.0) ```**

Adresse du code source : https://github.com/walkor/GlobalData

## Remarque
GlobalData nécessite une version de Workerman >= 3.3.0

## Installation

`composer require workerman/globaldata`

## Principe

Le composant GlobalData utilise les méthodes magiques de PHP ```__set __get __isset __unset``` pour déclencher la communication avec le serveur GlobalData, les variables réelles étant stockées sur le serveur GlobalData. Par exemple, lorsque vous définissez une propriété inexistante pour une classe cliente, la méthode magique ```__set``` est déclenchée, la classe cliente envoie une requête au serveur GlobalData dans la méthode ```__set``` pour stocker une variable. Lorsque vous accédez à une variable inexistante de la classe cliente, la méthode ```__get``` de la classe cliente envoie une requête au serveur GlobalData pour récupérer cette valeur, complétant ainsi le partage de variables entre les processus.


```php
require_once __DIR__ . '/vendor/autoload.php';

// Connexion au serveur Global Data
$global = new GlobalData\Client('127.0.0.1:2207');

// Déclenche $global->__isset('somedata') pour interroger le serveur sur la présence de la clé somedata
isset($global->somedata);

// Déclenche $global->__set('somedata',array(1,2,3)) pour notifier au serveur de stocker la valeur correspondante à somedata comme array(1,2,3)
$global->somedata = array(1,2,3);

// Déclenche $global->__get('somedata') pour interroger le serveur sur la valeur correspondante à somedata
var_export($global->somedata);

// Déclenche $global->__unset('somedata'), notifier au serveur de supprimer somedata et la valeur correspondante
unset($global->somedata);

```

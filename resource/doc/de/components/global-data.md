# GlobalData Variablen-Sharing-Komponente
**``` (Workerman-Version >= 3.3.0 erforderlich) ```**

Quellcode: https://github.com/walkor/GlobalData

## Hinweis
GlobalData erfordert Workerman-Version >= 3.3.0

## Installation

`composer require workerman/globaldata`

## Prinzip

Durch Verwendung der magischen PHP-Methoden ```__set __get __isset __unset``` wird die Kommunikation mit dem GlobalData-Server ausgelöst, wobei die tatsächlichen Variablen auf dem GlobalData-Server gespeichert werden. Zum Beispiel, wenn einer Client-Klasse eine nicht vorhandene Eigenschaft zugewiesen wird, wird die magische Methode ```__set``` ausgelöst, und die Client-Klasse sendet eine Anfrage an den GlobalData-Server, um eine Variable zu speichern. Wenn auf eine nicht existierende Variable der Client-Klasse zugegriffen wird, wird die Methode ```__get``` der Klasse ausgelöst, und der Client sendet eine Anfrage an den GlobalData-Server, um diesen Wert zu lesen, um eine gemeinsame Verarbeitung von Variablen zwischen Prozessen zu ermöglichen.


```php
require_once __DIR__ . '/vendor/autoload.php';

// Verbindung zum GlobalData-Server herstellen
$global = new GlobalData\Client('127.0.0.1:2207');

// Auslösen von $global->__isset('somedata'), um abzufragen, ob der Server einen Wert mit dem Schlüssel "somedata" gespeichert hat
isset($global->somedata);

// Auslösen von $global->__set('somedata',array(1,2,3)), um den Server darüber zu informieren, dass der Wert von "somedata" auf array(1,2,3) gesetzt werden soll
$global->somedata = array(1,2,3);

// Auslösen von $global->__get('somedata') zur Abfrage des Werts von "somedata" vom Server
var_export($global->somedata);

// Auslösen von $global->__unset('somedata'), um den Server darüber zu informieren, dass "somedata" und der entsprechende Wert gelöscht werden sollen
unset($global->somedata);

```

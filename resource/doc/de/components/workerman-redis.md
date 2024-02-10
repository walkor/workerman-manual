# workerman-redis

## Einführung

workeman/redis ist eine auf Workerman basierende asynchrone Redis-Komponente.

> **Hinweis**
> Das Hauptziel dieses Projekts ist die Implementierung von asynchronen Redis-Abonnements (subscribe, pSubscribe). Da Redis schnell genug ist, ist es nicht erforderlich, diesen asynchronen Client zu verwenden, es sei denn, Sie benötigen asynchrone Abonnements für psubscribe subscribe. In diesem Fall bietet die Verwendung der Redis-Erweiterung eine bessere Leistung.

## Installation:
```bash
composer require workerman/redis
```

## Callback-Verwendung

```php
use Workerman\Worker;
use Workerman\Redis\Client;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:6161');

$worker->onWorkerStart = function() {
    global $redis;
    $redis = new Client('redis://127.0.0.1:6379');
};

$worker->onMessage = function(TcpConnection $connection, $data) {
    global $redis;
    $redis->set('key', 'hello world');    
    $redis->get('key', function ($result) use ($connection) {
        $connection->send($result);
    });  
};

Worker::runAll();
```

## Verwendung von Co-Routinen

> **Hinweis**
> Die Verwendung von Co-Routinen erfordert workerman>=5.0, workerman/redis>=2.0.0 und die Installation von composer require revolt/event-loop ^1.0.0

```php
use Workerman\Worker;
use Workerman\Redis\Client;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:6161');

$worker->onWorkerStart = function() {
    global $redis;
    $redis = new Client('redis://127.0.0.1:6379');
};

$worker->onMessage = function(TcpConnection $connection, $data) {
    global $redis;
    $redis->set('key', 'hello world');    
    $result = $redis->get('key');
    $connection->send($result);
};

Worker::runAll();
```

Wenn keine Rückruffunktion festgelegt ist, gibt der Client das Ergebnis der asynchronen Anfrage auf synchrone Weise zurück, ohne den aktuellen Prozess zu blockieren, sodass Anfragen parallel verarbeitet werden können.

> **Hinweis**
> pSubscribe subscribe unterstützt keine Verwendung von Co-Routinen

# Dokumentation
**Hinweis**

**Im Callback-Modus haben die Rückruffunktionen in der Regel 2 Parameter ($result, $redis), wobei `$result` das Ergebnis und `$redis` die Redis-Instanz ist. Zum Beispiel:**
```php
use Workerman\Redis\Client;
$redis = new Client('redis://127.0.0.1:6379');
// Festlegen einer Rückruffunktion zur Überprüfung des Set-Aufrufergebnisses
$redis->set('key', 'value', function ($result, $redis) {
    var_dump($result); // true
});
// Rückruffunktionen sind alle optional, hier wird beispielsweise die Rückruffunktion weggelassen
$redis->set('key1', 'value1');
// Rückruffunktionen können verschachtelt werden
$redis->get('key', function ($result, $redis){
    $redis->set('key2', 'value2', function ($result) {
        var_dump($result);
    });
});
```

## **Verbindung**
```php
use Workerman\Redis\Client;
// Rückruffunktion weggelassen
$redis = new Client('redis://127.0.0.1:6379');
// Mit Rückruffunktion
$redis = new Client('redis://127.0.0.1:6379', [
    'connect_timeout' => 10 // Verbindungstimeout auf 10 Sekunden setzen, Standardwert ist 5 Sekunden, wenn nicht festgelegt
], function ($success, $redis) {
    // Rückruf des Verbindungsergebnisses
    if (!$success) echo $redis->error();
});
```

## **auth**
```php
// Passwortüberprüfung
$redis->auth('password', function ($result) {
    
});
// Überprüfung von Benutzername und Passwort
$redis->auth('username', 'password', function ($result) {

});
```

## **pSubscribe**

Abonnieren eines oder mehrere Kanäle, die einem bestimmten Muster entsprechen.

Jedes Muster verwendet \* als Platzhalter, zum Beispiel it\* passt zu allen Kanälen, die mit it beginnen (it.news, it.blog, it.tweets usw.). news.\* passt zu allen Kanälen, die mit news. beginnen (news.it, news.global.today usw.) und so weiter.

Hinweis: Die Rückruffunktion von pSubscribe hat 4 Parameter ($pattern, $channel, $message, $redis).

Nachdem eine Instanz von $redis die pSubscribe- oder subscribe-Methode aufgerufen hat, werden weitere Methodenaufrufe dieser Instanz ignoriert.

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');
$redis->psubscribe(['news*', 'blog*'], function ($pattern, $channel, $message) {
    echo "$pattern, $channel, $message"; // news*, news.add, news content
});

Timer::add(5, function () use ($redis2){
    $redis2->publish('news.add', 'news content');
});
```

## **subscribe**

Zur Abonnierung von Informationen für einen oder mehrere bestimmte Kanäle.

Hinweis: Die Rückruffunktion von subscribe hat 3 Parameter ($channel, $message, $redis).

Nachdem eine Instanz von $redis die pSubscribe- oder subscribe-Methode aufgerufen hat, werden weitere Methodenaufrufe dieser Instanz ignoriert.

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');
$redis->subscribe(['news', 'blog'], function ($channel, $message) {
    echo "$channel, $message"; // news, news content
});

Timer::add(5, function () use ($redis2){
    $redis2->publish('news', 'news content');
});
```

## **publish**

Zum Senden von Informationen an einen bestimmten Kanal.

Gibt die Anzahl der Abonnenten zurück, die die Information erhalten haben.

```php
$redis2->publish('news', 'news content');
```


## **select**
```php
// Rückruffunktion weggelassen
$redis->select(2);
$redis->select('test', function ($result, $redis) {
    // Der select-Parameter muss numerisch sein, daher ist $result hier false
    var_dump($result, $redis->error());
});
```

## **get**

Befehl zum Abrufen des Werts des angegebenen Schlüssels. Wenn der Schlüssel nicht vorhanden ist, wird NULL zurückgegeben. Wenn der Schlüsselwert nicht vom Typ String ist, wird false zurückgegeben.

```php
$redis->get('key', function($result) {
     // Wenn der Schlüssel nicht vorhanden ist, wird NULL zurückgegeben, sonst wird false zurückgegeben
    var_dump($result);
});
```

## **set**

Zum Setzen des Werts des angegebenen Schlüssels. Wenn der Schlüssel bereits einen anderen Wert enthält, überschreibt SET den alten Wert und ignoriert den Typ.

```php
$redis->set('key', 'value');
$redis->set('key', 'value', function($result){});
// Das dritte Argument kann die Verfallszeit übergeben, die nach 10 Sekunden abläuft
$redis->set('key','value', 10);
$redis->set('key','value', 10, function($result){});
```

## **setEx, pSetEx**

Zum Setzen des Werts des angegebenen Schlüssels und seiner Ablaufzeit. Wenn der Schlüssel bereits vorhanden ist, ersetzt der SETEX-Befehl den alten Wert.

```php
// Beachten Sie, dass das zweite Argument die Ablaufzeit in Sekunden ist
$redis->setEx('key', 3600, 'value'); 
// pSetEx Einheit ist Millisekunden
$redis->pSetEx('key', 3600, 'value'); 
```

## **del**

Zum Löschen eines vorhandenen Schlüssels, der Rückgabewert ist die Anzahl der gelöschten Schlüssel (nicht existierende Schlüssel werden nicht gezählt).

```php
// Löschen eines Schlüssels
$redis->del('key');
// Löschen mehrerer Schlüssel
$redis->del(['key', 'key1', 'key2']);
```

## **setNx**

(**SET**if**N**ot e**X**ists) Befehl setzt den angegebenen Wert für den Schlüssel, wenn der Schlüssel nicht vorhanden ist.

```php
$redis->del('key');
$redis->setNx('key', 'value', function($result){
    var_dump($result); // 1
});
$redis->setNx('key', 'value', function($result){
    var_dump($result); // 0
});
```

## **exists**

Befehl wird verwendet, um zu überprüfen, ob der angegebene Schlüssel existiert. Der Rückgabewert ist die Anzahl vorhandener Schlüssel.

```php
$redis->set('key', 'value');
$redis->exists('key', function ($result) {
    var_dump($result); // 1
}); 
$redis->exists('NonExistingKey', function ($result) {
    var_dump($result); // 0
}); 

$redis->mset(['foo' => 'foo', 'bar' => 'bar', 'baz' => 'baz']);
$redis->exists(['foo', 'bar', 'baz'], function ($result) {
    var_dump($result); // 3
}); 
```

## **incr, incrBy**

Erhöht den im Schlüssel gespeicherten numerischen Wert um eins/den angegebenen Wert. Wenn der Schlüssel nicht vorhanden ist, wird zunächst der Wert des Schlüssels auf 0 initialisiert und dann die incr/incrBy-Operation durchgeführt.

Wenn der Wert einen falschen Typ enthält oder der Wert des Strings nicht in eine Zahl umgewandelt werden kann, wird false zurückgegeben. Bei Erfolg wird der erhöhte Wert zurückgegeben.

```php
$redis->incr('key1', function ($result) {
    var_dump($result);
}); 
$redis->incrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **incrByFloat**

Fügt dem im Schlüssel gespeicherten Wert den angegebenen Gleitkommazuwachs hinzu. Wenn der Schlüssel nicht vorhanden ist, wird der Wert des Schlüssels zuerst auf 0 gesetzt und dann die Additionsoperation durchgeführt.
Wenn der Wert einen falschen Typ enthält oder der Wert des Strings nicht in eine Zahl umgewandelt werden kann, wird false zurückgegeben. Bei Erfolg wird der erhöhte Wert zurückgegeben.

```php
$redis->incrByFloat('key1', 1.5, function ($result) {
    var_dump($result);
}); 
```

## **decr, decrBy**

Der Befehl verringert den im Schlüssel gespeicherten Wert um eins/den angegebenen Wert. Wenn der Schlüssel nicht vorhanden ist, wird zunächst der Wert des Schlüssels auf 0 initialisiert und dann die decr/decrBy-Operation durchgeführt.

Wenn der Wert einen falschen Typ enthält oder der Wert des Strings nicht in eine Zahl umgewandelt werden kann, wird false zurückgegeben. Bei Erfolg wird der verringerte Wert zurückgegeben.

```php
$redis->decr('key1', function ($result) {
    var_dump($result);
}); 
$redis->decrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **mGet**

Gibt alle für den angegebenen oder die angegebenen Schlüssel gespeicherten Werte zurück. Wenn ein Schlüssel nicht existiert, wird für diesen Schlüssel NULL zurückgegeben.

```php
$redis->set('key1', 'value1');
$redis->set('key2', 'value2');
$redis->set('key3', 'value3');
$redis->mGet(['key0', 'key1', 'key5'], function ($result) {
    var_dump($result); // [null, 'value1', null];
}); 
```
```php
$redis->set('x', '42');
$redis->getSet('x', 'lol', function ($result) {
    var_dump($result); // '42'
});
$redis->get('x', function ($result) {
    var_dump($result); // 'lol'
});
```

```php
$redis->randomKey(function($key) use ($redis) {
    $redis->get($key, function ($result) {
        var_dump($result); 
    });
});
```

```php
$redis->select(0);	// switch to DB 0
$redis->set('x', '42');	// write 42 to x
$redis->move('x', 1, function ($result) { 	// move to DB 1
    var_dump($result); // 1
});
$redis->select(1);	// switch to DB 1
$redis->get('x', function ($result) {
    var_dump($result); // '42'
});
```

```php
$redis->set('x', '42');
$redis->rename('x', 'y', function ($result) {
    var_dump($result); // true
});
```

```php
$redis->del('y');
$redis->set('x', '42');
$redis->renameNx('x', 'y', function ($result) {
    var_dump($result); // 1
});
```

```php
$redis->set('x', '42');
$redis->expire('x', 3);
```

```php
$redis->keys('*', function ($keys) {
    var_dump($keys);
});
$redis->keys('user*', function ($keys) {
    var_dump($keys);
});
```

```php
$redis->type('key', function ($result) {
    var_dump($result); // string set list zset hash none
});
```

```php
$redis->set('key', 'value1');
$redis->append('key', 'value2', function ($result) {
    var_dump($result); // 12
});
$redis->get('key', function ($result) {
    var_dump($result); // 'value1value2'
});
```

```php
$redis->set('key', 'string value');
$redis->getRange('key', 0, 5, function ($result) {
    var_dump($result); // 'string'
});
$redis->getRange('key', -5, -1 , function ($result) {
    var_dump($result); // 'value'
});
```

```php
$redis->set('key', 'Hello world');
$redis->setRange('key', 6, "redis", function ($result) {
    var_dump($result); // 11
});
$redis->get('key', function ($result) {
    var_dump($result); // 'Hello redis'
});
```

```php
$redis->set('key', 'value');
$redis->strlen('key', function ($result) {
    var_dump($result); // 5
});
```

```php
$redis->set('key', "\x7f");
$redis->getBit('key', 0, function ($result) {
    var_dump($result); // 0
});
```

```php
$redis->set('key', "*");	// ord("*") = 42 = 0x2f = "0010 1010"
$redis->setBit('key', 5, 1, function ($result) {
    var_dump($result); // 0
});
```

```php
$redis->set('key1', "abc");
$redis->bitOp( 'AND', 'dst', 'key1', 'key2', function ($result) {
    var_dump($result); // 3
});
```

```php
$redis->set('key', 'hello');
$redis->bitCount( 'key', 0, 0, function ($result) {
    var_dump($result); // 3
});
$redis->bitCount( 'key', function ($result) {
    var_dump($result); //21
});
```

```php
$redis->del('s');
$redis->sAdd('s', 5);
$redis->sAdd('s', 4);
$redis->sAdd('s', 2);
$redis->sAdd('s', 1);
$redis->sAdd('s', 3);
$redis->sort('s', [], function ($result) {
    var_dump($result); // 1,2,3,4,5
});
$redis->sort('s', ['sort' => 'desc'], function ($result) {
    var_dump($result); // 5,4,3,2,1
});
$redis->sort('s', ['sort' => 'desc', 'store' => 'out'], function ($result) {
    var_dump($result); // (int)5
});
```


```php
$redis->set('key', 'value', 10);
// 秒为单位
$redis->ttl('key', function ($result) {
    var_dump($result); // 10
});
// 毫秒为单位
$redis->pttl('key', function ($result) {
    var_dump($result); // 9999
});
// key 不存在
$redis->pttl('key-not-exists', function ($result) {
    var_dump($result); // -2
});
```


```php
$redis->persist('key');
```

```php
$redis->mSet(['key0' => 'value0', 'key1' => 'value1']);
```

```php
$redis->del('h');
$redis->hSet('h', 'key1', 'hello', function ($r) {
    var_dump($r); // 1
});
$redis->hGet('h', 'key1', function ($r) {
    var_dump($r); // hello
});
$redis->hSet('h', 'key1', 'plop', function ($r) {
    var_dump($r); // 0
});
$redis->hGet('h', 'key1', function ($r) {
    var_dump($r); // plop
});
```

```php
$redis->del('h');
$redis->hSetNx('h', 'key1', 'hello', function ($r) {
    var_dump($r); // 1
});
$redis->hSetNx('h', 'key1', 'world', function ($r) {
    var_dump($r); // 0
});
```

```php
$redis->hGet('h', 'key1', function ($result) {
    var_dump($result);
});
```
## **hLen**

Wird verwendet, um die Anzahl der Felder in einem Hash-Table abzurufen.

Wenn der Schlüssel nicht existiert, wird 0 zurückgegeben.

```php
$redis->del('h');
$redis->hSet('h', 'key1', 'hallo');
$redis->hSet('h', 'key2', 'plop');
$redis->hLen('h', function ($result) {
    var_dump($result); // 2
});
```

## **hDel**

Der Befehl wird verwendet, um eines oder mehrere angegebene Felder in der Hash-Tabelle "key" zu löschen. Nicht vorhandene Felder werden ignoriert.

Gibt die Anzahl der erfolgreich gelöschten Felder zurück, ohne die ignorierten Felder zu zählen. Wenn der Schlüssel kein Hash ist, wird false zurückgegeben.

```php
$redis->hDel('h', 'key1');
```

## **hKeys**

Ruft alle Bereiche in der Hash-Tabelle als Array ab.

Wenn der Schlüssel nicht existiert, wird ein leeres Array zurückgegeben. Wenn der Schlüssel keine Hash-Tabelle ist, wird false zurückgegeben.

```php
$redis->hKeys('key', function ($result) {
    var_dump($result);
});
```

## **hVals**

Gibt alle Werte der Felder in der Hash-Tabelle als Array zurück.

Wenn der Schlüssel nicht existiert, wird ein leeres Array zurückgegeben. Wenn der Schlüssel keine Hash-Tabelle ist, wird false zurückgegeben.

```php
$redis->hVals('key', function ($result) {
    var_dump($result);
});
```

## **hGetAll**

Gibt alle Felder und Werte in der Hash-Tabelle als assoziatives Array zurück.

Wenn der Schlüssel nicht existiert, wird ein leeres Array zurückgegeben. Wenn der Schlüssel kein Hash-Typ ist, wird false zurückgegeben.

```php
$redis->del('h');
$redis->hSet('h', 'a', 'x');
$redis->hSet('h', 'b', 'y');
$redis->hSet('h', 'c', 'z');
$redis->hSet('h', 'd', 't');
$redis->hGetAll('h', function ($result) {
    var_export($result); 
});
```
Rückgabe
```php
array (
    'a' => 'x',
    'b' => 'y',
    'c' => 'z',
    'd' => 't',
)
```

## **hExists**

Überprüft, ob das angegebene Feld in der Hash-Tabelle existiert. Wenn das Feld existiert, gibt es 1 zurück, wenn das Feld oder der Schlüssel nicht existiert, gibt es 0 zurück. Im Falle eines Fehlers wird false zurückgegeben.

```php
$redis->hExists('h', 'a', function ($result) {
    var_dump($result); //
});
```

## **hIncrBy**

Wird verwendet, um den Wert des Feldes in der Hash-Tabelle um den angegebenen Inkrementwert zu erhöhen. Das Inkrement kann auch negativ sein und entspricht einer Verringerung des Feldwertes.

Wenn der Schlüssel der Hash-Tabelle nicht existiert, wird eine neue Hash-Tabelle erstellt und der Befehl HINCRBY ausgeführt.

Wenn das angegebene Feld nicht existiert, wird der Wert des Feldes vor der Befehlsausführung auf 0 initialisiert.

Ein HINCRBY-Befehl, der auf ein Feld angewendet wird, das einen gespeicherten Zeichenfolgenwert enthält, gibt false zurück.

Der Wert dieser Operation ist auf 64-Bit-Vorzeichen-Integer beschränkt.

```php
$redis->del('h');
$redis->hIncrBy('h', 'x', 2,  function ($result) {
    var_dump($result);
});
```

## **hIncrByFloat**

Ähnlich wie hIncrBy, aber das Inkrement ist ein Fließkommawert.

## **hMSet**

Setzt mehrere Feld-Wert-Paare gleichzeitig in die Hash-Tabelle.

Dieser Befehl überschreibt vorhandene Felder in der Hash-Tabelle. Wenn die Hash-Tabelle nicht existiert, wird eine leere Hash-Tabelle erstellt und der HMSET-Befehl ausgeführt.

```php
$redis->del('h');
$redis->hMSet('h', ['name' => 'Joe', 'sex' => 1])
```

## **hMGet**

Gibt als assoziatives Array die Werte der angegebenen Felder in der Hash-Tabelle zurück.

Wenn das angegebene Feld nicht in der Hash-Tabelle existiert, ist der entsprechende Wert null. Wenn der Schlüssel kein Hash ist, wird false zurückgegeben.

```php
$redis->del('h');
$redis->hSet('h', 'field1', 'value1');
$redis->hSet('h', 'field2', 'value2');
$redis->hMGet('h', ['field1', 'field2', 'field3'], function ($r) {
    var_export($r);
});
```
Ausgabe
```php
array (
 'field1' => 'value1',
 'field2' => 'value2',
 'field3' => null
)
```

## **blPop, brPop**

Entfernt und ruft das erste/letzte Element der Liste ab. Wenn die Liste leer ist, blockiert sie, bis ein Element zum Abholen verfügbar ist oder bis ein Timeout erreicht wird.

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');

$redis->blPop(['key1', 'key2'], 10, function ($r) {
    var_export($r); // array ( 0 => 'key1',1 => 'a')
});

Timer::add(1, function () use ($redis2) {
    $redis2->lpush('key1', 'a');
});
```

## **bRPopLPush**

Entfernt das letzte Element aus einer Liste und fügt es an den Anfang einer anderen Liste an. Wenn die Liste leer ist, blockiert sie, bis ein Element zum Abholen verfügbar ist oder bis ein Timeout erreicht wird. Wenn ein Timeout auftritt, wird null zurückgegeben.

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');
$redis->del(['key1', 'key2']);
$redis->bRPopLPush('key1', 'key2', 2, function ($r) {
    var_export($r);
});
Timer::add(2, function () use ($redis2) {
    $redis2->lpush('key1', 'a');
    $redis2->lRange('key2', 0, -1, function ($r) {
        var_dump($r);
    });
}, null, false);
```

## **lIndex**

Ruft das Element in der Liste anhand des Index ab. Negative Indizes sind ebenfalls möglich: -1 bezieht sich auf das letzte Element, -2 auf das vorletzte Element usw.

Wenn der angegebene Index nicht im Bereich der Liste liegt, wird null zurückgegeben. Wenn der Schlüssel keine Liste ist, wird false zurückgegeben.

```php
$redis->del('key1']);
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lindex('key1', 0, function ($r) {
    var_dump($r); // A
});
```

## **lInsert**

Fügt ein Element vor oder nach einem bestimmten Element in die Liste ein. Wenn das bestimmte Element nicht in der Liste existiert, wird nichts eingefügt.

Wenn die Liste nicht existiert, wird sie als leere Liste betrachtet. Wenn der Schlüssel keine Liste ist, wird false zurückgegeben.

```php
$redis->del('key1');
$redis->lInsert('key1', 'after', 'A', 'X', function ($r) {
    var_dump($r); // 0
});
$redis->lPush('key1', 'A');
$redis->lPush('key1', 'B');
$redis->lPush('key1', 'C');
$redis->lInsert('key1', 'before', 'C', 'X', function ($r) {
    var_dump($r); // 4
});
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['A', 'B', 'X', 'C']
});
```

## **lPop**

Entfernt das erste Element der Liste und gibt es zurück.

Wenn die Liste "key" nicht existiert, wird null zurückgegeben.

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lPop('key1', function ($r) {
    var_dump($r); // A
});
```

## **lPush**

Fügt ein oder mehrere Elemente am Anfang der Liste hinzu. Wenn der Schlüssel nicht existiert, wird eine leere Liste erstellt und das LPUSH ausgeführt. Wenn der Schlüssel existiert, aber keine Liste ist, wird false zurückgegeben.

**Hinweis:** Vor Redis 2.4 akzeptierte der LPUSH-Befehl nur einen einzelnen Wert.

```php
$redis->del('key1');
$redis->lPush('key1', 'A');
$redis->lPush('key1', ['B','C']);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lPushx**

Fügt ein Element am Anfang einer vorhandenen Liste hinzu. Die Operation ist ungültig, wenn die Liste nicht existiert, und gibt 0 zurück. Wenn der Schlüssel keine Liste ist, wird false zurückgegeben.

Die Rückgabe ist die Länge der Liste nach dem lPushx-Befehl.

```php
$redis->del('key1');
    $redis->lPush('key1', 'A');
$redis->lPushx('key1', ['B','C'], function ($r) {
    var_dump($r); // 3
});
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lRange**

Gibt die Elemente in einem angegebenen Bereich der Liste zurück. Der Bereich wird durch die Index-Offsets START und END angegeben. 0 bezeichnet das erste Element der Liste, 1 das zweite Element usw. Negative Indizes sind ebenfalls möglich, -1 bezeichnet das letzte Element, -2 das vorletzte usw.

Gibt ein Array der angegebenen Elemente im Bereich zurück. Wenn der Schlüssel keine Liste ist, wird false zurückgegeben.

```php
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lRem**

Entfernt Elemente aus der Liste, die den angegebenen Wert entsprechen, abhängig von der COUNT-Option.

Die COUNT-Option kann folgende Werte haben:

*   count > 0: Sucht vom Anfang der Liste bis zum Ende; entfernt die Anzahl der COUNT übereinstimmenden Elemente.
*   count < 0: Sucht vom Ende der Liste bis zum Anfang; entfernt die Anzahl des Absolutwerts von COUNT übereinstimmenden Elemente.
*   count = 0: Entfernt alle übereinstimmenden Werte in der Liste.

Gibt die Anzahl der entfernten Elemente zurück. Wenn die Liste nicht existiert, wird 0 zurückgegeben. Wenn die Liste kein Liste-Typ ist, wird false zurückgegeben.

```php
$redis->lRem('key1', 2, 'A', function ($r) {
    var_dump($r); 
});
```

## **lSet**

Setzt das Element anhand des Index in der Liste.

Gibt true bei Erfolg zurück. Bei einem Index, der außerhalb des Bereichs liegt, oder bei einer leeren Liste wird false zurückgegeben.

```php
$redis->lSet('key1', 0, 'X');
```
## **lTrim**

lTrim wird verwendet, um eine Liste zu beschneiden, d.h. die Liste enthält nur die Elemente innerhalb des angegebenen Bereichs, und Elemente außerhalb des angegebenen Bereichs werden gelöscht.

Der Index 0 steht für das erste Element der Liste, 1 steht für das zweite Element und so weiter. Sie können auch negative Indizes verwenden, wobei -1 das letzte Element der Liste darstellt, -2 das zweitletzte Element und so weiter.

Bei Erfolg wird true zurückgegeben, bei Misserfolg wird false zurückgegeben.

```php
$redis->del('key1');

$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['A', 'B', 'C']
});
$redis->lTrim('key1', 0, 1);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['A', 'B']
});
```

## **rPop**

Wird verwendet, um das letzte Element einer Liste zu entfernen und den Wert des entfernten Elements zurückzugeben.

Wenn die Liste nicht existiert, wird null zurückgegeben.

```php
$redis->rPop('key1', function ($r) {
    var_dump($r);
});
```

## **rPopLPush**

Wird verwendet, um das letzte Element einer Liste zu entfernen und das Element zu einer anderen Liste hinzuzufügen.

```php
$redis->del('x', 'y');
$redis->lPush('x', 'abc');
$redis->lPush('x', 'def');
$redis->lPush('y', '123');
$redis->lPush('y', '456');
$redis->rPopLPush('x', 'y', function ($r) {
    var_dump($r); // abc
});
$redis->lRange('x', 0, -1, function ($r) {
    var_dump($r); // ['def']
});
$redis->lRange('y', 0, -1, function ($r) {
    var_dump($r); // ['abc', '456', '123']
});
```

## **rPush**

Fügt einen oder mehrere Werte am Ende einer Liste ein und gibt die Länge der Liste nach dem Einfügen zurück.

Wenn die Liste nicht existiert, wird eine leere Liste erstellt und die RPUSH-Operation durchgeführt. Wenn die Liste existiert, aber keine Liste ist, wird false zurückgegeben.

**Hinweis:** In Redis Version 2.4 und davor akzeptierte der RPUSH-Befehl nur einen einzelnen Wert.

```php
$redis->del('key1');
$redis->rPush('key1', 'A', function ($r) {
    var_dump($r); // 1
});
```

## **rPushX**

Fügt einen Wert am Ende einer vorhandenen Liste ein und gibt die Länge der Liste zurück. Wenn die Liste nicht existiert, ist die Operation ungültig und gibt 0 zurück. Wenn die Liste existiert, aber keine Liste ist, wird false zurückgegeben.

```php
$redis->del('key1');
$redis->rPushX('key1', 'A', function ($r) {
    var_dump($r); // 0
});
```

## **lLen**

Gibt die Länge der Liste zurück. Wenn die Liste "key" nicht existiert, wird sie als leere Liste interpretiert und 0 zurückgegeben. Wenn "key" kein Listen-Typ ist, wird false zurückgegeben.

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lLen('key1', function ($r) {
    var_dump($r); // 3
});
```

## **sAdd**

Fügt ein oder mehrere Mitgliedselemente zu einer Menge hinzu. Bereits vorhandene Elemente werden ignoriert.

Wenn die Menge "key" nicht existiert, wird eine Menge erstellt, die nur die hinzugefügten Elemente als Mitglieder enthält. Wenn "key" kein Set-Typ ist, wird false zurückgegeben.

**Hinweis:** In Redis Version 2.4 und davor akzeptierte SADD nur ein einzelnes Mitglied.

```php
$redis->del('key1');
$redis->sAdd('key1' , 'member1');
$redis->sAdd('key1' , ['member2', 'member3'], function ($r) {
    var_dump($r); // 2
});
$redis->sAdd('key1' , 'member2', function ($r) {
    var_dump($r); // 0
});
```

## **sCard**

Gibt die Anzahl der Elemente in der Menge zurück. Wenn die Menge "key" nicht existiert, wird 0 zurückgegeben.

```php
$redis->del('key1');
$redis->sAdd('key1' , 'member1');
$redis->sAdd('key1' , 'member2');
$redis->sAdd('key1' , 'member3');
$redis->sCard('key1', function ($r) {
    var_dump($r); // 3
});
$redis->sCard('keyX', function ($r) {
    var_dump($r); // 0
});
```

## **sDiff**

Gibt die Differenz zwischen der ersten Menge und den anderen Mengen zurück, d.h. die Elemente, die nur in der ersten Menge vorkommen. Nicht vorhandene Set-Keys werden als leere Menge betrachtet.

```php
$redis->del('s0', 's1', 's2');

$redis->sAdd('s0', '1');
$redis->sAdd('s0', '2');
$redis->sAdd('s0', '3');
$redis->sAdd('s0', '4');
$redis->sAdd('s1', '1');
$redis->sAdd('s2', '3');
$redis->sDiff(['s0', 's1', 's2'], function ($r) {
    var_dump($r); // ['2', '4']
});
```

## **sDiffStore**

Speichert die Differenz zwischen den gegebenen Sets in einem angegebenen Set. Wenn das angegebene Set-Key bereits existiert, wird es überschrieben.

```php
$redis->del('s0', 's1', 's2');
$redis->sAdd('s0', '1');
$redis->sAdd('s0', '2');
$redis->sAdd('s0', '3');
$redis->sAdd('s0', '4');
$redis->sAdd('s1', '1');
$redis->sAdd('s2', '3');
$redis->sDiffStore('dst', ['s0', 's1', 's2'], function ($r) {
    var_dump($r); // 2
});
$redis->sMembers('dst', function ($r) {
    var_dump($r); // ['2', '4']
});
```

## **sInter**

Gibt die Schnittmenge aller gegebenen Sets zurück. Nicht vorhandene Set-Keys werden als leere Menge betrachtet. Wenn eines der gegebenen Sets leer ist, ist das Ergebnis ebenfalls leer.

```php
$redis->del('s0', 's1', 's2');
$redis->sAdd('key1', 'val1');
$redis->sAdd('key1', 'val2');
$redis->sAdd('key1', 'val3');
$redis->sAdd('key1', 'val4');
$redis->sAdd('key2', 'val3');
$redis->sAdd('key2', 'val4');
$redis->sAdd('key3', 'val3');
$redis->sAdd('key3', 'val4');
$redis->sInter(['key1', 'key2', 'key3'], function ($r) {
    var_dump($r); // ['val4', 'val3']
});
```

## **sInterStore**

Speichert die Schnittmenge der gegebenen Sets in einem angegebenen Set und gibt die Anzahl der Elemente im gespeicherten Schnittmengen-Set zurück. Wenn das angegebene Set bereits existiert, wird es überschrieben.

```php
$redis->sAdd('key1', 'val1');
$redis->sAdd('key1', 'val2');
$redis->sAdd('key1', 'val3');
$redis->sAdd('key1', 'val4');

$redis->sAdd('key2', 'val3');
$redis->sAdd('key2', 'val4');

$redis->sAdd('key3', 'val3');
$redis->sAdd('key3', 'val4');

$redis->sInterStore('output', 'key1', 'key2', 'key3', function ($r) {
    var_dump($r); // 2
});
$redis->sMembers('output', function ($r) {
    var_dump($r); // ['val4', 'val3']
});
```

## **sIsMember**

Überprüft, ob das Mitgliedselement ein Mitglied der Menge ist.

Wenn das Mitgliedselement ein Mitglied der Menge ist, wird 1 zurückgegeben. Wenn das Mitgliedselement kein Mitglied der Menge ist oder wenn der Schlüssel nicht existiert, wird 0 zurückgegeben. Wenn der Key nicht den Typ Set hat, wird false zurückgegeben.

```php
$redis->sIsMember('key1', 'member1', function ($r) {
    var_dump($r); 
});
```

## **sMembers**

Gibt alle Mitglieder der Menge zurück. Wenn der Schlüssel nicht existiert, wird eine leere Menge zurückgegeben.

```php
$redis->sMembers('s', function ($r) {
    var_dump($r); 
});
```

## **sMove**

Bewegt das angegebene Mitglieds-Element von der Quellmenge zur Zielliste.

SMOVE ist eine atomare Operation. Wenn die Quellmenge nicht existiert oder das angegebene Mitglieds-Element nicht enthält, wird die SMOVE-Operation nicht ausgeführt und es wird nur 0 zurückgegeben. Andernfalls wird das Mitglieds-Element aus der Quellmenge entfernt und der Zielliste hinzugefügt.

Wenn Quelle oder Ziel nicht vom Typ Set sind, wird false zurückgegeben.

```php
$redis->sMove('key1', 'key2', 'member13');
```

## **sPop**

Entfernt ein oder mehrere zufällige Elemente aus der Menge und gibt die entfernten Elemente zurück.

Wenn die Menge nicht existiert oder leer ist, wird null zurückgegeben.

```php
$redis->del('key1');
$redis->sAdd('key1' , 'member1');
$redis->sAdd('key1' , 'member2');
$redis->sAdd('key1' , 'member3');
$redis->sPop('key1', function ($r) {
    var_dump($r); // member3
});
$redis->sAdd('key2', ['member1', 'member2', 'member3']);
$redis->sPop('key2', 3, function ($r) {
    var_dump($r); // ['member1', 'member2', 'member3']
});
```

## **sRandMember**

Das Srandmember-Befehl in Redis wird verwendet, um ein zufälliges Element aus der Menge zurückzugeben.

Ab der Redis-Version 2.6 akzeptiert der Srandmember-Befehl einen optionalen Parameter "count":

* Wenn count eine positive Zahl ist und kleiner als die Kardinalität der Menge ist, gibt der Befehl ein Array mit count unterschiedlichen Elementen zurück. Wenn count größer oder gleich der Kardinalität der Menge ist, wird die gesamte Menge zurückgegeben.
* Wenn count eine negative Zahl ist, gibt der Befehl ein Array zurück, bei dem die Elemente möglicherweise mehrfach vorkommen und die Länge des Arrays der absoluten Anzahl von count entspricht.

Diese Operation ähnelt SPOP, aber SPOP entfernt ein zufälliges Element aus der Menge und gibt es zurück, während Srandmember nur ein zufälliges Element zurückgibt, ohne die Menge zu ändern.

```php
$redis->del('key1');
$redis->sAdd('key1' , 'member1');
$redis->sAdd('key1' , 'member2');
$redis->sAdd('key1' , 'member3'); 

$redis->sRandMember('key1', function ($r) {
    var_dump($r); // member1
});

$redis->sRandMember('key1', 2, function ($r) {
    var_dump($r); // ['member1', 'member2']
});
$redis->sRandMember('key1', -100, function ($r) {
    var_dump($r); // ['member1', 'member2', 'member3', 'member3', ...]
});
$redis->sRandMember('empty-set', 100, function ($r) {
    var_dump($r); // []
}); 
$redis->sRandMember('not-a-set', 100, function ($r) {
    var_dump($r); // []
});
```
## **sRem**

Entfernt ein oder mehrere Elemente aus einer Menge, wobei nicht vorhandene Elemente ignoriert werden.

Gibt die Anzahl der erfolgreich entfernten Elemente zurück, ohne die ignorierten Elemente zu zählen.

Gibt false zurück, wenn der Schlüssel nicht vom Typ Menge ist.

Vor Redis Version 2.4 akzeptierte `SREM` nur ein einzelnes Element.

```php
$redis->sRem('key1', ['member2', 'member3'], function ($r) {
    var_dump($r); 
});
```

## **sUnion**

Gibt die Vereinigungsmenge der angegebenen Mengen zurück. Nicht vorhandene Schlüssel werden als leere Menge betrachtet.

```php
$redis->sUnion(['s0', 's1', 's2'], function ($r) {
    var_dump($r); // []
});
```

## **sUnionStore**

Speichert die Vereinigungsmenge der angegebenen Mengen in der angegebenen Zielmenge und gibt die Anzahl der Elemente zurück. Wenn die Zielmenge bereits existiert, wird sie überschrieben.

```php
$redis->del('s0', 's1', 's2');
$redis->sAdd('s0', '1');
$redis->sAdd('s0', '2');
$redis->sAdd('s1', '3');
$redis->sAdd('s1', '1');
$redis->sAdd('s2', '3');
$redis->sAdd('s2', '4');
$redis->sUnionStore('dst', 's0', 's1', 's2', function ($r) {
    var_dump($r); // 4
});
$redis->sMembers('dst', function ($r) {
    var_dump($r); // ['1', '2', '3', '4']
});
```

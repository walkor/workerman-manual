# webman-redis

## Introduzione

webman/redis è un componente Redis asincrono basato su workerman.

> **Nota**
> Lo scopo principale di questo progetto è realizzare la sottoscrizione asincrona di Redis (subscribe, pSubscribe).
> Poiché Redis è sufficientemente veloce, questo client asincrono è necessario solo per la sottoscrizione asincrona di ps psubscribe, altrimenti è consigliabile utilizzare l'estensione Redis per prestazioni migliori.

## Installazione:

```composer require workerman/redis```

## Uso Callback

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

## Uso Coroutine

> **Nota**
> L'uso delle coroutine richiede workerman>=5.0, workerman/redis>=2.0.0 e l'installazione di composer require revolt/event-loop ^1.0.0

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

Quando non si impostano le funzioni di callback, il client utilizzerà le modalità di ritorno asincrone e non bloccherà il processo corrente durante le richieste, consentendo il processamento concorrente delle richieste.

> **Nota**
> L'uso delle coroutine non è supportato per psubscribe subscribe.

# Documentazione
**Spiegazione**

**Nel metodo di callback, di solito ci sono 2 parametri ($result, $redis) nella funzione di callback, where `$result` è il risultato e `$redis` è un'istanza di Redis. Ad esempio:**
```php
use Workerman\Redis\Client;
$redis = new Client('redis://127.0.0.1:6379');
// Imposta la funzione di callback per verificare i risultati della chiamata set
$redis->set('key', 'value', function ($result, $redis) {
    var_dump($result); // true
});
// Le funzioni di callback sono opzionali, qui viene omessa la funzione di callback
$redis->set('key1', 'value1');
// Le funzioni di callback possono essere nidificate
$redis->get('key', function ($result, $redis){
    $redis->set('key2', 'value2', function ($result) {
        var_dump($result);
    });
});
```

## **Connessione**
```php
use Workerman\Redis\Client;
// Funzione di callback omessa
$redis = new Client('redis://127.0.0.1:6379');
// Con funzione di callback
$redis = new Client('redis://127.0.0.1:6379', [
    'connect_timeout' => 10 // Impostazione del timeout di connessione a 10 secondi, valore predefinito 5 secondi se non impostato
], function ($success, $redis) {
    // Callback di risultato della connessione
    if (!$success) echo $redis->error();
});
```

## **Auth**
```php
// Verifica della password
$redis->auth('password', function ($result) {

});
// Verifica nome utente e password
$redis->auth('username', 'password', function ($result) {

});
```

## **pSubscribe**

Sottoscrive uno o più canali che corrispondono a modelli specifici.

Ogni modello utilizza il carattere di * come simbolo jolly. Ad esempio, it* corrisponde a tutti i canali che iniziano con it (it.news, it.blog, it.tweets, ecc.). news.* corrisponde a tutti i canali che iniziano con news. (news.it, news.global.today, ecc.), e così via.

Nota: la funzione di callback di pSubscribe ha 4 parametri ($pattern, $channel, $message, $redis).

Dopo che un'istanza di $redis ha chiamato l'interfaccia pSubscribe o subscribe, chiamare altri metodi su questa istanza sarà ignorato.

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

## **Subscribe**

Utilizzato per sottoscrivere i messaggi per uno o più canali specifici.

Nota: la funzione di callback di subscribe ha 3 parametri ($channel, $message, $redis).

Dopo che un'istanza di $redis ha chiamato l'interfaccia pSubscribe o subscribe, chiamare altri metodi su questa istanza sarà ignorato.

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

## **Publish**

Utilizzato per inviare un messaggio a un canale specifico.

Restituisce il numero di abbonati che hanno ricevuto il messaggio.

```php
$redis2->publish('news', 'news content');
```

## **Select**
```php
// Funzione di callback omessa
$redis->select(2);
$redis->select('test', function ($result, $redis) {
    // Il parametro select deve essere un numero, quindi $result sarà false in questo caso
    var_dump($result, $redis->error());
});
```

## **Get**

Comando utilizzato per ottenere il valore di una chiave specifica. Se la chiave non esiste, restituisce NULL. Se il valore memorizzato in key non è di tipo stringa, restituisce false.
```php
$redis->get('key', function($result) {
     // Se la chiave non esiste, restituisce NULL; se si verifica un errore, restituisce false
    var_dump($result);
});
```

## **Set**

Utilizzato per impostare il valore di una chiave specifica. Se la chiave esiste già, sovrascrive il valore precedente, ignorando il tipo.
```php
$redis->set('key', 'value');
$redis->set('key', 'value', function($result){});
// Il terzo parametro può essere utilizzato per specificare il tempo di scadenza, scaduto dopo 10 secondi
$redis->set('key','value', 10);
$redis->set('key','value', 10, function($result){});
```

## **SetEx, pSetEx**

Imposta il valore e il tempo di scadenza per una chiave specificata. Se la chiave esiste già, pSetEx sostituirà il valore esistente.
```php
// Notare che il secondo parametro specifica il tempo di scadenza in secondi
$redis->setEx('key', 3600, 'value'); 
// pSetEx utilizza il tempo in millisecondi
$redis->pSetEx('key', 3600, 'value'); 
```

## **Del**

Utilizzato per eliminare una chiave esistente. Restituisce il numero di chiavi eliminate (le chiavi non esistenti non vengono conteggiate).
```php
// Elimina una chiave
$redis->del('key');
// Elimina più chiavi
$redis->del(['key', 'key1', 'key2']);
```

## **SetNx**

(Set if Not exists) Utilizzato per impostare il valore di una chiave solo se la chiave non esiste.
```php
$redis->del('key');
$redis->setNx('key', 'value', function($result){
    var_dump($result); // 1
});
$redis->setNx('key', 'value', function($result){
    var_dump($result); // 0
});
```

## **Exists**

Utilizzato per controllare se una chiave specificata esiste. Restituisce il numero di chiavi esistenti.
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

## **Incr, IncrBy**

Incrementa il valore memorizzato nella chiave di uno o un valore specificato. Se la chiave non esiste, verrà inizializzata a 0 prima di eseguire l'operazione incr/incrBy.
Se il valore contiene un tipo errato, o se una stringa non può essere rappresentata come numero, restituirà false.
Restituisce il valore incrementato se successo.
```php
$redis->incr('key1', function ($result) {
    var_dump($result);
}); 
$redis->incrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **IncrByFloat**

Incrementa il valore memorizzato nella chiave di un valore float specificato. Se la chiave non esiste, verrà inizializzata a 0 prima di eseguire l'operazione di incremento.
Se il valore contiene un tipo errato, o se una stringa non può essere rappresentata come numero, restituirà false.
Restituisce il valore incrementato.
```php
$redis->incrByFloat('key1', 1.5, function ($result) {
    var_dump($result);
}); 
```

## **Decr, DecrBy**

Decrementa il valore memorizzato nella chiave di uno o un valore specificato. Se la chiave non esiste, verrà inizializzata a 0 prima di eseguire l'operazione decr/decrBy.
Se il valore contiene un tipo errato, o se una stringa non può essere rappresentata come numero, restituirà false.
Restituisce il valore decrementato.
```php
$redis->decr('key1', function ($result) {
    var_dump($result);
}); 
$redis->decrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **mGet**

Restituisce i valori di tutte le chiavi specificate. Se una chiave specificata non esiste, restituirà NULL.
```php
$redis->set('key1', 'value1');
$redis->set('key2', 'value2');
$redis->set('key3', 'value3');
$redis->mGet(['key0', 'key1', 'key5'], function ($result) {
    var_dump($result); // [null, 'value1', null];
}); 
```
## **getSet**

Utilizzato per impostare il valore di una chiave specifica e restituire il vecchio valore della chiave.

```php
$redis->set('x', '42');
$redis->getSet('x', 'lol', function ($result) {
    var_dump($result); // '42'
}) ;
$redis->get('x', function ($result) {
    var_dump($result); // 'lol'
}) ;
```

## **randomKey**

Restituisce casualmente una chiave dal database corrente.
```php
$redis->randomKey(function($key) use ($redis) {
    $redis->get($key, function ($result) {
        var_dump($result); 
    }) ;
})
```

## **move**

Sposta la chiave del database corrente nel database specificato db.
```php
$redis->select(0);	// passa al DB 0
$redis->set('x', '42');	// scrive 42 in x
$redis->move('x', 1, function ($result) { 	// sposta in DB 1
    var_dump($result); // 1
}) ;  
$redis->select(1);	// passa al DB 1
$redis->get('x', function ($result) {
    var_dump($result); // '42'
}) ;
```

## **rename**

Modifica il nome della chiave; se la chiave non esiste, restituisce false.
```php
$redis->set('x', '42');
$redis->rename('x', 'y', function ($result) {
    var_dump($result); // true
}) ;
```

## **renameNx**

Cambia il nome della chiave solo se il nuovo nome non esiste.
```php
$redis->del('y');
$redis->set('x', '42');
$redis->renameNx('x', 'y', function ($result) {
    var_dump($result); // 1
}) ;
```

## **expire**

Imposta il tempo di scadenza della chiave; dopo la scadenza la chiave non sarà più disponibile. Il tempo è misurato in secondi. Restituisce 1 in caso di successo, 0 se la chiave non esiste, e false in caso di errore.
```php
$redis->set('x', '42');
$redis->expire('x', 3);
```

## **keys**

Usato per trovare tutte le chiavi che corrispondono al modello specificato.
```php
$redis->keys('*', function ($keys) {
    var_dump($keys); 
}) ;
$redis->keys('user*', function ($keys) {
    var_dump($keys); 
}) ;
```

## **type**

Restituisce il tipo di valore memorizzato nella chiave. Il risultato sarà una stringa, che può essere string, set, list, zset, hash o none, che indica l'assenza della chiave.
```php
$redis->type('key', function ($result) {
    var_dump($result); // string set list zset hash none
}) ;
```

## **append**

Se la chiave esiste ed è una stringa, l'operazione APPEND aggiornerà il valore della chiave aggiungendo il valore alla fine e restituendo la lunghezza della stringa. Se la chiave non esiste, APPEND imposterà la chiave con il valore specificato e restituirà la lunghezza della stringa. Se la chiave esiste ma non è una stringa, restituirà false.

```php
$redis->set('key', 'value1');
$redis->append('key', 'value2', function ($result) {
    var_dump($result); // 12
}) ; 
$redis->get('key', function ($result) {
    var_dump($result); // 'value1value2'
}) ;
```

## **getRange**

Restituisce una sottostringa della stringa memorizzata nella chiave specificata. La sottostringa è determinata da due offset start e end (inclusi). Se la chiave non esiste, restituisce una stringa vuota. Se la chiave non è di tipo stringa, restituisce false.
```php
$redis->set('key', 'string value');
$redis->getRange('key', 0, 5, function ($result) {
    var_dump($result); // 'string'
}) ; 
$redis->getRange('key', -5, -1 , function ($result) {
    var_dump($result); // 'value'
}) ;
```

## **setRange**

Sovrascrive il valore della chiave con la stringa specificata, a partire dall'offset specificato. Se la chiave non esiste, imposta la chiave con la stringa specificata e restituisce la lunghezza della stringa. Se la chiave non è di tipo stringa, restituisce false.

```php
$redis->set('key', 'Hello world');
$redis->setRange('key', 6, "redis", function ($result) {
    var_dump($result); // 11
}) ; 
$redis->get('key', function ($result) {
    var_dump($result); // 'Hello redis'
}) ; 
```

## **strLen**

Restituisce la lunghezza del valore della stringa memorizzato in una data chiave. Se la chiave non contiene una stringa, restituisce false.
```php
$redis->set('key', 'value');
$redis->strlen('key', function ($result) {
    var_dump($result); // 5
}) ; 
```

## **getBit**

Restituisce il valore del bit specificato all'offset della stringa memorizzata nella chiave.
```php
$redis->set('key', "\x7f"); // questo è 0111 1111
$redis->getBit('key', 0, function ($result) {
    var_dump($result); // 0
}) ; 
```

## **setBit**

Imposta o cancella il bit specificato all'offset della stringa memorizzata nella chiave. Restituisce 0 o 1 che rappresenta il valore prima della modifica.
```php
$redis->set('key', "*");	// ord("*") = 42 = 0x2a = "0010 1010"
$redis->setBit('key', 5, 1, function ($result) {
    var_dump($result); // 0
}) ; 
```

## **bitOp**

Esegue operazioni bitwise tra le chiavi (contenenti valori stringa) e memorizza il risultato nella chiave di destinazione.

Il comando BITOP supporta quattro operazioni bitwise: AND, OR, XOR e NOT.

Restituisce la dimensione della stringa memorizzata nella chiave di destinazione, che è uguale alla dimensione della stringa di input più lunga.
```php
$redis->set('key1', "abc");
$redis->bitOp( 'AND', 'dst', 'key1', 'key2', function ($result) {
    var_dump($result); // 3
}) ;
```

## **bitCount**

Conta il numero di bit impostati (conta la popolazione) nella stringa. Di default, verranno controllati tutti i byte della stringa. È possibile specificare un intervallo aggiuntivo con i parametri start e end. Come il comando GETRANGE, è possibile utilizzare valori negativi per indicizzare i byte dalla fine della stringa, dove -1 è l'ultimo byte, -2 è il penultimo, e così via. Restituisce il numero di bit impostati nella stringa, considerando solo i bit con valore 1. Se la chiave non esiste, restituirà zero.

```php
$redis->set('key', 'hello');
$redis->bitCount( 'key', 0, 0, function ($result) {
    var_dump($result); // 3
}) ;
$redis->bitCount( 'key', function ($result) {
    var_dump($result); //21
}) ;
```

## **sort**

Il comando sort può ordinare gli elementi di una lista, di un insieme o di un insieme ordinato. 

Prototipo: `sort($key, $options, $callback);`

L'opzione può essere specificata con i seguenti campi opzionali
~~~
$options = [
     'by' => 'some_pattern_*',
    'limit' => [0, 1],
    'get' => 'some_other_pattern_*', // o un array di pattern
    'sort' => 'asc', // o 'desc'
    'alpha' => true,
    'store' => 'external-key'
];
~~~

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

## **ttl, pttl**

Restituisce il tempo rimanente per la scadenza della chiave, in secondi o millisecondi.

Se la chiave non ha un tempo di scadenza, restituisce -1. Se la chiave non esiste, restituisce -2.

```php
$redis->set('key', 'value', 10);
// in secondi
$redis->ttl('key', function ($result) {
    var_dump($result); // 10
});
// in millisecondi
$redis->pttl('key', function ($result) {
    var_dump($result); // 9999
});
// La chiave non esiste
$redis->pttl('key-not-exists', function ($result) {
    var_dump($result); // -2
});
```

## **persist**

Rimuove la scadenza della chiave specificata, rendendola permanente. 

Restituisce 1 se la scadenza è stata rimossa con successo, 0 se la chiave non esiste o non ha una scadenza impostata, e false in caso di errore.
```php
$redis->persist('key');
```

## **mSet, mSetNx**

Imposta contemporaneamente più coppie di chiave-valore in un'unica operazione atomica. In mSetNx, restituirà 1 solo se tutte le chiavi sono state impostate.

Restituisce 1 in caso di successo, 0 in caso di fallimento e false in caso di errore.
```php
$redis->mSet(['key0' => 'value0', 'key1' => 'value1']);
```

## **hSet**

Assegna un valore a un campo specifico di un hash. Se il campo è nuovo e il valore viene impostato correttamente, restituisce 1. Se il campo esiste già e il vecchio valore viene sovrascritto da uno nuovo, restituisce 0.

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

## **hSetNx**

Assegna un valore a un campo specifico di un hash solo se il campo non esiste. Se l'hash non esiste, viene creato e viene eseguita l'operazione di assegnazione HSETNX solo se il campo non esiste.

```php
$redis->del('h');
$redis->hSetNx('h', 'key1', 'hello', function ($r) {
    var_dump($r); // 1
});
$redis->hSetNx('h', 'key1', 'world', function ($r) {
    var_dump($r); // 0
});
```

## **hGet**

Restituisce il valore del campo specificato in un hash. Se il campo o l'hash specificato non esistono, restituisce null.

```php
$redis->hGet('h', 'key1', function ($result) {
    var_dump($result);
});
```
## **hLen**

Utilizzato per ottenere il numero dei campi nel campo hash.

Quando la chiave non esiste, restituisce 0.

```php
$redis->del('h');
$redis->hSet('h', 'key1', 'ciao');
$redis->hSet('h', 'key2', 'plop');
$redis->hLen('h', function ($result) {
    var_dump($result); // 2
});
```

## **hDel**

Il comando viene utilizzato per eliminare uno o più campi specificati nel campo hash key; i campi inesistenti verranno ignorati.

Restituisce il numero di campi eliminati con successo, escludendo quelli ignorati. Se key non è un hash, restituisce false.

```php
$redis->hDel('h', 'key1');
```

## **hKeys**

Restituisce tutti i campi del hash come array.

Se la chiave non esiste, restituisce un array vuoto. Se la chiave non è un hash, restituisce false.

```php
$redis->hKeys('key', function ($result) {
    var_dump($result);
});
```

## **hVals**

Restituisce tutti i valori dei campi del hash come array.

Se la chiave non esiste, restituisce un array vuoto. Se la chiave non è un hash, restituisce false.

```php
$redis->hVals('key', function ($result) {
    var_dump($result);
});
```

## **hGetAll**

Restituisce tutti i campi e i valori del hash come array associativo.

Se la chiave non esiste, restituisce un array vuoto. Se la chiave non è di tipo hash, restituisce false.

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
Output
```php
array (
    'a' => 'x',
    'b' => 'y',
    'c' => 'z',
    'd' => 't',
)
```

## **hExists**

Controlla se il campo specificato esiste nel campo hash. Restituisce 1 se esiste, 0 se il campo o la chiave non esistono, e false in caso di errore.

```php
$redis->hExists('h', 'a', function ($result) {
    var_dump($result); //
});
```

## **hIncrBy**

Utilizzato per aumentare il valore del campo nel campo hash di una quantità specificata.

È possibile specificare un incremento negativo per effettuare una sottrazione. Se la chiave hash non esiste, verrà creata e sarà eseguito il comando HINCRBY.

Se il campo specificato non esiste, il suo valore viene inizializzato a 0 prima dell'esecuzione del comando.

Il comando restituisce false se viene eseguito su un campo che contiene una stringa.

Il valore dell'operazione è limitato a 64 bit di rappresentazione di numeri con segno.

```php
$redis->del('h');
$redis->hIncrBy('h', 'x', 2,  function ($result) {
    var_dump($result);
});
```

## **hIncrByFloat**

Simile a hIncrBy, ma l'incremento è di tipo float.

## **hMSet**

Imposta contemporaneamente più coppie campo-valore nel campo hash.

Questo comando sovrascrive i campi esistenti nel campo hash. Se il campo hash non esiste, verrà creato un campo hash vuoto prima dell'esecuzione di HMSET.

```php
$redis->del('h');
$redis->hMSet('h', ['name' => 'Joe', 'sex' => 1])
```

## **hMGet**

Restituisce i valori dei campi specificati nel campo hash come array associativo.

Se il campo specificato non esiste nel campo hash, il suo valore sarà null. Se la chiave non è di tipo hash, restituisce false.

```php
$redis->del('h');
$redis->hSet('h', 'field1', 'value1');
$redis->hSet('h', 'field2', 'value2');
$redis->hMGet('h', ['field1', 'field2', 'field3'], function ($r) {
    var_export($r);
});
```
Output
```php
array (
 'field1' => 'value1',
 'field2' => 'value2',
 'field3' => null
)
```

## **blPop, brPop**

Rimuovono e ottengono il primo/ultimo elemento della lista. Se la lista è vuota, verrà bloccata fino alla scadenza o fino a quando sarà disponibile un elemento da estrarre.

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

Rimuove l'ultimo elemento dalla lista sorgente, lo sposta nella lista destinazione e lo restituisce. Se la lista è vuota, verrà bloccata fino alla scadenza.

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

Restituisce l'elemento della lista corrispondente all'indice specificato. È possibile utilizzare indici negativi per accedere agli elementi dalla fine della lista.

Restituisce null se l'indice specificato non è nell'intervallo della lista. Se la chiave non è di tipo lista, restituisce false.

```php
$redis->del('key1']);
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lindex('key1', 0, function ($r) {
    var_dump($r); // A
});
```

## **lInsert**

Inserisce un elemento prima o dopo un elemento specificato nella lista. Se l'elemento specificato non esiste nella lista, non viene eseguita alcuna operazione.

Se la lista non esiste, viene considerata vuota e non viene eseguita alcuna operazione.

Se la chiave non è di tipo lista, restituisce false.

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

Rimuove e restituisce il primo elemento della lista.

Restituisce null se la lista key non esiste.

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lPop('key1', function ($r) {
    var_dump($r); // A
});
```

## **lPush**

Inserisce uno o più valori all'inizio della lista. Se la chiave non esiste, verrà creata una lista vuota prima di eseguire LPUSH.

**Nota:** Prima della versione 2.4 di Redis, il comando LPUSH accettava solo un singolo valore.

```php
$redis->del('key1');
$redis->lPush('key1', 'A');
$redis->lPush('key1', ['B','C']);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lPushx**

Inserisce un valore all'inizio di una lista esistente. Se la lista non esiste, l'operazione non viene eseguita e restituisce 0. Se la chiave non è di tipo lista, restituisce false.

Restituisce la lunghezza della lista dopo l'esecuzione di lPushx.

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

Restituisce gli elementi della lista in un intervallo specificato. Gli indici partono da 0 e possono essere anche negativi per contare dal fondo della lista.

Restituisce un array contenente gli elementi specificati nell'intervallo. Se la chiave non è di tipo lista, restituisce false.

```php
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lRem**

Rimuove gli elementi della lista che corrispondono al valore specificato in base al parametro COUNT.

COUNT può assumere i seguenti valori:

* count > 0 : cerca dalla testa alla coda e rimuove COUNT elementi uguali a VALUE.
* count < 0 : cerca dalla coda alla testa e rimuove |COUNT| elementi uguali a VALUE.
* count = 0 : rimuove tutti gli elementi uguali a VALUE.

Restituisce il numero di elementi rimossi. Se la lista non esiste, restituisce 0. Se la chiave non è di tipo lista, restituisce false.

```php
$redis->lRem('key1', 2, 'A', function ($r) {
    var_dump($r); 
});
```

## **lSet**

Imposta il valore di un elemento della lista utilizzando l'indice specificato.

Restituisce true in caso di successo, false se l'indice specificato non è valido o se la lista è vuota.

```php
$redis->lSet('key1', 0, 'X');
```
## lTrim

La funzione lTrim taglia una lista, cioè mantiene solo gli elementi all'interno dell'intervallo specificato, eliminando gli elementi al di fuori dell'intervallo specificato.

L'indice 0 rappresenta il primo elemento della lista, 1 rappresenta il secondo elemento della lista e così via. È anche possibile utilizzare indici negativi, ad esempio -1 rappresenta l'ultimo elemento della lista, -2 rappresenta il penultimo elemento della lista e così via.

Restituisce true in caso di successo, false altrimenti.

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

## rPop

Utilizzato per rimuovere l'ultimo elemento della lista e restituisce il valore rimosso.

Restituisce null se la lista non esiste.

```php
$redis->rPop('key1', function ($r) {
    var_dump($r);
});
```

## rPopLPush

Rimuove l'ultimo elemento di una lista e lo aggiunge a un'altra lista, restituendo il valore rimosso.

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

## rPush

Inserisce uno o più valori alla fine della lista (a destra) e restituisce la lunghezza della lista dopo l'inserimento.

Se la lista non esiste, viene creata una lista vuota e l'operazione di RPUSH viene eseguita. Quando la lista esiste ma non è una lista, restituisce false.

Nota: Nella versione di Redis 2.4, il comando RPUSH accetta solo un singolo valore.

```php
$redis->del('key1');
$redis->rPush('key1', 'A', function ($r) {
    var_dump($r); // 1
});
```

## rPushX

Inserisce un valore nella lista esistente alla fine (destra) e restituisce la lunghezza della lista. Se la lista non esiste, l'operazione non viene eseguita e restituisce 0.

Se la lista esiste ma non è una lista, restituisce false.

```php
$redis->del('key1');
$redis->rPushX('key1', 'A', function ($r) {
    var_dump($r); // 0
});
```

## lLen

Restituisce la lunghezza della lista. Se la chiave della lista non esiste, viene interpretata come una lista vuota e restituisce 0. Se la chiave non è di tipo lista, restituisce false.

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lLen('key1', function ($r) {
    var_dump($r); // 3
});
```

## sAdd

Aggiunge uno o più membri a un insieme, ignorando i membri già esistenti nell'insieme.

Se la chiave dell'insieme non esiste, viene creata un insieme contenente solo i membri aggiunti.

Restituisce false se la chiave non è di tipo insieme.

Nota: Nella versione di Redis 2.4, SADD accetta solo un singolo membro.

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

## sCard

Restituisce il numero di elementi nell'insieme. Se la chiave dell'insieme non esiste, restituisce 0.

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

## sDiff

Restituisce la differenza tra il primo insieme e gli altri insiemi, ovvero gli elementi unici del primo insieme. Visti come insiemi vuoti gli insiemi mancanti.

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

## sDiffStore

Salva la differenza tra gli insiemi specificati in un insieme specificato. Se la chiave specificata esiste, verrà sovrascritta.

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

## sInter

Restituisce l'intersezione di tutti gli insiemi specificati. Se un insieme specificato non esiste o è vuoto, l'intersezione sarà vuota.

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

## sInterStore

Salva l'intersezione degli insiemi specificati in un insieme specificato e restituisce il numero di elementi nell'insieme risultante. Sovrascrive la chiave specificata se esiste.

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

## sIsMember

Verifica se un membro è presente nell'insieme.

Restituisce 1 se il membro è presente, 0 se non è presente o la chiave non esiste, e false se la chiave corrispondente non è di tipo insieme.

```php
$redis->sIsMember('key1', 'member1', function ($r) {
    var_dump($r); 
});
```

## sMembers

Restituisce tutti i membri dell'insieme. Se la chiave dell'insieme non esiste, restituisce un insieme vuoto.

```php
$redis->sMembers('s', function ($r) {
    var_dump($r); 
});
```

## sMove

Muove un membro specificato dall'insieme di origine all'insieme di destinazione. L'operazione è atomica.

Se l'insieme di origine non esiste o non contiene il membro specificato, l'operazione non viene eseguita e restituisce 0. Se l'insieme di destinazione contiene già il membro, verrà semplicemente rimosso dall'insieme di origine.

Restituisce false se l'insieme di origine o di destinazione non è di tipo insieme.

```php
$redis->sMove('key1', 'key2', 'member13');
```

## sPop

Rimuove uno o più elementi casuali dall'insieme specificato e restituisce gli elementi rimossi.

Restituisce null se l'insieme non esiste o è vuoto.

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

## sRandMember

La funzione Srandmember di Redis restituisce un elemento casuale dall'insieme specificato.

A partire dalla versione 2.6 di Redis, Srandmember accetta un parametro opzionale count:

* Se count è un numero positivo e inferiore alla cardinalità dell'insieme, restituirà un array di count elementi unici. Se count è maggiore o uguale alla cardinalità dell'insieme, restituirà l'intero insieme.
* Se count è un numero negativo, restituirà un array di elementi che potrebbero ripetersi, con una lunghezza uguale al valore assoluto di count.

Questa operazione è simile a SPOP, ma mentre SPOP rimuove e restituisce l'elemento dall'insieme, Srandmember restituisce solo l'elemento senza modificare l'insieme.

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
## sRem

Rimuove uno o più elementi membri dal set e ignora quelli che non esistono.

Restituisce il numero di elementi rimossi con successo, escludendo quelli ignorati.

Restituisce false se la chiave non è di tipo set.

Prima della versione 2.4 di Redis, SREM accettava solo un singolo membro.

```php
$redis->sRem('key1', ['member2', 'member3'], function ($r) {
    var_dump($r); 
});
```

## sUnion

Restituisce l'unione dei set forniti. I set inesistenti vengono trattati come set vuoti.

```php
$redis->sUnion(['s0', 's1', 's2'], function ($r) {
    var_dump($r); // []
});
```

## sUnionStore

Calcola l'unione dei set forniti e la memorizza nel set di destinazione specificato, restituendo il numero di elementi. Se il set di destinazione esiste già, viene sovrascritto.

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

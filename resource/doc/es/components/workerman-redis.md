# workerman-redis

## Introducción

workeman/redis es un componente asíncrono de redis basado en workerman.

> **Nota**
> El propósito principal de este proyecto es implementar la suscripción asíncrona de redis (subscribe, pSubscribe). Debido a que redis es bastante rápido, a menos que se necesite la suscripción asíncrona psubscribe subscribe, no es necesario utilizar este cliente asíncrono. El uso de la extensión de redis proporcionará un mejor rendimiento.

## Instalación:

```composer require workerman/redis```

## Uso de devolución de llamada

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

## Uso de la corriente

> **Nota**
> El uso de la corriente requiere workerman>=5.0, workerman/redis>=2.0.0 y la instalación de composer require revolt/event-loop ^1.0.0

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

Cuando no se establecen las funciones de devolución de llamada, el cliente utilizará un enfoque sincrónico para devolver los resultados de las solicitudes asíncronas. El proceso de solicitud no bloqueará el subproceso actual, lo que permite el manejo concurrente de las solicitudes.

> **Nota**
> El uso de la corriente no es compatible con psubscribe subscribe.

# Documentación
**Aclaración**

**En el modo de devolución de llamada, la función de devolución de llamada usualmente tiene 2 parámetros ($result, $redis), donde `$result` es el resultado y `$redis` es la instancia de redis. Por ejemplo:**
```php
use Workerman\Redis\Client;
$redis = new Client('redis://127.0.0.1:6379');
// Establecer la función de devolución de llamada para evaluar el resultado de la llamada set
$redis->set('key', 'value', function ($result, $redis) {
    var_dump($result); // true
});
// Las funciones de devolución de llamada son parámetros opcionales, por lo que aquí se omite la función de devolución de llamada
$redis->set('key1', 'value1');
// Las funciones de devolución de llamada pueden ser anidadas
$redis->get('key', function ($result, $redis){
    $redis->set('key2', 'value2', function ($result) {
        var_dump($result);
    });
});
```

## **Conexión**
```php
use Workerman\Redis\Client;
// Omitir la devolución de llamada
$redis = new Client('redis://127.0.0.1:6379');
// Con devolución de llamada
$redis = new Client('redis://127.0.0.1:6379', [
    'connect_timeout' => 10 // Establecer un tiempo de espera de conexión de 10 segundos, el valor predeterminado es 5 segundos
], function ($success, $redis) {
    // Devolución de llamada del resultado de la conexión
    if (!$success) echo $redis->error();
});
```

## **auth**
```php
// Autenticación de contraseña
$redis->auth('password', function ($result) {
    
});
// Autenticación de nombre de usuario y contraseña
$redis->auth('username', 'password', function ($result) {

});
```

## **pSubscribe**

Suscribe uno o más canales que coincidan con los patrones dados.

Cada patrón utiliza el carácter * como comodín, por ejemplo, it\* coincidirá con todos los canales que comiencen con it (it.news, it.blog, it.tweets, etc.). news.\* coincidirá con todos los canales que comiencen con news. (news.it, news.global.today, etc.), y así sucesivamente.

Nota: la función de devolución de llamada de pSubscribe tiene 4 parámetros ($pattern, $channel, $message, $redis).

Una vez que una instancia de $redis llama a los métodos pSubscribe o subscribe, llamar a otros métodos en la misma instancia será ignorado.

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

Se utiliza para suscribirse a uno o varios canales específicos.

Nota: la función de devolución de llamada de subscribe tiene 3 parámetros ($channel, $message, $redis).

Una vez que una instancia de $redis llama a los métodos pSubscribe o subscribe, llamar a otros métodos en la misma instancia será ignorado.

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

Se utiliza para enviar información a un canal específico.

Devuelve la cantidad de suscriptores que recibieron la información.

```php
$redis2->publish('news', 'news content');
```

## **select**
```php
// Omitir la devolución de llamada
$redis->select(2);
$redis->select('test', function ($result, $redis) {
    // El argumento select debe ser numérico, por lo que $result aquí es false
    var_dump($result, $redis->error());
});
```

## **get**

El comando se utiliza para obtener el valor de una clave específica. Si la clave no existe, devuelve NULL. Si el valor almacenado en la clave no es de tipo cadena, devuelve false.

```php
$redis->get('key', function($result) {
     // Si la clave no existe, devuelve NULL; si hay un error, devuelve false
    var_dump($result);
});
```

## **set**

Se utiliza para establecer el valor de una clave específica. Si la clave ya tiene un valor almacenado, SET sobrescribirá el valor antiguo, independientemente de su tipo.

```php
$redis->set('key', 'value');
$redis->set('key', 'value', function($result){});
// El tercer parámetro puede ser el tiempo de expiración, expirará después de 10 segundos
$redis->set('key','value', 10);
$redis->set('key','value', 10, function($result){});
```

## **setEx, pSetEx**

Establece el valor y el tiempo de expiración para una clave específica. Si la clave ya existe, el comando SETEX reemplazará el valor antiguo.

```php
// Nota que el segundo parámetro es el tiempo de expiración en segundos
$redis->setEx('key', 3600, 'value'); 
// pSetEx con el tiempo de expiración en milisegundos
$redis->pSetEx('key', 3600, 'value'); 
```

## **del**

Se utiliza para eliminar claves existentes. Devuelve el número de claves eliminadas (las claves no existentes no se cuentan).

```php
// Eliminar una clave
$redis->del('key');
// Eliminar varias claves
$redis->del(['key', 'key1', 'key2']);
```

## **setNx**

(SET if Not eXists) Se utiliza para establecer un valor para una clave solo si esta no existe.

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

Se utiliza para verificar si una clave específica existe. Devuelve el número de claves que existen.

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

Incrementa el valor de una clave en uno o en un valor específico. Si la clave no existe, se inicializará con 0 antes de realizar la operación incr/incrBy. Si el valor contiene un tipo incorrecto o el valor de tipo cadena no se puede representar como un número, devolverá false. Si tiene éxito, devolverá el valor incrementado.

```php
$redis->incr('key1', function ($result) {
    var_dump($result);
}); 
$redis->incrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **incrByFloat**

Agrega un valor flotante específico al valor almacenado de una clave. Si la clave no existe, INCRBYFLOAT inicializará la clave con 0 antes de realizar la operación de suma. Si el valor contiene un tipo incorrecto o el valor de tipo cadena no se puede representar como un número, devolverá false. Si tiene éxito, devolverá el valor sumado.

```php
$redis->incrByFloat('key1', 1.5, function ($result) {
    var_dump($result);
}); 
```

## **decr, decrBy**

Disminuye el valor de una clave en uno o en un valor específico. Si la clave no existe, se inicializará con 0 antes de realizar la operación decr/decrBy. Si el valor contiene un tipo incorrecto o el valor de tipo cadena no se puede representar como un número, devolverá false. Si tiene éxito, devolverá el valor disminuido.

```php
$redis->decr('key1', function ($result) {
    var_dump($result);
}); 
$redis->decrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **mGet**

Devuelve los valores de una o más claves especificadas. Si alguna de las claves no existe, devuelve NULL.

```php
$redis->set('key1', 'value1');
$redis->set('key2', 'value2');
$redis->set('key3', 'value3');
$redis->mGet(['key0', 'key1', 'key5'], function ($result) {
    var_dump($result); // [null, 'value1', null];
}); 
```
## **getSet**

Se utiliza para establecer el valor de una clave específica y devolver el valor antiguo de la clave.

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

Devuelve una clave aleatoria de la base de datos actual.
```php
$redis->randomKey(function($key) use ($redis) {
    $redis->get($key, function ($result) {
        var_dump($result); 
    }) ;
})
```

## **move**

Mueve la clave de la base de datos actual a la base de datos dada.
```php
$redis->select(0);	// cambia a la base de datos 0
$redis->set('x', '42');	// escribe 42 en x
$redis->move('x', 1, function ($result) { 	// mueve a la base de datos 1
    var_dump($result); // 1
}) ;  
$redis->select(1);	// cambia a la base de datos 1
$redis->get('x', function ($result) {
    var_dump($result); // '42'
}) ;
```

## **rename**

Cambia el nombre de una clave. Retorna falso si la clave no existe.
```php
$redis->set('x', '42');
$redis->rename('x', 'y', function ($result) {
    var_dump($result); // true
}) ;
```

## **renameNx**

Cambia el nombre de una clave si la nueva clave no existe.
```php
$redis->del('y');
$redis->set('x', '42');
$redis->renameNx('x', 'y', function ($result) {
    var_dump($result); // 1
}) ;
```

## **expire**

Establece el tiempo de expiración de una clave en segundos. Devuelve 1 en caso de éxito, 0 si la clave no existe y falso en caso de error.
```php
$redis->set('x', '42');
$redis->expire('x', 3);
```

## **keys**

Busca todas las claves que coinciden con el patrón dado.
```php
$redis->keys('*', function ($keys) {
    var_dump($keys); 
}) ;
$redis->keys('user*', function ($keys) {
    var_dump($keys); 
}) ;
```

## **type**

Devuelve el tipo de valor almacenado en una clave. Puede ser string, set, list, zset, hash o none si la clave no existe.
```php
$redis->type('key', function ($result) {
    var_dump($result); // string set list zset hash none
}) ;
```

## **append**

Si la clave existe y es una cadena, APPEND agrega el valor al final de la clave y devuelve la longitud de la cadena resultante. 
Si la clave no existe, simplemente establece la clave con el valor dado y devuelve la longitud de la cadena.
Si la clave existe pero no es una cadena, devuelve falso.

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

Obtiene una subcadena de la cadena almacenada en la clave dada. Devuelve una cadena vacía si la clave no existe y falso si la clave no es de tipo cadena.
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
Reemplaza parte de la cadena almacenada en una clave con una nueva cadena a partir de un desplazamiento dado. Si la clave no existe, la crea y establece la cadena. Devuelve la longitud de la cadena modificada.

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

Obtiene la longitud de la cadena almacenada en la clave dada. Devuelve falso si la clave no almacena una cadena.

```php
$redis->set('key', 'value');
$redis->strlen('key', function ($result) {
    var_dump($result); // 5
}) ; 
```

## **getBit**

Obtiene el bit en el desplazamiento dado de la cadena almacenada en la clave.
```php
$redis->set('key', "\x7f"); // esto es 0111 1111
$redis->getBit('key', 0, function ($result) {
    var_dump($result); // 0
}) ; 
```

## **setBit**

Establece o borra el bit en el desplazamiento dado de la cadena almacenada en la clave. Devuelve 0 o 1, que es el valor antes de la modificación.

```php
$redis->set('key', "*");	// ord("*") = 42 = 0x2f = "0010 1010"
$redis->setBit('key', 5, 1, function ($result) {
    var_dump($result); // 0
}) ; 
```

## **bitOp**

Realiza una operación a nivel de bits en múltiples claves (almacenando cadenas) y almacena el resultado en la clave de destino.

BITOP admite cuatro operaciones a nivel de bits: **AND**, **OR**, **XOR** y **NOT**.

Devuelve la longitud de la cadena almacenada en la clave de destino, que es igual a la longitud de la cadena más larga de entrada.

```php
$redis->set('key1', "abc");
$redis->bitOp( 'AND', 'dst', 'key1', 'key2', function ($result) {
    var_dump($result); // 3
}) ;
```

## **bitCount**

Cuenta el número de bits establecidos en una cadena (conteo de población). 

Por defecto, verifica todos los bytes en la cadena. Se pueden especificar intervalos de conteo utilizando los parámetros adicionales *start* y *end*.

Similar al comando GETRANGE, los valores de inicio y fin pueden ser negativos para indexar bytes desde el final de la cadena, donde -1 es el último byte, -2 es el penúltimo byte, y así sucesivamente. 

Devuelve el número de bits establecidos en la cadena. Para claves inexistentes, se considera como una cadena vacía y el comando devuelve 0.

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

El comando sort ordena los elementos de una lista, conjunto o conjunto ordenado.

Formato: `sort($key, $options, $callback);`

Donde las opciones son pares clave-valor opcionales:

~~~
$options = [
     'by' => 'some_pattern_*',
    'limit' => [0, 1],
    'get' => 'some_other_pattern_*', // o un array de patrones
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

Devuelve el tiempo restante para que una clave expire en segundos/milisegundos.

Si la clave no tiene TTL, devuelve -1. Si la clave no existe, devuelve -2.

```php
$redis->set('key', 'value', 10);
// en segundos
$redis->ttl('key', function ($result) {
    var_dump($result); // 10
});
// en milisegundos
$redis->pttl('key', function ($result) {
    var_dump($result); // 9999
});
// clave no existe
$redis->pttl('key-not-exists', function ($result) {
    var_dump($result); // -2
});
```

## **persist**

Elimina el tiempo de expiración de una clave, evitando que expire. 

Devuelve 1 si tiene éxito, 0 si la clave no existe o no tiene tiempo de expiración, y falso si ocurre un error.

```php
$redis->persist('key');
```

## **mSet, mSetNx**

Establece múltiples pares clave-valor en un solo comando atómico. mSetNx solo devuelve 1 si se establecen todas las claves.

Devuelve 1 en caso de éxito, 0 en caso de fallo, y falso si ocurre un error.

```php
$redis->mSet(['key0' => 'value0', 'key1' => 'value1']);
```

## **hSet**

Establece el valor de un campo en el hash. 

Si el campo es nuevo y el valor se establece con éxito, devuelve 1. Si el campo ya existe en el hash y su valor antiguo es reemplazado por el nuevo, devuelve 0.

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

Establece el valor de un campo en el hash si el campo no existe.

Si el hash no existe, se crea un nuevo hash y se realiza la operación hSetNx.

Si el campo ya existe en el hash, la operación es inválida.

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

Devuelve el valor de un campo en el hash.

Si el campo o el hash no existen, devuelve null.
```php
$redis->hGet('h', 'key1', function ($result) {
    var_dump($result);
});
```  
## hLen

Se usa para obtener el número de campos en una tabla hash.

Devuelve 0 cuando la clave no existe.

```php
$redis->del('h');
$redis->hSet('h', 'key1', 'hello');
$redis->hSet('h', 'key2', 'plop');
$redis->hLen('h', function ($result) {
    var_dump($result); // 2
});
```

## hDel

Se utiliza para eliminar uno o más campos específicos de la tabla hash en la clave dada. Los campos que no existen serán ignorados.

Devuelve la cantidad de campos eliminados exitosamente, excluyendo los campos ignorados. Devuelve false si la clave no es una tabla hash.

```php
$redis->hDel('h', 'key1');
```

## hKeys

Obtiene todos los campos de la tabla hash como un array.

Si la clave no existe, devuelve un array vacío. Si la clave no corresponde a una tabla hash, devuelve false.

```php
$redis->hKeys('key', function ($result) {
    var_dump($result);
});
```

## hVals

Devuelve todos los valores de los campos de la tabla hash como un array.

Si la clave no existe, devuelve un array vacío. Si la clave no corresponde a una tabla hash, devuelve false.

```php
$redis->hVals('key', function ($result) {
    var_dump($result);
});
```

## hGetAll

Devuelve todos los campos y valores de la tabla hash en forma de un array asociativo.

Si la clave no existe, devuelve un array vacío. Si la clave no corresponde a una tabla hash, devuelve false.

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
Resultado
```php
array (
    'a' => 'x',
    'b' => 'y',
    'c' => 'z',
    'd' => 't',
)
```

## hExists

Verifica si un campo específico de la tabla hash existe. Devuelve 1 si existe, 0 si el campo o la clave no existen, y false si hay un error.

```php
$redis->hExists('h', 'a', function ($result) {
    var_dump($result); //
});
```

## hIncrBy

Se utiliza para incrementar el valor de un campo específico en la tabla hash por el valor especificado.

El incremento también puede ser un número negativo, lo que equivale a una operación de resta en el campo especificado.

Si la clave de la tabla hash no existe, se creará una nueva tabla hash y se ejecutará el comando HINCRBY.

Si el campo especificado no existe, se inicializará su valor en 0 antes de ejecutar el comando.

HINCRBY en un campo que almacena un valor de cadena devolverá false.

El valor de esta operación está limitado a 64 bits (números enteros con signo).

```php
$redis->del('h');
$redis->hIncrBy('h', 'x', 2,  function ($result) {
    var_dump($result);
});
```

## hIncrByFloat

Similar a hIncrBy, pero el incremento es un número decimal.

## hMSet

Establece múltiples pares campo-valor en la tabla hash.

Este comando sobrescribirá los campos existentes en la tabla hash. Si la tabla hash no existe, se creará una tabla hash vacía y se ejecutará la operación HMSET.

```php
$redis->del('h');
$redis->hMSet('h', ['name' => 'Joe', 'sex' => 1])
```

## hMGet

Devuelve los valores de uno o varios campos dados de la tabla hash como un array asociativo.

Si el campo especificado no existe en la tabla hash, su valor correspondiente será null. Si la clave no es una tabla hash, devuelve false.

```php
$redis->del('h');
$redis->hSet('h', 'field1', 'value1');
$redis->hSet('h', 'field2', 'value2');
$redis->hMGet('h', ['field1', 'field2', 'field3'], function ($r) {
    var_export($r);
});
```
Salida
```php
array (
 'field1' => 'value1',
 'field2' => 'value2',
 'field3' => null
)
```

## blPop, brPop

Elimina y obtiene el primer/último elemento de la lista. Si la lista está vacía, bloquea la lista hasta que haya un elemento para sacar o el tiempo de espera se agote.

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');

$redis->blPop(['key1', 'key2'], 10, function ($r) {
    var_export($r); // array ( 0 => 'key1', 1 => 'a')
});

Timer::add(1, function () use ($redis2) {
    $redis2->lpush('key1', 'a');
});
```

## bRPopLPush

Obtiene el último elemento de una lista y lo inserta al inicio de otra lista. Si la lista está vacía, bloquea la lista hasta que haya un elemento para sacar o el tiempo de espera se agote. Devuelve null si se agota el tiempo.

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

## lIndex

Obtiene el elemento de una lista según el índice proporcionado. También se pueden usar índices negativos, donde -1 representa el último elemento de la lista, -2 el penúltimo, y así sucesivamente.

Devuelve null si el índice especificado está fuera del rango de la lista. Si la clave no es una lista, devuelve false.

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lindex('key1', 0, function ($r) {
    var_dump($r); // A
});
```

## lInsert

Inserta un elemento antes o después de un elemento específico de la lista. No hace nada si el elemento especificado no está en la lista. Si la lista no existe, se considera una lista vacía y no se realiza ninguna operación.

Devuelve 0 si no se inserta ningún elemento, 1 si se inserta antes del elemento especificado, 4 si se inserta después del elemento especificado, y un array con los elementos de la lista después de la inserción.

Si la clave no es de tipo lista, devuelve false.

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

## lPop

Elimina y retorna el primer elemento de la lista.

Devuelve null si la lista no existe.

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lPop('key1', function ($r) {
    var_dump($r); // A
});
```

## lPush

Inserta uno o más valores al principio de la lista. Si la clave no existe, se crea una lista vacía y se realiza la operación LPUSH. 

**Nota:** En las versiones de Redis anteriores a la 2.4, el comando LPUSH solo acepta un valor único.

```php
$redis->del('key1');
$redis->lPush('key1', 'A');
$redis->lPush('key1', ['B','C']);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## lPushx

Inserta un valor al principio de una lista existente, de lo contrario, la operación no tiene efecto y devuelve 0. Si la clave no es de tipo lista, devuelve false.

El valor devuelto después de la operación lPushx es la longitud de la lista.

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

## lRange

Devuelve una lista de elementos especificados en la lista, utilizando un rango especificado por los desplazamientos START y END. Donde 0 representa el primer elemento de la lista, 1 el segundo elemento, y así sucesivamente. También se pueden usar índices negativos, donde -1 representa el último elemento de la lista, -2 el penúltimo, y así sucesivamente.

Devuelve un array que contiene los elementos especificados en el rango. Si la clave no es de tipo lista, devuelve false.

```php
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## lRem

Remueve los elementos de una lista que son iguales al valor especificado, según el parámetro COUNT.

Los valores posibles para COUNT son:

* count > 0: Comenzará a buscar desde el principio de la lista y eliminará los elementos coincidentes con VALUE en la cantidad de COUNT.
* count < 0: Comenzará a buscar desde el final de la lista y eliminará los elementos coincidentes con VALUE en la cantidad del valor absoluto de COUNT.
* count = 0: Elimina todos los valores coincidentes con VALUE de la lista.

Devuelve la cantidad de elementos eliminados. Devuelve 0 si la lista no existe. Devuelve false si la clave no es de tipo lista.

```php
$redis->lRem('key1', 2, 'A', function ($r) {
    var_dump($r); 
});
```

## lSet

Establece el valor de un elemento en la lista mediante un índice específico.

Devuelve true si tiene éxito, false si el índice está fuera de rango o si la lista está vacía.

```php
$redis->lSet('key1', 0, 'X');
```
## lTrim

Recorta una lista, es decir, mantiene solo los elementos dentro del intervalo especificado y elimina los elementos que están fuera de ese intervalo.

El índice 0 representa el primer elemento de la lista, 1 representa el segundo elemento, y así sucesivamente. También se pueden usar índices negativos, donde -1 representa el último elemento de la lista, -2 representa el penúltimo elemento, y así sucesivamente.

Devuelve true en caso de éxito y false en caso de fallo.

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

Elimina el último elemento de la lista y devuelve el valor eliminado.

Cuando la lista no existe, devuelve null.

```php
$redis->rPop('key1', function ($r) {
    var_dump($r);
});
```

## rPopLPush

Elimina el último elemento de una lista y lo añade a otra lista, devolviendo el elemento eliminado.

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

Inserta uno o varios valores al final de una lista y devuelve la longitud de la lista después de la inserción.

Si la lista no existe, se crea una lista vacía y se realiza la operación RPUSH. Si la lista existe pero no es del tipo lista, devuelve false.

Nota: Antes de la versión 2.4 de Redis, el comando RPUSH solo aceptaba un valor individual.

```php
$redis->del('key1');
$redis->rPush('key1', 'A', function ($r) {
    var_dump($r); // 1
});
```

## rPushX

Inserta un valor al final de una lista existente y devuelve la longitud de la lista. Si la lista no existe, la operación es inválida y devuelve 0. Si la lista existe pero no es del tipo lista, devuelve false.

```php
$redis->del('key1');
$redis->rPushX('key1', 'A', function ($r) {
    var_dump($r); // 0
});
```

## lLen

Devuelve la longitud de una lista. Si la lista 'key' no existe, se interpreta como una lista vacía y devuelve 0. Si 'key' no es del tipo lista, devuelve false.

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

Añade uno o varios elementos miembros a un conjunto. Los miembros existentes en el conjunto son ignorados.

Si el conjunto 'key' no existe, se crea un conjunto con los elementos añadidos. Si 'key' no es del tipo conjunto, devuelve false.

Nota: Antes de la versión 2.4 de Redis, el comando SADD solo aceptaba un miembro.

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

Devuelve la cantidad de elementos en un conjunto. Cuando el conjunto 'key' no existe, devuelve 0.

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

Devuelve los elementos únicos del primer conjunto que no están en otros conjuntos. Los conjuntos inexistentes se consideran como conjuntos vacíos.

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

Almacena la diferencia entre los conjuntos dados en el conjunto especificado. Si el conjunto especificado ya existe, se sobrescribe.

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

Devuelve la intersección de todos los conjuntos dados. Los conjuntos inexistentes se consideran como conjuntos vacíos. Si alguno de los conjuntos está vacío, el resultado también estará vacío.

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

Almacena la intersección de los conjuntos dados en el conjunto especificado y devuelve la cantidad de elementos almacenados. Si el conjunto ya existe, se sobrescribe.

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

Determina si un miembro es parte de un conjunto.

Si el miembro es parte del conjunto, devuelve 1. Si el miembro no es parte del conjunto o si la clave no existe, devuelve 0. Si la clave no es del tipo conjunto, devuelve false.

```php
$redis->sIsMember('key1', 'member1', function ($r) {
    var_dump($r); 
});
```

## sMembers

Devuelve todos los miembros de un conjunto. Si el conjunto no existe, se considera como un conjunto vacío.

```php
$redis->sMembers('s', function ($r) {
    var_dump($r); 
});
```

## sMove

Mueve un miembro específico desde un conjunto de origen a un conjunto de destino.

SMOVE es una operación atómica.

Si el conjunto de origen no existe o no contiene el miembro especificado, SMOVE no hace nada y devuelve 0. Si el miembro se elimina del conjunto de origen y se agrega al conjunto de destino, devuelve 1.

Si el conjunto de origen o destino no es del tipo conjunto, devuelve false.

```php
$redis->sMove('key1', 'key2', 'member13');
```

## sPop

Elimina uno o varios elementos aleatorios de un conjunto y los devuelve. Si el conjunto no existe o está vacío, devuelve null.

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

El comando Srandmember de Redis devuelve un elemento aleatorio de un conjunto.

A partir de la versión 2.6 de Redis, el comando Srandmember acepta un parámetro opcional 'count':
*   Si 'count' es un número positivo menor que el tamaño del conjunto, devuelve un array con 'count' elementos únicos. Si 'count' es mayor o igual al tamaño del conjunto, devuelve el conjunto completo.
*   Si 'count' es un número negativo, devuelve un array con elementos que pueden repetirse y la longitud es el valor absoluto de 'count'.

Esta operación es similar a SPOP, pero SPOP elimina el elemento aleatorio del conjunto y lo devuelve, mientras que Srandmember solo devuelve el elemento aleatorio sin modificar el conjunto.

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

Elimina uno o varios miembros de un conjunto, ignorando los miembros que no existen.

Devuelve la cantidad de elementos eliminados exitosamente, excluyendo los elementos ignorados.

Devuelve false si la clave no es de tipo conjunto.

Antes de la versión 2.4 de Redis, sRem solo aceptaba un valor único de miembro.

```php
$redis->sRem('key1', ['member2', 'member3'], function ($r) {
    var_dump($r); 
});
```

## **sUnion**

Devuelve la unión de los conjuntos dados. Los conjuntos que no existen se consideran como conjuntos vacíos.

```php
$redis->sUnion(['s0', 's1', 's2'], function ($r) {
    var_dump($r); // []
});
```

## **sUnionStore**

Almacena la unión de los conjuntos dados en el conjunto destination especificado, y devuelve la cantidad de elementos. Si destination ya existe, se sobrescribe.

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

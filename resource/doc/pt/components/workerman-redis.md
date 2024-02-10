# workerman-redis

## Introdução

workeman/redis é um componente de redis assíncrono baseado em workerman.

> **Nota**
> O objetivo principal deste projeto é implementar a assinatura assíncrona do redis (subscribe, pSubscribe).
> Como o redis é rápido o suficiente, a menos que seja necessário assinaturas assíncronas psubscribe subscribe, não é necessário usar este cliente assíncrono. A extensão redis oferecerá um desempenho melhor.

## Instalação:

```php
composer require workerman/redis
```

## Uso de Callback

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

## Uso de Corotina

> **Nota**
> O uso de corotina requer workerman>=5.0, workerman/redis>=2.0.0 e a instalação com composer require revolt/event-loop ^1.0.0

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

Quando não há função de retorno de chamada definida, o cliente retornará o resultado do pedido assíncrono de forma síncrona, sem bloquear o processo atual, permitindo o processamento concorrente dos pedidos.

> **Nota**
> psubscribe subscribe não suporta o uso de corotina.

# Documentação

**Explicação**

**No método de retorno de chamada, geralmente há 2 parâmetros na função de retorno ($result, $redis), onde `$result` é o resultado e `$redis` é a instância do redis. Por exemplo:**
```php
use Workerman\Redis\Client;
$redis = new Client('redis://127.0.0.1:6379');
// Definir a função de retorno para verificar o resultado do comando set
$redis->set('key', 'value', function ($result, $redis) {
    var_dump($result); // true
});
// As funções de retorno são opcionais, então é omitida aqui
$redis->set('key1', 'value1');
// Funções de retorno podem ser aninhadas
$redis->get('key', function ($result, $redis){
    $redis->set('key2', 'value2', function ($result) {
        var_dump($result);
    });
});
```

## **Conexão**
```php
use Workerman\Redis\Client;
// Omitir função de retorno
$redis = new Client('redis://127.0.0.1:6379');
// Com função de retorno
$redis = new Client('redis://127.0.0.1:6379', [
    'connect_timeout' => 10 // Definir um tempo de conexão de 10 segundos, o padrão é 5 segundos se não estiver definido
], function ($success, $redis) {
    // Função de retorno da conexão
    if (!$success) echo $redis->error();
});
```

## **auth**
```php
// Autenticação de senha
$redis->auth('senha', function ($result) {

});
// Autenticação de nome de usuário e senha
$redis->auth('nome de usuário', 'senha', function ($result) {

});
```

## **pSubscribe**

Subscreve a um ou mais canais que correspondem a um padrão específico.

Cada padrão usa '*' como curinga, por exemplo, it\* corresponde a todos os canais que começam com "it" (como it.news, it.blog, it.tweets, etc). news.* corresponde a todos os canais que começam com "news." (como news.it, news.global.today, etc), e assim por diante.

Observe: a função de retorno de pSubscribe possui 4 parâmetros ($pattern, $channel, $message, $redis).

Após chamar os métodos pSubscribe ou subscribe da instância $redis, a chamada de outros métodos na mesma instância será ignorada.

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

Usado para assinar um ou mais canais específicos.

Observação: a função de retorno de subscribe possui 3 parâmetros ($channel, $message, $redis).

Após chamar os métodos pSubscribe ou subscribe da instância $redis, a chamada de outros métodos na mesma instância será ignorada.

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

Usado para enviar informações para um canal específico.

Retorna o número de assinantes que receberam a mensagem.

```php
$redis2->publish('news', 'news content');
```

## **select**
```php
// Omitir função de retorno
$redis->select(2);
$redis->select('test', function ($result, $redis) {
    // O parâmetro select deve ser numérico, então $result será false aqui
    var_dump($result, $redis->error());
});
```

## **get**

Comando usado para obter o valor de uma chave específica. Se a chave não existir, retorna NULL. Se o valor armazenado na chave não for do tipo string, retorna false.
```php
$redis->get('key', function($result) {
     // Se a chave não existir, retorna NULL, se houver um erro, retorna false
    var_dump($result);
});
```

## **set**

Usado para definir o valor de uma chave específica. Se a chave já tiver um valor, o SET sobrescreverá o valor antigo, ignorando o tipo.
```php
$redis->set('key', 'value');
$redis->set('key', 'value', function($result){});
// O terceiro parâmetro pode levar um tempo de expiração, expira em 10 segundos
$redis->set('key','value', 10);
$redis->set('key','value', 10, function($result){});
```

## **setEx, pSetEx**

Define o valor de uma chave com seu tempo de expiração. Se a chave já existir, o comando SETEX substituirá o valor antigo.

```php
// Observe que o segundo parâmetro é o tempo de expiração em segundos
$redis->setEx('key', 3600, 'value'); 
// pSetEx com tempo em milissegundos
$redis->pSetEx('key', 3600, 'value');
```

## **del**

Usado para excluir chaves existentes. O resultado é o número de chaves excluídas (as chaves inexistentes não são contadas).
```php
// Excluir uma chave
$redis->del('key');
// Excluir várias chaves
$redis->del(['key', 'key1', 'key2']);
```

## **setNx**

(Set If Not Exists) usado para definir um valor para uma chave que não existe.
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

Usado para verificar se uma chave específica existe. O resultado é o número de chaves existentes.
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
## incr, incrBy

Incrementa o valor de uma chave específica em 1 ou pelo valor especificado. Se a chave não existir, ela será inicializada como 0 e a operação de incremento / incremento será executada.
Se o valor for de um tipo incorreto ou se uma string não puder ser representada como número, retorna false. Retorna o valor após o aumento com sucesso.
```php
$redis->incr('key1', function ($result) {
    var_dump($result);
}); 
$redis->incrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## incrByFloat

Adiciona o valor flutuante especificado à chave. Se a chave não existir, a operação assume que o valor inicial é 0. Se o valor for de um tipo incorreto ou se uma string não puder ser representada como número, retorna false. Retorna o valor após o aumento com sucesso.
```php
$redis->incrByFloat('key1', 1.5, function ($result) {
    var_dump($result);
}); 
```

## decr, decrBy

Diminui o valor da chave em 1 ou pelo valor especificado. Se a chave não existir, ela será inicializada como 0 e a operação de decremento / decremento será executada.
Se o valor for de um tipo incorreto ou se uma string não puder ser representada como número, retorna false. Retorna o valor após a redução com sucesso.
```php
$redis->decr('key1', function ($result) {
    var_dump($result);
}); 
$redis->decrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## mGet

Retorna todos (um ou mais) valores das chaves especificadas. Se alguma chave não existir, retornará NULL para essa chave.
```php
$redis->set('key1', 'value1');
$redis->set('key2', 'value2');
$redis->set('key3', 'value3');
$redis->mGet(['key0', 'key1', 'key5'], function ($result) {
    var_dump($result); // [null, 'value1', null];
}); 
```
## **getSet**

Usado para definir o valor de uma chave especificada e retornar o valor antigo da chave.

```php
$redis->set('x', '42');
$redis->getSet('x', 'lol', function ($result) {
    var_dump($result); // '42'
});
$redis->get('x', function ($result) {
    var_dump($result); // 'lol'
});
```

## **randomKey**

Retorna uma chave aleatória do banco de dados atual.
```php
$redis->randomKey(function($key) use ($redis) {
    $redis->get($key, function ($result) {
        var_dump($result); 
    });
});
```

## **move**

Move a chave do banco de dados atual para o banco de dados especificado.
```php
$redis->select(0); // mudar para o DB 0
$redis->set('x', '42'); // escrever 42 em x
$redis->move('x', 1, function ($result) { // mover para o DB 1
    var_dump($result); // 1
});
$redis->select(1); // mudar para o DB 1
$redis->get('x', function ($result) {
    var_dump($result); // '42'
});
```

## **rename**

Renomeia uma chave, retornando false se a chave não existe.
```php
$redis->set('x', '42');
$redis->rename('x', 'y', function ($result) {
    var_dump($result); // true
});
```

## **renameNx**

Renomeia uma chave somente se a nova chave não existir.
```php
$redis->del('y');
$redis->set('x', '42');
$redis->renameNx('x', 'y', function ($result) {
    var_dump($result); // 1
});
```

## **expire**

Define o tempo de expiração de uma chave em segundos. Retorna 1 em caso de sucesso, 0 se a chave não existe e falso em caso de erro.
```php
$redis->set('x', '42');
$redis->expire('x', 3);
```

## **keys**

Usado para encontrar todas as chaves que correspondem ao padrão especificado.
```php
$redis->keys('*', function ($keys) {
    var_dump($keys);
});
$redis->keys('user*', function ($keys) {
    var_dump($keys);
});
```

## **type**

Retorna o tipo de valor armazenado em uma chave, que pode ser string, set, list, zset, hash ou none (caso a chave não exista).
```php
$redis->type('key', function ($result) {
    var_dump($result); // string set list zset hash none
});
```

## **append**

Se a chave existir e for uma string, APPEND acrescenta o valor à chave existente e retorna o comprimento da string resultante.

Se a chave não existir, APPEND cria a chave com o valor especificado e retorna o comprimento da string.
Se a chave existir, mas não for uma string, retorna false.
```php
$redis->set('key', 'value1');
$redis->append('key', 'value2', function ($result) {
    var_dump($result); // 12
});
$redis->get('key', function ($result) {
    var_dump($result); // 'value1value2'
});
```

## **getRange**

Obtém uma subcadeia de uma string armazenada em uma chave. Retorna uma string vazia se a chave não existir e false se a chave não for uma string.
```php
$redis->set('key', 'string value');
$redis->getRange('key', 0, 5, function ($result) {
    var_dump($result); // 'string'
});
$redis->getRange('key', -5, -1 , function ($result) {
    var_dump($result); // 'value'
});
```

## **setRange**

Substitui uma parte de uma string por outra a partir da posição especificada. Se a chave não existir, ela é criada e definida com a string especificada.
Retorna o comprimento da string após a modificação.
```php
$redis->set('key', 'Hello world');
$redis->setRange('key', 6, "redis", function ($result) {
    var_dump($result); // 11
});
$redis->get('key', function ($result) {
    var_dump($result); // 'Hello redis'
});
```

## **strLen**

Retorna o comprimento da string armazenada em uma chave. Retorna false se a chave não armazenar uma string.
```php
$redis->set('key', 'value');
$redis->strlen('key', function ($result) {
    var_dump($result); // 5
});
```

## **getBit**

Retorna o valor de um bit em uma posição específica de uma string armazenada em uma chave.
```php
$redis->set('key', "\x7f"); // isso é 0111 1111
$redis->getBit('key', 0, function ($result) {
    var_dump($result); // 0
});
```

## **setBit**

Define ou limpa o valor de um bit em uma posição específica de uma string armazenada em uma chave. Retorna 0 ou 1, representando o valor antes da modificação.
```php
$redis->set('key', "*"); // ord("*") = 42 = 0x2f = "0010 1010"
$redis->setBit('key', 5, 1, function ($result) {
    var_dump($result); // 0
});
```

## **bitOp**

Realiza operações bitwise em uma ou mais chaves e armazena o resultado em uma chave de destino.
As operações suportadas são **AND**, **OR**, **XOR** e **NOT**.
Retorna o tamanho da string armazenada na chave de destino, que é igual ao tamanho da string mais longa de entrada.
```php
$redis->set('key1', "abc");
$redis->bitOp( 'AND', 'dst', 'key1', 'key2', function ($result) {
    var_dump($result); // 3
});
```

## **bitCount**

Calcula o número de bits setados (população) em uma string.
Por padrão, verifica todos os bytes na string. A contagem só pode ser especificada passando os parâmetros adicionais *inicio* e *fim*.
Assim como no comando GETRANGE, *inicio* e *fim* podem conter valores negativos para indexar bytes a partir do final da string, onde -1 é o último byte, -2 é o penúltimo byte, e assim por diante.
Retorna o número de bits com valor 1 na string.
Chaves inexistentes são tratadas como strings vazias, portanto o comando retornará zero.
```php
$redis->set('key', 'hello');
$redis->bitCount( 'key', 0, 0, function ($result) {
    var_dump($result); // 3
});
$redis->bitCount( 'key', function ($result) {
    var_dump($result); // 21
});
```

## **sort**

A ordenação pode ser realizada nos elementos de listas, conjuntos e conjuntos ordenados.
O protótipo é `sort($key, $options, $callback)`.

O parâmetro de opções é um array que pode conter os seguintes valores:
~~~
$options = [
    'by' => 'some_pattern_*',
    'limit' => [0, 1],
    'get' => 'some_other_pattern_*', // or an array of patterns
    'sort' => 'asc', // or 'desc'
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

Retorna o tempo restante para expirar uma chave, em segundos ou milissegundos.

Se a chave não tiver tempo para viver, retorna -1. Se a chave não existir, retorna -2.
```php
$redis->set('key', 'value', 10);
// em segundos
$redis->ttl('key', function ($result) {
    var_dump($result); // 10
});
// em milissegundos
$redis->pttl('key', function ($result) {
    var_dump($result); // 9999
});
// chave não existe
$redis->pttl('key-not-exists', function ($result) {
    var_dump($result); // -2
});
```

## **persist**

Remove o tempo de vida de uma chave, tornando-a com tempo de vida indefinido.

Retorna 1 se o tempo de vida for removido com sucesso, 0 se a chave não existir ou não tiver tempo de vida, e false em caso de erro.
```php
$redis->persist('key');
```

## **mSet, mSetNx**

Define diversos pares chave-valor em um único comando atômico. mSetNx apenas retorna 1 se todos os pares de chaves forem definidos.
Retorna 1 em caso de sucesso, 0 em caso de falha e false em caso de erro.
```php
$redis->mSet(['key0' => 'value0', 'key1' => 'value1']);
```

## **hSet**

Atribui um valor a um campo específico no hash.

Se o campo é novo e foi atribuido com sucesso, retorna 1. Se o campo já existir no hash e o valor antigo foi sobrescrito com sucesso, retorna 0.
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

Atribui um valor a um campo no hash somente se o campo não existir.

Se o hash não existir, um novo hash é criado e a operação é realizada.
Se o campo já existir no hash, a operação é inválida.

Se a chave não existir, um novo hash é criado e o comando HSETNX é executado.
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

Retorna o valor de um campo específico no hash.

Se o campo ou a chave não existirem, retorna null.
```php
$redis->hGet('h', 'key1', function ($result) {
    var_dump($result);
});
```
## **hLen**

Usado para obter a quantidade de campos em uma tabela hash.

Retorna 0 se a chave não existir.

```php
$redis->del('h');
$redis->hSet('h', 'key1', 'hello');
$redis->hSet('h', 'key2', 'plop');
$redis->hLen('h', function ($result) {
    var_dump($result); // 2
});
```

## **hDel**

Comando usado para excluir um ou mais campos específicos da tabela hash. Os campos inexistentes serão ignorados.

Retorna a quantidade de campos excluídos com sucesso, sem contar os campos ignorados. Retorna false se a chave não for uma hash.

```php
$redis->hDel('h', 'key1');
```

## **hKeys**

Obtém todos os campos da tabela hash em um array.

Retorna um array vazio se a chave não existir ou false se a chave não for uma hash.

```php
$redis->hKeys('key', function ($result) {
    var_dump($result);
});
```

## **hVals**

Retorna os valores de todos os campos da tabela hash em um array.

Retorna um array vazio se a chave não existir ou false se a chave não for uma hash.

```php
$redis->hVals('key', function ($result) {
    var_dump($result);
});
```

## **hGetAll**

Retorna um array associativo contendo todos os campos e valores da tabela hash.

Retorna um array vazio se a chave não existir, ou false se a chave não for do tipo hash.

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
Retorno
```php
array (
   'a' => 'x',
   'b' => 'y',
   'c' => 'z',
   'd' => 't',
)
```

## **hExists**

Verifica se um campo específico existe na tabela hash. Retorna 1 se existir, 0 se o campo ou a chave não existirem, e false em caso de erro.

```php
$redis->hExists('h', 'a', function ($result) {
    var_dump($result); //
});
```

## **hIncrBy**

Usado para incrementar o valor de um campo na tabela hash por um valor especificado. O incremento pode ser um número negativo para subtrair do campo.

Se a chave da hash não existir, uma nova hash será criada e o comando HINCRBY será executado.

Se o campo especificado não existir, o valor do campo será inicializado como 0 antes da execução do comando.

Ao executar o comando HINCRBY em um campo que armazena um valor de string, ele retorna false.

O valor resultante é limitado a um número inteiro de 64 bits.

```php
$redis->del('h');
$redis->hIncrBy('h', 'x', 2, function ($result) {
    var_dump($result);
});
```

## **hIncrByFloat**

Similar ao hIncrBy, mas o valor do incremento é do tipo float.

## **hMSet**

Define múltiplos pares campo-valor na tabela hash.

Este comando substituirá os campos existentes na tabela hash.

Se a tabela hash não existir, uma nova tabela hash será criada e o comando HINCRBY será executado.

```php
$redis->del('h');
$redis->hMSet('h', ['name' => 'Joe', 'sex' => 1])
```

## **hMGet**

Retorna um array associativo contendo os valores de um ou mais campos especificados da tabela hash.

Se o campo especificado não existir na tabela hash, o valor correspondente no array será null. Se a chave não for do tipo hash, retorna false.

```php
$redis->del('h');
$redis->hSet('h', 'field1', 'value1');
$redis->hSet('h', 'field2', 'value2');
$redis->hMGet('h', ['field1', 'field2', 'field3'], function ($r) {
    var_export($r);
});
```
Saída
```php
array (
  'field1' => 'value1',
  'field2' => 'value2',
  'field3' => null
)
```
## **blPop, brPop**

Remove e obtém o primeiro/último elemento da lista, bloqueando se a lista estiver vazia até que um elemento possa ser removido ou um tempo limite seja atingido.

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

Remove o último elemento da lista e o insere no início de outra lista; bloqueia se a lista estiver vazia até que um elemento possa ser removido ou um tempo limite seja atingido. Retorna null se houver um tempo limite.

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

Obtém o elemento da lista por índice. Índices negativos podem ser usados, -1 representa o último elemento da lista, -2 o penúltimo, e assim por diante.

Retorna null se o índice estiver fora do intervalo da lista ou false se a chave não for uma lista.

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lindex('key1', 0, function ($r) {
    var_dump($r); // A
});
```

## **lInsert**

Insere um elemento antes ou depois de um elemento específico na lista. Não faz nada se o elemento especificado não existir na lista ou se a lista não existir.

Retorna 0 se não realizar nenhuma inserção, o número de elementos na lista após a inserção em outros casos. Retorna false se a chave não for do tipo lista.

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

Remove e retorna o primeiro elemento da lista.

Retorna null se a lista key não existir.

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lPop('key1', function ($r) {
    var_dump($r); // A
});
```

## **lPush**

Insere um ou mais valores no início da lista. Se a chave não existir, uma lista vazia será criada e o comando será executado. Retorna false se a chave existir mas não for do tipo lista.

**Nota:** Antes da versão 2.4 do Redis, o comando LPUSH só aceitava um único valor.

```php
$redis->del('key1');
$redis->lPush('key1', 'A');
$redis->lPush('key1', ['B','C']);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lPushx**

Insere um valor no início de uma lista existente, operação inválida se a lista não existir. Retorna 0 se a operação não for realizada, o comprimento da lista após a inserção em outros casos. Retorna false se a chave não for do tipo lista.

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

Retorna os elementos da lista dentro do intervalo especificado pelos índices START e END. O índice 0 representa o primeiro elemento, 1 o segundo e assim por diante. Índices negativos podem ser usados, -1 representa o último elemento da lista, -2 o penúltimo, e assim por diante.

Retorna um array contendo os elementos dentro do intervalo especificado. Retorna false se a chave não for do tipo lista.

```php
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lRem**

Remove os elementos da lista que são iguais ao valor especificado. A quantidade de elementos removidos depende do parâmetro COUNT. Se a contagem for 0, todos os elementos com o valor especificado serão removidos. 

Retorna a quantidade de elementos removidos. Se a lista não existe, retorna 0. Retorna false se a chave não for do tipo lista.

```php
$redis->lRem('key1', 2, 'A', function ($r) {
    var_dump($r); 
});
```

## **lSet**

Define o valor de um elemento na lista por um índice específico. Retorna true em caso de sucesso, retorna false se o índice estiver fora do intervalo da lista ou se a lista estiver vazia.

```php
$redis->lSet('key1', 0, 'X');
```
## **lTrim**

Trim sobre uma lista significa que apenas os elementos dentro do intervalo especificado serão mantidos na lista, e os elementos fora desse intervalo serão excluídos.

O índice 0 representa o primeiro elemento da lista, 1 representa o segundo elemento e assim por diante. Você também pode usar índices negativos, onde -1 indica o último elemento da lista, -2 indica o penúltimo elemento e assim por diante.

Retorna true em caso de sucesso e false em caso de falha.

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

Usado para remover o último elemento de uma lista e retorna o valor removido.

Quando a lista não existe, retorna null.

```php
$redis->rPop('key1', function ($r) {
    var_dump($r);
});
```

## **rPopLPush**

Remove o último elemento de uma lista e o adiciona a outra lista, retornando o valor removido.

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

Insere um ou mais valores no final de uma lista e retorna o comprimento da lista após a inserção.

Se a lista não existir, uma lista vazia será criada e a operação RPUSH será executada. Se a lista existir, mas não for do tipo lista, retorna false.

**Observação:** Na versão do Redis anterior à 2.4, o comando RPUSH aceitava apenas um valor.

```php
$redis->del('key1');
$redis->rPush('key1', 'A', function ($r) {
    var_dump($r); // 1
});
```

## **rPushX**

Insere um valor no final de uma lista existente e retorna o comprimento da lista. Se a lista não existir, a operação falha e retorna 0. Se a lista existir, mas não for do tipo lista, retorna false.

```php
$redis->del('key1');
$redis->rPushX('key1', 'A', function ($r) {
    var_dump($r); // 0
});
```

## **lLen**

Retorna o comprimento da lista. Se a chave da lista não existir, é considerada como uma lista vazia e retorna 0. Se a chave não for do tipo lista, retorna false.

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

Adiciona um ou mais membros a um conjunto, ignorando os membros que já estão presentes.

Se a chave do conjunto não existir, cria um conjunto com os membros adicionados.

Se a chave do conjunto não for do tipo conjunto, retorna false.

**Observação:** Na versão do Redis anterior à 2.4, o comando SADD aceitava apenas um membro por vez.

```php
$redis->del('key1');
$redis->sAdd('key1', 'member1');
$redis->sAdd('key1', ['member2', 'member3'], function ($r) {
    var_dump($r); // 2
});
$redis->sAdd('key1', 'member2', function ($r) {
    var_dump($r); // 0
});
```

## **sCard**

Retorna a quantidade de elementos do conjunto. Se a chave do conjunto não existir, retorna 0.

```php
$redis->del('key1');
$redis->sAdd('key1', 'member1');
$redis->sAdd('key1', 'member2');
$redis->sAdd('key1', 'member3');
$redis->sCard('key1', function ($r) {
    var_dump($r); // 3
});
$redis->sCard('keyX', function ($r) {
    var_dump($r); // 0
});
```

## **sDiff**

Retorna os elementos que estão presente no primeiro conjunto e não em outros conjuntos, ou seja, os elementos que são exclusivos do primeiro conjunto. Os conjuntos ausentes são tratados como conjuntos vazios.

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

Armazena a diferença entre os conjuntos fornecidos em um novo conjunto e retorna a quantidade de elementos armazenados.

Se o conjunto especificado já existir, ele será substituído.

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

Retorna a interseção de todos os conjuntos fornecidos. Os conjuntos ausentes são tratados como conjuntos vazios.

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

Armazena a interseção dos conjuntos fornecidos em um novo conjunto e retorna a quantidade de elementos armazenados.

Se o conjunto especificado já existir, ele será substituído.

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

Verifica se o membro dado pertence ao conjunto especificado.

Retorna 1 se o membro pertence ao conjunto, 0 se não pertence ou se a chave não existir, e false se a chave não for do tipo conjunto.

```php
$redis->sIsMember('key1', 'member1', function ($r) {
    var_dump($r); 
});
```

## **sMembers**

Retorna todos os membros do conjunto. Se a chave do conjunto não existir, retorna um conjunto vazio. 

```php
$redis->sMembers('s', function ($r) {
    var_dump($r); 
});
```

## **sMove**

Move um membro de um conjunto de origem para um conjunto de destino.

SMOVE é uma operação atômica. Se o conjunto de origem não existir ou não contiver o membro especificado, a operação SMOVE não é executada e retorna 0. Caso contrário, o membro é removido do conjunto de origem e adicionado ao conjunto de destino.

Se o conjunto de origem ou de destino não for do tipo conjunto, retorna false.

```php
$redis->sMove('key1', 'key2', 'member13');
```

## **sPop**

Remove um ou mais elementos aleatórios do conjunto especificado e retorna os elementos removidos.

Se o conjunto não existir ou estiver vazio, retorna null.

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

O comando Srandmember do Redis é usado para retornar um elemento aleatório do conjunto.

A partir da versão 2.6 do Redis, o comando Srandmember aceita um parâmetro count opcional:

*   Se count for um número positivo e menor que o número de membros do conjunto, o comando retornará um array de count membros, onde cada membro é exclusivo. Se count for maior ou igual ao número de membros do conjunto, o comando retornará todo o conjunto.
*   Se count for um número negativo, o comando retornará um array onde os membros podem ser repetidos várias vezes, com o comprimento do array sendo o valor absoluto de count.

Essa operação é semelhante ao SPOP, mas enquanto o SPOP remove e retorna um elemento aleatório do conjunto, o Srandmember apenas retorna o elemento aleatório, sem alterar o conjunto.

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

Remove um ou mais membros de um conjunto, os membros que não existem serão ignorados.

Retorna a quantidade de membros removidos com sucesso, excluindo os membros ignorados.

Retorna falso se a chave não for do tipo conjunto.

Antes da versão 2.4 do Redis, o comando SREM aceitava apenas um valor de membro.

```php
$redis->sRem('chave1', ['membro2', 'membro3'], function ($r) {
    var_dump($r); 
});
```

## **sUnion**

O comando retorna a união dos conjuntos fornecidos. Os conjuntos que não existem são tratados como conjuntos vazios.

```php
$redis->sUnion(['s0', 's1', 's2'], function ($r) {
    var_dump($r); // []
});
```

## **sUnionStore**

Armazena a união dos conjuntos fornecidos no conjunto de destino especificado e retorna a quantidade de elementos. Se o conjunto de destino já existir, ele será substituído.

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

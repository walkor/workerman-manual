# increment
**``` (Requires Workerman version >= 3.3.0) ```**
```php
bool \GlobalData\Client::increment(string $key[, int $step = 1])
```
Incremento atômico. Aumenta o valor de um elemento numérico pelo tamanho especificado pelo parâmetro step. Se o valor do elemento não for do tipo numérico, ele será tratado como 0 antes de ser aumentado. Retorna falso se o elemento não existir.

## Parâmetros

 ``` $key ```

Chave do valor. (Por exemplo, ```$global->abc```, ```abc``` é a chave)

 ``` $value ```

O tamanho pelo qual o valor do elemento será aumentado.

## Valor retornado
Retorna true se tiver sucesso, caso contrário, retorna false.


## Exemplo

```php
$global = new GlobalData\Client('127.0.0.1:2207');

$global->some_key = 0;

// Incremento não-atômico
$global->some_key++;

echo $global->some_key."\n";

// Incremento atômico
$global->increment('some_key');

echo $global->some_key."\n";
```

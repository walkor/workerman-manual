# adicionar
**``` (requer versão Workerman >=3.3.0) ```**
```php
bool \GlobalData\Client::add(string $key, mixed $value)
```
Adiciona atomicamente. Se a chave já existe, retornará falso.

## Parâmetros

 ``` $key ```

Chave. (Por exemplo, ```$global->abc```, ```abc``` é a chave)

 ``` $value ```

Valor a ser armazenado.

## Valor de retorno
Retorna true se for bem-sucedido, caso contrário retorna falso.


## Exemplo

```php
$global = new GlobalData\Client('127.0.0.1:2207');

if($global->add('alguma_chave', 10))
{
    // O valor de $global->alguma_chave foi atribuído com sucesso
    echo "adicionar sucesso " , $global->alguma_chave;
}
else
{
    // $global->alguma_chave já existe, falha ao atribuir valor
    echo "adicionar falha ", var_export($global->alguma_chave);
}
```

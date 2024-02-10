# cas
**``` (Requires Workerman version >= 3.3.0) ```**
```php
bool \GlobalData\Client::cas(string $key, mixed $old_value, mixed $new_value)
```
Substituição atômica, substitua ```$old_value``` por ```$new_value```.
Apenas se o valor correspondente a essa chave não foi modificado por outro cliente desde a última vez que foi acessado pelo cliente atual, o valor pode ser escrito.

## Parâmetros

``` $key ```

Chave. (por exemplo, se for ```$global->abc```, ```abc``` é a chave)

``` $old_value ```

Dado antigo

``` $new_value ```

Novo dado

## Valor de Retorno
Retorna true se a substituição for bem-sucedida, caso contrário retorna false.

## Observação:

Quando vários processos operam simultaneamente em uma variável compartilhada, às vezes é necessário considerar problemas de concorrência.

Por exemplo, os processos A e B estão adicionando um membro à lista de usuários ao mesmo tempo.
Os processos A e B têm a variável de lista de usuários atual como ```$global->user_list = array(1,2,3)```.
O processo A está manipulando a variável ```$global->user_list```, adicionando um usuário 4.
O processo B está manipulando a variável ```$global->user_list```, adicionando um usuário 5.
O processo A configura a variável como ```$global->user_list = array(1,2,3,4)``` com sucesso.
O processo B configura a variável como ```$global->user_list = array(1,2,3,5)``` com sucesso.
Neste momento, a variável configurada pelo processo B substitui a variável configurada pelo processo A, resultando na perda de dados.

Isso ocorre devido a leitura e escrita não serem uma operação atômica, resultando em problemas de concorrência.
Para resolver esse tipo de problema de concorrência, pode-se usar a interface de substituição atômica (cas).
Antes de alterar um valor, a interface cas verifica se o valor foi alterado por outro processo com base em ```$old_value```. Se foi alterado, a substituição não é realizada e retorna false. Caso contrário, retorna true.
Veja o exemplo abaixo.

**Observação:**
Alguns dados compartilhados sendo substituídos por concorrência não são um problema, como por exemplo, o maior lance atual em um sistema de leilão, ou o estoque atual de um determinado produto.

## Exemplo

```php
$global = new GlobalData\Client('127.0.0.1:2207');

// Inicializar a lista
$global->user_list = array(1,2,3);

// Adiciona um valor atômico à user_list
do
{
    $old_value = $new_value = $global->user_list;
    $new_value[] = 4;
}
while(!$global->cas('user_list', $old_value, $new_value));

var_export($global->user_list);
```

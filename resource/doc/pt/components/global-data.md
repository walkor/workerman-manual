# Componente de Compartilhamento de Variáveis GlobalData
**``` (Requer Workerman versão >= 3.3.0) ```**

Código fonte: https://github.com/walkor/GlobalData

## Nota
GlobalData requer Workerman versão >= 3.3.0

## Instalação

`composer require workerman/globaldata`

## Princípio

Usando os métodos mágicos de PHP ```__set __get __isset __unset``` para acionar a comunicação com o servidor GlobalData, as variáveis reais são armazenadas no servidor GlobalData. Por exemplo, ao definir um atributo inexistente em uma classe de cliente, o método mágico ```__set``` é acionado, onde a classe do cliente envia uma solicitação para o servidor GlobalData para armazenar uma variável. Quando se acessa uma variável inexistente na classe do cliente, o método ```__get``` da classe é acionado e o cliente envia uma solicitação para o servidor GlobalData para ler esse valor, completando assim o compartilhamento de variáveis entre processos.

```php
require_once __DIR__ . '/vendor/autoload.php';

// Conectar ao servidor GlobalData
$global = new GlobalData\Client('127.0.0.1:2207');

// Acionar $global->__isset('somedata') para verificar se o servidor armazena um valor com a chave somedata
isset($global->somedata);

// Acionar $global->__set('somedata',array(1,2,3)) para notificar o servidor armazenar o valor correspondente a somedata como array(1,2,3)
$global->somedata = array(1,2,3);

// Acionar $global->__get('somedata') para consultar o valor correspondente a somedata no servidor
var_export($global->somedata);

// Acionar $global->__unset('somedata') para notificar o servidor a excluir somedata e o valor correspondente
unset($global->somedata);
```

# Princípio de reinicialização suave
## O que é uma reinicialização suave?

A reinicialização suave difere de uma reinicialização normal, pois permite reiniciar o serviço (geralmente referindo-se a negócios de conexão curta) sem impacto aos usuários, a fim de recarregar o programa PHP e concluir a atualização do código empresarial.

A reinicialização suave é normalmente aplicada durante a atualização do negócio ou durante o processo de lançamento da versão, e pode evitar os impactos temporários de indisponibilidade do serviço causados por reinicializações de serviço devido a atualizações de código.

> **Nota**
> O sistema Windows não suporta recarga.

> **Nota**
> Para negócios de conexão longa (por exemplo, WebSocket), as conexões serão desconectadas durante a reinicialização do processo. A solução é usar uma arquitetura semelhante à [gatewayWorker](https://www.workerman.net/doc/gateway-worker), onde um grupo de processos é dedicado à manutenção da conexão e a propriedade [reloadable](../worker/reloadable.md) desse grupo de processos é definida como falso. A lógica empresarial é iniciada por outro grupo de processos de trabalhadores, e a comunicação entre o gateway e os processos de trabalhadores é realizada por meio de TCP. Quando houver necessidade de alterações no negócio, basta reiniciar os processos de trabalhadores.

## Limitações
**Nota: Somente os arquivos carregados em callbacks on{...} serão atualizados após uma reinicialização suave; arquivos carregados diretamente no script de inicialização ou código fixo não serão atualizados.**

#### O seguinte código não será atualizado após uma recarga
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onMessage = function($connection, $request) {
    $connection->send('hi'); // O código fixo não é suportado para atualização
};
``` 

```php
$worker = new Worker('http://0.0.0.0:1234');
require_once __DIR__ . '/seu/caminho/MessageHandler.php'; // Os arquivos carregados diretamente no script de inicialização não suportam atualizações
$messageHandler = new MessageHandler();
$worker->onMessage = [$messageHandler, 'onMessage']; // Supondo que a classe MessageHandler tenha um método onMessage
```

#### O seguinte código será atualizado após uma recarga
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onWorkerStart = function($worker) { // onWorkerStart é o callback acionado após o lançamento do processo
    require_once __DIR__ . '/seu/caminho/MessageHandler.php'; // Os arquivos carregados após a inicialização do processo suportam atualizações
    $messageHandler = new MessageHandler();
    $worker->onMessage = [$messageHandler, 'onMessage'];
};
```
Após uma alteração no MessageHandler.php, executando `php start.php reload`, o MessageHandler.php será recarregado na memória para atualizar a lógica empresarial.

> **Dica**
> O código acima utiliza a declaração `require_once` para fins de demonstração, mas se o seu projeto suportar o carregamento automático por PSR-4, a declaração `require_once` não é necessária.

## Princípio de reinicialização suave

O WorkerMan consiste em um processo principal e processos secundários. O processo principal monitora os processos secundários, que são responsáveis por receber conexões de clientes e processar dados recebidos dos clientes, realizando o devido processamento e retornando os dados aos clientes. Ao atualizar o código empresarial, apenas precisamos atualizar os processos secundários para alcançar o objetivo da atualização do código.

Quando o processo principal do WorkerMan recebe um sinal de reinicialização suave, ele envia um sinal de encerramento seguro (permitindo que o processo em questão complete o processamento da solicitação atual antes de sair) para um dos processos secundários. Após a saída deste processo, o processo principal cria um novo processo secundário (carregando o novo código PHP), e então o processo principal envia o comando de parar para outro processo antigo, reiniciando assim um processo de cada vez, até que todos os processos antigos sejam substituídos.

Vemos que a reinicialização suave, na verdade, faz com que os processos de negócios antigos saiam um por um e crie novos processos para substituí-los. Para garantir que os usuários não sejam afetados durante a reinicialização suave, é necessário garantir que os processos não mantenham informações de estado relacionadas aos usuários, ou seja, é preferível que os processos de negócios sejam sem estado, evitando a perda de informações devido à saída dos processos.

# send
## Descrição:
```php
mixed Connection::send(mixed $data [,$raw = false])
```

Envia dados para o cliente

## Parâmetros

``` $data ```

Os dados a serem enviados. Se um protocolo for especificado ao inicializar a classe Worker, o método de codificação do protocolo será automaticamente chamado para empacotar os dados de acordo com o protocolo antes de enviar para o cliente.

``` $raw ```

Se deve enviar os dados brutos, ou seja, sem chamar o método de codificação do protocolo. O padrão é falso, o que significa que o método de codificação do protocolo será chamado automaticamente.

## Valor de Retorno

true - Indica que os dados foram gravados com sucesso no buffer de envio do socket da conexão.

null - Indica que os dados foram gravados no buffer de envio do nível de aplicação da conexão e aguardam a escrita no buffer de envio do socket do sistema.

false - Indica que a operação de envio falhou. Isso pode ocorrer se a conexão do cliente estiver fechada ou se o buffer de envio do nível de aplicação da conexão estiver cheio.

## Observações
O retorno de ```true``` apenas significa que os dados foram gravados com sucesso no buffer de envio do socket da conexão e não significa que os dados foram enviados com sucesso para o buffer de recebimento do socket de destino, muito menos significa que a aplicação do destino pegou os dados do buffer de recebimento do socket local. **No entanto, mesmo que isso seja verdade, desde que o send não retorne false e a rede não esteja desconectada, e o cliente esteja recebendo normalmente, pode-se dizer que os dados podem ser entregues ao destino.**

Devido ao fato de que os dados no buffer de envio do socket são enviados de forma assíncrona para o destino pelo sistema operacional, o sistema operacional não fornece um mecanismo de confirmação correspondente para a camada de aplicação, então **a camada de aplicação** não pode saber quando os dados no buffer de envio do socket começarão a ser enviados e **a camada de aplicação** tampouco pode saber se os dados no buffer de envio do socket foram enviados com sucesso. Devido a essas razões, o workerman não pode fornecer diretamente uma interface de confirmação de mensagens.

Se o aplicativo precisar garantir que cada mensagem seja recebida pelo cliente, pode-se adicionar um mecanismo de confirmação ao aplicativo. O mecanismo de confirmação pode variar dependendo do negócio e pode haver várias maneiras de implementar o mesmo mecanismo de confirmação para um determinado negócio.

Por exemplo, um sistema de bate-papo pode usar o seguinte mecanismo de confirmação. Cada mensagem é armazenada no banco de dados e cada mensagem tem um campo indicando se foi lida ou não. Quando um cliente recebe uma mensagem, ele envia um pacote de confirmação para o servidor, que então marca a mensagem como lida. Quando o cliente se conecta ao servidor (geralmente quando o usuário faz login ou reconexão), ele verifica se há mensagens não lidas no banco de dados e, em caso positivo, envia para o cliente. Da mesma forma, quando o cliente recebe a mensagem, ela notifica o servidor que foi lida. Desta forma, pode-se garantir que cada mensagem será recebida pelo destinatário. Claro, os desenvolvedores também podem usar sua própria lógica de confirmação.

## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Chama automaticamente \Workerman\Protocols\Websocket::encode para empacotar os dados no protocolo websocket antes de enviar
    $connection->send("hello\n");
};
// Executa o worker
Worker::runAll();
```

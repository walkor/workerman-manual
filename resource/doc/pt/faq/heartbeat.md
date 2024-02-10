# Pulsação

Nota: Aplicações de conexão de longa duração devem incluir uma pulsação, caso contrário, a conexão pode ser encerrada por nós de roteamento devido à falta de comunicação por um longo período de tempo.

A pulsação tem principalmente dois propósitos:

1. O cliente envia periodicamente dados para o servidor para evitar que a conexão seja fechada por firewalls de certos nós devido à falta de comunicação por um longo período de tempo.

2. O servidor pode determinar se o cliente está online através da pulsação. Se o cliente não enviar dados dentro do tempo estabelecido, é considerado offline. Isso permite detectar eventos de desconexão do cliente devido a circunstâncias extremas (queda de energia, desconexão de rede, etc.).

Intervalo recomendado para a pulsação:

Recomenda-se que o cliente envie pulsações com um intervalo inferior a 60 segundos, por exemplo, 55 segundos.

> O formato dos dados da pulsação não possui requisitos específicos, desde que o servidor possa identificá-lo.

## Exemplo de Pulsação

```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Intervalo de pulsação de 55 segundos
define('HEARTBEAT_TIME', 55);

$worker = new Worker('text://0.0.0.0:1234');

$worker->onMessage = function(TcpConnection $connection, $msg) {
    // Temporariamente define um atributo lastMessageTime na conexão para registrar o momento em que a última mensagem foi recebida
    $connection->lastMessageTime = time();
    // Outra lógica de negócios...
};

// Após o início do processo, configura um temporizador para ser executado a cada 10 segundos
$worker->onWorkerStart = function($worker) {
    Timer::add(10, function() use ($worker){
        $time_now = time();
        foreach($worker->connections as $connection) {
            // É possível que esta conexão ainda não tenha recebido nenhuma mensagem, portanto lastMessageTime é definido como o tempo atual
            if (empty($connection->lastMessageTime)) {
                $connection->lastMessageTime = $time_now;
                continue;
            }
            // Se o intervalo de tempo desde a última comunicação for maior que o intervalo de pulsação, o cliente é considerado offline e a conexão é fechada
            if ($time_now - $connection->lastMessageTime > HEARTBEAT_TIME) {
                $connection->close();
            }
        }
    });
};

Worker::runAll();
```

Com a configuração acima, se o cliente não enviar dados para o servidor por mais de 55 segundos, o servidor considerará o cliente desconectado, fechará a conexão e acionará onClose.

## Reconexão (Importante)

Quer seja o cliente ou o servidor que envia a pulsação, a conexão pode ser encerrada. Por exemplo, quando o JavaScript do navegador é pausado ao minimizar a janela, ao mudar para outras guias, ao desligar o computador, ao alternar redes em dispositivos móveis, sinal fraco, tela preta em dispositivos móveis, aplicativos móveis em segundo plano, falha de roteador, desconexão ativa, entre outros. Especialmente em um ambiente de internet externa complexo, muitos nós de roteamento podem limpar as conexões inativas em até 1 minuto, por isso a recomendação de um intervalo de pulsação inferior a 1 minuto.

As conexões são facilmente desconectadas em ambientes de internet externa, portanto, a reconexão é uma funcionalidade essencial para aplicações de conexão de longa duração (a reconexão só pode ser realizada pelo cliente, o servidor não pode implementá-la). Por exemplo, o WebSocket do navegador deve ouvir o evento onclose e, quando isso ocorrer, estabelecer uma nova conexão (para evitar problemas, a conexão deve ser estabelecida com atraso). De forma mais rígida, o servidor também deve enviar dados de pulsação periodicamente e o cliente deve monitorar se os dados de pulsação do servidor expiraram. Se não receber os dados de pulsação do servidor dentro do tempo estabelecido, a conexão deve ser considerada desconectada e fechada, sendo posteriormente estabelecida uma nova conexão.

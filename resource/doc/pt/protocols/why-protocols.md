# O papel do protocolo de comunicação
Como o TCP é baseado em fluxo, os dados de solicitação enviados pelo cliente fluem para o servidor como água. O servidor deve verificar se os dados estão completos, pois pode ser apenas parte de uma solicitação que chegou ao servidor, ou até mesmo várias solicitações conectadas chegaram ao servidor. Para determinar se a solicitação chegou completamente ou separá-la de várias solicitações conectadas, é necessário estabelecer um protocolo de comunicação.

## Por que estabelecer um protocolo no Workerman?
O desenvolvimento tradicional em PHP é baseado na Web, e geralmente é o protocolo HTTP que é usado. O processamento e análise do protocolo HTTP são de responsabilidade exclusiva do servidor Web, então os desenvolvedores não precisam se preocupar com questões de protocolo. No entanto, quando precisamos desenvolver com base em um protocolo não-HTTP, os desenvolvedores precisam considerar o protocolo.

## Protocolos suportados pelo Workerman
Atualmente, o Workerman suporta os protocolos HTTP, websocket, texto (ver apêndice para detalhes), frame (ver apêndice para detalhes) e ws (ver apêndice para detalhes). Ao comunicar com base nesses protocolos, eles podem ser usados diretamente, sendo especificados ao inicializar o Worker, da seguinte forma:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// websocket://0.0.0.0:2345 indica a escuta na porta 2345 usando o protocolo websocket
$websocket_worker = new Worker('websocket://0.0.0.0:2345');

// Protocolo de texto
$text_worker = new Worker('text://0.0.0.0:2346');

// Protocolo de frame
$frame_worker = new Worker('frame://0.0.0.0:2347');

// Trabalhador tcp, transmissão direta com base em soquete, sem uso de nenhum protocolo de camada de aplicação
$tcp_worker = new Worker('tcp://0.0.0.0:2348');

// Trabalhador udp, sem uso de nenhum protocolo de camada de aplicação
$udp_worker = new Worker('udp://0.0.0.0:2349');

// Trabalhador dominio unix, sem uso de nenhum protocolo de camada de aplicação
$unix_worker = new Worker('unix:///tmp/wm.sock');
```

## Utilizando um protocolo de comunicação personalizado
Quando os protocolos de comunicação internos do Workerman não atendem às necessidades de desenvolvimento, os desenvolvedores podem personalizar seu próprio protocolo de comunicação, conforme descrito na próxima seção.

**Dica:**
O Workerman possui um protocolo de texto embutido, com formato de protocolo de texto + quebra de linha. O desenvolvimento e depuração do protocolo de texto são muito simples e podem ser usados na maioria dos cenários de protocolos personalizados e suportam a depuração do telnet. Se os desenvolvedores desejarem desenvolver seu próprio protocolo de aplicação, eles podem simplesmente usar o protocolo de texto, sem necessidade de desenvolver um separadamente.

Consulte o apêndice sobre o protocolo de texto para mais detalhes.

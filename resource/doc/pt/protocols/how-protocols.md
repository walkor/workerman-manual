## Como personalizar um protocolo

Na verdade, é bastante simples criar seu próprio protocolo. Protocolos simples geralmente consistem em duas partes:
* Identificador para delimitar os dados
* Definição do formato dos dados

## Um exemplo

### Definição do protocolo
Vamos assumir que o identificador que delimita os dados é a quebra de linha "\n" (observe que os dados da solicitação em si não podem conter a quebra de linha), e o formato dos dados é Json. Abaixo está um exemplo de um pacote de solicitação que segue essas regras.

```json
{"type":"message","content":"hello"}
```

Observe que no final dos dados da solicitação acima, há um caractere de quebra de linha (representado em PHP com uma string de **aspas duplas** "\n"), indicando o término da solicitação.

### Etapas de implementação
No Workerman, se quisermos implementar o protocolo acima, considerando que o nome do protocolo é JsonNL e o projeto é chamado de MyApp, precisaremos seguir os passos abaixo:

1. Colocar o arquivo de protocolo na pasta Protocols do projeto, por exemplo, o arquivo ficará em MyApp/Protocols/JsonNL.php.

2. Implementar a classe JsonNL com o namespace `Protocols;`. Esta classe deve obrigatoriamente implementar três métodos estáticos: input, encode e decode.

Observação: O Workerman chamará automaticamente esses três métodos estáticos para realizar a fragmentação, empacotamento e desempacotamento. Consulte a explicação detalhada do fluxo de execução abaixo.

### Fluxo de interação entre o Workerman e a classe de protocolo
1. Suponha que o cliente envie um pacote de dados para o servidor. O servidor, ao receber os dados (que podem ser somente uma parte dos dados), imediatamente chamará o método `input` do protocolo para verificar o comprimento do pacote. O método `input` retornará o comprimento `$length` para o framework Workerman.
2. Após obter o valor de `$length`, o Workerman verificará se o buffer de dados atual já recebeu dados de comprimento `$length`. Se ainda não recebeu, o Workerman continuará aguardando dados até que o comprimento do buffer seja igual ou superior a `$length`.
3. Quando o comprimento do buffer for suficiente, o Workerman extrairá (ou seja, **fragmentação**) os dados do buffer com o comprimento `$length` e chamará o método `decode` do protocolo para realizar o **desempacotamento**. Os dados desempacotados serão armazenados em `$data`.
4. Após o desempacotamento, o Workerman passará os dados `$data` para a camada de negócios por meio da chamada de retorno `onMessage($connection, $data)`. Na função `onMessage`, a aplicação pode usar a variável `$data` para obter os dados completos e desempacotados enviados pelo cliente.
5. Quando a camada de negócios precisa enviar dados para o cliente através do método `$connection->send($buffer)`, o Workerman automaticamente utilizará o método `encode` do protocolo para **empacotar** o `$buffer` antes de enviá-lo para o cliente.

### Implementação específica

**Implementação de MyApp/Protocols/JsonNL.php**

```php
namespace Protocols;
class JsonNL
{
    public static function input($buffer)
    {
        // Encontra a posição do caractere de quebra de linha "\n"
        $pos = strpos($buffer, "\n");
        // Se não houver caractere de quebra de linha, o comprimento do pacote não pode ser determinado, retorna 0 e aguarda mais dados
        if($pos === false)
        {
            return 0;
        }
        // Se houver caractere de quebra de linha, retorna o comprimento atual do pacote (incluindo o caractere de quebra de linha)
        return $pos+1;
    }

    public static function encode($buffer)
    {
        // Serializa em JSON e adiciona a quebra de linha como marcador de término da solicitação
        return json_encode($buffer)."\n";
    }

    public static function decode($buffer)
    {
        // Remove a quebra de linha e decodifica para um array
        return json_decode(trim($buffer), true);
    }
}
```

Com isso, o protocolo JsonNL está implementado e pode ser utilizado no projeto MyApp. Um exemplo de uso pode ser visto abaixo.

Arquivo: MyApp\start.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$json_worker = new Worker('JsonNL://0.0.0.0:1234');
$json_worker->onMessage = function(TcpConnection $connection, $data) {

    // $data contém os dados enviados pelo cliente, e esses dados já foram processados pelo JsonNL::decode
    echo $data;
    
    // Os dados enviados através do $connection->send serão automaticamente empacotados com o método JsonNL::encode e enviados para o cliente
    $connection->send(array('code'=>0, 'msg'=>'ok'));
    
};
Worker::runAll();
```

> **Observação**:
> O Workerman tentará carregar o protocolo sob o namespace `Protocols`, portanto, ao utilizar `new Worker('JsonNL://0.0.0.0:1234')`, o Workerman tentará carregar o protocolo `Protocols\JsonNL`. Se ocorrer um erro como `Class 'Protocols\JsonNL' not found`, consulte a implementação do [carregamento automático](../faq/autoload.md) para resolver esse problema.

### Descrição da interface do protocolo
As classes de protocolo desenvolvidas no Workerman devem implementar obrigatoriamente três métodos estáticos: input, encode e decode. A descrição da interface do protocolo está disponível no arquivo Workerman/Protocols/ProtocolInterface.php, conforme mostrado abaixo:

```php
namespace Workerman\Protocols;

use \Workerman\Connection\ConnectionInterface;

interface ProtocolInterface
{
    public static function input($recv_buffer, ConnectionInterface $connection);
    public static function decode($recv_buffer, ConnectionInterface $connection);
    public static function encode($data, ConnectionInterface $connection);
}
```

## Observação:
O Workerman não exige estritamente que a classe de protocolo seja baseada na ProtocolInterface. Na prática, a classe do protocolo só precisa ter os três métodos estáticos: input, encode e decode.

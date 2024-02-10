# Alguns exemplos

## Exemplo 1

### Definição do Protocolo
  * Os 10 primeiros bytes são usados para salvar o comprimento total do pacote de dados, completado com zeros se necessário
  * O formato dos dados é XML

### Exemplo de Pacote de Dados
```xml
0000000121<?xml version="1.0" encoding="ISO-8859-1"?>
<request>
    <module>user</module>
    <action>getInfo</action>
</request>
```
O número 0000000121 representa o comprimento total do pacote, seguido pelos dados em formato XML

### Implementação do Protocolo
```php
namespace Protocols;
class XmlProtocol
{
    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer) < 10)
        {
            // Menos de 10 bytes, retorna 0 para aguardar mais dados
            return 0;
        }
        $total_len = base_convert(substr($recv_buffer, 0, 10), 10, 10);
        return $total_len;
    }

    public static function decode($recv_buffer)
    {
        $body = substr($recv_buffer, 10);
        return simplexml_load_string($body);
    }

    public static function encode($xml_string)
    {
        $total_length = strlen($xml_string)+10;
        $total_length_str = str_pad($total_length, 10, '0', STR_PAD_LEFT);
        return $total_length_str . $xml_string;
    }
}
```

## Exemplo 2

### Definição do Protocolo
  * Os primeiros 4 bytes representam o comprimento total do pacote no formato de int unsigned em ordem de bytes de rede
  * A parte dos dados é uma string Json

### Exemplo de Pacote de Dados
<pre>
****{"type":"message","content":"hello all"}
</pre>
Os primeiros quatro bytes representam um int unsigned em ordem de bytes de rede e são caracteres invisíveis, seguidos pelos dados em formato Json

### Implementação do Protocolo
```php
namespace Protocols;
class JsonInt
{
    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer)<4)
        {
            return 0;
        }
        $unpack_data = unpack('Ntotal_length', $recv_buffer);
        return $unpack_data['total_length'];
    }

    public static function decode($recv_buffer)
    {
        $body_json_str = substr($recv_buffer, 4);
        return json_decode($body_json_str, true);
    }

    public static function encode($data)
    {
        $body_json_str = json_encode($data);
        $total_length = 4 + strlen($body_json_str);
        return pack('N',$total_length) . $body_json_str;
    }
}
```

## Exemplo 3 (Envio de Arquivo usando Protocolo Binário)

### Definição do Protocolo
```C
struct
{
  unsigned int total_len;  // Comprimento total do pacote em ordem de bytes de rede
  char         name_len;   // Comprimento do nome do arquivo
  char         name[name_len]; // Nome do arquivo
  char         file[total_len - BinaryTransfer::PACKAGE_HEAD_LEN - name_len]; // Dados do arquivo
}
```
### Exemplo do Protocolo
<pre> *****logo.png****************** </pre>
Os primeiros quatro bytes * representam um int unsigned em ordem de bytes de rede e são caracteres invisíveis, o quinto * é um byte que armazena o comprimento do nome do arquivo, seguido pelo nome do arquivo e, em seguida, os dados binários originais do arquivo

### Implementação do Protocolo
```php
namespace Protocols;
class BinaryTransfer
{
    // Comprimento do cabeçalho do protocolo
    const PACKAGE_HEAD_LEN = 5;

    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer) < self::PACKAGE_HEAD_LEN)
        {
            return 0;
        }
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        return $package_data['total_len'];
    }


    public static function decode($recv_buffer)
    {
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        $name_len = $package_data['name_len'];
        $file_name = substr($recv_buffer, self::PACKAGE_HEAD_LEN, $name_len);
        $file_data = substr($recv_buffer, self::PACKAGE_HEAD_LEN + $name_len);
         return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        return $data;
    }
}
```

### Exemplo de Uso do Protocolo no Servidor
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('BinaryTransfer://0.0.0.0:8333');
// Salva o arquivo em /tmp
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### Exemplo de Cliente de Arquivo client.php (usando PHP para simular o cliente de upload)
```php
<?php
/** Cliente de Envio de Arquivo **/
// Endereço de envio
$address = "127.0.0.1:8333";
// Verifica o caminho do arquivo a ser enviado
if(!isset($argv[1]))
{
   exit("use php client.php \$file_path\n");
}
$file_to_transfer = trim($argv[1]);
// Arquivo a ser enviado não existe localmente
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer not exist\n");
}
// Estabelece a conexão via socket
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
// Configuração de bloqueio
stream_set_blocking($client, 1);
// Nome do arquivo
$file_name = basename($file_to_transfer);
// Comprimento do nome do arquivo
$name_len = strlen($file_name);
// Dados binários do arquivo
$file_data = file_get_contents($file_to_transfer);
// Comprimento do cabeçalho do protocolo: 4 bytes para o comprimento do pacote + 1 byte para o comprimento do nome do arquivo
$PACKAGE_HEAD_LEN = 5;
// Pacote do protocolo
$package = pack('NC', $PACKAGE_HEAD_LEN  + strlen($file_name) + strlen($file_data), $name_len) . $file_name . $file_data;
// Executa o envio
fwrite($client, $package);
// Exibe o resultado
echo fread($client, 8192),"\n";
```

### Exemplo de Uso do Cliente
Execute o seguinte comando no terminal: ```php client.php <caminho do arquivo>```

Por exemplo: ```php client.php abc.jpg```
## Exemplo Quatro (Carregamento de arquivo usando protocolo de texto)

### Definição do Protocolo

json + quebra de linha, o json contém o nome do arquivo e os dados do arquivo codificados em base64_encode (aumentará o tamanho em 1/3)

### Amostra do Protocolo

{"file_name":"logo.png","file_data":"PD9waHAKLyo......"}\n

Preste atenção que ao final há um caractere de quebra de linha, em PHP é representado pelo caractere de aspas duplas ```"\n"```

### Implementação do Protocolo

```php
namespace Protocols;
class TextTransfer
{
    public static function input($recv_buffer)
    {
        $recv_len = strlen($recv_buffer);
        if($recv_buffer[$recv_len-1] !== "\n")
        {
            return 0;
        }
        return strlen($recv_buffer);
    }

    public static function decode($recv_buffer)
    {
        // Desempacotando
        $package_data = json_decode(trim($recv_buffer), true);
        // Obtendo o nome do arquivo
        $file_name = $package_data['file_name'];
        // Obtendo os dados do arquivo codificados em base64_encode
        $file_data = $package_data['file_data'];
        // Decodificando base64 para obter os dados binários originais do arquivo
        $file_data = base64_decode($file_data);
        // Retornando os dados
        return array(
            'file_name' => $file_name,
            'file_data' => $file_data,
        );
    }

    public static function encode($data)
    {
        // Pode codificar os dados a serem enviados para o cliente de acordo com a necessidade, aqui, os dados são simplesmente retornados como texto
        return $data;
    }
}
```

### Exemplo de Uso do Protocolo no Servidor

Explicação: a sintaxe é a mesma que a utilizada para o carregamento binário, ou seja, é possível alternar os protocolos quase sem alterar o código de negócio algum

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('TextTransfer://0.0.0.0:8333');
// Salvar arquivo em /tmp
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### Cliente do Arquivo textclient.php (usando PHP para simular o envio do arquivo)

```php
<?php
/** Cliente para envio de arquivo **/
// Endereço de envio
$address = "127.0.0.1:8333";
// Verificar o parâmetro do caminho do arquivo a ser enviado
if(!isset($argv[1]))
{
   exit("use php client.php \$file_path\n");
}
// Caminho do arquivo a ser enviado
$file_to_transfer = trim($argv[1]);
// O arquivo local a ser enviado não existe
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer not exist\n");
}
// Estabelecer conexão com o socket
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
stream_set_blocking($client, 1);
// Nome do arquivo
$file_name = basename($file_to_transfer);
// Dados binários do arquivo
$file_data = file_get_contents($file_to_transfer);
// Codificação base64
$file_data = base64_encode($file_data);
// Pacote de dados
$package_data = array(
    'file_name' => $file_name,
    'file_data' => $file_data,
);
// Pacote de protocolo json + quebra de linha
$package = json_encode($package_data)."\n";
// Executar o envio
fwrite($client, $package);
// Imprimir resultado
echo fread($client, 8192),"\n";
``` 

### Exemplo de Uso do Cliente

Executar o comando na linha de comando ```php textclient.php <caminho do arquivo>```

Por exemplo, ```php textclient.php abc.jpg```

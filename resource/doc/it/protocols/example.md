# Alcuni esempi

## Esempio uno

### Definizione del protocollo
  - Un'intestazione fissa di 10 byte per salvare la lunghezza dell'intero pacchetto, con zeri aggiunti se la lunghezza è inferiore a 10 byte
  - Il formato dei dati è XML

### Esempio di pacchetto
```xml
0000000121<?xml version="1.0" encoding="ISO-8859-1"?>
<request>
    <module>user</module>
    <action>getInfo</action>
</request>
```
Dove 0000000121 rappresenta la lunghezza dell'intero pacchetto, seguito dal contenuto del pacchetto in formato XML

### Implementazione del protocollo
```php
namespace Protocols;
class XmlProtocol
{
    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer) < 10)
        {
            // Se la lunghezza è inferiore a 10 byte, restituisci 0 e aspetta ulteriori dati
            return 0;
        }
        // Restituisci la lunghezza del pacchetto, comprensiva della lunghezza dell'intestazione e del corpo del pacchetto
        $total_len = base_convert(substr($recv_buffer, 0, 10), 10, 10);
        return $total_len;
    }

    public static function decode($recv_buffer)
    {
        // Corpo del pacchetto
        $body = substr($recv_buffer, 10);
        return simplexml_load_string($body);
    }

    public static function encode($xml_string)
    {
        // Lunghezza totale del pacchetto (lunghezza del corpo + 10 per l'intestazione)
        $total_length = strlen($xml_string) + 10;
        // Aggiungi zeri davanti per raggiungere la lunghezza di 10 byte nell'intestazione
        $total_length_str = str_pad($total_length, 10, '0', STR_PAD_LEFT);
        // Restituisci i dati
        return $total_length_str . $xml_string;
    }
}
```

## Esempio due

### Definizione del protocollo
  - Un'intestazione di 4 byte in formato di rete, unsigned int, che indica la lunghezza dell'intero pacchetto
  - La parte dati è una stringa JSON

### Esempio di pacchetto
<pre>
****{"type":"message","content":"hello all"}
</pre>
Dove i primi quattro byte asterisco (*) rappresentano un unsigned int in formato di rete e sono caratteri non visibili, seguiti dai dati del pacchetto in formato JSON

### Implementazione del protocollo
```php
namespace Protocols;
class JsonInt
{
    public static function input($recv_buffer)
    {
        // Se la lunghezza dei dati è inferiore a 4 byte, non è possibile determinare la lunghezza del pacchetto, restituisci 0 e aspetta ulteriori dati
        if(strlen($recv_buffer) < 4)
        {
            return 0;
        }
        // Utilizza la funzione unpack per convertire i primi 4 byte in numeri, che rappresentano la lunghezza dell'intero pacchetto
        $unpack_data = unpack('Ntotal_length', $recv_buffer);
        return $unpack_data['total_length'];
    }

    public static function decode($recv_buffer)
    {
        // Rimuovi i primi 4 byte per ottenere i dati JSON del pacchetto
        $body_json_str = substr($recv_buffer, 4);
        // Decodifica JSON
        return json_decode($body_json_str, true);
    }

    public static function encode($data)
    {
        // Codifica i dati in JSON per ottenere il corpo del pacchetto
        $body_json_str = json_encode($data);
        // Calcola la lunghezza totale del pacchetto (4 per l'intestazione + lunghezza della stringa JSON)
        $total_length = 4 + strlen($body_json_str);
        // Restituisci i dati confezionati
        return pack('N', $total_length) . $body_json_str;
    }
}
```

## Esempio tre (trasferimento di file con protocollo binario)

### Definizione del protocollo
```C
struct
{
  unsigned int total_len;  // Lunghezza dell'intero pacchetto, byte di ordine di rete big-endian
  char         name_len;   // Lunghezza del nome del file
  char         name[name_len]; // Nome del file
  char         file[total_len - BinaryTransfer::PACKAGE_HEAD_LEN - name_len]; // Dati del file
}
```
### Esempio del protocollo
<pre> *****logo.png****************** </pre>
Dove i primi quattro byte asterisco (*) rappresentano un unsigned int in formato di rete e sono caratteri non visibili, il quinto * è il numero di byte utilizzato per memorizzare la lunghezza del nome del file, seguito direttamente dal nome del file e successivamente dai dati binari non elaborati del file

### Implementazione del protocollo
```php
namespace Protocols;
class BinaryTransfer
{
    // Lunghezza dell'intestazione del protocollo
    const PACKAGE_HEAD_LEN = 5;

    public static function input($recv_buffer)
    {
        // Se i dati non sono lunghi quanto l'intestazione del protocollo, continua ad aspettare
        if(strlen($recv_buffer) < self::PACKAGE_HEAD_LEN)
        {
            return 0;
        }
        // Scompatta i dati
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // Restituisci la lunghezza del pacchetto
        return $package_data['total_len'];
    }


    public static function decode($recv_buffer)
    {
        // Scompatta i dati
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // Lunghezza del nome del file
        $name_len = $package_data['name_len'];
        // Estrai il nome del file dal flusso di dati
        $file_name = substr($recv_buffer, self::PACKAGE_HEAD_LEN, $name_len);
        // Estrai i dati binari del file dal flusso di dati
        $file_data = substr($recv_buffer, self::PACKAGE_HEAD_LEN + $name_len);
         return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // Puoi codificare i dati da inviare al client in base alle tue esigenze, qui vengono restituiti solo i dati iniziali di testo
        return $data;
    }
}
```

### Esempio di utilizzo del protocollo sul server

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('BinaryTransfer://0.0.0.0:8333');
// Salva il file in tmp
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### Esempio di utilizzo del client per inviare file client.php (in questo esempio, il client viene simulato in PHP per l'upload)

```php
<?php
/** Client di upload file **/
// Indirizzo di upload
$address = "127.0.0.1:8333";
// Controlla i parametri del percorso del file da inviare
if(!isset($argv[1]))
{
   exit("Usa php client.php \$file_path\n");
}
// Percorso del file da inviare
$file_to_transfer = trim($argv[1]);
// Il file da inviare non esiste localmente
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer non esiste\n");
}
// Stabilisci la connessione del socket
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
// Imposta il blocco
stream_set_blocking($client, 1);
// Nome del file
$file_name = basename($file_to_transfer);
// Lunghezza del nome del file
$name_len = strlen($file_name);
// Dati binari del file
$file_data = file_get_contents($file_to_transfer);
// Lunghezza dell'intestazione del protocollo 4 byte per la lunghezza + 1 byte per la lunghezza del nome del file
$PACKAGE_HEAD_LEN = 5;
// Pacchetto del protocollo
$package = pack('NC', $PACKAGE_HEAD_LEN  + strlen($file_name) + strlen($file_data), $name_len) . $file_name . $file_data;
// Esegui l'upload
fwrite($client, $package);
// Stampa i risultati
echo fread($client, 8192),"\n";
```

### Esempio di utilizzo del client
Esegui da linea di comando ```php client.php <percorso_del_file>```

Ad esempio ```php client.php abc.jpg```
## Esempio 4 (Caricamento di file tramite protocollo di testo)

### Definizione del protocollo

json + nuova riga, il json contiene il nome del file e i dati del file codificati in base64 (aumenterà il volume del 1/3)

### Esempio del protocollo

{"file_name":"logo.png","file_data":"PD9waHAKLyo......"}\n

Nota che alla fine c'è un carattere di nuova riga, rappresentato da ```"\n"``` tra virgolette doppie in PHP.

### Implementazione del protocollo

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
        // Decodifica il pacchetto
        $package_data = json_decode(trim($recv_buffer), true);
        // Prende il nome del file
        $file_name = $package_data['file_name'];
        // Prende i dati del file codificati in base64
        $file_data = $package_data['file_data'];
        // Decodifica base64 per ottenere i dati binari originali del file
        $file_data = base64_decode($file_data);
        // Ritorna i dati
        return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // Si può codificare i dati da inviare al client secondo le proprie esigenze, qui semplicemente restituiamo il testo originale
        return $data;
    }
}

```

### Esempio di utilizzo del protocollo lato server

Spiegazione: La sintassi è la stessa del caricamento binario, quindi è possibile passare al protocollo con pochissime modifiche al codice di business.

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('TextTransfer://0.0.0.0:8333');
// Salvare il file in/tmp
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### Esempio di utilizzo del client di file textclient.php (in questo caso verrà simulato il client in PHP per il caricamento)
```php
<?php
/** Client di caricamento file **/
// Indirizzo di caricamento
$address = "127.0.0.1:8333";
// Controlla i parametri del percorso di caricamento del file
if(!isset($argv[1]))
{
   exit("utilizzare php client.php \$file_path\n");
}
// Percorso del file da caricare
$file_to_transfer = trim($argv[1]);
// Il file da caricare non esiste localmente
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer non esiste\n");
}
// Stabilire una connessione socket
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
stream_set_blocking($client, 1);
// Nome del file
$file_name = basename($file_to_transfer);
// Dati binari del file
$file_data = file_get_contents($file_to_transfer);
// Codifica in base64
$file_data = base64_encode($file_data);
// Pacchetto di dati
$package_data = array(
    'file_name' => $file_name,
    'file_data' => $file_data,
);
// Pacchetto del protocollo json + invio
$package = json_encode($package_data)."\n";
// Esecuzione del caricamento
fwrite($client, $package);
// Stampare il risultato
echo fread($client, 8192),"\n";
```

### Esempio di utilizzo del client

Eseguire il comando nella riga di comando ```php textclient.php <percorso_del_file>```

Ad esempio ```php textclient.php abc.jpg```

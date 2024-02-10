# Beispiele

## Beispiel Eins

### Protokolldefinition
  * Die ersten 10 Byte sind festgelegt, um die Gesamtlänge des Datenpakets zu speichern. Wenn die Stellen nicht ausreichen, werden sie mit 0 aufgefüllt.
  * Datenformat ist XML

### Beispieldatenpaket
```xml
0000000121<?xml version="1.0" encoding="ISO-8859-1"?>
<request>
    <module>user</module>
    <action>getInfo</action>
</request>
```
Hier steht 0000000121 für die Gesamtlänge des Datenpakets, gefolgt vom Inhalt des Datenkörpers im XML-Format.

### Protokollimplementierung
```php
namespace Protocols;
class XmlProtocol
{
    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer) < 10)
        {
            // Weniger als 10 Bytes, Rückgabe 0, um auf weitere Daten zu warten
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
        $total_length = strlen($xml_string) + 10;
        $total_length_str = str_pad($total_length, 10, '0', STR_PAD_LEFT);
        return $total_length_str . $xml_string;
    }
}
```

## Beispiel Zwei

### Protokolldefinition
  * Die ersten 4 Bytes sind als Netzwerk-Byte-Reihenfolge für eine nicht-negative ganze Zahl definiert und kennzeichnen die Gesamtlänge des Pakets.
  * Der Datenbereich ist ein JSON-String.

### Beispieldatenpaket
<pre>
****{"type":"message","content":"hello all"}
</pre>

Hier repräsentieren die ersten vier Sternchen vier Bytes in Netzwerk-Byte-Reihenfolge für die Gesamtlänge des Datenpakets, gefolgt vom Datenkörper im JSON-Format.

### Protokollimplementierung
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

## Beispiel Drei (Dateiübertragung mit binärem Protokoll)

### Protokolldefinition
```C
struct
{
  unsigned int total_len;  // Gesamtlänge des Pakets, in Big-Endian-Netzwerkbyte-Reihenfolge
  char         name_len;   // Länge des Dateinamens
  char         name[name_len]; // Dateiname
  char         file[total_len - BinaryTransfer::PACKAGE_HEAD_LEN - name_len]; // Dateidaten
}
```

### Beispiel für das Protokoll
<pre> *****logo.png****************** </pre>

Hier repräsentiert das erste Sternchen vier Zeichen in Big-Endian-Netzwerkbyte-Reihenfolge für die Gesamtlänge des Datenpakets, gefolgt von einem einzelnen Zeichen zur Speicherung der Dateinamenlänge, gefolgt vom Dateinamen und den Rohdaten der Binärdatei.

### Protokollimplementierung
```php
namespace Protocols;
class BinaryTransfer
{
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

### Beispiel für die Verwendung des Serverprotokolls

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('BinaryTransfer://0.0.0.0:8333');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("Upload erfolgreich. Speicherpfad $save_path");
};

Worker::runAll();
```

### Beispiel für den Client "client.php" (Verwendung von PHP, um Datei zu übertragen)
```php
<?php
/** Dateiübertragungsclient **/
$address = "127.0.0.1:8333";

if(!isset($argv[1]))
{
   exit("Verwendung: php client.php \$file_path\n");
}

$file_to_transfer = trim($argv[1]);

if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer existiert nicht\n");
}

$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}

stream_set_blocking($client, 1);

$file_name = basename($file_to_transfer);
$name_len = strlen($file_name);
$file_data = file_get_contents($file_to_transfer);
$PACKAGE_HEAD_LEN = 5;

$package = pack('NC', $PACKAGE_HEAD_LEN  + strlen($file_name) + strlen($file_data), $name_len) . $file_name . $file_data;

fwrite($client, $package);

echo fread($client, 8192),"\n";
```

### Beispiel für die Client-Verwendung
Führen Sie den folgenden Befehl in der Befehlszeile aus: ```php client.php <Dateipfad>```
Zum Beispiel: ```php client.php abc.jpg```
## Beispiel vier (Verwendung des Textprotokolls zum Hochladen von Dateien)

### Protokolldefinition

Json mit Zeilenumbruch. Das Json enthält den Dateinamen und die base64_encodierte (das Volumen wird um 1/3 vergrößert) Dateidaten.

### Protokollbeispiel

{"file_name":"logo.png","file_data":"PD9waHAKLyo......"}\n

Beachten Sie, am Ende steht ein Zeilenumbruchzeichen. In PHP wird dies durch das Doppelanführungszeichen ```"\n"``` dargestellt.

### Protokollumsetzung
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
        // Entpacken
        $package_data = json_decode(trim($recv_buffer), true);
        // Dateinamen abrufen
        $file_name = $package_data['file_name'];
        // base64_encodierte Dateidaten abrufen
        $file_data = $package_data['file_data'];
        // base64_decode zur Wiederherstellung der ursprünglichen Binärdateidaten
        $file_data = base64_decode($file_data);
        // Daten zurückgeben
        return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // Hier können die an den Client gesendeten Daten je nach Bedarf codiert werden. Hier werden sie einfach als Text zurückgegeben.
        return $data;
    }
}
```

### Beispiel für die Verwendung des Serverprotokolls
Hinweis: Die Schreibweise ist die gleiche wie beim binären Upload, sodass fast keine Änderungen am Geschäftscode erforderlich sind, um das Protokoll zu wechseln.

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('TextTransfer://0.0.0.0:8333');
// Datei in tmp speichern
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("Upload erfolgreich. Speicherort $save_path");
};

Worker::runAll();
```

### Clientdatei textclient.php (hier wird der Client-Upload in PHP simuliert)
```php
<?php
/** Datei-Upload-Client **/
// Upload-Adresse
$address = "127.0.0.1:8333";
// Überprüfen der Dateipfadparameter
if(!isset($argv[1]))
{
   exit("Verwenden Sie php client.php \$file_path\n");
}
// Dateipfad für den Upload
$file_to_transfer = trim($argv[1]);
// Die zu übertragende Datei existiert lokal nicht
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer nicht vorhanden\n");
}
// Socket-Verbindung herstellen
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
stream_set_blocking($client, 1);
// Dateiname
$file_name = basename($file_to_transfer);
// Dateibinärdaten
$file_data = file_get_contents($file_to_transfer);
// Base64-Codierung
$file_data = base64_encode($file_data);
// Datenpaket
$package_data = array(
    'file_name' => $file_name,
    'file_data' => $file_data,
);
// Protokollpaket Json + Zeilenumbruch
$package = json_encode($package_data)."\n";
// Upload ausführen
fwrite($client, $package);
// Ergebnis ausgeben
echo fread($client, 8192),"\n";
```

### Beispiel für die Verwendung des Clients
Führen Sie den Befehl ```php textclient.php <Dateipfad>``` in der Befehlszeile aus.

Beispiel: ```php textclient.php abc.jpg```

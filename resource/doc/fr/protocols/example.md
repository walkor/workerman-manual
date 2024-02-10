# Quelques exemples

## Exemple 1

### Définition du protocole
  * Un en-tête fixe de 10 octets est utilisé pour stocker la longueur totale du paquet de données, en complétant avec des zéros si nécessaire
  * Le format des données est au format XML

### Exemple de paquet de données
```xml
0000000121<?xml version="1.0" encoding="ISO-8859-1"?>
<request>
    <module>user</module>
    <action>getInfo</action>
</request>
```
Où 0000000121 représente la longueur totale du paquet de données, suivi du contenu du corps du paquet au format XML

### Implémentation du protocole
```php
namespace Protocols;
class XmlProtocol
{
    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer) < 10)
        {
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

## Exemple 2

### Définition du protocole
  * Un entier non signé en octets réseau de 4 octets est utilisé pour marquer la longueur totale du paquet
  * La partie des données est une chaîne JSON

### Exemple de paquet de données
<pre>
****{"type":"message","content":"hello all"} 
</pre>

Où les quatre astérisques représentent un entier non signé en octets réseau, qui est un caractère invisible, suivi des données au format JSON

### Implémentation du protocole
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

## Exemple 3 (Transfert de fichier binaire à l'aide d'un protocole)

### Définition du protocole
```C
struct
{
  unsigned int total_len;  // Longueur totale du paquet, en octets réseau big endian
  char         name_len;   // Longueur du nom de fichier
  char         name[name_len]; // Nom du fichier
  char         file[total_len - BinaryTransfer::PACKAGE_HEAD_LEN - name_len]; // Données du fichier
}
```

### Exemple de protocole
<pre> *****logo.png****************** </pre>

Où les quatre astérisques au début représentent un entier non signé en octets réseau, qui est un caractère invisible, le cinquième asterisque stocke la longueur du nom de fichier sur un octet, suivi du nom du fichier, puis des données binaires brutes du fichier

### Implémentation du protocole
```php
namespace Protocols;
class BinaryTransfer
{
    // Longueur de l'en-tête du protocole
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

### Exemple d'utilisation du protocole côté serveur

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('BinaryTransfer://0.0.0.0:8333');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("Téléchargement réussi. Chemin de sauvegarde $save_path");
};

Worker::runAll();
```

### Client de fichier exemple client.php (utilisation de PHP pour simuler le client d'envoi)

```php
<?php
/** Client d'envoi de fichier **/
// Adresse d'envoi
$address = "127.0.0.1:8333";
// Vérifie le paramètre du chemin du fichier à envoyer
if(!isset($argv[1]))
{
   exit("Utilisation: php client.php \$file_path\n");
}
// Chemin du fichier à envoyer
$file_to_transfer = trim($argv[1]);
// Le fichier à envoyer n'existe pas localement
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer n'existe pas\n");
}
// Établir une connexion par socket
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
// Mettre en mode blocage
stream_set_blocking($client, 1);
// Nom du fichier
$file_name = basename($file_to_transfer);
// Longueur du nom du fichier
$name_len = strlen($file_name);
// Données binaires du fichier
$file_data = file_get_contents($file_to_transfer);
// Longueur de l'en-tête du protocole : 4 octets pour la longueur du paquet + 1 octet pour la longueur du nom du fichier
$PACKAGE_HEAD_LEN = 5;
// Paquet protocolaire
$package = pack('NC', $PACKAGE_HEAD_LEN  + strlen($file_name) + strlen($file_data), $name_len) . $file_name . $file_data;
// Exécution de l'envoi
fwrite($client, $package);
// Affichage du résultat
echo fread($client, 8192),"\n";
```

### Exemple d'utilisation du client
Exécutez dans la ligne de commande ```php client.php <chemin_du_fichier>```

Par exemple, ```php client.php abc.jpg```
## Exemple 4 (Upload de fichier en utilisant un protocole texte)

### Définition du protocole

json + saut de ligne, le json contient le nom du fichier et les données du fichier encodées en base64 (augmente la taille de 1/3).

### Exemple de protocole

{"file_name":"logo.png","file_data":"PD9waHAKLyo......"}\n

Notez qu'à la fin se trouve un saut de ligne, représenté par le caractère de double citation dans PHP ```"\n"```

### Implémentation du protocole

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
        // Désemballer
        $package_data = json_decode(trim($recv_buffer), true);
        // Récupérer le nom du fichier
        $file_name = $package_data['file_name'];
        // Récupérer les données du fichier encodées en base64
        $file_data = $package_data['file_data'];
        // Décoder en base64 pour retrouver les données binaires originales du fichier
        $file_data = base64_decode($file_data);
        // Retourner les données
        return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // Vous pouvez encoder les données à envoyer au client selon vos besoins. Ici, les données sont simplement renvoyées telles quelles.
        return $data;
    }
}

```

### Exemple d'utilisation du protocole côté serveur

Remarque: La syntaxe est la même que pour l'envoi de données binaires, ce qui permet de presque pas modifier le code métier.

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('TextTransfer://0.0.0.0:8333');
// Enregistrer le fichier dans tmp
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("Téléchargement réussi. Chemin de sauvegarde $save_path");
};

Worker::runAll();
```

### Client de fichier textclient.php (utilisation de PHP pour simuler le client d'envoi)

```php
<?php
/** Client d'envoi de fichier **/
// Adresse d'envoi
$address = "127.0.0.1:8333";
// Vérifier les paramètres du chemin de fichier à envoyer
if(!isset($argv[1]))
{
   exit("Utilisez php client.php \$chemin_fichier\n");
}
// Chemin du fichier à envoyer
$file_to_transfer = trim($argv[1]);
// Le fichier à envoyer n'existe pas localement
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer n'existe pas\n");
}
// Établir la connexion socket
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
stream_set_blocking($client, 1);
// Nom du fichier
$file_name = basename($file_to_transfer);
// Données binaires du fichier
$file_data = file_get_contents($file_to_transfer);
// Encodage en base64
$file_data = base64_encode($file_data);
// Paquet de données
$package_data = array(
    'file_name' => $file_name,
    'file_data' => $file_data,
);
// Paquet de protocole json + saut de ligne
$package = json_encode($package_data)."\n";
// Envoyer
fwrite($client, $package);
// Afficher le résultat
echo fread($client, 8192),"\n";
```

### Exemple d'utilisation du client

Exécutez la commande dans le terminal: ```php textclient.php <chemin_fichier>```

Par exemple: ```php textclient.php abc.jpg```

# Some Examples

## Example One

### Protocol Definition
  * The header is fixed at 10 bytes in length to store the entire length of the data packet, with leading zeros to pad if necessary
  * Data format is XML

### Sample Data Packet
```xml
0000000121<?xml version="1.0" encoding="ISO-8859-1"?>
<request>
    <module>user</module>
    <action>getInfo</action>
</request>
```
Where 0000000121 represents the entire length of the data packet, followed by the XML data content

### Protocol Implementation
```php
namespace Protocols;
class XmlProtocol
{
    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer) < 10)
        {
            // Less than 10 bytes, return 0 to wait for more data
            return 0;
        }
        // Return the total length, including the header length and the body length
        $total_len = base_convert(substr($recv_buffer, 0, 10), 10, 10);
        return $total_len;
    }

    public static function decode($recv_buffer)
    {
        // Request body
        $body = substr($recv_buffer, 10);
        return simplexml_load_string($body);
    }

    public static function encode($xml_string)
    {
        // Body + header length
        $total_length = strlen($xml_string)+10;
        // Make the length 10 bytes long, with leading zeros if necessary
        $total_length_str = str_pad($total_length, 10, '0', STR_PAD_LEFT);
        // Return data
        return $total_length_str . $xml_string;
    }
}
```

## Example Two

### Protocol Definition
  * 4-byte network byte order unsigned int in the header to indicate the length of the entire packet
  * Data section is a JSON string

### Sample Data Packet
<pre>
****{"type":"message","content":"hello all"}
</pre>

Where the first four *s represent a network byte order unsigned int data, which cannot be seen, followed by the JSON data content

### Protocol Implementation
```php
namespace Protocols;
class JsonInt
{
    public static function input($recv_buffer)
    {
        // If the received data is less than 4 bytes, the packet length is unknown, return 0 to wait for more data
        if(strlen($recv_buffer)<4)
        {
            return 0;
        }
        // Use the unpack function to convert the first 4 bytes into a number, which represents the entire data packet length
        $unpack_data = unpack('Ntotal_length', $recv_buffer);
        return $unpack_data['total_length'];
    }

    public static function decode($recv_buffer)
    {
        // Remove the first 4 bytes to get the body JSON data
        $body_json_str = substr($recv_buffer, 4);
        // JSON decoding
        return json_decode($body_json_str, true);
    }

    public static function encode($data)
    {
        // JSON encode to get the body
        $body_json_str = json_encode($data);
        // Calculate the length of the entire packet, 4 bytes for the header + body length
        $total_length = 4 + strlen($body_json_str);
        // Return the packed data
        return pack('N',$total_length) . $body_json_str;
    }
}
```

## Example Three (Uploading files using binary protocol)

### Protocol Definition
```C
struct
{
  unsigned int total_len;  // Entire packet length, big-endian network byte order
  char         name_len;   // Length of the file name
  char         name[name_len]; // File name
  char         file[total_len - BinaryTransfer::PACKAGE_HEAD_LEN - name_len]; // File data
}
```

### Sample Data Packet
<pre> *****logo.png****************** </pre>

Where the first four \*s represent a network byte order unsigned int data, which cannot be seen, the 5th \* is a byte storing the length of the file name, followed by the file name, followed by the original binary file data

### Protocol Implementation
```php
namespace Protocols;
class BinaryTransfer
{
    // Protocol header length
    const PACKAGE_HEAD_LEN = 5;

    public static function input($recv_buffer)
    {
        // If the length is not enough for a protocol header, continue to wait
        if(strlen($recv_buffer) < self::PACKAGE_HEAD_LEN)
        {
            return 0;
        }
        // Unpack
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // Return packet length
        return $package_data['total_len'];
    }


    public static function decode($recv_buffer)
    {
        // Unpack
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // File name length
        $name_len = $package_data['name_len'];
        // Extract the file name from the data stream
        $file_name = substr($recv_buffer, self::PACKAGE_HEAD_LEN, $name_len);
        // Extract the file binary data from the data stream
        $file_data = substr($recv_buffer, self::PACKAGE_HEAD_LEN + $name_len);
         return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // Encode the data to be sent to the client according to your own needs, here it is simply returned as plain text
        return $data;
    }
}
```

### Server Protocol Usage Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('BinaryTransfer://0.0.0.0:8333');
// Save the file to tmp directory
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### Client File client.php (Uploading files using PHP to simulate a client)

```php
<?php
/** File upload client **/
// Upload address
$address = "127.0.0.1:8333";
// Check the file path parameter for uploading
if(!isset($argv[1]))
{
   exit("use php client.php \$file_path\n");
}
// File path for upload
$file_to_transfer = trim($argv[1]);
// The file to upload does not exist locally
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer not exist\n");
}
// Establish a socket connection
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
// Set to blocking mode
stream_set_blocking($client, 1);
// File name
$file_name = basename($file_to_transfer);
// File name length
$name_len = strlen($file_name);
// File binary data
$file_data = file_get_contents($file_to_transfer);
// Protocol header length is 4 bytes for the packet length + 1 byte for the file name length
$PACKAGE_HEAD_LEN = 5;
// Protocol packet
$package = pack('NC', $PACKAGE_HEAD_LEN  + strlen($file_name) + strlen($file_data), $name_len) . $file_name . $file_data;
// Execute upload
fwrite($client, $package);
// Print the result
echo fread($client, 8192),"\n";
```

### Client Usage Example
Run in the command line ```php client.php <file_path>```

For example, ```php client.php abc.jpg```
## Example Four (File Upload Using Text Protocol)

### Protocol Definition

json plus newline, where json contains the file name and base64_encode encoded file data (which increases the size by 1/3)

### Protocol Sample

{"file_name":"logo.png","file_data":"PD9waHAKLyo......"}\n

Note that the end is a newline character, represented as ```"\n"``` in PHP using double quotation marks.

### Protocol Implementation
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
        // Unpack
        $package_data = json_decode(trim($recv_buffer), true);
        // Retrieve file name
        $file_name = $package_data['file_name'];
        // Retrieve base64_encoded file data
        $file_data = $package_data['file_data'];
        // Decoding base64_encoded data back to original binary file data
        $file_data = base64_decode($file_data);
        // Return data
        return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // Encode the data to be sent to the client according to your own needs. Here it is simply returned as the original text.
        return $data;
    }
}

```

### Server-Side Protocol Usage Example
Explanation: The code is similar to the binary upload approach, allowing for almost no modifications to the business logic to switch protocols.

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('TextTransfer://0.0.0.0:8333');
// Save the file to tmp
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### Client-Side File textclient.php (Using PHP to Simulate Client-Side Upload)
```php
<?php
/** File Upload Client **/
// Upload address
$address = "127.0.0.1:8333";
// Check upload file path parameter
if(!isset($argv[1]))
{
   exit("use php client.php \$file_path\n");
}
// Upload file path
$file_to_transfer = trim($argv[1]);
// The file to be uploaded does not exist locally
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer not exist\n");
}
// Establish a socket connection
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
stream_set_blocking($client, 1);
// File name
$file_name = basename($file_to_transfer);
// File binary data
$file_data = file_get_contents($file_to_transfer);
// Base64 encoding
$file_data = base64_encode($file_data);
// Data package
$package_data = array(
    'file_name' => $file_name,
    'file_data' => $file_data,
);
// Protocol packet json+carriage return
$package = json_encode($package_data)."\n";
// Execute upload
fwrite($client, $package);
// Print result
echo fread($client, 8192),"\n";
```

### Client-Side Usage Example
Run in the command line ```php textclient.php <file_path>```

For example: ```php textclient.php abc.jpg```

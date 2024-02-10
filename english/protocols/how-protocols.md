## How to Customize Protocol

In fact, it is relatively simple to create your own protocol. A simple protocol generally consists of two parts:
 * Identifiers for differentiating data boundaries
 * Data format definition

## An Example

### Protocol Definition
Assuming the identifier for differentiating data boundaries is the newline character "\n" (Note that the request data itself cannot contain newline characters), and the data format is JSON. For example, the following is a request packet that conforms to this rule.

<pre>
{"type":"message","content":"hello"}
</pre>

Note that the request data above has a newline character at the end (represented as **double quotes** "\n" in PHP), indicating the end of a request.

### Implementation Steps
To implement the aforementioned protocol in Workerman, assuming the protocol is named JsonNL and is located in the project MyApp, the following steps are required
1. Place the protocol file in the Protocols folder of the project, for example, the file would be MyApp/Protocols/JsonNL.php
2. Implement the JsonNL class with the namespace `Protocols;`. It must implement three static methods: input, encode, and decode.

Note: Workerman will automatically call these three static methods to implement fragmentation, unpacking, and packing. Refer to the execution process explanation below for the specific process.

### Interaction Process between Workerman and Protocol Class
1. Assuming a client sends a data packet to the server, upon receiving the data (which may be partial), the server immediately calls the protocol's `input` method to check the length of this packet, and the `input` method returns the length value `$length` to the Workerman framework.
2. After obtaining the `$length` value, the Workerman framework checks whether the current data buffer has received data of length `$length`. If not, it continues to wait for data until the length of data in the buffer is not less than `$length`.
3. Once the length of the data in the buffer is sufficient, Workerman will retrieve a data packet of length `$length` from the buffer (i.e., **fragmentation**) and then call the protocol's `decode` method to **unpack** the data, resulting in `$data`.
4. After unpacking, Workerman passes the data `$data` to the business in the form of a callback `onMessage($connection, $data)`, and the business within `onMessage` can then use the `$data` variable to obtain the complete and unpacked data sent by the client.
5. If the business within `onMessage` needs to send data to the client by calling `$connection->send($buffer)`, Workerman automatically uses the protocol's `encode` method to **pack** the `$buffer` before sending it to the client.

### Specific Implementation

**Implementation of MyApp/Protocols/JsonNL.php**

```php
namespace Protocols;
class JsonNL
{
    /**
     * Check the integrity of the packet
     * If the length of the packet can be obtained in $buffer, then return the length of the entire packet
     * Otherwise, return 0 to continue waiting for data
     * If there is an issue with the protocol, returning false will cause the current client connection to be disconnected
     * @param string $buffer
     * @return int
     */
    public static function input($buffer)
    {
        // Get the position of the newline character "\n"
        $pos = strpos($buffer, "\n");
        
        // If there is no newline character, the length of the packet cannot be determined, so return 0 to continue waiting for data
        if($pos === false)
        {
            return 0;
        }
        
        // If there is a newline character, return the current packet length (including the newline character)
        return $pos+1;
    }

    /**
     * Packing. It will be automatically called when sending data to the client
     * @param string $buffer
     * @return string
     */
    public static function encode($buffer)
    {
        // JSON serialization, and add a newline character as the request end marker
        return json_encode($buffer)."\n";
    }

    /**
     * Unpacking. It will be automatically called when the number of received data bytes equals the value returned by input (greater than 0)
     * and will be passed to the onMessage callback function as the $data parameter
     * @param string $buffer
     * @return string
     */
    public static function decode($buffer)
    {
        // Remove the newline character and convert it back to an array
        return json_decode(trim($buffer), true);
    }
}
```

With this, the JsonNL protocol is fully implemented and can be used in the MyApp project. The usage example is presented as follows:

File: MyApp\start.php

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$json_worker = new Worker('JsonNL://0.0.0.0:1234');
$json_worker->onMessage = function(TcpConnection $connection, $data) {

    // $data contains the data sent by the client, which has already been processed by JsonNL::decode
    echo $data;
    
    // Data sent using $connection->send will automatically be packed using the JsonNL::encode method before being sent to the client
    $connection->send(array('code'=>0, 'msg'=>'ok'));
    
};
Worker::runAll();
...
```

> **Note**
> Workerman will attempt to load protocols under the `Protocols` namespace. For example, `new Worker('JsonNL://0.0.0.0:1234')` will attempt to load the `Protocols\JsonNL` protocol.
> If an error like `Class 'Protocols\JsonNL' not found` occurs, refer to the [autoloading](../faq/autoload.md) section to implement automatic loading.

### Protocol Interface Explanation
The protocol class developed in Workerman must implement three static methods: input, encode, decode. For the protocol interface explanation, refer to Workerman/Protocols/ProtocolInterface.php, which is defined as follows:

```php
namespace Workerman\Protocols;

use \Workerman\Connection\ConnectionInterface;

/**
 * Protocol interface
 * @author walkor <walkor@workerman.net>
 */
interface ProtocolInterface
{
    /**
     * Used for segmenting recv_buffer received in a request
     *
     * If the length of the request packet can be obtained in $recv_buffer, then return the length of the entire packet
     * Otherwise, return 0, indicating that more data is needed to determine the length of the current request packet
     * If false or a negative number is returned, it signifies an erroneous request, and the connection will be disconnected
     *
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     * @return int|false
     */
    public static function input($recv_buffer, ConnectionInterface $connection);

    /**
     * Used for request unpacking
     *
     * When input returns a value greater than 0 and Workerman has received enough data,
     * the decode method is automatically called, and the data is then passed to the onMessage callback as the second parameter
     * This means that when a complete client request is received, decode will be automatically called for decoding, without the need for manual invocation in the business code
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     */
    public static function decode($recv_buffer, ConnectionInterface $connection);

    /**
     * Used for request packing
     *
     * When data needs to be sent to the client by calling $connection->send($data);
     * the $data will be automatically packed using encode into a format that complies with the protocol before being sent to the client
     * This means that the data sent to the client will be automatically encoded, without the need for manual invocation in the business code
     * @param ConnectionInterface $connection
     * @param mixed $data
     */
    public static function encode($data, ConnectionInterface $connection);
}
```

## Note:
Workerman does not strictly require the protocol class to be based on the ProtocolInterface, in practice, as long as the class contains the three static methods input, encode, and decode, it is sufficient.

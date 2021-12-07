## 如何定制协议

实际上制定自己的协议是比较简单的事情。简单的协议一般包含两部分:
 * 区分数据边界的标识
 * 数据格式定义

## 一个例子

### 协议定义
这里假设区分数据边界的标识为换行符"\n"（注意请求数据本身内部不能包含换行符），数据格式为Json，例如下面是一个符合这个规则的请求包。

<pre>
{"type":"message","content":"hello"}

</pre>

注意上面的请求数据末尾有一个换行字符(在PHP中用**双引号**字符串"\n"表示)，代表一个请求的结束。

### 实现步骤
在WorkerMan中如果要实现上面的协议，假设协议的名字叫JsonNL，所在项目为MyApp，则需要以下步骤

1、协议文件放到项目的Protocols文件夹，例如文件MyApp/Protocols/JsonNL.php

2、实现JsonNL类，以```namespace Protocols;```为命名空间，必须实现三个静态方法分别为 input、encode、decode

注意：workerman会自动调用这三个静态方法，用来实现分包、解包、打包。具体流程参考下面执行流程说明。

### workerman与协议类交互流程
1、假设客户端发送一个数据包给服务端，服务端收到数据(可能是部分数据)后会立刻调用协议的```input```方法，用来检测这包的长度，```input```方法返回长度值```$length```给workerman框架。
2、workerman框架得到这个```$length```值后判断当前数据缓冲区中是否已经接收到```$length```长度的数据，如果没有就会继续等待数据，直到缓冲区中的数据长度不小于```$length```。
4、缓冲区的数据长度足够后，workerman就会从缓冲区截取出```$length```长度的数据(即**分包**)，并调用协议的```decode```方法**解包**，解包后的数据为```$data```。
3、解包后workerman将数据```$data```以回调```onMessage($connection, $data)```的形式传递给业务，业务在onMessage里就可以使用```$data```变量得到客户端发来的完整并且已经解包的数据了。
4、当```onMessage```里业务需要通过调用```$connection->send($buffer)```方法给客户端发送数据时，workerman会自动利用协议的```encode```方法将```$buffer```**打包**后再发给客户端。

### 具体实现

**MyApp/Protocols/JsonNL.php的实现**

```php
namespace Protocols;
class JsonNL
{
    /**
     * 检查包的完整性
     * 如果能够得到包长，则返回包的在buffer中的长度，否则返回0继续等待数据
     * 如果协议有问题，则可以返回false，当前客户端连接会因此断开
     * @param string $buffer
     * @return int
     */
    public static function input($buffer)
    {
        // 获得换行字符"\n"位置
        $pos = strpos($buffer, "\n");
        // 没有换行符，无法得知包长，返回0继续等待数据
        if($pos === false)
        {
            return 0;
        }
        // 有换行符，返回当前包长（包含换行符）
        return $pos+1;
    }

    /**
     * 打包，当向客户端发送数据的时候会自动调用
     * @param string $buffer
     * @return string
     */
    public static function encode($buffer)
    {
        // json序列化，并加上换行符作为请求结束的标记
        return json_encode($buffer)."\n";
    }

    /**
     * 解包，当接收到的数据字节数等于input返回的值（大于0的值）自动调用
     * 并传递给onMessage回调函数的$data参数
     * @param string $buffer
     * @return string
     */
    public static function decode($buffer)
    {
        // 去掉换行，还原成数组
        return json_decode(trim($buffer), true);
    }
}
```

至此，JsonNL协议实现完毕，可以在MyApp项目中使用，使用方法例如下面

文件：MyApp\start.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$json_worker = new Worker('JsonNL://0.0.0.0:1234');
$json_worker->onMessage = function(TcpConnection $connection, $data) {

    // $data就是客户端传来的数据，数据已经经过JsonNL::decode处理过
    echo $data;
    
    // $connection->send的数据会自动调用JsonNL::encode方法打包，然后发往客户端
    $connection->send(array('code'=>0, 'msg'=>'ok'));
    
};
Worker::runAll();
...
```

### 协议接口说明
在WorkerMan中开发的协议类必须实现三个静态方法，input、encode、decode，协议接口说明见Workerman/Protocols/ProtocolInterface.php，定义如下：
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
     * 用于在接收到的recv_buffer中分包
     *
     * 如果可以在$recv_buffer中得到请求包的长度则返回整个包的长度
     * 否则返回0，表示需要更多的数据才能得到当前请求包的长度
     * 如果返回false或者负数，则代表错误的请求，则连接会断开
     *
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     * @return int|false
     */
    public static function input($recv_buffer, ConnectionInterface $connection);

    /**
     * 用于请求解包
     *
     * input返回值大于0，并且WorkerMan收到了足够的数据，则自动调用decode
     * 然后触发onMessage回调，并将decode解码后的数据传递给onMessage回调的第二个参数
     * 也就是说当收到完整的客户端请求时，会自动调用decode解码，无需业务代码中手动调用
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     */
    public static function decode($recv_buffer, ConnectionInterface $connection);

    /**
     * 用于请求打包
     *
     * 当需要向客户端发送数据即调用$connection->send($data);时
     * 会自动把$data用encode打包一次，变成符合协议的数据格式，然后再发送给客户端
     * 也就是说发送给客户端的数据会自动encode打包，无需业务代码中手动调用
     * @param ConnectionInterface $connection
     * @param mixed $data
     */
    public static function encode($data, ConnectionInterface $connection);
}
```

## 注意：
Workerman中没有严格要求协议类必须基于ProtocolInterface实现，实际上协议类只要类包含了input、encode、decode三个静态方法即可。










# 一些例子

## 例子一

### 协议定义
  * 首部固定10个字节长度用来保存整个数据包长度，位数不够补0
  * 数据格式为xml

### 数据包样本
```xml
0000000121<?xml version="1.0" encoding="ISO-8859-1"?>
<request>
    <module>user</module>
    <action>getInfo</action>
</request>
```
其中0000000121代表整个数据包长度，后面紧跟xml数据格式的包体内容

### 协议实现
```php
namespace Protocols;
class XmlProtocol
{
    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer) < 10)
        {
            // 不够10字节，返回0继续等待数据
            return 0;
        }
        // 返回包长，包长包含 头部数据长度+包体长度
        $total_len = base_convert($recv_buffer, 10, 10);
        return $total_len;
    }

    public static function decode($recv_buffer)
    {
        // 请求包体
        $body = substr($recv_buffer, 10);
        return simplexml_load_string($body);
    }

    public static function encode($xml_string)
    {
        // 包体+包头的长度
        $total_length = strlen($xml_string)+10;
        // 长度部分凑足10字节，位数不够补0
        $total_length_str = str_pad($total_length, 10, '0', STR_PAD_LEFT);
        // 返回数据
        return $total_length_str . $xml_string;
    }
}
```

## 例子二

### 协议定义
  * 首部4字节网络字节序unsigned int，标记整个包的长度
  * 数据部分为Json字符串

### 数据包样本
<pre>
****{"type":"message","content":"hello all"}
</pre>

其中首部四字节*号代表一个网络字节序的unsigned int数据，为不可见字符，紧接着是Json的数据格式的包体数据

### 协议实现
```php
namespace Protocols;
class JsonInt
{
    public static function input($recv_buffer)
    {
        // 接收到的数据还不够4字节，无法得知包的长度，返回0继续等待数据
        if(strlen($recv_buffer)<4)
        {
            return 0;
        }
        // 利用unpack函数将首部4字节转换成数字，首部4字节即为整个数据包长度
        $unpack_data = unpack('Ntotal_length', $recv_buffer);
        return $unpack_data['total_length'];
    }

    public static function decode($recv_buffer)
    {
        // 去掉首部4字节，得到包体Json数据
        $body_json_str = substr($recv_buffer, 4);
        // json解码
        return json_decode($body_json_str, true);
    }

    public static function encode($data)
    {
        // Json编码得到包体
        $body_json_str = json_encode($data);
        // 计算整个包的长度，首部4字节+包体字节数
        $total_length = 4 + strlen($body_json_str);
        // 返回打包的数据
        return pack('N',$total_length) . $body_json_str;
    }
}
```

## 例子三（使用二进制协议上传文件）
### 协议定义
```C
struct
{
  unsigned int total_len;  // 整个包的长度，大端网络字节序
  char         name_len;   // 文件名的长度
  char         name[name_len]; // 文件名
  char         file[total_len - BinaryTransfer::PACKAGE_HEAD_LEN - name_len]; // 文件数据
}
```
### 协议样本

<pre> *****logo.png****************** </pre>

其中首部四字节\*号代表一个网络字节序的unsigned int数据，为不可见字符，第5个\*是用一个字节存储文件名长度，紧接着是文件名，接着是原始的二进制文件数据

### 协议实现
```php
namespace Protocols;
class BinaryTransfer
{
    // 协议头长度
    const PACKAGE_HEAD_LEN = 5;

    public static function input($recv_buffer)
    {
        // 如果不够一个协议头的长度，则继续等待
        if(strlen($recv_buffer) < self::PACKAGE_HEAD_LEN)
        {
            return 0;
        }
        // 解包
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // 返回包长
        return $package_data['total_len'];
    }


    public static function decode($recv_buffer)
    {
        // 解包
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // 文件名长度
        $name_len = $package_data['name_len'];
        // 从数据流中截取出文件名
        $file_name = substr($recv_buffer, self::PACKAGE_HEAD_LEN, $name_len);
        // 从数据流中截取出文件二进制数据
        $file_data = substr($recv_buffer, self::PACKAGE_HEAD_LEN + $name_len);
         return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // 可以根据自己的需要编码发送给客户端的数据，这里只是当做文本原样返回
        return $data;
    }
}
```

### 服务端协议使用示例

```php
use Workerman\Worker;
require_once '/your/path/Workerman/Autoloader.php';

$worker = new Worker('BinaryTransfer://0.0.0.0:8333');
// 保存文件到tmp下
$worker->onMessage = function($connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### 客户端文件 client.php （这里用php模拟客户端上传）
```php
<?php
/** 上传文件客户端 **/
// 上传地址
$address = "127.0.0.1:8333";
// 检查上传文件路径参数
if(!isset($argv[1]))
{
   exit("use php client.php \$file_path\n");
}
// 上传文件路径
$file_to_transfer = trim($argv[1]);
// 上传的文件本地不存在
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer not exist\n");
}
// 建立socket连接
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
// 设置成阻塞
stream_set_blocking($client, 1);
// 文件名
$file_name = basename($file_to_transfer);
// 文件名长度
$name_len = strlen($file_name);
// 文件二进制数据
$file_data = file_get_contents($file_to_transfer);
// 协议头长度 4字节包长+1字节文件名长度
$PACKAGE_HEAD_LEN = 5;
// 协议包
$package = pack('NC', $PACKAGE_HEAD_LEN  + strlen($file_name) + strlen($file_data), $name_len) . $file_name . $file_data;
// 执行上传
fwrite($client, $package);
// 打印结果
echo fread($client, 8192),"\n";
```

### 客户端使用示例
命令行中运行 ```php client.php <文件路径>```

例如 ```php client.php abc.jpg```


## 例子四（使用文本协议上传文件）

### 协议定义

json+换行，json中包含了文件名以及base64_encode编码（会增大1/3的体积）的文件数据


### 协议样本

{"file_name":"logo.png","file_data":"PD9waHAKLyo......"}\n

注意末尾为一个换行符，在PHP中用双引号字符```"\n"```标识

### 协议实现
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
        // 解包
        $package_data = json_decode(trim($recv_buffer), true);
        // 取出文件名
        $file_name = $package_data['file_name'];
        // 取出base64_encode后的文件数据
        $file_data = $package_data['file_data'];
        // base64_decode还原回原来的二进制文件数据
        $file_data = base64_decode($file_data);
        // 返回数据
        return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // 可以根据自己的需要编码发送给客户端的数据，这里只是当做文本原样返回
        return $data;
    }
}

```

### 服务端协议使用示例
说明：写法与二进制上传写法一样，即能做到几乎不用改动任何业务代码便可以切换协议

```php
use Workerman\Worker;
require_once '/your/path/Workerman/Autoloader.php';

$worker = new Worker('TextTransfer://0.0.0.0:8333');
// 保存文件到tmp下
$worker->onMessage = function($connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### 客户端文件 textclient.php （这里用php模拟客户端上传）
```php
<?php
/** 上传文件客户端 **/
// 上传地址
$address = "127.0.0.1:8333";
// 检查上传文件路径参数
if(!isset($argv[1]))
{
   exit("use php client.php \$file_path\n");
}
// 上传文件路径
$file_to_transfer = trim($argv[1]);
// 上传的文件本地不存在
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer not exist\n");
}
// 建立socket连接
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
stream_set_blocking($client, 1);
// 文件名
$file_name = basename($file_to_transfer);
// 文件二进制数据
$file_data = file_get_contents($file_to_transfer);
// base64编码
$file_data = base64_encode($file_data);
// 数据包
$package_data = array(
    'file_name' => $file_name,
    'file_data' => $file_data,
);
// 协议包 json+回车
$package = json_encode($package_data)."\n";
// 执行上传
fwrite($client, $package);
// 打印结果
echo fread($client, 8192),"\n";
```

### 客户端使用示例
命令行中运行 ```php textclient.php <文件路径>```

例如 ```php textclient.php abc.jpg```



# いくつかの例

## 例1

### プロトコルの定義
  * ヘッダーは10バイトの固定長で、パケット全体の長さを保存し、桁が足りない場合は0で埋める
  * データ形式はXML

### データパケットの例
```xml
0000000121<?xml version="1.0" encoding="ISO-8859-1"?>
<request>
    <module>user</module>
    <action>getInfo</action>
</request>
```
ここで、0000000121はパケット全体の長さを表し、その後にXMLデータ形式の本体が続きます。

### プロトコルの実装
```php
namespace Protocols;
class XmlProtocol
{
    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer) < 10)
        {
            // 10バイトに満たない場合、データを待機するために0を返す
            return 0;
        }
        // パケット長を返す。ヘッダーデータ長+パケット本体の長さが含まれている
        $total_len = base_convert(substr($recv_buffer, 0, 10), 10, 10);
        return $total_len;
    }

    public static function decode($recv_buffer)
    {
        // リクエストパケットの本体
        $body = substr($recv_buffer, 10);
        return simplexml_load_string($body);
    }

    public static function encode($xml_string)
    {
        // 本体+ヘッダーの長さ
        $total_length = strlen($xml_string)+10;
        // 長さが10バイトに満たない場合、0で埋める
        $total_length_str = str_pad($total_length, 10, '0', STR_PAD_LEFT);
        // データを返す
        return $total_length_str . $xml_string;
    }
}
```

## 例2

### プロトコルの定義
  * 4バイトのネットワークバイトオーダーの符号なし整数で、パケット全体の長さを示す
  * データ部分はJSON文字列

### データパケットの例
<pre>
****{"type":"message","content":"hello all"}
</pre>

ここで、最初の4バイトの*はネットワークバイトオーダーの符号なし整数データを表し、可視化できない文字です。その後にJSONフォーマットのデータ本体が続きます。

### プロトコルの実装
```php
namespace Protocols;
class JsonInt
{
    public static function input($recv_buffer)
    {
        // 受信したデータが4バイトに満たないため、パケットの長さが分からないので0を返してデータを待機する
        if(strlen($recv_buffer)<4)
        {
            return 0;
        }
        // unpack関数を使用して最初の4バイトを数字に変換し、これがパケットの全体の長さになる
        $unpack_data = unpack('Ntotal_length', $recv_buffer);
        return $unpack_data['total_length'];
    }

    public static function decode($recv_buffer)
    {
        // 最初の4バイトを取り除いて、パケット本体のJSONデータを取得する
        $body_json_str = substr($recv_buffer, 4);
        // JSONデコード
        return json_decode($body_json_str, true);
    }

    public static function encode($data)
    {
        // JSONエンコードして本体を取得
        $body_json_str = json_encode($data);
        // パケット全体の長さを計算する。4バイトのヘッダー+本体のバイト数
        $total_length = 4 + strlen($body_json_str);
        // パックされたデータを返す
        return pack('N',$total_length) . $body_json_str;
    }
}
```

## 例3（バイナリプロトコルを使用したファイルのアップロード）

### プロトコルの定義
```C
struct
{
  unsigned int total_len;  // パケット全体の長さ、ビッグエンディアン
  char         name_len;   // ファイル名の長さ
  char         name[name_len]; // ファイル名
  char         file[total_len - BinaryTransfer::PACKAGE_HEAD_LEN - name_len]; // ファイルデータ
}
```
### プロトコルの例

<pre> *****logo.png****************** </pre>

ここで、最初の4バイトの\*はビッグエンディアンの符号なし整数データを表し、可視化できない文字です。5番目の\*は1バイトで、ファイル名の長さを示し、後にファイル名のオリジナルのバイナリデータが続きます。

### プロトコルの実装
```php
namespace Protocols;
class BinaryTransfer
{
    // プロトコルのヘッダー長
    const PACKAGE_HEAD_LEN = 5;

    public static function input($recv_buffer)
    {
        // プロトコルのヘッダーの長さに満たない場合、データを待機する
        if(strlen($recv_buffer) < self::PACKAGE_HEAD_LEN)
        {
            return 0;
        }
        // アンパック
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // パケット全体の長さを返す
        return $package_data['total_len'];
    }


    public static function decode($recv_buffer)
    {
        // アンパック
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        // ファイル名の長さ
        $name_len = $package_data['name_len'];
        // データストリームからファイル名を切り出す
        $file_name = substr($recv_buffer, self::PACKAGE_HEAD_LEN, $name_len);
        // データストリームからバイナリデータを切り出す
        $file_data = substr($recv_buffer, self::PACKAGE_HEAD_LEN + $name_len);
         return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // クライアントに送信するデータを自分の必要に応じてエンコードできますが、ここではテキストをそのまま返しています
        return $data;
    }
}
```

### サーバープロトコルの使用例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('BinaryTransfer://0.0.0.0:8333');
// tmpフォルダにファイルを保存
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### クライアントファイル client.php（ここではPHPを使用してクライアントを模倣してファイルをアップロードしています）
```php
<?php
/** ファイルアップロードクライアント **/
// アップロード先のアドレス
$address = "127.0.0.1:8333";
// アップロードファイルのパスのパラメータをチェック
if(!isset($argv[1]))
{
   exit("use php client.php \$file_path\n");
}
// アップロードファイルのパス
$file_to_transfer = trim($argv[1]);
// アップロードするファイルが存在しない
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer not exist\n");
}
// ソケット接続を確立
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
// ブロッキングモードに設定
stream_set_blocking($client, 1);
// ファイル名
$file_name = basename($file_to_transfer);
// ファイル名の長さ
$name_len = strlen($file_name);
// バイナリデータ
$file_data = file_get_contents($file_to_transfer);
// プロトコルヘッダーの長さ 4バイトのパケット長+1バイトのファイル名の長さ
$PACKAGE_HEAD_LEN = 5;
// プロトコルパケット
$package = pack('NC', $PACKAGE_HEAD_LEN  + strlen($file_name) + strlen($file_data), $name_len) . $file_name . $file_data;
// アップロードの実行
fwrite($client, $package);
// 結果を表示
echo fread($client, 8192),"\n";
```

### クライアントの使用例
コマンドラインで次のように実行します ```php client.php <ファイルパス>```

例 ```php client.php abc.jpg```
## 例4（テキストプロトコルを使用してファイルをアップロードする）

### プロトコルの定義

json + 改行で、jsonにはファイル名とbase64_encodeでエンコードされた（サイズが1/3増加する）ファイルデータが含まれています。

### プロトコルサンプル

{"file_name":"logo.png","file_data":"PD9waHAKLyo......"}\n

末尾には改行文字があり、PHPでは二重引用符```"\n"```で示されます。

### プロトコルの実装
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
        // パッケージのデコード
        $package_data = json_decode(trim($recv_buffer), true);
        // ファイル名の取得
        $file_name = $package_data['file_name'];
        // base64_encodeでエンコードされたファイルデータの取得
        $file_data = $package_data['file_data'];
        // base64_decodeを使用して元のバイナリファイルデータに戻す
        $file_data = base64_decode($file_data);
        // データを返す
        return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // 必要に応じてクライアントに送信するデータをエンコードします。ここではテキストをそのまま返します
        return $data;
    }
}

```
### サーバーサイドでのプロトコルの使用例
説明：バイナリアップロードと同様の方法で書くことができます。つまり、ほとんどのビジネスコードを変更することなくプロトコルを切り替えることができます。

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('TextTransfer://0.0.0.0:8333');
// tmpにファイルを保存
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```
### クライアント側のファイル textclient.php (ここではPHPでクライアントアップロードをシミュレートします)
```php
<?php
/** ファイルをアップロードするクライアント **/
// アップロード先アドレス
$address = "127.0.0.1:8333";
// アップロードファイルのパスパラメータの確認
if(!isset($argv[1]))
{
   exit("use php client.php \$file_path\n");
}
// アップロードファイルのパス
$file_to_transfer = trim($argv[1]);
// アップロードするファイルが存在しない
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer not exist\n");
}
// ソケット接続を確立
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
stream_set_blocking($client, 1);
// ファイル名
$file_name = basename($file_to_transfer);
// ファイルのバイナリデータ
$file_data = file_get_contents($file_to_transfer);
// base64エンコード
$file_data = base64_encode($file_data);
// パッケージデータ
$package_data = array(
    'file_name' => $file_name,
    'file_data' => $file_data,
);
// プロトコルパケット json + 改行
$package = json_encode($package_data)."\n";
// アップロードの実行
fwrite($client, $package);
// 結果を出力
echo fread($client, 8192),"\n";
```
### クライアントの使用例
コマンドラインで実行 ```php textclient.php <ファイルパス>```

例 ```php textclient.php abc.jpg```

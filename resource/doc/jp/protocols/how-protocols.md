## プロトコルのカスタマイズ方法

実際には、独自のプロトコルを定義することは比較的簡単です。一般的に、シンプルなプロトコルには2つの部分が含まれます:
 * データ境界を区別する識別子
 * データの形式を定義する部分

## 例

### プロトコルの定義
ここでは、データ境界を区別する識別子を改行文字"\n"とし、データ形式をJsonとして定義します。以下はこのルールに準拠したリクエストパケットの例です。

<pre>
{"type":"message","content":"hello"}
</pre>

上記のリクエストデータの最後に改行文字(これはPHPでは**ダブルクオート**文字列"\n"として表されます)が含まれており、リクエストの終了を表しています。

### 実装手順
Workermanでは、上記のプロトコルを実装する場合、プロトコルの名前をJsonNLとし、プロジェクトがMyAppであると仮定します。その場合、以下の手順が必要です。

1、プロトコルファイルをプロジェクトのProtocolsディレクトリに配置します。例えば、ファイルMyApp/Protocols/JsonNL.phpを配置します。

2、JsonNLクラスを実装します。これは```namespace Protocols;```を名前空間として含める必要があり、input、encode、decodeの3つの静的メソッドを実装する必要があります。

注意：workermanはこれらの3つの静的メソッドを自動的に呼び出し、パッケージの分割、パッケージの展開、パッケージの作成を実装します。具体的なフローについては、以下の実行フローの説明を参照してください。

### workermanとプロトコルクラスのやり取りフロー
1、クライアントがサーバにデータパケットを送信すると、サーバはデータ（一部の可能性があります）を受信するとすぐに、プロトコルの```input```メソッドを呼び出し、このパケットの長さを検査します。```input```メソッドは長さ値```$length```をworkermanフレームワークに返します。
2、workermanフレームワークはこの```$length```値を受け取ると、現在のデータバッファに既に```$length```の長さのデータが受信されているかどうかを判断し、受信されていない場合はデータをさらに待機し、データバッファ内のデータ長が```$length```未満になるまで待機します。
3、データバッファの長さが十分ある場合、workermanはデータバッファから```$length```の長さのデータ（つまり**パッケージの分割**）を切り取り、プロトコルの```decode```メソッドを呼び出して**パッケージの展開**を行います。展開後のデータは```$data```に格納されます。
4、展開後、workermanはデータ```$data```をコールバック```onMessage($connection, $data)```の形式でビジネスに渡します。ビジネス側はonMessage内で```$data```変数を使用してクライアントからの完全で解析されたデータを取得できます。
5、```onMessage```内でビジネスが```$connection->send($buffer)```を呼び出してクライアントにデータを送信する場合、workermanは自動的にプロトコルの```encode```メソッドを使用して```$buffer```を**パッケージング**した後、クライアントに送信します。

### 具体的な実装

**MyApp/Protocols/JsonNL.phpの実装**

```php
namespace Protocols;
class JsonNL
{
    /**
     * パケットの完全性を確認する
     * パケットの長さが得られる場合はパケットのバッファ内の長さを返し、それ以外の場合は0を返してさらなるデータを待ちます
     * 問題がある場合はfalseを返し、現在のクライアントの接続が切断されます
     * @param string $buffer
     * @return int
     */
    public static function input($buffer)
    {
        // 改行文字"\n"の位置を取得
        $pos = strpos($buffer, "\n");
        // 改行がない場合はパケットの長さが分かりませんので、さらなるデータを待ちます
        if($pos === false)
        {
            return 0;
        }
        // 改行がある場合は、現在のパケットの長さ（改行を含む）を返します
        return $pos+1;
    }

    /**
     * パッケージング、クライアントにデータを送信する際に自動的に呼び出されます
     * @param string $buffer
     * @return string
     */
    public static function encode($buffer)
    {
        // Jsonデータをシリアライズし、終了を表すために改行文字を追加します
        return json_encode($buffer)."\n";
    }

    /**
     * 展開、受信したデータのバイト数がinputで返された値（0より大きい値）と等しい場合に自動的に呼び出され、
     * onMessageコールバック関数の$dataパラメータに渡されます
     * @param string $buffer
     * @return string
     */
    public static function decode($buffer)
    {
        // 改行を削除し、配列に戻します
        return json_decode(trim($buffer), true);
    }
}
```

以上で、JsonNLプロトコルの実装が完了しました。MyAppプロジェクトで使用することができます。以下は使用方法の例です。

ファイル：MyApp\start.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$json_worker = new Worker('JsonNL://0.0.0.0:1234');
$json_worker->onMessage = function(TcpConnection $connection, $data) {

    // $dataはクライアントから送られてきたデータで、データは既にJsonNL::decodeで処理されています
    echo $data;
    
    // $connection->sendのデータは自動的にJsonNL::encodeメソッドを使用してパッケージングされ、それからクライアントに送信されます
    $connection->send(array('code'=>0, 'msg'=>'ok'));
    
};
Worker::runAll();
...
```

> **注意**
> workermanは`Protocols`名前空間内のプロトコルを読み込もうとします。例えば、`new Worker('JsonNL://0.0.0.0:1234')`は、`Protocols\JsonNL`プロトコルを読み込もうとします。
> `Class 'Protocols\JsonNL' not found`というエラーが発生した場合は、[自動読み込み](../faq/autoload.md)を参照して自動ロードを実装してください。

### プロトコルインターフェースの説明
Workermanで開発されたプロトコルクラスは、input、encode、decodeの3つの静的メソッドを実装する必要があります。プロトコルインターフェースの詳細については、Workerman/Protocols/ProtocolInterface.phpを参照するか、以下のように定義されます。

```php
namespace Workerman\Protocols;

use \Workerman\Connection\ConnectionInterface;

/**
 * プロトコルインターフェース
* @author walkor <walkor@workerman.net>
 */
interface ProtocolInterface
{
    /**
     * 受信したrecv_buffer内でパッケージングするために使用されます
     *
     * $recv_buffer内でリクエストパケットの長さが取得できる場合は、完全なパケットの長さを返します
     * そうでない場合は0を返し、現在のリクエストパケットの長さを得るためにさらにデータが必要であることを示します
     * falseまたは負の数を返すと、無効なリクエストを示し、接続が切断されます
     *
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     * @return int|false
     */
    public static function input($recv_buffer, ConnectionInterface $connection);

    /**
     * リクエストの展開に使用されます
     *
     * inputの戻り値が0より大きく、Workermanが十分なデータを受信した場合、自動的にdecodeを呼び出し、メッセージコールバックをトリガーし、decodeで解析されたデータがonMessageコールバックの第2引数に渡されます
     * つまり、クライアントリクエストパケットを完全に受信したときに自動的にdecode解析が呼び出され、ビジネスコードで手動で呼び出す必要はありません
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     */
    public static function decode($recv_buffer, ConnectionInterface $connection);

    /**
     * リクエストパッケージの作成に使用されます
     *
     * クライアントにデータを送信する必要がある場合、$connection->send($data);を自動的にencodeでデータをパッケージングし、プロトコルに適したデータ形式に変換します
     * つまり、クライアントに送信されるデータは自動的にencodeでパッケージ化されるため、ビジネスコードで手動で呼び出す必要はありません。
     * @param ConnectionInterface $connection
     * @param mixed $data
     */
    public static function encode($data, ConnectionInterface $connection);
}
```

## 注意：
Workermanでは、厳密にプロトコルクラスがProtocolInterfaceに基づいて実装される必要はありません。実際、プロトコルクラスには、input、encode、decodeの3つの静的メソッドが含まれていれば十分です。

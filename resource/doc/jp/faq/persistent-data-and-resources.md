# オブジェクトおよびリソースの永続化
従来のWeb開発では、PHPで作成したオブジェクト、データ、リソースなどはリクエスト完了後にすべて解放され、永続化が困難でした。しかし、Workermanではこれらを簡単に実現できます。

Workermanでは、特定のデータリソースをメモリ中に永続的に保存するには、リソースをグローバル変数またはクラスの静的メンバに配置することができます。

たとえば、以下のコードでは、

現在のプロセスのクライアント接続数をグローバル変数```$connection_count```に保存しています。

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// グローバル変数、現在のプロセスのクライアント接続数を保存
$connection_count = 0;

$worker = new Worker('tcp://0.0.0.0:1236');

$worker->onConnect = function(TcpConnection $connection)
{
    // 新しいクライアント接続がある場合、接続数を+1
    global $connection_count;
    ++$connection_count;
    echo "now connection_count=$connection_count\n";
};

$worker->onClose = function(TcpConnection $connection)
{
    // クライアントが切断された場合、接続数を-1
    global $connection_count;
    $connection_count--;
    echo "now connection_count=$connection_count\n";
};

```

## PHP変数のスコープについては、以下を参照してください：
https://php.net/manual/zh/language.variables.scope.php

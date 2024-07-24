# ハートビート

注意：長期の接続アプリケーションでは、ハートビートを追加する必要があります。そうでないと、通信が長時間ないことにより、ルーティングノードによって強制的に切断される可能性があります。

ハートビートには主に2つの目的があります：
1. クライアントは定期的にサーバーにデータを送信することで、通信がない長い時間によって一部のノードのファイアウォールによって切断されることを防ぎます。
2. サーバーはハートビートを使用して、クライアントがオンラインであるかどうかを判断できます。クライアントが一定時間内にデータを送信しない場合、クライアントをオフラインと見なします。これにより、極端な状況（停電、ネットワーク障害など）によるクライアントのオフラインを検出できます。

ハートビート間隔の推奨値：
クライアントがハートビートを送信する間隔は60秒未満であることが推奨されます。例えば、55秒です。

> ハートビートのデータ形式には制限がありません。サーバーが認識できれば良いです。

## ハートビートの例
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// ハートビート間隔は55秒
define('HEARTBEAT_TIME', 55);

$worker = new Worker('text://0.0.0.0:1234');

$worker->onMessage = function(TcpConnection $connection, $msg) {
    // connectionに一時的なlastMessageTimeプロパティを設定し、前回メッセージを受信した時間を記録する
    $connection->lastMessageTime = time();
    // 他のビジネスロジック...
};

// プロセスが起動した後、10秒ごとに実行されるタイマーを設定
$worker->onWorkerStart = function($worker) {
    Timer::add(10, function()use($worker){
        $time_now = time();
        foreach($worker->connections as $connection) {
            // この接続がまだメッセージを受信していない可能性があるため、lastMessageTimeを現在の時刻に設定する
            if (empty($connection->lastMessageTime)) {
                $connection->lastMessageTime = $time_now;
                continue;
            }
            // 前回の通信からの経過時間がハートビート間隔を超える場合、クライアントはすでにオフラインと見なされ、接続は閉じられます。
            if ($time_now - $connection->lastMessageTime > HEARTBEAT_TIME) {
                $connection->close();
            }
        }
    });
};

Worker::runAll();
```

上記の設定では、クライアントが55秒間データをサーバーに送信しない場合、サーバーはクライアントがオフラインと見なし、接続を閉じてonCloseをトリガします。

## 再接続（重要）

クライアントがハートビートを送信するか、サーバーがハートビートを送信するかに関係なく、接続は切断される可能性があります。たとえば、ブラウザが最小化され、JavaScriptが一時停止される、ブラウザが他のタブに切り替わったり、コンピューターがスリープモードに入ったりする場合、モバイル端末がネットワークを切り替えたり、信号が弱くなったり、携帯電話がスリープモードに入ったり、モバイルアプリがバックグラウンドに切り替わったり、ルーターの障害、ビジネスによる切断などです。特に外部ネットワーク環境では、多くのルーティングノードは非アクティブな接続を1分ごとにクリアすることがあります。これが、ハートビート間隔が1分未満であることが推奨される理由でもあります。

外部ネットワーク環境では、接続が簡単に切断されるため、再接続は長時間の接続アプリケーションに必要な機能です（再接続はクライアントのみが行うことができ、サーバーは実装できません）。たとえば、ブラウザのWebsocketはoncloseイベントを監視し、oncloseが発生した場合、新しい接続を確立します（クラッシュを回避するために接続の延長が必要です）。より厳格には、サーバーも定期的にハートビートデータを送信し、クライアントは定期的にサーバーのハートビートデータのタイムアウトを監視し、指定された時間内にサーバーのハートビートデータを受信しない場合、接続が切断されたと見なし、closeを実行し、新しい接続を確立する必要があります。
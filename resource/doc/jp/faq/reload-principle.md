# スムーズな再起動の原理
## スムーズな再起動とは？

通常の再起動とは異なり、スムーズな再起動はユーザーに影響を与えずにサービス（通常は短い接続ビジネスを指す）を再起動し、PHPプログラムを再読み込んでビジネスコードの更新を完了することができます。

スムーズな再起動は、ビジネスの更新やバージョンのリリースプロセスで通常のコードのリリースによる一時的なサービスの使用停止を避けることができます。

> **注意**
> Windowsシステムではreloadはサポートされていません。

> **注意**
> 長期接続（例：websocket）ビジネスでは、プロセスのスムーズな再起動時に接続が切断されます。解決策は、[gatewayWorker](https://www.workerman.net/doc/gateway-worker)のようなアーキテクチャを使用し、接続を維持するための一連のプロセスを使用し、このプロセスの[reloadable](../worker/reloadable.md)特性をfalseに設定します。ビジネスロジックは他の一連のworkerプロセスを起動して処理し、gatewayとworkerプロセスはtcp通信を使用して相互に呼び出します。ビジネスの変更が必要な場合、workerプロセスを再起動するだけで済みます。

## 制限
**注意：on{...}コールバック内でロードされたファイルのみがスムーズな再起動後に自動的に更新されます。起動スクリプトで直接ロードされるファイルやハードコーディングされたコードは、リロードしても自動的に更新されません。**

#### 以下のコードはリロード後に更新されません
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onMessage = function($connection, $request) {
    $connection->send('hi'); // ハードコーディングされたコードはホットリロードをサポートしていません
};
```

```php
$worker = new Worker('http://0.0.0.0:1234');
require_once __DIR__ . '/your/path/MessageHandler.php'; // 起動スクリプトで直接ロードされたファイルはホットリロードをサポートしていません
$messageHandler = new MessageHandler();
$worker->onMessage = [$messageHandler, 'onMessage']; // MessageHandlerクラスにonMessageメソッドがあると仮定します
```

#### 以下のコードはリロード後に自動的に更新されます
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onWorkerStart = function($worker) { // onWorkerStartはプロセス起動後にトリガーされるコールバックです
    require_once __DIR__ . '/your/path/MessageHandler.php'; // プロセス起動後にロードされるファイルはホットリロードをサポートします
    $messageHandler = new MessageHandler();
    $worker->onMessage = [$messageHandler, 'onMessage'];
};
```
MessageHandler.phpを変更した後に `php start.php reload` を実行すると、MessageHandler.phpがメモリに再度ロードされてビジネスロジックが更新されます。

> **ヒント**
> 上記のコードはデモンストレーションのために`require_once`ステートメントを使用していますが、プロジェクトがpsr4自動ロードをサポートしている場合は、`require_once`ステートメントを呼び出す必要はありません。

## スムーズな再起動の原理

Workermanは、親プロセスと子プロセスに分かれており、親プロセスは子プロセスを監視し、子プロセスはクライアントの接続と接続からのリクエストデータを受信し、適切な処理を行い、クライアントにデータを返します。ビジネスコードが更新される場合、実際には子プロセスを更新するだけでコードの更新が可能です。

Workermanの親プロセスはスムーズな再起動シグナルを受信すると、親プロセスは安全な終了（対応するプロセスが現在のリクエストを処理し終えるまで終了しない）シグナルを1つの子プロセスに送信します。そのプロセスが終了すると、親プロセスは新しいプロセスを作成します（このプロセスは新しいPHPコードを読み込んでいます）、次に親プロセスは別の古いプロセスに停止コマンドを再び送信し、このように古いプロセスを1つずつ再起動し、すべての古いプロセスが置き換えられるまで続けます。

スムーズな再起動では、ユーザーに影響を与えないようにするために、プロセス内でユーザーに関連する状態情報を保存しないことが求められます。つまり、ビジネスプロセスは可能な限りステートレスであるべきであり、プロセスの終了による情報の損失を防ぐ必要があります。

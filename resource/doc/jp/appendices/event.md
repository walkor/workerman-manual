# 現在サポートされているWorkermanのイベントドライバー

| 名前  | 依存拡張 | コルーチン対応 | 優先度 | Workermanバージョン |
|-----|------|--|-----|
|  Workerman\Events\Select   |   なし   | 対応していない  |  デフォルト   |  >=3.0  ｜
|  Workerman\Events\Revolt   |   event(オプション)   | 対応 |  [revolt/event-loop](https://github.com/revoltphp/event-loop)のインストールが必要   |  >=5.0  |
|  Workerman\Events\Event   |   event   | 対応していない |  デフォルト   |  >=3.0  |
|  Workerman\Events\Swoole   |  [swoole](https://github.com/swoole/swoole-src)   | 対応 |  手動で設定が必要   |  >=4.0  |
|  Workerman\Events\Swow   |   [swow](https://github.com/swow/swow)   | 対応 |  手動で設定が必要   |  >=5.0  |

* 各種の内部ドライバーはそれぞれ独自の機能を提供します。例えば`Revolt`を使用すると、WorkermanはPHPの組み込み[Fiber協程(ファイバー)](https://www.php.net/manual/zh/language.fibers.php)をサポートします。`Swoole`を使用すると、WorkermanはSwooleのコルーチンをサポートします。
* 各イベントドライバーは排他的な関係にあります。たとえば`Revolt`のFiberコルーチンを使用する場合、SwooleやSwowのコルーチンを使用することはできません。
* `Revolt`を使用する場合は、`composer require revolt/event-loop ^1.0.0` をインストールし、インストール後にWorkermanの内部ドライバーが自動的に優先されるように設定されます。
* `Swoole`および`Swow`は`Worker::$eventLoopClass`を手動で設定する必要があります（次の段落を参照）。
* Swooleはデフォルトで[一键協程Runtime](https://wiki.swoole.com/#/runtime?id=runtime)が有効になっていません。つまり、Pdo、Redis、PHPの組み込みファイルの読み書きは引き続きブロッキング呼び出しです。
* Swooleの一键協程を有効にするには、 `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);` を手動で呼び出す必要があります。

> **注意**
> Swow拡張機能はPHPの組み込み関数の動作を自動的に変更するため、Swow拡張機能を有効にしているがイベントドライバーとしてSwowを使用していない場合、Workermanはリクエストやシグナルに応答できません。そのため、Swowを内部ドライバーとして使用しない場合は、php.iniからSwowをコメントアウトする必要があります。

詳細は[workerman協程](../fiber.md)を参照してください。

# Workermanのイベントドライバーを設定する

以下はWorkermanでイベントドライバーを手動で設定する方法です。

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// バックエンドイベントドライバーを手動で設定
Worker::$eventLoopClass = Workerman\Events\Revolt::class;
//Worker::$eventLoopClass = Workerman\Events\Select::class;
//Worker::$eventLoopClass = Workerman\Events\Event::class;
//Worker::$eventLoopClass = Workerman\Events\Swoole::class;
//Worker::$eventLoopClass = Workerman\Events\Swow::class;
$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    Coroutine::create(function() use ($connection) {
        $cli = new Client('example.com', 80);
        $cli->get('/get');
        $connection->send($cli->body);
    });
};

Worker::runAll();
```

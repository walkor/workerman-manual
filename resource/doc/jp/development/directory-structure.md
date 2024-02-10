# ディレクトリ構造
```
Workerman                      // Workermanのコアコード
    ├── Connection                 // ソケット接続に関連する
    │   ├── ConnectionInterface.php// ソケット接続インタフェース
    │   ├── TcpConnection.php      // TCP接続クラス
    │   ├── AsyncTcpConnection.php // 非同期TCP接続クラス
    │   └── UdpConnection.php      // UDP接続クラス
    ├── Events                     // ネットワークイベントライブラリ
    │   ├── EventInterface.php     // ネットワークイベントライブラリインタフェース
    │   ├── Event.php              // Libeventネットワークイベントライブラリ
    │   ├── Ev.php                 // Libevネットワークイベントライブラリ
    │   ├── Swoole.php             // Swooleネットワークイベントライブラリ
    │   └── Select.php             // Selectネットワークイベントライブラリ
    ├── Lib                        // よく使われるクラスライブラリ
    │   ├── Constants.php          // 定数の定義
    │   └── Timer.php              // タイマー
    ├── Protocols                  // プロトコルに関連する
    │   ├── ProtocolInterface.php  // プロトコルインタフェースクラス
    │   ├── Http                   // HTTPプロトコルに関連する
    │   │   ├── Chunk.php    // HTTPチャンククラス
    │   │   ├── Request.php  // HTTPリクエストクラス
    │   │   ├── Response.php  // HTTPレスポンスクラス
    │   │   ├── ServerSentEvents.php  // SSEクラス
    │   │   ├── Session
    │   │   │   ├── FileSessionHandler.php  // セッションファイルのストレージ
    │   │   │   └── RedisSessionHandler.php // セッションRedisのストレージ
    │   │   ├── Session.php  // セッションクラス
    │   │   └── mime.types   // mimeマッピングファイル
    │   ├── Http.php               // HTTPプロトコルの実装
    │   ├── Text.php               // Textプロトコルの実装
    │   ├── Frame.php              // Frameプロトコルの実装
    │   └── Websocket.php          // Websocketプロトコルの実装
    ├── Worker.php                 // Worker
    ├── WebServer.php              // WebServer
    └── Autoloader.php             // 自動ロードクラス
```

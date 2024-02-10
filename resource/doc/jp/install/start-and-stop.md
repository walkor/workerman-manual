# 起動と停止

Workermanの起動と停止などのコマンドはすべてコマンドラインで行います。

Workermanを起動するには、まず起動エントリーファイルが必要で、その中にサービスがリッスンするポートとプロトコルが定義されています。[はじめての手引き--シンプルな開発例のセクション](../getting-started/simple-example.md)を参照してください。


例えば[workerman-chat](https://www.workerman.net/workerman-chat)では、起動エントリーはstart.phpです。

### 起動

デバッグモードで起動する

```php start.php start```

デーモンモードで起動する

```php start.php start -d```

### 停止
```php start.php stop```

### 再起動
```php start.php restart```

### 平滑な再起動
```php start.php reload```

### 状態の確認
```php start.php status```

### 接続状態の確認（Workermanのバージョン>=3.5.0が必要）
```php start.php connections```

## デバッグモードとデーモンモードの違い

1. デバッグモードでは、echo、var_dump、printなどの出力関数は直接端末に出力されます。

2. デーモンモードでは、echo、var_dump、printなどの出力は標準で/dev/nullファイルにリダイレクトされます。```Worker::$stdoutFile = '/your/path/file';```を設定してファイルのパスを変更できます。

3. デバッグモードでは、端末を閉じるとWorkermanもそれに続いて終了します。

4. デーモンモードでは、端末を閉じてもWorkermanは後ろで正常に動作し続けます。


## 平滑な再起動とは？

参照 [平滑な再起動の原理](../faq/reload-principle.md)

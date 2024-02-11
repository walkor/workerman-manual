# 環境要求

## Windowsユーザー
workermanはバージョン3.5.3からLinuxシステムとWindowsシステムの両方をサポートするようになりました。

1、PHP>=5.4が必要で、PHPの環境変数を正しく設定する必要があります。

2、Windows版のWorkermanには依存関係がありません。

3、インストール、使用、および使用制限については[**こちら**](https://www.workerman.net/windows)を参照してください。

4、WindowsでのWorkermanの使用にはいくつかの制限があるため、本番環境ではLinuxシステムをお勧めします。Windowsシステムは開発環境にのみお勧めします。

 ``` ====このページの以下はLinuxユーザーのみに適用されます。Windowsユーザーは無視してください。==== ```

## Linuxユーザー（Mac OSを含む）
LinuxユーザーはLinux版のWorkermanのみを使用できます。

1、PHP>=5.4をインストールし、pcntl、posix拡張機能をインストールしてください。

2、event拡張機能をインストールすることをお勧めしますが、必須ではありません（注意：event拡張機能にはPHP>=5.4が必要です）。

### Linux環境チェックスクリプト
Linuxユーザーは以下のスクリプトを実行して、ローカル環境がWorkermanの要件を満たしているかどうかをチェックすることができます。

```curl -Ss https://www.workerman.net/check | php```

スクリプトですべてがOKと表示されれば、Workermanの実行環境を満たしていることになります。

（注意：チェックスクリプトではevent拡張機能のチェックは行われていません。同時接続数が1024を超える場合は、event拡張機能をインストールすることをお勧めします。インストール方法については次のセクションを参照してください）

## 詳細説明

### PHP-CLIについて

Workermanは[PHPコマンドライン（PHP-CLI）](https://php.net/manual/zh/features.commandline.php)モードで動作します。PHP-CLIはPHP-FPMやApacheのMOD-PHPとは独立した実行可能プログラムであり、互いに衝突することはなく、依存関係もありません。

### Workermanが依存する拡張機能について

1、[pcntl拡張機能](https://cn2.php.net/manual/zh/book.pcntl.php)

pcntl拡張機能はLinux環境でプロセス制御を可能にする重要な拡張機能であり、Workermanはプロセスの作成、シグナル制御、タイマー、プロセスの状態監視などの機能に依存しています。この拡張機能はWindowsプラットフォームではサポートされていません。

2、[posix拡張機能](https://cn2.php.net/manual/zh/book.posix.php)

posix拡張機能により、PHPはLinux環境で[POSIX標準](https://baike.baidu.com/view/209573.htm)で提供されるインタフェースを呼び出すことができます。Workermanは、この関連インタフェースを使用してデーモン化やユーザーグループ制御などの機能を実現しています。この拡張機能はWindowsプラットフォームではサポートされていません。

3、[Event拡張機能](https://php.net/manual/zh/book.event.php) または [libevent拡張機能](https://cn2.php.net/manual/en/book.libevent.php)

event拡張機能により、PHPはシステムの[Epoll](https://baike.baidu.com/view/1385104.htm)、Kqueueなどの高度なイベント処理メカニズムを使用できます。これにより、Workermanは高並行接続時のCPU利用率を大幅に向上させることができます。高並行長接続に関連するアプリケーションでは非常に重要です。libevent拡張機能（またはevent拡張機能）は必須ではありませんが、インストールしない場合はデフォルトでPHPのネイティブの選択イベント処理メカニズムが使用されます。

## 拡張機能のインストール方法

[拡張機能のインストール](../appendices/install-extension.md)セクションを参照してください。

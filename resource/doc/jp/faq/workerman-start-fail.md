# workermanの起動に失敗する

## 現象1
起動後に以下のようなエラーが発生します：
```php
php start.php start
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (Address already in use) in ...workerman/Worker.php on line xxxx
```

**キーワード**: ```Address already in use```

**根本的な原因**: ポートが使用中で、起動できません。

#### 解決法1

```netstat -anp | grep 端口号``` コマンドを使用して、どのプログラムがポートを使用しているかを確認し、それに対応するプログラムを停止してポートを解放します。

#### 解決法2
関連するプログラムを停止できない場合は、workermanのポートを変更することで解決できます。

#### 解決法3
Workermanが使用しているポートがありながらも、stopコマンドで停止できない場合（一般的にはpidファイルが失われたり、開発者によってメインプロセスがkillされた場合）、以下の2つのコマンドを実行してWorkermanプロセスを終了させます。

```shell
killall php
ps aux|grep WorkerMan|awk '{print $2}'|xargs kill -9
```

#### 解決法4
もし本当にポートを使用しているプログラムがない場合、おそらく開発者がworkermanで同じポートを2つ以上リッスンするように設定したためですので、開発者は起動スクリプトが同じポートをリッスンしていないか確認してください。

#### 解決法5
プログラムがreusePortを有効にしているかどうかを確認し、reusePortを無効にしてみてください。

## 現象2
起動後に以下のようなエラーが発生します：
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxx (Cannot assign requested address) in ...workerman/Worker.php on line xxxx
```
または
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (在其上下文中，该请求的地址无效) in ...workerman/Worker.php on line xxxx
```

**キーワード**: `Cannot assign requested address`または`该请求的地址无效`

**失敗原因**: 起動スクリプトで間違ったIPが指定されており、それがホストのIPではない。ホストのIPまたは```0.0.0.0```を指定してください。

**ヒント**: Linuxシステムでは```ifconfig```コマンドですべてのネットワークカードのIPを確認できます。
クラウドサーバー（Alibaba Cloud / Tencent Cloudなど）を使用している場合、公開IPは実際にはプロキシIP（例：Alibaba Cloudのプライベートネットワーク）である可能性があるため、公開IPではリッスンできません。しかし、それでも```0.0.0.0```を使用してバインドすることができます。

## 現象3
```php
Waring stream_socket_server has been disabled for security reasons in ...
```

**失敗原因**: stream_socket_server関数がphp.iniで無効化されています。

**解決方法**

1、```php --ini``` コマンドを実行してphp.iniファイルを見つけます。

2、php.iniを開き、disable_functions項目を見つけて、その中からstream_socket_serverを削除します。

## 現象4
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://0.0.0.0:xxx (Permission denied)
```

**失敗原因**: Linuxでは、ポート番号が1024よりも小さい場合はroot権限が必要です。

**解決法**

1024よりも大きなポートを使用するか、rootユーザーでサービスを起動してください。

# インストール手順
Workermanは実際にはPHPのコードパッケージであり、すでにPHP環境が整備されている場合、Workermanのソースコードまたはデモをダウンロードして実行するだけです。

**Composerのインストール：**
```sh
composer require workerman/workerman
```

> **注意**
> 一部のComposerプロキシミラーが完全ではない場合は、以下のコマンドを使用して`composer config -g --unset repos.packagist` を実行してプロキシを削除してください。

# Windowsユーザー（必読）

Workerman 3.5.3以降、WorkermanはWindowsおよびLinuxシステムの両方をサポートしています。
Windowsユーザーは、PHP環境変数を設定する必要があります。

 ` ===このページの以下の内容はLinux環境のWorkermanにのみ適用され、Windowsユーザーは無視してください=== `

# Linuxシステム環境のテスト
Linuxシステムでは、以下のスクリプトを使用して、ローカルのPHP環境がWorkermanの実行要件を満たしているかどうかをテストできます。
 `curl -Ss https://www.workerman.net/check | php`

上記のスクリプトがすべて「ok」を表示した場合、Workermanの要件を満たしていることを示し、[公式サイト](https://www.workerman.net/)から例をダウンロードして実行できます。

すべてがOKでない場合は、以下の文書を参照して、不足している拡張機能をインストールします。

（注意：テストスクリプトはイベント拡張機能をテストしていません。もしビジネスの同時接続数が1024を超える場合は、イベント拡張機能をインストールする必要があります。また、[Linuxカーネルの最適化](../appendices/kernel-optimization.md)も必要です。拡張機能のインストール方法は以下の説明に従ってください。）

# 既存のPHP環境に拡張機能をインストールする

## pcntlおよびposix拡張のインストール：

**CentOSシステム**
PHPがyumを使用してインストールされている場合、次のコマンドを実行して、pcntlおよびposix拡張機能をインストールできます。
```shell
yum install php-process
```

インストールに失敗した場合やPHPがyumを使用していない場合は、[付録-拡張機能のインストール](../appendices/install-extension.md)の「方法三：ソースコードでのコンパイルインストール」を参照してください。

**Debian/Ubuntu/Mac OSシステム**
[付録-拡張機能のインストール](../appendices/install-extension.md)の「方法三：ソースコードでのコンパイルインストール」を参照してください。

## event拡張のインストール：
より多くの同時接続数をサポートするためには、event拡張をインストールし、[Linuxカーネルの最適化](../appendices/kernel-optimization.md)を行う必要があります。インストール手順は以下の通りです。

**CentOSシステム**

1、event拡張の依存であるlibevent-develパッケージをインストールするために、次のコマンドを実行します。
```shell
yum install libevent-devel -y
# インストールできない場合は、次のコマンドを試してください
# yum install libevent2-devel -y
```

2、event拡張をインストールするために、次のコマンドを実行します。
(event拡張はPHP>=5.4が必要です)
```shell
pecl install event
```
注意：`Include libevent OpenSSL support [yes] :`とプロンプトが表示された場合は、`no`と入力してEnterを押します。その他のプロンプトはすべてEnterを押してください。

3、`php --ini`を実行してphp.iniファイルを見つけ、最後の行に以下の設定を追加します。
```shell
extension=event.so
```

**Debian/Ubuntuシステムのインストール**

1、libevent-devパッケージをインストールするために、次のコマンドを実行します。
```shell
apt-get install libevent-dev -y
# インストールできない場合、以下のコマンドを試してください
# apt-get install libevent2-dev -y
```

2、event拡張をインストールするために、次のコマンドを実行します。
```shell
pecl install event
```
注意：`Include libevent OpenSSL support [yes] :` とプロンプトが表示された場合は、 `no` と入力してEnterを押してください。

3、`php --ini`を実行してphp.iniファイルを見つけ、最後の行に以下の設定を追加します。
```shell
extension=event.so
```

**Mac OSシステムのインストール手順**

Macシステムは一般的に開発用のマシンとして使用されるため、イベント拡張をインストールする必要はありません。

# 新しいシステムのインストール（PHPと拡張機能の新規インストール）

## CentOSシステムのインストール手順

1、以下のコマンドを実行して（この手順にはphp-cliのメインプログラム、pcntl、posix、libeventライブラリおよびgitプログラムのインストールが含まれています）
```shell
yum install php-cli php-process git gcc php-devel php-pear libevent-devel -y
```

2、event拡張をインストールするために、次のコマンドを実行します。
（注意: event拡張はPHP>=5.4が必要です）
```shell
pecl install event
```
注意： `Include libevent OpenSSL support [yes] :` とプロンプトが表示された場合は、 `no` と入力してEnterを押します。その他のプロンプトはすべてEnterを押してください。

3、`php --ini`を実行してphp.iniファイルを見つけ、最後の行に以下の設定を追加します。
```shell
extension=event.so
```

4、以下のコマンドを実行して、Workermanメインプログラムをgithubからダウンロードします。
```shell
git clone https://github.com/walkor/Workerman
```

5、[入門ガイド--シンプルな開発例](../getting-started/simple-example.md)を参照して、エントリーファイルを作成して実行してください。もしくは[公式サイト](https://www.workerman.net/)からダウンロードしたデモを実行してください。

## Debian/Ubuntuシステムのインストール手順

1、以下のコマンドを実行して（この手順にはphp-cliのメインプログラム、libeventライブラリおよびgitプログラムのインストールが含まれています）
```shell
apt-get install php-cli git gcc php-pear php-dev libevent-dev -y
```

2、event拡張をインストールするために、次のコマンドを実行します。
（注意: event拡張はPHP>=5.4が必要です）
```shell
pecl install event
```
注意： `Include libevent OpenSSL support [yes] :` とプロンプトが表示された場合は、 `no` と入力してEnterを押してください。

3、`php --ini`を実行してphp.iniファイルを見つけ、最後の行に以下の設定を追加します。
```shell
extension=event.so
```

4、以下のコマンドを実行して、Workermanメインプログラムをgithubからダウンロードします。
```shell
git clone https://github.com/walkor/Workerman
```

5、[入門ガイド--シンプルな開発例](../getting-started/simple-example.md)を参照して、エントリーファイルを作成して実行してください。もしくは[公式サイト](https://www.workerman.net/)からダウンロードしたデモを実行してください。

## Mac OSシステムのインストール手順

**方法1：** MacシステムにはPHP Cliが搭載されていますが、おそらく```pcntl``` 拡張機能が不足しています。

1、参考:[付録-拡張機能のインストール](../appendices/install-extension.md)セクションで「方法三：ソースコードでのコンパイルインストール」を参照してください。

2、参考:[付録-拡張機能のインストール](../appendices/install-extension.md)セクションの方法4を使用して、```event```拡張機能をインストールします（開発用のマシンでは省略可）。

3、https://www.workerman.net/download/workermanzip からWorkermanメインプログラムをダウンロードするか、[公式サイト](https://www.workerman.net/) から例をダウンロードして実行してください。

**方法2：** ```brew``` コマンドを使用してPHPおよび対応する拡張をインストールする

1、以下のコマンドを実行して ```brew``` ツールをインストールします（すでに ```brew``` をインストール済みの場合はこの手順をスキップしてください）
```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2、以下のコマンドを実行して ```php``` をインストールします
```shell
brew install php
```

3、以下のコマンドを実行して ```event``` 拡張をインストールします
```shell
brew install php-event
```

4、[公式サイト](https://www.workerman.net/) から例をダウンロードして実行してください

# Event拡張の説明
[Event拡張](https://php.net/manual/zh/book.event.php)は必須ではありませんが、ビジネスに1000を超える同時接続が必要な場合は、Eventのインストールを推奨し、非常に大規模な同時接続をサポートできます。ビジネスの同時接続が比較的少ない場合（1000以下の同時接続など）、Eventをインストールする必要はありません。

## よくある問題
1、以下のエラーが発生した場合: `checking for include/event2/event.h... not found`、libevent-dev(el)ライブラリを削除してlibevent2-dev(el)を再インストールしてください。
CentOSシステム：yum remove libevent-devel && yum install libevent2-devel
Debian/Ubuntuシステム：apt-get remove libevent-dev && apt-get install libevent2-dev

2、次のエラーが発生した場合:`NOTICE: PHP message: PHP Warning: PHP Startup: Unable to load dynamic library '.../event.so' - ..../event.so: undefined symbol: php_sockets_le_socket in Unknown on line 0`。
`event.so` と `socket.so` の読み込み順序を変更し、`php.ini` ファイルで `extension=socket.so` を `extension=event.so` の前に配置し、先にsocket拡張をロードするようにしてください。

# Channel分散通信コンポーネント
**``` （Workermanバージョン>=3.3.0が必要です） ```**

ソースコード：https://github.com/walkor/Channel

Channelはプロセス間通信またはサーバ間通信を完了するための分散通信コンポーネントです。

## 特徴
1、購読/発行モデルに基づく
2、ノンブロッキングIO

## 原理
ChannelにはChannel/ServerサーバーとChannel/Clientクライアントが含まれています。

Channel/Clientはconnectインタフェースを使用してChannel/Serverに接続し、長期間の接続を維持します。

Channel/Clientはonインタフェースを呼び出して、Channel/Serverに自分がどのイベントに関心を持っているかを通知し、イベントコールバック関数を登録します（コールバックはChannel/Clientが存在するプロセスで発生します）。

Channel/Clientはpublishインタフェースを使用して、特定のイベントと関連データをChannel/Serverに公開します。

Channel/Serverはイベントとデータを受け取った後、そのイベントに関心を持つChannel/Clientにそれを配信します。

Channel/Clientはイベントとデータを受け取った後、onインタフェースで設定されたコールバックをトリガーします。

Channel/Clientは自分の興味を持つイベントだけを受け取り、コールバックをトリガーします。

## インストール

`composer require workerman/channel`

## 注意
ChannelはWorkerman環境でのみ使用でき、php-fpm環境では使用できません。

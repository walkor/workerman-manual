# IPv6

질문: 고객이 IPv4 주소를 통해 액세스하거나 IPv6 주소를 통해 액세스할 수 있도록하려면 어떻게해야 합니까?

답변: 컨테이너를 초기화 할 때 주소를 ' [::] '로 설정하면 됩니다.

예를 들어
```php
$worker = new Worker('http://[::]:8080');
$gateway = new Gateway('websocket://[::]:8081');
```

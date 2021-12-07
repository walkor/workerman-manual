# workerman/http-client
## 说明
 [workerman/http-client](https://github.com/walkor/http-client)是一个异步http客户端组件。所有请求响应异步非阻塞，内置连接池，消息请求和响应符合PSR7规则。

## 安装：
```
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
composer require workerman/http-client
```

## 示例：

```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function(){
    $http = new Workerman\Http\Client();
    
    $http->get('https://example.com/', function($response){
            var_dump($response->getStatusCode());
            echo $response->getBody();
        }, function($exception){
            echo $exception;
        });
    
    $http->post('https://example.com/', ['key1'=>'value1','key2'=>'value2'], function($response){
            var_dump($response->getStatusCode());
            echo $response->getBody();
        }, function($exception){
            echo $exception;
        });
    
    $http->request('https://example.com/', [
            'method'  => 'POST',
            'version' => '1.1',
            'headers' => ['Connection' => 'keep-alive'],
            'data'    => ['key1' => 'value1', 'key2'=>'value2'],
            'success' => function ($response) {
                echo $response->getBody();
            },
            'error'   => function ($exception) {
                echo $exception;
            }
        ]);
};
Worker::runAll();
```

## 注意：

1、项目首先要加载`require __DIR__ . '/vendor/autoload.php';`

2、所有的异步编码必须在```onXXX```回调中编写

3、支持基于workerman开发的所有项目，包括GatewayWorker、PHPSocket.io等




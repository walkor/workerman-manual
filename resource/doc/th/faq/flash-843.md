# เปิดพอร์ต 843 สำหรับ Flash

เมื่อ Flash ส่งคำร้องขอการเชื่อมต่อโดยโซเก็ทไปยังเซิร์ฟเวอร์ระยะไกล มันจะเริ่มจากการส่งคำร้องขอไฟล์นโยบายความปลอดภัยไปยังพอร์ต 843 ของเซิร์ฟเวอร์ที่เกี่ยวข้อง หากไม่มีไฟล์นโยบายความปลอดภัยนั้น Flash จะไม่สามารถเชื่อมต่อกับเซิร์ฟเวอร์ได้ ใน Workerman เราสามารถเปิดพอร์ต 843 และส่งกลับไฟล์นโยบายความปลอดภัยได้ดังต่อไปนี้

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$flash_policy = new Worker('tcp://0.0.0.0:843');
$flash_policy->onMessage = function(TcpConnection $connection, $message)
{
    $connection->send('<?xml version="1.0"?><cross-domain-policy><site-control permitted-cross-domain-policies="all"/><allow-access-from domain="*" to-ports="*"/></cross-domain-policy>'."\0");
};

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```

ส่วนของเนื้อหาของนโยบายความปลอดภัย XML สามารถปรับแต่งตามความต้องการของคุณได้

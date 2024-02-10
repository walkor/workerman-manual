# المثال 1
**``` (متطلبات Workerman >= 3.3.0) ```**

نظام الإرسال المتعدد العملية القائمة على Worker

``` php
    use Workerman\Worker;
    use Workerman\Connection\TcpConnection;
    require_once __DIR__ . '/vendor/autoload.php';

    $channel_server = new Channel\Server('0.0.0.0', 2206);

    $worker = new Worker('websocket://0.0.0.0:1234');
    $worker->count = 8;
    // تعيين مجموعة عرضيات الاتصالات إلى مجموعة
    $group_con_map = array();
    $worker->onWorkerStart = function(){
        // يتصل عميل Channel بخادم القناة
        Channel\Client::connect('127.0.0.1', 2206);

        // استمع إلى حدث إرسال رسالة المجموعة العالمية
        Channel\Client::on('send_to_group', function($event_data){
            $group_id = $event_data['group_id'];
            $message = $event_data['message'];
            global $group_con_map;
            var_dump(array_keys($group_con_map));
            if (isset($group_con_map[$group_id])) {
                foreach ($group_con_map[$group_id] as $con) {
                    $con->send($message);
                }
            }
        });
    };
    $worker->onMessage = function(TcpConnection $con, $data){
        // أضف رسالة المجموعة {"cmd":"add_group", "group_id":"123"}
        // أو قم بإرسال رسالة {"cmd":"send_to_group", "group_id":"123", "message":"هذه هي الرسالة"}
        $data = json_decode($data, true);
        var_dump($data);
        $cmd = $data['cmd'];
        $group_id = $data['group_id'];
        switch($cmd) {
            // انضم إلى مجموعة الاتصال
            case "add_group":
                global $group_con_map;
                // أضف الاتصال إلى مجموعة الاتصال المقابلة
                $group_con_map[$group_id][$con->id] = $con;
                // سجل هذا الاتصال المضاف في أي مجموعات، للسماح بتنظيف group_con_map بيانات المجموعة المقابلة في onClose
                $con->group_id = isset($con->group_id) ? $con->group_id : array();
                $con->group_id[$group_id] = $group_id;
                break;
            // ارسال رسالة إلى المجموعة
            case "send_to_group":
                // ينشر عميل Channel حدث إرسال رسالة المجموعة لجميع العمليات في جميع خوادم
                Channel\Client::publish('send_to_group', array(
                    'group_id'=>$group_id,
                    'message'=>$data['message']
                ));
                break;
        }
    };
    // هنا مهم جدًا، عند إغلاق الاتصال ، يجب حذف الاتصال من بيانات مجموعة العرضيات العالمية، لتجنب تسرب الذاكرة
    $worker->onClose = function(TcpConnection $con){
        global $group_con_map;
        // تحقق من الاتصالات المضافة لجميع المجموعات واحذف بيانات group_con_map المقابلة
        if (isset($con->group_id)) {
            foreach ($con->group_id as $group_id) {
                unset($group_con_map[$group_id][$con->id]);
                if (empty($group_con_map[$group_id])) {
                    unset($group_con_map[$group_id]);
                }
            }
        }
    };

    Worker::runAll();
```


## الاختبار (الافتراضي أن جميع الأجهزة تعمل على 127.0.0.1)
1- تشغيل الخادم
``` shell
php start.php start
Workerman[del.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.4.2          PHP version:7.1.3
------------------------ WORKERS -------------------------------
user          worker         listen                    processes status
liliang       ChannelServer  frame://0.0.0.0:2206       1         [OK] 
liliang       none           websocket://0.0.0.0:1234   12        [OK] 
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

2- توصيل العميل بالخادم

افتح متصفح Chrome، واضغط على F12 لفتح لوحة تحكم التصحيح. في النافذة المنبثقة، ادخل الأمر التالي في الجدول (أو ضع الكود التالي في صفحة HTML وقم بتشغيله باستخدام جافا سكريبت)

```javascript
// يفترض أن عنوان الخادم هو 127.0.0.1، يرجى تغييره إلى عنوان الخادم الفعلي أثناء التجربة
ws = new WebSocket('ws://127.0.0.1:1234');
ws.onmessage = function(data){console.log(data.data)};
ws.onopen = function() {
	ws.send('{"cmd":"add_group", "group_id":"123"}');
    ws.send('{"cmd":"send_to_group", "group_id":"123", "message":"هذه هي الرسالة"}');
};

```

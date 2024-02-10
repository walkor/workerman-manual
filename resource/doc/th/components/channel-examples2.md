# ตัวอย่างที่ 1
**``` (ต้องการ Workerman เวอร์ชัน >= 3.3.0) ```**

ระบบส่งข้อความแบบหลายๆ โปรเซสถ่ายภาพของ Worker

``` php
    use Workerman\Worker;
    use Workerman\Connection\TcpConnection;
    require_once __DIR__ . '/vendor/autoload.php';

    $channel_server = new Channel\Server('0.0.0.0', 2206);

    $worker = new Worker('websocket://0.0.0.0:1234');
    $worker->count = 8;
    // การจับคู่การเชื่อมต่อของกลุ่มโดยรวมไปยังการเชื่อมต่อแบบระเบิด
    $group_con_map = array();
    $worker->onWorkerStart = function(){
        // การเชื่อมต่อผู้ใช้ Channel กับตัวเครื่องที่เป็นเซิร์ฟเวอร์
        Channel\Client::connect('127.0.0.1', 2206);

        // ตรวจสอบเหตุการณ์การส่งข้อความของกลุ่มที่ได้รับทั้งหมด
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
        // เข้าร่วมกลุ่มข้อความ {"cmd":"add_group", "group_id":"123"}
        // หรือ ส่งข้อความถึงกลุ่ม {"cmd":"send_to_group", "group_id":"123", "message":"นี่คือข้อความ"}
        $data = json_decode($data, true);
        var_dump($data);
        $cmd = $data['cmd'];
        $group_id = $data['group_id'];
        switch($cmd) {
            // เพิ่มการเชื่อมต่อเข้าสู่กลุ่ม
            case "add_group":
                global $group_con_map;
                // เพิ่มการเชื่อมต่อเข้าไปในอาร์เรย์กลุ่มที่เหมาะกับ
                $group_con_map[$group_id][$con->id] = $con;
                // บันทึกว่าการเชื่อมต่อนี้ได้เข้าร่วมกลุ่มใดบ้างเพื่อทำความสะอาดของ group_con_map ในเวลาที่กำหนด
                $con->group_id = isset($con->group_id) ? $con->group_id : array();
                $con->group_id[$group_id] = $group_id;
                break;
            // ส่งข้อความถึงกลุ่ม
            case "send_to_group":
                // Channel\Client แจ้งเหตุการณ์การส่งข้อความผ่านทางทุกเซิร์ฟเวอร์และโปรเซส
                Channel\Client::publish('send_to_group', array(
                    'group_id'=>$group_id,
                    'message'=>$data['message']
                ));
                break;
        }
    };
    // ส่วนนี้สำคัญมาก โปรเซสการปิดการเชื่อมต่อเพื่อลบการเชื่อมต่อที่ได้จากข้อมูลกลุ่มทั้งหมด เพื่อป้องกันการรั่วหน่อยในหน่วยความจำ
    $worker->onClose = function(TcpConnection $con){
        global $group_con_map;
        // วิ่งย้อนกลับไปที่การเชื่อมต่อที่เข้าระหว่างทุกกลุ่มเพื่อลบข้อมูลที่สอดคล้องใน group_con_map
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

## การทดสอบ (ถ้าเป็นเครื่องเดียวกันทั้งหมด 127.0.0.1)
1. เริ่มเซิร์ฟเวอร์
```php
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
2. เชื่อมต่อไปยังเซิร์ฟเวอร์ (Client)

เปิดเบราว์เซอร์ Chrome แล้วกด F12 เพื่อเปิดคอนโซล Debug ในโหมด Console พิมพ์คำสั่งต่อไปนี้ (หรือวางรหัสด้านล่างลงบนหน้าเว็บ HTML เพื่อใช้รัน JavaScript)

```javascript
// สมมติว่า IP ของเซิร์ฟเวอร์คือ 127.0.0.1 กรุณาแก้ไข IP เซิร์ฟเวอร์จริงก่อนทดสอบ
ws = new WebSocket('ws://127.0.0.1:1234');
ws.onmessage = function(data){console.log(data.data)};
ws.onopen = function() {
	ws.send('{"cmd":"add_group", "group_id":"123"}');
    ws.send('{"cmd":"send_to_group", "group_id":"123", "message":"นี่คือข้อความ"}');
};

```

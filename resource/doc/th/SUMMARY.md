* [บทนำ](README.md)
* [หลักการ](principle.md)
* [คำแนะนำสำหรับนักพัฒนา](must-read.md)
* คู่มือเริ่มต้น
    * [คุณสมบัติ](getting-started/feature.md)
    * [ตัวอย่างการพัฒนาง่าย](getting-started/simple-example.md)
* การติดตั้ง
    * [ความต้องการของระบบ](install/requirement.md)
    * [ดาวน์โหลดและติดตั้ง](install/install.md)
    * [เริ่มต้นและหยุด](install/start-and-stop.md)
* กระบวนการการพัฒนา
    * [คำแนะนำก่อนการพัฒนา](development/before-development.md)
    * [โครงสร้างไดเรกทอรี](development/directory-structure.md)
    * [มาตรฐานการพัฒนา](development/standard.md)
    * [กระบวนการพื้นฐาน](development/process.md)
* โปรโตคอลการสื่อสาร
    * [ความสำคัญของโปรโตคอลการสื่อสาร](protocols/why-protocols.md)
    * [การปรับแต่งโปรโตคอลการสื่อสาร](protocols/how-protocols.md)
    * [ตัวอย่างบางตัวอย่าง](protocols/example.md)
* [คลาส Worker](worker.md)
    * [เมทอดคอนสตรักเตอร์](worker/construct.md)
    * แอตทริบิวต์
        * [id](worker/workerid.md)
        * [count](worker/count.md)
        * [name](worker/name.md)
        * [protocol](worker/protocol.md)
        * [transport](worker/transport.md)
        * [reusePort](worker/reuse-port.md)
        * [connections](worker/connections.md)
        * [stdoutFile](worker/stdout-file.md)
        * [pidFile](worker/pid-file.md)
        * [logFile](worker/log-file.md)
        * [user](worker/user.md)
        * [reloadable](worker/reloadable.md)
        * [daemonize](worker/daemonize.md)
        * [globalEvent](worker/global-event.md)
    * แอตทริบิวต์ของคอลล์แบ็ก
        * [onWorkerStart](worker/on-worker-start.md)
        * [onWorkerReload](worker/on-worker-reload.md)
        * [onConnect](worker/on-connect.md)
        * [onMessage](worker/on-message.md)
        * [onClose](worker/on-close.md)
        * [onBufferFull](worker/on-buffer-full.md)
        * [onBufferDrain](worker/on-buffer-drain.md)
        * [onError](worker/on-error.md)
    * อินเทอร์เฟส
        * [runAll](worker/run-all.md)
        * [stopAll](worker/stop-all.md)
        * [listen](worker/listen.md)
* [คลาส TcpConnection](tcp-connection.md)
    * แอตทริบิวต์
        * [id](tcp-connection/id.md)
        * [protocol](tcp-connection/protocol.md)
        * [worker](tcp-connection/worker.md)
        * [maxSendBufferSize](tcp-connection/max-send-buffer-size.md)
        * [defaultMaxSendBufferSize](tcp-connection/default-max-send-buffer-size.md)
        * [defaultMaxPackageSize](tcp-connection/default-max-package-size.md)
    * แอตทริบิวต์ของคอลล์แบ็ก
        * [onMessage](tcp-connection/on-message.md)
        * [onClose](tcp-connection/on-close.md)
        * [onBufferFull](tcp-connection/on-buffer-full.md)
        * [onBufferDrain](tcp-connection/on-buffer-drain.md)
        * [onError](tcp-connection/on-error.md)
    * อินเทอร์เฟส
        * [send](tcp-connection/send.md)
        * [getRemoteIp](tcp-connection/get-remote-ip.md)
        * [getRemotePort](tcp-connection/get-remote-port.md)
        * [close](tcp-connection/close.md)
        * [destroy](tcp-connection/destroy.md)
        * [pauseRecv](tcp-connection/pause-recv.md)
        * [resumeRecv](tcp-connection/resume-recv.md)
        * [pipe](tcp-connection/pipe.md)
* [คลาส AsyncTcpConnection](async-tcp-connection.md)
    * [__construct](async-tcp-connection/construct.md)
    * [connect](async-tcp-connection/connect.md)
    * [reconnect](async-tcp-connection/reconnect.md)
    * [transport](async-tcp-connection/transport.md)
* [คลาส AsyncUdpConnection](async-udp-connection.md)
    * [__construct](async-udp-connection/construct.md)
    * [connect](async-udp-connection/connect.md)
    * [send](async-udp-connection/send.md)
    * [close](async-udp-connection/close.md)
* คลาสตัวจับเวลา Timer
    * [add](timer/add.md)
    * [del](timer/del.md)
    * [ข้อควรระวังในการใช้ตัวจับเวลา](timer/notice.md)
    * [crontab](timer/crontab.md)
* บริการ Http
    * [คำขอ](http/request.md)
    * [การตอบกลับ](http/response.md)
    * [เซสชัน](http/session.md)
    * [การควบคุมเซสชัน](http/session-control.md)
    * [SSE](http/SSE.md)
* Debugging
    * [การdebuggingพื้นฐาน](debug/base.md)
    * [คำสั่ง status เพื่อตรวจสอบสถานะการทำงาน](debug/status.md)
    * [debug โปรเซสที่ไม่สามารถใช้งานได้](debug/busy-process.md)
    * [การดักจับแพคเกจเครือข่าย](debug/tcpdump.md)
    * [การติดตามการเรียกใช้ระบบ](debug/strace.md)
* คอมโพเนนต์ที่ใช้บ่อย
    * [คอมโพเนนต์ GlobalData สำหรับแชร์ข้อมูล](components/global-data.md)
        * [GlobalDataServer](components/global-data-server.md)
        * [GlobalDataClient](components/global-data-client.md)
            * [add](components/global-data-client-add.md)
            * [cas](components/global-data-client-cas.md)
            * [increment](components/global-data-client-increment.md)
    * [คอมโพเนนต์ Channel สำหรับการสื่อสารแบบกระจาย](components/channel.md)
        * [ChannelServer](components/channel-server.md)
        * [channelClient](components/channel-client.md)
            * [connect](components/channel-client-connect.md)
            * [on](components/channel-client-on.md)
            * [publish](components/channel-client-publish.md)
            * [unsubsribe](components/channel-client-unsubsribe.md)
        * [ตัวอย่าง-การส่งข้อความในกลุ่ม](components/channel-examples.md)
        * [ตัวอย่าง-การส่งข้อความเป็นกลุ่ม](components/channel-examples2.md)
    * [คอมโพเนนต์ FileMonitor สำหรับการตรวจสอบไฟล์](components/file-monitor.md)
    * [คอมโพเนนต์ MySQL](components/mysql.md)
        * [workerman/mysql](components/workerman-mysql.md)
        * [คลาสฐานข้อมูลอื่น ๆ](components/other-mysql-class.md)
    * [คอมโพเนนต์ Redis](components/redis.md)
        * [workerman/redis](components/workerman-redis.md)
    * [คอมโพเนนต์ http แบบไม่สม่ำเสมอ](components/async-http.md)
        * [workerman/http-client](components/workerman-http-client.md)
    * [คอมโพเนนต์ Message Queue แบบไม่สม่ำเสมอ](components/async-message-queue.md)
        * [workemran/mqtt](components/workerman-mqtt.md)
        * [workerman/redis-queue](components/workerman-redis-queue.md)
        * [workerman/stomp](components/workerman-stomp.md)
        * [workerman/rabbitmq](components/workerman-rabbitmq.md)
    * [งานที่ต้องทำตามลำดับเวลา Crontab](components/crontab.md)
    * [Memcache](components/memcache.md)
* คำถามที่พบบ่อย
    * [การตรวจสอบการเชื่อมต่ออย่างต่อเนื่อง](faq/heartbeat.md)
    * [การโหลดคลาสโดยอัตโนมัติ](faq/autoload.md)
    * [เหตุผลที่ทำให้การเชื่อมต่อของผู้ใช้ล้มเหลว](faq/client-connect-fail.md)
    * [ระบบที่รองรับการใช้งานพร็อกเซสเป็นหลายประการ](faq/about-multi-thread.md)
    * [การทำงานร่วมกับ framework อื่น](faq/work-with-other-framework.md)
    * [การทำงานพร้อมกันของ workerman หลายรุ่น](faq/running-concurent.md)
    * [โปรโตคอลที่รองรับ](faq/protocols.md)
    * [การตั้งค่าจำนวนโปรเซส](faq/processes-count.md)
    * [การตรวจสอบจำนวนการเชื่อมต่อของผู้ใช้](faq/connection-status.md)
    * [การสร้างและเก็บรงุงวัตถุเเละทรัพยากร](faq/persistent-data-and-resources.md)
    * [โค้ดตัวอย่างที่ไม่ทำงาน](faq/demo-not-work.md)
    * [การเริ่มต้นล้มเหลว](faq/workerman-start-fail.md)
    * [การหยุดล้มเหลว](faq/stop-fail.md)
    * [การรองรับการเชื่อมต่อเป็นจำนวนมาก](faq/how-many-connections.md)
    * [การเปลี่ยนแปลงการทำงานของโค้ดไม่ได้](faq/change-code-not-work.md)
    * [การส่งข้อมูลไปยังผู้ใช้ที่กำหนด](faq/send-data-to-client.md)
    * [การส่งข้อมูลผ่านการเชื่อมต่อแบบนําเอง](faq/active-push.md)
    * [การส่งข้อมูลในโปรเจกต์อื่น](faq/push-in-other-project.md)
    * [การดำเนินงานของงานแบบไม่สม่ำเสมอ](faq/async-task.md)
    * [เหตุผลของข้อผิดพลาดในการส่งข้อมูลไปยังผู้ใช้](faq/about-send-fail.md)
    * [การพัฒนาในระบบ windows และการใช้งานในระบบ linux](faq/windows-to-linux.md)
    * [การรองรับ socket.io หรือไม่](faq/socketio-support.md)
    * [การหยุดการทำงานของ workerman เมื่อหยุดการเชื่อมต่อด้วย ssh](faq/ssh-close-and-workerman-stop.md)
    * [ความสัมพันธ์กับ nginx และ apache](faq/relationship-with-apache-nginx.md)
    * [การปิดการใช้ฟังก์ชั่นที่ไม่ได้ใช้งาน](faq/disable-function-check.md)
    * [หลักการการทำงานของการรีโหลดอย่างราบรื่น](faq/reload-principle.md)
    * [การเปิดพอร์ต 843 สำหรับ Flash](faq/flash-843.md)
    * [การส่งข้อมูลไปยังทุกอย่าง](faq/how-to-broadcast.md)
    * [การสร้างบริการ udp](faq/how-to-create-udp-service.md)
    * [การตรวจสอบการเชื่อมต่อ ipv6](faq/ipv6.md)
    * [การปิดการใช้งานการเชื่อมต่อที่ไม่ได้รับอนุญาต](faq/close-unauthed-connections.md)
    * [การเข้ารหัสการส่ง-รับ ที่สนับสนุน ssl/tls](faq/ssl-support.md)
    * [การสร้างบริการ wss](faq/secure-websocket-server.md)
    * [การสร้างบริการ https](faq/secure-http-server.md)
    * [การใช้ workerman ในฐานะการเชื่อมต่อของลูกค้า](faq/use-workerman-as-client-side.md)
    * [การใช้ workerman เป็นลูกค้าของ ws/wss](faq/as-wss-client.md)
    * [แอพพลิเคชันวีแวกซ์](faq/weixin-app.md)
    * [แพทเทิร์นการเขียน callback ของ PHP](faq/callback_methods.md)
    * [การรับค่าไอพีจากพร็อกซี](faq/get-real-ip-from-proxy.md)
    * [การเริ่มต้นพร้อมกับระบบ](faq/start-with-system.md)
    * [การรับข้อมูลแบบฐานสิบหก](faq/send-recv-hexadecimal-data.md)
    * [การรับคำขอบางตัวแล้วทำการเริ่มต้นใหม่](faq/max-requests.md)
    * [การเริ่มต้น multi-worker สำหรับระบบ windows](faq/multi-woker-for-windows.md)
    * [คำขอที่เข้ารวดเดียวไปยังโปรเซสบางประการ](faq/requests-concentrated-in-certain-processes.md)
* ภาคผนวก
    * [การปรับแต่ง kernel ของ Linux](appendices/kernel-optimization.md)
    * [การทดสอบแรงดันของระบบ](appendices/stress-test.md)
    * [การติดตั้งส่วนขยาย](appendices/install-extension.md)
    * [โปรโตคอล websocket](appendices/about-websocket.md)
    * [โปรโตคอล ws](appendices/about-ws.md)
    * [โปรโตคอล text](appendices/about-text.md)
    * [โปรโตคอล frame](appendices/about-frame.md)
    * [ฟังก์ชั่น/คุณสมบัติที่ไม่มีในระบบ](appendices/unavailable-functions.md)
* [ข้อมูลลิขสิทธิ์](license.md)
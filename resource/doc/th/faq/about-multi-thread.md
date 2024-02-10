Workerman รองรับการทำงานแบบ multi-thread หรือเปล่า?

Workerman มี [เวอร์ชัน MT multi-thread](https://github.com/walkor/workerman-MT) ซึ่งมีความขึ้นอยู่กับการขยายตัวของ [pthreads ส่วนขยาย](https://php.net/manual/zh/book.pthreads.php)  แต่เนื่องจากส่วนขยาย pthreads ยังไม่เสถียรพอ ดังนั้นเวอร์ชัน multi-thread ของ Workerman ได้ถูกยุติการพัฒนาแล้ว

**ในปัจจุบัน Workerman และผลิตภัณฑ์รองรับการทำงานบนหลายกระบวนการเป็นจำนวนหนึ่งและเป็น single-thread**

# Workerman이 멀티 스레드를 지원하나요?

Workerman에는 [pthreads 확장](https://php.net/manual/zh/book.pthreads.php)에 의존하는 [MT 멀티 스레드 버전](https://github.com/walkor/workerman-MT)이 있지만, pthreads 확장이 아직 안정적이지 않아 Workerman 멀티 스레드 버전은 더 이상 유지보수되지 않고 있습니다.

**현재 Workerman 및 관련 제품은 멀티 프로세스 단일 스레드를 기반으로 하고 있습니다.**

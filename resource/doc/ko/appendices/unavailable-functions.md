# 지원되지 않는 함수

지원되지 않는 함수/문장 | 대체 방안 | 설명
----|------|----
pcntl_fork | 미리 프로세스 수 설정 |
php://input | [`$request->rawBody()`](http/request.md) | HTTP 프로토콜을 사용하는 응용프로그램에서 POST의 원시 데이터를 가져오기 위해 사용됨
exit | return | exit를 사용하면 프로세스가 종료되며 반환하려면 직접 return 문을 사용해야 함
die | return | die를 사용하면 프로세스가 종료되며 반환하려면 직접 return 문을 사용해야 함
header cookie session 관련 함수 | [`$request`](http/request.md) 및 [`$response`](http/response.md) 클래스 참조 |
set_time_limit| 없음 | 0으로만 설정 가능하며, 그렇지 않으면 workerman 프로세스가 일정 시간 후에 종료됨

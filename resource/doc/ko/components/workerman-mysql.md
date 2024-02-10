# Workerman/MySQL

## 설명
메모리 상주형 프로그램은 MySQL을 사용할 때 종종 `mysql gone away` 오류를 겪을 수 있습니다. 이는 프로그램과 MySQL의 연결이 장시간 동안 통신되지 않아 MySQL 서버에서 연결이 끊어지는 것으로 인한 오류입니다. 이 데이터베이스 클래스는 이 문제를 해결할 수 있으며, `mysql gone away` 오류가 발생할 때 자동으로 한 번 재시도합니다.

## 의존성 확장
이 MySQL 클래스는 [pdo](https://php.net/manual/zh/book.pdo.php) 및 [pdo_mysql](https://php.net/manual/zh/ref.pdo-mysql.php) 두 확장에 의존하며, 이러한 확장이 없으면 `Undefined class constant 'MYSQL_ATTR_INIT_COMMAND' in ....` 오류가 발생합니다.

명령줄에서 `php -m`을 실행하면 설치된 php cli의 모든 확장이 나열됩니다. 만약 pdo 또는 pdo_mysql이 없다면 직접 설치하셔야 합니다.

**CentOS 시스템**
PHP5.x
```bash
yum install php-pdo
yum install php-mysql
```
PHP7.x
```bash
yum install php70w-pdo_dblib.x86_64
yum install php70w-mysqlnd.x86_64
```
패키지 이름을 찾을 수 없는 경우에는 `yum search php mysql`을 사용해 보십시오.

**Ubuntu/Debian 시스템**
PHP5.x
```bash
apt-get install php5-mysql
```
PHP7.x
```bash
apt-get install php7.0-mysql
```
패키지 이름을 찾을 수 없는 경우에는 `apt-cache search php mysql`를 사용해 보십시오.

**상기 방법으로 설치할 수 없는 경우?**
위의 방법으로 설치할 수 없는 경우에는 [workerman 매뉴얼 - 부록 - 확장 설치 - 방법 삼 소스 코드 컴파일 설치]를 참고하십시오.

## Workerman/MySQL 설치
**방법 1:**
Composer를 사용하여 설치할 수 있으며, 다음 명령을 명령줄에서 실행할 수 있습니다(composer 소스는 외국에 있기 때문에 설치 과정이 매우 느릴 수 있음).
```bash
composer require workerman/mysql
```
위 명령이 성공하면 vendor 디렉토리가 생성되고, 프로젝트에서 vendor 아래의 autoload.php를 가져오면 됩니다.
```php
require_once __DIR__ . '/vendor/autoload.php';
```

**방법 2:**
[소스 코드 다운로드](https://github.com/walkor/mysql/archive/master.zip)를 통해 소스를 다운로드하고 풀어서 프로젝트 디렉토리에 넣은 다음 소스 파일을 직접 가져오면 됩니다.
```php
require_once '/your/path/of/mysql-master/src/Connection.php';
```

## 주의
onWorkerStart 콜백에서 데이터베이스 연결을 초기화하는 것을 강력히 권장합니다. Worker::runAll();을 실행하기 전에 연결을 초기화하면 부모 프로세스에서 초기화되며 자식 프로세스가 이 연결을 상속받아 동일한 데이터베이스 연결을 공유하면 오류가 발생할 수 있습니다.

## 예제
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    // db 인스턴스를 전역 변수에 저장 (또는 특정 클래스의 정적 멤버에 저장할 수도 있음)
    global $db;
    $db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // 전역 변수를 통해 db 인스턴스 가져오기
    global $db;
    // SQL 실행
    $all_tables = $db->query('show tables');
    $connection->send(json_encode($all_tables));
};
// Worker 실행
Worker::runAll();
```
## 구체적인 MySQL/Connection 사용법
```php
// db 연결 초기화
$db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');

// 모든 데이터 가져오기
$db->select('ID,Sex')->from('Persons')->where('sex= :sex AND ID = :id')->bindValues(array('sex'=>'M', 'id' => 1))->query();
//같음
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' AND ID = 1")->query();
//같음
$db->query("SELECT ID,Sex FROM `Persons` WHERE sex='M' AND ID = 1");

// 한 행 데이터 가져오기
$db->select('ID,Sex')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->row();
//같음
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' ")->row();
//같음
$db->row("SELECT ID,Sex FROM `Persons` WHERE sex='M'");

// 한 열 데이터 가져오기
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->column();
//같음
$db->select('ID')->from('Persons')->where("sex= 'F' ")->column();
//같음
$db->column("SELECT `ID` FROM `Persons` WHERE sex='M'");

// 단일 값 가져오기
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->single();
//같음
$db->select('ID')->from('Persons')->where("sex= 'F' ")->single();
//같음
$db->single("SELECT ID FROM `Persons` WHERE sex='M'");

// 복잡한 쿼리
$db->select('*')->from('table1')->innerJoin('table2','table1.uid = table2.uid')->where('age > :age')->groupBy(array('aid'))->having('foo="foo"')->orderByASC/*orderByDESC*/(array('did'))
->limit(10)->offset(20)->bindValues(array('age' => 13));
// 같음
$db->query('SELECT * FROM `table1` INNER JOIN `table2` ON `table1`.`uid` = `table2`.`uid`
WHERE age > 13 GROUP BY aid HAVING foo="foo" ORDER BY did LIMIT 10 OFFSET 20');

// 삽입
$insert_id = $db->insert('Persons')->cols(array(
    'Firstname'=>'abc',
    'Lastname'=>'efg',
    'Sex'=>'M',
    'Age'=>13))->query();
같음
$insert_id = $db->query("INSERT INTO `Persons` ( `Firstname`,`Lastname`,`Sex`,`Age`)
VALUES ( 'abc', 'efg', 'M', 13)");

// 업데이트
$row_count = $db->update('Persons')->cols(array('sex'))->where('ID=1')
->bindValue('sex', 'F')->query();
// 같음
$row_count = $db->update('Persons')->cols(array('sex'=>'F'))->where('ID=1')->query();
// 같음
$row_count = $db->query("UPDATE `Persons` SET `sex` = 'F' WHERE ID=1");

// 삭제
$row_count = $db->delete('Persons')->where('ID=9')->query();
// 같음
$row_count = $db->query("DELETE FROM `Persons` WHERE ID=9");

// 트랜잭션
$db->beginTrans();
....
$db->commitTrans(); // 또는 $db->rollBackTrans();
```

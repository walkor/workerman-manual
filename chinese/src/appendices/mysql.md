## Gateway/Worker模型 数据库使用示例

1、数据库配置Applications/XXX/Config/Db.php
```php
<?php
namespace Config;
/**
 * mysql配置
 * @author walkor
 */
class Db
{
    /**
     * 数据库的一个实例配置，则使用时像下面这样使用
     * $user_array = Db::instance('user')->select('name,age')->from('users')->where('age>12')->query();
     * 等价于
     * $user_array = Db::instance('user')->query('SELECT `name`,`age` FROM `users` WHERE `age`>12');
     * @var array
     */
    public static $user = array(
        'host'    => '127.0.0.1',
        'port'    => 3306,
        'user'    => 'your_user_name',
        'password' => 'your_password',
        'dbname'  => 'user',
        'charset'    => 'utf8',
    );
}
```


2、Applications/XXX/Event.php
```php
<?php
use \GatewayWorker\Lib\Gateway;
use \GatewayWorker\Lib\Db;

/**
 * 数据库示例，假设有个user库，里面有个user表
 */
class Event
{

   /**
    * 有消息时触发该方法，根据发来的命令打印2个用户信息
    * @param int $client_id 发消息的client_id
    * @param string $message 消息
    * @return void
    */
   public static function onMessage($client_id, $message)
   {
        // 发来的消息
        $commend = trim($message);
        if($commend !== 'get_user_list')
        {
            Gateway::sendToClient($client_id, "unknown commend\n");
            return;
        }
        // 获取用户列表（这里是临时的一个测试数据库）
        $ret = Db::instance('user')->select('*')->from('users')->where('uid>3')->offset(5)->limit(2)->query();
        // 打印结果
        return Gateway::sendToClient($client_id, var_export($ret, true));
   }

}
```

# 数据库类使用的一些示例
## 配置
在Config/Db.php中配置数据库信息，如果有多个数据库，可以在Db.php中配置多个实例
例如下面配置了两个数据库实例

```php
<?php
namespace Config;
class Db
{
    // 数据库实例1
    public static $db1 = array(
        'host'    => '127.0.0.1',
        'port'    => 3306,
        'user'    => 'mysql_user',
        'password' => 'mysql_password',
        'dbname'  => 'db1',
        'charset'    => 'utf8',
    );

    // 数据库实例2
    public static $db2 = array(
        'host'    => '127.0.0.1',
        'port'    => 3306,
        'user'    => 'mysql_user',
        'password' => 'mysql_password',
        'dbname'  => 'db2',
        'charset'    => 'utf8',
    );
}
```
## 使用方法

```php
use \GatewayWorker\Lib\Db;
$db1 = Db::instance('db1');
$db2 = Db::instance('db2');

// 获取所有数据
$db1->select('ID,Sex')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->query();
//等价于
$db1->select('ID,Sex')->from('Persons')->where("sex= 'F' ")->query();
//等价于
$db1->query("SELECT ID,Sex FROM `Persons` WHERE sex=‘M’");


// 获取一行数据
$db1->select('ID,Sex')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->row();
//等价于
$db1->select('ID,Sex')->from('Persons')->where("sex= 'F' ")->row();
//等价于
$db1->row("SELECT ID,Sex FROM `Persons` WHERE sex=‘M’");


// 获取一列数据
$db1->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->column();
//等价于
$db1->select('ID')->from('Persons')->where("sex= 'F' ")->column();
//等价于
$db1->column("SELECT `ID` FROM `Persons` WHERE sex=‘M’");

// 获取单个值
$db1->select('ID,Sex')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->single();
//等价于
$db1->select('ID,Sex')->from('Persons')->where("sex= 'F' ")->single();
//等价于
$db1->single("SELECT ID,Sex FROM `Persons` WHERE sex='M'");

// 复杂查询
$db1->select('*')->from('table1')->innerJoin('table2','table1.uid = table2.uid')->where('age > :age')->groupBy(array('aid'))->having('foo="foo"')->orderBy(array('did'))->limit(10)->offset(20)->bindValues(arra
y('age' => 13));
// 等价于
$db1->query(SELECT * FROM `table1` INNER JOIN `table2` ON `table1`.`uid` = `table2`.`uid` WHERE age > 13 GROUP BY aid HAVING foo="foo" ORDER BY did LIMIT 10 OFFSET 20“);

// 插入
$insert_id = $db1->insert('Persons')->cols(array('Firstname'=>'abc', 'Lastname'=>'efg', 'Sex'=>'M', 'Age'=>13))->query();
等价于
$insert_id = $db1->query("INSERT INTO `Persons` ( `Firstname`,`Lastname`,`Sex`,`Age`) VALUES ( 'abc', 'efg', 'M', 13)");

// 更新
$row_count = $db1->update('Persons')->cols(array('sex'))->where('ID=1')->bindValue('sex', 'F')->query();
// 等价于
$row_count = $db1->update('Persons')->cols(array('sex'=>'F'))->where('ID=1')->query();
// 等价于
$row_count = $db1->query("UPDATE `Persons` SET `sex` = 'F' WHERE ID=1");

// 删除
$row_count = $db1->delete('Persons')->where('ID=9')->query();
// 等价于
$row_count = $db1->query("DELETE FROM `Persons` WHERE ID=9");

```

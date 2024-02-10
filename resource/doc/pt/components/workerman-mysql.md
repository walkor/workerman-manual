# Workerman/MySQL

## Descrição
Os programas residentes na memória frequentemente encontram o erro "mysql gone away" ao usar o MySQL, o qual é causado pela falta de comunicação entre o programa e o MySQL durante longos períodos, resultando na desconexão por parte do servidor MySQL. Esta classe de banco de dados pode resolver esse problema, pois tenta reconectar automaticamente quando o erro "mysql gone away" ocorre.

## Dependências
Esta classe MySQL depende de duas extensões: [pdo](https://php.net/manual/zh/book.pdo.php) e [pdo_mysql](https://php.net/manual/zh/ref.pdo-mysql.php). Se essas extensões estiverem ausentes, um erro do tipo "Undefined class constant 'MYSQL_ATTR_INIT_COMMAND' in ...." será reportado.

Para listar todas as extensões PHP já instaladas no CLI, execute o comando `php -m`. Se pdo ou pdo_mysql estiverem ausentes, instale-os manualmente:

**Sistema CentOS**

PHP 5.x
```bash
yum install php-pdo
yum install php-mysql
```

PHP 7.x
```bash
yum install php70w-pdo_dblib.x86_64
yum install php70w-mysqlnd.x86_64
```

Se não encontrar os nomes dos pacotes, tente procurar usando `yum search php mysql`.

**Sistema Ubuntu/Debian**

PHP 5.x
```bash
apt-get install php5-mysql
```

PHP 7.x
```bash
apt-get install php7.0-mysql
```

Se não encontrar os nomes dos pacotes, tente procurar usando `apt-cache search php mysql`.

**Incapaz de instalar usando os métodos acima?**

Se os métodos acima não funcionarem, consulte o [manual do Workerman - Apêndice - Instalação de Extensões - Método Três: Compilação e Instalação do Código Fonte](../appendices/install-extension.md).

## Instalação do Workerman/MySQL
**Método 1:**

Pode ser instalado via Composer. Execute o seguinte comando no terminal (a instalação pode ser lenta devido à fonte do Composer no exterior).

```bash
composer require workerman/mysql
```

Após a execução bem-sucedida desse comando, a pasta "vendor" será criada. Em seguida, inclua o arquivo "autoload.php" do diretório "vendor" em seu projeto:

```php
require_once __DIR__ . '/vendor/autoload.php';
```

**Método 2:**

[Baixe o código-fonte](https://github.com/walkor/mysql/archive/master.zip), descompacte o diretório em seu projeto (em qualquer local) e faça um require direto do arquivo de origem.

```php
require_once '/caminho/do/seu/mysql-master/src/Connection.php';
```

## Atenção
É altamente recomendável inicializar a conexão com o banco de dados no callback onWorkerStart para evitar a inicialização da conexão antes da execução de `Worker::runAll();`. Conexões inicializadas antes da execução de `Worker::runAll();` pertencem ao processo principal e serão herdadas pelos sub-processos, resultando em erros causados pelo compartilhamento da mesma conexão de banco de dados entre o processo principal e os sub-processos.

## Exemplo
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    // Armazenar a instância do banco de dados em uma variável global (também pode ser armazenada em uma propriedade estática de uma classe)
    global $db;
    $db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Obter a instância do banco de dados através da variável global
    global $db;
    // Executar SQL
    $all_tables = $db->query('show tables');
    $connection->send(json_encode($all_tables));
};
// Executar o worker
Worker::runAll();
```

## Uso específico do MySQL/Connection
```php
// Inicializar a conexão com o banco de dados
$db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');

// Obter todos os dados
$db->select('ID,Sex')->from('Persons')->where('sex= :sex AND ID = :id')->bindValues(array('sex'=>'M', 'id' => 1))->query();
// Equivalente a
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' AND ID = 1")->query();
// Equivalente a
$db->query("SELECT ID,Sex FROM `Persons` WHERE sex='M' AND ID = 1");


// Obter uma linha de dados
$db->select('ID,Sex')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->row();
// Equivalente a
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' ")->row();
// Equivalente a
$db->row("SELECT ID,Sex FROM `Persons` WHERE sex='M'");


// Obter uma coluna de dados
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->column();
// Equivalente a
$db->select('ID')->from('Persons')->where("sex= 'F' ")->column();
// Equivalente a
$db->column("SELECT `ID` FROM `Persons` WHERE sex='M'");

// Obter um único valor
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->single();
// Equivalente a
$db->select('ID')->from('Persons')->where("sex= 'F' ")->single();
// Equivalente a
$db->single("SELECT ID FROM `Persons` WHERE sex='M'");

// Consulta complexa
$db->select('*')->from('table1')->innerJoin('table2','table1.uid = table2.uid')->where('age > :age')->groupBy(array('aid'))->having('foo="foo"')->orderByASC/*orderByDESC*/(array('did'))
->limit(10)->offset(20)->bindValues(array('age' => 13));
// Equivalente a
$db->query('SELECT * FROM `table1` INNER JOIN `table2` ON `table1`.`uid` = `table2`.`uid`
WHERE age > 13 GROUP BY aid HAVING foo="foo" ORDER BY did LIMIT 10 OFFSET 20');

// Inserir
$insert_id = $db->insert('Persons')->cols(array(
    'Firstname'=>'abc',
    'Lastname'=>'efg',
    'Sex'=>'M',
    'Age'=>13))->query();
Equivalente a
$insert_id = $db->query("INSERT INTO `Persons` ( `Firstname`,`Lastname`,`Sex`,`Age`)
VALUES ( 'abc', 'efg', 'M', 13)");

// Atualizar
$row_count = $db->update('Persons')->cols(array('sex'))->where('ID=1')
->bindValue('sex', 'F')->query();
// Equivalente a
$row_count = $db->update('Persons')->cols(array('sex'=>'F'))->where('ID=1')->query();
// Equivalente a
$row_count = $db->query("UPDATE `Persons` SET `sex` = 'F' WHERE ID=1");

// Excluir
$row_count = $db->delete('Persons')->where('ID=9')->query();
// Equivalente a
$row_count = $db->query("DELETE FROM `Persons` WHERE ID=9");

// Transação
$db->beginTrans();
....
$db->commitTrans(); // or $db->rollBackTrans();
```

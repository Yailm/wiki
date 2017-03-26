* [DBI](#dbi)
    * [执行SQL语句](#执行sql语句)
        * [处理引号](#处理引号)
        * [占位符](#占位符)
    * [错误信息](#错误信息)
    * [绑定列](#绑定列)
    * [数据库事务](#数据库事务)

## DBI
```perl
my $dbh = DBI->connect("DBI:mysql:perl", 'root', '') or die "Connection to perl failed; $DBI:errstr";
my $sth = $dbh->prepare("select * from coaches");
$sth->execute;

print "<div align='center'>";
print h2("Contents of the \"coaches\" Table");

$sth->finish;
$dbh->disconnect;
```
### 执行SQL语句

- prepare()与execute()方法
  - 适用于SELECT语句
  - 成功执行返回一个结果集（在DBI表示为$sth）
  - execute如果发生错误的话，便会返回一个undef，成功执行时总会返回真，不论影响到多少内容（即使是0行）
  - SELECT查询返回的结果集可以使用一些特殊方法来解析，例如fetchrow_array()和fetchrow_hashref()
  - prepare_cache()方法类似prepare方法，所不同它会查看以前是否执行过相同的SQL语句，如果有，则会提供缓存的语句句柄而不是创建新的句柄
- do()方法
  - 适用于不返回结果集的SQL语句，例如INSERT,UPDATE,DELETE语句
  - 返回影响到的行数，而执行失败的查询则会返回一条错误或undef
  - do()的缺点是它的性能，尤其在使用占位符的时候，每个查询它都得重复准备，执行等步骤

#### 处理引号

要把字符串发送到数据库，必须置于引号内，然而字符串本身也可能含有引号，这些引号必须以适当的方式予以舍弃，同时不同的数据库引号的规则也可能不同。这里我们调用quote方法来处理引号问题
```perl
$namestring = qq(Jerry O'connell);      # 相当于单引号了
$namestring= $dbh->quote($namestring);
$sth = $dbh->prepare("select * from coaches where name=$namestring");
$sth->execute();
```
#### 占位符

使用了占位符的语句相当于定制了计划，且数据库不会在查询完毕后立刻丢弃。占位符只能表示值，不能是字段或其他的，而execute会接受表示占位符的参数
```perl
$sth = $dhb->prepare("select name, wins, losses from teams where name=?");
$sth->execute($team_name);

my $sth = $dbh->prepare("insert into teams values(?, ?, ?)");
$sth->execute($team_name, $wins, $losses);

$dbh->do(qq/insert into teams values (?, ?, ?)/, undef, "Middle Monsters", 2, 32);
# undef那里可以放一些属性组成的匿名哈希
```
我们还有其他替换占位符的方法，利用bind_param()，它可以接受三个参数
  - 第一个为占位符的编号，即选择第几个占位符，从1开始
  - 第二个为要替换占位符的实际值
  - 第三个（可选）为占位符的类型

### 错误信息

* 在连接数据库的时候还可以同时设置一些属性，包括PrintError和RaiseError
  * PrintError默认为on，在任意DBI方法失败的时候自动生成一条警告信息
  * RaiseError默认为off，可用于强制生成错误并提供异常。如果被设置成on，则任何引发错误的DBI方法会造成输出错误信息（即$DBI::error）并退出程序。
  * 优先级PrintError会高于RaiseError，如果同时开启，则优先处理PrintError属性
* 错误诊断方法
  * err()对应错误代码数值，不过它返回的是一个不含有信息的字符串，描述错误信息
  * errstr()对应错误代码数值，不过它返回是一个不含有信息的字符串，描述错误信息
  * 在下一个DBI方法调用之前，这些错误信息都和被重新设置
* 错误诊断变量
  * $DBI::err和$DBI::errstr是类变量，功能就是类似前面所述的两个方法
    ```perl
    $dbh = DBI->connect("DBI:mysql:perl", 'root', '',
                        {
                            RaiseError => 1,    # Die if there are errors
                            PrintError => 0,    # Warn if there are errors
                        }
                        ) or die $DBI::errstr;
    $sth = $dbh->prepare("select name, wins, losses from teams")
        or die "Can't prepare sql statement", DBI->errstr;
    ```

### 绑定列

Perl支持将某个变量关联的数据库中的一个字段（列）值上。当获取该值的时候，对应变量自动更新为检索得到的值

DBI提供bind_columns()和fetch()方法来实现这一操作
```perl
my ($name, $wins, $losses);
$sth->bind_columns(\$name, \$wins, \$losses);
while ($sth->fetch()) {     # 每次调用，三个绑定的变量都会更新
    # fetch arow and return column value as scalars
    printf " %-25s%3d%8d\n", $name, $wins, $losses;
}
```
### 数据库事务

我们有时需要将一系列操作分组，如果其中一个语句执行失败的话，则其他所有语句都不会执行

默认情况下qutocommit是开启的，要使用事务机制，就必须关闭该模式。我们可以在连接数据的时候设置 `autocommit => 0` ，这时我们就要自己手动提交和回滚了
```perl
$dbh->rollback();
$dbh->commit()`
```
通过利用eval一整组execute（do）语句，$@的值判断是否发生了错误，再决定执行rollback或是commit
```perl
use DBI qw(:sql_types);
my $dbh = DBI->connect('DBI:mysql:perl', 'root', '', {
        PrintError => 1,
        AutoCommit => 0,
    });
my @rows = (
    ['Tampa Terrors', 3, 5],
    ['Los Alamos Lizzards', 12, 3],
    ['Detroit Demons', 22, 0],
    ['Cheyenne Chargers', 6, 0],
);

my $sql = qq{insert into teams values (?, ?, ?)};
my $sth = $dbh->prepare($sql);
foreach $param (@rows) {
    eval {
        $sth->bind_param(1, $param->[0], SQL_VARCHAR);
        $sth->bind_param(2, $param->[1], SQL_INTEGER);
        $sth->bind_param(3, $param->[2], SQL_INTEGER);
        $sth->execute or die;
    };
}
if ($@) {
    warn "Database error: $DBI::errstr\n";
    $dbh->rollback();
} else {
    $dbh->commit;
    print "Success!\n";
}
$sth->finish;
$dbh->disconnect;
```

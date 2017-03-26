* [语法](#语法)
    * [循环](#循环)
        * [循环控制](#循环控制)
        * [循环的区别](#循环的区别)
    * [分支](#分支)
        * [三目操作符](#三目操作符)
        * [given-when](#given-when)
        * [变量值的确定](#变量值的确定)
    * [报表](#报表)
    * [按位操作符](#按位操作符)
    * [do语句块](#do语句块)
    * [BEGIN和END](#begin和end)
    * [倒装句](#倒装句)
    * [排序](#排序)
    * [智能匹配操作符](#智能匹配操作符)
    * [一点函数](#一点函数)
        * [时间函数](#时间函数)
        * [系统相关](#系统相关)
            * [chdir](#chdir)
            * [通配](#通配)
            * [文件操作](#文件操作)
            * [运行外部程序](#运行外部程序)
            * [信号](#信号)
            * [获取用户信息](#获取用户信息)
            * [进程信息](#进程信息)
        * [字符串函数](#字符串函数)
            * [index & rindex](#index-&-rindex)
            * [substr](#substr)
            * [sprintf & printf](#sprintf-&-printf)
            * [oct & hex](#oct-&-hex)
            * [tr & y](#tr-&-y)
        * [算数函数](#算数函数)

## 语法

### 循环

Perl中有5种循环(for, foreach, while, until, 裸块)
```perl
foreach $rock (qw/ bedrock slate lava /) {}
```
$rock为控制变量，在循环执行的时候都会取得新的值，但是其并不是列表的复制品，实际上它就是列表元素本身。所以如果在循环中改变它的值，列表中的元素也会改变，当循环结束的时候，控制变量也会恢复循环之前的值，如果这个变量是新定义的，那此时就是undef
```perl
foreach (1..10) {
    print;
}   # 一般函数不加参数时自动指定$_，如chomp等等
```
#### 循环控制

* last相当于其他语言的break一样，可以中断循环的执行，而且也可以跳出裸块
* next相当于其他语言的continue，跳过这次循环
* redo能将控制返回到当前循环块的顶端，相当于重新开始本次循环
* 标签能够从内层对外层的循环进行控制，标签由字母、数字和下划线组成，不能以数字开头，通常都是全大写字母。在循环中有需要就可以在last、next或redo的后面加上这个标签
  ```perl
  LINE: while (<>) {
      foreach (split) {
          last LINE if /__END__/;
          next LINE;
          redo LINE;
      }
  }
  ```
* continue代码块执行了next的情况下被调用也可以执行
  ```perl
  while ( ) {
    ...
  } continue {   }
  # 每次while的语句块执行完毕都会去执行continue语句块，即使执行了next
  ```

#### 循环的区别
```perl
while (<STDIN>) {
    print "I saw $_\n";
}   # 这会每输入一行就输出一次
foreach (<STDIN>) {
    print "I saw $_\n";
}   # 当输入完毕的时候才输出
```
### 分支

#### 三目操作符
```perl
$expression ? $if_true_expr : $if_false_expr;
```
```perl
my $size =
    ($width < 10) ? "smail" :
    ($width < 20) ? "medium" :
    ($width < 50) ? "large" :
                    "extra-large";
```
#### given-when

given-when控制结构类似switch语句

given会将参数化名为$_，每个when条件都尝试用 **智能匹配 ~~** 对参数作测试
```perl
use 5.010001;
given ( $ARGV[0] ) {
    when ( 'Fred' ) { say 'Name is Fred' }
    when ( /fred/ ) { say 'Name has fred in it' }
    when ( /\AFred/ ) { say 'Name starts with Fred' }
    default     { say "I don't see a Fred" }
}
```
given-when是在每一分支上默默加上了break，跳出这个结构。但是我们可以显式的在每个分支加上continue，这样执行语句后就不会自动跳出结构了

PS. 每一句when语句都可以看成是单独的if语句，这和某些语言的switch不一样，贯穿也必须要考虑条件。还有，default相当是一个永远为真的when语句

如果我们要判断的数就在$_中，我们就能省略given这个语句了
```perl
foreach (@names) {
    # 当前遍利元素的名称就保存在$_中，省略了given语句
    # when语句前后都插入普通语句
    when ( /fred/i ) { ... }
    when ( /\AFred/ ) { ... }
}
```
#### 变量值的确定
```perl
($m < $n) && ($m = $n);
if ($m < $n) { $m = $n }
$m = $n if $m < $n;
# 这三句话意思是一样的
```
    my $last_name = $last_name{$someone} || '(no last name)';

以上的例子可以让变量使用允许使用的值或是默认值，我们也可以选择and和or，效果是一样的，不过优先级要低得多。但有个问题，在||前，0或空串等等这些虽然已经定义的值，都会被判断成false，从而使用默认值

这时有了//，在perl 5.010，这个操作符能够解决上面的问题，只要变量已经定义，就会返回逻辑真
```perl
use 5.010;
my $last_name = $last_name($someone) // '(no last name)';
---
printf "%s", $name // '';
```
### 报表

报表能编写具有一定格式的报告，组织和总结这些信息
```perl
format  FILEHANDLE=         # 启动格式，FILEHANDLE是输出的句柄，默认是STDOUT（可以省略）
picture line                # 用来描述输出的格式
value line (text to formatted)
                            # 用逗号隔开，并且与形象行中的字段符号一一对应，忽略值行中的任何空白字符
    .                       # 结束模板的定义
write FILEHANDLE;           # 输出，同样STDOUT可以省略
```
字段标志符即@，按照字段符号类型的字符数重复它，就是设定字段的宽度

| 字段标志符 | 描述           |
|------------|----------------|
| @          | 表示字段的开始 |
| @*         | 用于多行字段   |
| ^          | 用与填充字段   |
如果用多行字符串放进@中，只会显示第一行

| 字段显示符号 | 描述             |
|--------------|------------------|
| <            | 左对齐           |
| >            | 右对齐           |
| 竖线         | 居中             |
| #            | 数字             |
| .            | 指明小数点的位置 |

### 按位操作符

| 符号表示 | 操作名称 | 说明                                |
|----------|----------|-------------------------------------|
| n & m    | 按位或   | 哪些位在两边同时真                  |
| n 竖线 m | 按位或   | 哪些位在任一边为真                  |
| n ^ m    | 按位异或 | 哪些位在只有一边为真                |
| n << m   | 按位左移 | 将n向左移动m位，并由0填补最低有效位 |
| n >> m   | 按位右移 | 丢弃移出的最低有效位                |
| ~m       | 按位取反 | 也称取反码                          |

### do语句块

do语句块，相当于一个内嵌的子程序，返回值和子程序的类似
```perl
my $bowler = do {
    if ($condition) { 'Mary Ann' }
    elsif ($condition2) { 'Ginger' }
    else { 'The Professor' }
};

# 再比如将文件内容全部读入一个变量
my $file_contents = do {
    local $/;
    # $/是一个输入记录分隔符，默认为换行符，这样一次一行
    # local $/相当于把$/初始化，设置为""，通常用于将文件内容全部读取
    local @ARGV = ($filename);
    <>;
}
```
和eval一样，do后面也可以加上字符串代替语句块。这会尝试载入字符串对应的文件，编译和运行里面的代码`do $filename`。实际上，do操作符找到并读取文件里的内容，并将内容交给eval来执行

### BEGIN和END

BEGIN会在程序开始前立刻执行，多个BEGIN按照定义的顺序执行

END会在程序所有任务完成后执行（包括die语句退出也会执行），多个END与定义顺序相反执行
```perl
END {print 1};
END {print 2};
END {print 3};
--- output ---
321
```
### 倒装句
```perl
print $some if ($expression);
&func a, b until $n < 10;
$i * 2 unless i > 5;
&greet($_) foreach @person;
```
### 排序

Perl知道如何排序，但不明确我们要的排序规则，所以排序的时候我们只要提供排序子程序就行了。排序子程序在排序过程中会被多次调用，要求效率必须要高。

排序子程序中有两个定义好的变量a和b，不是是数据的拷贝，而是原始列表的别名，修改它们会弄乱原始数据。我们要做的就是让子程序返回一个数字，描述两个元素之间的比较

**$a在$b之前，返回-1，$b在$a之前，返回1，顺序无所谓返回0**
```perl
sub by_number {
    if ($a < $b) { -1 } elsif ($a > $b) { 1 } else { 0 }
}
my @result = sort by_number @some_numbers;
# 上面的子程序可以直接使用飞船符号<=>，这是个用来默认比较数字
# 所以写成
sub by_number { $a <=> $b }
# 对飞船操作符的理解对$a，$b分别进行<, =, >，然后返回-1，0，-1
# 还有一种三路操作符cmp，用来比较字符串，sort默认的就是这个
# 要对Unicode字符排序，通常会写成下面这样
use Unicode:Normalize;
sub equivalents { NFKD($a) cmp NFKD($b) }
# 多个条件排序
sub by_score_and_name {
    $score{$b} <=> $score{$a}
        or
    $a cmp $b
}
```
### 智能匹配操作符

`~~`智能匹配操作符会根据两边的操作数的数据类型自动判断该用何种方式进行比较或匹配，`use 5.010001`至少是5.10.1版
```perl
# 可以代替绑定操作符
say " I found Fred in the name" if $name ~~ /fred/;
# 更加超过的事
say "I found a key matching fred" if %names ~~ /fred/;
# 相当于这样的程序
my $flag = 0;
foreach my $key (keys %names) {
    next unless $key =~ /fred/;
    $flag = $key;
    last;
}
```
| Left   | Right  | Description and pseudocode                                                                                              |
|--------|--------|-------------------------------------------------------------------------------------------------------------------------|
| Any    | undef  | check whether Any is undefined<br>like: `!defined Any`                                                                  |
| Any    | Object | invoke `~~` overloading on Object, or die                                                                               |
|        | Array  |                                                                                                                         |
| Array1 | Array2 | recurse on paired elements of Array1 and Array2<br>like: `(Array1[0] ~~ Array2[0]) && (Array1[1] ~~ Array2[1]) && ... ` |
| Hash   | Array  | any Array elements exist as Hash keys<br>like: `grep {exist Hash->{$_} } Array`                                         |
| Regexp | Array  | any Array elements pattern match Regexp<br>like: `grep { /Regexp/ } Array`                                              |
| undef  | Array  | under in Array<br>like: `grep { !defined } Array`                                                                       |
| Any    | Array  | smartmatch each Array elements<br>like: `grep { Any ~~ $_ } Array`                                                      |
|        | Hash   |                                                                                                                         |
| Hash1  | Hash2  | all same keys in both Hashes<br>like: `keys Hash1 == grep {exists Hash2->{$_} } keys Hash1`                             |
| Array  | Hash   | any Array elements exist at Hash keys<br>like: `grep { exists Hash->{$_} } Array`                                       |
| Regexp | Hash   | any Hash keys pattern match Regexp<br>like: `grep { /Regexp/ } keys Hash`                                               |
| undef  | Hash   | always false (undef can't be a key)                                                                                     |
| Any    | Hash   | Hash key existence<br>like: `exists Hash->{Any}`                                                                        |
|        | Code   |                                                                                                                         |
| Array  | Code   | sub returns true on all Array elements<br>like: `!grep { !Code->($_) } Array`                                           |
| Hash   | Code   | sub returns true on all Hash keys<br>like: `!grep { !Code->($_) } keys Hash`                                            |
| Any    | Code   | sub passed Any return true<br>like: `Code->(Any)`                                                                       |
|        | Regexp |                                                                                                                         |
| Array  | Regexp | any Array elements match Regexp<br>like: `grep { /Regexp/ } Array`                                                      |
| Hash   | Regexp | any Hash keys match Regexp<br>like: `grep { /Regexp/ } keys Hash`                                                       |
| Any    | Regexp | pattern match                                                                                                           |
|        | Other  |                                                                                                                         |
| Object | Any    | invoke `~~` overloading on Object or fall back to                                                                       |
| Any    | Num    | numeric equality<br>like: `Any == Num`                                                                                  |
| Num    | nummy  | numeric equality<br>like: `Num == nummy`(a string made by numbers)                                                      |
| undef  | Any    | check whether undefined<br>like: `!defined(Any)`                                                                        |
| Any    | Any    | string equality<br>like: `Any eq Any`                                                                                   |

### 一点函数

#### 时间函数

+ localtime函数可以将epoch时间转化成比较容易阅读的形式
  ```perl
  my ($sec, $min, $hour, $day, $mon, $year, $wday, $yday, $isdst) = localtime $timestamp;
  # $timestamp就是epoch形式的时间了
  ```
  * mon 0..11
  * year 从1900算起的年数，必须加上1900才是实际年份
  * wday 0-6（星期日-星期六）
  * yday 0-364(365)表示目前是今年的第几天
+ gmtime函数返回的是世界标准时间（即格林威治标准时间）
+ time返回从系统时钟取得的时间戳
+ times函数返回一个4元素数组
  + 用户时间：用于执行用户代码的时间
  + 系统时间：用于执行系统调用的时间
  + 子进程的用户时间：所有已终止的子进程的总执行耗时
  + 子进程的系统时间：所有已终止的子进程执行系统调用的耗时

PS. 在不提供参数的情况下，localtime和gmtime函数，默认情况下都使用当前time函数返回的值

#### 系统相关

##### chdir

chdir操作和linux中的cd差不多，如果不加参数，就会跳会用户主目录，但是不能使用波浪线
```perl
chdir '/etc' or die "cannot chdir to /etc: $!";
```
##### 通配

glob函数能够些shell一样处理通配符，*表示任意字符，?表示单个字符，[]提供选择
```perl
# 这会列出当前文件所有内容，不包括以点号开头的文件
my @all_files = glob '*';
# 若要一次匹配多个模式，可以在参数中用空格隔开各个模式
my @all_files_including_dot = glob '.* *';
```
PS. 尖括号也可以处理文件名的通配，`my @all_files = <* .*>`，效果完全一样
  - 间接文件句柄读取，当<\>内仅是一个简单标量（不是哈希或是数组元素）
    ```perl
    my $name = 'FRED';
    my @lines = <$name>;  # 对句柄FRED进行间接文件句柄读取
    # 也可以使用readline操作符执行
    my $name = 'FRED';
    my @lines = readline FRED;
    my @lines = readline $name;   # 和上面一句是一样的
    ```

##### 文件操作

* unlink操作用来删除单个或多个文件，就是从目录里移除文件名的链接条目，并将它的链接数递减，必要时再释放inode
  ```perl
  unlink 'slate', 'bedrock', 'lava';
  unlink qw(slate bedrock lava);
  unlink glob '*.o';
  # 返回值是成功删除的文件数目
  my $success = unlink 'slate', 'bedrock', 'lava';
  print "I delete $success file(s) just now\n";
  # 我们是无法知道删除失败的是哪些文件，如果要知道只能循环每次删除一个才行
  foreach my $file (qw(slate bedrock lava)) {
      unlink $file or warn "failed on $file: $!\n";
  }
  ```
  PS. linux下删除文件，和文件本身的权限无关，和文件所在的上层目录有关

* rename重命名文件，`rename 'old', 'new'`或是使用胖箭头`rename 'old' => 'new'`，本质就是创建一个指向现有文件的新链接，然后删除原有文件

  其实这个函数和mv是一样的，所以你可以把文件移动到另一个目录`rename '/tmp/a', 'b'`

  同样有返回值，失败时返回假
  ```perl
  # 批量把名称.old结尾的文件改名为.new结尾
  foreach my $file (glob "*.old") {
      (my $newfile = $file) =~ s/\.old/\.new/r;
      # 也可以写my $newfile = $file =~ s/\.old/\.new/r;
      # 优先级是=~比较高，先执行后半句
      # 注意/r修饰符 perform non-destructive substitution and return the new value
      # 非破坏性替换，返回新的值，所以$file未被修改，并把结果赋值给了$newfile
      if (-e $newfile) {
          warn "can't rename $file to $newfile, $newfile exist\n";
      } elsif (rename $file => $newfile) {
          # 改名成功
      } else {
          warn "rename $file to $newfile failed: $!\n";
      }
  }
  ```
* link函数可以建立一个硬链接
  ```perl
  link 'old', 'new'
      or warn "can't link old to new: $!";
  ```
* symlink函数可以建立一个符号链接，如同`ln -s`，符号链接不会增加inode的链接数

  符号链接可以跨文件系统，也能为目录建立链接，指向任何文件名，甚至是不存在的文件名，但是不能防止数据丢失。如果目标文件丢失，`-l $symlink`仍然返回真，而`-e $symlink`则会返回假

* readlink会返回符号链接指向的位置，如果参数不是符号链接，则返回undef

* mkdir创建文件夹
  ```perl
  mkdir 'fred', 0755 or warn "Cannot make fred directory: $!";
  # 0755的0代表这是个八进制数，字符串则需要oct()函数，在windows下不用设置权限
  my ($name, $perm) = @ARGV;
  mkdir $name, oct($perm);
  ```
* rmdir每次只能删除一个目录，而且是空目录

* chmod修改文件或目录的权限

* umask用来获取或设置umask值

* chown修改一系列文件的拥有者以及所属组
  ```perl
  chmod 0755, 'fred', 'barney';   # 会返回成功更改的条目数目
  ---
  my $uid = 1004;
  my $gid = 100;
  chown $uid, $gid, glob '*.o';
  # 如果不知道标识符，可以使用getpwnam获取uid，用getgrnam获取gid
  defined(my $user = getpwnam 'fio') or die 'bad user';
  defined(my $group = getgrnam 'users') or die 'bad group';
  ```
* utime可以修改文件的时间戳
  ```perl
  my $now = time;
  my $age = $new - 24*60*60;
  utime $now, $ago, glob '*';
  # 将最后访问时间改成当前时间，最后修改时间改成一天前
  ```
  PS. 在文件有任何改动时，第三个时间ctime一定会被设定成当前时刻，所以utime无法修改它。ctime主要是给增量备份的程序使用的

  PS.. 如果由于文件不存在导致utime函数调用失败的话，便会调用open函数创建新文件

##### 运行外部程序

* fork函数用于创建进程，它创建调用进程的副本，子进程（继承上下文，打开的文件，真实用户ID，掩码，当前工作目录，信号等等）。可以根据fork函数返回不同的值区分父进程和子进程，父进程得到子进程的pid，而子进程得到0
* wait函数负责强制父进程等待子进程执行完毕，并将子进程的pid返回父进程，如果不存在就返回-1
* waitpid函数也能等待子进程完成，不过它能指定等待哪一个子进程，它还拥有控制阻塞的特殊标志
* system函数可以启动子进程，运行系统的外部程序。当system调用时，它根据当前的父进程（也就是调用它的程序）创建一份拷贝，称为子进程。子进程切换到运行的外部命令上，并继承原来进程中Perl的标准输出及标准错误
  ```perl
  # explain what has `system 'date'` done
  defined(my $pid = fork) or die "Cannot fork: $!";
  unless ($pid) {
      # child process
      exec 'date';
      die "Cannot exec date: $!";
  }
  # father process
  waitpid($pid, 0);
  ```
  ```perl
  system 'ls -l $HOME';   # 防止$HOME解释成变量，使用单引号
  system 'long_running_command_with parameters &';    # 调用后台命令，相当于孙进程
  system 'for i in *; do echo == $i ==; cat $i; done';
  # 这会盗用shell来处理，shell是子进程，要执行的命令是孙进程
  # system也可以用一个以上的参数调用，避免使用shell
  system 'tar', 'cvf', $tarfile, @dirs;
  # 单一参数调用与多参数相似
  system $command_line;   # 相当于
  system '/bin/sh', '-fc', $command_line;
  ```
  system的返回值是根据子进程的结束状态来决定，退出值0代表正常，非0代表有问题
  ```perl
  unless (system 'date') {
      print "We gave you a date\n";
  }
  ```
* exec可以使用system的所有语法，system会创建子进程，子进程在Perl睡眠期间执行任务，而exec让Perl自己去执行任务（pid不变）。

  在exec之后写的代码都无法执行，不过如果启动过程出现错误，后续的捕获语句还是可以继续运行的

* 反引号可以捕获外部程序运行结果，输入带有换行符，可以用chomp处理
  ```perl
  chomp(my $now = `date`);
  # 利用重定向也会被捕获
  my $output = `frobnite 2>&1`;
  # 在列表上下文，会自动取得拆成多行的数据，并都以换行符结尾
  my @files = `ls -l`;
  # 也可使用foreach，一行一行处理
  foreach (`who`) { ... }
  ```
* Perl可以启动一个**异步运行**的子程序，并和它保持通信，指导子进程结束。只要将命令放在open调用的文件名部分，在它前面或后面加上竖线（管道符号），这方法叫管道式打开
  ```perl
  open DATE, 'date|' or die "cannot pipe from date: $!";
  open MAIL, '|mail merlyn' or die "cannot pipe to mail: $!";
  ```
  竖线的位置是有区别的，第一个例子相当于`date | program`将标准输出连接到供程序读取的文件句柄DATE。同样的第二个例子`program | mail merlyn`将该命令的标准输入连接到供程序写入的文件句柄MAIL

  我们也可以写成多个参数的语句。这样第四个及其后的所有参数都将作为要执行的外部命令的参数
  ```perl
  open my $date_fh, '-|', 'date' or die '...';
  open my $mail_fh, '|-', 'mail', 'merlyn', or die '...';
  # 关闭了文件句柄就能使程序读到文件接尾标识符
  close $mail_fh;
  die "mail: non-zero exit of $?" if $?;
  # $? 和shell中的变量是一样的，0表示成功，非0表示失败，且每个结束的进程会覆盖前一个返回值
  ```

##### 信号

* kill能够发送信号到指定pid，信号的编号可以通过`kill -l`查看
  ```perl
  kill 2, 4201 or die "Cannot signal 4201 with SIGINT: $!";
  kill 'INT', 4201 or die "...";
  kill INT => 4201 or die "...";
  # 根据返回值判断进程是否存活
  unless (kill 0, $pid) {
      # 编号0的信号，尝试看看能不能向它发送信号，但不是真的发送，不会打扰到进程
      warn "$pid has gone away!";
  }
  ```
* alarm函数会让内核在规定数秒后将SIGALARM发送到调用进程，如果在alarm有效时间内再次调用，则会覆盖前面的值。返回值为先前定时器剩余的秒数

  当alarm到期时，将消息Alarm clock打印到屏幕上，也可以自己设定捕捉函数$SIG{AL处RM}

* sleep让进程睡眠一定的秒数，没有指定秒数则永远睡眠，直到有其他信号到达。自身也可以设置alarm来中断休眠

* 除了发送信号，也是可以支持接收信号的操作的。例如一个程序会在/tmp创建临时文件，程序正常结束时就会删除这些文件，如果中途收到中断信号，就可能在/tmp目录里留下垃圾，这就需要一个信号处理程序
  ```perl
  sub clean_up {
      # clear temp files
      unlink glob "$temp_directory/*";
      rmdir $temp_directory;
  }

  sub my_int_handler {
      # 信号到达调用函数时，会给子程序一个参数signal_name
      &clean_up();
      die "interrupted, exiting...\n";
  }
  # 绑定该子程序
  $SIG{'INT'} = 'my_int_handler';
  some_code_here();
  clean_up();
  ```
  对特殊哈希%SIG赋值会启动信号处理程序（直到撤销为止）。哈希键是信号名称（不需要写固定的前缀SIG），哈希值是子程序名（不需要与号，正确来说是子程序的引用），只要收到信号，脚本就是暂停手上的事务执行信号处理程序
  ```perl
  # 有两个预设值
  $SIG{'INT'} = 'IGNORE';   # 表示忽略
  $SIG{'INT'} = 'DEFAULT';  # 还原成初始
  ```
  如果信号程序子程序名没有结束程序（即上面的die）而是直接返回，那么程序会从先前中断的地方继续执行

##### 获取用户信息

getlogin函数负责中/etc/utmp中返回当前的登录信息

getpwuid函数以用户的uid作为参数，能够获得该uid的关联项
```perl
$loginname = getlogin || getpwuid($<)[0] || die "..";
```
##### 进程信息

- $$是运行脚本的Perl程序的进程ID号
- getppid函数返回父进程的进程ID号
- getgrp函数会返回指定pid的当前组进程。如果没有参数或者以0作为参数的话，该函数将返回当前进程的进程组ID
- getpriority函数能返回进程、进程组或者用户的当前优先级（nice值），并不是所有系统都支持该函数
  - `getpriority WHICH, WHO;`
    - WHICH有三种值，0表示进程优先级，1表示进程组优先级，2则表示用户的优先级
    - WHO的解释与进程优先级，进程组优先级和用户优先级的进程标识符相关联，值0表示当前进程、进程组或用户
    - PS. nice值为正数的进程将运行在较低的优先级上，反之同理。nice值的范围是从-20到19
    - `$nice_val = getpriority 0, 0;    # 当前进程的nice值`
- setpriority就能设置优先级了，系统若不能执行，就会返回错误
  - `priority WHICH, WHO, NICEVAL;`
  - 只有拥有超级用户的权限，否则就不能设置负的nice值
- getpwent函数能从/etc/passwd文件中检索信息，返回一个9元素数组。即每次调用getpwent的时候从/etc/passwd文件中获取一行内容
  - （登录名，加密后的命令，用户ID，组ID，配额Quota，注释，Geos用户信息，主目录，登录的shell）
  - `($name, $passwd, $uid, $gid, $quota, $comment, $geos, $dir, $shell) = getpwent;`
- getpwnam函数以用户名作为参数，并返回一个9元素数组
- getpwuid函数，和上面的类似，只是getpwuid以用户的ID作为参数

#### 字符串函数

##### index & rindex

index函数用来在字符串中查找子字符串，`$where = index($big, $small)`，寻找失败返回-1。

index函数还可以接受第三个参数，用来指定开始寻找的地方，否则就从最开头寻找

rindex即是最后出现的位置，也有一个可选的第三个参数，用来指定返回值的上限，也就是从那个位置往前找的意思

##### substr

substr函数可以截取较长字符串的一部分

它共有四个参数，一个原始字符串，一个起始位置，以及字符串的长度，要填充的字符串，找到的字符串将被返回。而且起始位置可以为负的，-1表示最后一个字符，以此类推。后面两个参数可以被省略

substr是可以放在等号左边的，这样就能够修改字符串了
```perl
my $string = "Hello world";
substr($string, 0, 5) = "Goodbye";  # Goodbye world
# 相当于
my $previous_value = substr($string, 0, 5, "Goodbye");
# 还可以这样使
substr($string, -20) =~ s/fred/barney/g;
```
##### sprintf & printf

sprintf函数和printf有相同的参数(文件句柄除外)，但是sprintf是不会打印出来的，用于字符串的生成与处理
```perl
my $data = sprintf "%s\n", $piece;
---
sub big_money {
    my $number = sprintf "%.2f", shift @_;
    1 while $number =~ g/^(-?\d+)(\d\d\d)/$1,$2/g;
    # 相当于while ($number =~ s///g) {1;} 循环内部的1没有特殊意义，占位而已
    $number = s/^(-?)/$1\$/r;   # 加上美元符号，$number是没有改动的(/r)
}
```
##### oct & hex

oct和hex都可以进行进制转化，分别是8和16进制

虽然oct默认是8进制，但它可以根据进制的前缀转换各种进制
```perl
oct('0377') # 8进制前缀
oct('377')  # 默认8进制
oct('0xDEADBEEF')   # 16进制
oct('0b1101')   # 2进制
oct('0b$bite')  # 将变量当作2进制转换成10进制
```
##### tr & y

tr和y函数，来源于UNIX的tr命令，主要是依靠一一对象的字符转译来替换字符。依然使用的绑定操作符 `=~`
```perl
    tr/a-z/A-Z/;    # 所有的英文字母分别对应其大写字母，即将字符串转换成大写
    tr/a-z/A-C/;    # 这属于左右不能一一对应，所以d-z都只能转换成C
    tr/a-z/a-c/d;   # d修饰符，从搜索字符日历中删除那些没有出现在替换字串中的字符，即d-z
    tr/a-z/*/c;     # c(complete)修饰符，将没有出现在搜索字符串中的字符转换成替换字符串中的相应（取最后一个），这里表示把a-z以外字符替换成*
    tr/*/*/s;       # s(squeeze挤压)修饰符，负责将所有重复的字符替换成相应的单个字符，这里将多个*挤压成单个星号
```
#### 算数函数

| 函数表示 | 描述                                              |
|----------|---------------------------------------------------|
| atan2    | arctangent                                        |
| cos      | 正弦                                              |
| exp      | 以e为底求EXPR的幂数                               |
| int      | 取整                                              |
| log      | 以e为底求EXPR的对数值                             |
| rand     | 返回一个介于0和EXPR之间的随机数（要求EXPR是正数） |
| sin      | 余弦                                              |
| sqrt     | 开方                                              |


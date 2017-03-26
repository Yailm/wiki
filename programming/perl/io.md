* [输入输出](#输入输出)
    * [钻石操作符](#钻石操作符)
        * [chomp](#chomp)
    * [文件IO](#文件io)
        * [文件测试](#文件测试)
            * [stat](#stat)
        * [open](#open)
        * [binmode](#binmode)
        * [read](#read)
        * [getc](#getc)
        * [flock](#flock)
        * [seek](#seek)
        * [tell](#tell)
    * [基本输出](#基本输出)
        * [print](#print)
            * [HERE文档](#here文档)
        * [printf](#printf)
            * [动态产生字符串](#动态产生字符串)
    * [目录句柄](#目录句柄)
    * [DBM数据库](#dbm数据库)
    * [相关的模块](#相关的模块)
        * [IO::Handle](#io::handle)
        * [IO::File](#io::file)
        * [IO::Tee](#io::tee)
        * [IO::Pipe](#io::pipe)
        * [IO::Null](#io::null)
        * [IO::Interactive](#io::interactive)
        * [IO::Dir](#io::dir)

## 输入输出

### 钻石操作符

`<>`操作符不仅可以从<STDIN>读入数据，而且也能够从命令行指定的文件源读取，需要特殊说明的是，钻石操作符不会检查命令行的参数。而命令行的参数保存在@ARGV中，<\>从中去读取，即使@ARGV被修改过

当文件句柄ARGV位于钻石操作符中，Perl会将命令行参数当成文件名来对待，就会把文件名赋值给$ARGV，并将数组shift
```perl
# [in bash]$ perl % temp 1 2 3 4 5
print "@ARGV\n";        # temp 1 2 3 4 5
$a = <ARGV>;
print "$a\n"            # 输出文件temp的内容
print "$ARGV\n";        # temp
print "@ARGV\n";        # 1 2 3 4 5，注意，数组已经被shift了

--- file: grab.pl ---
if ($#ARGV < 1) {
    die "Usage: $0 pattern filename(s)\n";
}
$pattern = shift;
while ($line = <ARGV>) {
    print "$ARGV: $.: $line" if $line =~ /$pattern/i;
    close ARGV if eof;
}
```
以上就是钻石操作符的由来了，<\>会隐式地调用ARGV数组中的每一个元素当作文件的句柄

eof函数用于检查是否到了文件末尾，如果对文件句柄FILEHANDLE的下一次读操作是发生在文件末尾或者文件没有打开，函数都会返回1。`eof FILEHANDLE`，如果eof没有带参数，将返回上一次文件读操作的eof状态。然而还能如下使用
```perl
open DB, 'emp.names';
while (<DB>) {
    print if (/Norma/..eof);    # 打印从Norma到文件末尾（eof）的所有内容
}
```
$^I这个变量的默认值为undef，当这个变量改变时(为字符串或空串)，Perl除了像以前把文件打开之外，还会把文件名改成`$filename.$^I`(并接)，而且还另外开开一个新文件取名为$filename，并将默认输出设定到这个新文件。所以`$^I`为空串，就是在修改原文件

#### chomp

<STDIN>可以获取用户的输入，且数据一般末尾都带有换行符
所以这样，chomp可以去掉换行符
```perl
chomp($text = <STDIN>);
chomp(@line = <STDIN>); * 读入所有行，不读换行符，直到<C-d>
```
chop函数可以删除标量型变量的最后一个字符，或是数组中**每个元素**的最后一个字符，同时返回最后删除的那个字符

chomp函数类似chop，但只有最后的字符为$/存储时（默认是换行符）才进行删除，用于数组时返回删掉的字符数目
```perl
chomp(my $word = <STDIN>);      # [input]: ba
$/ = 'a';
chomp($word);
print $word;                    # [output]: b
```
### 文件IO

#### 文件测试

- -r  File is readable by effective uid/gid.  
  effective就是文件比如加上了suid，运行之后的权限了
- -w  File is writable by effective uid/gid.
- -x  File is executable by effective uid/gid.
- -o  File is owned by effective uid. 只管用户的ID
- -R  File is readable by real uid/gid.   real就是用户本来的uid和gid了
- -W  File is writable by real uid/gid.
- -X  File is executable by real uid/gid.
- -O  File is owned by real uid. 只管用户的ID

- -e  File exists.
      die "Oops| A file called '$filename' already exists.\n"
          if -e $filename;
- -z  File has zero size (is empty). 对目录来说永远为假
- -s  File has nonzero size (returns size in bytes).    
  `my $size_in_kb = (-s) / 1000;`没有参数则默认使用$_，括号不能省
- -f  File is a plain file.
- -d  File is a directory.
- -l  File is a symbolic link (false if symlinks aren't)  
  注意符号键链接文件-f -l的返回结果都是真
- -p  File is a named pipe (FIFO), or Filehandle is a pipe.
- -S  File is a socket.
- -b  File is a block special file. 块设备文件(比如是可挂载的磁盘)
- -c  File is a character special file. 字符设备文件(比如某个I/O设备)
- -t  Filehandle is opened to a tty.
- -u  File has setuid bit set.
- -g  File has setgid bit set.
- -k  File has sticky bit set.
- -T  File is an ASCII or UTF-8 text file (heuristic guess).  
  对于-T -B并不一定是相对的，如果文件不存在，两者都为假；如果空文件，两者都为真
- -B  File is a "binary" file (opposite of -T).
- -M  Script start time minus file modification time, in days. 最后一次修改后至今的天数
- -A  Same for access time. 最后一次访问至今的天数
- -C  Same for inode change time (Unix, may differ for other platforms)  
  最后一次文件节点编号被变更后至今的天数

测试同一个文件的多项属性，若使用`if ( -r $file and -w $file) {}`，这是非常耗费资源的操作，因为取了两次$file的信息

这里perl有一种叫虚拟文件句柄`_`，可以避免重复劳动，所以上面的表达式可以写成`if (-r $file and -w _) {}`。而且也不要求要在同一条语句中使用，即`_`表示的上次查询的文件句柄
```perl
if (-r $file) { }
if (-w _) { }
---
use 5.010;  # 5.010后，可以把测试操作符排成一行，称为栈式排列
if (-w -r $file) { }
```
##### stat

stat可以返回文件所有的相关信息
```perl
my ($dev, $ino, $mode, $nlink, $uid, $gid, $rdev, $size, $atime, $mtime, $ctime, $blksize, $blocks)
    = stat $filename;
```
* dev和ino 即文件所在设备的编号与文件的inode编号，这两个编号决定了文件的唯一性，即使它具有多个不同的文件名(使用硬链接创建)
* mode 文件权限位集合，还包含其他信息位，unix的9个权限位对应$mode最低有效的权限位
* nlink 文件或目录的(硬)链接数，也就是这个条目有多少个真实名称，对目录来说总会是2(. ..)或是更大的数字，对文件来说则通常是1，在`ls -l`的结果中，权限位之后的数字就是文件链接数
* uid或gid 以数值形式表示uid和gid
* size 以字节为单位的文件大小，和-s测试操作符的返回值相同
* atime, mtime, ctime 以系统的时间格式来表示，一个32位的整数，表示从纪元(epoch)算起的秒数，

PS. 如果对符号链接调用stat会返回指向对象的信息，而非符号链接本身的信息（除非该链接指向的对象无法访问）。这时就可使用`lstat`来查看

#### open

open 句柄名 "(<|>|>>)?filename"

open 句柄名, '操作符', $file_name;

句柄名可以使用裸字，即没有特殊符号前缀，最好再用上大写字母。有6个保留句柄：STDIN, STDIN, STDOUT, STDERR, DATA, ARGV, ARGVOUT。（PS. 若要重新定义这些句柄名，需要先关闭close XXX;再定义

打开一个句柄时，若有同名的句柄已经打开，Perl会自动帮你关闭原来的那个

其他的写法
```perl
$HANDLE = 'filename';
open HANDLE;    # 省略了$filename

# 还能够指定数据的编码
open CONFIG, '<:encoding(UTF-8)', 'dino';
# 区别于下面这种写法
open CONFIG, '<:utf8', $file_name;
# 可能会有问题，这种写法直接把数据当成utf8，没有确认是否正确

open BEDROCK, '>:crlf', $file_name;
# 这个在windows上面是默认支持的，读取的时候可以转换成UNIX风格

open my $fh, '>', $filename or die "cannot create $filename: $!";    # 使用自定义的变量(一般这样用)，use autodie能自动配置die
close $fh;
```
暂时保存文件句柄
```perl
open SAVED, ">&STDOUT";     # 保存STDOUT句柄到SAVED
open STDOUT, ">temp";       # 新打开个STDOUT
open STDOUT, ">&SAVED";     # 恢复STDOUT句柄
```
```bash
# 列出perl的编码清单
perl -MEncode -le "print for Encode->encodings(':all')"
```
#### binmode

`binmode 句柄名, ':encoding(UTF-8)', $filename;`

二进制文件读写文件句柄

#### read

read函数可以从指定的文件句柄中将指定的字节读入变量中
```perl
number_of_bytes = read(FILEHANDLE, buffer, how_many_bytes);
# 常用格式
while (read(INPUT, $buffer, 1024)) {
    ...
}
```

sysread避开了标准的I/O缓冲，能直接从文件句柄中读取文件避免进入缓冲区，混用read和sysread可能会造成问题。配套的功能还有syswrite

#### getc

getc函数能从键盘或者文件中获得单个字符，如果碰到EOF，getc会返回空字符串
```perl
getc FILEHANDLE;

$ans = getc;    # 从默认句柄STDOUT中获取
$restofit = <>; # 获得剩余的输入数据
```
#### flock
```perl
open DB, ">>datafile";
flock DB, 2;
flock DB, 8;
```
| 操作名  | 操作数 | 具体工作                                       |
|---------|--------|------------------------------------------------|
| lock_sh | 1      | 创建共享锁（文件需能提供读权限）               |
| lock_ex | 2      | 创建排他锁（文件需能提供写权限，只有自己能用） |
| lock_nb | 4      | 创建非阻塞锁（创建失败时能返回程序）           |
| lock_un | 8      | 解除当前锁                                     |

PS. 在非UNIX系统中或是网络系统中访问，flock可能是无效的

#### seek

seek函数负责以随机的方式访问文件，允许用户在文件中直接跳转到指定内容所在的位置，如果成功返回1，否则返回0

seek FILENAME, BYTEOFFSET, FILEPOSITION;
  - 其中FILEPOSITION有3个数值
    - 0--文件开头位置
    - 1--文件中的当前位置
    - 2--文件中的末尾位置
  - BYTEOFFSET表示偏移量，是从FILEPOSITION到想要设定位置的字节距离（负数表示往回走）
  ```perl
  open my $fh, '<test.plx';
  read $fh, $buffer, 4;     # 读取4字节到buffer，当前的位置为seek($fh, 0, 1或seek($fh, 4, 0)
  seek($fh, 1, 1)           # 前进一步
  print <$fh>;
  seek $fh, 0, 0;           # 通过seek设置到文件的开头，重新读取
  print <$fh>;              # 配合od -c file来看，比较容易
  ```

#### tell

tell函数可以返回文件中当前字节的位置，能和seek函数配合使用

`tell FILEHANDLE`，如果省略了FH参数的话，则返回文件上一次读取的位置

### 基本输出

#### print

一些奇特的输出
```perl
print @arrays;      # 输出元素连成的字符串
print "@arrays";    # 输出元素以空格隔开的字符串（perl自动插入了空格）
    # 可以把上面理解成print处理的待打印的字符串列表，因此它的参数会在列表上下文执行
print <>;           # 相当于unix的cat命令，会一次读取完文件
print sort <>;      # 相当于unix的sort命令
print (2+3)*4;      # 输出是5。。。。print本来是带括号的
```
输出到句柄
```perl
print LOG "....";
printf STDERR "%d ...", $aidf;
printf(STDERR "%d ...", $aidf);
printf STDERR ("%d ...", $aidf);

select REDROCK; # 替换默认STDOUT为REDROCK
print ".....";
$| = 1;     # 这个特殊变量设置为1，就会使当前的句柄在每次输出操作后立即刷新缓冲区
# 如果没有给字符串加上引号，则必须指定文件句柄
print STDOUT Hello, world, "\n";
# __LINE和__FILE__记录了当前的行数和文件名，__PACKAGE__则记录了当前的包名
# 它们只能被写在引号的外边
print "I am in: ", __FILE__, __LINE__, "\n";
```

特殊变量__END__负责表示文件的逻辑末尾位置，任何位于__END__后面的内容都将被忽略，在UNIX中，表示文件末尾的控制序列`<C-d>`，而在MS-DOS中则是`<C-z>`，两者都等效于__END__的意思

__DATA__表示当前脚本文件中的一个文件句柄，用来存储数据，写在__DATA__后的内容会被当成数据
```perl
print <DATA>;
__DATA__
__DATA__可以写成__END__
print语句不要动
也能达到相同的结果
```
print打印其他进制的数字都会自动转换成10进制

##### HERE文档

HERE文档特性来自于UNIX shell，文档的内容必须在用户自定义终止符的字段之间
```perl
print <<EOF;
haha\n
EOF

print <<'EOF';  # here文档如同被单引号包围
haha\n
EOF

print << x4;    # 字符串输出4遍，注意，这里以一个空行作为结束
haha

print <<`EOF`;  # 字符串会被执行(与UNIX shell不同，Perl不允许在here文档中执行命令替换？)
date
EOF
```
#### printf
```perl
printf "Hello, %s, your password expires in %d days\n", $name, $days;
```
| 符号 | 解释                                                   |
|------|--------------------------------------------------------|
| %g   | 输出恰当的数字形式，自动选择浮点，整数甚至指数形式     |
| %d   | 十进制整数，舍去小数点后的数字，也可以设定宽度（见%s） |
| %s   | 字符串内插，可设定宽度，正数是右边对齐，反之           |
| %f   | 浮点数，会按需四舍五入，指定小数点后的输出位数         |
| %x   | 16进制                                                 |
| %o   | 代表8进制                                              |
| %%   | 字面上的%                                              |

##### 动态产生字符串
```perl
my @items = qw( wilma dino pebbles );
my $format = "The items are:\n" . ("%10s\n" x @items);
print $format, @items;
# 最酷的还是整成一句话
printf "The items are:\n". ("%10s\n" x @items), @items;
```
### 目录句柄

目录句柄和文件句柄差不多，打开opendir(代替open)，读取内容是readdir(代替readline)，不过读取的目录里的文件名，关闭closedir，同样还有seekdir，rewinddir，telldir
```perl
my $dir_to_process = '/etc';
opendir my $dh, $dir_to_process or die "Can't open $dir_to_process: $!";
foreach $file (readdir $dh) {
    print "one file in $dir_to_process is $file\n";
}
closedir $dh;
# 值得我们注意的是.和..也在里面，并且没有顺序，只能把它们过滤掉了
while ($name = readdir $dh) {
    next if $name eq '.' or $name eq '..';
    ...
}
```
### DBM数据库
```perl
dbmopen(%HASH, $filename, 0666)
```
我们可以将哈希绑定到一个数据库生，检索哈希的数据就相当于文件里的数据，后面是文件的权限，0代表开启检测（关联失败时，返回false），$filename不用加上.dir或.pag
  - dir文件含有索引目录
  - pag文件含有所有数据
```perl
dbmclose(%HASH)
```
还可以使用AnyDBM_File模块让Perl选择合适的DBM

### 相关的模块

#### IO::Handle

对于文件句柄的引用，实际上Perl是用了IO::Handle模块，也就是filehandle scalar是一个IO::Handle的对象。在v5.14后，我们不用显式`use IO::Handle`
```perl
open my $fh, '>', $filename or die '...';
$fh->print('....');
$fh->close;
```
#### IO::File

IO::File是IO::Handle的子类，work with files，自带
```perl
use IO::File;
my $fh = IO::File->new('>text.log') or die ...;
my $fh = IO::File->new('text.log', 'w');
my $fh = IO::File->new('castaway.log', O_WRONLY|O_APPEND);
my $fh = IO::File->new_tmpfile; # 这里打开一个匿名文件，new_tempfile是特定的关键词这里
# 当这些变量过了范围后，是会自动关闭的
# v5.6之后，可以使用open，打开一个匿名临时文件，不过我们找不到这个文件
open my $fh, '+>', undef
    or die "Could not open temp file: $!";
```
#### IO::Tee

IO::Tee模块能够帮助我们将内容同时输出到多个filehandle
```perl
use IO::Tee;
my $tee_fh = IO::Tee->new($log_fh, $scalar_fh);
print $tee_fh "something usefull";
# 除此之外，当IO::Tee的第一个参数是input句柄，后面跟着output句柄时，这时我们利用input读入并立即写入output中
my $tee_fh = IO::Tee->new($read_fh, $log_fh, $scalar_fh);
my $message = <$tee_fh>;    # reads from $read_fh, prints to $log_fh and $scalar_fh
```
#### IO::Pipe

IO::Pipe类似于提过的异步通信，我们可以给它一个命令，它会帮我们执行fork和exec命令（相当于system）并给我们一个filehandle让我们从中读取数据
```perl
use IO::Pipe;
my $pipe = IO::Pipe->new;
$pipe->reader("$^X -v");    # $^X 就是当前perl的可执行文件/usr/bin/perl之类的
while (<$pipe>) {
    print "Read: $_";
}
# 同样，也可以写数据到command中
$pipe->writer($command);
```
#### IO::Null

有时候我们要被迫输出一些东西，我们可以用这个模块应对
```perl
use IO::Null;
my $null_fh = IO::Null->new;
some_printing_thing($null_fh, @args);
```
#### IO::Interactive

IO::Interactive可以分辨程序当前是否处于交互状态，（即有时我们会手动运行程序，希望看到程序输出，而在计划任务cron中，程序运行的时候并不想要输出）
```perl
use IO::Interactive;
print { interactive } 'Bamboo car frame';
# 上面的interactive实际上是一个子程序，省略了()，调用模块中的is_interactive()
# 如果true，返回*STDOUT，false则返回一个filehandle类似上面的IO::Null
# 事实上，print输出句是推荐写成print {$filehandle) "..."这样perl就会知道$filehandle是一个句柄引用
print {*STDOUT} 'haha';
print {STDOUT} 'haha';
```
#### IO::Dir

这个模块并没有添加新的东西，只是包装成了一个面向对象的模块
```perl
use IO::Dir;
my $dir_fh = IO::Dir->new( '.' ) or die "...";
while (defined(my $file = $dir_fh->read)) {
    print "...";
}
$dir_fh->rewind;    # 倒回，重来
```

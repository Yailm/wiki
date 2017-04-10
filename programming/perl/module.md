
模块编写
------------------------------------

### 查找模块

#### 内部设置

模块查找的时候从@INC提供的值，可以通过`perl -le "print for @INC"`打印它

use语句执行在程序的编译阶段，如果想要通过修改@INC来定位模块，也必须让一些语句在编译阶段执行
```perl
BEGIN { unshift @INC, '/path/..'; }
use module;
```
然而不用这么麻烦，可以使用下面的这种语句
```perl
use lib qw( /path/ );
# 这是在编译阶段执行，path中不可以使用变量的
# 但是允许使用标量
use constant LIB_DIR => '/path/';
use lib LIB_DIR;
# 或利用perl的自带模块FindBin，$Bin返回当前目录
use FindBin qw($Bin);
# $Bin就是当前目录的路径，可以用来配合载入模块
use lib $Bin;
use lib $Bin/lib;
```
#### 外部设置

Perl会将环境变量PERL5LIB插入到@INC的开头
```bash
export PERL5LIB=/home/fio/perl5:/usr/local/lib/perl5 # 然后再执行我们的程序
# 或者在命令行参数中指定
perl -I/home/fio/perl5 program
```
### 使用模块

use 模块名称

也可以只导入模块的部分函数
- use 模块名称 qw / 函数名 /;
- use 模块名称 ('函数名', '函数名');  # 同上
- use 模块名称 qw / /; 只导入模块，不导入任何函数，要使用里面的函数就必须要写出全名。而且只要导入了模块，全名是随时都可以使用的

require也和use一样可以载入模块，不过require发生在程序运行时
```perl
use List::Util qw(...);
# 相当于
BEGIN { # what use is really doing
    require List::Util;
    List::Util->import(...);
}
```
require语句中的::都会被转换成文件分隔符
```perl
require "Island/Plotting/Maps.pm";  # pl后缀也行的，不过不能简写
```
h2ph是Perl发布包自带的脚本，能够将c的头文件转换成对应的Perl头文件(.ph)，比如通过该方法转换出syscall.ph后，`require "syscall.ph"就可以使用syscall函数了`

### 制作模块

- ExtUtils::MakeMaker模块使用的是Makefile.PL
- Module::Build较新，使用的是Build.PL

#### Module::Starter

命令`module-starter --module=Animal --author=any --email=example@gmail.com --verbose`

默认的，module-starter用Makefile.PL创建一个发行版，我们也可以使用参数制定创建的系统，`module-starter --build="Module::Build" ...`，其实已经内置了，使用--mb就行了

如果不想用那么长的命令行，可以将配置写入一个config文件中，而上层文件夹的路径由系统变量MODULE_STARTER_DIR指定
```bash
    $ cat $MODULE_STARTER_DIR/config
    author: Anyone
    email: anyone@gmail.com
    builder: Module::Build
    verbose: 1
    $ module-starter --module=Animal    # 之后我们就可以只用简单的参数来定义
```
Module::Starter::Plugin帮助我们创建插件

在我们的发行版添加额外的模块可以使用

    $ module-starter --module=Module1,Module2,Module2

也可以在已经创建好的发行版中添加模块，使用Module::Starter::AddModule

同时应该在我们的配置文件中声明要用的插件

    author: Anyone
    email: Anyone@gmail.com
    builder: Module::Build
    verbose: 1
    plugins: Module::Starter::AddModule

要添加新的模块到distribution中

    $ module-starter --module=Sheep(要添加的模块) --dist=.(被添加到的distribution)

### 编译模块

    perl Build.PL
    ./Build
    ./Build test
    ./Build disttest
    ./Build dist    # 最终获得一个tar.gz包

### 模块测试

Test::More可以进行模块的测试
```perl
use Test::More tests => 1;      # 表示我们有1个测试
# 如果我们不确定测试的数量，可以先不些tests，然后在结束时打上done_testing();
ok( try_motor(), 'The boat motor works');
ok( try_gas() eq 'Full', 'The gas tank is full');
is( check_gas(), 'Full', 'The gas tank is full');
isnt( check_gas(), 'Broken', 'The hull is intact'); # 不等于参数二就通过
like('Mary Ann', qr/Mary[ -]Anne?/, 'Mary Ann is a passenger');
unlike('Ginger', qr/Mary[ -]Anne?/, 'Ginger is a passenger');
is_deeply(\@this_array, \@that_array, 'The arrays are the same');
```
PS. 浮点数不总是那么精准，我们可以用Test::Number::Delta模块来解决

一个例子，测试文件内部 t/00-load.t
```perl
#!perl -T
use Test::More tests => 1;
BEGIN {
    use_ok( 'Animal') || print "Bail out!\n";   # use_ok尝试运行一个模块，失败则返回false
}
diag( "Testing Animal $Animal::VERSION, Perl $], $^X" );
```
#### 跳过测试

有时我们需要根据环境来跳过某些环境，就可以使用一个裸句块来创建单元代码并标记为SKIP来略过
```perl
SKIP: {
    skip 'Mac::Speech is not available' ,1 unless eval { require Mac::Speech };
    ok($tv_horse->say_it_aloud('I am Mr. Ed');
    # 当Test::More略过这些测试时，它输出特殊的ok信息来确保test数量正确
}
```
#### 面向对象测试

Test::More中还有用于面向对象测试的函数isa_ok, can_ok
```perl
isa_ok($object, $class);    # 是否属于这个类或是其子类也行
can_ok($module, @method);
can_ok($object, $method);   # 含有该方法
```
#### 测试分组
```perl
use Test::More;
subtest 'Major feature works' => sub {
    ok( defined &some_subroutine, 'Target sub is defined' );
    ok( -e $file, 'The necessary file is there' );
    is( some_subroutine(), $expected, 'Does the right thing' );
};
done_testing();
__OUTPUT__
测试成功的输出结果像这样
    ok 1 - Target sub is defined
    ok 2 - The necessary file is there
    ok 3 - Does the right thing
    1..3
        ok 1 - Major feature works
        1..1
```
#### 其他模块

##### Test::Output

Test::Output模块中的方法提供输出到STDOUT和STDERR的测试
```perl
use Test::More;
use Test::Output;
sub print_hello { print STDOUT "welcome aboard\n" }
sub print_error { print STDERR "there's a hole in the ship\n" }
stdout_is( \&print_hello, "welcome aboard\n" );
stderr_like( \&print_error, qr/ship/ );
done_testing();
# 程序足够短，也可以这样
stdout_is(sub { print 'welcome aboard' }, "welcome aboard");
# 写成inline block of code，像grep和map一样
stdout_is { print "welcome aboard" } "welcome aboard";
```
##### Test::Warn

Test::Warn模块用来测试警告信息的输出
```perl
use Test::More;
use Test::Warn;
sub add_letters { "skipper" + "Gilligan" }
warning_like { add_letters() } qr/nonnumeric/;
done_testing();
```
Test::NoWarnings和use warnings搭配，测试不存在的警告

##### Test::MockObject

Test::MockObject模块可以模仿一个类
```perl
my $Minnow = Test::MockObject->new();
$Minnow->set_true('engines_on');    # 设置该函数的返回值为true
$Minnow->set_false('has_maps');
```
比如测试一个类（方法、函数）时，需要用到某个实例但却不想引入大量的代码，可以用这个模仿该类的行为进行测试
```perl
# 以下是模拟数据库
use Test::More;
use Test::MockObject;
my $db = Test::MockObject->new();
$db->mock(
    list_names => sub { qw( Gilligan Skipper Professor ) }
    # 模拟返回3个表名
);
my @names = $db->list_names;
is(scalar @names, 3, 'Got the right number of result');
is($names[0], 'Gilligan', 'The first result is Gilligan');
done_testing();
```
##### Test::Buider

Test::Builder可以让我们创建自己的测试模块
```perl
package Test::Minnow::RequireItems; # 我们自己的模块
use strict;
use warnings;
use Exporter qw(import);
use vars qw(@EXPORT $VERSION);

use Test::Buider;
my $test = Test::Buider->new();
our $VERSION = '0.10';
our @EXPORT = qw(check_require_items_ok);

sub check_require_items_ok {
    my $who = shift;
    my $item = shift;
    my @required = qw(preserver sunscreen water_bottle jacket);
    my @missing = ();
    for my $item (@required) {
        unless (grep $item wq $_, @$items) {
            push @missing, $item;
        }
    }
    if (@missing) {
        $test->diag("$who needs @missing.\n");
        $test->ok(0);
    } else {
        $test->ok(1);
    }
}
1;
```
#### Build

当我们运行`./Build test`的时候，有些测试模块是被skipped的，写着Author tests not require for installation，写得很清楚，作者自己测试用的
```bash
% ./Build
% perl -Iblib/lib -T t/00-load.t # 测试刚刚的test文件，-T表示强制打开错误检查
blib模块可以代替-Iblib/lib，寻找周围的文件夹包括父目录的blib文件并添加到@INC中(5层)
% perl -Mblib -T t/00-load.t  # @INC最前面被添加了/tmp/Animal/blib/lib
% ./Build test 执行t和xt下所有的测试

# 获取测试的指标，需要Devel::Cover模块来收集指标
% ./Build testcover # 如果我们用的是Module::Build时
% HARNESS_PERL_SWITCHES=-MDevel::Cover make test    # 使用ExtUtils::MakeMaker
% cover     读取当前文件夹上次运行的指标
```
### 编写文档

Pod Command Paragraphs

    =head n 用来设置标题，n是数字表示等级
    =cut    可以返回到代码
    =over n 可以创建一个列表
    =back   从列表中返回

Pod Formatting Codes

    B<bold text>
    C<code text>
    E<named entity>
    I<italic text>
    L<linked text>
    如果内容里面有存在<, >符号，需要增加外面尖括号的层数


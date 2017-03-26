* [基础数据](#基础数据)
    * [列表](#列表)
        * [列表操作](#列表操作)
        * [批量操作](#批量操作)
            * [grep](#grep)
            * [map](#map)
                * [施瓦茨变换](#施瓦茨变换)
            * [List::Util](#list::util)
            * [List::MoreUtils](#list::moreutils)
        * [列表切片](#列表切片)
    * [哈希表](#哈希表)
        * [哈希切片](#哈希切片)
    * [子程序](#子程序)
        * [预先声明](#预先声明)
        * [原型](#原型)
    * [上下文](#上下文)
    * [引用](#引用)
        * [符号引用](#符号引用)
        * [化简](#化简)
        * [引用存在](#引用存在)
        * [弱引用](#弱引用)
        * [匿名数组](#匿名数组)
        * [匿名哈希](#匿名哈希)
        * [子程序的引用](#子程序的引用)
        * [匿名子程序](#匿名子程序)
            * [闭包](#闭包)
        * [open打开引用](#open打开引用)
        * [数据结构的处理](#数据结构的处理)


## 基础数据

- 定义常量 `use constant PI => 3.1415926`
- 8进制以0开头的数字 `$mode = 0755`
- 2进制以0b开头 `$bit = 0b10101010101110`
- 16进制以0x开头 `$color = 0x33cc99`
- `qq/../` 相当于 `"..."`
- `q/../` 相当于 `'...'`
- `qx/../` 相当于 \`..\`（反引号）
- `qw/a b c/` 就是 `('a', 'b', 'c')` 数组

- 和""不同，''内的字符不会有任何改变，.用来连接字符串
- 布尔值0和''还有空数组为假，其它都为真
- defined函数可以判断变量undef
- 数字与字符串的比较符号


  | `==` | `!=` | `<` | `\>` | `<=` | `>=` |
  |------|------|-----|------|------|------|
  | eq   | ne   | lt  | gt   | le   | ge   |

### 列表

列表用于变量快速赋值
```perl
($a, $b) = ("a", "b");
($a, $b) = ($b, $a);
my ($a, $b) = @_;
```
$#array为列表中最大的索引，可以通过对其直接赋值，缩减数组 `$#grades = 3`

#### 列表操作

- pop负责取出数组中最后一个元素并将其作为返回值返回
- push负责添加一个元素或是一串元素到数组的尾端
- shift和unshift类似上述的命令，处理的是数组的头端      (un)?shift ARRAY, LIST;
- splice操作符处理中间，最后两个参数为可选项，将删除的元素整合成一个列表返回<br>
    `splice(ARRAY, OFFSET, LENGTH, LIST)`
    * ARRAY为目标数组
    * OFFSET为操作的开始位置
    * LENGTH为操作长度
    * LIST为将要替换成的数组
- reverse逆序数组
    ```perl
    @fred = 6..10;
    @fred = reverse @fred;
    $scalar = reverse 'helloworld'  # dlrowolleh，也能够逆序字符串
    ```
- sort排序数组
- each将返回数组中下一个元素所对应的索引和值
    ```perl
    while ( my ($index, $value) = each @rocks ) {
        say "$index, $value";
    }
    ```

#### 批量操作

##### grep

grep筛选列表，如在1000的数字列表中筛选奇数`my @odd_numbers = grep { $_ % 2 } 1..1000`

grep的第一个参数是代码块{ }(不用括号)，使用$_作为列表每个元素的占位符（依然是别名，所以一定程度上可以完成类似map的操作）并返回真或假，代码块后的参数就是要被筛选的元素列表，不过也可以这样`my @matching_lines = grep /\bfred\b/i, <$fh>`。像这样的，代码块简单到只需一个表达式，语句后加个逗号就行了

grep操作符在标量上下文返回符合过滤条件的元素个数

##### map

map操作符可以对数组元素进行批量操作，返回一个新列表，用法和grep类似。但map代码块中最后一个表达式的方式和grep不同，它返回的不是逻辑真假值，而是该表达式的实际计算结果，最终返回一系列这样的结果组成的列表

TIP.
  - map并没有限定一个input只能对应一个output
  - grep和map中的$_都只是别名，可以在表达式中修改原数组，再加上返回的新数组，同时构建两个需要的数组

```perl
my $result = map { split // } @input_number;

my @result = map { $_, 3 * $_ } @input_number;
my %hash = @result;     # convert to hash
# 干脆直接写成
my %hash = map { $_, 3 * $_ } @input_number;

# 常用技巧，将列表转化成哈希来判断原列表是否存在某个值
my %hash = map { $_, 1 } @castaways;
my $person = 'Gilligan';
if ($hash{$person}) {
    print "$person is a castaways.\n"
}
```

###### 施瓦茨变换
```perl
my @output_date =           # 常见的
    map { EXTRACTION },     # { $_->[0] },
    sort { COMPARISON },    # { SORT COMPARISON USING $a->[1] AND $b->[1] }
    map [ CONSTRUCTION ],   # [ $_, EXPENSIVE FUNCTION OF $_ ]
    @input;
```
##### List::Util

List::Util包含在标准库中，它能提供各式常见的列表处理工具
```perl
use List::Util qw(first sum sum0 max maxstr min minstr shuffle product);
my $first_match = first { /\bPebbles\b/i } @characters;
my $total = sum(1..1000);
my $total0 = sum0 1..1000;       # 类似sum，当列表为空时，返回0而不是undef
my $max = max @array;
my $maxstr = maxstr @string;    # 利用字符串的比较
my $shuffle = shuffle(1..1000); # 乱序数组
my $foo = product 1..5;         # 乘积

use List::Util qw(any none all notall);
if (none { $_ > 100 } @numbers) {
    print "No elements over 100\n";
} elsif (any { $_ > 50 } @numbers) {
    print "Some elements over 50\n";
} elsif (all { $_ < 10 } @numbers) {
    print "All elements are less than 10\n";
}
```
里面的`reduce { BLOCK } @list`，其中BLOCK有两个已经定义的变量`$a, $b`
  - $a为上次{ BLOCK }中表达式的结果
  - $b为新添加进来的列表元素
  - 初始化时它们为列表前两个元素

reduce会返回最后一个{BLOCK}表达式的值；如果list为空，则返回undef；而当list的元素只有一个时，返回那个元素，BLOCK的代码也不会被执行

除此之外，该模块里还有适用于键值对的函数，同时读入两个元素a，b（也就是键值）进行操作。这样的操作有
  - pairgrep
  - pairfirst
  - pairmap
  - pairs(`pairmap { [ $a, $b ] } @kvlist`)
  - pairkeys(`pairmap { $a } @kvlist`)
  - pairvalues(`pairmap { $b } @kvlist`)
```perl
# 用reduce实现各种操作
$foo = reduce { defined($a)             ? $a :
                $code->(local $_ = $b)  ? $b : undef
        } undef, @list;     # first，注意这行前面这个神奇的undef，如果没了，这个语句也就废了
$foo = reduce { $a > $b ? $a : $b } 1..10;      # max同理min
$foo = reduce { $a gt $b ? $a : $b } 'a'..'z'   # maxstr同理minstr
$foo = reduce { $a + $b } 1..10;                # sum
$foo = reduce { $a . $b } @bar;                 # concat
# 再高级点的
$foo = reduce { $a || $code->(local $_ = $b) } 0, @bar;     # any
$foo = reduce { $a && $code->(local $_ = $b) } 1, @bar;     # all
$foo = reduce { $a && !$code->(local $_ = $b) } 1, @bar;     # none
$foo = reduce { $a || !$code->(local $_ = $b) } 0, @bar;     # notall
# 当然上面都是不完全实现，因为没有full short-circuit
```

##### List::MoreUtils

List::MoreUtils也提供不少的工具
```perl
use List::MoreUtils qw(none any all notall);
if (none { $_ > 100 } @numbers) {
    print "No elements over 100\n";
} elsif (any { $_ > 50 } @numbers) {
    print "Some elements over 50\n";
} elsif (all { $_ < 10 } @numbers) {
    print "All elements are less than 10\n";
}

# 按元素组处理列表的话，可以用natatime(n at a time, 同时处理N组）来取出想应位置上的元素
use List::MoreUtils qw(natatime):
my @array = 1..100;
my $iterator = natatime 3, @array;
while (my @triad = $iterator->()) {
    print "Got @triad\n";
}
--- output ---
Got 1 2 3
Got 4 5 6
...

# 构建一个大型列表，交错填充原始列表中各个位置上的元素
use List::MoreUtils qw(mesh);
my @abc = 'a'..'z';
my @number = 1..20;
my @large_array = mesh @abc, @number;
--- @large_array ---
a 1 b 2 c 3 d 4...
```
更多的函数可以继续看看`perldoc List::MoreUtils`

#### 列表切片

列表切片可以把列表当成数组，用索引取得里面的值

- 例如`my $mtime = (stat $some_file)[9]`，其中stat周围的括号是必须的，因为需要它们产生列表上下文。列表切片就是列表后面有一个方括号的表达式，而索引号在里面用逗号隔开就行

- 再如`my ($first, $last) = @name[0, -1]`，负数用来表示倒数第几个

- 对于`$names[..]`和`@names[..]`，变量引用前的符号决定了下标表达式的上下文，所以两个分别是在标量上下文和列表上下文中求得索引[列表]。即`@names[2, 5]`和`($names[2], $names[5])`是一样的

- 切片可以内插入字符串中，`print "Bedrock @names[ 9, 0, 1, 2]\n"`，各个元素也都是用空格隔开，准确的说，列表的这些元素会被Perl内置的$"变量填充，默认值是空格

- 列表切片还可以局部修改列表中的值`@items[2, 3] = ($new_address, $new_phone)`

### 哈希表

%hash

- `$hash{$key}`能直接访问哈希表元素，key都是简单的字符串，如果键名只是字母、数字和下划线组成的，且不以数字开头，那就可以省略引号。即`$hash('keystring')`写成`$hash{key}`，`$hash{foo.bar} = $hash{foobar}`
- `keys`和`value`函数分别返回哈希的键列表，值列表。若哈希为空，则两个函数都返回空列表。虽然返回的两个列表顺序不确定的，但是它们是相对应
- `each`迭代函数，可以以包含两个元素的列表返回键值对
  ```perl
  while (($key, $value) = each %hash) {
      print "$key => $value\n";
  }     # while的判断语句是布尔上下文，也是一种特殊的标量上下文
        # 赋值后的列表($key, $value)标量上下文即为列表元素的个数，为2，非0就是真值
        # 当哈希中所有的键值都被访问了，没有新的键值对了，此时each就会返回空列表，标量上下文变成0，假
  ```
- 哈希中都有一个迭代器，会记录着上次访问过的位置，而在迭代哈希的过程中增减键值对就可能导致一些问题
  * 重置迭代器的方法:
      * 使用`keys`或者`values`函数
      * 用新的列表重置整个哈希
      * `each`调用遍历整个哈希
- `my $count = keys %hash;` 这个标量上下文能快速返回键值对的个数，不必遍利
- 若哈希中至少一个键值对，就返回真
- `exist`函数能够检查哈希中是否存在某个键，也可以检查数组的下标
  ```perl
  exists $ARRAY[index];
  if (exists $books{dino}) {
      print "Hey, there's a library card for dino\n";
  }
  ```
- `delete`函数，删除指定的键及其对应的值，如果不存在，就直接结束，同样也可以用于数组
- `%ENV`这个哈希中存储着父进程传给Perl的环境变量(shell或是web服务器进程)

```perl
$hash{foo.bar} == $hash{foobar};
# 哈希表的创建
%hash = ('foo', 34, 'bar', 12); # 或者是
%hash = qw(foo 34 bar 12);      # 这些都对代码阅读有影响
%hash = (
    'foo' => 34,
    'bar' => 12,    # 最后的逗号便于程序的维护
);

$family_name{'fred'} = 'flintstone';    # 赋值
@array = %hash;         # 哈希列表转换，会将哈希中的键值都放入列表中
                        # 但不一定是按顺序展开（为快速检索作了排序），不过键值依然成对
%hash1 = %hash2;        # 复制
%inverse_hash = reverse %hash;      # 键值互换
        # 先将hash展开成键值对列表，就是(k1, v1, k2, v2,..)，然后用reverse反序，变成(vn, kn,..v1, k1)，再存回%inverse_hash
        # 如果其中哈希值键重复的话，列表后面的键会覆盖之前的
```
#### 哈希切片

哈希切片和数组切片差不多，看个例子就行
```perl
my @three_scores = @score{ qw/ barney fred dino/ };
# 注意score原本是个哈希，因为需要列表上下文使用@，这里返回一个值列表
@score{@players} = @bowling_scores;
# 这样就能够批量存入哈希了
print "There score were: @score{@players}\n";
# 字符串内插
```
### 子程序
```perl
sub sub_name {
    codes;
}
```
在子程序的运行过程中，它会不断运算，而最后一次运算的结果（不管是什么）都会成为子程序的返回值

子程序的参数存放在@_中，如果之前已经有了该变量，则会在子程序返回时被恢复。不仅如此，子程序也能支持传入一个哈希
```perl
sub left_pad {
    my %args = @_;
    my $newString = ($args{padChar} x ($args{width}) - length($args{oldString})), $args{oldString};
    return $newString;
}
print left_pad(oldString=>"pod", width=>10, padChar=>"+");
```
#### 预先声明

subs能够预先声明子程序，参数是子程序的列表，这样不需要&或括号，也能够调用子程序
```perl
use subs qw(fun1 fun2);
```
#### 原型

原型(prototype)又称作模板（template），可以像其他语言为子程序设定限制。原型将在编译阶段进行处理。而且使用原型子程序的时候必须不能使用&，否则限制将会无效
```perl
sub subroutine_name($$) {}  # takes two scalar arguments
sub subroutine_name(\@) {}  # argument must be an array, preceded with an @. 就是传入一个@array
sub subroutine_name($$;@) {}    # takes two scalar arguments and an optional array
# anything after the semi-colon is optional;
```
### 上下文
```perl
@people = qw( fred barney betty );
@sorted = sort @people; # 列表上下文：barney betty fred
$number = 42 + @people; # 标量上下文：45
@backwards = reverse @people;
$backwards = reverse @people;   # yttebyenrabderf
@a = undef; # (undef)
@a = ();    # 正确清空数组的方式
```
scalar函数可以强制指定标量上下文
```perl
print scalar reverse("HelloWorld");     # "dlroWolleH"
print scalar reverse("Hello", "World"); # "diroWolleH"
```
wantarray函数，如果子程序是在列表上下文中被调用，返回true；在标量上下文被调用，返回false；无要求返回，则是undef。

### 引用

在变量前面加上反斜杠\
```perl
my $ref_to_skipper = \@skipper;
my $second_ref_to_skipper = $ref_to_skipper;
# compare reference with ==
```
比如一个数组@array，分成两个部分@，array，@是个标识符，array是名，参照PeGS。而引用是和名有相当地位的
```perl
@array ------ @{$ref_to_array} ------ @$ref_to_array # 这就是解引用
$array[1] ------ ${$ref_to_array}[1]
```
如果我们想知道一个引用的类型时，可以使用ref操作符
```perl
my $ref_type = ref $hash_ref;
croak "I expected a hash reference" unless $ref_type eq 'HASH';
# 也不必记住'HASH'，可以再次使用ref
unless $ref_type eq ref {};
# 不然就在开头定义一个
use constant HASH => ref {}
croak "..." unless $ref_type eq HASH;
```
自带的模块Scalar::Util中有个reftype函数，可以检测ref的类型，不过reftype是从功能来判断的。假设一个对象引用像是哈希的引用，那么reftype $hash_ref eq ref {}就是真

| 常量   | 描述               |
|--------|--------------------|
| REF    | 指向指针的指针     |
| SCALAR | 指向变量的指针     |
| ARRAY  | 指向数组的指针     |
| HASH   | 指向散列的指针     |
| CODE   | 指向子程序的指针   |
| GLOB   | 指向typeglob的指针 |
| ''     | 不是引用           |

#### 符号引用

硬引用即是正常的引用，而符号引用内为另一个变量命名，而不是指向其变量值。例如typeglob就是一种符号引用，下面是另一种符号引用
```perl
$animal = 'dog';    # 把变量名animal存入符号表，并有一个引用指向值dog
$dog = 'lady'       # 同理
print "${$Animal}\n";   # 相当于一种跳转的引用，省略花括号也是可以的

$$animal = 'Lassie';
print "${$animal}\n";   # lassie

# 当然，这个可以被禁用
use strict 'refs';
```
#### 化简
```perl
${DUMMY}[$y] == DUMMY->[$y]
# 而且如果箭头两端都是下标，可以省略箭头
$all_with_names[2]->[1]->[0] == $all_with_names[2][1][0]
```
#### 引用存在

对数据结构而言，都存在至少一个到其本身的引用（初始定义的变量名也算）。当它失去所有到其内存地址的引用时，将视为不存在于内存当中
```perl
# 这个弄出一个匿名数组
my $ref;
{
    my $skipper = qw(blue_shirt hat jacket);    # ref count is 1
    $ref = \@skipper;                           # ref count is 2
}   # 此时$skipper失效了
print "$ref->[2]\n";                            # ref count is 1
```
但是当我们构建了一个循环引用时
```perl
{
    my @data1 = qw(one won);
    my @data2 = qw(two too to);
    push @data2, \@data1;
    push @data1, \@data2;
}   # 这就是循环引用了, $data1[2][3][2][3][2][3]..
```
大括号里@data1和@data2的引用都是2，当越出了范围，计数都变成了1，而我们却永远访问不到。这就是典型的内存泄漏了，会导致内存越用越少。解决方法很简单，在go out of scope前去掉循环引用就行了

    @data1 = (); @data2 = ();

#### 弱引用

Scalar::Util模块中的weaken函数可以将引用转化为弱引用，不进入正常引用的计数。当所有普通的引用全部消失，Perl就会清除该目标，而剩余的弱引用则会变成undef
```perl
use Scalar::Util qw(weaken);
$REGISTER{$self} = $self;
weaken($REGISTER{$self});
```
#### 匿名数组
```perl
my $ref = [ qw(blue hat jacket) ];
my $fruits = ['pineapple', 'papaya', 'mango'];
# 相当于上面的创建匿名数组
```
PS. Perl有自生成，只要有需要，它会尽力为我们创建变量
```perl
my $not_yet;
@$not_yet = (1, 2, 3);
```
#### 匿名哈希

同样的，花括号就可以用来构建匿名的哈希了。由于代码块和匿名哈希用的都是花括号，有时解释器不能分辨时，我们可以手动区分

- +{...} 在匿名哈希的括号前加上+，指明这是匿名哈希
- {;...} 在语句块的括号中加上分号，指明这是语句块

#### 子程序的引用
```perl
my $ref_to_subroutine = \&subroutine;   # 注意不要加括号
&{$ref_to_subroutine}(@args);   # 或者
&$ref_to_subroutine(@args);
$ref_to_subroutine->(@args);

for my $greet(\&skipper_greets, \&gilligan_greets) {
    $greet->('Professor');
}
```
#### 匿名子程序
```perl
my $ginger = sub { ... };
$ginger = ('skipper');
# 我们也可以直接在比较大的数据结构中使用
my %greet = {
    skipper => sub { ... },
    gilligan => sub { ... },
};
```
当我们使用匿名函数的时候想要调用自身
```perl
my $countdown;
$countdown = sub {
    state $n = 5;
    return unless $n > -1;
    say $n--;
    $countdown->(); # 调用自身
};
# 从Perl v5.16，可以使用__SUB__返回一个当前子程序的引用
my $countdown = sub {
    state $n = 5;
    return unless $n > -1;
    say $n--;
    __SUB__->();    # 在非匿名函数中也可以使用的
};
```
##### 闭包

闭包，能访问所有我们声明的在运行时存在的词法变量的子程序，不仅限于匿名子程序

在闭包内部访问变量能保证只要匿名子程序引用存在，变量的值就能保存
```perl
use File::Find;
my $callback;
{
    my $count = 0;
    $callback = sub { print ++$count, ":$File::Find::name\n" };
}
find($callback, '.');
```
callback就是个闭包，因它指向词法变量count。在裸块的结尾，count跑出了范围，然而，这个变量仍然被callback所指向的匿名子程序引用，所以该变量作为一个匿名的变量依旧存活

写成下面这样也许更优雅，从一个子程序返回子程序
```perl
use File::Find;
sub create_find_callback_that_counts {
    my $count = 0;
    return sub { print ++$count, ": $File::Find:name\n";
}
my $callback = create_find_callback_that_counts();
find($callback, '.');
```
闭包变量作为静态全局变量
```perl
{
    my $count;
    sub count_one { ++$count }
    sub count_so_far { return $count }
}
```
这个语句块必须要在使用里面的函数之前，或把代码放进BEGIN语句块中。BEGIN块会告诉Perl编译器，只要这个块被成功解释了（在编译阶段），就马上去执行这个块。完成后，块本身会被丢弃，保证了块内代码只会被执行一次，即使在循环或子程序中

PS
  - 用state也能够实现闭包的功能，但是state不能修饰数组和哈希，并不意味着不能定义，我们可以修饰其的引用
  - 闭包除了用在子程序中，还可以放在map中

#### open打开引用

从v5.16开始，可以使用open filehandle到变量的引用，而不仅是文件
```perl
open my $string_fh, '>', \my $string;
# 有时我们需要捕获输出，通过写入到变量的引用防止内容离开程序
use CGI;
open my $string_fh, '>', \my $string;
CGI->save($string_fh);  # 因为save的方法必须跟上filehandle
# 同样的nstore是直接跟上文件名写入数据，nstore_fd也必须跟上文件句柄
nstore_fd \@data, $string_fh;
```
或者我们来暂时转移STDOUT
```perl
{
    local *STDOUT;
    open STDOUT, '>' \$string;
    $some_obj->noisy_method();
}   # 出了这里，STDOUT又会恢复
```
再来有用的，我们对多行文件进行处理
```perl
my @lines = split /$/, $multiline_string;
foreach my $line (@lines) {
    &do;
}

open my $string_fh, '<', \$multiline_string;
while (<$string_fh>) {
    &do;
}
# 相比之下，两段代码的功能相同，但第一段代码引入了数组@lines，@lines内的数据是和多行字符串一样的，浪费了空间
```
#### 数据结构的处理

* Data::Dumper模块可以形象化数据的结构
  ```perl
  use Data::Dumper;
  print Dumper($data); # 或dump
  # 一般情况下，传入哈希的引用是为了保持哈希的结构，而不是被形象成分开的列表元素
  # 因为参数本身就是列表，Dumper以为你传入了多个参数
  ```
  Dumper显示出来的代码是符合Perl的语法的，这样一来我们也可以通过eval执行这串代码
  ```perl
  my $string = Dumper( \@data1, \@data2);
  my $data_structure = eval $string;
  # 但这样也有问题，Dumper中已经给两个数据结构命名成$VAR1 $VAR2
  # 要解决这个，我们可以用Dump方法，注意，是方法
  print Data::Dumper->Dump(
    [ \@data1, \@data2 ],
    [ qw(*data1, *data2) ]
  );
  # Dump伴随着两个array引用，第一个引用是变量我们想dump的，第二个含有名字列表我们想命名的
  # 这里还使用了*前缀，让Data::Dumper来帮我们决定变量的类型
  ```
* Storable模块中的freeze函数，返回一个代表数据结构的二进制字符串
  ```perl
  use Storable qw(freeze);
  my $frozen = freeze [ \@data1, \@data2 ]
  # 相比于Dumper的输出，freeze占用的容量更少
  # 我们可以将这数据存入文件，通过套接字传输，或其他句柄
  my $data = thaw($input);  # 恢复原来的数据
  # 利用nstore直接写入文件，不用store，n表示network order，能够跨机器
  nstore [\@data1, \@data2] $filename;
  # 再利用retrieve恢复数据
  my $array_ref = retrieve $filename;
  ```
  用处: 我们可以通过freeze再thaw，来深度复制一份相同且互不干扰的数据结构。正常情况下，不涉及引用的话，`my @array1 = @array2`也是不相干的。但也仅仅在不存在也不引用的情况下，这叫**shallow copy**。而利用前面dump之类的，是**deep copy**

  然而这一切都被设计好了，Storable里有个函数dclone，做的就是这事`my @packed = @{ dclone \@provisions }`

* YAML(Yet Another Markup Language)模块，将数据结构生成yaml的样式
  ```perl
  use YAML;
  print Dump(\%data);
  ```
* JSON(JavaScript Object Notation)
  ```perl
  use JSON;
  print to_json( \%total_bytes, {pretty => 1} );
  # 利用pretty属性让输出更加好看易读
  my $hash_ref = from_json($json_string);
  ```


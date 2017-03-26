* [变量作用域](#变量作用域)
    * [全局变量和私有变量](#全局变量和私有变量)
        * [符号表](#符号表)
    * [持久性私有变量](#持久性私有变量)
    * [包变量](#包变量)
    * [local变量](#local变量)

## 变量作用域

### 全局变量和私有变量

默认情况下，Perl中直接定义的变量都是全局变量，但我们可以借助my操作符来创建局部变量，例如`my ($total,$count)`

#### 符号表

Perl会在符号表中维护全局变量的记录，而符号表在整个程序内都有效。每一个标识符都有一个指针，指向每一个变量类型的槽位(slot)

| 包    | 标识符 | 类型    | 变量           |
|-------|--------|---------|----------------|
|       |        | SCALAR  | $bar           |
|       |        | ARRAY   | @bar           |
|       |        | HASH    | %bar           |
|       |        | CODE    | &bar           |
| foo:: | bar    | IO      | 文件和目录句柄 |
|       |        | GLOB    | *bar           |
|       |        | FORMAT  | 格式名         |
|       |        | NAME    | 标识符         |
|       |        | PACKAGE | 包             |

如果我们获得了GLOB类型，就可以像访问哈希表一样访问该部分，返回的**引用**就是指向该槽位对应的数据
```perl
$bar = 'Buster';
@bar = qw(Mimi Roscoe);
$foo = *bar{SCALAR};
$baz = *bar{ARRAY};
# *bar就相当于一张表了，变量表只能作为右值使用
```
我们可以使用typeglob赋值 `*foo = *bar` ，那么两个标识符将会一样，同步更改数据

也可以只把部分引用赋给一个typeglob，只有引用指向的部分受到影响。
```perl
*foo = \$scalar;    # 只会影响到typeglob的SCALAR部分
*foo = \@array;     # 只会影响到typeglob的ARRAY部分
```
文件句柄的引用 `\*FILEHANDLE`
如果尝试使用变量还未使用的槽位，得到的结果为undef。奇怪的是，在访问未使用的SCALAR槽位的时候，它返回的是一个指向undef的匿名标量引用

通过my的变量名称被不保存到符号表中，而是位于临时缓冲区里，由于typeglob仅能关联到特定包的符号表上，因此my不能对它私有化，若要本地化，必须使用local（PS，也不能使用our)
```perl
$color = 'rainbow';
@color = ("red", "green", "yellow");
&printit(*color);
sub printit {
    local *which = @_;
    print *which, "\n";         # *main::color
    $which = "Prism of Light";  # $color变成Prism of Light
    $which[0] = "PURPLE";       # PURPLE, green, yellow
}
```
### 持久性私有变量
```perl
sub marine {
    state $n = 0;
    $n += 1;
    print "$n\n";
}
```
state修饰的变量是属于当前子程序的私有变量，并且在多次调用这个子程序期间保留该变量的值

### 包变量

our修饰的变量就属于包变量（属于全局的变量），和my有点区别

1. 重复定义
  ```perl
  my $foo = 20;
  my $foo;      # 重复定义后foo的值又变成了undef了
  # 而our不会，重复定义只会得到一个alias，值也不会被清除掉
  ```
2. our修饰的变量定义在哪个package下，变量的前缀就一直是那个包名，如果在另一个包名下重新定义，完全不会影响到原来的变量

### local变量

local和my的区别，local就像是其所在语句块内的全局变量
```perl
sub test { print $val }
{
    my $val = 'haha';
    test;       # 不会输出
}   # my声明的变量对该语句块中调用的任何其他子例程是不可见的
{
    local $val = 'haha';
    test:       # 会输出haha
}   # local则是可见的
```

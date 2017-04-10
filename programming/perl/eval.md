
eval
------------------------------------

### 执行命令

eval直接跟上string，能将其当成代码来执行，但这通常都被当成**程序污点**来处理，非取巧时不推荐

### 捕获错误

- eval表达式能捕获在它监察范围的致命错误，并立即停止运行整个块，继续运行其余代码。
但其只是一个表达式，所以**语句结束应该写上分号**
```perl
# 将代码包裹在eval中
eval { $barney = $fred / $dino };
# 也可以配合HERE文档
eval<<"EOF";
    chdir ”joker" or die "Can't chdir: $!\n";
EOF
```
- eval的返回值是语句块中最后一条表达式的执行结果或是return语句，这和子程序中的一样。
如果捕获到了错误，返回值为undef，并在特殊变量$@中设置错误信息，否则特殊变量就是空的
```perl
my $barney = eval { $fred / $dino };
use 5.010;
my $barney = eval { $fred / $dino } // 'NaN';
# 如果没有有意义的返回值，为了程序的简短，我们可以自己构造返回值
unless ( eval { some_sub(); 1 } ) {
    print "I could't divide by \$dino: $@" if $@;
    # 其实[if $@]可以省略，这代码很有意思
}
# 列表上下文中，捕获到错误就返回空列表
my @averages = ( 2/3, eval { $fred / 0 }, 22/7 };
# 注意不是(2/3, undef, 22/7)，而是(2/3, 22/7)
```
- eval可以进行嵌套，通常内层的错误是不会泄露到外层的，但如果内层捕获到错误并用die报告的话，外层的eval就会捕获这个错误。
而且如果有了eval的嵌套，一般推荐在内层写上`local $@`，不干扰高层的错误
```perl
{
    local $@,   # 不干扰高层错误
    eval {
        die "An unexpected exception message" if $unexpected;
        die "Bad denominator" if $dino == 0;
        $barney = $fred / $dino;
    }
    if ($@ =~ /unexpected/ ) {
        ...;
    } elsif ($@ =~ /denominator/) {
        ...;
    }
}
```

#### 例外

eval有四种错误是不能捕获的
1. 出现在源代码中的语法错误
2. 让perl解释器本身崩溃的错误，内存溢出或收到无法接管的信号
3. 警告: `-w`、`use warnings`
4. exit命令

#### 相关模块

Try::Tiny模块提供更不易出错的捕获错误的方式，而且还把错误消息放到了默认变量$_里，规避$@的潜在干扰
```perl
use Try::Tiny;
try {
    ...;
} catch {   # catch和finally是可省略的
    ...;
} finally {
    ...;
};  # 注意，这里是要加分号的
```

#### autodie

autodie正如名字说的功能一样，但需要Perl 5.10.1
```perl
use autodie;
use autodie qw(open system :socket);    # 指定要实行的操作符
```
当autodie抛出错误时，它会把一个autodie:exception对象放到$@变量中
```perl
use 5.010;
use autodie;
use Try::Tiny;
try {
    open my $fh, '<', $filename;
} catch {
    when ('open') { say 'Got an open error' }
    # 对于when来说，错误已经在$_中了
}
```

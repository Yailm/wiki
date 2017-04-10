
命令行选项
------------------------------------

命令行选项是程序参数，常用于改变程序行为

### 选项类型

#### 单字符选项

每个选项前面都有连字符，我们需要逐个处理这些选项(处理方式：Getopt::Easy、Getopt::Std和Perl的-s选项)

    % foo -i -t -r

如果选项可能有取值(必需的或可选的)，在取值的选项之间可能会有分隔字符(处理方式：Getopt::Easy、Getopt::Std、Getop::Mixed和Perl的-s选项)

    % foo -i -t -d/usr/local
    % foo -i -t -d=/usr/local
    % foo -i -t -d /usr/local

成组的单字符选项，也叫作捆绑选项或集群选项(处理方式：Getopt::Easy、Getopt::Mixed、Getopt::Std)

    % foo -itr

#### 多字符选项

选项前有连字符，可能有取值(处理方式：Perl的-s选项)

    % foo -debug -verbose=1

具有双连字符的多字符选项，可以和单连字符的单字符选项或成组的单字符选项一起使用(处理方式：Getopt::Attribute、Getopt::Long、Getopt::Mixed)

    % foo --debug=1 -i -t
    % foo --debug=1 -it

单独的双连字符表示选项参数的结束，因为有时候程序的参数也会以连字符打头(处理方式：Getopt::Long、Getopt::Mixed)

    % foo -i -t --debug -- --this_is_an_argument

#### 使用别名

可以使用不同格式的选项或别名来表示相同的用途(处理方式：Getopt::Lucid、Getopt::Mixed)

    % foo -d
    % foo --debug

#### 古怪的选项

各种古怪的选项或者根本没特征的选项(处理方式：Getopt::Declare)

    % foo input=bar.txt --line 10-20

### 选项处理

#### Perl的-s

Perl的-s选项就会将选项转换成包内的变量，它可以处理单连字符和双连字符选项。选项可以用取值，也可以没有。

我们可以在命令行或脚本的第一行(shebang行)指定-s选项
```perl
#!/usr/bin/perl -s
# perl_s_abc.pl
use strict;
use vars qw($a $abc);
print "The value of the -a switch is [$a]\n";
print "The value of the -abc switch is [$abc]\n";
```
如果使用了选项但没有取值，Perl会将其取值设置为1，如果选项用等号指定了取值(这是指定取值的唯一方法)，Perl会将其设置为该选项的取值

    %perl ./perl_s_abc -abc=fred -a
    The value of the -a switch is [1]
    The value of the -abc switch is [fred]

我们双连选项时，-s选项也能处理

    %perl ./perl_s_debug.pl --debug=11

这会导致Perl产生一个叫作${'-debug'}的非法变量，该变量不能在strict安全模式中使用。可以使用符号引用的方式来绕过Perl的变量命名规则，所以我们将其用花括号包围起来。还要绕开正常的strict下的变量声明规则，所以我们必须关闭strict的‘refs’检查才能使用这些变量
```perl
#!/usr/bin/perl -s
# perl_s_debug.pl
use strict;
{
    no strict 'refs';
    print "The value of the --debug switch is [${'-debug'}]\n";
    print "The value of the --help switch is [${'-help'}]\n";
}
```

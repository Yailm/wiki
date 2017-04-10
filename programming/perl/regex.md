
正则表达式
------------------------------------

    $string =~ /pattern/;

双斜线表示法，实际上是`m/pattern/`的简写（定界符是正斜杠时可以省略m），也能和qw/ /一样，自由选择成对的定界符

  - `m(pattern)`
  - `m<pattern>`
  - `m#pattern#`
  - `m!pattern!`

使用成对的定界符，通常不用担心pattern中出现相同的定界符，因为它们在pattern也是成对存在的，Perl会自动识别这些。但有个特殊的，尖括号`<\>`不是正则表达式的元字符，所以它们不会成对出现，这时就必须要转义

pattern是支持变量内插的
```perl
print "Do you like Perl?\n";
my $answer = (<STDIN> =~ /\byes\b/i);   # 匹配成功返回1，否则空
if ($answer) { ... }
```
### 正则表达式的预编译

使用qr/ /操作符（类似qw，qx，可以更换分隔符），返回一个编译好的正则表达式的引用

```perl
my $regex = qr/$pattern/ 
#如果打印出$regex，会得到(?^:模式)，这仅仅是预编译，如果pattern有错误需要自己捕获
my $regex = eval { qr/$pattern/ };
# 然后编译好的$regex我们正常使用就行了
$string =~ m/$regex/;
$string =~ s/$regex/Skipper/
$string =~ $regex;
$string ~~ $regex;
```
这里有一种用法，我们将正则表达式的引用放在数组里
```perl
my $name = 'Ginger';
say 'match!' if $name ~~ @patterns;
# 这里的$name ~~ @patterns相当于grep { $name ~~ $_ } pattern,当然数组不为空时就说明至少匹配了一个

# 不仅如此，如果想知道具体哪个匹配，可以使用哈希
my @match = grep { $name =~ $pattern{$_}} keys %patterns;
say "Match @match" if @match;

# 用上学过的模块List::Util;
use List::Util qw(first);
my @match = first { $name =~ $pattern{$_}} keys %patterns;

# 下面是寻找正则匹配最右的index
rightmost('Mary Ann and Ginger', qr/Mary/, qr/Gin/);
sub rightmost {
    my ($string, @patterns) = @_;
    my $rightmost = -1;
    while (my($i, $pattern) = each @patterns) {
        $position = $string =~ m/$pattern/ ? $-[0] : -1;
        $rightmost = $position if $position > $rightmost;
        # @-和@+这两个数组分别是匹配成功的位置的左、右边界
    }
    return $rightmost;
}
```
### 分组

通常情况是指被括号所引起的内容

#### 不捕获分组

`(?:PATTERN)`

#### 命名分组

`(?<LABEL>PATTERN)`或`(?P<LABEL>PATTERN)`

获取到的分组会保存在特殊哈希`%+`里面，其中的键就是在分组中使用的特殊标签，其对应的值就是捕获的字符串
```perl
use 5.010;
my $name = 'Fred or Barney';
if ($names =~ m/(?<name1>\w+) (?:and|or) (?<name2>\w+)/) {
    # 匹配语句通常直接放在if的条件内
    say "I saw $+{name1} and $+{name2}";
}
```
#### 自动捕获分组

自动捕获分组是perl自己会执行且保留的分组，将整个目标字符串分成三部分，放到变量中

| 变量符号 | 内容                       |
|----------|----------------------------|
| $&       | 保存实际匹配的部分         |
| $\`      | 保存匹配区段之前的部分     |
| $'       | 保存剩下的未被匹配到的部分 |
```perl
if ("Hello there, neightor" =~ /\s(\w+),/) {
    print "That was ($`)($&)($').\n";
}

--- OUTPUT ---
That was (Hello)( there,)( neightor).
```

PS. 一旦在程序中任何部分使用了自动捕获变量，其他正则表达式的运行速度也会变慢

在5.10以上的版本，提供了修饰符/p，只会针对某个特定的正则表达式开启类似的自动捕获变量，但名称已经改变`${^PREMATCH}`, `${^MATCH}`, `${^POSTMATCH}`。好处是，其他的正则表达式的速度不会变慢，因为没有开启自动捕获分组
```perl
use 5.010;
if ("Hello there, neightor" =~ /\s(\w+),/p) {
    print "That was (${^PREMATCH})(${^MATCH})(${^POSTMATCH})";
}
```
#### 捕获变量

捕获分组依次点算左括号的序号，用`\1`，`\2`，而在Perl v5.010后，可以使用`\g{1}`，`\g1`，还可以填入负数，表示**相对**反向引用(从**当前**位置反向给捕获组编号)

    (  )(  ).(  )\g{-1}((  )   )
     -3  -2   -1         2  1

$1, $2, $3在**匹配结束**后取得相应括号的内容，这些捕获变量通常能存活到下次成功匹配为止，也就是说，失败的匹配不会改动上次匹配成功时捕获的内容

若使用命名分组，可以使用`\g{label}`或`\k<label>`，甚至用python的语法`(?P=label)`

### 模式匹配修饰符

- /i: 进行大小写无关的匹配
- /s: 可以让点号匹配任何字符（实际上与/m相反，将字符串作为单行处理）  
  原来的点号相当于[^\n]或是v5.12中的\N，使用了\s可以让点号变成[\d\D]
- /x: 允许我们在模式中插入空白符和注释，使它更易阅读、理解
  - 由于原来表示空白的空白符都会被Perl忽略，但我们可以通过转义方式实现。比如在前面加上反斜线或者使用\t，\s之类的
  - 注释使用#，如果要使用其字面意思，可以写成\#或者使用字符集[#]，还有，注释部分不要使用定界符，会被视为模式终点
- /m: 多行匹配，多行处理，会把换行符隔开的字符串当成行。[TODO]
- /a: 采用Ascii方式解释简写意义
- /u: 采用Unicode方式
- /l: 采用本地语言的方式
- /d: 按照传统的行为，即按Perl自己的推断
- /c: 当匹配失效时，防止匹配被重置，能够实现在一个位置多次试匹配
- /p: 单独为这次匹配开启自动捕获分组
- /g: 全局匹配，能够在目标字符串进行多次匹配
  ```perl
  my $line = "Just another regex hacker, Perl hacker, and that's it!\n";
  while (1) {
      my ($found, $type) = do {
          if ($line =~ /\G([a-z]+(?:'[ts])?)/igc) {
              ($1, "a word")
          } elsif ($line =~ /\G(\n)/gc) {
              ($1, "newline char")
          } elsif ($line =~ /\G(\s+)/gc) {
              ($1, "whitespace")
          } elsif ($line =~ /\G([[:punct:]])/gc) {
              ($1, "punctuation char")
          } else {
              last;
          }
      };
      print "Found a $type [$found]\n";
  }
  ```
模式匹配修饰符不仅可以放在分隔符的后面，这里它们也可以放在模式中，表示对括号中的模式有修饰效果
```perl
qr/(?mi:Giligant$)/
qr/abc(?i:Gilligant$)/
qr/abc(?-i:Gilligant$)/i    # 在括号中取消/i效果
qr/abc(?m-i:Gillgant$)/i    # 混合
```

### 锚位

通过定锚位，我们可以让模式仅在字符串指定位置匹配

- \A: 匹配字符串的**绝对**开头，若开头不匹配，是不会移到下一个位置的，例`m{\Ahttps?://}i`
- \z: 匹配字符串的**绝对**末尾，后面是不会有东西的
- \Z: 匹配字符串的行末，它允许后面出现换行符
- ^$: 行首锚位，行末锚位
  ```perl
  $_ = 'This is a wilma line
  barney is on another line
  but this ends in fred
  and a final dino line';
  /fred$/m      # success
  /^barney/m    # success
  /fred$/       # fail
  ```
  如果没有/m，^和$的行为就如同\A和**\Z**一样，但添加了多行匹配，就会改变原来的意思
- \b: 单词锚位，匹配任何单词的首尾  
  PS. 这里所说的单词是由\w字符组成的字符集，也就是英文字母，数字与下划线组成的字符串
- \B: 非单词锚位，匹配不是单词边界的位置
- \G: 匹配上一次匹配结束的地方，通常是m//g模式的离开位置
- \K: v5.10, 允许它之前的模式只匹配而不被替换，替换操作之对\K后的字符串有效

### 往返断言

- `(?=pattern)`: 正向前断言
- `(?!pattern)`: 负向前断言(模式不存在，才完成匹配)
- `(?<=pattern)`: 正向后断言
- `(?<!pattern)`: 负向后断言

### 优先级

| 由高到低                        |                              |
|---------------------------------|------------------------------|
| `(..), (?:..), (?<label>..)`    | 圆括号(分组或捕获)           |
| `*, +, ?, {n, m}`               | 量词                         |
| `\A, \Z, \z, ^, $, \b, \B, abc` | 锚位和序列(即是正常的字符串) |
|                                 | 择一竖线                     |
| `a, [abc], \d, \1, \g{2}`       | 原子(单个字符)               |

### 查找替换

    s/PATTERN/REPLACEMENT/
```perl
$_ = "He's out bowling with Barney tonight.";
s/Barney/Fred/;
# 还可以用上自动捕获分组的三种变量
s/\w+$/($`!)$&/;
```
- s///返回值是布尔值，只有替换成功的时候为真
- s///的定界符也是可以替换的，但是如果是有左右之分的成对字符，这里就必须使用两对，`s{p1}{p2}`，`s[p1][p2]`，`s<p1><p2>`
- 同样使用`=~`来绑定
- 额外的修饰符
  - /o: 只编译模式一次，用于优化搜索流程
  - /e: 将替换一侧作为表达式来求值，例`s/65/6*7/e`，用42替换65
- 一些大小写转换
  - \U: 会将其后的所有字符转换成大写
  - \L: 会将其后的所有字符转换成小写
  - \u和\l: 只会影响紧跟其后的第一个字符
  - 还可以将它们混用，`s/(fred|barney)/\L\u$1/ig`，转换成开头大写的英文，\L\u反过来也是一样的
  - 不仅能用在replacement中，还可以用在任何双引号的字符串中
- TIP: 无损替换-----`(my $copy = $origin) =~ s/\d+ ribs?/10 ribs/;`

### 正则表达式相关

#### split

split操作，可以根据给定的模式（默认是空白符）拆分字符串，返回字段列表

    split(DELIMITER, $string, LIMIT)

- LIMIT限制拆分的字段数，可省略
- 如果两个分隔符连在一起，就会产生空字段，split会保留开头的空字段，却不会保留结尾的空字段
- join功能与split相反，会将列表合成一个字符串
  - 第一个参数（字符串）理解成胶水，会把它加入剩余所有参数之间

#### 列表上下文的匹配
```perl
$_ = 'Hello there, neightor';
my ($first, $second, $third) = /(\S+) (\S+), (\S+)/;
---
$text = 'Hello there, neightor';
my @words = ($text =~ /([a-z]+)/ig);
---
my $data = 'Barney Rubble Fred Flintstone Wilma Flintstone';
my %last_name = ($data =~ /(\w+)\s+(\w+)/g);
```
列表上下文中，如果匹配成功返回所有捕获变量的列表，如果匹配失败，则会返回空列表

#### 正则表达式的模块

因为有了预编译的模式，就有了含有实用的预编译模式的模块

* Regexp::Common提供了许多正则表达式，存放在哈希%RE中
  ```perl
  use Regexp::Common qw(URI);
  while (<>) {
      print if /$RE{URI}{HTTP}/;
  }
  ```
* Regexp::Assemble能够帮助我们构建高效的模块，如我们原来的表达式`qr/(?:Mr. Howell|Mrs .Howell|Mary Ann)/`，其中三个表达式都以M开头，但是我们仍然检查了3次，尝试用模块来构建
  ```perl
  use Regexp::Assemble;
  my $ra = Regexp::Assemble->new;
  for ('Mr. Howell', 'Mrs. Howell', 'Mary Ann') {
      $ra->add("\Q$_");
  }
  say $ra->re;  # (?^:M(?:rs?\. Howell|ary Ann))
  ```


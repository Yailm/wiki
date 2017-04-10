
面向对象
------------------------------------

类->方法就是包调用method子程序，在perl里面子程序和方法是没有区别的。
```perl
Cow::speak;
Cow->speak;
$beast = 'Cow';
$beast->speak;  # 都是一样的

# 真正
Class->method(@args);
Class::method('Class', @args);
# Class::method就是包名下的方法，比如speak，就是Cow::speak
```
### 类的实例化
```perl
# 以下为尝试，并不是常规做法
my $name = 'Mr. Ed';
my $tv_horse = \$name;  # $$tv_horse就是'Mr. Ed'
bless $tv_horse, 'Horse';
# $tv_horse是一个实例数据（马的名字）的引用，bless操作符根据引用找到其指的变量将$tv_horse变成一个horse对象。
# bless函数可以带有两个参数，第一个参数就是引用，第二个参数则是包的名称，如果没有提供第二个参数的话，就假定使用当前的包

sub name {  # 获得name的函数
    my $self = shift;
    $$self;
}
sub named { # 定义名字的函数，bless同时也返回了$name的一个引用
    my ($class, $name) = @_;
    bless \$name, $class;
}   # 但当我们使用Horse->name时会出现错误
# ref操作符可以解决，当作用于一个被bless过的标量时，返回一个字符串，而返回undef当被用于string
sub name {
    my $either = shift;
    ref $either ? $$either          # it's an instance, return name
        : "an unnamed $either";     # it's a class, return generic
}

# 一般实例中的数据不只一个，我们可以用其他数据结构的引用
my $lost = bless { name => 'Bo', color => 'white' }, 'Sheep';
$lost->{name}   # 仍然可以当成哈希来使用
sub name {
    my $class = shift;
    $class->{name};
}

# 通过wantarray函数，我们改进get_color函数
sub set_color {
    my $self = shift;
    if (defined wantarray) {
        # this method call is not in void context(有上下文，即要求返回值)
        my $old = $selfl->{color};
        $self->{color} = shift;
        $old;   # set_color会返回旧值
    } else {
        # this method call is in void context
        $self->{color} = shift;
    }
}
# 然后我们可以同时写这样的语句
$tv_horse->set_color(
    $eating->set_color( color_from_user() ) # 无上下文，设置同样的颜色
);
my $old_color = $tv_horse->set_color('orange');

# 简洁的getter和setter
sub color { $_[0]->{color} }
sub set_color { $_[0]->{color} = $_[1] }

# 试着写在一起
sub color {
    my $self = shift;
    if (@_) {
        # setter
        $self->{color} = shift;
    } else {
        # getter
        $self->{color};
    }   # 简洁但不建议
}

# 我们有时不希望实例的方法和类的方法混在一起使用
use Carp qw(croak);
sub instance_only {
    ref(my $self = shift) or croak "instance variable needed";
    # use $self as the instance
}
```
### 继承

当我们有一个父类Animal要被继承，在子类中
```perl
use Animal;
our @ISA = qw(Animal);  # @ISA中包含了此类继承的父类
sub function { }        # 要被重载的方法或是添加的新的方法，父类的东西是默认保留到子类的
# 开头的那两句话，可以缩减成一句话
use parent qw(Animal);  # 优点是这运行在程序的编译阶段
# 如果父类在同一文件中
use parent --norequire, 'Foo', 'bar';
# 如果在v5.101以上，可以用base取代
use base 'Foo', 'bar';
```
> base和parent的[区别](http://stackoverflow.com/questions/876471/what-is-the-difference-between-parent-and-base-in-perl-5)

重载时使用$class->SUPER::speak;调用父类的原方法
```perl
sub speak {
    my $self = shift;
    $self->SUPER::speak;
    # 这会自动搜索@ISA的父亲，找到那个被重载的方法并调用
    print "...";
}
```
### UNIVERSAL

Perl寻找类的方法，先在该类中查找，再去@ISA中列出的类中查找。而且使用的深度搜索，按顺序查找该数组中元素类及其父类的方法，再换下一个元素。再没有就查找UNIVERSAL这个特殊的类，其实就是相当于UNIVERSAL是所有类的父类就行了

UNIVERSAL中包含DOES和can函数，所以这两个适用于所有的类
```perl
use v5.10;
if (Horse->DOES('Animal')) {    # does Horse do Animal?
    print "A Horse is an Animal.\n";
}

# 测试类或是实例是否一类确定的东西
my @horse = grep $_->DOES('Horse'), @all_animals;
my @horse_only = grep ref $_ eq 'Horse', @all_animals;
# 两句是不同的，前者包含马以及马的子类，而后者只有马这个类
```
### AUTOLOAD

如果Perl在上述过程中都找不到方法，它会重复该顺序查找AUTOLOAD方法。如果存在AUTOLOAD，则它会被调用，传给它相同的参数，而原来要调用的函数名称会被存储在$AUTOLOAD中（包含包名）

用处，当程序过大时，比较少用到的函数可以到使用的时候再临时加载
```perl
use Carp qw(croak);
sub AUTOLOAD {
    our $AUTOLOAD;
    (my $method = $AUTOLOAD) =~ s/.*:://s;  # remove package name
    if ($method eq 'eat') {
        # define eat;
        eval q{
            sub eat {
                # long definition goes here
            }
        };  # end of eval's q{} string
        die $@ if $@;
        goto &eat;      # jump into it
    } else {
        croak "$_[0] does not know how to $method\n";
    }
}
```
运行过一次AUTOLOAD后，&eat就已经被定义过了，再次使用eat就不会来到这个函数来了

AUTOLOAD还可以用于加速属性访问器，这样有时候我们可能有很多的属性，我们不需要每个都写一个getter和setter
```perl
use Carp qw(croak);
sub AUTOLOAD {
    my @elements = qw(color age weight height);
    our $AUTOLOAD;
    if ($AUTOLOAD =~ /::(\w+)$/ and grep $1 eq $_, @elements) {
        my $field = ucfirst $1;     # first character in uppercase
        {
            no strict 'refs';
            *$AUTOLOAD = sub { $_[0]->{$field} };
            # *$AUTOLOAD是类型通配符(typeglob)
        }
        goto &$AUTOLOAD;
    } elsif ($AUTOLOAD =~ /::set_(\w+)$/ and grep $1 eq $_, @elements) {
        my $field = ucfirst $1;
        {
            no strict 'refs';
            *$AUTOLOAD = sub { $_[0]->{$field} = $_[1] };
        }
        goto &$AUTOLOAD;
    }
}

# 然而我们想到这里的话，有了CPAN，我们也不用自己写模块，已经有了Class::MethodMaker这个模块
package Animal;
use Class::MethodMaker
    new_with_init   => 'new',
    get_set         => [-eiffel => [qw(color height name age)]],
    abstract        => [qw(sound)];
# sound方法被定义成抽象方法，由子类来实现
sub init {  # 生成的new方法会调用init
    my $self = shift;
    $self->set_color($self->default_color);
}
sub named {
    my $self = shift->new;
    $self->set_name(shift);
    $self;
}
sub generic_name {
    my $either = shift;
    ref $either ? $either->name : "an unnamed $either";
}
sub speak {
    my $self = shift;
    print $self->generic_name, ' goes ', $self->sound, "\n";
}
sub eat {
    my $self = shift;
    my $food = shift;
    print $self->generic_name, " eats $food\n";
}
sub default_color {
    'brown'
}
```
### DESTROY

当一个对象被摧毁时会自动寻找（如果存在AUTOLOAD则会进入里面）并调用DESTROY函数，查找不到也不会有任何提示（同import和unimport一样）

如果一个对象中包含其他的对象，摧毁时先从外层开始调用DESTROY函数。因为只有外层的结构消失，内部的数据引用才可能变成0
```perl
sub DESTROY {
    my $self = shift;
    $self->SUPER::DESTROY if $self->can('SUPER::DESTROY');
    # 调用父类的原方法
    &do_more();
}
```
### import

import函数看起来像是这样的
```perl
sub import {
    for (qw(filename basename fileparse)) {
        no strict 'refs';
        *{"main::$_"} = \&$_;   # 创建main包下的别名
    }
}
```
通常情况下，我们用Exporter模块的import
```perl
package Animal::Utils;
use parent qw(Exporter);    # 让其成为父类
use Exporter qw(import);    # 直接导入，更常见

# Exporter提供的import会检查模块包中的@EXPORT
our @EXPORT = qw(basename dirname fileparse);
```
  - @EXPORT定义了该模块默认导入的函数，当我们没有传函数列表给import(包括use中的import)时，它会自己查看@EXPORT。注意import没有提供列表和提供一个空的列表是不同的
  - @EXPORT_OK包含我们不想默认被子程序引入的，虽然可以被显式引入（写在列表中）
  - @EXPORT_FAIL列出了不能导出的符号，可以通过写全名引入
  - 要指定任何子程序名，必须要在@EXPORT或@EXPORT_OK中，否则该请求会被Exporter->import拒绝

#### 标签分组

分组的引入
```perl
use Fcntl qw(:flock);
# :DEFAULT标签，在我们没有提供import的列表时，则会被自动引入，也可以被混合使用
use Navigate::SeatOfPants qw(:DEFAULT get_north from_professor);
# 也可以使用正则表达式
use Testmodule qw(/^fu/);
```
分组的实现，通过将分组存入%EXPORT_TAGS，键值为组名（不带冒号），值为一个列表的引用
```perl
our %EXPORT_TAGS = {
    all         => [@EXPORT, @EXPORT_OK],
    gps         => [ qw( according_to_GPS) ],
    direction   => [ qw(
        get_north_from_professor
        according_to_GPS
        guess_direction_toward
        ask_the_skipper_about
    ) ],
};
```
### caller

caller函数能够返回the calling package，我们改进刚才的import函数
```perl
sub import {
    my $package = caller;
    foreach my $name (qw(filename basename fileparse)) {
        no strict 'refs';
        *{$package . "::$name"} = \&$_;
    }
}
# 还可以获得更多的信息
my ($package, $file, $line) = caller;
warn "I was called by $package in $file\n";
```
### Moose

Moose是Perl中专门面向对象的模块
```perl
package Horse;
use Moose;
use namespace::autoclean;
# 一般使用这个或在定义完一个类后加上no Moose;

has 'name' => ( is => 'rw' );
has 'color' => ( is => 'rw' );

__PACKAGE__->meta->make_immutable;  # 使该类不可变
1;
```
仅仅是这样定义，我们已经可以这样使用了
```perl
use Horse;
use v5.10;
my $talking = Horse->new( name => 'Mr. Ed' );
$talking->color('grey')
say $talking->name, ' is colored', $talking->color;
```
- 设定继承`extends Animal`
- 设定默认值
  ```perl
  has 'sound' => ( is => 'ro', default => 'neigh' );
  has 'sound' => ( is => 'ro', default => sub {
      confess shift, " needs to defined sound!"
  });
  ```
- 设定为必须
  ```perl
  has 'name' => ( is => 'rw', require => 1 )
  ```
- 设定为read-only时，我们不能通过该方法来修改属性，但我们可以指定某些方法可以修改
  ```perl
  has 'color' => (
      is => 'ro',
      writer => '_private_set_color',   # 指定可以修改该属性的类
      default => sub { shift->default_color },
  )
  ```
- 设定限定的方法参数结构
  ```perl
  has 'color' => ( is => 'rw', isa => 'Str' );
  # 具体预置的类型，可以到Moose::Util::TypeConstraints中找，或自己定一个类型
  {     # 在裸块中定义，之后还能回到原来的包
      use Moose::Util::TypeConstraints;
      use namespace::autoclean;
      {
          my %colors = map { $_, 1 } qw( white brown black );
          subtype 'ColorStr'
              => as 'Str'
              => as where { exists $colors{$_} }
              => message {
                  "I don't think [$_] is a real color"
              }
      }
  }
  # 比较简写的方法
  use Moose::Util::TypeConstraints;
  use namespace::autoclean;
  enum 'ColorStr' => [qw( white brown black )];
  ```
- 除了直接从Animal继承，我们也可以将Animal定义成一个role
  ```perl
  package Animal;
  use Moose::Role;
  require qw(sound);    # 要求‘子类’要实现的方法
  has 'name' => ( is => 'rw' );
  has 'color' => ( is => 'rw' );
  sub speak {
      my $self = shift;
      print $self->name, " goes ", $self->sound, "\n";
  }
  1;

  # Horse
  package Horse;
  use Moose;
  with 'Animal';
  sub sound { 'neigh' }
  1;
  ```
  - 此时，我们要调用’父类‘的方法已经不能使用SUPER，因为@ISA中并没有存放父类
    - after，会在’子类‘的方法前调用父类的方法
      ```perl
      with "Animal"
      after 'speak' => sub {
          print "[but you can barely hear it!]\n";
      }
      ```
    - before 在最后再调用父类的方法
    - around 能够自定义位置 第一个参数就是role的原方法引用
      ```perl
      has 'name' => ( is => 'rw' );
      around 'name' => sub {
          my $next = shift;
          my $self = shift;
          blessed $self ? $self->next(@_) : "an unnamed $self"
      }
      ```
  - 快速定义多个属性的方法
    ```perl
    has $_ ( is => 'rw', default => 0)
        foreach qw(wins places shows losses);
    ```


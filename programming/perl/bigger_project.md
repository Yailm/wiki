- 将程序的部分代码写到单独的文件中，后缀为.pm
- 使用require
      require 'DropAuchor.pm';
    - 文件中任何的语法错误都会导致程序终止，所以die语句是不需要的
    - 文件的最后一条语句必须返回真值，正因为如此，所以require要导入的文件内容后面都会有个神秘的`1;`，这已经形成一种传统
- 为了防止导入文件时，程序的某些命名冲突，可以使用package命令
      package Navigation;
    - package命令必须放在pm文件的开头
    - package名字必须开头大写，而且不能和已经存在CPAN或核心的名字重复
    - package名字可以设定成多层Minnow::Navigation，这是可以的
    - 这样相当于在接下来命名中，都会默认在前面加上Navigation::在名字前面
    - 仅仅是名字扩展了而已，@$%&等标识符还是在最前面的
    - 除非名字已经包含一个或多个双分号，不然几乎所有变量，数组，哈希，子程序和文件句柄都是加上前缀的
    - 每个模块文件如果没有定义package，默认都是在main包下，可以用来标识模块内的东西和外来的，这样即使外来的函数冲突也没事(use中包含了import函数，所以其不能这样用)
          sub main::turn_toward_heading {  }
    - 当其他程序require该文件后，要调用其中的变量或子程序必须写上完整的名字（其它的等下说）
          Navigation::func();
          $Navigation::variable;
    - 后面的package语句会覆盖前面的package，除非在另一个语句块中，这时会保存前面的，直到语句块退出后再还原
      ```perl
      package Navigation;
      {
          package main;
          sub add {}    # main::add
      }
      sub aaa {}        # Navigation::add
      ```
    - 还有些名字始终在包main中，无论当前包的名称
          ARGV ARGVOUT ENV INC STDERR STDIN STDOUT
          还有一些特殊变量$_ $1等等
    - 被my修饰的变量不会添加包名前缀，因为出了文件或语句块就不可用了
    - 也可以选择使用package语句块
      ```perl
      package Packagename {
      }   # 相当于将package写在括号里，只是比较好看而已，同时也可以加上版本
      package Packagename 0.15 {
      }
      ```


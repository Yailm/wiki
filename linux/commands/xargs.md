
xargs
------------------------------------

由于很多命令不支持管道来传递参数，而且参数的字节大小还要限制

### 选项解释

* [-0] 当stdin含有特殊元字符时候，将其当成一般字符，像/、空格等

* [-a file] 从文件中读入作为stdin
  ```
  $ cat test
  $!/bin/sh
  echo "hello world\n"
  $ xargs -a test echo
  #!/bin/sh echo hello world\n
  ```
* [-e flag] 注意有时候可能是-E，flag必须是一个以空格分隔的标志，当xargs分析到含有flag这个标志的时候就停止
  ```
  $ cat txt
  /bin tao shou kun
  $ cat txt | xargs -E 'shou' echo
  /bin tao
  ```
* [-p] 当每次执行一个argument的时候询问一次用户
  ```
  $ cat test | xargs -p echo
  echo /bin tao shou kun ?..y
  ```
* [-n num] 后面加次数，表示命令在执行的时候一次用的argument的个数，默认是用所有
  ```
  $ cat test | xargs -n1 echo
  /bin
  tao
  shou
  kun
  ```
* [-t] 表示先打印命令，然后再执行
  ```
  $ cat test | xargs -t echo
  echo /bin tao shou kun
  /bin tao shou kun
  ```
* [-i] 或是 [-I] 将xargs的每项名称，一般是一行一行赋值给{}
  ```
  $ ls | xargs -t -i mv {}{,.bak}
  ```
* [-r] no-run-if-empty 当xargs的输入为空的时候则停止xargs，不去执行

* [-s num] 命令行的最好字符数，指的是xargs后面那个命令的最大命令行字符数(包含空格)
  ```
  $ ls | xargs -i -x -s 14 echo "{}"
  test
  tmp
  xargs: arguments line too long
  ```
* [-L num] 每个命令行最多可以有的num行非空格输入
* [-d delim] 分隔符，默认的xargs的分隔符是回车，argument的分隔符是空格，这里修改的是xargs的分隔符
  ```
  $ cat test | xargs -i -p echo {}
  echo /bin tao shou kun ?...y
  $ cat test | xargs -i -p -d " " echo {}
  echo /bin ?...y
  echo tao ?...
  ```
* [-x] exit的意思，主要配合-s使用
* [-P] 最多同时运行的进程数，默认是1，如果设置成0，xargs将同时运行尽可能多的进程

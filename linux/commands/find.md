
find
------------------------------------

### 命令格式

    find pathname -options [-print -exec -ok ...]

### 命令功能

用于在文件树中查找文件，并作出相应的处理

### 命令参数

* pathname: find命令所查找的目录路径
* -print:   find命令所匹配的文件输出到标准输出
* -print0:  find命令使用`'\0'`作为文件的定界符，这样就能搜索包含空格的文件
* [-exec](#exec):    find命令对匹配的文件执行该参数所给出的shell命令，命令形式为`command {} \;`
* -ok:      和-exec的作用相同，在执行每个命令之前，都会给出提示，让用户来确定是否执行

### 命令选项

* -name 按照文件名(不包括目录名)
* -path 按照路径
* -perm 按照文件权限
* -prune 使用这选项可以使find命令不在当前指定的目录下查找，如果同时使用-depth选项，那么-prune将被忽略
* -user 按照文件属主
* -group 按照文件所属的主
* -mtime -n +n 按照文件的更改时间
* -nogroup 查找无有效所属组的文件
* -nouser 查找无有效属主的文件
* -newer file1 ! file2 查找更改时间比file1新但比file2旧的文件
* -type 查找某一类型的文件
  * b - 块设备文件
  * d - 目录
  * c - 字符设备文件
  * p - 管道文件
  * l - 符号链接文件
  * f - 普通文件
* -size n 查找文件长度为n块的文件，带c时表示文件长度以字节计
* -depth find命令会优先搜索最深层的文件
* -fstype 查找位于某一文件系统中的文件
* -mount 不跨越文件系统mount点
* -follow 跟随符号链接
* -cpio 对匹配的文件使用cpio命令[?]
* -amin n 查找系统最后n分钟访问的文件
* -atime n 查找系统最后n*24小时访问的文件
* -cmin n 被更改文件文件状态
* -ctime n
* -mmin n 被改变文件数据的文件
* -mtime n
* -regex 使用正则表达式，匹配路径
* -iregex 忽略大小写的正则
* -maxdepth n 指定搜索的最大深度

### 使用实例

> 查找指定时间内修改过的文件

      find -atime -2
      # 超过48小时内修改过的文件

> 根据关键字查找

      find . -name "*.log"
      # 在当前目录查找以log为后缀的文件

> 按照目录或文件的权限来查找文件

      find /opt/soft/test/ -perm 777

> 按类型查找

      find . -type f -name "*.log"

> 按大小查找文件

      find . -size +1000c -print
      find . -type f -size -2k

> 查找txt和pdf文件

      find . \( -name "*.txt" -o -name "*.pdf" \) -print
      find . -regex ".*\(\.txt|\.pdf\)$"

> 否定参数，查找所有非txt文本

      find . ! -name "*.txt" -print

> 指定搜索的深度，打印当前目录的文件

      find . -maxdepth 1 -type f

> 比file.txt修改时间更近的所有文件

      find . -type f -newer file.txt -print

### exec

    find . -type f -exec ls -l {} \;
    find . -type -f -mtime +14 -exec rm {} \;
    find . -name "*.log" -mtime +5 -ok rm {} \;
    find /etc -name "passwd*" -exec grep "root" {} \;
    find . -name "*.log" -exec mv {} .. \;
    find . -name "*.log" -exec cp {} test3 \;

### xargs

在使用find命令的-exec选项处理匹配到的文件时，find命令将所有匹配到的文件一起传递给exec执行。但有些系统对能够传递给exec的命令长度有限制，会给出参数列太长或参数列溢出。这就是xargs的用处所在，xargs每次只获取一部分文件而不是全部

#### xargs实例

> 查找系统中的每一个普通文件，然后测试它们分别属于哪类文件

      find . -type f -print | xargs file

> 在整个系统查找内存信息转储文件(core dump)，然后把结果保存到/tmp/core.log文件中

      find / -name "core" -print | xargs echo "" > /tmp/core.log

> 在当前目录下查找所有用户具有读、写和执行权限的文件，并收回相应的写权限

      find . -perm -7 -print | xargs chmod o-w


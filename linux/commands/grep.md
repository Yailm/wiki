
grep
------------------------------------

### 命令格式

grep [options] match_pattern file

### 常用参数

* -o 只输出匹配的文本行
* -b 打印匹配位置的字节或字符偏移，通常配合-o使用
* -v 只输出没有匹配的文本行
* -c 统计文件中包含文本的**行**数
* -n 打印匹配的行号
* -i 搜索时忽略大小写
* -l 只打印文件名
* -q 不输出，配合$0判断结果
* --include
* --exclude

### 使用实例

> 基础使用

      grep pattern file
      grep pattern file1 file2 ...

> 拓展正则表达式

      grep -E '[a-z]+' file
      egrep '[a-z]+' file

> 只输出匹配部分

      echo this is a line. | egrep -o "[a-z]+\."
      # line.

> 取补集

      grep -v pattern file

> 在多级目录中对文本递归搜索

      grep "class" . -R -n

> 匹配多个模式

      grep -e "class" -e "virtual" file

> 综合应用

      cat LOG.* | tr a-z A-Z | grep " FROM " | grep "WHERE"

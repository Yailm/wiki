
grep
------------------------------------

### 命令格式

grep [options] match_pattern file

### 常用参数

* -o 只输出匹配的文本行
* -v 只输出没有匹配的文本行
* -c 统计文件中包含文本的次数
* -n 打印匹配的行号
* -i 搜索时忽略大小写
* -l 只打印文件名

### 使用实例

> 在多级目录中对文本递归搜索

      grep "class" . -R -n

> 匹配多个模式

      grep -e "class" -e "virtual" file

> 综合应用

      cat LOG.* | tr a-z A-Z | grep " FROM " | grep "WHERE"

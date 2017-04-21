
tr
------------------------------------

translate or delete characters

### 参数说明

* -c 将没有出现在搜索字符串的字符替换，set2是可选的

        tr -c '[set1]' '[set2]'

* -d 将搜索字符串没有出现在替换字符串的字符删除

        tr -d '[set]'

* -s 将搜索字符串中多个连续重复的字符替换成一个

        tr -s '[set]'

* -t 执行常规替换前，先将搜索字符串长度裁减和替换字符一样

### 使用实例

> 删除输入的数字

      echo "Hello 123 World 456" | tr -d '0-9'

> 删除非数字

      echo hello r34 | tr -d -c '0-9'

> 将文件列表中的数字列表相加

      cat sum.txt | echo $[ $(tr '\n' '+' 0)]

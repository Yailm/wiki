
rename
------------------------------------

    rename [options] <expression> <replacement> <file>

### 使用实例

> 将JPG更名为jpg

      rename *.JPG *.jpg

> 文件名中空格替换成字符"_"

      rename 's/ /_/g' *

> 转换文件名中的大小写

      rename 'y/A-Z/a-z/' *
      rename 'y/a-z/A-Z/' *

> 将所有的mp3文件移入指定目录

      find path -type f -name "*.mp3" -exec mv {} target_dir \;

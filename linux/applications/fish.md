
fish-shell
------------------------------------

#### IO redirection

- to write standard error to a file, write `^DESTINATION`
- to append standard error to a file, write `^^DESTINATION`
- 同样保留了原来的, to redirect output of FD N, write N>DESTIANTION

DESTINATION can be one of the following:
- A filename
- &符号跟上数字
- &-, the file descriptor will be close?

#### Autosuggestions

- to accept the autosuggestion (replacing the command line contents), press right arrow or ctrl-F
- to accept the first suggested word, alt-left or alt-F

#### Command line editor

switch with fish_vi_key_bindings and back with fish_default_key_bindings.

- `tab` 补全当前命令. `shift tab` 补全当前命令并进行搜索
- `home` or `ctrl-a` 将光标移动到行首
- `end` or `ctrl-e` 将光标移动到行末，如果有建议可用，则接受补全
- 方向键左右, `ctrl-b` and `ctrl-f` 左右移动一个字符，有建议可用就接受补全
- `alt-左右键` 左右移动一个单词，如果行为空，则在目录历史中移动。若已经在行末，则接受建议的第一个单词（还有`alt-f`)
- `方向键上下` 在命令历史中搜索已经输入的字符
- `alt-方向键上下` 在命令历史中搜索含有光标下的标记
- `ctrl-c` 删除整行
- `ctrl-d` 删除光标右边的字符，如果行为空，则退出
- `ctrl-k` 把光标到行末的内容移动到killring
- `ctrl-u` 把光标到行首的内容移动到killring
- `ctrl-l` 清屏
- `ctrl-w` 将路径的最后一部分（直到/）移动到killring
- `alt-d` 将下一个单词移动到killring
- `alt-w` 显示光标下命令的简短信息
- `alt-l` 显示当前目录下的内容，如果光标在参数目录上，该目录内容则会被显示
- `alt-p` 添加 | less
- `alt-c` 单词开头大写
- `alt-u` 光标下的单词全部大写
- `alt-h` or `F1` 显示帮助手册
- `ctrl-t` 交换最后两个字符
- `alt-t` 交换最后两个单词
- `ctrl-y` 从killring粘贴内容
- `alt-y` 在使用`ctrl-y`后，使用这命令切换到前一内容

#### Multiline editing

- 在一个未关闭的块中按下enter
- 按下`alt-enter`
- 输入反斜线 \

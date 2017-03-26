### vim-surround

vim插件快速添加包围文字的符号

#### 基本使用

An asterisk (*) is used to denote the cursor position

| Old text              | Command | New text                  |
|-----------------------|---------|---------------------------|
| "Hello *world!"       | ds"     | Hello world!              |
| [123+4*56]/2          | cs])    | (123+456)/2               |
| "Look ma, I'm *HTML!" | cs"<q>  | <q>Look ma, I'm HTML!</q> |
| if *x>3 {             | ysW(    | if ( x> 3) {              |
| my $str = *whee!;     | vllllS' | my $str = 'whee!';        |


- normal mode
  - `ds`: 删除包围
  - `cs`: 更改包围
  - `ys`: 需先输入一个vim motion(w, b, e, p)或是对象(iw, is, as, ip之类的); 为了方便，`yss`可以用来操作在当前行上
  - `yS`; 会额外将选择的内容放在独立的行上并缩进
- visual mode
  - `S`: 对选中的内容包围
  - `gS`:如yS，会自动缩进

#### 开闭符号

如果添加的包围符号是开符号，则还会自动插入一个空格，闭符号则不会

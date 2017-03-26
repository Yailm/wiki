### Tabular

  vim插件用来标齐文本

#### example

    " 通过逗号来分隔区域，默认是右对齐空一格"
    :Tabularize /,

    " 右对齐，没有空格"
    :Tabularize /,/r0

    “ 多个模式
    :Tabularize /,/r1c0l1

#### 详细

在命令最后的模式，是循环利用在每个划分的区域里面的

如:
    abc,def,ghi
    a,b,c

应用`:Tabularize /,/r1c0`，abc,def,ghi首先被分成5个区域，'abc' ',' 'def' ',' 'ghi'，而模式只有两种，则轮换使用

最后的结果是，c0模式都被','这个区域所使用了

    abc ,def ,ghi
      a ,  b ,  c


shell
------------------------------------

- 获得变量的长度

        length=${#var}

- 识别当前使用的shell

        echo $SHELL
        echo $0

- 检查是否为root用户

        if [ $UID -ne 0 ]; then
            echo Non root user, Please run as root
        else
            echo Root user
        fi
        # root用户的UID是0

- 数学计算

  变量默认被赋值为字符串

  - let命令

        let result=no1+no2
        let no1++
        let no2--
        let no1+=6

  - []操作符

        result=$[ no1 + no2 ]
        result=$[ $no1+5 ]  #也可以加上$前缀

  - (())操作符，同上

        result=$(( no1 + 5 ))

  - expr

        result=`expr 3 + 5`  #数字和符号必须要有空格
        result=`expr $no1 + 5`

    目前以上都不支持浮点运算

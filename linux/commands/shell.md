
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

- 环境变量添加

        prepend() { [ -d "$2" ] && eval $1=\"$2\$\{$1:+':'\$$1\}\" && export $1; }
        # 其中
        ${parameter:+expression}
        # parameter有值不为空，则使用expression的值

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
        result=$(expr $no1 + 5)

    目前以上都不支持浮点运算

- 创建一个文件描述符3并打开读取文件

        exec 3<input.txt
        cat <&3

        exec 4>output.txt
        echo test >&4
        exec 5>>append.txt
        echo test >&5

- 定义数组

        array=(1 2 3 4 5 6)
        echo $array; echo ${array[0]}   # 相等
        array[3]='777'
        echo ${array[*]}    # 使用通配符展开，打印所有值
        echo ${array[@]}    # 正常打印所有值
        echo ${#array[*]}   # 数组长度
        echo ${#array[@]}   # 同上

- 取消转义

        \command
        # 如果有人alias ls='./script.sh'，\ls就只执行原来的ls

- tput和stty

        tput sc # 存放光标当前位置
        tput rc # 恢复光标位置
        tput ed # 删除光标后面的字符

- 定义函数

        function fname()
        {
            ...
        }   # 或
        fname()
        {
            ...
        }
        # 执行
        fname arg1 arg2...

- $@为列表方式打印所有参数

- $*类似$@，但参数被当成单个实体

- ()操作符生成一个子shell，不会对当前的shell产生任何影响，如cd

        output=$(ls | cat -n)
        (cd /tmp; ls)
        pwd

- true是作为/bin/true文件执行实现的，而每一次循环都会产生这样一个新进程
- `:`是shell内建的命令，它总能产生返回为0的退出码

        repeat() { while :; do $@ && return; done } # 比while true快多了

- IFS的默认值是空白字符(\t,\n, )，作为定界符

        items="name,sex,rollno,location"
        oldIFS=$IFS
        IFS=,
        for item in $items;
        do
            echo Item: $item
        done
        IFS=$oldIFS

- 生成列表{1..50}

        echo {1..50}
        echo {a..z}

- 循环

    - for循环

            for i in {a..z}; do actions; done
            for i in $list; do action; done
            for ((i=0;i<10;i++)) { actions; }   # c语言形式

    - while循环

            while condition;
            do
                commands;
            done

    - until循环

            until condition;
            do
                commands;
            done

- if条件与判断

        if condition;
        then
            commands;
        fi

        if condition;
        then
            commands;
        elif
            commands;
        fi

    - 算术比较

            [ $var -eq 0 ]
            # -eq -gt -lt -ge 大于等于 -le 小于等于 -ne
            # -o 逻辑或 -a 逻辑与

    - 文件系统

            [ -f $var ] # 包含正常的文件路径或文件名
            [ -x $var ] # 可执行
            [ -d $var ] # 目录
            [ -e $var ] # 存在
            [ -c $var ] # 字符设备文件
            [ -b $var ] # 块设备文件
            [ -w $var ] # 可写
            [ -r $var ] # 可读
            [ -L $var ] # 符号链接

    - 字符串比较，最好使用双中括号

            [[ $str1 == $str2 ]]
            [[ $str1 != $str2 ]]
            [[ $str1 > $str2 ]]
            [[ $str1 < $str2 ]]
            [[ -z $str1 ]]
            [[ -n $str1 ]]

    test可以取代前面的中括号

- %操作符

    ${VAR%.*}的含义: 从$VAR删除位于%右侧的通配符(.*)所匹配的字符串，通配符从右向左
    %属于非贪婪操作，从右到左找到匹配的最短结果，%%是贪婪的，会去寻找最长的结果

        VAR=hack.fun.book.txt
        echo ${VAR%.*}      # hack.fun.book
        echo ${VAR%%.*}     # hack

- #操作符

    `#`操作符类似上面的，不过它的匹配是从左到右

        VAR=hack.fun.book.txt
        echo ${VAR#*.}      # fun.book.txt
        echo ${VAR##*.}     # txt

- diff

        diff -u file1 file2 [> file1.patch]
        patch -p1 file1 < file1.patch
        diff -Naur dir1 dir2

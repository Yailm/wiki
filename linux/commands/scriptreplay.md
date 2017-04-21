
scriptreplay
------------------------------------

script和scriptreplay命令可以把终端回话记录到一个文件

    $ script -t 2> timing.log -a output.session
    Script started, file is output.session
    
    ...execute commands
    $ exit
    Script started, file is output.session


回放

    $ scriptreplay timing.log output.session

witness the miracle

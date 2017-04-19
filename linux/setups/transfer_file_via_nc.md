
transfer file via netcat
------------------------------------

computer A

    $ nc -l -p 12345 > receive_file

computer B

    $ nc server_ip 12345 < send_file

or do transfer of directories and files

    $ nc -l -p 12345 | tar xz -C /path/to/root/of/tree
    $ tar czf - ./directory_tree_to_xfer | nc server_ip 12345

refer to [nakkaya.com](https://nakkaya.com/2009/04/15/using-netcat-for-file-transfers/)

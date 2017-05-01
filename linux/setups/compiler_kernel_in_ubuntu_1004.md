
compiler kernel on ubuntu 10.04
------------------------------------

    # apt-get install build-essential kernel-package libncurses5-dev
    # apt-get install linux-source
    # cd /usr/src/
    # tar jxvf linux-source[TAB]
    # cd linu-source[TAB]
    # cp ../linux-headers[TAB]/.config .
    # make menuconfig  # load, save

    # make
    # make install
    # make modules
    # make modules_install
    # mkinitramfs -o /boot/initrd.img-(KERNEL_NAME)

    # vi /boot/grub/grub.cfg

Refer to [cnblog](http://www.cnblogs.com/jianyungsun/archive/2011/03/24/1993723.html)


initrd.img with lvm support
------------------------------------

part of /boot/grub/grub.cfg

    menuentry 'Ubuntu, with Linux 2.6.35-22-generic' --class ubuntu --class gnu-linux --class gnu --class os {
        recordfail
        insmod part_msdos
        insmod ext2
        set root='(hd0,msdos3)'
        search --no-floppy --fs-uuid --set $UUID_OF_BOOT_FILESYSTEM
        linux   /vmlinuz-2.6.35-22-generic root=/dev/mapper/$LVM_VOLUME_GROUP-root ro   quiet splash
        initrd  /initrd.img-2.6.35-22-generic
    }

It shows that root is on the lvm volume which makes booting failed and dropping into a busybox

1. bring up the volume group (if `-a y` does not work try `-a yes`)

        vgchang -a y

2. get the root LV, /boot, and /dev mounted under the seperated tree

        mkdir /newroot
        mount /dev/yourVG/rootLV /newroot
        mount /dev/yourBoot /newroot/boot
        mount -o bind /dev /newroot/dev

3. copy the needed packages into /newroot tree (ubuntu)

        apt-get install --reinstall lvm2    # make sure you have lvm2.deb
        cp /var/cache/apt/archives/*.deb /new/tmp/

4. chroot the new tree

        chroot /newroot
        cd /tmp
        dpkg -i *.deb

5. make sure kernel that you need is regenerated

        update-initramfs -u -k all

Refer to [Fixing unbootable installation on LVM root from Desktop LiveCD](https://askubuntu.com/questions/26886/fixing-unbootable-installation-on-lvm-root-from-desktop-livecd)

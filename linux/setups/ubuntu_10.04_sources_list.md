
ubuntu 10.04 sources.list
------------------------------------

    $ sudo vi /etc/apt/sources.list

add following lines

    deb http://old-releases.ubuntu.com/ubuntu lucid main restricted universe multiverse   
    deb http://old-releases.ubuntu.com/ubuntu lucid-security main restricted universe multiverse   
    deb http://old-releases.ubuntu.com/ubuntu lucid-updates main restricted universe multiverse   
    deb http://old-releases.ubuntu.com/ubuntu lucid-proposed main restricted universe multiverse   
    deb http://old-releases.ubuntu.com/ubuntu lucid-backports main restricted universe multiverse   
    deb-src http://old-releases.ubuntu.com/ubuntu lucid main restricted universe multiverse   
    deb-src http://old-releases.ubuntu.com/ubuntu lucid-security main restricted universe multiverse   
    deb-src http://old-releases.ubuntu.com/ubuntu lucid-updates main restricted universe multiverse   
    deb-src http://old-releases.ubuntu.com/ubuntu lucid-proposed main restricted universe multiverse   
    deb-src http://old-releases.ubuntu.com/ubuntu lucid-backports main restricted universe multiverse

    $ sudo apt-get update

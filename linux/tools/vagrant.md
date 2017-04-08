
vagrant
------------------------------------

### 介绍

Vagrant是为了方便的实现虚拟化环境开发而设计的，基于virtualbox等虚拟机管理软件的接口，提供一个可配置、轻量级的虚拟开发环境

### 配置

当我们安装好vagrant后，该开始考虑在VM上面安装操作系统了。
一个打包好的操作系统在vagrant中被称为box，[vagrantbox.es](http://www.vagrantbox.es/)上有预配置好的box，只需要下载安装就行了

#### 添加box

    vagrant box add {title} 远端box地址或者本地的box路径

{title}是box的标识名称，可以任意替换，默认是base，下面一些例子

    vagrant box add base http://files.vagrantup.com/lucid64.box
    vagrant box add base https://dl.dropbox.com/u/7225008/Vagrant/CentOS-6.3-x86_64-minimal.box
    vagrant box add base CentOS-6.3-x86_64-minimal.box
    vagrant box add "CentOS 6.3 x86_64 minimal" CentOS-6.3-x86_64-minimal.box

通过vagrant box add 这样的方式安装远程的box可能很慢，建议先下载

#### 初始化

    vagrant init {title}

如果box的标识是base，则可以省略，会在当前目录生成Vagrantfile文件，里面有虚拟机的配置信息

#### 启动

    vagrant up

#### 连接到虚拟机

    vagrant ssh

PS. windows需要使用putty或xshell4等来连接

  > 主机地址：127.0.0.1
  > 端口: 2222
  > 用户名: vagrant
  > 密码: vagrant

### vagrantfile配置

1. box配置

        config.vm.box = "base"

    设定要启动的box，
    VirtualBox提供了VBoxManage这个命令行工具，可以让我们设定VM，用modifyvm这个命令让我们可以设定VM的名称和内存大小等等，这里说的名称指的是在VirtualBox中显示的名称
    我们也可以在Vagrantfile中进行设定

        config.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--name", "astaxie", "--memory", "512"]
        end

2. 网络设置

    Vagrant有两种方式来进行网络连接，一种是host-only(主机模式)，意思是主机和虚拟机之间的网络互访，而不是虚拟机访问internet的技术，也就是只有你一個人自High，其他人访问不到你的虚拟机。另一种是Bridge(桥接模式)，该模式下的VM就像是局域网中的一台独立的主机，也就是说需要VM到你的路由器要IP，这样的话局域网里面其他机器就可以访问它了，一般我们设置虚拟机都是自high为主，所以我们的设置一般如下：

        config.vm.network :private_network, ip: "11.11.11.11"

    这里我们虚拟机设置为hostonly，并且指定了一个IP，IP的话建议最好不要用192.168..这个网段，因为很有可能和你局域网里面的其它机器IP冲突，所以最好使用类似11.11..这样的IP地址。

3. hostname

        config.vm.hostname = "slave1"

    设置hostname非常重要，因为当我们有很多台虚拟服务器的时候，都是依靠hostname來做识别的，例如Puppet或是Chef

4. 同步目录

        config.vm.synced_folder  "/Users/astaxie/data", "/vagrant_data"

    第一个参数是主机的目录，第二个参数是虚拟机挂载的目录

5. 端口转发

        config.vm.network :forwarded_port, guest: 80, host: 8080

    把对host机器上8080端口的访问请求forward到虚拟机的80端口的服务上

如果VM已经启动了，需要使用`vagrant reload`来重启并更新配置

### 其他操作

- vagrant box list  

        $ vagrant box list
        base (virtualbox)

- vagrant box remove

        $ vagrant box remove base virtualbox
        Removing box 'base' with provider 'virtualbox'...

- vagrant destroy

  停止当前这个运行的虚拟机并销毁所有创建的资源

- vagrant halt

  关机

- vagrant package

  打包命令，可以把当前的运行的虚拟机环境进行打包

- vagrant plugin

  安装和卸载插件

- vagrant provision

  通常情况下Box只做最基本的设置，而不是设置好所有的环境，因此Vagrant通常使用Chef或者Puppet来做进一步的环境搭建。那么Chef或者Puppet称为provisioning，而该命令就是指定开启相应的provisioning。按照Vagrant作者的说法，所谓的provisioning就是"The problem of installing software on a booted system"的意思。除了Chef和Puppet这些主流的配置管理工具之外，我们还可以使用Shell来编写安装脚本。

- vagrant reload
- vagrant suspend
- vagrant resume
- vagrant status
- vagrant ssh-config

  输出用于ssh连接的一些信息

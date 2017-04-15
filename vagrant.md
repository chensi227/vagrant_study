#vagrant 的基本使用和常见问题

###常见命令

#####添加box
- vagrant box [boxname] add [box位置]  例: vagrant box add ubuntu-14.04 ubuntu-14.04-amd64.box
#####查看box列表
- vagrant box list
#####初始化
- vagrant init [boxname]   初始化会生成配置文件 vagrantFiles   *不同的box需要在不同的文件下初始化*
#####启动
- vagrant up *需要在生成配置文件的文件夹下启动*
#####连接ssh
- vagrant ssh *推荐git软件   xshell有时候会连接不上*
- 退出ssh 可用exit命令
#####删除对应的box
 - vagrant box remove [boxname]
#####停止当前正在运行的虚拟机并销毁所有创建的资源
-  vagrant destroy
#####关机
- vagrant halt
#####重启
- vagrant reload  *重新启动虚拟机，主要用于重新载入配置文件*  进入ssh后用exit退出
#####打包命令，可以把当前的运行的虚拟机环境进行打包
- vagrant package
#####获取当前虚拟机的状态
- vagrant status
#####挂起当前的虚拟机
- vagrant suspend
#####输出用于ssh连接的一些信息
- vagrant ssh-config
#####挂起当前的虚拟机
- vagrant suspend
####用于安装卸载插件
- vagrant provision
#####恢复前面被挂起的状态
- vagrant resume

###常见问题
#####设置网络配置
######端口映射(Forwarded port)
- 使用端口映射时,宿主机器的端口需要未被占用。
* 使用小技巧 配置环境时需要多个端口的转发
>比如 nginx,redis,mysql 可配置多个
___
    config.vm.network "forwarded_port", guest: 80, host: 8888 ,id: 'nginx'
____
    config.vm.network "forwarded_port", guest: 8888, host: 8889 ,id: 'apache'

######私有网络（Private network）
*使用私有网络时最好设置一个网络内没有的网络 如果启动出错请记住设置虚拟机的网络
*管理->全局设定->网络->仅主机（host-only）网络->随便修改一个虚拟网卡，Ip为：192.168.10.10 ip为你自己给私有网络设置的网络*

###vagrant使用vmware
1.安装vagrant支持vmware的插件
vmware-fusion：vagrant plugin install vagrant-vmware-fusion
vmware-workstation：vagrant plugin install vagrant-vmware-workstation

2.在你的vagrantfile里定义使用vmware
Vagrant.configure("2") do |config|
 # ... other config up here

 # Prefer VMware Fusion before VirtualBox
 config.vm.provider "vmware_fusion"
 config.vm.provider "virtualbox"
end

3.使用--provider=vmware_fusion参数跟在vagrant命令后面去执行各种命令，比如vagrant up --provider=vmware_fusion



###其他
- 不同的版本agrant和virtualBox组合可能会存在一些启动上面的问题。推荐使用下面这个组合。
____
    vagrant1.9.1+virtualBox4.3.12
    vagrant1.9.2+virtualBox5.0.20


### 模拟打造多机器的分布式系统
前面这些单主机单虚拟机主要是用来自己做开发机，从这部分开始的内容主要将向大家介绍如何在单机上通过虚拟机来打造分布式造集群系统。这种多机器模式特别适合以下几种人：

1. 快速建立产品网络的多机器环境，例如web服务器、db服务器
1. 建立一个分布式系统，学习他们是如何交互的
1. 测试API和其他组件的通信
1. 容灾模拟，网络断网、机器死机、连接超时等情况

Vagrant支持单机模拟多台机器，而且支持一个配置文件Vagrntfile就可以跑分布式系统。

现在我们来建立多台VM跑起來，並且让他们之间能够相通信，假设一台是应用服务器、一台是DB服务器，那么这个结构在Vagrant中非常简单，其实和单台的配置差不多，你只需要通过`config.vm.define`来定义不同的角色就可以了，现在我们打开配置文件进行如下设置：

	Vagrant.configure("2") do |config|
	  config.vm.define :web do |web|
	    web.vm.provider "virtualbox" do |v|
	          v.customize ["modifyvm", :id, "--name", "web", "--memory", "512"]
	    end
	    web.vm.box = "base"
	    web.vm.hostname = "web"
	    web.vm.network :private_network, ip: "11.11.1.1"
	  end

	  config.vm.define :db do |db|
	    db.vm.provider "virtualbox" do |v|
	          v.customize ["modifyvm", :id, "--name", "db", "--memory", "512"]
	    end
	    db.vm.box = "base"
	    db.vm.hostname = "db"
	    db.vm.network :private_network, ip: "11.11.1.2"
	  end
	end

这里的设置和前面我们单机设置配置类似，只是我们使用了`:web`以及`:db`分別做了两个VM的设置，并且给每个VM设置了不同的`hostname`和IP，设置好之后再使用`vagrant up`将虚拟机跑起来：

	$ vagrant up
	Bringing machine 'web' up with 'virtualbox' provider...
	Bringing machine 'db' up with 'virtualbox' provider...
	[web] Setting the name of the VM...
	[web] Clearing any previously set forwarded ports...
	[web] Creating shared folders metadata...
	[web] Clearing any previously set network interfaces...
	[web] Preparing network interfaces based on configuration...
	[web] Forwarding ports...
	[web] -- 22 => 2222 (adapter 1)
	[web] Running any VM customizations...
	[web] Booting VM...
	[web] Waiting for VM to boot. This can take a few minutes.
	[web] VM booted and ready for use!
	[web] Setting hostname...
	[web] Configuring and enabling network interfaces...
	[web] Mounting shared folders...
	[web] -- /vagrant
	[db] Setting the name of the VM...
	[db] Clearing any previously set forwarded ports...
	[db] Fixed port collision for 22 => 2222. Now on port 2200.
	[db] Creating shared folders metadata...
	[db] Clearing any previously set network interfaces...
	[db] Preparing network interfaces based on configuration...
	[db] Forwarding ports...
	[db] -- 22 => 2200 (adapter 1)
	[db] Running any VM customizations...
	[db] Booting VM...
	[db] Waiting for VM to boot. This can take a few minutes.
	[db] VM booted and ready for use!
	[db] Setting hostname...
	[db] Configuring and enabling network interfaces...
	[db] Mounting shared folders...
	[db] -- /vagrant

看到上面的信息输出后，我们就可以通过`vagrant ssh`登录虚拟机了，但是这次和上次使用的不一样了，这次我们需要指定相应的角色，用来告诉ssh你期望连接的是哪一台：

	$ vagrant ssh web
	vagrant@web:~$

	$ vagrant ssh db
	vagrant@db:~$

是不是很酷！现在接下来我们再来验证一下虚拟机之间的通信，让我们先使用ssh登录web虚拟机，然后在web虚拟机上使用ssh登录db虚拟机(默认密码是`vagrant`)：

	$ vagrant ssh web
	Linux web 2.6.32-38-server #83-Ubuntu SMP Wed Jan 4 11:26:59 UTC 2012 x86_64 GNU/Linux
	Ubuntu 10.04.4 LTS

	Welcome to the Ubuntu Server!
	 * Documentation:  http://www.ubuntu.com/server/doc
	New release 'precise' available.
	Run 'do-release-upgrade' to upgrade to it.

	Welcome to your Vagrant-built virtual machine.
	Last login: Thu Aug  8 18:55:44 2013 from 10.0.2.2
	vagrant@web:~$ ssh 11.11.1.2
	The authenticity of host '11.11.1.2 (11.11.1.2)' can't be established.
	RSA key fingerprint is e7:8f:07:57:69:08:6e:fa:82:bc:1c:f6:53:3f:12:9e.
	Are you sure you want to continue connecting (yes/no)? yes
	Warning: Permanently added '11.11.1.2' (RSA) to the list of known hosts.
	vagrant@11.11.1.2's password:
	Linux db 2.6.32-38-server #83-Ubuntu SMP Wed Jan 4 11:26:59 UTC 2012 x86_64 GNU/Linux
	Ubuntu 10.04.4 LTS

	Welcome to the Ubuntu Server!
	 * Documentation:  http://www.ubuntu.com/server/doc
	New release 'precise' available.
	Run 'do-release-upgrade' to upgrade to it.

	Welcome to your Vagrant-built virtual machine.
	Last login: Thu Aug  8 18:58:50 2013 from 10.0.2.2
	vagrant@db:~$

通过上面的信息我们可以看到虚拟机之间通信是畅通的，所以现在开始你伟大的架构设计吧，你想设计怎么样的架构都可以，唯一限制你的就是你主机的硬件配置了。
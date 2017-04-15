## php7.1.2 安装方法 ##
安装php7.1.2

更新源文件

- sudo apt-get update
- sudo apt-get install -y language-pack-en-base
- locale-gen en_US.UTF-8
- sudo apt-get install software-properties-common
- sudo LC_ALL=en_US.UTF-8 add-apt-repository ppa:ondrej/php
- sudo apt-get update
- sudo apt-get install php7.1
- sudo apt-get install php7.1-dev # 否则phpize怎么用？
- sudo apt-get install php-pear
- sudo pecl install swoole

*********************************************

### 安装mysql5.7 ###


1. 下载.deb包到你的服务器：
wget http://dev.mysql.com/get/mysql-apt-config_0.5.3-1_all.deb
2. 然后使用dpkg命令添加Mysql的源：
sudo dpkg -i mysql-apt-config_0.5.3-1_all.deb
注意在添加源的时候，会叫你选择安装 MySQL 哪个应用，这里选择 Server 即可，再选择 MySQL 5.7 后又会回到选择应用的那个界面，此时选择 Apply 即可。
3. 安装
sudo apt-get update
sudo apt-get install mysql-server
4. 注意
如果你已经通过 ppa 的方式安装了 MySQL 5.6，首先得去掉这个源
sudo apt-add-repository --remove ppa:ondrej/mysql-5.6
如果没有 apt-add-repository 先安装上
sudo apt-get install software-properties-common

#### 也可以 ####
- wget http://dev.mysql.com/get/mysql-apt-config_0.6.0-1_all.deb
- sudo dpkg -i mysql-apt-config_0.6.0-1_all.deb
- sudo apt-get update
- sudo apt-get install mysql-server-5.7  如果不行请用sudo apt-get install mysql-server
- **这样才能通过apt来安装mysql5.7**
- **在安装过程中，会要求输入root的密码。**

### 安装nginx ###
2、添加]Nginx](http://nginx.org/)官方提供的源
$ echo "deb http://nginx.org/packages/ubuntu/ trusty nginx" >> /etc/apt/sources.list
$ echo "deb-src http://nginx.org/packages/ubuntu/ trusty nginx" >> /etc/apt/sources.list


如果你想安装Nginx1.9以上的版本可以在packages后添加/mainline，这是主线版本.
//安装Nginx1.9.*
$ echo "deb http://nginx.org/packages/mainline/ubuntu/ trusty nginx" >> /etc/apt/sources.list
$ echo "deb-src http://nginx.org/packages/mainline/ubuntu/ trusty nginx" >> /etc/apt/sources.list
3、更新源并安装Nginx
$ sudo apt-get update
$ sudo apt-get install nginx
安装Nginx完成后可查看版本号，输入
$ /usr/sbin/nginx -v


配置php
1.配置php.ini
sudo vim /etc/php/7.1/fpm/php.ini
输入/fix_pathinfo搜索，将cgi.fix_pathinfo=1改为cgi.fix_pathinfo=0：
2. 编辑fpm的配置文件： 运行：
sudo vim /etc/php/7.1/fpm/pool.d/www.conf
找到listen = /run/php/php7.1-fpm.sock修改为listen = 127.0.0.1:9000。使用9000端口。

service php7.1-fpm stop

service php7.1-fpm start

sudo service php7.1-fpm restart
sudo service php7.1-cli restart
 sudo service nginx restart
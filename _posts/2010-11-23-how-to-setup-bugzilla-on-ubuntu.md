---
layout: post
title: Ubuntu 学习笔记之bugzilla的安装及配置
tags: bugzilla
categories: 环境搭建
---


Bugzilla 是用於软件缺陷追踪的網絡應用程式，由Mozilla 計劃开发和应用。1998年 ，网景公司 開放其原始碼，後以Mozilla Public License 协议授權。眾多组织应用它作免费软件 和产权软件 的产品缺陷追踪。（来自Wiki百科）



Bugzilla需要一些其他软件才能在ubuntu上跑得欢快，以下是一些必须的：

1.apache2，安装命令 : sudo apt-get install apache2

2.mysqlserver，安装命令: sudo apt-get install mysql-server（安装完成的最后，会让你输入root管理员的帐号密码，记住该密码，此root非用户组中的root，而是mysql数据库的root管理员根帐号）

3.sendmail，安装命令： sudo apt-get install sendmail

4.其他的一些perl模块



apache2，mysqlserver，sendmail的安装很简单，将上面的命令敲进终端，按照提示往下走就可以了。

接下来可以开始bugzilla了，首先请从bugzilia主页（http://www.bugzilla.org/）上下载一个bugzillia安装包。

我下载的版本是一个名为bugzilla-3.6.3.tar.gz的文件。



将该文件解压缩到/var/www/目录下（命令tar -C /var/www/ -xvf bugzilla-3.6.3.tar.gz），解压完毕，该目录下将会有一个名为bugzilla-3.6.3的目录，将该目录改名为bugzilla并切换到/var/www/bugzilla/目录（命令为mv bugzilla-3.6.3 bugzilla & cd /var/www/bugzilla/）。

在bugzilla目录下有一个checksetup.pl的文件，运行该文件（sudo perl checksetup.pl），运行完毕，会告诉你当前bugzilla还差那些perl模块需要安装，并且有提示安装命令。这里我们只需要安装必须的一些包就可以了，可选包可以不用安装。安装这些必须的包的命令一般为：

/usr/bin/perl   install-module.pl --all

这句命令将安装bugzilla所需要的一系列perl模块，基本上运行完毕，安装bugzilla的前期准备工作都做的差不多了。

如果在公司用代理上网的话，/usr/bin/perl   install-module.pl --all这个可能不管用，我的下载资源列表有一安装bugzilla所需要的perl模块的压缩包，可以下载来自行安装perl模块。



其实这个时候bugzilla差不多可以算是安装好一大半了，剩下的就是一些配置的问题了。

第一个配置，mysql的配置。

bugzilla需要用mysql数据库来管理bugs，其默认的数据库名字为bugs，默认的数据库管理员帐号为bugs，默认管理员密码为空，这些都是写在配置文件localconfig中的。所以我们需要添加一个bugs的mysql数据管理员用户，并创建一个名为bugs的数据库来保存bugzilla提交的bugs。

在终端输入： mysql -u root -p （用mysql的root管理员登录mysql，以添加用户bugs），终端会提示输入密码，即之前安装mysql时的最后输入的root密码。

进入mysql界面后，输入： grant all on *.* to bugs@localhost identified by '';flush privileges;（ 别忘最后的‘；'号），这样我们就创建好了mysql的用户bugs，供bugzilla使用。最后创建bugs数据库文件。在mysql界面中输入：CREATE DATABASE bugs; 即可。



mysql数据库配置好了之后，我们就可以顺利安装bugzilla了。



另外还得修改localconfig文件中的一句话：$webservergroup = 'www-data';这里为什么填www-data呢，这是由我们安装好的apache2的环境变数决定的，该变数存在文件/etc/apache2/envvars中，文件的内容如下：

# settings are defined via environment variables and then used in apache2ctl,
# /etc/init.d/apache2, /etc/logrotate.d/apache2, etc.
export APACHE_RUN_USER=www-data 
export APACHE_RUN_GROUP=www-data

这就是我们要填www-data的原因了，另外我们需要修改bugzilla目录的owner和groups。因为apache2环境变数决定是其是用www-data用户组来执行的。修改的命令为：sudo chgrp -R root.www-data bugzilla。



在终端执行下面的命令：sudo perl checksetup.pl ，运行完毕bugzilla将会顺利的安装好，在安装的最后会让你输入bugzilla系统的管理员帐号和密码，这个必须记好了。



在bugzilla顺利安装好之好，我们还得配置好apache2服务噐，我们的bugzilla才能正常工作哈。所以第二个配置，即是配置好bugzilla目录的服务器权限。打开apache2的配置文件httpd.conf（sudo vi /etc/apache2/httpd.conf），在其中添加如下内容：



<Directory "/var/www/bugzilla/">

AddHandler cgi-script .cgi 

Options +Indexs +ExecCGI +FollowSymLinks

DirectoryIndex index.cgi

AllowOverride None

Order allow,deny

Allow from all

</Directory>



保存退出。



之后重启apache2服务器和mysqlserver（运行命令：sudo /etc/init.d/apache2 restart & sudo /etc/init.d/mysql restart ），bugzilla的环境算是基本上搭建好了，接下来打开firefox就可以用了。



在地址一栏输入http://localhost/bugzilla就可以看到bugzilla的首页了，截图如下：
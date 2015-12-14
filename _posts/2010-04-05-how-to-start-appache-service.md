---
layout: post
title: apache服务，或者说httpd服务，如何启动，如何开机启动。
tags: apache
categories: 环境搭建
---


操作系统环境：红帽5，具体如下：
# uname -a 
Linux machine1 2.6.18-164.el5xen #1 SMP Tue Aug 18 15:59:52 EDT 2009 x86_64 x86_64 x86_64 GNU/Linux
# cat /etc/redhat-release 
Red Hat Enterprise Linux Server release 5.4 (Tikanga)

apache，或者说httpd，版本：
# /usr/sbin/httpd -v 
Server version: Apache/2.2.3
Server built:   Jul 15 2009 09:02:25
或者
# /usr/sbin/apachectl -v 
Server version: Apache/2.2.3
Server built:   Jul 15 2009 09:02:25

之前一直在说apache，或者httpd；
其实httpd是服务，apache是个商标；
就像纯净水是产品，而娃哈哈是个品牌；
但是因为apache太有名，似乎说apache，就是在说httpd服务了。
因此，后文只说httpd服务。


/usr/sbin/apachectl其实是个脚本；
/usr/sbin/httpd 才是真正的程序；

下面回答如何启动httpd服务？
脚本启动：
# /usr/sbin/apachectl start 
[root@radius guoq]# ps -ef|grep apache
apache    6680  6679  0 09:49 ?        00:00:00 /usr/sbin/httpd -k start
apache    6681  6679  0 09:49 ?        00:00:00 /usr/sbin/httpd -k start
apache    6682  6679  0 09:49 ?        00:00:00 /usr/sbin/httpd -k start
apache    6683  6679  0 09:49 ?        00:00:00 /usr/sbin/httpd -k start
apache    6684  6679  0 09:49 ?        00:00:00 /usr/sbin/httpd -k start
apache    6685  6679  0 09:49 ?        00:00:00 /usr/sbin/httpd -k start
apache    6686  6679  0 09:49 ?        00:00:00 /usr/sbin/httpd -k start
apache    6687  6679  0 09:49 ?        00:00:00 /usr/sbin/httpd -k start
root      6689  5393  0 09:49 pts/1    00:00:00 grep apache
停止就是# /usr/sbin/apachectl stop；你说对了； 

如果读一下脚本/usr/sbin/apachectl, 就会发现两个小秘密： 
1. 脚本接受参数 start，stop，restart，还有 graceful，graceful－stop；
2. 其实，脚本还是把参数传递给了 /usr/sbin/httpd；

因此，我们可以 
#/usr/sbin/httpd -k start
启动服务； 
#/usr/sbin/httpd -k stop
停止服务；

下面回答如何开机启动？
如果搜索一下
# find / -name "httpd" 
/var/log/httpd
/usr/sbin/httpd
/usr/lib64/httpd
/etc/rc.d/init.d/httpd 
/etc/logrotate.d/httpd
/etc/httpd
/etc/sysconfig/httpd
/home/guoq/osrc/tcl8.4.19/tests/httpd
/opt/soft/httpd-2.2.14/httpd
/opt/soft/httpd-2.2.14/.libs/httpd
/opt/apache2.2.14/bin/httpd

我们会发现apache已经给我们准备好了开机启动脚本，
/etc/rc.d/init.d/httpd

可以检查它是否在开机启动列表： 
# chkconfig --list | grep httpd
httpd           0:关闭  1:关闭  2:关闭  3:关闭  4:关闭  5:关闭  6:关闭

如果需要，可以将它加入开机启动列表： 
#chkconfig --add httpd
或者，从开机列表中删除： 
#chkconfig --del httpd 

在我的系统中，它已经在开机启动列表： 
httpd           0:关闭  1:关闭  2:关闭  3:关闭  4:关闭  5:关闭  6:关闭 
只是它没有被允许开机自动启动

我希望它在当前的运行级别下，自动启动，我最近在学点Java，还有PHP；
# chkconfig --level 5 httpd on 
# chkconfig --list httpd
httpd           0:关闭  1:关闭  2:关闭  3:关闭  4:关闭  5:启用   6:关闭

wait，我怎么知道我的运行级别？
# runlevel 
N 5

全文完。


不知为什么上面的方法，在我的那台ubuntu上跑不起来，于是到网上再找找了

找到下面的方法：

 

   1)添加程序脚本到/etc/init.d目录下 
      sudo  cp /home/cnscn/my_servd  /etc/init.d/
  
   2)添加到启动列表 
      sudo   update-rc.d  my_servd  defaults

  3) 就会产生以下连接: 
       Adding system startup for /etc/init.d/my_servd ...
       /etc/rc0.d/K20my_servd -> ../init.d/my_servd
       /etc/rc1.d/K20my_servd -> ../init.d/my_servd
       /etc/rc6.d/K20my_servd -> ../init.d/my_servd
       /etc/rc2.d/S20my_servd -> ../init.d/my_servd
       /etc/rc3.d/S20my_servd -> ../init.d/my_servd
       /etc/rc4.d/S20my_servd -> ../init.d/my_servd
       /etc/rc5.d/S20my_servd -> ../init.d/my_servd

   4) 指定启动、关闭级别 （20表示一个级别） (注意后面的 . ) 
            sudo update-rc.d  my_servd  start  20   3  4  5  .      在3,4,5级别上启动
            sudo update-rc.d  my_servd  start  20   0 1 2 6 .      在3,4,5级别上关闭
      
      或
            sudo update-rc.d my_servd  start 20 3 4 5 .   stop 20 0 1 2 6 . 

   5) 移除服务 
      sudo update-rc.d  -f  my_servd  remove

 

试了下，好像可以了耶 ！！

 
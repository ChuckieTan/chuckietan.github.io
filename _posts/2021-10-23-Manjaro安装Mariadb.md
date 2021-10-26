---
layout: post
title: Manjaro安装Mariadb
categories: 数据库
keywords: SQL, MySQL, Mariadb, Manjaro, Arch Linux
---

Mariadb是MySQL的一个复刻。由于MySQL被Oracle公司收购，MySQL的一些原始开发者担心MySQL会有开源方面的某些隐患，故领导开发了Mariadb。

如今，Mariadb已经作为许多Linux发行版的默认MySQL实现，在Arch Linux中，MySQL也被从官方软件仓库移入AUR中。

在Arch Linux系的Linux上安装软件是一个很方便的事，安装Mariadb也是一样，因为Arch Linux系的软件仓库是所有Linux发行版中最为庞大的，而且还有由社区进行维护的非官方软件仓库AUR，所以安装大部分软件都只是几条命令的事。

## 安装Mariadb

```bash
yay -S mariadb
# 从官方软件仓库中安装Mariadb
sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
# 在启动mariadb.service之前需要运行此条命令指定相关目录
sudo systemctl enable mariadb
# 设置开机自启动
sudo systemctl start mariadb
# 启动mariadb服务
```

这样，Mariadb已经安装完毕。但之后还得进行一些安全性的配置。

## 安全性的配置

运行`mysql_secure_installation`命令来进行安全性的配置。

`mysql_secure_installation`会执行下面几个设置：

* 给root用户设置密码
* 移除匿名用户
* 禁止root远程登录
* 移除测试数据库
* 重新加载特权表使修改生效

这些安全配置非常重要，在生产环境中安装完MySQL或者Mariadb一定要执行一遍`mysql_secure_installation`来保证安全。具体操作如下：

```
➜  ~ sudo mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
havent set the root password yet, you should just press enter here.

Enter current password for root (enter for none): # 设置root密码，初次运行直接回车
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] 
Enabled successfully!
Reloading privilege tables..
 ... Success!


You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] # 设置root密码
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] # 移除匿名用户，建议移除
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] # 禁止root远程登录
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] # 移除测试数据库
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] #重新加载特权表
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

至此，Mariadb就安装完成了。

---
title: 在CentOS上安装MySQL
date: 2016-06-12 10:46:00
tags: Linux
comments: false
---
## 背景
主要就是为了有一个地方能够存储数据，来提供一些服务，当然包括这个网站的数据服务。因为买的是阿里云的ECS，又没钱买RDS服务，所以干脆把网站和数据库放在一起了。

## 通过yum安装
1. 查看已安装的MySQL信息
```
# yum list installed | grep mysql
```

2. 如果有，可以卸载
```
# yum -y remove mysql-libs.x86_64
```

3. 安装MySQL
```
# yum -y install mysql-server mysql mysql-devel
```

4. MySQL的几个配置文件和命令
```
配置文件位置: /etc/my.cnf
MySQL数据库文件存放位置: /var/lib/mysql/
启动MySQL: service mysqld start
停止MySQL: service mysqld stop
重起MySQL: service mysqld restart
设置开机启动: chkconfig mysqld on
查看是否开机启动（2到5为on则正常）: chkconfig --list | grep mysqld
```

## 配置MySQL
首先第一次登陆MySQL，使用root账户，不需要密码
```
# mysql -u root
```

修改root的账户密码，以及删除匿名用户
```
mysql>select user,host,password from mysql.user;
mysql>set password for root@localhost=password('你的root密码');
mysql>update mysql.user set host='%' where user='root' and host='localhost' // 为了后面可以使用root在其他主机连接MySQL服务
mysql>delete from mysql.user where user=''; // 删除匿名用户
mysql>GRANT ALL PRIVILEGES ON *.* TO ‘myuser’@'%’ IDENTIFIED BY ‘mypassword’ WITH GRANT OPTION; // 授予myuser用户使用mypassword连接本机的MySQL服务，最好重启一次mysqld
```

### 连接MySQL
我是下载的MySQL提供的GUI工具<a href="http://dev.mysql.com/downloads/workbench/">MySQL Workbench</a>测试连接的，使用界面十分简单。Windows用户记得安装.net Framework，Mac用户应该直接可以使用。

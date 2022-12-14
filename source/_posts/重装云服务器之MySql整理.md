---
title: 重装云服务器之MySql整理
tags:
  - 云
  - MySql
categories: MySql
abbrlink: 3286839751
date: 2022-07-22 13:16:41
cover:
---

> 由于再下把服务器玩崩了（^-^），需要重装服务器系统，这次我选择LAMP ;因为我懒得再次安装MySQL之类的了。



![image-20220816143145916](重装云服务器之MySql整理/image-20220816143145916.png)

选择系统镜像，点击重置即可安装完毕，点击应用管理，可以看到应用安装完的应用信息；

上面也写了密码放置的位置

![image-20220816143209941](重装云服务器之MySql整理/image-20220816143209941.png)

我们连接到服务器 

~~~shell
[root@ls_patrick ~]# cat /root/REAME
~~~

查看到我们初始的应用信息和密码；



![image-20220816143228910](重装云服务器之MySql整理/image-20220816143228910.png)

复制密码进入我们的MySQL ；

~~~shell
[root@ls_patrick ~]# mysql -uroot -p
Enter password:
~~~

在Enter password:后粘贴这段密码

![image-20220816143238735](重装云服务器之MySql整理/image-20220816143238735.png)

看到欢迎信息就进入成功了；

一般我们会使用客户端连接（Navicat等)。接下来，我们需要开启 远程连接并开放端口，这样客户端才能连接上

命令行登入mysql后;

~~~shell
MySQL [(none)]> use mysql ;
Database changed

~~~

查看root权限信息。
~~~
MySQL [mysql]> select host, user from user;
+-----------+---------------+
| host      | user          |
+-----------+---------------+
| 127.0.0.1 | root          |
| localhost | mysql.session |
| localhost | mysql.sys     |
| localhost | root          |
+-----------+---------------+
4 rows in set (0.00 sec)

~~~

修改root用户的host

```shell
MySQL [mysql]> update user set host = '%' where user = 'root';
```
如果遇到以下发生错误，不理会
```
ERROR 1062 (23000): Duplicate entry '%-root' for key 'PRIMARY'
```
更新权限
```
MySQL [mysql]> flush privileges;
```

再次查看root权限信息，可以看到修改成功了
```

MySQL [mysql]> select host, user from user;
+-----------+---------------+
| host      | user          |
+-----------+---------------+
| %         | root          |
| 127.0.0.1 | root          |
| localhost | mysql.session |
| localhost | mysql.sys     |
+-----------+---------------+
4 rows in set (0.00 sec)

```

```shell
# 为防火墙添加 3306 端口
firewall-cmd --zone=public --add-port=3306/tcp --permanent
# 重启防火墙
firewall-cmd --reload
```

或者在百度云上配置

![image-20220816143312119](重装云服务器之MySql整理/image-20220816143312119.png)

![image-20220816143328243](重装云服务器之MySql整理/image-20220816143328243.png)

~~~shell
[root@ls_patrick ~]# service mysqld restart
Shutting down MySQL..                                      [  OK  ]
Starting MySQL..                                           [  OK  ]
~~~



![image-20220816143342606](重装云服务器之MySql整理/image-20220816143342606.png)

重置MySQL密码

进入mysql（如果忘记密码可以 `mysqld_safe --skip-grant-tables &` 直接运行下面无密码命令进入mysql）
1
设置新密码

~~~shell
 UPDATE mysql.user SET Password=PASSWORD('new-password') WHERE User='root';
~~~


在 mysql> 提示符下，键入以下命令刷新：

~~~shell
FLUSH PRIVILEGES;

exit;
~~~


重启 MySQL 服务器。

```shell
[root@ls_patrick ~]# service mysqld restart
Shutting down MySQL..                                      [  OK  ]
Starting MySQL..                                           [  OK  ]
```



MYSQL 就到此暂告一段落


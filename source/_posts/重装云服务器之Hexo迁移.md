---
title: 重装云服务器之Hexo迁移
date: 2022-07-22 16:04:22
tags: 
 - 云
 - Hexo
categories: Hexo
cover: 
---




>  接着上次MySql配置完毕，接下来把我们之前的hexo博客重新发布上来

## 一、服务器环境搭建

首先安装一些常用工具

```shell
[root@ls_patrick ~]# yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel perl-devel
```

![](http://rebp38war.bkt.clouddn.com/img/20220722145908.png)

刷刷刷.........一会就全部安装完成了，我们在安装git 安装的命令如下

```shell
[root@ls_patrick ~]# yum install -y git
```

安装完成后用git --version查看一下版本，能查到证明安装成功了。

```shell
[root@ls_patrick ~]# git --version
git version 1.8.3.1
```

接下来创建git使用的用户。

```shell
[root@ls_patrick ~]# useradd HelloWorldKing1
[root@ls_patrick ~]# passwd HelloWorldKing1 // 设置密码
Changing password for user HelloWorldKing1.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
[root@ls_patrick ~]# chmod 740 /etc/sudoers // 设置权限
```
[root@ls_patrick ~]# vim /etc/sudoers // 修改root权限 按i直接进入编辑状态。找个风水宝地（新的一行就可以）输入下面的命令
```
HelloWorldKing1 ALL=(ALL)    ALL
```

紧接着。

```shell
[root@ls_patrick ~]# chmod 600 /etc/sudoers //改回权限
[root@ls_patrick ~]# mkdir /home/hexo
[root@ls_patrick ~]# chown HelloWorldKing1:HelloWorldKing1 -R /home/hexo
```
完成后正式开始安装Nginx
```shell
[root@ls_patrick ~]# yum install -y nginx  //安装nginx
[root@ls_patrick ~]# systemctl start nginx.service  //启动Nginx服务
```
<font color=red>如果启动时，发生以下错误</font>

```shell
Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details.
```
分析：端口被占用了


```shell
[root@ls_patrick ~]# netstat -tnlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      18727/mysqld
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1519/sshd
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1370/master
tcp6       0      0 :::781                  :::*                    LISTEN      2477/./bcm-agent
tcp6       0      0 :::80                   :::*                    LISTEN      950/httpd
tcp6       0      0 :::22                   :::*                    LISTEN      1519/sshd
tcp6       0      0 ::1:25                  :::*                    LISTEN      1370/master
```

想起了是服务器镜像是LAMP 。可能是apache的80端口和nginx的80端口冲突了

进入apache的 conf目录修改端口

```shell
[root@ls_patrick conf]# vim httpd.conf
```

将80端口改为8080  :wq 保存退出

```shell
 Listen 8080
```

进入bin目录 重启apache

```shell
[root@ls_patrick bin]# apachectl restart
```

 再次启动nginx

```shell
[root@ls_patrick conf]# systemctl start nginx.service
```

如果正常启动，没有任何报错，说明Nginx配置好了。 **建立git仓库**

```shell
[root@ls_patrick conf]# su root
[root@ls_patrick conf]# cd /home/HelloWorldKing1
[root@ls_patrick HelloWorldKing1]# git init --bare blog.git  //创建Git仓库
[root@ls_patrick HelloWorldKing1]# chown HelloWorldKing1:HelloWorldKing1 -R blog.git  //授予Git仓库权限
```

同步网站根目录

```shell
[root@ls_patrick HelloWorldKing1]# cd blog.git/hooks/

[root@ls_patrick hooks]# vim post-receive
// 把下面的内容拷贝进去
#!/bin/sh
git --work-tree=/home/hexo  --git-dir=/home/HelloWorldKing1/blog.git checkout -f
```

再次修改权限，

```shell
chmod +x post-receive
```

这样就已经把服务器端的环境基本配置好了。



## 二、上传博客到服务器

接下来把博客重新上传到云服务器

有了文章我们就可以把博客上传到云服务器了。但是要想用Git把代码传到服务器，是需要密钥的。这就就直接创建一下密钥。在本地电脑打开`powerShell`，创建密钥。创建密钥的命令是 `ssh-keygen -t rsa` 如果你已经有了密钥就不用再次创建了，特别是公司已经有很多git管理的项目时，否则需要都进行重新配置。我这里是已经有密钥了。 到服务端创建一个存放密钥的文件夹。

```
su HelloWorldKing1
mkdir ~/.ssh   //创建存放密钥的文件夹
vim ~/.ssh/authorized_keys  //写入密钥
```

来到自己的客户端，找到`C:\Users\Administrator\.ssh` 打开`id_rsa`文件，直接复制全部，就可以完成了。 这样我们就可以通过HelloWorldKing1这个用户向服务器提交代码了，我们要测试一下，看看是否配置的可以连接上 **本地测试** 打开本地主机的`powerShell`，然后用ssh进行连接。

```
ssh -v HelloWorldKing1@180.76.183.66 //服务器ip
```

<font color=red>发生错误WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!</font>



<img src="http://rebp38war.bkt.clouddn.com/img/20220722155039.png" style="zoom:67%;" />

是因为本地记录了上次连接信息；需要这这段删掉即可

![](http://rebp38war.bkt.clouddn.com/img/20220722155006.png)



修改本地的hexo配置文件`_config.yml`,打开文件后，拖动到最后边，输入下面的配置

```
deploy:
  type: git
  repository: HelloWorldKing1@180.76.183.66:/home/HelloWorldKing1/blog.git
  branch: master
npm install --save hexo-deployer-git
```

安装完成后，输入下面的命令

```
hexo clean
hexo g -d
```

如果一切正常，就可以直接传到服务器上，然后输入网址`http://120.48.107.220/`就可以完成博客的浏览了，我这里没有使用域名，如果你有域名，只要把域名解析到这个网址就可以了。

![](http://rebp38war.bkt.clouddn.com/img/20220722155619.png)

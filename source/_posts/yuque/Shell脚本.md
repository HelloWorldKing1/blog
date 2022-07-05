---
title: Shell脚本
urlname: sq2vqv
date: '2022-07-05 10:04:46 +0800'
tags: []
categories: []
---

> 💡 本质是一个文件，文件里面存放的是 特定格式的指令，系统可以使用脚本解析器 翻译或解析指令并执行

- shell 脚本就是说我们把原来 linux 命令或语句放在一个文件中，然后通过这个程序文件去执行时，我们就说这个程序为 shell 脚本或 shell 程序；我们可以在脚本中输入一系统的命令以及相关的语法语句组合，比如变量，流程控制语句等，把他们有机结合起来就形成了一个功能强大的 shell 脚本

# 记录下 linux 下 shell 脚本的基本使用

贴一段项目部署中写的一段 sell 代码：

```shell
#!/bin/bash
app=hsmj-cpa-app-1.0.0-SNAPSHOT.jar
port=9086
path=/usr/local/hsmj/project
deploy_path=/usr/local/hsmj/project/deploy
log_path=/data/log/hsmj/
echo this is app : $app
echo port : $port
#若项目已启动，杀死旧进程
appId=`ps -ef | grep "$app" | grep -v grep | awk '{print $2}'`
echo appId = $appId
if [ "$appId" != "" ]; then
echo kill $appId
kill -9 $appId
echo sleep 3s
sleep 1
echo sleep 2s
sleep 1
echo sleep 1s
sleep 1
fi
#将jar包从jenkins工作空间中移动到指定路径下
rm -rf $deploy_path/$app
mv $path/$app $deploy_path/$app
cd $deploy_path
#防止进程被杀死
#BUILD_ID=dontKillMe
#后台进程形式启动项目
JAVA_HOME=/usr/local/java
echo "java环境"+$JAVA_HOME
nohup $JAVA_HOME/bin/java -jar -Dserver.port=$port -Dspring.profiles.active=test -Xmx256m -Xms128m  $deploy_path/$app  >/dev/null  2>&1 &
echo $app start success
#jps -l
exit 0

```

## 接下逐段解析代码

```shell
#!/bin/bash
```

**#!用来声明脚本由什么 shell 解释，否则使用默认 shell**

> 有不少时候不规范的写法能够忽略掉这一句，执行起来好像也是 ok，结果没什么不同,linux 系统上默认都是执行/bin/bash 来执行 shell 脚本若是有些用户使用的是 csh，那么缺乏第一行的“#！/bin/bash 的 shell 脚本执行结果就可能存在语法不兼容的问题，致使结果异常或者根本不能执行。linux

### shell 脚本也支持变量赋值

```shell
app=hsmj-cpa-app-1.0.0-SNAPSHOT.jar
port=9086
path=/usr/local/hsmj/project
deploy_path=/usr/local/hsmj/project/deploy
log_path=/data/log/hsmj/
```

#### echo 输出控制台信息

> Shell 的 echo 指令与 PHP 的 echo 指令类似，都是用于字符串的输出。

```shell
echo this is app : $app
echo port : $port
```

##### 1.显示普通字符串:

`echo "It is a test"`
这里的双引号完全可以省略，以下命令与上面实例效果一致：
`echo It is a test`

##### 2.显示转义字符

`echo "\"It is a test\""`
结果将是:
`"It is a test"`
同样，双引号也可以省略

##### 3.显示变量

read 命令从标准输入中读取一行,并把输入行的每个字段的值指定给 shell 变量
`#!/bin/sh read name echo "$name It is a test"`
以上代码保存为 test.sh，name 接收标准输入的变量，结果将是:
`[root@www ~]# sh test.sh OK #标准输入 OK It is a test #输出`

##### 4.显示换行

`echo -e "OK! \n" # -e 开启转义 echo "It is a test"`
输出结果：
`OK! It is a test`

##### 5.显示不换行

`#!/bin/sh echo -e "OK! \c" # -e 开启转义 \c 不换行 echo "It is a test"`
输出结果：
`OK! It is a test`

##### 6.显示结果定向至文件

`echo "It is a test" > myfile`

##### 7.原样输出字符串，不进行转义或取变量(用单引号)

`echo '$name\"'`
输出结果：
`$name\"`

##### 8.显示命令执行结果

`echo `date`` **注意：** 这里使用的是反引号 **`**, 而不是单引号 **'\*\*。
结果将显示当前日期
`Thu Jul 24 10:08:46 CST 2014`

#### 加下来这段代码表示 如果项目已经启动了，可以杀死旧的进程

```shell
#若项目已启动，杀死旧进程
appId=`ps -ef | grep "$app" | grep -v grep | awk '{print $2}'`
echo appId = $appId
if [ "$appId" != "" ]; then
echo kill $appId
kill -9 $appId
echo sleep 3s
sleep 1
echo sleep 2s
sleep 1
echo sleep 1s
sleep 1
fi
```

##### shell 的变量可以定义一长串的代码

`appId=`ps -ef | grep "$app" | grep -v grep | awk '{print $2}'``

##### 判断语句(`if[]; then fi`)

```shell
if [ "$appId" != "" ]; then
xxx
fi
```

##### 并且使用 $xxx 来调用变量

```shell
echo appId = $appId
if [ "$appId" != "" ]; then
echo kill $appId
kill -9 $appId
```

#### `sleep 1`表示休眠 1 秒

#### shell 当然也可以使用 linux 中的命令，并使用调用变量的形式，让代码更加灵活

```shell
#将jar包从jenkins工作空间中移动到指定路径下
rm -rf $deploy_path/$app
mv $path/$app $deploy_path/$app
cd $deploy_path
#防止进程被杀死
#BUILD_ID=dontKillMe
#后台进程形式启动项目
JAVA_HOME=/usr/local/java
echo "java环境"+$JAVA_HOME
nohup $JAVA_HOME/bin/java -jar -Dserver.port=$port -Dspring.profiles.active=test -Xmx256m -Xms128m  $deploy_path/$app  >/dev/null  2>&1 &
echo $app start success
```

#### 最后`exit `退出程序

```shell
exit  0：正常运行程序并退出程序；

exit  1：非正常运行导致退出程序；

exit 0 可以告知你的程序的使用者：你的程序是正常结束的。如果 exit 非 0 值，那么你的程序的使用者通常会认为
你的程序产生了一个错误。
在 shell 中调用完你的程序之后，用 echo $? 命令就可以看到你的程序的 exit 值。在 shell 脚本中，通常会根据
上一个命令的 $? 值来进行一些流程控制。0代表程序正确的执行，如下图例子所示：
```

### Shell 批量处理

> 如果有多个项目需要脚本来启动，每个都写脚本比较繁杂，可以考虑用 for 循环来批量处理

```shell
#!/bin/bash
path=/usr/local/hsmj/project
deploy_path=/usr/local/hsmj/project/deploy
array=([1]=hsmj-desk-app.sh [2]=hsmj-flow-app.sh [3]=hsmj-sys-app.sh [4]=hsmj-spt-app.sh  [5]=hsmj-auth.sh [6]=hsmj-autho.sh [7]=hsmj-gateway.sh [8]=hsmj-cpa-app.sh [9]=hsmj-comp-app.sh  [10]=hsmj-cust-app.sh [11]=hsmj-ctm-app.sh [12]=hsmj-ctm-api.sh [13]=hsmj-cto-app.sh [14]=hsmj-cto-job.sh )

for file in `ls $deploy_path`

do

if [[ $file == hsmj-*.jar ]]; then
	mv $deploy_path/$file $path/$file
	echo "move "$file" success"
fi

done

for b in ${array[*]}

do

sh $path/$b

echo "run "$path/$b" success"

done
。、
```

#### 这里 表示定义的数组，存入需要操作的项目

```shell
array=([1]=hsmj-desk-app.sh [2]=hsmj-flow-app.sh [3]=hsmj-sys-app.sh [4]=hsmj-spt-app.sh  [5]=hsmj-auth.sh [6]=hsmj-autho.sh [7]=hsmj-gateway.sh [8]=hsmj-cpa-app.sh [9]=hsmj-comp-app.sh  [10]=hsmj-cust-app.sh [11]=hsmj-ctm-app.sh [12]=hsmj-ctm-api.sh [13]=hsmj-cto-app.sh [14]=hsmj-cto-job.sh )
```

#### `for in do done`进行循环操作

```shell
for file in `ls $deploy_path`

do

if [[ $file == hsmj-*.jar ]]; then
	mv $deploy_path/$file $path/$file
	echo "move "$file" success"
fi

done
```

`for file in `ls $deploy_path``可以在循环目录下的所有文件 `if [[$file == hsmj-*.jar]]; `\*模糊匹配文件名

## 记录下 shell 使用中遇到的问题

```shell
[root@localhost project]# ./ctm-all.sh
-bash: ./ctm-all.sh: 权限不够
```

#### 刚创建的脚本权限不足导致的错误；需要我们先给这个脚本赋权限；

解决方法：
使用`chmod +x ctm-all.sh` 即可解决；

```shell
[root@localhost project]# ./ctm-all.sh
-bash: ./ctm-all.sh: /bin/bash^M: 坏的解释器: 没有那个文件或目录
```

#### 接着遇到的这个问题是，我的脚本在 Windows 中编写，导入 Linux 中运行的，因为在 Windows 中换行符为 \r\n，linux 下换行符为 \n；所导致的问题

解决方法：
只需要去掉多余的 \r 回车符 即可。操作办法可以用 sed 命令进行全局替换
`sed 's/\r//' -i ctm-all.sh`

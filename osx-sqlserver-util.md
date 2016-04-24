<!--
author: liuys
date: 2016-04-24
title: OS X 备份还原SQLServer数据库
tags: SQLServer
category: SQL
status: publish
summary: 在虚拟机中工作，来回的切换屏幕，感觉效率太低下了，我也就是在需要还原数据库的时候才进一下虚拟机，能不能不进虚拟机界面来远程还原数据库呢？答案是肯定的。作者的思路是首先使用FTP将备份文件上传到服务器上，然后再使用客户端执行脚本进行还原数据库！
-->
#### 吐槽
首先来吐槽一下微软，搞个SQLServer无法跨平台（目前），听说微软想把SQLServer做成跨平台的产品，忍不住有些小激动，一看发布时间，预计到2017年中了，看来还需要在OS X下开虚拟机、装windows、装SQLServer，体积可不小啊，整个装下来需要10G左右的空间，并且随着频繁的使用，体积还会越来越大，对于我这种只有256G的硬盘来说，有点捉襟见肘。
吐槽完毕！！！
#### 摘要
在虚拟机中工作，来回的切换屏幕，感觉效率太低下了，我也就是在需要还原数据库的时候才进一下虚拟机，能不能不进虚拟机界面来远程还原数据库呢？答案是肯定的。作者的思路是首先使用FTP将备份文件上传到服务器上，然后再使用客户端执行脚本进行还原数据库！

#### 一、测试如何使用脚本还原SQLServer数据库
先看一下脚本

```sql
RESTORE FILELISTONLY FROM DISK = 'C:\DATABAK\you_bak_filename.bak';
```
这样可以还原数据库，但是前提是你的服务器上没有`you_bak_filename.bak`所包含的数据库，如果存在，是不能进行还原的，来点复杂的

```sql
RESTORE DATABASE you_db_name FROM DISK = 'C:\SQLSERVERDATA\you_bak_filename.bak' WITH REPLACE, MOVE 'you_db_name' TO 'C:\SQLSERVERDATA\you_db_name.mdf', MOVE 'you_db_name_LOG' TO 'C:\SQLSERVERDATA\you_db_name.ldf';
```
大致意思就是先还原`you_db_name`数据库，不管这个原数据库名字是什么，然后将`mdf`和`ldf`文件移到正确的位置，这样会把你的`you_db_name`的数据库覆盖还原，请不要在生产环境这样做。
#### 二、测试OS X客户端连接SQLServer数据库
OS X下或者说linux下是有客户端可以连接的

```shell
brew install freetds
```
安装好以后测试一下

```shell
tsql -S localwin -U sa -P youpassword
```
输入`sql`然后再输入`go`，回车看看是不是可以连上了!
#### 三、测试FTP上传文件
使用FTP上传文件很简单，但是如何做到无交互这个稍微有一点点难度，万事问百度

```shell
ftp -n localwin<<EOF
		quote USER ftpuser
		quote PASS ftppass
		binary
		put 你的文件的路径 上传后的文件的路径
		quit
EOF
```
OK，思路已经清晰了，具体如何做，相信您已经有自己的想法了，下面是作者写的一个简单的shell，已经上传到github上了，还请大神批评指点，链接在[这里](https://github.com/liuyongsheng/osx-mssql-util.git)!
####四、用法
这个脚本可以直接 

```shell
sh yaopath/sqlserver.sh param1 param2 param3 param4
```
但是为了方便，可以使用`alias` 进行重新定义，当然也可以作为`oh-my-zsh`的插件运行，最简单的定义结果是

```shell
re localwin DB_20160424.bak
```
大致意思就是还原`localwin`上的`DB_20160424.bak`所包含的数据库，注意：备份文件中包含多个数据库的备份情况笔者没有测试，请自行测试！脚本中定义了备份文件的放置位置，需要将文件放到此文件夹中才能正确读取。
####五、扩展
有了远程还原数据库的经验，在此基础上进行了扩展，可以实现备份远程数据库到本地！例如：

```shell
bk localwin you_db_name
```
可以像sqlplus一样运行sql

```shell
sql localwin
```

本文完，转载请注明出处！






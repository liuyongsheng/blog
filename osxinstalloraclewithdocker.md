<!--
author: liuys
date: 2016-05-03
title: OS X 安装Oracle
tags: Oracle
category: Docker
status: publish
summary: docker简单的理解就是将你的应用程序及其依赖的环境打包成镜像（可以想象成Class），在运行应用时，需要基于镜像文件生成容器（可以想象成类实例），如果你对容器进行修改是不会对镜像有任何影响的（可以想象成设置属性值），只有在commit容器的时候，才会对镜像照成影响，当然你也可以对镜像打上标签，运行应用时可以选择任意版本的镜像。
只有原生的linux内核才支持docker，其基本原理是基于linux的cgroup进行管理，有兴趣的同学可以深入学习下；OS X和windows运行docker需要运行一个linux虚拟机，然后通过原生的docker客户端和虚拟机内核进行通讯，来达到“用户体验一样”的效果。
-->
如果你想在OS X上安装Oracle，但又不想破坏OS X系统的配置，那么使用docker安装是再合适不过的了，当然也可以使用虚拟机安装，收费的有VMware Fusion、Parallels Desktop等，开源的有Oracle公司的VirtualBox等，当然这几种产品各有特长，看个人喜好，本文介绍重点是使用docker进行安装。
#### 一、docker 入门
docker简单的理解就是将你的应用程序及其依赖的环境打包成镜像（可以想象成Class），在运行应用时，需要基于镜像文件生成容器（可以想象成类实例），如果你对容器进行修改是不会对镜像有任何影响的（可以想象成设置属性值），只有在`commit`容器的时候，才会对镜像造成影响，当然你也可以对镜像打上标签，运行应用时可以选择任意版本的镜像。
只有原生的Linux内核才支持docker，其基本原理是基于linux的cgroup进行管理，有兴趣的同学可以深入学习下；OS X和Windows运行docker需要运行一个Linux虚拟机，然后通过原生的docker客户端和虚拟机内核进行通讯，来达到“用户体验一样”的效果。
OS X安装docker，推荐使用`brew`来进行安装

```sh
brew install docker
```
当然这个还只是一个客户端，那么服务端应该怎么安装呢？推荐使用`docker-machine`，顾名思义就是创建`docker`的机器，也确实如其名，下面简单介绍`docker-machine`。

#### 二、docker-machine 入门
```sh
brew install docker-machine
docker-machine create -d virtualbox default 
```
这样就创建了一个名字是default的`docker`机器，是基于VirtualBox的，当然你也可以使用其他公司的虚拟化产品，但是需要下载相应的驱动，`docker-machine`集成了常见虚拟机的驱动，记得之前使用的VMware Fusion创建的虚拟机还需要下载单独的驱动，近期的版本都不需要了，应该是官方集成了，官方支持的驱动列表在[这里](https://github.com/docker/machine/tree/master/drivers)；本文使用开源、短小精悍的虚拟化产品`xhyve`，项目主页在[这里](https://github.com/mist64/xhyve.git)，高端大气上档次，如果你不想再安装一个虚拟机产品，那么这个就是最好的选择，那么开工！
首先下载`xhyve`源码，编译、安装，安装后就一个命令文件，大小只有210k，推荐使用brew安装，同时安装`docker-machine`驱动

```sh
brew install xhyve docker-machine-driver-xhyve
docker-machine create -d xhyve oracle --xhyve-experimental-nfs-share=true
```
我们又创建了一台名字叫做oracle的docker虚拟机，命令中设置了`xhyve-experimental-nfs-share=true`，这个参数的作用是对宿主机共享目录的控制，默认为`false`，默认情况下，你不能使用`docker run -v `参数来指定卷目录；创建完成后，执行一下`eval $(docker-machine env oracle)`将设置当前命令行窗口的环境，然后你就可以使用`docker images`来列出当前机器中所拥有的镜像了，运行一个Hello Word 试试

```sh
docker run centos echo Hello Word
```
看一看控制台是不是输出了Hello Word，执行这一句命令，首先docker会从本地寻找centos镜像，如果本地没有，它会去docker公共的镜像库中下载一个，当然你懂的，这个下载异常的慢；OK，基础环境搭建完毕了，那么我们开始安装Oracle！

#### 三、安装Oracle
使用docker安装oracle，网上有现成的oracle镜像，但是镜像文件太大，并且oracle的设置不一定满足你的要求，下载完了后还需要再进行修改，并且不能定制，后面会提到！
首先下载[这个项目](https://github.com/jaspeen/oracle-11g.git)，当然熟悉了DockerFile之后，你也可以自己写，分析了作者的源代码，这个就是一个引导安装oracle的DockerFile，使用静默安装，可以安装你想要的Oracle版本，你也可以修改代码内容，定制你需要安装的Oracle；首先要按照这个项目build一个docker镜像出来

```sh
$ git clone https://github.com/jaspeen/oracle-11g.git
$ cd <your_file_path>/oracle-11g
$ docker build -t oracle_base .
$ docker images 
REPOSITORY      TAG               IMAGE ID          CREATED             SIZE
oracle_base    latest             602ff20c5416      4 hours ago         337.9 MB
```
如果不出什么意外，就生成了一个基础的docker镜像，名字叫`oracle_base`，下面要依据这个镜像来安装oracle

```sh
docker run --privileged --name oracle11g -p 1521:1521 -v <your_database_path>:/install oracle_base
```
执行这个条命令，来安装oracle，首先下载的oracle安装文件解压到`<you_database_path>/database`中，`docker run`参数中有个`-v`，这个参数的意思是把安装文件目录挂载到`/install`目录中，安装文件的路径在容器中为`/install/database`，容器启动后，这个脚本会自动自动进行安装数据库程序和创建数据库，安装完成后执行

```sh
docker commit oracle11g oracle11g-installed
```
OK，将安装后的容器提交到docker镜像，以后需要创建oracle容器就可以使用`oracle-installed`这个镜像了

```sh
docker run -idt -p 1521:1521 --name oracle oracle-installed
```
在后台启动一个oracle容器，并将本机的1521端口和容器的1521端口进行映射，使用sqlplus就可连上oracle了，OK现在可以享受数据服务了！
注：第一次创建了oracle容器后，之后就可以使用`docker start oracle` 来启动了！完整的启动命令如下：

```sh
$ docker-machine start oracle
$ eval $(docker-machine env oracle)
$ docker start oracle
```
如果你觉得麻烦，可以写一个脚本文件，放到你认为合理的位置，就可以一个命令启动了！

#### 四、将oracle数据和容器分离开
作者这样使用了一段时间，觉得还算满意，但是有两个问题：

第一、`xhyve`创建的虚拟磁盘，不能分割成多文件（目前版本0.20），并且虚拟机磁盘占用磁盘空间和实际使用的空间严重不符，就比如我安装的oracle11gr2，实际占用空间为4~5G，但是就是眼睁睁的看着这个硬盘大概占了19G多，无奈啊，硬盘小，这种开销简直是浪费生命，实测是因为在安装的过程中，执行`docker commit`等命令时，生成的临时文件占用的磁盘空间没有被收回；

第二、在数据库中保存了很多数据，每一次想要更改一下容器的端口或者想挂载一下目录进去就需要先将当前容器提交至docker image，然后在新的docker image上运行镜像，久而久之，镜像文件就非常大，用着用着默认分的20G硬盘就占满了，是的确实占满了！

由于数据保存在容器内，每次提交都会将数据提交到镜像中，迁移也是一个麻烦，DockerFile中有一个命令，叫做 `VOLUME`这个命令的含义就是开放一个容器，和`docker run`中的 `-v`参数配合使用，就可以将容器中的数据写入到宿主机上，那么我们对DockerFile稍作修改

```sh
FROM centos:7
MAINTAINER jaspeen

ADD assets /assets
RUN chmod -R 755 /assets
RUN /assets/setup.sh
VOLUME /opt/oracle/oradata #增加了这一项，这个是存放oracle数据库文件的
EXPOSE 1521
CMD ["/assets/entrypoint.sh"]
```
重新编译oracle基础镜像，然后运行以下命令进行安装

```sh
docker run --privileged --name oracle11g -p 1521:1521 -v <your_database_path>:/install -v <your_data_path>/oradata:/opt/oracle/oradata oracle_base
```
重新安装后，orcle的数据文件就安装到宿主机了，如果再次进行调整容器的配置，就不需要再进行`docker commit`了，直接删除容器，再运行`docker run`了，这样你的数据还在那，只是在你运行oracle容器的时候，需要将数据文件目录映射到容器中的相应位置，完整的命令如下：

```sh
docker run -idt -p 1521:1521 -v <your_data_path>/data:/opt/oracle/oradata --name oracle oracle11g-installed
```
注：笔者在这样设置后，遇到了一个问题，安装oracle后，一共生成了两个控制文件，一个在`/opt/oracle/oradata`目录，也就是映射到宿主机上的，另外一个在容器`${ORACLE_HOME}/flash_recovery_area/orcl`目录下面，如果你创建了数据，并且删除了容器，再用`oracle11g-installed`这个镜像运行容器时oracle会起不来，原因是，两个控制文件不一致照成的，最简单的办法是安装好oracle后，将容器中的控制文件移到`/opt/oracle/oradata`目录中，也确实应该这样做！

全文完，转载请注明出处！


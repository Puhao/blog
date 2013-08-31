---
title: Raspberry Pi下使用MongoDB
date: '2013-08-31'
description:Raspberry Pi下折腾MongoDB
categories:
- 技术
tags:
- MongoDB
- Raspberry Pi
- Linux
- screen
---
其实我更想把标题改为Raspberry Pi下折腾MongoDB。在开始折腾前，先介绍一个linux下的工具。

安装准备
---
在很多场景下，我都是都是SSH登录到Linux主机进行操作的，而不是直接在Linux主机上面通过键盘操作。一旦我们断开连接，如果任务没有执行完，那这个我们在这个SSH下发起的任务都会被终止掉，这个时候我们借助screen可以很好的帮我们把这个问题解决掉。关于screen，推荐阅读下[《Linux技巧：使用screen管理你的远程会话》](http://www.ibm.com/developerworks/cn/linux/l-cn-screen/),详细介绍了为什么会关闭，到怎么使用screen。  
Raspberry Pi官方提供的镜像下面没有自带screen，因此需要我们先安装下。

```
sudo apt-get install screen
```

安装MongoDB
----
由于没有官方镜像提供，我们选择以源码方式安装。

1.	相关工具安装

	```
	sudo apt-get install build-essential libboost-filesystem-dev libboost-program-options-dev libboost-system-dev libboost-thread-dev scons libboost-all-dev git
	```		
2.	获取源代码  
	MongoDB的源代码是基于x86机器的little endian的，Raspberry Pi的CPU是基于ARM的big endian，在github上面有big endian的[nonx86版本](https://github.com/skrabban/mongo-nonx86)提供。
	
	```
	git clone https://github.com/skrabban/mongo-nonx86
	```
	
3.	编译安装

	如果是256m内存版本的Raspberry Pi，则编译安装的时候会出错，显示内存不足。因此还需要修改swap配置。修改``  /etc/dphys-swapfile ``，把`` CONF_SWAPSIZE `` 改成200。由于编译安装时间很长，需要几个小时，这个时候就可以借助screen了，命令输入完后做其他事情去好了，等待完成。
	
	```
	cd mongo-nonx86
	sudo scons
	sudo scons --prefix=/opt/mongo install
	```
	
配置MongoDB
-----
安装完MongoDB后，简单配置就可以使用MongoDB了，建议把MongoDB作为系统服务的一部分。

1.	设置工作目录

	*	增加用户
		
		```
		 sudo adduser mongodb 
		```
	*	创建日志目录
	
		```
		mkdir /var/log/mongodb/
		sudo chown mongodb /var/log/mongodb/
		```
	*	创建数据库目录
		
		```
		mkdir /var/lib/mongodb
		sudo chown mongodb /var/lib/mongodb	
		```
2.	增加系统服务

	源代码里面已经有系统服务的配置脚本了，不需要我们自己来写了。
	
	```
	sudo cp debian/init.d /etc/init.d/mongod
	sudo cp debian/mongodb.conf /etc/
	```
	修改下系统服务脚本`` /etc/ini.d/mongod ``里面50行，把`` DAMEON ``设置为`` /opt/mongo/bin/mongod ``,或者我们也可以不修改文件，直接建立个软连接。
	
	```
	sudo ln -s /opt/mongo/bin/mongod /usr/bin/mongod
	```
	
	使用chkconfig 增加系统服务，Raspberry Pi没有自带，需先安装下。
	
	```
	sudo apt-get install chkconfig
	sudo chkconfig --add mongod
	```
	
使用MongoDB
----
Raspberry Pi下面只能使用32位版本的MongoDB。32位版本的MongoDB有个限制就是最多处理2G的数据。另外，32位版本的MongoDB启动的时候，默认操作journal是不开启的，如果意外关闭MongoDB，则再次启动前需要进行repair修复。如果想避免这种情况，可以修改启动脚本，将journal开启，这样由于journal的开启占用总的数据，32位的MongoDB能处理的数据还要再下降一些。随着数据的增多，我遇到过在导出数据的时候出错，查看log文件发现是内存不够，建议更换到64位版本的MongoDB。  
可以使用asyn，share等方式配置MongoDB来同步数据。如果想到Raspberry Pi里面的数据导入到其他服务器，除了前面的方法外，还可以直接使用 mongodump导出数据，然后 mongorestore来完成了数据的迁移。  

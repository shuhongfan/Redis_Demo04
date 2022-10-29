# Redis安装说明

大多数企业都是基于Linux服务器来部署项目，而且Redis官方也没有提供Windows版本的安装包。因此课程中我们会基于Linux系统来安装Redis.

此处选择的Linux版本为CentOS 7.

Redis的官方网站地址：https://redis.io/





# 1.单机安装Redis

## 1.1.安装Redis依赖

Redis是基于C语言编写的，因此首先需要安装Redis所需要的gcc依赖：

```sh
yum install -y gcc tcl
```



## 1.2.上传安装包并解压

然后将课前资料提供的Redis安装包上传到虚拟机的任意目录：

![image-20211211071712536](assets/image-20211211071712536.png)

例如，我放到了/usr/local/src 目录：

![image-20211211080151539](assets/image-20211211080151539.png)

解压缩：

```sh
tar -xzf redis-6.2.6.tar.gz
```

解压后：

![image-20211211080339076](assets/image-20211211080339076.png)

进入redis目录：

```sh
cd redis-6.2.6
```



运行编译命令：

```sh
make && make install
```

如果没有出错，应该就安装成功了。



默认的安装路径是在 `/usr/local/bin`目录下：

![image-20211211080603710](assets/image-20211211080603710.png)

该目录以及默认配置到环境变量，因此可以在任意目录下运行这些命令。其中：

- redis-cli：是redis提供的命令行客户端
- redis-server：是redis的服务端启动脚本
- redis-sentinel：是redis的哨兵启动脚本



## 1.3.启动

redis的启动方式有很多种，例如：

- 默认启动
- 指定配置启动
- 开机自启



### 1.3.1.默认启动

安装完成后，在任意目录输入redis-server命令即可启动Redis：

```
redis-server
```

如图：

![image-20211211081716167](assets/image-20211211081716167.png)



这种启动属于`前台启动`，会阻塞整个会话窗口，窗口关闭或者按下`CTRL + C`则Redis停止。不推荐使用。



### 1.3.2.指定配置启动

如果要让Redis以`后台`方式启动，则必须修改Redis配置文件，就在我们之前解压的redis安装包下（`/usr/local/src/redis-6.2.6`），名字叫redis.conf：

![image-20211211082225509](assets/image-20211211082225509.png)

我们先将这个配置文件备份一份：

```
cp redis.conf redis.conf.bck
```



然后修改redis.conf文件中的一些配置：

```properties
# 允许访问的地址，默认是127.0.0.1，会导致只能在本地访问。修改为0.0.0.0则可以在任意IP访问，生产环境不要设置为0.0.0.0
bind 0.0.0.0
# 守护进程，修改为yes后即可后台运行
daemonize yes 
# 密码，设置后访问Redis必须输入密码
requirepass 123321
```



Redis的其它常见配置：

```properties
# 监听的端口
port 6379
# 工作目录，默认是当前目录，也就是运行redis-server时的命令，日志、持久化等文件会保存在这个目录
dir .
# 数据库数量，设置为1，代表只使用1个库，默认有16个库，编号0~15
databases 1
# 设置redis能够使用的最大内存
maxmemory 512mb
# 日志文件，默认为空，不记录日志，可以指定日志文件名
logfile "redis.log"
```



启动Redis：

```sh
# 进入redis安装目录 
cd /usr/local/src/redis-6.2.6
# 启动
redis-server redis.conf
```



停止服务：

```sh
# 利用redis-cli来执行 shutdown 命令，即可停止 Redis 服务，
# 因为之前配置了密码，因此需要通过 -u 来指定密码
redis-cli -u 123321 shutdown
```



### 1.3.3.开机自启

我们也可以通过配置来实现开机自启。

首先，新建一个系统服务文件：

```sh
vi /etc/systemd/system/redis.service
```

内容如下：

```conf
[Unit]
Description=redis-server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/redis-server /usr/local/src/redis-6.2.6/redis.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```



然后重载系统服务：

```sh
systemctl daemon-reload
```



现在，我们可以用下面这组命令来操作redis了：

```sh
# 启动
systemctl start redis
# 停止
systemctl stop redis
# 重启
systemctl restart redis
# 查看状态
systemctl status redis
```



执行下面的命令，可以让redis开机自启：

```sh
systemctl enable redis
```



# 2.Redis客户端

安装完成Redis，我们就可以操作Redis，实现数据的CRUD了。这需要用到Redis客户端，包括：

- 命令行客户端
- 图形化桌面客户端
- 编程客户端



## 2.1.Redis命令行客户端

Redis安装完成后就自带了命令行客户端：redis-cli，使用方式如下：

```sh
redis-cli [options] [commonds]
```

其中常见的options有：

- `-h 127.0.0.1`：指定要连接的redis节点的IP地址，默认是127.0.0.1
- `-p 6379`：指定要连接的redis节点的端口，默认是6379
- `-a 123321`：指定redis的访问密码 

其中的commonds就是Redis的操作命令，例如：

- `ping`：与redis服务端做心跳测试，服务端正常会返回`pong`

不指定commond时，会进入`redis-cli`的交互控制台：

![image-20211211110439353](assets/image-20211211110439353.png)



## 2.2.图形化桌面客户端

GitHub上的大神编写了Redis的图形化桌面客户端，地址：https://github.com/uglide/RedisDesktopManager

不过该仓库提供的是RedisDesktopManager的源码，并未提供windows安装包。



在下面这个仓库可以找到安装包：https://github.com/lework/RedisDesktopManager-Windows/releases

![image-20211211111351885](assets/image-20211211111351885.png)

### 2.2.1.安装

在课前资料中可以找到Redis的图形化桌面客户端：

![image-20211214154938770](assets/image-20211214154938770.png)

解压缩后，运行安装程序即可安装：

![image-20211214155123841](assets/image-20211214155123841.png)

此处略。

安装完成后，在安装目录下找到rdm.exe文件：

![image-20211211110935819](assets/image-20211211110935819.png)

双击即可运行：

![image-20211214155406692](assets/image-20211214155406692.png)



### 2.2.2.建立连接

点击左上角的`连接到Redis服务器`按钮：

![image-20211214155424842](assets/image-20211214155424842.png)

在弹出的窗口中填写Redis服务信息：

![image-20211211111614483](assets/image-20211211111614483.png)

点击确定后，在左侧菜单会出现这个链接：

![image-20211214155804523](assets/image-20211214155804523.png)

点击即可建立连接了：

![image-20211214155849495](assets/image-20211214155849495.png)

Redis默认有16个仓库，编号从0至15.  通过配置文件可以设置仓库数量，但是不超过16，并且不能自定义仓库名称。

如果是基于redis-cli连接Redis服务，可以通过select命令来选择数据库：

```sh
# 选择 0号库
select 0
```






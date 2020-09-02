### centos常用命令合集

#### 防火墙相关

>>>关闭防火墙



systemctl stop firewalld.service            #停止firewall
systemctl disable firewalld.service        #禁止firewall开机启动

>>>开启端口



firewall-cmd --zone=public --add-port=80/tcp --permanent

> > > 命令含义

--zone #作用域
--add-port=80/tcp #添加端口，格式为：端口/通讯协议
--permanent #永久生效，没有此参数重启后失效 

当开启端口 出现 FirewallD is not running 错误的时候  可能是未开启防火墙导致的  

开启端口是需要先开启防火墙   

可以通过

firewall-cmd --state firewall-cmd --state firewall-cmd --state firewall-cmd --state

 firewall-cmd --state 命令查看防火墙的状态  显示running即已开启了。 

systemctl start firewalld  开启防火墙命令 

>>>重启防火墙

firewall-cmd --reload  firewall-cmd reload

其他常用命令：

firewall-cmd --list-port firewall-cmd --list-port firewall-cmd --list-port

firewall-cmd --list-port                    查看开启的全部端口号
firewall-cmd --state                          ##查看防火墙状态，是否是running
firewall-cmd --reload                          ##重新载入配置，比如添加规则之后，需要执行此命令
firewall-cmd --get-zones                      ##列出支持的zone
firewall-cmd --get-services                    ##列出支持的服务，在列表中的服务是放行的
firewall-cmd --query-service ftp              ##查看ftp服务是否支持，返回yes或者no
firewall-cmd --add-service=ftp                ##临时开放ftp服务
firewall-cmd --add-service=ftp --permanent    ##永久开放ftp服务
firewall-cmd --remove-service=ftp --permanent  ##永久移除ftp服务
firewall-cmd --add-port=80/tcp --permanent    ##永久添加80端口 
iptables -L -n                                ##查看规则，这个命令是和iptables的相同的
man firewall-cmd                              ##查看帮助

#### nginx 相关

重启命令:

注意：首先你要进入nginx文件夹下的sbin目录中，然后执行命令：

./nginx -s reload

#### tomcat相关

#### 查看端口命令

netstat -lntp netstate -lntp netsate -lntp  netstate -lntp

netstat -lntp #查看监听(Listen)的端口

netstat -antp #查看所有建立的TCP连接   netstate -antp netstate -antp

其他关于查看服务器网络信息命令：
1、查看Linux系统主机名： Linux学习，http:// linux.it.net.cn

   hostname

​    localhost.localdomain
2、查看服务器IP地址：

ifconfig|grep 'inet addr:'|grep -v '127.0.0.1'|cut -d: -f2|awk '{ print $1}'

​    192.168.17.238
​    192.168.1.9
3、查看linux网关：

route |grep default

​    default 192.168.1.1 0.0.0.0 UG 0 0 0 em1
4、查看linux打开服务：

chkconfig --list|grep 启用 #查看开启的服务

​    sshd 0:关闭 1:关闭 2:启用 3:启用 4:启用 5:启用 6:关闭
​    httpd 0:关闭 1:关闭 2:关闭 3:启用 4:关闭 5:关闭 6:关闭
5、查看服务器DNS配置：

cat /etc/resolv.conf

​    nameserver 192.168.0.66
​    nameserver 202.106.0.20
6、其他网络信息：

iptables -L #查看防火墙规则

route -n #查看路由表

netstat -s #查看网络统计信息



### 关机命令

Linux centos关机命令：

- 　　1、halt 立刻关机
- 　　2、poweroff 立刻关机
- 　　3、shutdown -h now 立刻关机(root用户使用)
- 　　4、shutdown -h 10 10分钟后自动关机

## **在 netstat 输出中显示 PID 和进程名称 netstat -p**

netstat -p 可以与其它开关一起使用，就可以添加 “PID/进程名称” 到 netstat 输出中，这样 debugging 的时候可以很方便的发现特定端口运行的程序。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# netstat -pt Active Internet connections (w/o servers) Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name tcp        1      0 ramesh-laptop.loc:47212 192.168.185.75:www        CLOSE_WAIT  2109/firefox tcp        0      0 ramesh-laptop.loc:52750 lax:www ESTABLISHED 2109/firefox
```

**补充：**

netstat –lnp 查看监听端口(查看网络连接状况)

netstat –an  查看所有的链接

netstat –an |grep IP+端口 |grep –ic estab 查看某个端口的并发数 

yum -y install net-tools



## 解压文件的命令

```
1.进入文件所在文件夹
2.执行命令：
   tar -zxvf 文件名
```

## 输出当前进入到的文件夹的路径

```
pwd
```

## 删除指定后缀的文件

```
例如：
rm *.cmd
```





### chown -R命令的使用

​        chown将指定文件的拥有者改为指定的用户或组，用户可以是用户名或者用户ID；组可以是组名或者组ID；文件是以空格分开的要改变权限的文件列表，支持通配符。系统管理员经常使用chown命令，在将文件拷贝到另一个用户的名录下之后，让用户拥有使用该文件的权限。

**1．命令格式：**

　　　　chown [选项]... [所有者][:[组]] 文件...

**2．命令功能：**

　　　　通过chown改变文件的拥有者和群组。在更改文件的所有者或所属群组时，可以使用用户名称和用户识别码设置。普通用户不能将自己的文件改变成其他的拥有者。其操作权限一般为管理员。

##### 3**．命令参数：**

　　**必要参数:**

　　　-c 显示更改的部分的信息

　　　-f 忽略错误信息

　　　-h 修复符号链接

　　　-R 处理指定目录以及其子目录下的所有文件

　　　-v 显示详细的处理信息

　　　-deference 作用于符号链接的指向，而不是链接文件本身

**选择参数:**

　　　　--reference=<目录或文件> 把指定的目录/文件作为参考，把操作的文件/目录设置成参考文件/目录相同拥有者和群组

　　　　--from=<当前用户：当前群组> 只有当前用户和群组跟指定的用户和群组相同时才进行改变

　　　　--help 显示帮助信息

　　　　--version 显示版本信息 



### 清屏命令：

```
（1）clear 
    这个命令将会刷新屏幕，本质上只是让终端显示页向后翻了一页，如果向上滚动屏幕还可以看到之前的操作信息。一般都会用这个命令。
（2）ctrl+l（等价clear）
（3）reset
    这个命令将完全刷新终端屏幕，之前的终端输入操作信息将都会被清空，这样虽然比较清爽，但整个命令过程速度有点慢，使用较少。
```








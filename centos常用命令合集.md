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

sudo ../../sbin/nginx/ -s reload

sudo ../../sbin/nginx/ -r reload

sudo ../../sbin/nginx/ -e reload

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


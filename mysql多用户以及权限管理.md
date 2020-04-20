# mysql多用户以及权限管理

## 一. 创建用户

### 命令:

```
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```

### 说明：

- username：你将创建的用户名
- host：指定该用户在哪个主机上可以登陆，如果是本地用户可用localhost，如果想让该用户可以**从任意远程主机登陆**，可以使用通配符`%`
- password：该用户的登陆密码，密码可以为空，如果为空则该用户可以不需要密码登陆服务器

### 例子：

```
CREATE USER 'dog'@'localhost' IDENTIFIED BY '123456';
CREATE USER 'pig'@'192.168.1.101_' IDENDIFIED BY '123456';
CREATE USER 'pig'@'%' IDENTIFIED BY '123456';
CREATE USER 'pig'@'%' IDENTIFIED BY '';
CREATE USER 'pig'@'%';
```

## 二. 授权:

### 命令:

```
GRANT privileges ON databasename.tablename TO 'username'@'host'
```

### 说明:

- privileges：用户的操作权限，如`SELECT`，`INSERT`，`UPDATE`等，如果要授予所的权限则使用`ALL`
- databasename：数据库名
- tablename：表名，如果要授予该用户对所有数据库和表的相应操作权限则可用`*`表示，如`*.*`

### 例子:

```
GRANT SELECT, INSERT ON test.user TO 'pig'@'%';
GRANT ALL ON *.* TO 'pig'@'%';
GRANT ALL ON maindataplus.* TO 'pig'@'%';
```

### 注意:

用以上命令授权的用户不能给其它用户授权，如果想让该用户可以授权，用以下命令:

```
GRANT privileges ON databasename.tablename TO 'username'@'host' WITH GRANT OPTION;
```

## 三.设置与更改用户密码

### 命令:

```
SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');
```

如果是当前登陆用户用:

```
SET PASSWORD = PASSWORD("newpassword");
```

### 例子:

```
SET PASSWORD FOR 'pig'@'%' = PASSWORD("123456");
```

## 四. 撤销用户权限

### 命令:

```
REVOKE privilege ON databasename.tablename FROM 'username'@'host';
```

## 说明:

privilege, databasename, tablename：同授权部分

### 例子:

```
REVOKE SELECT ON *.* FROM 'pig'@'%';
```

### 注意:

假如你在给用户`'pig'@'%'`授权的时候是这样的（或类似的）：`GRANT SELECT ON test.user TO 'pig'@'%'`，则在使用`REVOKE SELECT ON *.* FROM 'pig'@'%';`命令并不能撤销该用户对test数据库中user表的`SELECT` 操作。相反，如果授权使用的是`GRANT SELECT ON *.* TO 'pig'@'%';`则`REVOKE SELECT ON test.user FROM 'pig'@'%';`命令也不能撤销该用户对test数据库中user表的`Select`权限。

具体信息可以用命令`SHOW GRANTS FOR 'pig'@'%';` 查看。

## 五.删除用户

### 命令:

```
DROP USER 'username'@'host';
```

今天开发中在Centos7中安装MySQL5.6版本后，在表中新建了一个weicheng的账户，并且设置了密码，但是在用weicheng账号登陆mysql发现，如果使用“mysql -uweicheng -p”登陆会报错，即使密码正确也不能登录，最后发现，直接用“mysql -uweicheng”不输入密码也可以登陆。
后来，查询了资料原因是:应为数据库里面有空用户,通过
select * from mysql.user where user='';
查询如果有，然后通过
use mysql;
delete from user where user = '';
删除了多余的空白账户， 然后，通过
flush privileges;­
重载一次权限表,最后用
service mysqld restart
重启mysql服务，问题得到解决，至此mark一下！
Tip：
1、一定要记住重启mysql服务，否则不会生效，自己就是因为没有重启msyql导致一直得不到解决！
2、msyql的用户表在mysql数据库中的user表中，主要字段有host，user，password等，作为mysql用的管理的主要表。

mysql刷新权限命令：FLUSH PRIVILEGES;（一般用于数据库用户信息更新后）

还有一种方法，就是重启mysql服务器也可以


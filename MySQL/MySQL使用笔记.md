# MySQL  使用笔记

## 一、在Windows上ZIP包安装MySQL

[MySQL Docs](https://dev.mysql.com/doc/refman/8.0/en/windows-install-archive.html)

### 1、解压ZIP包

选择安装路径，一般情况下，MySQL Server安装在`C:\mysql`,如果你没有安装在`C:\mysql`, 就必须在启动时指定安装路径或在配置文件（option file）中配置安装路径。

### 2、创建配置文件

在Windows上，当MySql server启动时，它会在下面几个路径下寻找配置文件（option file）：

* the Windows directory
* `C:\`
* the MySQL installation directory. （建议）

> The Windows directory一般是`C:\WINDOWS`. 可以在CMD下使用`echo %WINDIR%`命令获得。

MySQL首先从上述每个路径下寻找 `my.ini` 文件中的配置信息, 然后再从 `my.cnf` 文件寻找配置信息.但是为了避免误解，最好使用一个配置文件 `my.ini` 。

文件中包含MySQL Server，MySQL client等需要的相关配置。例如，如果MySQL安装在 `E:\mysql` 且 data director为 `E:\mydata\data`, 配置文件如下：

```properties
[mysqld]
# set basedir to your installation path
basedir=E:/mysql
# set datadir to the location of your data directory
datadir=E:/mydata/data
```

注意配置文件中是正斜杠/，如果要使用反斜杠需要添加转义符，如：

```properties
datadir=E:\\mydata\\data
```

### 3、初始化data文件夹

data文件夹包含了MySQL的各种重要信息，刚下载的ZIP包下没有data文件夹。可以使用以下命令初始化MySQL:

```plain
bin\mysqld --initialize --console
bin\mysqld --initialize-insecure --console
```

* 使用`--initialize` 初始化会为`root`用户自动生成一个随机密码 .
* 使用`--initialize-insecure`,不会自动生成密码。

> 注意：在初始化文件夹时会用到`basedir`和`datadir`配置项，所以需要先创建配置文件。

data文件夹的初始化过程中，server会创建一个`'root'@'localhost'`管理员账户和一些预留账户，其中一些预留账户被锁定无法使用客户端登陆。

### 4、启动 MySQL Server

```plain
mysqld --console
```

### 5、安装为 windows 服务

```plain
mysqld --install
```

### 6、修改Root密码

* 如果使用`--initialize`初始化data文件夹，使用随机生成的密码登录。
* 如果使用`--initialize-insecure`初始化data文件夹，使用下面命令登录：

```plain
mysql -u root --skip-password
```

连接成功后,使用下面命令修改/设置密码：

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root-password';
```

### 7、完整的配置文件

```properties
[mysqld]
# 设置3306端口
port=3306
# 设置mysql的安装目录
basedir=D:\\developer\\mysql-8.0.19-winx64
# 设置mysql数据库的数据的存放目录
datadir=D:\\developer\\mysql-8.0.19-winx64\\data
# 允许最大连接数
max_connections=200
# 允许连接失败的次数。
max_connect_errors=10
# 服务端使用的字符集默认为UTF8
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 默认使用“mysql_native_password”插件认证
default_authentication_plugin=mysql_native_password
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8
```

## 二、在 Ubuntu 上安装 MySQL

[MySQL Docs](https://dev.mysql.com/doc/mysql-apt-repo-quick-guide/en/)

### 1、添加 MySQL APT 仓库

```plain
sudo wget https://dev.mysql.com/get/mysql-apt-config_0.8.15-1_all.deb

sudo dpkg -i mysql-apt-config_0.8.15-1_all.deb

sudo apt-get update
```

### 2、使用 APT 安装 MySQL

```plain
sudo apt-get install mysql-server
```

安装过程中需要为 root 用户设置密码。

### 3、切换版本

```plain
sudo dpkg-reconfigure mysql-apt-config
sudo apt update
```

### 4、开启和停止MySQL服务

```plain
sudo service mysql status

sudo service mysql stop

sudo service mysql start
```

### 5、卸载MySQL

```plain
sudo apt-get remove mysql-server
sudo apt-get autoremove
```

## 三、Mac下安装MySQL

<https://dev.mysql.com/doc/refman/8.0/en/macos-installation-pkg.html>

## 四、使用 Docker 安装

```java
# 拉取镜像 tag : 5.7、8.0

docker pull mysql:tag

docker run -itd --name mysql-test -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:tag
```


> 更新: 2025-09-13 07:47:38  
> 原文: <https://www.yuque.com/thinkspace/lcb0zg/piivwo>
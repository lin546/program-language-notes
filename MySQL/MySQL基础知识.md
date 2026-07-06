# MySQL 基础知识



## 一、基础知识


### 1、UTF-8 与 UTF8mb4


我们知道 UTF-8 使用 1~4 个字节来表示一个字符。但是我们常用的字符使用 1~3 个字节就可以表示了。而在 MySQL 中，字符集表示一个字符所用的最大字节长度在一定程度上会影响系统的性能。所以 MySQL 定义了下面两个概念：



+ **utf8mb3**：阉割过的 UTF-8 字符集，只使用 1~3 字节表示字符。
+ **utf8mb4**：正宗的 UTF-8 字符集，使用 1~4 字节表示字符。



> 注意：在MySQL 中 utf8 是 utf8mb3 的别名，在 8.0 版本之前默认使用的是 latin1，在 MySQL 8.0 中默认使用的utfmb8。
>

## 二、权限表


安装MySQL时会自动创建一个mysql的数据库，mysql数据库里存储的都是权限表。

### 1、user表
user表时MySQL中最重要的一个权限表，记录允许连接到服务器的账号信息，里面的权限是全局级的。user表有39个字段，这些字段可以分为4类：

> 1、用户列：包括Host、User、Password。其中User和Host为User表的联合主键。当用户与服务器之间建立连接时，输入的用户名。主机名，密码都匹配时才能登陆成功。
>
>  
>
> 2、权限列：user表的权限列包括Select_priv、Insert_priv等以priv结尾的字段。这些字段的值只有Y和N。Y表示该权限可以用到所有数据库上；N表示该权限不能用到所有数据库上。
>
>  
>
> 3、安全列：ssl_type；ssl_cipher；x509_issuer；x509_subject； ssl用于加密；x509标准可以用来标识用户。普通的发行版都没有加密功能。可以使用SHOW VARIABLES LIKE 'have_openssl'语句来查看是否具有ssl功能。
>
>  
>
> 4、资源控制列:user表的4个资源控制列是：
>
>  
>
> + max_questions：每小时可以允许执行多少次查询；
> + max_updates：每小时可以允许执行多少次更新；
> + max_connections：每小时可以建立多少连接；
> + max_user_connections：单个用户可以同时具有的连接数。
>
>  
>
> 默认值为0，表示无限制。
>



### 2、db表


user表中权限列设置的是全局性的权限，而db表存储了某个用户对一个数据库的权限。



db表中分为用户列和权限列：



（1）用户列



+ Host：主机名；
+ Db：数据库名；
+ User：用户名；



（2）、权限列



剩下的*_priv皆为权限列。



> 注意：  
用户先根据user表的内容获取权限，然后再根据db表的内容获取权限。也就是说db表可以覆盖user表中的权限。
>



### 3、 tables_priv表和columns_priv表


> tables_priv：可以对单个表进行权限设置：
>
>  
>
> + tables_priv表包含8个字段：
> + Host：主机名；
> + DB：数据库名；
> + User：用户名；
> + Table_name：表名
> + Table_priv：对表进行操作的权限(Select,Insert,Update,Delete,Create,Drop,Grant,References,Index,Alter)
> + Column_priv：对表中的数据列进行操作的权限(Select,Insert,Update,Rederences)；
> + Timestamp：修改权限的时间
> + Grantor：权限的设置者
>
>  
>
> columns_priv：可以对单个数据列进行权限设置，有7个列，作用同上：
>
>  
>
> + Host、Db、User、Table_name、Column_name、Column_priv、Timestamp
>
>  
>
> **    MySQL权限分配是按照user表-> db表 -> table_priv表 -> columns_priv表的顺序进行分配的。** **在数据库系统中，先判断user表中的值是否为'Y'，如果user表中的值是'Y'，就不需要检查后面的表。如果user表为N，则一次检查后面的表。**
>



### 4、proces_priv表


>  
>
> + procs_priv表可以对存储过程和存储函数进行权限设置。
> + procs_priv表包含8个字段，分别是：
> + Host：主机名；
> + Db：数据库名；
> + User：用户名；
> + Routine_name：存储过程或函数名称；
> + Routine_type：类型(取值有：FUNCTION或PROCEDURE)；
> + Proc_priv：拥有的权限(Execute：执行；Alter Routine：修改；Grant：权限赋予)；
> + Timestamp：字段存储更新的时间；
> + Grantor：字段设置者；
>
>  
>



## 三、账户管理


### 1、登陆MySQL服务器


```sql
 mysql -h localhost -u root -p  [database]
```



`-p`：代标password， 可以在`-p`后直接输入密码，`-p`与密码之间没有空格 ，但是这时密码是显示在屏幕上的，故不安全。



### 2、退出MySQL服务器


> exit和ctrl+c
>



### 3、创建普通用户


在MySQL数据库中，创建用户有三种方式



3.1、使用create user语句来创建新用户



使用create user语句创建用户，必须拥有全局的create user权限。其格式如下



```sql
CREATE USER user[IDENTIFIED BY [PASSWORD] 'password'],
[user[IDENTIFIED BY [PASSWORD] 'password']]...
```



其中user参数表示新建用户的账户，user由用户名（user）和主机名（host）构成，格式为user[@host](https://my.oschina.net/u/116016)，IDENTIFIED BY关键字用来设置用户的密码；password参数表示用户的密码：如果密码是一个普通的字符串，就不需要使用PASSWORD关键字，可以使用初始密码。



示例：



```sql
create user 'guolin'@'%' IDENTIFIED BY 'admin'
```



执行之后user表会增加一行记录，但权限暂时全部为N。



3.2、使用insert 语句新建普通用户



可以使用INSERT语句直接将用户的信息添加到mysql.user表。但必须拥有mysql.user表的INSERT权限。另外，ssl_cipher、x509_issuer、x509_subject没有值，必须要设置值，否则INSERT语句无法执行。



示例：



```sql
INSERT INTO mysql.user(Host,User,Password,ssl_cipher,x509_issuer,x509_subject) VALUES('%','newuser1',PASSWORD('123456'),'','','')
```



执行INSERT之后，要使用下面命令来使用户生效：



FLUSH PRIVILEGES



3.3、使用grant 语句创建新用户



使用grant语句不仅可以创建新用户，还可以在创建的同时对用户授权，使用grant语句创建新用户时必须拥有grant权限，grant语句是添加新用户并授权他们访问MySQL对象的首选方法，grant语句的基本格式如下：



```sql
GRANT priv_type ON database.table
TO user[IDENTIFIED BY [PASSWORD] 'password']
[,user [IDENTIFIED BY [PASSWORD] 'password']...]
```



+ priv_type：参数表示新用户的权限；
+ databse.table：参数表示新用户的权限范围；
+ user：参数新用户的账户，由用户名和主机构成；
+ IDENTIFIED BY关键字用来设置密码；
+ password：新用户密码；



### 4、删除普通用户


4.1、DROP USER语句删除普通用户



要使用drop user必须拥有mysql数据库的全局create user权限。



```sql
drop user username1[,username2]...

user 是需要删除的用户，由用户名user和主机名host构成
例如：drop user 'username'@'localhost'
```



4.2、delete语句删除普通用户



可以使用delete语句直接将用户的信息从mysql.user表中删除，但必须拥有mysql.user表的delete权限。



```sql
DELETE FROM mysql.user WHERE Host = '%' AND User = 'admin'
```



删除完成后，一样要FLUSH PRIVILEGES才生效。



### 5、密码管理


root用户修改自己的密码



> 1、通过MySQLadmin命令修改root密码
>
>  
>
> MySQLadmin命令的基本语法格式如下：
>
>  
>
>  
>
> 5.2、通过修改user表修改密码
>
>  
>
> update user表中的password字段的值，也可以达到修改的目的，同样要有权限才可以成功。
>
>  
>
> UPDATE user SET Password = PASSWORD('123') WHERE USER = 'myuser'
>
>  
>
> FLUSH PRIVILEGES后生效。
>
>  
>
> 5.3、使用SET语句来修改密码
>
>  
>
> 使用root用户登录到MySQL服务器后，可以使用SET语句来修改密码：
>
>  
>
> root修改自己的密码，不需要用户名
>
>  
>
> SET PASSWORD = PASSWORD("123");
>
>  
>
> 为了使更改生效，需要重启mysql或使用FLUSH PRIVILEGES语句刷新权限，重新加载权限表。
>

```plain
mysqladmin -u -username -h localhost -p password "new_password"
```



root用户修改普通用户密码



> 1、使用SET语句修改普通用户的密码
>
>  
>
> 格式为:set password for 'user'@'host' = PASSWORD('newpwd');
>
>  
>
> 只有root可以通过更新更新MySQL数据库的用户来更改其他用户的密码，同时使用普通用户修改自己的密码，可省略FOR子句更改自己的密码：
>
>  
>
>  
>
> 2、通过update语句修改普通用户的密码
>
>  
>
> 使用root用户登录后，执行下面语句：
>
>  
>
>  
>
> 执行FLUSH PRIVILEGES刷新权限表
>

```plain
set password = password('newpwd')
```

```plain
update mysql.user set password =password("newpwd")
where user = "username" and host="hostname"
```



普通用户修改自己的密码



> 通过set语句设置自己的密码
>
>  
>
> set password = password('newpwd')
>



## 四、权限管理


账户权限信息被存储在MySQL数据库的user、db、host、tables_priv、columns_priv和procs_priv表中。在MySQL启动时，服务器将这些数据库表中权限信息的内容读入内存。



GRANT和REVOKE语句所涉及的权限名称如下：



![](https://gitee.com/lin546/pictures/raw/master/picgo_img/20190621135405.png)



### 1、授权


授予的权限分为多个层级：



>  
>
> + 全局层级：全局权限适用于一个给定服务器中的所有数据库，这些权限存储在mysql.user表中。grant all on _._ 和revoke all on _._ 只授予和撤销全局权限。
> + 数据库层级：数据库权限适用于一个给定数据库中的所有目标，这些权限存储在mysql.db表中，grant all on db_name.和revoke all on db_name.只授予和撤销数据库权限。
> + 表层级：表权限适用于一个给定表中的所有列，这些权限存储在mysql.tables_priv表中。grant all db_name.tbl_name.和revoke all on db_name.tbl_name.只授予和撤销表权限。
> + 列层级：列权限适用于一个给定表中的单一列。这些权限存储在mysql.columns_priv表中。
>
>  
>



grant语法如下：



```sql
GRANT priv_type [(column_list)] ON database.table
TO user [IDENTIFIED BY [PASSWORD] 'password']
[,user [IDENTIFIED BY [PASSWORD] 'password']]...
WITH with_option[with_option]
```



+ priv_type参数表示权限类型；
+ column_list：参数表示权限作用于哪些列上，没设置则位于整个表上；
+ user参数由用户名和主机名构成；形式是"'username'@'hostname'"；
+ IDENTIFIED BY参数用于为用户设置密码；
+ password：用户新密码；



WITH关键字后面带有一个或多个with_option参数。有5个选项：



+ GRANT OPTION：被授权的用户可以将这些权限赋予给别的用户；
+ MAX_QUERIES_PER_HOUR count：设置没消失可以允许执行count次查询；
+ MAX_UPDATES_PER_HOUR count：设置每个消失可以允许执行count次更新；
+ MAX_CONNECTIONS_PER_HOUR count：设置每小时可以建立count个连接；
+ MAX_USER_CONNECTIONS count：设置单个用户可以同时具有的count个连接数；



示例：



GRANT SELECT,UPDATE ON _._  
TO 'myuser'@'%'  
WITH GRANT OPTION;



### 2、收回权限


收回权限，就是取消某个用户的某些权限。MySQL中使用REVOKE关键字来为用户设置权限。



Revoke语句由两种语法格式，第一种语法是收回指定用户的所有权限，语法如下：



```sql
revoke all privileges,grant option
from 'user'@'host'[,'user'@'host' ...]
```



另一种为取消某些用户的某些权限



```sql
REVOKE priv_type[(column_list)]
ON database.table
FROM user[,user]
```



column_list：参数表示权限作用于哪些列上，没设置则位于整个表上；



示例：回收用户myuser的SELECT权限



```sql
REVOKE SELECT ON *.*FROM 'myuser'@'%'
```



收回myuser的所有权限：



```sql
REVOKE ALL PRIVILEGES,GRANT OPTION FROM 'myuser'@'%'
```



### 3、查看权限


SHOW GRANTS语句用于查看权限。同时mysql数据库下的user表中存储着用户的基本权限。



```sql
SELECT * FROM mysql.user

SHOW GRANTS
```



## 参考
+ [https://www.jb51.net/article/132521.htm](https://www.jb51.net/article/132521.htm)
+ [https://www.cnblogs.com/geaozhang/p/6724393.html](https://www.cnblogs.com/geaozhang/p/6724393.html)
+ [https://www.cnblogs.com/beyang/p/7580814.html](https://www.cnblogs.com/beyang/p/7580814.html)



> 更新: 2023-08-06 17:06:02  
> 原文: <https://www.yuque.com/thinkspace/lcb0zg/oqq10i>
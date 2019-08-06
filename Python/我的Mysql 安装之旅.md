## 我的Mysql 安装之旅

[下载](https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.17-winx64.zip)   [参考](https://jingyan.baidu.com/article/fc07f989bf2cc712ffe51902.html)

首页--社区--mysql社区服务，下载的是zip格式文件；如果是在MySQL on windows则找到的是下载msi格式文件。这两种格式很有意思。

+ zip格式是解压就能使用的，但需要手动配置很多东西
+ msi格式是按照弹出的向导一步步安装到最后来使用的。

 最终的结果都是一样的，也就是zip解压后，手动配置得到的结果就是msi格式一步步后引导的生成的。msi是windows installers的程序安装数据包，和exe文件很像，zip文件是压缩格式文件。我们使用zip解压配置方式，同时解释msi安装方式过程所作的事情

#### 下载解压

注意解压之后的文件夹路径要想清楚，因为这个路径就确定了是mysql数据库服务器的路径了。

#### **配置环境变量**

电脑--属性-高级系统设置-环境变量，就可以找到环境变量

点击系统变量下面Path，然后点击编辑，然后点击新建，将mysql的bin路径直接复制粘贴到下面。然后确定。

#### **配置Mysql的my.ini文件**

解压的mysql根目录文件下面新建一个my.ini文件 , 将下面这部分拷贝到这个my.ini文件里面

```
[client]port=3306default-character-set=utf8

[mysqld]
# 设置服务器的访问端口和字符集
port=3306
character_set_server=utf8
# 允许最大连接数
max_connections=20
# 设置为自己MYSQL的安装目录
basedir=D:\RDC\MySQL\mysql-8.0.17-winx64
# 设置为MYSQL的数据目录
datadir=D:\RDC\MySQL\mysql-8.0.17-winx64\data
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```

**注意上面的basedir和datadir属性值是你自己的MySQL的根路径。**

####  初始化MySQL 

##### service服务安装

以管理员身份打开cmd命令窗口。输入：mysqld --initialize --user=mysql --console 然后回车。

```
PS C:\Users\admin> mysqld --initialize --user=mysql --console
2019-08-01T08:37:07.407857Z 0 [System] [MY-013169] [Server] D:\RDC\MySQL\mysql-8.0.17-winx64\bin\mysqld.exe (mysqld 8.0.17) initializing of server in progress as process 20076
2019-08-01T08:37:07.409334Z 0 [Warning] [MY-013242] [Server] --character-set-server: 'utf8' is currently an alias for the character set UTF8MB3, but will be an alias for UTF8MB4 in a future release. Please consider using UTF8MB4 in order to be unambiguous.
2019-08-01T08:37:42.599820Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: ImBNas4Zdf_u
2019-08-01T08:37:55.981047Z 0 [System] [MY-013170] [Server] D:\RDC\MySQL\mysql-8.0.17-winx64\bin\mysqld.exe (mysqld 8.0.17) initializing of server has completed
```

A temporary password is generated for root@localhost: **ImBNas4Zdf_u** 

ImBNas4Zdf_u 就是你的初始密码。这个一定要记住，可先复制到文本中保存下来。

 然后执行 mysqld --install 命令。看到下面显示service成功安装。即wins的service服务安装了。

``` 
Remove of the Service Denied! 如果不是管理员身份打开cmd命令窗口 , 会报错, 管理员身份打开, 重新install即可
```

)

##### service服务细节

service服务可以在 任务管理器--服务看到。我的服务名是mysqlSun，只要在 mysqld --install mysqlSun 时跟上你自己想要的服务名就可以服务名，默认就是mysql这个名字。删除服务的命令是：mysqld --remove mysqlSun。此时我的mysql服务是关闭的。 

##### 开启服务  net start mysql

执行 **net start mysql** 就开启服务了。只有开启服务才能链接操作数据库的。看下面显示，此时mysql服务器已经开启了。

#### **登陆数据库 ** mysql -u root -p

 开启服务后，继续执行 mysql -u root -p ，此时会让输入密码，这个时候据要把上面mysql给你的默认密码拷过来了。输入密码回车。

 输入密码回车后，显示下面 Welcome... 字样时，恭喜你！这时才真正成功安装mysql数据了。还是绿色安装，zip解压版的，不想要时不用卸载的那种。



#### 修改密码

执行 set password='123'; 此时密码就改成数字 123 了，注意执行命令时要带上分号哦，因为时操作数据命令。



#### 退出数据库

 quit; 或者 exit ; 或者ctrl c  或者ctrl z + kill -9  所有的加;回车才会执行

![](D:\DJGitBook\数据库\img\Mysql.png)

上图就是两处忘了加分号;



#### [Navicat 连接MySQL 8.0.11 出现2059错误](https://www.cnblogs.com/lifan1998/p/9177731.html)

mysql8 之前的版本中加密规则是mysql_native_password,而在mysql8之后,加密规则是caching_sha2_password

```mysql
mysql -uroot -ppassword #登录
use mysql; #选择数据库
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER; #更改加密方式
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password'; #更新用户密码
FLUSH PRIVILEGES; #刷新权限
```

#### [解决MySQL数据库中1045错误的方法——Windows系统](https://blog.csdn.net/lzf_hlh/article/details/80885139)





```mysql
create user ‘lovedj’@‘localhost’ identified by ‘123’; #--->有错
create user loveDJ3 @ localhost identified by "jianchi1314";  #--->有错
create user lovedj @ ‘localhost’ identified by "123"; #--->有错
create user loveDJ3 @ localhost identified by "jianchi1314";  #--->有错


create user lovedj @'%' identified by 'jianchi1314'; # 粗暴省事不安全

grant all privileges on *.* to lovedj@"%";    		# 直接超级权限  粗暴有用不安全

delete from user where user='loveDJ';				# 粗暴省事不安全
```

navicat  2054 :

```mysql
# 选择数据库
use mysql; 
select user,plugin from user where user ='root';

# 更新用户密码：
ALTER USER'lovedj '@'%' IDENTIFIED BY 'jianchi1314' PASSWORD EXPIRE NEVER;

# 更改加密方式：
ALTER USER'lovedj '@'%' IDENTIFIED WITH mysql_native_password BY 'jianchi1314';

# 刷新权限：
FLUSH PRIVILEGES;
```



```mysql
# 创建DjGoShop数据库  mysql默认小写 

net start mysql
mysql -u lovedj -p
create database DjGoShop charset=utf8;
use DjGoShop;
show databases;
```


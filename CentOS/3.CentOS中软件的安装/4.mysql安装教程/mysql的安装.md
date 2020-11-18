**系统软件：**[CentOS-7.4](https://pan.baidu.com/s/1HwhZa1xWg8aDipRdQgfkyg)   **mysql软件**：通过yum命令在线安装

> ##### 1.安装前的准备

- 先使用`yum list|grep mysql`命令来查看yum源中是否有mysql-server相关的可用包；

![image-20191115234346180](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118133808.png)

由上图可知，虽然有很多跟mysql有关的包，但是都不是与mysql-server相关的，这时候如果使用`yum -y install mysql-server`命令来安装mysql的话，就会出现下图的情况。

![image-20191115234749744](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118133832.png) 

为了解决上述情况，可以通过先安装mysql的repo源的方式来安装mysql服务。

- 首先通过`wget http://repo.mysql.com/mysql57-community-release-el7-11.noarch.rpm`命令下载mysql的repo源；
- 然后使用`rpm -ivh mysql57-community-release-el7-11.noarch.rpm`命令来进行安装；
- 安装完成后，可以使用`ll /etc/yum.repos.d/mysql-community*`命令进行查看，如下图所示，则说明repo源安装成功。

![image-20191116004727658](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118133843.png)

这时候如果我们再使用`yum list|grep mysql`命令查看yum源中mysql相关的包时，会发现多了很多包，而且还有mysql-community-server包，这时候我们就可以使用yum命令来安装mysql软件了。

> ##### 2.mysql的安装和登录

- 使用`yum -y install mysql-server`命令安装mysql包；
- 安装完成后，先设置一下默认字符集，使用`vim /etc/my.cnf`命令打开my.cnf文件，然后在里面添加`character-set-server=utf8`这样一行内容后保存退出，如下图所示。

![image-20191116014037233](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118133852.png) 

- 然后使用`systemctl status mysqld`命令查看一下mysql服务的启动状态，如果没有启动，则使用`systemctl start mysqld`命令进行启动，如果已经启动，则使用`systemctl restart mysqld`命令重启一下，然后就可以进行登录了；
- 在mysql5.7中，初始密码已经不再是空密码了，而是自动给我们生成了一个临时密码，所以这个时候我们要先获取到这个临时密码，可以使用`cat /var/log/mysqld.log|grep 'temporary password'`命令进行获取。如下图所示，红框标记处就是我这边的临时密码。

![image-20191116011739897](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118133857.png)

- 获取到临时密码后，我们可以使用`mysql -u root -p`命令进行登录，然后在输入密码的时候输入临时密码即可。

> ##### 3.密码的修改和重置

我们在登录成功后是无法操作任何数据库和表的，必须要先修改密码才行。

- 可以使用`alter user 'root'@'localhost' identified by '这里是你要修改的密码';`语句进行密码的修改，mysql5.7中要求我们的密码要是一个强密码，所以这个密码必须要包含大小写字母、数字及标点符号，且长度要在6位以上；
- 密码修改完成之后，下次再登录mysql就可以使用新密码了。但是由于这个密码是一个强密码，比较难记，所以我们极有可能会忘记，如果忘记密码的话，可以通过以下步骤进行密码的重置：
  - 首先使用`vim /etc/my.cnf`命令打开my.cnf文件，把`skip-grant-tables`添加到my.cnf文件中后保存退出；
  - 然后使用`systemctl restart mysqld`命令重启下mysql服务；
  - 之后使用`mysql -u root -p`命令登录mysql，这时候密码已经是空了，不用输入密码就可以登录了；
  - 登录后首先使用`use mysql;`语句切换数据库；
  - 然后使用`update user set Authentication_string=password('这里是新密码') where User='root';`语句换一个新密码；
  - 之后使用`flush privileges;`语句刷一下权限；
  - 最后就是使用`exit`退出mysql，退出后使用`vim /etc/my.cnf`命令打开my.cnf文件，把之前增加的`skip-grant-tables`这一行内容删除掉后保存退出，再使用`systemctl restart mysqld`命令重启下mysql服务即可。

> ##### 4.防火墙的配置

- 首先使用`service firewalld status`命令查看防火墙是否已经开启，如果没有开启，则使用`systemctl start firewalld.service`命令进行开启，开启后，使用`firewall-cmd --zone=public --add-port=3306/tcp --permanent`命令开放3306端口；
- 然后使用`firewall-cmd --reload`命令使配置生效，这时候我们可以使用`firewall-cmd --list-ports`命令查看防火墙中已开放的端口里有没有3306端口，有就说明添加成功了。

> ##### 5.自启动的配置

- 依次执行`systemctl enable mysqld`命令和`systemctl daemon-reload`命令即可；
- 然后可以重启下CentOS系统，不执行启动mysql服务的命令，直接使用`mysql -u root -p`命令进行登录，可以登录就表示设置成功了。

> ##### 6.使用工具进行远程连接

- 首先登录进数据库，然后使用`GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '你的密码' WITH GRANT OPTION;`语句开放远程连接的权限即可。这里如果担心安全问题，不想远程连接root用户的话，也可以新建一个用户，比如admin用户，只需把以上语句中的root改成admin就可以了，然后后面输入的密码就是为admin用户设置的密码；
- 之后再使用`flush privileges;`语句刷一下权限，然后使用`exit`退出数据库，退出后最好是使用`systemctl restart mysqld`命令重启下mysql数据库服务；
- 最后打开远程连接工具，输入IP地址、用户名、密码和端口号后就可以进行连接了，如下图所示。

![image-20191116190633135](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118133908.png) 

---

**至此，mysql5.7在CentOS中的安装结束。**
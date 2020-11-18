**系统软件：**[CentOS-7.4](https://pan.baidu.com/s/1HwhZa1xWg8aDipRdQgfkyg)   **tomcat软件：**[apache-tomcat-8.5.47.tar.gz](https://pan.baidu.com/s/1EUr6d2woY6mKPTLwvO3rrw)

> **1.解压缩软件**

解压缩命令：`tar -zxvf apache-tomcat-8.5.47.tar.gz -C /usr/local`

**注意：以上命令是指将软件解压缩到指定目录下，这里指定的目录是/usr/local，不指定目录的话，默认解压缩到当前目录下。**

> **2.配置环境变量**

- 使用vim编辑器打开配置文件，命令为：`vim /etc/profile`；
- 在配置文件的最下方添加如下内容：

```xml
export CATALINA_HOME=/usr/local/apache-tomcat-8.5.47
```

**注意：上面等号右边的是软件解压缩后的目录。**

- 添加完成之后，在命令模式下，使用`:wq`命令保存退出；
- 使配置生效，命令为：`source /etc/profile`。

> **3.配置UTF-8字符集**

- 打开tomcat目录下conf文件夹中的server.xml文件，命令为：`vim /usr/local/apache-tomcat-8.5.47/conf/server.xml`；
- 找到配置8080默认端口的位置，在xml节点末尾添加`URIEncoding="UTF-8"`，如下图所示：

![20191112232946](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118133626.jpg) 

- 添加完成之后，在命令模式下，使用`:wq`命令保存退出。

> **4.tomcat的验证**

- 使用`cd /usr/local/apache-tomcat-8.5.47/bin`命令进入到tomcat的bin目录下，然后执行`./startup.sh`命令，看到如下界面，说明tomcat启动成功(安装tomcat前需要先安装jdk，否则这里可能无法正常启动)；

![20191112233613](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118133633.jpg)

- 使用`firewall-cmd --list-ports`命令查看防火墙中已开放的端口里有没有8080端口，如果出现类似"FirewallD is not running"这样的字样，说明防火墙没有开启，则使用`systemctl start firewalld.service`命令开启防火墙；
- 防火墙开启后，再使用`firewall-cmd --list-ports`命令查看是否有8080端口，如果还是没有，则使用`firewall-cmd --zone=public --add-port=8080/tcp --permanent`命令开放8080端口；
- 开放8080端口后，可以使用`firewall-cmd --reload`命令使配置生效，或者重启系统也可以。如果是重启系统的话，由于之前已经启动的tomcat已停止运行，所以这时候别忘了再启动一下tomcat，然后再使用`firewall-cmd --list-ports`命令就会列出已经开放的8080端口了，最后在浏览器中输入ip地址和端口号进行访问，出现如下页面就说明tomcat验证成功了。

![20191113003114](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118133643.jpg)

------

**至此，tomcat在CentOS中的安装结束。**
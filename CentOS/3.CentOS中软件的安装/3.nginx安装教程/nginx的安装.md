**系统软件：**[CentOS-7.4](https://pan.baidu.com/s/1HwhZa1xWg8aDipRdQgfkyg)   **nginx软件：**[linux-nginx-1.16.1.tar.gz](https://pan.baidu.com/s/1B27lp9-tmgHfoKBRJLepTg)

> **1.安装依赖包**

##### 安装gcc：

- 先使用`gcc -v`命令查询版本信息，看系统是否安装过gcc；
- 如果系统中没有安装过，则使用`yum install gcc`命令进行安装。

##### 安装pcre：

- 安装命令为：`yum install pcre-devel`

##### 安装zlib：

- 安装命令为：`yum install zlib zlib-devel`

##### 安装openssl：

- 安装命令为：`yum install openssl openssl-devel`

如果不想一个个地安装这些依赖包，也可以使用综合命令`yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel`直接进行一次性安装。

**注意**：安装过程中如果出现类似"Is this ok [y/d/N]:"字样，表示问你是否确定安装，在冒号后输入y就可以了，y就是yes的意思。

> **2.解压缩软件**

解压缩命令：`tar -zxvf linux-nginx-1.16.1.tar.gz`

> **3.nginx的安装**

- 首先通过`cd nginx-1.16.1`命令进入到nginx解压缩后的目录中；
- 然后执行`./configure`来安装nginx；
- 之后继续执行`make`命令，最后再执行`make install`命令。

**注意**：nginx默认是安装在"/usr/local/nginx"路径下的，如果安装完成之后不知道安装在哪里了，可以通过`whereis nginx`命令查看。

> **4.添加防火墙访问权限**

- 首先使用`service firewalld status`命令查看防火墙是否已经开启，如果没有开启，则使用`systemctl start firewalld.service`命令进行开启，开启后，使用`firewall-cmd --zone=public --add-port=80/tcp --permanent`命令开放80端口；
- 然后使用`firewall-cmd --reload`命令使配置生效，这时候我们可以使用`firewall-cmd --list-ports`命令查看防火墙中已开放的端口里有没有80端口，有就说明添加成功了。

> **5.nginx的验证**

- 使用`/usr/local/nginx/sbin/nginx`命令或者进入到nginx的sbin目录下使用`./nginx`命令来启动nginx，由于nginx启动成功后不会有什么比较明显的提示，所以如果我们想要确定是否真的成功启动了的话，可以通过`ps -ef|grep nginx`命令来查看是否有nginx相关的进程，有的话就说明启动成功了；
- 启动成功后直接在浏览器的地址栏中输入 "http://ip:80/" 或者 "http://ip/" 进行访问即可，这里的ip是我们CentOS系统的ip，出现如下页面就说明nginx已经安装成功并可以正常使用。

![20191113182808](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118133708.jpg) 

> **6.nginx的入门案例**

- 首先使用`vim /usr/local/nginx/conf/nginx.conf`命令来编辑nginx.conf文件；

- 然后在nginx.conf文件中增加"include vhost/*.conf;"，之后在命令模式下使用`:wq`命令保存退出。如下图所示：

![20191113180236](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118133723.jpg)

**注意**：这里不这么增加也行，也可以把下面将要写的一些配置直接写在nginx.conf文件中，但是这个文件毕竟是系统文件，所有的东西都写在这个文件中的话，不好管理不说，一个不注意还可能把这个文件给搞废了，所以为了保险起见，这里就把我们要写的东西都先放在一个目录下，然后再通过include把它们都引入到这个nginx.conf文件中。

- 上一步引入的是vhost目录下所有以conf结尾的文件，由于vhost目录还不存在，所以这里使用`mkdir /usr/local/nginx/conf/vhost`命令创建一个vhost目录；
- 然后在该目录下创建一个名为baidu.conf的文件，并添加以下内容到该文件中：

```xml
server {
  listen 80;
  autoindex on;
  server_name www.test.com;
  access_log /usr/local/nginx/logs/access.log combined;
  index index.html index.htm index.jsp index.php;
  #error_page 404 /404.html;
  if ( $query_string ~* ".*[\;'\<\>].*" ){
      return 404;
  }
  location / {
      proxy_pass http://www.baidu.com;
      add_header Access-Control-Allow-Origin '*';
  }
}
```

下面我来解释一下以上内容中一些关键部分的主要含义，如下图所示：

![20191113225100](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118133740.jpg) 

- 这时候为了以防万一，可以进入到nginx的sbin目录下使用`./nginx -s reload`命令来重启下nginx；
- 一般如果是作学习用的话，我们都只会把CentOS系统安装在虚拟机上，我虚拟机中的CentOS系统是没有界面的那种，所以如果想要通过浏览器访问上面的地址，就只能使用本机的windows系统上安装的浏览器了。如果是这样的话，直接在浏览器中访问是不会有结果的，这时候还需要进入"C:\Windows\System32\drivers\etc"目录中配置一下hosts文件。我们需要在hosts文件中加入下面这一行内容，然后保存即可(下面的那个地址应该是你虚拟机中CentOS的主机地址，我这里粘的是我这边的)。

```xml
192.168.124.15  www.test.com
```

- 然后我们在浏览器的地址栏中输入 "www.test.com" 后进行访问，发现出现的就是百度首页的页面了，这时候就说明我们已经成功地使用nginx完成了请求转发。

------

**至此，nginx在CentOS中的安装结束。**


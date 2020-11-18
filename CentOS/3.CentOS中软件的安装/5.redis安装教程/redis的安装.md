**系统软件：**[CentOS-7.4](https://pan.baidu.com/s/1HwhZa1xWg8aDipRdQgfkyg)   **redis软件：**[redis-5.0.6](http://download.redis.io/releases/)

> ##### 1.依赖包的安装

- 先执行`yum -y install gcc-c++`命令;
- 然后再执行`yum -y install tcl`命令。

**注**：第一个依赖包CentOS系统中一般默认都会带的有，不过为了以防万一，所以这里还是装了一遍。

> ##### 2.redis的安装

- 在压缩包的目录下执行`tar -zxvf redis-5.0.6.tar.gz -C /usr/local/`命令对压缩包进行解压缩操作，后面跟的那个是解压缩后的路径，这个路径可以根据自身需求进行设置；
- 我这边进入到"usr/local"目录下后，就会看到一个redis-5.0.6目录，然后进入该目录中，执行`make`命令；
- 之后执行`make test`命令，出现如下内容则表示已经安装完成且没有任何问题。

![test](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118133931.png)

> ##### 3.redis的启动和关闭

**常规启动**：

- 进入到redis的src目录下，使用`./redis-server`命令进行启动，出现下图内容表示启动成功。

![启动](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118133948.png)

**注**：这种启动方式属于前台启动，这种方式的缺陷就是，一旦启动redis就不能在该标签页进行其他操作了，必须新建一个标签页或者强制停止redis才行。

**通过修改配置文件启动**：

- 进入redis的目录下，如`/usr/local/redis-5.0.6`目录，然后打开该目录下的redis.conf文件；
- 把该文件中daemonize的状态从no改为yes后保存退出，如下图所示；

![状态](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118133958.png)

- 然后进入redis的src目录中，使用`./redis-server ../redis.conf`命令启动redis服务，如下图所示，这种启动方式属于后台启动，如果我们不确定redis服务是否启动的话，可以使用`ps -ef|grep redis`命令查看一下后台是否有redis服务的进程。

![image-20191119180023603](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118134005.png)

**注**：这种启动方式属于后台启动，这种启动方式的好处就是，启动redis服务后，我们还可以在该标签页中进行一些其他的操作，而且也不用停止redis服务。

**redis的关闭**：

- 进入到redis的src目录下，使用`./redis-cli shutdown`命令进行关闭即可，这时候再使用`ps -ef|grep redis`命令进行查看的话，会发现已经没有redis的进程服务了；
- 如果是使用前台启动的方式来启动redis的话，我们也可以使用`ctrl+c`组合键进行强制关闭。

**注**：一般不推荐使用`ctrl+c`组合键进行强制关闭这种方式，因为这种方式不会进行持久化，所以会有数据丢失的风险。

> ##### 4.redis自带客户端的启动

- 首先启动redis服务；
- 然后进入到redis的src目录下使用`./redis-cli`命令启动redis的客户端即可，下图就表示已经成功启动客户端了；

![image-20191119214209615](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118134014.png)

- 如果想要退出客户端的话，使用`quit`命令即可；

> ##### 5.jedis客户端的使用

![image-20191119220450347](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118134027.png)

**注**：需要注意的是，如果使用java代码或者redis的图形界面工具连接redis的话，则需要在防火墙中开放redis的端口才行，开放redis端口的命令是`firewall-cmd --zone=public --add-port=6379/tcp --permanent`， 然后再使用`firewall-cmd --reload`命令使配置生效即可。 

---

**至此，redis在CentOS中的安装结束。**
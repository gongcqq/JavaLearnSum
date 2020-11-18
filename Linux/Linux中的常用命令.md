### 1.关机和重启命令

`shutdown -h now`  		  立刻进行关机

`shutdown -h 1`     		 一分钟后进行关机

`shutdown -r now`  		  重启计算机

`reboot`     	      	 重启计算机

`sync`		   		   把内存的数据同步到硬盘

![1](https://i.loli.net/2020/05/21/LFzevZB3TGSmPad.gif)

### 2.用户相关命令

#### 2.1 创建和删除用户

可以使用`useradd 用户名`来创建一个用户，创建好用户后，会自动为该用户创建一个家目录，而且这个家目录是在**/home**目录里的。使用该用户登录主机的时候，登录后的默认路径就是家目录的路径。

<font color="gray">假设创建了一个tom用户，那么家目录的路径就是**/home/tom**。</font>

也可以使用`useradd -d 目录 用户名`的方式在创建用户的时候自定义该用户的家目录。

创建好用户后，可以使用`passwd 用户名`命令来为该用户设置密码，如果想更改该用户的密码，也是使用这个命令。

创建好用户后，可以使用`cat /etc/passwd|cut -f 1 -d:`命令来查看该用户是否创建成功。

如果不需要这个用户了，可以使用`userdel 用户名`命令删除该用户，如果是使用`userdel -r 用户名`命令的话，表示连该用户的家目录一并进行删除。 

![2.1](https://i.loli.net/2020/05/24/SIHPthMrDa9jc6s.gif)

<font color="red">注：</font>在删除用户时，我们一般不会将该用户的家目录一起删除掉。

#### 2.2 查询用户信息

直接使用`id 用户名`就可以查询用户信息了。

![image](https://i.loli.net/2020/05/25/F3NsecKq9Y1PQIR.png) 

第一个表示用户id号，第二个表示用户所在组的id号，第三个表示组名。

如果用户不存在，就会显示无此用户。

![1](https://i.loli.net/2020/05/25/mM8FDJkpKcnTqiN.png) 






























> **1.修改CentOS的网络适配器设置**

如下图所示：

<img src="https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118131623.jpg" alt="设置适配器"  />

> **2.查看虚拟机中的网关地址**

- 步骤一：

![1](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118132010.jpg)

- 步骤二：

![2](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118132036.jpg)

由上图可以看出，我这边的网关地址是：`192.168.68.2`

> **3.配置静态IP**

- 进入CentOS7系统中，执行`vim /etc/sysconfig/network-scripts/ifcfg-ens33`命令打开ifcfg-ens33文件；
- 打开文件后，修改BOOTPROTO的值为static，如果ONBOOT的值是no的话，改成yes，然后再新增IPADDR、NETMASK和GATEWAY这三个参数，我这边修改前后的内容如下所示；

**修改前：**

![3](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118132742.jpg) 

**修改后：**

```php
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static          #这里要改成static，默认是dhcp
IPADDR="192.168.68.12"    #这个就是静态IP地址，可以随便想一个，但必须要和VMnet8的子网IP在同一网段
NETMASK="255.255.255.0"   #这个是子网掩码，直接和我写的一样就行
GATEWAY="192.168.68.2"    #这个是网关地址，就是前面在虚拟机中查到的那个地址
DNS1="192.168.68.2"       #这个和网关地址保持一致就行
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=e911c873-36b8-4a3d-b5ea-ff770aa7b077
DEVICE=ens33
ONBOOT=yes                #这里是表示是否开机启动，这里默认如果是no就改成yes，是yes的话就不用改了
IPV6_PRIVACY=no
ZONE=public
```

- 使用`systemctl restart network`命令重新启动网络服务，使配置生效；
- 然后使用`ifconfig`命令查看一下IP地址，发现已经变成了上面配置的那个静态IP地址了。

![5](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118132756.jpg)

---

**以上就是CentOS7中配置静态IP地址的方法。**


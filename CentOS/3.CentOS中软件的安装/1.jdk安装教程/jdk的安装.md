**系统软件：**[CentOS-7.4](https://pan.baidu.com/s/1HwhZa1xWg8aDipRdQgfkyg)  **JDK软件：**[jdk-8u231-linux-x64.rpm](https://pan.baidu.com/s/1DBpTE5z3trB_LhivqNx0Uw)

> **1.清理已经存在的JDK**

首先要检查一下之前是否安装过JDK，如果安装过，先卸载掉。有时候刚安装好的CentOS系统，就会自带一个OpenJDK。

查看命令：`rpm -qa|grep jdk`

卸载命令：`sudo yum remove XXX`(XXX为上一个命令查到的结果)

**注**：如果通过查看命令查不到的话，就不用卸载了。

> **2.给软件包赋予权限**

权限赋予命令：`sudo chmod 777 jdk-8u231-linux-x64.rpm`

这里的777权限是一个全开权限，7代表的是赋予读、写、执行的权限，777表示给用户、用户组、其他人都赋予读、写、执行的权限。

**注**：之所以赋予全开权限，是因为每个人的情况不一样，所以没法确定我们目前使用的账号是否具有读、写、执行的权限，所以干脆赋予全开权限，这样也是为了避免因为权限问题而影响JDK软件的安装。

> **3.安装JDK软件**

安装命令：`sudo rpm -ivh jdk-8u231-linux-x64.rpm`

> **4.默认安装路径介绍**

执行过安装命令后，JDK会被默认安装在/usr/java目录下，可以通过`cd /usr/java`命令进入该路径下进行查看。

> **5.配置JDK环境变量**

- 使用vim编辑器打开配置文件，命令为：`vim /etc/profile`；

- 把光标移动到文件的最下方，保持输入法为英文状态，然后按键盘上的`i`进入到可编辑模式；
- 在文件的最下方输入以下内容：

```bash
export JAVA_HOME=/usr/java/jdk1.8.0
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
```

如下图所示：

<img src="https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118133553.jpg" alt="20191112182145"  />

**注**：JAVA_HOME后的路径是JDK的默认安装路径。如果使用的是我提供的软件包的话，安装后的路径就是usr/java/jdk1.8.0_231-amd64，我这里是把jdk1.8.0_231-amd64目录改成了jdk1.8.0目录。

- 内容加好之后按键盘上的`ESC`进入命令模式，然后输入vim编辑器的`:wq`命令进行保存退出；
- 使配置生效，命令为：`source /etc/profile`；
- 执行`java -version`命令验证是否已经安装成功，看到下图所示内容则表示安装成功：

![20191112182642](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20201118133606.jpg) 

------

**至此，JDK在CentOS中的安装结束。**
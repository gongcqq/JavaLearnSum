### 1.Kubernetes简介

#### 1.1 Kubernetes是什么

1. Kubernetes，简称k8s，是用8代替八个字符"ubernete"而成的缩写；
2. Kubernetes是一个开源的容器编排引擎，用来对容器化应用进行自动化部署、扩缩和管理，Kubernetes的目标是让部署容器化的应用简单并且高效；
3. Kubernetes是一个可移植、可扩展的开源平台，用于管理容器化的工作负载和服务，可促进声明式配置和自动化。它还拥有一个庞大且快速增长的生态系统，Kubernetes的服务、支持和工具广泛可用。

#### 1.2 应用部署方式的演进

![container](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210301114626.jpg) 

##### 传统部署时代：

早期，各个组织机构在物理服务器上运行应用程序。无法为物理服务器中的应用程序定义资源边界，这会导致资源分配的问题。例如，如果多个应用程序在一个物理服务器上运行，那么在某些实例中，一个应用程序可能会占用大部分资源，从而导致其他应用程序的性能不佳。

解决这个问题的方案是在不同的物理服务器上运行每个应用程序。但是由于资源利用不足而无法扩展， 并且维护许多物理服务器的成本很高。

##### 虚拟化部署时代：

为了解决传统部署方式带来的问题，引入了虚拟化。虚拟化技术允许你在单个物理服务器的CPU上运行多个虚拟机（VM）。 虚拟化可以实现应用程序在虚拟机之间的隔离，并在一个应用程序的信息不能被另一个应用程序自由访问的情况下提供一定的安全级别。

虚拟化技术能够更好地利用物理服务器上的资源，并具有更好的可伸缩性，因为应用程序可以很容易地添加或更新，降低硬件成本等等，通过虚拟化，你可以将一组物理资源表示为一次性虚拟机集群。

每个VM都是一台完整的机器，在虚拟硬件之上运行所有组件，包括它自己的操作系统。

##### 容器部署时代：

容器类似于VM，但是它们放宽了隔离属性，以便在应用程序之间共享操作系统（OS）。因此，容器被认为是轻量级的。与VM类似，容器有自己的文件系统、CPU、内存、进程空间等等。由于它们与底层基础架构分离，因此可以跨云和OS发行版本进行移植。

容器之所以变得流行，是因为它具有很多优秀，比如：

- 应用程序创建和部署的敏捷性：与使用VM镜像相比，容器镜像的创建更加简单和高效；
- 持续开发、集成和部署：通过快速有效的回滚(由于镜像的不可变性)，提供可靠和频繁的容器镜像构建和部署；
- 关注开发与运维的分离：在构建/发布时(而不是在部署时)创建应用程序容器镜像，从而将应用程序与基础架构分离；
- 可观察性：不仅可以显示操作系统级别的信息和指标，还可以显示应用程序的运行状况和其他指标信号；
- 跨开发、测试和生产的环境一致性：在便携式计算机上运行与在云中运行相同；
- 跨云和操作系统发行版本的可移植性：可在Ubuntu、RHEL、CoreOS、本地、Google Kubernetes Engine和其他任何地方运行；
- 以应用程序为中心的管理：提高抽象级别，从在虚拟硬件上运行OS到使用逻辑资源在OS上运行应用程序；
- 松散耦合、分布式、弹性、解放的微服务：应用程序被分解成较小的独立部分，并且可以动态部署和管理，而不是在一台大型单机上整体运行；
- 资源隔离：可预测的应用程序性能；
- 资源利用：高效率和高密度。

#### 1.3 Kubernetes的作用

容器是打包和运行应用程序的良好方式。在生产环境中，我们需要管理运行应用程序的容器，并确保不会出现停机。假设一个容器发生了故障，则需要启动另一个容器，如果能够通过系统自动完成该操作，就会变得更容易。而Kubernetes就可以解决这一问题。

Kubernetes为我们提供了一个灵活运行分布式系统的框架。它负责应用程序的扩展和故障转移，提供部署模式等。例如，Kubernetes可以轻松地管理系统的Canary部署。

Kubernetes可以为我们提供以下功能：

- **批处理**

  Kubernetes可以提供一次性任务、定时任务，满足批量数据处理和分析的场景。

- **服务发现和负载均衡**

  Kubernetes可以使用DNS名称或自己的IP地址公开容器。如果进入到容器的流量很高，Kubernetes能够负载平衡并分发网络流量，从而使部署稳定。

- **存储编排**

  Kubernetes允许我们自动挂载我们选择的存储系统，例如本地存储、公共云提供商等等。

- **自动部署和回滚**

  我们可以使用Kubernetes为部署的容器描述所需的状态，它可以以可控的速度将实际状态更改为所需状态。例如，我们可以自动化Kubernetes为我们的部署创建新的容器，删除现有的容器并将其所有资源用到新容器中。

- **自动装箱**

  假设为Kubernetes提供了一个节点集群，Kubernetes可以使用这些节点来运行容器化的任务。告诉Kubernetes每个容器需要多少CPU和内存(RAM)。Kubernetes可以将容器放到我们的节点上，并充分利用我们的资源。

- **自我修复**

  Kubernetes会重启失败的容器、替换容器、杀死不响应用户定义的健康检查的容器，并且在它们准备好提供服务之前不会将它们通告给客户端。

- **密钥与配置管理**

  Kubernetes允许我们存储和管理敏感信息，例如密码、OAuth令牌和SSH密钥。我们可以在不重建容器镜像的情况下部署和更新密钥以及应用程序配置，也无需在堆栈配置中暴露密钥。

#### 1.4 Kubernetes的架构

当我们部署完Kubernetes, 即拥有了一个完整的集群。一个Kubernetes集群由一组被称作节点的机器组成。这些节点上运行着Kubernetes所管理的容器化应用。集群应至少拥有一个工作节点。

##### 1.4.1 Kubernetes的集群节点

- **Master Node**

  该节点是Kubernetes的集群控制节点，对集群进行调度管理，接受集群外用户到集群的操作请求。Master Node由==API Server==、==Scheduler==、==etcd==、==Controller-Manager==组成。

- **Worker Node**

  该节点是集群的工作节点，节点上运行着用户的业务应用容器。Worker Node包括==kubelet==、==kube-proxy==以及容器运行环境(==Container Runtime==)，比如docker就是一种容器运行环境。

##### 1.4.2 Kubernetes架构图

以下是Kubernetes的架构图：

![k8s架构图](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210302094338.jpg) 

上图中涉及到的各个组件的含义，下文会有详细讲解。

### 2.Kubernetes集群的搭建

#### 2.1 前置知识点

目前生产部署Kubernetes集群主要有两种方式：

- **kubeadm**

  Kubeadm是一个Kubernetes部署工具，提供`kubeadm init`和`kubeadm join`来快速部署Kubernetes集群。具体详情也可以访问[官方地址](https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/)。

- **二进制包**

  我们可以从github下载发行版的二进制包，手动部署每个组件，组成Kubernetes集群。

> Kubeadm方式降低了部署的门槛，但屏蔽了很多细节，遇到问题很难排查。如果想要更加可控，还是推荐使用二进制包方式部署Kubernetes集群，虽然手动部署麻烦点，但是期间可以学习到很多的工作原理，也利于后期维护。

#### 2.2 kubeadm方式搭建k8s集群(单master)

kubeadm是官方社区推出的一个用于快速部署k8s集群的工具，这个工具能通过两条指令完成一个k8s集群的部署：

1. 使用`kubeadm init`创建一个Master节点；
2. 使用`kubeadm join <Master节点的IP和端口>`将Node节点加入到当前集群中。

##### 2.2.1 安装要求

- 一台或多台机器，操作系统为CentOS7；

- 硬件配置：2GB或更多RAM，2个CPU或更多CPU；

- 集群中所有机器之间网络互通；

- 可以访问外网，因为需要联网拉取镜像；

- 需要禁止swap分区。

##### 2.2.2 最终目标

1. 在所有节点上安装Docker和kubeadm；
2. 部署Kubernetes Master；
3. 部署容器网络插件；
4. 部署Kubernetes Node，将节点加入到Kubernetes集群中；
5. 部署Dashboard Web页面，可视化查看Kubernetes资源。

##### 2.2.3 主机规划

| 角色   | IP            |
| ------ | ------------- |
| master | 192.168.68.11 |
| node1  | 192.168.68.12 |
| node2  | 192.168.68.13 |

##### 2.2.4 系统初始化

###### 2.2.4.1 关闭防火墙

```shell
#临时关闭
systemctl stop firewalld

#永久关闭
systemctl disable firewalld
```

###### 2.2.4.2 关闭 selinux

```shell
#临时关闭
setenforce 0

#永久关闭
sed -i 's/enforcing/disabled/' /etc/selinux/config
```

###### 2.2.4.3 关闭swap

```shell
#临时关闭
swapoff -a

#永久关闭
sed -ri 's/.*swap.*/#&/' /etc/fstab 
```

###### 2.2.4.4 设置主机名

```shell
#在192.168.68.11主机上执行以下命令
hostnamectl set-hostname master

#在192.168.68.12主机上执行以下命令
hostnamectl set-hostname node1

#在192.168.68.13主机上执行以下命令
hostnamectl set-hostname node2
```

###### 2.2.4.5 在master中添加hosts

```shell
cat >> /etc/hosts << EOF
192.168.68.11 master
192.168.68.12 node1
192.168.68.13 node2
EOF
```

###### 2.2.4.6 将桥接的IPv4流量传递到iptables的链

```shell
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

#生效命令
sysctl --system
```

###### 2.2.4.7 时间同步

```shell
yum install -y ntpdate
ntpdate -u time.windows.com
```

> **注意**：以上系统初始化涉及到的命令，如无特殊说明，均需在所有主机上执行。

##### 2.2.5 安装Docker、kubeadm、kubelet

Kubernetes默认的容器运行环境为docker，因此要先安装Docker。我这边所有的主机上之前都安装过docker了，所以docker的安装这里就不再进行介绍了。

###### 2.2.5.1 添加阿里云YUM软件源

```shell
#对所有主机执行以下操作命令
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

###### 2.2.5.2 安装kubeadm，kubelet和kubectl

由于版本更新频繁，所以这里指定版本号进行安装部署：

```shell
#对所有主机执行安装命令
yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0

#对所有主机执行以下命令，让kubelet开机启动
systemctl enable kubelet
```

##### 2.2.6 部署Kubernetes Master

```bash
#在master主机上执行以下命令
kubeadm init --apiserver-advertise-address=192.168.68.11 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.18.0 --service-cidr=10.96.0.0/12 --pod-network-cidr=10.244.0.0/16
```

**命令中的参数含义如下：**

![kubeadm-init](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210304172944.png)    

命令的执行需要一段时间(期间可以重新开个窗口，使用`docker images`查看镜像下载的情况)，当出现如下内容时，说明已经执行成功了。

![image-20210302154448491](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210302154452.png)  

这个图中也有接下来需要我们执行的步骤，这些步骤下面也会说到。

==注意==：执行上面的`kubeadm init`那一长串命令的时候，也有可能会报下面的错：

![image-20210305220911407](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210305221028.png) 

如果真的报上图的错的话，依次执行以下命令后重启主机，然后再次执行`kubeadm init`那一长串命令即可。

```shell
#先使用cat命令查看ip_forward的值是否为0，如果为0就继续执行下面的命令
cat /proc/sys/net/ipv4/ip_forward

#上面的值如果为0的话，这里给改成1即可
echo "1" > /proc/sys/net/ipv4/ip_forward

#重启网络服务
service network restart

#最后为了保险起见，可以使用reboot命令重启主机
reboot
```


> 由于默认拉取镜像的地址(k8s.gcr.io)国内无法访问，所以`kubeadm init`命令后面的image-repository参数的值使用的是阿里云镜像仓库地址。

**使用kubectl工具：**

```shell
#在master节点依次执行以下命令
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

上面的命令执行完成之后，我们也可以通过`kubectl get nodes`命令进行查看，会显示如下内容：

```shell
AME     STATUS     ROLES    AGE   VERSION
master   NotReady   master   11m   v1.18.0
```

##### 2.2.7 加入Kubernetes Node

```shell
#在每个worker node主机上执行以下命令，每个人的这个命令都是不一样的，我们在执行完kubeadm init命令后，会输出下面这个命令，直接复制即可
kubeadm join 192.168.68.11:6443 --token ba295y.k5pqeuidfvqrrx2x \
    --discovery-token-ca-cert-hash sha256:0b7edbb6ebe7adf0a67f6a5ec7b8d8493263440f406628327800d9bafbe6c4f9
```

==注意==：在执行`kubeadm join`命令的时候，如果也出现执行`kubeadm init`命令时候的错误，那就还是按照上面提到的解决方案进行解决即可。

以上命令的token默认有效期为24小时，当过期之后，该token就不可用了。如果需要重新创建token，操作如下：

```shell
#在master节点执行以下命令
kubeadm token create --print-join-command

#我们也可以在master节点执行以下命令来查看token信息
kubeadm token list
```

当我们使用`kubeadm join`命令向集群中加入新节点后，在master主机上再次使用`kubectl get nodes`命令，可以发现节点已经添加成功了，如下所示：

```shell
NAME     STATUS     ROLES    AGE   VERSION
master   NotReady   master   44m   v1.18.0
node1    NotReady   <none>   15m   v1.18.0
node2    NotReady   <none>   14m   v1.18.0
```

##### 2.2.8 部署CNI网络插件

```shell
#我们可以使用如下命令来部署CNI网络插件(以下命令要在master主机执行)
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

如果出现"Unable to connect to the server"错误，那说明该地址已经被墙了，如果不知道怎么科学上网的话，可以使用如下命令获取并执行我翻墙下载的这个配置文件：

```shell
#在master主机上下载压缩包
wget https://files.cnblogs.com/files/gongcqq/kube-flannel.tar.gz

#解压压缩包，以便获取kube-flannel.yml文件
tar -zxvf kube-flannel.tar.gz

#执行kube-flannel.yml文件。如果网络环境不好，可以按照下面"注意事项"中把相关镜像先下载下来，再执行该文件
kubectl apply -f kube-flannel.yml
```

以上命令执行完成之后，再在master主机使用`kubectl get nodes`命令进行查看，发现节点状态都从之前的==NotReady==变成了==Ready==，如下所示：

```shell
NAME     STATUS   ROLES    AGE    VERSION
master   Ready    master   139m   v1.18.0
node1    Ready    <none>   110m   v1.18.0
node2    Ready    <none>   110m   v1.18.0
```

我们也可以在master主机上使用`kubectl get pods -n kube-system`命令查看pod的运行情况，查询结果如下：

![image-20210302174746085](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210302174748.png) 

**注意事项**：上面在使用`kubectl apply -f kube-flannel.yml`命令的时候会拉取一个flannel镜像，如果网络环境不好，可能会导致拉取失败，从而导致集群部署失败。我这边已经将该镜像打成了名为[flannel-v0.13.1-rc2.tar](https://gongsl.lanzous.com/iYJpGmkeiaj)的tar包，我们可以下载下来通过ftp的方式传到集群的各个主机上，假设都传到了主机的"/root"目录下，那么直接使用如下命令还原镜像即可：

```shell
docker load -i /root/flannel-v0.13.1-rc2.tar
```

##### 2.2.9 测试kubernetes集群

在Kubernetes集群中创建一个pod，验证是否正常运行：

```shell
#在master主机上创建并运行一个pod
kubectl create deployment nginx --image=nginx

#暴露端口
kubectl expose deployment nginx --port=80 --type=NodePort
```

以上命令执行完成之后，我们可以通过`kubectl get pod,svc`命令查看pod的运行情况：

![image-20210302181219698](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210302181739.png) 

由上图可知，刚才创建的pod正在运行中，且nginx暴露给外部访问的端口是**30412**，如果在浏览器中使用集群中任何一个主机加这个端口都能访问到nginx的话，就说明集群搭建成功了。我使用集群中的三个主机加上端口分别进行了访问，都是可以访问到nginx的，说明kubernetes集群搭建成功了。

```bash
http://192.168.68.11:30412
http://192.168.68.12:30412
http://192.168.68.13:30412
```

![image-20210302182217942](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210302182224.png) 

##### 2.2.10 部署Dashboard Web页面

**注意**：部署Dashboard Web页面章节涉及的所有命令均是在master节点上执行的。

1. 获取recommended.yaml文件

```shell
#可以使用如下命令进行下载
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.3/aio/deploy/recommended.yaml
```

以上地址如果访问不了的话，可以使用如下方式下载：

```shell
#下载recommended.yaml的压缩包
wget https://files.cnblogs.com/files/gongcqq/recommended.tar.gz

#解压压缩包以获取recommended.yaml文件
tar -zxvf recommended.tar.gz
```

默认Dashboard只能集群内部访问，修改Service为NodePort类型，暴露到外部：

![image-20210306004457104](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210306004502.png) 

> **注意**：如果是通过上面第二个地址中的recommended.tar.gz获取到的recommended.yaml文件的话，就不用再增加NodePort类型了，因为压缩包中的recommended.yaml文件是已经增加过的。

2. 执行recommended.yaml文件

```shell
kubectl apply -f recommended.yaml
```

执行完成后，可以通过`kubectl get pods -n kubernetes-dashboard`命令进行状态查看，下图就是正常状态：

![image-20210306004943314](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210306004947.png) 

3. 创建用户并生成token

```shell
#创建用户
kubectl create serviceaccount dashboard-admin -n kube-system

#用户授权
kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin

#生成用户token
kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
```

下图就是用户的token信息：

![image-20210306005317155](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210306005320.png) 

4. 登录Dashboard页面

浏览器输入集群中任一主机ip加30001端口都可访问到Dashboard页面，这里以master主机为例，如下所示：

```http
https://192.168.68.11:30001/
```

进入到登录页面后，需要填写用户token，填写完成后，直接登录即可：

![image-20210306010103826](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210306010108.png) 

下面就是登录成功后的主界面：

![image-20210306010249999](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210306010255.png) 

#### 2.3 kubeadm方式搭建k8s集群(多master)

##### 2.3.1 安装要求

- 一台或多台机器，操作系统为CentOS7；

- 硬件配置：2GB或更多RAM，2个CPU或更多CPU；

- 集群中所有机器之间网络互通；

- 可以访问外网，因为需要联网拉取镜像；

- 需要禁止swap分区。

##### 2.3.2 主机规划

| 角色    | IP            |
| ------- | ------------- |
| master1 | 192.168.68.11 |
| master2 | 192.168.68.12 |
| node1   | 192.168.68.21 |
| node2   | 192.168.68.22 |

##### 2.3.3 系统初始化

###### 2.3.3.1 关闭防火墙

```shell
#临时关闭
systemctl stop firewalld

#永久关闭
systemctl disable firewalld
```

###### 2.3.3.2 关闭 selinux

```shell
#临时关闭
setenforce 0

#永久关闭
sed -i 's/enforcing/disabled/' /etc/selinux/config
```

###### 2.3.3.3 关闭swap

```shell
#临时关闭
swapoff -a

#永久关闭
sed -ri 's/.*swap.*/#&/' /etc/fstab 
```

###### 2.3.3.4 重置iptables

```shell
#重置iptables
iptables -F && iptables -X && iptables -F -t nat && iptables -X -t nat && iptables -P FORWARD ACCEPT
```

###### 2.3.3.5 设置主机名

```shell
#在192.168.68.11主机上执行以下命令
hostnamectl set-hostname master1

#在192.168.68.12主机上执行以下命令
hostnamectl set-hostname master2

#在192.168.68.21主机上执行以下命令
hostnamectl set-hostname node1

#在192.168.68.22主机上执行以下命令
hostnamectl set-hostname node2
```

###### 2.3.3.6 配置hosts

```shell
cat >> /etc/hosts << EOF
192.168.68.11 master1
192.168.68.12 master2
192.168.68.21 node1
192.168.68.22 node2
EOF
```

###### 2.3.3.7 系统参数设置

```shell
#制作配置文件
cat > /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
vm.swappiness=0
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
EOF

#使文件生效
sysctl -p /etc/sysctl.d/kubernetes.conf
```

###### 2.3.3.8 时间同步

```shell
yum install -y ntpdate
ntpdate -u time.windows.com
```

> **注意**：以上系统初始化涉及到的命令，如无特殊说明，均需在所有主机上执行。

##### 2.3.4 安装必要工具(所有节点)

###### 2.3.4.1 工具说明

- **kubeadm:**  部署集群用的命令；
- **kubelet:** 在集群中每台机器上都要运行的组件，负责管理pod、容器的生命周期；
- **kubectl:** 集群管理工具（可选，只要在控制集群的节点上安装即可）。

###### 2.3.4.2 安装步骤

```shell
#配置yum源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

#找到要安装的版本号
yum list kubeadm --showduplicates | sort -r

#这里用的是1.18.0
yum install -y kubeadm-1.18.0 kubelet-1.18.0 kubectl-1.18.0 --disableexcludes=kubernetes

#让kubelet开机启动
systemctl enable kubelet
```

##### 2.3.5 准备配置文件(任意节点)

###### 2.3.5.1 下载配置文件

```shell
#下载压缩包(我这边就下载到master1主机的/root路径下)
wget https://files.cnblogs.com/files/gongcqq/kubernetes-ha-kubeadm.tar.gz

#解压压缩包
tar -zxvf kubernetes-ha-kubeadm.tar.gz

#目录结构如下：
kubernetes-ha-kubeadm
├── addons
│   ├── calico-rbac-kdd.yaml
│   ├── calico.yaml
│   └── dashboard-all.yaml
├── configs
│   ├── keepalived-backup.conf
│   ├── keepalived-master.conf
│   └── kubeadm-config.yaml
├── global-config.properties
├── init.sh
└── scripts
    └── check-apiserver.sh
```

###### 2.3.5.2 目录结构及文件说明

- **addons**

  kubernetes的插件，比如calico和dashboard。

- **configs**

  包含了部署集群过程中用到的各种配置文件。

- **scripts**

  包含部署集群过程中用到的脚本，如keepalive检查脚本。

- **global-configs.properties**

  全局配置，包含各种易变的配置内容。

- **init.sh**

  初始化脚本，配置好global-config.properties之后，可以使用该脚本自动生成所有的配置文件。

###### 2.3.5.3 生成需要的配置

```shell
#进入到下载的配置的目录中
cd kubernetes-ha-kubeadm

#编辑配置属性(根据文件中的注释自定义修改)
vim global-config.properties

#生成配置文件(确保执行过程没有异常信息)
./init.sh

#查看生成的配置文件，确保脚本执行成功
tree target/

#生成的配置文件的目录结构如下：
target/
├── addons
│   ├── calico-rbac-kdd.yaml
│   ├── calico.yaml
│   └── dashboard-all.yaml
├── configs
│   ├── keepalived-backup.conf
│   ├── keepalived-master.conf
│   └── kubeadm-config.yaml
└── scripts
    └── check-apiserver.sh
```

##### 2.3.6 部署keepalived-apiserver高可用

###### 2.3.6.1 安装keepalived

```shell
#在两个master节点上安装keepalived(一主一备)
yum install -y keepalived
```

###### 2.3.6.2 创建keepalived配置文件

```shell
#在两个master主机上执行以下命令
mkdir -p /etc/keepalived

#分发配置文件，由于配置文件我放到master1主机上了，所以master1主机执行以下操作
cp /root/kubernetes-ha-kubeadm/target/configs/keepalived-master.conf /etc/keepalived/keepalived.conf

#分发配置文件，将master1主机上的配置文件分发到master2主机上(在master1主机上操作)
scp /root/kubernetes-ha-kubeadm/target/configs/keepalived-backup.conf root@192.168.68.12:/etc/keepalived/keepalived.conf 

#分发监测脚本，在master1主机上依次执行下面两条命令
cp /root/kubernetes-ha-kubeadm/target/scripts/check-apiserver.sh /etc/keepalived/
scp /root/kubernetes-ha-kubeadm/target/scripts/check-apiserver.sh root@192.168.68.12:/etc/keepalived/
```

###### 2.3.6.3 启动keepalived

```shell
#分别在master1主机和master2主机上执行以下命令
systemctl enable keepalived && systemctl start keepalived

#检查状态
systemctl status keepalived

#查看日志
journalctl -f -u keepalived
```

##### 2.3.7 部署第一个主节点(master1)

```shell
#进入配置文件目录
cd /root/kubernetes-ha-kubeadm/configs

#执行kubeadm初始化系统(注意保存最后打印的加入集群的命令)
kubeadm init --config=kubeadm-config.yaml --upload-certs

#依次执行以下命令(上一步会有提示)
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 测试一下kubectl
kubectl get pods --all-namespaces

# *****这里备份一下上面执行kubeadm init时打印的join命令*****
kubeadm join 192.168.68.31:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:ee2cd31271fedd30fc76599eb5dc842f439af01032e418b26e5aceaeeb6f0e8a
```

##### 2.3.8 部署网络插件-calico

```shell
#在master1主机上创建工作目录
mkdir -p /etc/kubernetes/addons

#拷贝文件到工作目录下
cp /root/kubernetes-ha-kubeadm/target/addons/calico* /etc/kubernetes/addons/

#部署calico
kubectl apply -f /etc/kubernetes/addons/calico-rbac-kdd.yaml
kubectl apply -f /etc/kubernetes/addons/calico.yaml
```

#### 2.4 ~~二进制方式搭建k8s集群~~

##### 2.4.1 安装要求

- 一台或多台机器，操作系统为CentOS7；

- 硬件配置：2GB或更多RAM，2个CPU或更多CPU；

- 集群中所有机器之间网络互通；

- 可以访问外网，因为需要联网拉取镜像；

- 需要禁止swap分区。

##### 2.4.2 主机规划

| 角色   | IP            |
| ------ | ------------- |
| master | 192.168.68.11 |
| node1  | 192.168.68.12 |
| node2  | 192.168.68.13 |

##### 2.4.3 系统初始化

```shell
#关闭防火墙
systemctl stop firewalld     #临时关闭
systemctl disable firewalld  #永久关闭

#关闭 selinux
setenforce 0                                        #临时关闭
sed -i 's/enforcing/disabled/' /etc/selinux/config  #永久关闭

#关闭swap
swapoff -a                           #临时关闭
sed -ri 's/.*swap.*/#&/' /etc/fstab  #永久关闭

#设置主机名
hostnamectl set-hostname master #在192.168.68.11主机上执行以下命令
hostnamectl set-hostname node1  #在192.168.68.12主机上执行以下命令
hostnamectl set-hostname node2  #在192.168.68.13主机上执行以下命令

#在master主机中添加hosts
cat >> /etc/hosts << EOF
192.168.68.11 master
192.168.68.12 node1
192.168.68.13 node2
EOF

#将桥接的IPv4流量传递到iptables的链
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system  #生效命令

#时间同步
yum install -y ntpdate
ntpdate -u time.windows.com
```

> **注意**：以上系统初始化涉及到的命令，如无特殊说明，均需在所有主机上执行。

##### 2.4.4 部署Etcd集群

Etcd是一个分布式键值存储系统，Kubernetes使用Etcd进行数据存储，所以先准备一个Etcd数据库，为解决Etcd单点故障，应采用集群方式部署。这里使用3台机器组建集群，可容忍1台机器故障。当然，也可以使用5台机器组建集群，可容忍2台机器故障。

| 节点名称 | IP            |
| -------- | ------------- |
| etcd-1   | 192.168.68.11 |
| etcd-2   | 192.168.68.12 |
| etcd-3   | 192.168.68.13 |

> **注意**：这里为了节省机器，就与K8s节点机器复用了。也可以独立于k8s集群之外用新的主机部署，只要apiserver能连接到就行。

###### 2.4.4.1 准备cfssl证书生成工具

cfssl是一个开源的证书管理工具，使用json文件生成证书，相比openssl更方便使用。可以找任意一台机器操作，这里在Master主机(192.168.68.11)上操作。下面生成证书以及部署集群的步骤都是在master主机上操作的，后面会再把相关文件拷贝到其他主机上。

```shell
#默认情况下，使用下面的三条命令进行下载即可，但是下载速度很慢，所以这里就不通过这种方式了
#wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
#wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
#wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
```

```shell
#我已经提前把cfssl相关工具下载并压缩到压缩包中了，压缩包已经上传到网上，所以直接使用如下地址下载即可
wget https://files.cnblogs.com/files/gongcqq/cfssl.tar.gz

#解压并进入相应目录中
tar -zxvf cfssl.tar.gz && cd cfssl

#赋权
chmod +x cfssl_linux-amd64 cfssljson_linux-amd64 cfssl-certinfo_linux-amd64

#移动到相应路径
mv cfssl_linux-amd64 /usr/local/bin/cfssl
mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
mv cfssl-certinfo_linux-amd64 /usr/bin/cfssl-certinfo
```

###### 2.4.4.2 生成Etcd证书

1. 自签证书颁发机构(CA)

```shell
#创建并进入工作目录
mkdir -p ~/TLS/{etcd,k8s} && cd ~/TLS/etcd

#自签CA
cat > ca-config.json<< EOF
{
    "signing": {
        "default": {
            "expiry": "87600h"
        },
        "profiles": {
            "www": {
                "expiry": "87600h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
EOF

cat > ca-csr.json<< EOF
{
    "CN": "etcd CA",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing"
        }
    ]
}
EOF

#生成证书
cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
```

2. 使用自签CA签发Etcd HTTPS证书

```shell
#创建证书申请文件，下面的主机地址要改成自己的主机地址
cat > server-csr.json<< EOF
{
    "CN": "etcd",
    "hosts": [
        "192.168.68.11",
        "192.168.68.12",
        "192.168.68.13"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "BeiJing",
            "ST": "BeiJing"
        }
    ]
}
EOF

#生成证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www server-csr.json | cfssljson -bare server
```

> **注意**：上述命令的hosts字段中的IP为所有etcd节点的集群内部通信IP，一个都不能少，为了方便后期扩容，也可以多写几个预留的IP。

###### 2.4.4.3 部署Etcd集群

1. 创建工作目录并解压二进制包

```shell
#创建工作目录
mkdir -p /opt/etcd/{bin,cfg,ssl}

#从Github下载二进制文件
wget https://github.com/etcd-io/etcd/releases/download/v3.4.9/etcd-v3.4.9-linux-amd64.tar.gz

#解压文件
tar -zxvf etcd-v3.4.9-linux-amd64.tar.gz

#移动相应文件到对应的工作目录下
mv etcd-v3.4.9-linux-amd64/{etcd,etcdctl} /opt/etcd/bin/
```

2. 创建etcd配置文件

```shell
cat > /opt/etcd/cfg/etcd.conf << EOF
#[Member]
ETCD_NAME="etcd-1"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.68.11:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.68.11:2379"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.68.11:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.68.11:2379"
ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.68.11:2380,etcd-2=https://192.168.68.12:2380,etcd-3=https://192.168.68.13:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF
```

- `ETCD_NAME`：节点名称，集群中唯一；
- `ETCD_DATA_DIR`：数据目录；
- `ETCD_LISTEN_PEER_URLS`：集群通信监听地址；
- `ETCD_LISTEN_CLIENT_URLS`：客户端访问监听地址；
- `ETCD_INITIAL_ADVERTISE_PEER_URLS`：集群通告地址；
- `ETCD_ADVERTISE_CLIENT_URLS`：客户端通告地址；
- `ETCD_INITIAL_CLUSTER`：集群节点地址；
- `ETCD_INITIAL_CLUSTER_TOKEN`：集群Token；
- `ETCD_INITIAL_CLUSTER_STATE`：加入集群的当前状态，new是新集群，existing表示加入已有集群。

3. 拷贝前面生成的证书

```shell
cp ~/TLS/etcd/ca*pem ~/TLS/etcd/server*pem /opt/etcd/ssl/
```

4. systemd管理etcd

```shell
cat > /usr/lib/systemd/system/etcd.service << EOF
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
[Service]
Type=notify
EnvironmentFile=/opt/etcd/cfg/etcd.conf
ExecStart=/opt/etcd/bin/etcd \
--cert-file=/opt/etcd/ssl/server.pem \
--key-file=/opt/etcd/ssl/server-key.pem \
--peer-cert-file=/opt/etcd/ssl/server.pem \
--peer-key-file=/opt/etcd/ssl/server-key.pem \
--trusted-ca-file=/opt/etcd/ssl/ca.pem \
--peer-trusted-ca-file=/opt/etcd/ssl/ca.pem \
--logger=zap
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF
```

5. 将master主机上生成的文件拷贝到其余主机上

```shell
#复制文件或目录到192.168.68.12主机，复制目录要加-r参数，复制文件的话不用加
scp -r /opt/etcd/ root@192.168.68.12:/opt/
scp /usr/lib/systemd/system/etcd.service root@192.168.68.12:/usr/lib/systemd/system/

#复制文件或目录到192.168.68.13主机
scp -r /opt/etcd/ root@192.168.68.13:/opt/
scp /usr/lib/systemd/system/etcd.service root@192.168.68.13:/usr/lib/systemd/system/
```

**复制完成之后，我们需要分别修改一下etcd.conf文件。**

- 在192.168.68.12主机上分别执行如下操作：

```shell
#删除复制过来的etcd.conf文件
rm -f /opt/etcd/cfg/etcd.conf

#重新生成一个etcd.conf文件，注意下面有些主机IP要改成本机IP，不能再是master主机的IP
cat > /opt/etcd/cfg/etcd.conf << EOF
#[Member]
ETCD_NAME="etcd-2"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.68.12:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.68.12:2379"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.68.12:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.68.12:2379"
ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.68.11:2380,etcd-2=https://192.168.68.12:2380,etcd-3=https://192.168.68.13:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF
```

- 在192.168.68.13主机上分别执行如下操作：

```shell
#删除复制过来的etcd.conf文件
rm -f /opt/etcd/cfg/etcd.conf

#重新生成一个etcd.conf文件，注意下面有些主机IP要改成本机IP，不能再是master主机的IP
cat > /opt/etcd/cfg/etcd.conf << EOF
#[Member]
ETCD_NAME="etcd-3"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.68.13:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.68.13:2379"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.68.13:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.68.13:2379"
ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.68.11:2380,etcd-2=https://192.168.68.12:2380,etcd-3=https://192.168.68.13:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF
```

6. 启动etcd并设置开机启动

```shell
#重载daemon
systemctl daemon-reload

#启动etcd
systemctl start etcd

#设置开机启动
systemctl enable etcd

#运行状态查看
systemctl status etcd
```

**注意**：需要在集群中的所有主机上都执行以上三条命令。而且在启动etcd的时候，要先启动worker node上的etcd，最后再启动master上的etcd。还是启动不起来的话，电脑就切换下网络或者重启虚拟机。

7. 查看集群状态

```shell
#在集群的任一启动etcd的主机上执行如下命令
ETCDCTL_API=3 /opt/etcd/bin/etcdctl \
--cacert=/opt/etcd/ssl/ca.pem \
--cert=/opt/etcd/ssl/server.pem \
--key=/opt/etcd/ssl/server-key.pem \
--endpoints="https://192.168.68.11:2379,https://192.168.68.12:2379,https://192.168.68.13:2379" endpoint health
```

- 如果输出如下信息，说明集群中的节点都是健康的：

![image-20210303161728305](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210303162624.png) 

- 假设将192.168.68.13主机上的etcd停掉，然后在集群中另两台主机的任一主机上执行以上命令，会打印如下信息：

![image-20210303162620114](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210303162658.png) 

> 如果我们想要查看etcd的日志的话，可以使用`journalctl -u etcd`命令，也可以使用`more /var/log/messages`命令。

##### 2.4.5 部署Master Node

由于这里搭建k8s用到的三台主机我之前都已经安装过了docker，所以这里就不再演示docker的安装步骤了。如果没有安装的话，这里需要先安装docker，然后再部署Master Node。

###### 2.4.5.1 生成kube-apiserver证书

1. 自签证书颁发机构(CA)

```shell
#在master节点创建并进入工作目录
mkdir -p ~/TLS/k8s/apiserver && cd ~/TLS/k8s/apiserver

#自签CA
cat > ca-config.json<< EOF
{
    "signing": {
        "default": {
            "expiry": "87600h"
        },
        "profiles": {
            "kubernetes": {
                "expiry": "87600h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
EOF

cat > ca-csr.json<< EOF
{
    "CN": "kubernetes",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF

#生成证书
cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
```

2. 使用自签CA签发kube-apiserver HTTPS证书

```shell
#创建证书申请文件，下面的主机地址要改成自己的主机地址
cat > server-csr.json<< EOF
{
    "CN": "kubernetes",
    "hosts": [
        "10.0.0.1",
        "127.0.0.1",
        "192.168.68.11",
        "192.168.68.12",
        "192.168.68.13",
        "kubernetes",
        "kubernetes.default",
        "kubernetes.default.svc",
        "kubernetes.default.svc.cluster",
        "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "BeiJing",
            "ST": "BeiJing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF

#生成证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes server-csr.json | cfssljson -bare server
```

###### 2.4.5.2 下载并解压二进制包

我们可以通过[github](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.18.md#v1183)下载二进制包，直接点击下载kubernetes-server-linux-amd64.tar.gz即可。

![未标题-1](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210304172956.jpg) 

```shell
#创建工作目录
mkdir -p /opt/kubernetes/{bin,cfg,ssl,logs}

#将从Github下载的二进制包通过ftp的方式放到主机的/root/TLS/k8s/apiserver路径下，并进入该目录
cd /root/TLS/k8s/apiserver

#解压
tar -zxvf kubernetes-server-linux-amd64.tar.gz

#解压后，通过如下命令进入到解压后的目录的bin路径下
cd kubernetes/server/bin

#然后执行如下的拷贝命令
cp kube-apiserver kube-scheduler kube-controller-manager /opt/kubernetes/bin
cp kubectl /usr/bin
```

###### 2.4.5.3 部署kube-apiserver

1. 创建配置文件

**注意**：下面的两个`\\`第一个是转义符，第二个是换行符，使用转义符是为了使用EOF时保留换行符。

```shell
cat > /opt/kubernetes/cfg/kube-apiserver.conf << EOF
KUBE_APISERVER_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/opt/kubernetes/logs \\
--etcd-servers=https://192.168.68.11:2379,https://192.168.68.12:2379,https://192.168.68.13:2379 \\
--bind-address=192.168.68.11 \\
--secure-port=6443 \\
--advertise-address=192.168.68.11 \\
--allow-privileged=true \\
--service-cluster-ip-range=10.0.0.0/24 \\
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota,NodeRestriction \\
--authorization-mode=RBAC,Node \\
--enable-bootstrap-token-auth=true \\
--token-auth-file=/opt/kubernetes/cfg/token.csv \\
--service-node-port-range=30000-32767 \\
--kubelet-client-certificate=/opt/kubernetes/ssl/server.pem \\
--kubelet-client-key=/opt/kubernetes/ssl/server-key.pem \\
--tls-cert-file=/opt/kubernetes/ssl/server.pem \\
--tls-private-key-file=/opt/kubernetes/ssl/server-key.pem \\
--client-ca-file=/opt/kubernetes/ssl/ca.pem \\
--service-account-key-file=/opt/kubernetes/ssl/ca-key.pem \\
--etcd-cafile=/opt/etcd/ssl/ca.pem \\
--etcd-certfile=/opt/etcd/ssl/server.pem \\
--etcd-keyfile=/opt/etcd/ssl/server-key.pem \\
--audit-log-maxage=30 \\
--audit-log-maxbackup=3 \\
--audit-log-maxsize=100 \\
--audit-log-path=/opt/kubernetes/logs/k8s-audit.log"
EOF
```

- `logtostderr`：启用日志；
- `v`：日志等级；
- `log-dir`：日志目录；
- `etcd-servers`：etcd集群地址；
- `bind-address`：监听地址；
- `secure-port`：https安全端口；
- `advertise-address`：集群通告地址；
- `allow-privileged`：启用授权；
- `service-cluster-ip-range`：Service虚拟IP地址段；
- `enable-admission-plugins`：准入控制模块；
- `authorization-mode`：认证授权，启用RBAC授权和节点自管理；
- `enable-bootstrap-token-auth`：启用TLS bootstrap机制；
- `token-auth-file`：bootstrap token文件；
- `service-node-port-range`：Service nodeport类型默认分配端口范围；
- `kubelet-client-xxx`：apiserver访问kubelet客户端证书；
- `tls-xxx-file`：apiserver https证书；
- `etcd-xxxfile`：连接Etcd集群证书；
- `audit-log-xxx`：审计日志。

2. 关于TLS Bootstrapping机制

上面创建的kube-apiserver.conf文件中包含了一个TLS bootstrap机制，这里大致介绍一下它。

==TLS Bootstraping==：Master节点上的apiserver在启用TLS认证之后，Node节点上的kubelet和kube-proxy要与kube-apiserver进行通信，必须使用CA签发的有效证书才可以，当Node节点很多时，这种客户端证书颁发需要大量工作，同样也会增加集群扩展复杂度。为了简化流程，Kubernetes引入了TLS bootstraping机制来自动颁发客户端证书，kubelet会以一个低权限用户自动向apiserver申请证书，kubelet的证书由apiserver动态签署。所以强烈建议在Node上使用这种方式，目前主要用于kubelet，kube-proxy还是由我们统一颁发一个证书。

**TLS bootstraping工作流程：**

![绘图](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210304173320.jpg)  

3. 创建token文件

```shell
#生成token
head -c 16 /dev/urandom | od -An -t x | tr -d ' '

#创建上面的kube-apiserver.conf文件中包含的token.csv文件
cat > /opt/kubernetes/cfg/token.csv << EOF
dec817590fd3d12f6f5880216c2ec696,kubelet-bootstrap,10001,"system:node-bootstrapper"
EOF

#这里的token需要保存一下，因为在下面部署worker node的时候会用到。
```

> **格式**：token，用户名，UID，用户组

4. 拷贝前面生成的证书

```shell
#把前面生成的证书拷贝到配置文件的路径下
cp ~/TLS/k8s/apiserver/ca*pem ~/TLS/k8s/apiserver/server*pem /opt/kubernetes/ssl/
```

5. systemd管理apiserver

```shell
cat > /usr/lib/systemd/system/kube-apiserver.service << EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=/opt/kubernetes/cfg/kube-apiserver.conf
ExecStart=/opt/kubernetes/bin/kube-apiserver \$KUBE_APISERVER_OPTS
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF
```

6. 启动并设置开机启动

```shell
systemctl daemon-reload
systemctl start kube-apiserver
systemctl enable kube-apiserver
```

###### 2.4.5.4 授权kubelet-bootstrap用户允许请求证书

```shell
kubectl create clusterrolebinding kubelet-bootstrap \
--clusterrole=system:node-bootstrapper \
--user=kubelet-bootstrap
```

###### 2.4.5.5 部署kube-controller-manager

1. 创建配置文件

```shell
cat > /opt/kubernetes/cfg/kube-controller-manager.conf << EOF
KUBE_CONTROLLER_MANAGER_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/opt/kubernetes/logs \\
--leader-elect=true \\
--master=127.0.0.1:8080 \\
--bind-address=127.0.0.1 \\
--allocate-node-cidrs=true \\
--cluster-cidr=10.244.0.0/16 \\
--service-cluster-ip-range=10.0.0.0/24 \\
--cluster-signing-cert-file=/opt/kubernetes/ssl/ca.pem \\
--cluster-signing-key-file=/opt/kubernetes/ssl/ca-key.pem \\
--root-ca-file=/opt/kubernetes/ssl/ca.pem \\
--service-account-private-key-file=/opt/kubernetes/ssl/ca-key.pem \\
--experimental-cluster-signing-duration=87600h0m0s"
EOF
```

- `leader-elect`：当该组件启动多个时，自动选举（HA）；
- `master`：通过本地非安全本地端口8080连接 apiserver；
- `cluster-signing-*-file`：自动为kubelet颁发证书的CA，与apiserver保持一致。

2. systemd管理controller-manager

```shell
cat > /usr/lib/systemd/system/kube-controller-manager.service << EOF
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=/opt/kubernetes/cfg/kube-controller-manager.conf
ExecStart=/opt/kubernetes/bin/kube-controller-manager \$KUBE_CONTROLLER_MANAGER_OPTS
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF
```

3. 启动并设置开机启动

```shell
systemctl daemon-reload
systemctl start kube-controller-manager
systemctl enable kube-controller-manager
```

###### 2.4.5.6 部署kube-scheduler

1. 创建配置文件

```shell
cat > /opt/kubernetes/cfg/kube-scheduler.conf << EOF
KUBE_SCHEDULER_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/opt/kubernetes/logs \\
--leader-elect \\
--master=127.0.0.1:8080 \\
--bind-address=127.0.0.1"
EOF
```

- `leader-elect`：当该组件启动多个时，自动选举（HA）；
- `master`：通过本地非安全本地端口8080连接apiserver。

2. systemd管理scheduler

```shell
cat > /usr/lib/systemd/system/kube-scheduler.service << EOF
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=/opt/kubernetes/cfg/kube-scheduler.conf
ExecStart=/opt/kubernetes/bin/kube-scheduler \$KUBE_SCHEDULER_OPTS
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF
```

3. 启动并设置开机启动

```shell
systemctl daemon-reload
systemctl start kube-scheduler
systemctl enable kube-scheduler
```

###### 2.4.5.7 查看集群状态

所有组件都启动成功后，可以通过kubectl工具的`kubectl get cs`命令查看当前集群组件状态：

![image-20210305091708842](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210305091725.png) 

##### 2.4.6 部署Worker Node

**注意**：部署Worker Node前，要确保所有的Worker Node上都已成功安装了docker，我这边之前都安装过了，所以就不演示具体的安装过程了。

###### 2.4.6.1  创建工作目录并拷贝二进制文件

```shell
#在所有的worker节点上执行以下命令
mkdir -p /opt/kubernetes/{bin,cfg,ssl,logs}

#在master节点上执行以下命令，将二进制文件拷贝到所有的worker节点对应的工作目录下
scp /root/TLS/k8s/apiserver/kubernetes/server/bin/{kubelet,kube-proxy} root@192.168.68.12:/opt/kubernetes/bin/ && scp /root/TLS/k8s/apiserver/kubernetes/server/bin/{kubelet,kube-proxy} root@192.168.68.13:/opt/kubernetes/bin/
```

###### 2.4.6.2 部署kubelet

1. 创建配置文件

```shell
#在node1节点(192.168.68.12)上执行以下命令
cat > /opt/kubernetes/cfg/kubelet.conf << EOF
KUBELET_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/opt/kubernetes/logs \\
--hostname-override=node1 \\
--network-plugin=cni \\
--kubeconfig=/opt/kubernetes/cfg/kubelet.kubeconfig \\
--bootstrap-kubeconfig=/opt/kubernetes/cfg/bootstrap.kubeconfig \\
--config=/opt/kubernetes/cfg/kubelet-config.yml \\
--cert-dir=/opt/kubernetes/ssl \\
--pod-infra-container-image=gongsl/pause-amd64:3.0"
EOF

#在node2节点(192.168.68.13)上执行以下命令
cat > /opt/kubernetes/cfg/kubelet.conf << EOF
KUBELET_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/opt/kubernetes/logs \\
--hostname-override=node2 \\
--network-plugin=cni \\
--kubeconfig=/opt/kubernetes/cfg/kubelet.kubeconfig \\
--bootstrap-kubeconfig=/opt/kubernetes/cfg/bootstrap.kubeconfig \\
--config=/opt/kubernetes/cfg/kubelet-config.yml \\
--cert-dir=/opt/kubernetes/ssl \\
--pod-infra-container-image=gongsl/pause-amd64:3.0"
EOF
```

- `hostname-override`：显示名称，集群中唯一；
- `network-plugin`：启用CNI；
- `kubeconfig`：空路径，会自动生成，后面用于连接apiserver；
- `bootstrap-kubeconfig`：首次启动向apiserver申请证书；
- `config`：配置参数文件；
- `cert-dir`：kubelet证书生成目录；
- `pod-infra-container-image`：管理Pod网络容器的镜像。

2. 配置参数文件

```shell
#在所有worker节点上都执行以下命令
cat > /opt/kubernetes/cfg/kubelet-config.yml << EOF
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 0.0.0.0
port: 10250
readOnlyPort: 10255
cgroupDriver: cgroupfs
clusterDNS: 
  - 10.0.0.2
clusterDomain: cluster.local
failSwapOn: false
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /opt/kubernetes/ssl/ca.pem
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
maxOpenFiles: 1000000
maxPods: 110
EOF
```

3. 拷贝证书文件

```shell
##在master节点上依次执行以下两条命令
scp /opt/kubernetes/ssl/ca.pem root@192.168.68.12:/opt/kubernetes/ssl/
scp /opt/kubernetes/ssl/ca.pem root@192.168.68.13:/opt/kubernetes/ssl/
```

4. 生成bootstrap.kubeconfig文件

- 在master主机上执行以下命令，用于生成bootstrap.kubeconfig文件：

```shell
#设置临时环境变量，可以使用"export -p"命令进行查看
export KUBE_APISERVER="https://192.168.68.11:6443" #master主机上的apiserver IP:PORT
export TOKEN="a6197a27fd379d91ab55107bc1a26af7" #与master主机上的token.csv中的token保持一致

#设置集群参数
kubectl config set-cluster kubernetes \
--certificate-authority=/opt/kubernetes/ssl/ca.pem \
--embed-certs=true \
--server=${KUBE_APISERVER} \
--kubeconfig=bootstrap.kubeconfig

#设置客户端认证参数
kubectl config set-credentials "kubelet-bootstrap" \
--token=${TOKEN} \
--kubeconfig=bootstrap.kubeconfig

#设置上下文参数
kubectl config set-context default \
--cluster=kubernetes \
--user="kubelet-bootstrap" \
--kubeconfig=bootstrap.kubeconfig

#设置默认上下文
kubectl config use-context default --kubeconfig=bootstrap.kubeconfig
```

- 生成后的bootstrap.kubeconfig文件内容如下：

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUR2akNDQXFhZ0F3SUJBZ0lVVjBvVDZMc09kQW1Ed29XLzQ3MENWSzhmaEQwd0RRWUpLb1pJaHZjTkFRRUwKQlFBd1pURUxNQWtHQTFVRUJoTUNRMDR4RURBT0JnTlZCQWdUQjBKbGFXcHBibWN4RURBT0JnTlZCQWNUQjBKbAphV3BwYm1jeEREQUtCZ05WQkFvVEEyczRjekVQTUEwR0ExVUVDeE1HVTNsemRHVnRNUk13RVFZRFZRUURFd3ByCmRXSmxjbTVsZEdWek1CNFhEVEl4TURNd016QTROVEV3TUZvWERUSTJNRE13TWpBNE5URXdNRm93WlRFTE1Ba0cKQTFVRUJoTUNRMDR4RURBT0JnTlZCQWdUQjBKbGFXcHBibWN4RURBT0JnTlZCQWNUQjBKbGFXcHBibWN4RERBSwpCZ05WQkFvVEEyczRjekVQTUEwR0ExVUVDeE1HVTNsemRHVnRNUk13RVFZRFZRUURFd3ByZFdKbGNtNWxkR1Z6Ck1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0NBUUVBM04xR1hTUW1pV2Vpa25aNk1qTm0KajJYTGg4VDJSOU05aHpDZGk4Nk9XVzNaYTk0MDRPQ3h2ODhReUZ3QXl3NkdRK2JUS05aTDhBZHdRMUtzVEwrVAoyVlUwY0pVVXdnZFA3Wks3WXFDVy84VkFKUHdOQUFLSkUwZUt0UmdhRmtDUnhhcFN1MGR1TU1NdUVNYmdKVVFnCmtqZVBvdlp6dHgvMjd0ZHN0Q3VVaXZ6M01KQmxBYlYwejYzd1d4MnJGTTRlMjdwT1M2YlRIVVBRdjUwanArcHMKRHA0eGFKRWo1VTBXU3I1NFJrUnErSEdxemZmNy9iWGxlZ2h3QzVURW1DNVJSRzlFMlMzWWFkTndtK3o3SDFXdAo0cm10RWs1dXR6UlVnWlk5UmpUOFU3RUJRTUF3OVRQZEZhMkl2a25MVEVxQkN2eXNsR055QTREWUJrN091dTR3CkV3SURBUUFCbzJZd1pEQU9CZ05WSFE4QkFmOEVCQU1DQVFZd0VnWURWUjBUQVFIL0JBZ3dCZ0VCL3dJQkFqQWQKQmdOVkhRNEVGZ1FVM1owTmJYZWg2SlRHL0p1bkJiQW5LOW5rRGhNd0h3WURWUjBqQkJnd0ZvQVUzWjBOYlhlaAo2SlRHL0p1bkJiQW5LOW5rRGhNd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFIUHFWODZQMU1OQ09keFVGSFA2CkpQeElhZ0orK3pRc0duRkFLLytwbTdzR0d2eHV2aW5LMGo4R2FENWIyLzg4bjBRSFU3T2xmTnhIRzVIUUJDVUMKK1NRY1FoVGFLSFcyeGdhWDdmdG92dVFlNzh4K0FBSEFuamt4MDJjRmpneHdSRnBQVWFZMUVLNXdwQis5NHhDZwplVndQeHBoZTFwWjBXeGNIb3ZEZFNTcFdNazNRanBBZ3k3QUlURU85STduVGRQcTBXbWp5V3JiRmZMMnVISEJVCmdFRUE2N1E1WHg3Rml0emZiSTVGbVM1M0tidU91V2x4emdoMUJ1S0pyaHBWUytmaXZRTzBiSDhrTXpYQ0tLR0sKdVUvcGlWeTdQb2JPYW5yeXhGK0w2MCswTDFvTXdRYmlwNEVpaDNqS2I3N1duZUU1c05XbThoOW8xdE5VSnZzTgo3cms9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://192.168.68.11:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubelet-bootstrap
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: kubelet-bootstrap
  user:
    token: a6197a27fd379d91ab55107bc1a26af7
```

- 将生成后的bootstrap.kubeconfig文件拷贝到所有的worker主机上：

```shell
#拷贝bootstrap.kubeconfig到node1主机(192.168.68.12)
scp bootstrap.kubeconfig root@192.168.68.12:/opt/kubernetes/cfg

#拷贝bootstrap.kubeconfig到node2主机(192.168.68.13)
scp bootstrap.kubeconfig root@192.168.68.13:/opt/kubernetes/cfg
```

5. systemd管理kubelet

```shell
#在所有的worker主机上执行以下命令
cat > /usr/lib/systemd/system/kubelet.service << EOF
[Unit]
Description=Kubernetes Kubelet
After=docker.service
[Service]
EnvironmentFile=/opt/kubernetes/cfg/kubelet.conf
ExecStart=/opt/kubernetes/bin/kubelet \$KUBELET_OPTS
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF
```

6. 启动并设置开机启动

```shell
#在所有的worker主机上依次执行以下命令
systemctl daemon-reload
systemctl start kubelet
systemctl enable kubelet
```

###### 2.4.6.3  批准kubelet证书申请并加入集群

我们可以在master主机上使用`kubectl get csr`命令查看kubelet证书请求：

![image-20210305153702353](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210305153707.png) 

然后使用`kubectl certificate approve`命令批准申请：

```shell
#分别批准两个worker node的证书，命令后跟的是证书的名称
kubectl certificate approve node-csr-djCWIw5727-htth7GCF1WKviMQeFfJjazZC0WL8Vd68
kubectl certificate approve node-csr-z7OaLkThPMNaAkzvw-VLYc-5bLRCWk3gfBUsCM_2fK4
```

证书批准后，我们在master主机上使用`kubectl get node`命令就可以查看到集群中各节点的信息了。

![image-20210305154559485](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20210305154602.png)  

###### 2.4.6.4 部署kube-proxy

1. 创建配置文件

```shell
#在所有worker节点上执行以下命令
cat > /opt/kubernetes/cfg/kube-proxy.conf << EOF
KUBE_PROXY_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/opt/kubernetes/logs \\
--config=/opt/kubernetes/cfg/kube-proxy-config.yml"
EOF
```

2. 配置参数文件

```shell
#在node1节点(192.168.68.12)上执行以下命令
cat > /opt/kubernetes/cfg/kube-proxy-config.yml << EOF
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
metricsBindAddress: 0.0.0.0:10249
clientConnection:
  kubeconfig: /opt/kubernetes/cfg/kube-proxy.kubeconfig
hostnameOverride: node1
clusterCIDR: 10.0.0.0/24
EOF

#在node2节点(192.168.68.13)上执行以下命令
cat > /opt/kubernetes/cfg/kube-proxy-config.yml << EOF
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
metricsBindAddress: 0.0.0.0:10249
clientConnection:
  kubeconfig: /opt/kubernetes/cfg/kube-proxy.kubeconfig
hostnameOverride: node2
clusterCIDR: 10.0.0.0/24
EOF
```

3. 生成kube-proxy.kubeconfig文件

- 在master主机上执行以下命令，生成kube-proxy证书：

```shell
#切换目录
cd /root/TLS/k8s/apiserver

#在/root/TLS/k8s/apiserver目录下创建证书请求文件
cat > kube-proxy-csr.json<< EOF
{
    "CN": "system:kube-proxy",
    "hosts": [],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "BeiJing",
            "ST": "BeiJing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF

#然后在该目录下执行生成证书的命令
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy
```

- 在master主机上执行以下命令，生成kube-proxy.kubeconfig文件：

```shell
#设置临时环境变量
export KUBE_APISERVER="https://192.168.31.71:6443"

#设置集群参数
kubectl config set-cluster kubernetes \
--certificate-authority=/opt/kubernetes/ssl/ca.pem \
--embed-certs=true \
--server=${KUBE_APISERVER} \
--kubeconfig=kube-proxy.kubeconfig

#设置客户端认证参数
kubectl config set-credentials kube-proxy \
--client-certificate=/root/TLS/k8s/apiserver/kube-proxy.pem \
--client-key=/root/TLS/k8s/apiserver/kube-proxy-key.pem \
--embed-certs=true \
--kubeconfig=kube-proxy.kubeconfig

#设置上下文参数
kubectl config set-context default \
--cluster=kubernetes \
--user=kube-proxy \
--kubeconfig=kube-proxy.kubeconfig

#设置默认上下文
kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
```

- 将生成的kube-proxy.kubeconfig文件拷贝到所有的worker node主机上：

```shell
#拷贝到node1主机上(192.168.68.12)
scp kube-proxy.kubeconfig root@192.168.68.12:/opt/kubernetes/cfg

#拷贝到node2主机上(192.168.68.13)
scp kube-proxy.kubeconfig root@192.168.68.13:/opt/kubernetes/cfg
```

4. systemd管理kube-proxy

```shell
#在所有的worker节点上执行以下命令
cat > /usr/lib/systemd/system/kube-proxy.service << EOF
[Unit]
Description=Kubernetes Proxy
After=network.target
[Service]
EnvironmentFile=/opt/kubernetes/cfg/kube-proxy.conf
ExecStart=/opt/kubernetes/bin/kube-proxy \$KUBE_PROXY_OPTS
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF
```

5. 启动并设置开机启动

```shell
#在所有的worker节点上依次执行以下命令
systemctl daemon-reload
systemctl start kube-proxy
systemctl enable kube-proxy
```

###### 2.4.6.5 部署CNI网络

**注意**：部署CNI网络涉及的命令都是在master主机上执行的。

- 解压二进制包并移动到指定工作目录：

```shell
#创建指定工作目录
mkdir -p /opt/cni/bin && cd /opt/cni

#下载CNI二进制文件
wget https://github.com/containernetworking/plugins/releases/download/v0.8.6/cni-plugins-linux-amd64-v0.8.6.tgz

#解压CNI二进制文件到指定目录
tar -zxvf cni-plugins-linux-amd64-v0.8.6.tgz -C /opt/cni/bin
```

- 开始部署CNI网络：

```shell
#下载压缩包
wget https://files.cnblogs.com/files/gongcqq/kube-flannel.tar.gz

#解压压缩包，以便获取kube-flannel.yml文件
tar -zxvf kube-flannel.tar.gz

#执行kube-flannel.yml文件
kubectl apply -f kube-flannel.yml
```

**==注意==：由于网络部署失败，导致二进制安装失败。**












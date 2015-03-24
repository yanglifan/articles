# 一、前言
Docker 是最近在云计算领域出现的新技术。目前，Docker 和以其为代表的容器技术的热度已经改过了之前的 OpenStack。Docker 以及其所代表的容器技术的流行，即使因为软件技术的进步，更是由于其符合云计算对软件领域所带来新思想。在如今的互联网和企业应用开发领域，微服务和 DevOps 是两个思想颇为深入人心。而 Docker 技术的出现和其对整个容器技术及其生态圈发展的促进，解决了这个微服务和 DevOps 这两个思想实践中的很多难题，使得前面两种思想大规模地实现成为了可能。所以，我们有必要地深入了解一下 Docker 这个技术，看看它会对云计算时代的软件开发产生什么样的影响。

**本文将介绍如下内容：**

1. 什么是容器技术
1. 什么是 Docker
1. Docker 是如何实现的
1. 为什么要使用 Docker

至于 Docker 的用法，不在这里做介绍。Docker 的入门使用，可以前往 Docker 官网。各种高级用法、技巧、经验，可以前往技术网站 CSDN、InfoQ 和 CoreOS、Centurylink Labs 等这些使用 Docker 的云计算厂商的网站。

# 二、容器技术简介
容器技术有时会被称为轻量化虚拟技术。但不同于基于 Hypervisor 的传统虚拟化技术，容器技术并不会虚拟硬件。容器本身和容器内的进程都是运行在宿主 Linux 系统的内核之上。但与直接运行的进程不同，运行在容器内的进程会被隔离和约束。从而以直接运行的高效实现了虚拟技术的大部分效果。
 
![在此输入图片描述][1]

## 容器技术的历史 
容器技术并不是一个新鲜事物，早在1979年出现的在 Unix 系统中的 chroot 便是容器技术的雏形。而随后出现的 BSD Jail、Solaris Containers 和 OpenVZ 都算是容器技术的先驱。但容器技术开始普及却是在2007年，这一年 Google 贡献出 cgroups，并且从 2.6.4 开始，Linux 内核包含了这一组件，随后容器技术开始逐渐普及。但容器技术真正大放异彩则要等到2013年 Docker 0.10 版本发布。
 
# 三、Docker 简介
在 Docker 出现之前，不仅 Google 大量使用容器技术，国内的如淘宝也使用容器技术搭建了自己的应用平台。影响力最大的开源 PaaS 解决方案 CloudFoundry，也在使用自己的容器解决方案 Warden。而 Docker 发布之后，因其极有可能成为未来企业应用、互联网应用和云计算应用的开发、部署的中心角色，所以得到了几乎所有的业界大佬的追捧，Google、VMware、微软、RedHat 等等都已全力推动 Docker 技术的发展。同时，围绕 Docker，出现了一系列以 CoreOS 为代表的新技术。

Docker 的出现并非创造了一个新的容器技术，而是在 LXC (LinuX Container)<sup>注1</sup>、cgroups、namespaces 技术之上所构建的一种技术：
 
* Docker 简化了容器的运行：它通过一个简单的命令就能够运行起一个容器 `docker run [params] [image] [command (optional)]`
* Docker 简化了容器镜像的构建和分发：Docker 提供了 `Dockerfile` 和 `docker commit` 两种方式构建镜像，并且提供了 Docker image registry 机制以保存和分发镜像
 
## 形象地解释
打一个比方，集装箱（容器）对于远洋运输（应用运行）来说十分重要。集装箱（容器）能保护货物（应用），让其不会相互碰撞（应用冲突）而损坏，也能保障当一些危险货物发生规模不大的爆炸（应用崩溃）时不会波及其它货物（应用）但是把货物（应用）装载在集装箱（容器）中并不是一件简单的事情。而出色的码头工人（Docker）的出现解决了这一问题。它（Docker）使得货物装载到集装箱（容器）这一过程变得轻而易举。对于远洋运输（应用运行）而言，用多艘小货轮（虚拟机）代替原来的大货轮（实体机）也能保证货物（应用）彼此之间的安全，但是和集装箱（容器）比，成本过高，但适合运输某些重要货物（应用）。
 
![在此输入图片描述][2]

# 四、Docker 的组成
Docker 主要有 Docker Hub 和 Docker 引擎组成。前者是Docker 官方提供的容器镜像仓库；后者运行在宿主机上，可分为服务器端和客户端两部分。服务器端负责构建、运行和分发 Docker 容器等重要工作，客户端负责接收用户的命令和服务程序进行通信。

![在此输入图片描述][3]

除了这两部分，Dockerfile 也是不得不提的，它虽然不能算作一个独立的组件，但是却是 Docker 中很重要的部分。通过 Dockerfile，技术人员可以创建自己的 Docker 容器镜像。Dockerfile 起到了连接开发与运维的桥梁的作用，非常符合现在 DevOps 的潮流。

## Docker Hub
Docker Hub 是 Docker 官方所提供的一个镜像仓库。在运行 Docker 容器或构建自己的容器镜像时，都会直接或间接地使用到 Docker Hub 中的镜像。

## Docker Engine
Docker Engine 承载了 Docker 容器在宿主机上运行启停、Docker 镜像的构建等功能等功能。是我们接触最多的组件。接下来简单介绍一下 Docker 常见的命令：

* `run` 运行一个容器，如果镜像不存在则先下载。常用参数有 `-d`、`-t`、`-i` 等
* `pull` 下载容器镜像
* `start/stop` 启动/停止一个 Docker 容器
* `rm` 删除一个容器
* `rmi` 删除一个容器镜像
* `commit` 将容器中的修改提交至镜像中
* `logs` 显示容器运行的控制台输出
* `build` 从 Dockerfile 构建一个镜像
* `inspect` 显示容器运行参数，通过输入一个 JSON 格式的值来显示相应的结果
* `images` 显示当前宿主机上的所有镜像

## Dockerfile
通过编写 Dockerfile，我们可以构建自己的镜像。看一个 Dockerfile 的例子：
```
FROM dockerfile/java:oracle-java8
MAINTAINER Lifan Yang <yanglifan@gmail.com>
ADD device.jar /device.jar
EXPOSE 8080
ENTRYPOINT java -jar /device.jar
```
`FROM` 指令的意思是说你的镜像是基于一个什么镜像。`dockerfile/java:oracle-java8` 是一个镜像的名字，它也是有一个基础镜像狗狗见。其实它也是基于例如 Ubuntu、CentOS 这样的 Base 镜像。关于 Base 镜像的制作方法，Docker 官网上有介绍，需要专门的工具，这里不再作介绍。`MAINTAINER` 指令是可选的。`ADD` 指令是用来将一个文件或目录添加到 Docker 镜像中，前面是源文件，后面是目标文件。源文件必须使用相对路径。`EXPOSE` 指令用来容器间暴露端口，其指定的端口也会被 `-P` 参数映射给宿主机的一个随机端口上。`ENTRYPOINT` 可以用来指定运行 Docker 容器时，在容器中执行的命令是什么。如果需要运行多个命令，可以通过 [Supervisor](https://docs.docker.com/articles/using_supervisord/) 来执行。

除了这些指令，还有一些常用的指令。例如，用于在构建过程中执行命令的 `CMD` 指令；用于在容器中设置环境变量的 `ENV` 指令。详细请见 [Dockerfile Reference](https://docs.docker.com/reference/builder/)
 
# 五、Docker 的实现

接下来要介绍 Docker 所使用的几个重要技术：namespaces、cgroups、LXC 和 AUFS。

## namespaces
Linux 容器通过 Kernel 的 namespaces 技术，为一个或一组进程创建独立的 `pid`、`net` 等 namespaces，从而与其它进程相互隔离。下面将介绍 namespaces 都会对哪些资源进行分组控制以实现相互隔离：
 
### `pid` namespace 
不同容器中进程是通过 pid namespace 隔离开的，且不同容器中可以有相同 pid。具有以下特征:

* 每个 namespace 中的 pid 是有自己的 pid=1 的初始进程
* 每个 namespace 中的进程只能影响自己的同一个 namespace 或子 namespace 中的进程
* 因为 `/proc` 包含正在运行的进程，因此在容器中的 `/proc` 目录只能看到自己 namespace 中的进程
* 因为 namespace 允许嵌套，父 namespace 可以影响子 namespace 的进程，所以子 namespace 的进程可以在父 namespace 中看到，但是具有不同的 pid
 
### `net` namespace
如果仅仅隔离了进程空间，还是会有问题。比如如果你在多个容器中运行 Apache 服务器，那只能有一个服务器使用 80 端口。对此你有两个选择，让每个 Apache 服务器使用不同的端口，或者隔离网络空间。

`net` namespace 使得每个容器都有自己的 `lo` loopback 接口。同时，还有一个通常被命名为 `eth0` 的网络接口，通过这个接口，容器可以和 host 或其它容器进行通信。`eth0` interface 会被分配一个 172.17.0.XXX 的 IP 地址，容器之间可以通过这个 IP 地址相互通信。

![在此输入图片描述][4]
**图3：**容器内部 `ifconfig` 命令查看的结果

同时，容器的这个 `eth0` 网卡在 Host 中的名字是一个类似 `vethdfb7` 的略显古怪的名字。这个网卡会和 `docker0` 网卡桥接在一起。

![在此输入图片描述][5]

### `ipc` namespace
`ipc` namepace 对于不熟悉 Unix 的人（包括之前的我）吸引力不是很大。毕竟，现在进程之间的通信多数是通过网络实现的。但实际上，Unix `ipc` 有着十分广泛的应用，比如管道就是 `ipc` 的一种。对 `ipc` 就不做过多介绍了，总之 Linux Kernel namespaces 可以让不同容器的 `ipc` 相互隔离。

### `mnt` namespace
如名所示，`mnt` namespace 是处理挂载点的。`mnt` namespace 可以使不同容器拥有不同的挂载的文件系统和 root 目录。在一个 `mnt` namespace 挂载的文件系统只能被同一个 namespace 里的进程所见。
 
### `uts` namespace 
`uts` namespace 用于控制 hostname 的隔离。

有了以上几种隔离，一个容器就可以对外展现出一个独立计算机的能力，并且不同容器内的资源在操作系统层面实现了隔离。 然而不同 namespace 之间资源还是相互竞争的，仍然需要类似 ulimit 来管理每个容器所能使用的资源 - Docker 采用的是 cgroup。
 
参考文献
[1] http://blog.dotcloud.com/under-the-hood-linux-kernels-on-dotcloud-part
 
## cgroups
namespaces 对进程分组以实现资源隔离，但这隔离还是不够的。一个进程可以通过占用过度的硬件资源的方式去影响另一个分组中的进程。所以，要想实现完善的资源隔离，不仅要对资源分组，还要能对这一组内的进程所使用的资源进行约束。cgroups 就是用来实现这个目的的。cgroups 的全称是 Control Groups，在 2003 年由 Google 的工程师实现，在 2007 年加入 Linux Kernel。cgroups 可限制进程对 CPU、内存、块存储和网络的使用。这里不对 cgroups 的使用方法作介绍，感兴趣的同学可以参考如下：

* [SysAdminCasts.com: Introduction to Linux Control Groups (Cgroups)](https://sysadmincasts.com/episodes/14-introduction-to-linux-control-groups-cgroups)
* [Docker.com: Runtime Metrics](https://docs.docker.com/articles/runmetrics/)

虽然 cgroups 提供了对 IO 资源使用的约束的功能，但 Docker 目前(1.3)尚未提供支持。

### 内存
cgroups 可以控制进程所能使用的内存和 swap 空间的大小。通过 `-m` 参数，Docker 可以限制容器所能使用的最大内存数：
```
docker run -m 128m -d container_img cmd_name
```

### CPU
Docker 可以通过 cgroups 限制容器所能使用的 CPU 资源。在执行 `docker run` 命令时可以通过参数 `--cpu-shares` (`-c`)、`--cpuset` 对 CPU 做出限制。

#### `--cpu-shares` (`-c`)
`-c` 设置的是一个**相对值**，这个值将影响到此容器内的进程所能使用的 CPU 时间片。新运行的 Docker 容器默认使用 1024，对一个单独的 Docker 的容器来说，这个值没有任何意义。当启动两个 Docker 容器的时候，这两个容器将平分 CPU 时间片。如果两个容器，一个不指定 `-c`，另一个设置为 `-c 512`，那前者将使用大致 2/3 的计算能力，另一个将使用大致 1/3 的计算能力。但是 `-c` 对容器所能使用 CPU 的运行频率等没有任何影响。

#### `--cpuset`
可让你指定 Docker 容器中的进程运行在第几块 CPU 上。后面跟一个数组或用逗号分隔的多个数字 `0,1,2`。比如下列命令将会使你的容器运行在第一个 CPU 核心上。
```
docker run -i -t --cpuset 0 ubuntu:14.04
```

*注：cgroups 使用的是一种伪文件系统形式的接口。这个伪文件系统实际存在于内存中，但映射在目录中。用户通过在这个目录中写入文件来对 cgroups 进行操作。*

## AUFS
*注：最新的 Docker 使用 BTRFS 替代 AUFS，但所要实现的功能相同*
AUFS 的全称是 Another Union File System。AUFS（包括其它 UFS）的一个重要能力是能使两个目录结构合二为一。这有什么用呢？Docker 的镜像都是有多个层组成的，最上层是一个可读写的，而下面的层则是只读的。通过 AUFS 的目录融合的能力，实现了既可随意读写，又保证了下层的内容安全的目的。见下图可以有一个形象的认识。下图中的 bootfs 层包含了 Linux Kernel。在其上是某个特定的 Linux 发行版本的不同于 Kernel 的文件层，在下图中是 Debian。再往上有包含 emacs 和 Apache 的两个层。这些层在容器运行时都是只读的。最上面就是容器运行时可读写的层了。上面的层可以只读访问下面的层里的文件。这样的层次结构可以通过 Dockerfile 来创建。

![在此输入图片描述][6]

那这样的一个结构有什么样的好处呢？主要有下面几点：

### 节省磁盘和内存的存储空间
节省磁盘存储空间是因为不同的容器镜像之间可以共享相同的层。例如，两个不同的 Java 应用的容器镜像，它们都是基于 Ubuntu 14.04 和 JDK7。那在同一台 Host 中，这两个镜像就会使用相同的 Ubuntu 14.04 和 JDK7 的层。

节省内存空间是因为 Linux 为了加快磁盘访问，会将一些磁盘上的文件加载到内存中。所以，节省磁盘空间的同时也就可以间接地解释内存的使用。

### 加快部署速度
同样，可共享的镜像层能加快部署速度。因为相同的层不用被重复下载部署。

### 允许对文件任意改动
上传可读写的层可以对下面的只读层中的文件做任意修改，但这其实是 copy-on-write，所以，这种修改对下面的层其实是安全的。

# 六、Docker 的生态圈
Docker 再好，单靠 Docker 自身是无法满足互联网和企业应用的各种复杂需求的。好在 Docker 的出现带动了一些列技术的发展，形成了一个庞大的[生态圈](http://www.mindmeister.com/389671722/docker-ecosystem)。这个生态圈中的产品可大致分为如下几类：

## 容器编排管理
以 Google Kubernets 和 Apache Mesos 为代表。主要解决基于容器组成分布式集群应用的管理工作，例如对容器的运行状态的监控、容器自动化的故障恢复、基于容器的应用的扩容和缩容、服务发现。

## 基于容器的操作系统
以 CoreOS 和 Redhat Atomic 为代表。它们抛弃了 Linux 上面传统的包管理机制，而使用 Docker 作为应用的运行平台。同时精简系统。CoreOS 还引入了 Ectd、Fleet 等组件以更好地支持分布式系统。

## 基于容器的平台
PaaS 平台不是什么新鲜的概念，但却一直处于发育不良的状态。Docker 的出现给 PaaS 的发展带来了新的机遇，Docker 使得 PaaS 应用的部署有了统一的格式。以 Flynn 和 Deis 为代表的新的 PaaS 技术平台都是以 Docker 为基础的。

## 网络
如今的一台服务器可以轻松应付几百上千的 Docker 容器同时运行在其中。可以想见，在一个服务器集群中的 Docker 容器会有多少。如果对这么多的 Docker 容器所使用的网络进行组织管理便成为新的挑战。在这个领域主要有 Pipework、Weave 和 Flannel 等技术

## 配置管理工具
像 Puppet、Ansible 这样的配置管理工具早在 Docker 出现之前就已被广泛使用，但 Docker 的出现给这些技术带来了新的变化。是否能更好地支持对 Docker 容器集群的配置管理决定了这些技术今后的发展。

# 七、总结
本文简单介绍了 Docker 出现的背景、意义，Docker 的组成和背后的技术以及其所带动的生态圈。但作为一个新出现并在快速发展的基础性的技术，一两篇文章显然只能让人有一个最基本的认识。同时，任何技术也都有其两面性，Docker 作为一个新技术在实践中也存在这个非常多的问题。即便在国外，Docker 的应用也是出于起步阶段。所以还有很长的路要走。但是从最近一年多的发展看，Docker 无疑是一个非常有生命力的技术，必定会在今后的一段时间内成为一个热门、主流的技术。

虽然 Docker 的出现更多地是改变了服务器应用的部署和运维。但作为开发者来说，部署和运维的模型会对开发也产生很重大的影响，而且 Docker 的出现能使开发人员更好地参与到运维中来，这将促使应用更快、更好地迭代和发布。

更重要地是，云计算毫无疑问是未来互联网和企业应用的发展方向，而 Docker 和其所代表的容器技术将在未来一段时间内成为云计算的基础性技术。从这个角度来讲，Docker 是我们每一个从业人员必须了解甚至熟练掌握的一门技术。


  [1]: http://static.oschina.net/uploads/space/2014/1231/185336_4TLb_1158769.jpg
  [2]: http://static.oschina.net/uploads/space/2014/1231/185402_u1jv_1158769.jpg
  [3]: http://static.oschina.net/uploads/space/2014/1231/185423_hU94_1158769.png
  [4]: http://static.oschina.net/uploads/space/2014/1231/185518_7V88_1158769.png
  [5]: http://static.oschina.net/uploads/space/2014/1231/185602_HLA9_1158769.png
  [6]: http://static.oschina.net/uploads/space/2014/1231/185612_4cys_1158769.png
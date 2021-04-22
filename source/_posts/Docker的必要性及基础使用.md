---
title: Docker的必要性及基础使用
date: 2020-09-30 17:42:01
tags:
categories: 学习笔记
---

在阅读我们Docker的 [官方文档](https://www.docker.com/)我们知道：

* Docker  是世界领先的软件容器平台。
* 使用Docker 可以使开发人员消除一起协作开发遇到的一些问题；如"在我的电脑上可正常运行"。

 <!--more-->

* 使用Docker 可以使运维人员在隔离容器中并行运行和管理应用，获得更好的计算密度。
* 使用Docker 企业可以构建敏捷的软件交付管道，以更快的速度，更高的安全性和可靠的信誉为 Linux 和 Wiindows Server 应用发布新功能。

那么我们为什么需要 Docker 呢？

## 环境配置的难题

我们开发一般在写程序，需要好多个环境，主要的可以分为：

* **开发环境：**我们自己本地写代码的环境；
* **测试环境：** 提供给测试伙伴测试的环境；
* **生产环境：**测试完成可以上线的环境；

  我们在开发和学习编辑过程中， 好多时间都浪费在环境上：

* 如果重装了系统。我之前的项目，就得弄好多的配置，才能跑起来。比如配置MySQL，配置各种的环境变量等；
* 假如我们在网上学习跟这老师的步骤去写Demo,不知道你们遇到没，反正有些Demo我总有Bug。
* 我们在本地开发环境一切安好， 但是在测试环境上就出错了，或者测试环境是好的，一上线也是各种报错(心累~~);

所以就有“千万不要和程序员说，你的代码有bug”这样的笑话：

* 程序员的第一反应是你的环境有问题，第二就是你不会使用吧。
* 你要是跟他说“这个程序运行的怎么跟预期的不一样啊，是不是我操作的问题啊？”。
* 我们就会第一反应:“这是不是出bug了”

## 虚拟机

虚拟机就是带环境安装的一种解决方案。它可以在一种操作系统里面运行另外一种操作系统，比如我在 Windows 系统里面运行 Linnux 系统。应用程序对此是毫无感知的，，因为虚拟机看上去跟真实系统一摸一样，对于底成系统来说，虚拟机就是一个普通的文件，不需要了就删掉，对其他部分毫无影响。

虽然我们可以通过虚拟机还原软件的的原始环境，但是这个方案有几点缺点：

* **资源占用多**

  虚拟机会独自占用一部分内存和硬盘空间，它运行的时候其他程序就无法使用这些资源了，哪怕虚拟机里面的应用程序真正使用的内存只有1MB，虚拟机依然会需要几百MB的内存才能运行。

* **繁琐的步骤多**

  虚拟机是完整的操作系统，一些系统级别的步骤是无法跳过的，如用户登陆。

* **启动慢**

  启动操作系统需要多久的时间，启动虚拟机就需要多久的时间，可能需要等特别长的时间，应用程序才会运行。

## Linux容器

由于虚拟机存在这些缺点，Linux发明出了另外一种虚拟化技术；linux容器(Linux Container)LXC.。

Linux容器不是模拟一个完整的的操作系统，而是对进程进行隔离。或者说，在正常的进程外面套了一个保护层，对于容器里面的进程来说，它接触到的各种资源都是虚拟的，从而实现与底层系统的隔离。

由于容器是进程级别的，所以相比虚拟机有很多的优势：

* **启动快**

  容器里面的应用直接就是底层系统的一个进程，，而不是虚拟机内部的进程，所以启动容器就相当于启动本机的一个进程，而不是启动一个操作系统，速度就快很多了。

* **资源占用小**

  容器只占用需要的资源，不占用那些不需要额资源；虚拟机由于是完整的操作系统，不可避免的要占用所有的资源，另外，多个容器可以共享资源，虚拟机都是独享资源。

* **体积小**

  容器只需要打包用到的组件即可，而虚拟机是整个系统的打包，所以容器文件比虚拟机文件要小很多。

最后说明，容器有点像轻量级的虚拟机，能够提供虚拟化的环境，但是成本开销就小很多。

## Docker是什么？

Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。

它是目前最流行额 Linux 容器解决方案。

Docker 将应用程序与该程序的依赖打包在一个文件里面，运行这个文件，就会生成一个虚容器。程序在这个虚拟容器运行， 就好像在真实的物理机上运行一样。有了 Docker 就不用担心环境问题。

总体来说，Docker 的接口相当的简单， 用户可以方便的创建和使用容器，把自己的应用放入容器，容器还可以进行版本管理，复制，分享，修改，就像管理普通的代码一样。

## Docker 的用途

Docker 的主要用途，目前主要有三大类：

* **提供一次性的环境**

  比如：本地测试他人的软件，持续集成的时候提供单元测试和构建环境。

* **提供弹性的云服务**

  因为 Docker 容器可以随关随开，，很适合动态扩容和缩容。

* **组建微服务架构**

  通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构

## Docker 的基本概念

Docker 包含三个基本概念

* **镜像（image）**

  操作系统分为内核和用户空间，就Linux而言，内核启动后，，会挂载`root`文件系统为其提供用户空间支持。而 Docker镜像，就相当于一个 `root`文件系统。 Docker镜像就是一个特殊的文件系统，除了提供容器运行时所需的程序，库，资源，配置等文件外，还包含了一些为运行时准备的一些配置参数（如：匿名卷，，环境变量，用户等）。镜像不包含任何的动态数据，，其内容在构建完成之后也不会改变。

* **容器（container）**

  镜像（image）和 容器（container）的关系，就像面向对象设计中的 `类` 和 `实例` 一样。镜像是静态的定义，，容器是镜像运行时的实体。容器可以被创建，启动通知，删除，暂停等。

  容器的实质就是进程，但是与直接宿主执行的进程不同，容器进程运行于属于自己的独立的 `命令空间`。因此容器有自己`root`文件系统，自己的网络配置，自己的进程空间，甚至自己的用户ID空间。

* **仓库（repository）**

  镜像构建完成之后，可以很容易的在当前宿主机上运行，但是，如果需要在服务器上使用这个镜像，我们就需要一个集中的存储，分发镜像的服务。Docker 官方提供的 Docker [Hub](https://hub.docker.com/)  就是这样的服务，不过这是公共仓库。

## Dcoker 的安装

Docker 有三个更新的频道：`stable` `,`test` 和 `nightly

官方网站上有各种环境下的 [安装指南](https://docs.docker.com/get-docker/)

安装完成之后，可以使用一下命令检测 Docker 是否安装成功

```bash
docker version
```

或者：

````
docker info
````

  如果安装了就会得到关于 docker 的信息

{% asset_img 1.png %}

## hello world 实例

现在我们通过简单的 image 文件 [hello world](https://hub.docker.com/_/hello-world) 来感受一下 Docker

运行下面的命令，将image 文件从 仓库拉取到本地

```
docker image pull library/hello-world
```

这个代码中的 `docker image pull`  是抓取 image 文件的命令。`library/hello-world`是 image 文件在仓库里面的位置。·`library`是 image 文件所在的组`hello-world`是image文件的名称。

由于是 Docker 官方提供的image 文件，都放在 `library`组里面，属于默认组， 所以我们在抓取 Docker 官方 image 文件的时候，可以省略 `library`·。

所以上面的命令可以写成：

```
docker image pull hello-world
```

抓取过程：

{% asset_img 3.png %}

抓取成功之后，我们查看本机是否有该 image 文件：

```
docker image ls
```

{% asset_img 4.png %}

我们可以看出本机是有该 image 文件了的、

接下来，我们运行这个 image 文件：

```
docker container run hello-world
```

`docker container run`命令会从 image 文件 生成一个正在运行的容器。

注意：`docker container run`命令具有 自动抓取 image 文件的功能，如果发现本地没有指定的 image 文件，就会从仓库自动抓取。

如果运行成功，就会在屏幕上得到以下的输出：

{% asset_img 5.png %}

输出这些提示之后就会自动停止运行，容器就会自动终止。

下面会通过Docker 的三个基本概念来介绍。

## 镜像（image文件） 

Docker 把应用程序及其依赖，打包在 image 文件里面。image 是二进制文件。

只有通过这个文件，才能生成 Docker 容器（container），image文件可以看成是容器的模版。Docker 根据 image 文件生成容器的实例。同一个image文件可以生成多个同时运行的容器实例。

### 获取镜像

Docker 运行容器前需要在本地存在对应的镜像（image），如果不存在该镜像，Docker就会从镜像仓库下载该镜像。在[Docker Hub](https://hub.docker.com/search?q=&type=image) 上有大量的镜像可以使用。 

我们可以使用 `Docker pull`命令获取镜像，命令格式为：

```
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```

* 其中具体的`选项`可以通过`Docker pull --help`查看。如：

  ```
  $  Docker pull --help
  Options:
    -a, --all-tags                Download all tagged images in the repository
        --disable-content-trust   Skip image verification (default true)
        --platform string         Set platform if server is multi-platform capable
    -q, --quiet                   Suppress verbose output
  ```

  

* Docker 镜像仓库地址：地址的格式一般是 `<域名/IP>[:端口]`。默认的地址是 [Docker Hub](https://hub.docker.com/search?q=&type=image) 
* 仓库名：仓库名是两段式名称，即：`<用户名>/<软件名>`。对弈仓库是 [Docker Hub](https://hub.docker.com/search?q=&type=image) 如果不给出用户名，那么默认为`library`(官方镜像)。

举个🌰：

我们从 [Docker Hub](https://hub.docker.com/search?q=&type=image) 仓库上拉取一个 `hello-world`镜像（image）

```bash
$ Docker pull hello-world
Using default tag: latest
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:8c5aeeb6a5f3ba4883347d3747a7249f491766ca1caa47e5da5dfcf6b9b717c0
Status: Downloaded newer image for hello-world:latest
docker.io/library/hello-world:latest
```

这命令明没有给出 Docker 镜像（image）仓库地址，那么将会从[Docker Hub](https://hub.docker.com/search?q=&type=image) 获取镜像（image）。镜像的名字叫做`hello-world`，因此会获取官方镜像`library/hello-world`仓库中的最新的版本的镜像(我们没有执行用哪个标签的版本，所以是最新的)。

### 列出镜像

可以使用命令`docker image ls 列出已经下载下来的镜像。

```
$ docker image ls
REPOSITORY                                 TAG                       IMAGE ID            CREATED             SIZE
express-for-docker                         0.0.1                     75b97434d52d        2 days ago          920MB
<none>                                     <none>                    b2d9a05ebb31        2 days ago          920MB
<none>                                     <none>                    1083c90adc0f        2 days ago          920MB
<none>                                     <none>                    914f52bcbe8b        2 days ago          920MB
<none>                                     <none>                    7032438fb234        2 days ago          920MB
shuliqi/express-for-docker                 0.0.1                     e46126aa45a5        6 days ago          920MB
r.addops.soft.360.cn/qixiao/backend-test   20200529163234-5ed720bd   a5e1f04e8d6a        4 months ago        2.71GB
node                                       12.17.0                   a37df1a0b8f0        4 months ago        918MB
hello-world                                latest                    bf756fb1ae65        9 months ago        13.3kB
```

我们可以看出来，列表包含了 `仓库名`，`标签`，`镜像ID`，`创建时间`，`所占用的空间`。其中镜像ID 是镜像的唯一标识、

 image 文件是通用的，一台机器的 image 文件拷贝到另外一台机器，依然是可以使用的。所以一般来说，为了节省时间，我们应该尽量使用别人制作好的 image 文件，而不是自己制作，即使自己制作，也应该基于别人 image 文件进行加工，而不是从零制作。

为了方便共享，inage 文件制作完成之后可以上传到网上的仓库。Docker 的官方仓库 [Docker hub](https://hub.docker.com/) 应该是最常用的。

#### 虚悬镜像

在上面的镜像中， 有一些比较特殊镜像，这些镜像没有仓库名。也没有标签，均为：`<none>`

```bash
<none>                                     <none>                    b2d9a05ebb31        2 days ago          920MB
<none>                                     <none>                    1083c90adc0f        2 days ago          920MB
<none>                                     <none>                    914f52bcbe8b        2 days ago          920MB
<none>                                     <none>                    7032438fb234        2 days ago          920MB
```

原因是这样的，这些镜像原先是有镜像和标签的，但是随着这些镜像的官网维护，发布了新版本之后， 重新 `Docker	 pull` 的时候。旧的镜像名被转移到新下载的镜像身上了。而这个旧的镜像上的名称则被取消，从而成为了 `<none>`。除了docker pull`可能导致这种情况。`docker build`也同样可以导致这种现象。

由于新旧镜像同名，旧镜像名称被取消。从而出现仓库名，标签均为`<none>`的镜像。这类镜像就被称为**虚悬镜像（dangling image）**

一般使用`docker image ls-f dangling=true`来显示这类镜像：

```bash
$ docker image ls -f dangling=true
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              b2d9a05ebb31        2 days ago          920MB
<none>              <none>              1083c90adc0f        2 days ago          920MB
<none>              <none>              914f52bcbe8b        2 days ago          920MB
<none>              <none>              7032438fb234        3 days ago          920MB
```

一般来说，虚悬惊险已经失去了存在的价值，是可以随意删除，可以使用下面的命令删除

```bash
docker image prune
WARNING! This will remove all dangling images.
Are you sure you want to continue? [y/N] y
Deleted Images:
deleted: sha256:914f52bcbe8b3931a0f28de7f0fdac94718eb679af0eee6056a4339896a2986b
deleted: sha256:8b34628d630ae6f1313ed41ce1539acaa66a2adcc20c5f888475008da5f8f8af
deleted: sha256:107e046f0614e4beb0296e38933989724b232bef66013a83beb5e4a507b15875
deleted: sha256:7adb718c36f311154759d7659bb3a56edc0eef972fe808960a1890a260a344d5
deleted: sha256:49e7e6fadd8e6517ee862d5281a849faa45adf6e097ce280927e5861936731ba
deleted: sha256:b8cfe3d0a0f3c1b19554f9e4cb39978810793399d4c20eb819b7b67205ad58f0
Total reclaimed space: 5.13MB
```

#### 中间层镜像

为了加快镜像的构建，重复利用资源，Docker 会利用**中间镜像**。所以在使用一段时间后，可能会看到一些依赖的中间层镜像。默认的命令`docker image ls`只会列出顶层镜像。如果希望现实包含中间层镜像在内的所有镜像的话，需要加`-a`参数

```
$ docker image ls -a
```

这样就会看到很多无标签的镜像，与之前的虚悬镜像 不同，这些无标签的镜像很多都是中间层镜像，是其他镜像所依赖的镜像，这些无标签的镜像就不应该删除。否则会导致上层镜像会因为失去依赖而出错。

### 删除镜像

如果想要删除本地的镜像，可以使用`Docker image rm`命令，其格式为：

```bash
$ docker image rm [选项] <镜像1> [<镜像2>...]
```

其中`镜像` 可以是 `镜像短ID`，`镜像长ID`,`镜像名` 或者 `镜像摘要`.

举个🌰：

我们先列出本地的镜像（image）

```bash
$ docker image ls
REPOSITORY                                 TAG                       IMAGE ID            CREATED             SIZE
express-for-docker                         0.0.1                     75b97434d52d        3 days ago          920MB
<none>                                     <none>                    b2d9a05ebb31        3 days ago          920MB
r.addops.soft.360.cn/qixiao/backend-test   20200529163234-5ed720bd   a5e1f04e8d6a        4 months ago        2.71GB
```

我们可以使用`镜像长ID`来删除镜像。使用脚本的时候可能会用长ID。但是人工输入的话就会太累了。所以更多的时候是使用`短ID`来删除镜像、·docker image ls·列出的就是镜像的短ID。

假如我们要删`express-for-docker `·镜像，就可以使用短ID执行：

```bash
$ docker image rm 75b97434d52d
Untagged: express-for-docker:0.0.1
Deleted: sha256:75b97434d52d019ad7ee9c1e1d2c903bc6d5e7261a429cb0f97c0302a0323279
Deleted: sha256:4605991534be3e41e1dd0dc142be83617424b94f68feab431ee4cdf88be944f2
Deleted: sha256:b77592ec1b655839244fdaa2bf94f7ebe606e20ae617154aedaed5e6bd04b9dd
Deleted: sha256:15c1b2c7cd44394daa5b22ddae5e66a5a8804bef4a70ef172738f3b8cbf4a8dd
Deleted: sha256:ffa48c079ea207b279b6fffaa41b893e5c66c9c5908fe616e07978e32ed4b341
Deleted: sha256:ffcf86b030c4b1ecba89778455e52e9ce61ecf22fb2ee7fe8e642d52b0f7f82d
Deleted: sha256:e24e7952278d2bde9a56a78ef1887ca732e786a3735f8e3c1cbc315b88edb0b6
Deleted: sha256:60d46f3b1697a276dff10b74fb8199f523c171ece03bd1e372a0b6ce3c6ada23
Deleted: sha256:4e891fdf8b241174de56b22682015358baf52302f4a7a2ddbd0ff417a71d0e2d
```

也可以使用镜像名字，即：`<仓库名>：<标签>`，来删除镜像.

````bash
$ docker image ls
REPOSITORY                                 TAG                       IMAGE ID            CREATED             SIZE
<none>                                     <none>                    b2d9a05ebb31        3 days ago          920MB
r.addops.soft.360.cn/qixiao/backend-test   20200529163234-5ed720bd   a5e1f04e8d6a        4 months ago        2.71GB
hello-world                                latest                    bf756fb1ae65        9 months ago        13.3kB
$ docker image rm  hello-world:latest
Untagged: hello-world:latest
Untagged: hello-world@sha256:8c5aeeb6a5f3ba4883347d3747a7249f491766ca1caa47e5da5dfcf6b9b717c0
Deleted: sha256:bf756fb1ae65adf866bd8c456593cd24beb6a0a061dedf42b26a993176745f6b
Deleted: sha256:9c27e219663c25e0f28493790cc0b88bc973ba3b1686355f221c38a36978ac63
 shuliqi@shuliqideMacBook-Pro  ~ 
````

当然也可以使用 `镜像摘要`来删除镜像。

首先我们使用命令`docker image ls --digests`先列出镜像的摘要

```bash
$ docker image ls --digests
REPOSITORY                                 TAG                       DIGEST                                                                    IMAGE ID            CREATED             SIZE
<none>                                     <none>                    <none>                                                                    b2d9a05ebb31        3 days ago          920MB
r.addops.soft.360.cn/qixiao/backend-test   20200529163234-5ed720bd   sha256:d88b37ff35301dc149c346345cb0609600201b720b14d29f52f7ef232a098e39   a5e1f04e8d6a        4 months ago        2.71GB
hello-world                                latest                    sha256:8c5aeeb6a5f3ba4883347d3747a7249f491766ca1caa47e5da5dfcf6b9b717c0   bf756fb1ae65        9 months ago        13.3kB
```

然后使用`镜像摘要`删除:

```bash
$ docker image rm sha256:8c5aeeb6a5f3ba4883347d3747a7249f491766ca1caa47e5da5dfcf6b9b717c0
```

### 使用Dockerfile 定制镜像

这一部分也是很重要的一个知识点， 我们放在下面的内容 **实例：制作自己的 Docker 容器**来一起讲。

## 容器（container 文件）

image 文件生成的容器实例，本身也是一个文件，称为容器文件。

一般来说，一旦容器生成，就会同时存在两个文件： image 文件 和 容器文件，而且关闭容器不会删除容器文件。只是容器停止运行而已。

### 列出容器：

列出本机容器 可以使用命令 `docker container ls`。其命令格式为：

```
$ docker container ls [OPTIONS]
```

OPTIONS 的选项很多。具体可以看看 [docker container ls](https://docs.docker.com/engine/reference/commandline/container_ls/)的文档。

列出本机正在运行的容器：

```
$ docker container ls 
```

列出本机所有的容器，包括终止运行的容器：

````
$ docker container ls -a
CONTAINER ID        IMAGE                              COMMAND                  CREATED             STATUS                        PORTS               NAMES
d211b923866c        ubuntu                             "/bin/bash"              40 minutes ago      Exited (0) 40 minutes ago                         goofy_feistel
a80be7a20f56        ubuntu                             "/bin/bash"              40 minutes ago      Exited (0) 40 minutes ago                         inspiring_clarke
8d91eaa8f2e1        ubuntu                             "/bin/bash"              3 hours ago         Exited (0) 3 hours ago                            bold_mendel
995f498d0371        ubuntu                             "/bin/bash"              3 hours ago         Exited (0) 3 hours ago                            thirsty_mestorf
33b16714d2b8        shuliqi/express-for-docker:0.0.1   "docker-entrypoint.s…"   3 hours ago         Up 3 hours                    3000/tcp            great_lewin
251d68312383        ubuntu                             "/bin/bash"              3 hours ago         Exited (130) 3 hours ago                          vibrant_banach
1f50021cd2b8        ubuntu                             "/bin/bash"              3 hours ago         Exited (0) 3 hours ago                            gifted_keldysh
````

得到的结果之中有容器的ID。这个ID的使用场景就很多，比如终止容器的时候需要提供。

### 容器启动

启动容器有两个方式：

* 一种是基于镜像新建一个容器并启动；
* 一种是将终止状态的容器重新启动；

#### 新建并启动

即第一种启动容器的得方式。启动所需的命令：`docker run`。其格式为：

```
$ docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

OPTION 选项 具体可以看官方 [docker run](https://docs.docker.com/engine/reference/commandline/run/)

下面的内容会是我们比较常用的参数的例子。

我们继续以官方的事镜像`Hello world`来演示。如：

```
$ docker run hello-world

Hello from Docker!
...
```

我们拉取我自己上传的一个镜像（image）`express-for-docker`，再举个特别的🌰：

```
docker pull shuliqi/express-for-docker:0.0.1
```

如果提示没有权限获取，可能没有登录  [Docker hub](https://hub.docker.com/) 。需要注册登录一下，下面的内容有讲，可以移步往下面内容看看。

```bash
$ docker run -t -i shuliqi/express-for-docker:0.0.1 /bin/bash
root@f6cf67490ae4:/shuliqi#
```

上面这个命令启动了一个bash 终端，允许用户进行交互。

* **-t：**表示容器的 Shell 映射到当前的 Shell，然后你在本机窗口输入的命令，就会传入容器
* **-i:**   表示让容器的标准输入保持开启
* **/bin/bash:**  容器启动后，内部第一个执行的命令。这里是起订 Bash.保证用户使用 Shell

我们再试一个命令：

```bash
$ docker run  -t -i -p 8000:3000 shuliqi/express-for-docker:0.0.1 /bin/bash
root@e400118aabd7:/shuliqi# node app.js
111
Example app listening at http://localhost:3000
```

然后打开我们浏览器，访问http://localhost:8000/ 就会看到如下界面：

{% asset_img 11.png %}

其中:

* **-p:** 是指将容器的某个端口映射到本机的某个端口。这个例子就是讲 容器的 3000 端口 映射到本机的 8000 端口

#### 启动已终止的容器

可以使用`docker container start`命令来启动一个已终止的容器。其格式为：

```
$ docker container start [OPTIONS] CONTAINER [CONTAINER...]
```

OPTION 的选项具体可以看官方文档：[docker container start](https://docs.docker.com/engine/reference/commandline/container_start/)

例如：

```
docker container start e400118aabd7
```

更多的时候我们是需要Docker 在后台进行而不是直接把执行命令输出的结果输出到当前目录下。那么久可以是用`-d`来参数来实现。

```
$ docker run -d -t -i -p 8000:300
597e02ae84f497a8ff6ce03b87ecc40cd2c2a44b3eba41ed927e7aa7beded980
```

使用·-d· 参数启动后会返回一个唯一的id。

这时候继续访问 ：http://localhost:8000 还是会看到和之前一样的界面。

如果我们想看容器的输出结果。可以使用`docker container logs [containerID or NAMES]:`

````
$ docker container logs 597e02ae84f497a8ff6ce03b87ecc40cd2c2a44b3eba41ed927e7aa7beded980
111
Example app listening at http://localhost:3000
````

### 终止容器运行

终止容器可以使用：`docker container stop [containerID]`命令来终止。其格式为：

````
$ docker container stop [OPTIONS] CONTAINER [CONTAINER...]
````

例如：

```
$ docker container stop 597e02ae84f4
597e02ae84f4
```

上面的命令会导致 containerID 为 597e02ae84f4 的容器停止运行。

可以通过 `docker container ls -a`来查看终止状态的容器：

```
$ docker container ls -a
CONTAINER ID        IMAGE                              COMMAND                  CREATED             STATUS                        PORTS                                      NAMES
597e02ae84f4        shuliqi/express-for-docker:0.0.1   "docker-entrypoint.s…"   8 minutes ago       Exited (137) 7 seconds ago                                               optimistic_antonelli
```

### 进入容器

在前面我们说过 使用 `-d`参数时，容器启动后悔进入后台。

但是有些时候需要进入容器进行一些操作。那么可以使用`docker attach`或 `docker exec`来进入容器。但是推荐使用`docker exec `进入容器。因为使用`docker attach`进入容器。如果在stdin中exit(退出),是会导致容器停止的。而`docker exex`则不会。其格式分别为：

```
$ docker attach [OPTIONS] CONTAINER
```

```
$ docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

使用`docker exec`的话， 后面可以跟多个参数，可以使用`docker exec --help`来查看参数。这里主要说名`-t`，·-i·

参数。

如果只使用`-i`参数的话，由于没有分配伪终端。界面没有我们熟悉的Linux命令符提示器。但是命令执行结果依然可以返回。

当`-i`,`-t`参数一起使用时，则可以看到我们熟悉的Linux命令提示符。

```
$ docker exec -i 33b16714d2b8 bash
ls
Dockerfile
README.md
app.js
node_modules
package-lock.json
package.json
```

```
$ docker exec -i -t 33b16714d2b8 bash
root@33b16714d2b8:/shuliqi# ls
Dockerfile  README.md  app.js  node_modules  package-lock.json	package.json
```

### 导出和导入容器

使用`docker export`来导出本地的某个容器。其格式为：

```
docker export [OPTIONS] CONTAINER
```

OPTIONS 只有一个选项:

* **--output , -o:**将输入内容写到文件

例如：

```
$ docker container ls
CONTAINER ID        IMAGE                              COMMAND                  CREATED             STATUS              PORTS               NAMES
33b16714d2b8        shuliqi/express-for-docker:0.0.1   "docker-entrypoint.s…"   3 hours ago         Up 3 hours          3000/tcp            great_lewin
$ docker export 33b16714d2b8 > express-docker.tar
$ ls
express-docker.tar

```

```
$ docker export -o="shuliqi.tar" 33b16714d2b8
$ ls
express-docker.tar shuliqi.tar
```

可以使用`docker import`从容器快照文件中再导入为镜像。其格式为：

```
$ docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
```

OPTION选项可以查看官方文档  [docker import](https://docs.docker.com/engine/reference/commandline/import/)

例如：

```
$ cat express-docker.tar  | docker import - express-docker.tar

sha256:d16ef3cbe87943de5c4a7c91f11de88b6b1854f7b8898f64060c7580c2840dda
$ docker image ls
REPOSITORY                                 TAG                       IMAGE ID            REPOSITORY                                 TAG                       IMAGE ID            CREATED             SIZE
express-docker.tar                         latest                    d16ef3cbe879        45 seconds ago      913MB
```

### 删除容器

终止运行的容器文件，依然会占据硬盘空间，可以使用`docker container rm`命令来删除。其格式为：

```
$ docker container rm [OPTIONS] CONTAINER [CONTAINER...]
```

如果删除一个 运行中的容器，可以添加 `-f` 参数。

使用`docker container ls --a`命令，可以查看所有已创建的包括终止状态的容器。如果数量太多的话一个一个删除会很麻烦。**清理所有处于终止状态的容器**可以使用命令 `docker container prune`

## Dockerfile 文件

以上的内容我们学会了如何使用 image 和容器文件，但是如何生成 image 文件呢？

这就需要 DockerFile 文件，它是一个文本文件，用来配置 image，Docker 根据该文件生成二进制的 image 文件。

那么我们就通过一个实例，来演示如何编写 DcokerFile 文件吧。

## 实例：制作自己的 Docker 容器

我们先创建一个 express 搭建的项目传到自己的 github。我已经创建完了 [express-for-docker](https://github.com/shuliqi/express-for-docker)。怎么创建 express 实例，可以看[express的官网](https://expressjs.com/zh-cn/starter/hello-world.html)。

当然我们上面使用的好多例子也用到了我们上传的 image 文件 [express-for-docker](https://hub.docker.com/r/shuliqi/express-for-docker)

接下来我们开始制作自己的 Docker 容器

### 编写 Dockerfile 文件

首先， 我们在项目的根目录下新建一个文本文件 `.dockerignore`,并写下下面的内容：

```dockerfile
.git
node-modules
npm-debug.log
```

上面的代码表示，这三个路径需要排除，不要打包进 image 文件。如果没有路径需要排除，则这个文件不需要新建。

然后我们在项目的根目录下新建一个文本文件 `Dockerfile`,写入下面的内容：

```dockerfile
FROM node:8.4
COPY . /shuliqi
WORKDIR /shuliqi
RUN ["npm", "install"]
EXPOSE 3000/tcp
```

上面的五行代码分别代表什么意思呢？

* **FROM node:8.4**

  这个 image 文件继承官方的得 node image，冒号表示标签，这里的标签是12.17.0。即12.17.0 版本的node

* **COPY . /app**

  将当前目录下的所有文件（除了.dockerignore排除的路径）都拷贝到 image 文件的/app目录下面。

* **WORKDIR /app**

  指定接下来的工作路径为/app

* **RUN ["npm", "install"]**

  在/app目录下面，运行 npm install 安装依赖。注意：安装后的所有依赖，都将打包进入  image 文件

* **EXPOSE 3000**

  将容器的 3000 端口暴露出来，允许外部连接这个端口。

关于完整的一个 Dockerfile各种命令的使用，我们在下一节讲， 请继续关注哦(* ￣3)(ε￣ *)！！

### 创建 image 文件

有了 .Dockerfile 文件之后， 就可以使用 `docker  build`命令创建 image 文件。`docker build` 的具体格式为：

```
$ docker build [OPTIONS] PATH | URL | -
```

当然 OPTION 选项也是很多的。具体也可以翻阅官方文档 [docker build ](https://docs.docker.com/engine/reference/commandline/build/)。

这里我们讲一个常用到的OPTION 选项：

* **--tag, -t:**镜像的名字以及标签通常 name:tag 格式 或是是 name 格式。

name继续我们的是咧。我们来构建 image 文件：

```
docker  build -t express=for-docker .
//或者
docker  build -t express=for-docker:0.0.1 .
```

这个命令中的 `-t`参数用来指定 image 文件的名字。后面还可以使用冒号来指定标签。如果不指定的话，默认的标签就是 latest。最后那个表示表示 Dockerfile文件所在的路径。

```
$ docker build -t express-for-docker .
Sending build context to Docker daemon  2.011MB
Step 1/6 : FROM node:8.4
 ---> 386940f92d24
Step 2/6 : COPY . /shuliqi
 ---> Using cache
 ---> 04a303d12b5f
Step 3/6 : WORKDIR /shuliqi
 ---> Using cache
 ---> fe2455b89e40
Step 4/6 : RUN ["npm", "install"]
 ---> Using cache
 ---> 4b9c8652ccc9
Step 5/6 : EXPOSE 3000/tcp
 ---> Using cache
 ---> 11fbd94359f4
Step 6/6 : CMD node app.js
 ---> Using cache
 ---> 1c4d4712b8c0
Successfully built 1c4d4712b8c0
Successfully tagged express-for-docker:latest
$ docker image ls
REPOSITORY                                 TAG                       IMAGE ID            CREATED              SIZE
express-for-docker                         latest                    1c4d4712b8c0        About a minute ago   674MB
```

打包完成后，我们在使用 `docker image ls`查看所有的image文件，是可以看到我们刚刚打包的image文件（express-for-docker）

### 生成容器

使用`docker run`命令就会从image 文件生成容器：

```
$ docker run -t -p 8000:3000  express-for-docker:0.0.1 /bin/bash
```

上面命令的参数含义如下：

* **-p参数：** 容器的 3000 端口 映射到 本机的 8000 端口
* **-t 参数：** 容器额 Shell  映射到当前的 Shell，然后在本机窗口输入命令，就会传入容易
* **express-for-docker:0.0.1**： image文件的文件名和tag
* **/bin/bash：**容器启动后，内部第一个执行的命令。这里是启动 Bash，保证用户可以使用 Shell。

执行上面的命令，如果一切正常，那么就会返回命令行提示符号

```
root@622773f4bec9:/app#
```

这个提示就说明你已经在容器里面了，返回的提示符就是容器里面的 Shell 提示符。

接下来我们执行以下命令：

```
root@622773f4bec9:/app# node app.js
```

这命令就是启动我们这个项目的， 这时打开 http://127.0.0.1:8000 网页显示 “Hello World!”

### CMD 命令

上面的例子， 容器启动后，我们需要手动的 输入 ` node app.js`来启动项目。其实我们可以吧这个命令写在 Dockerfile 文件里面，这样容器启动后，这个命令就会执行了，不需要我们手动输入。

```
FROM node:12.17.0
COPY . /shuliqi
WORKDIR /shuliqi
RUN ["npm", "install"]
EXPOSE 3000/tcp
CMD node app.js
```

多出来的这与 一个命令`CMD node app.js` 它表示容器启动后自动执行  `node app.js`.。

**CMD 和 RUN 命令的区别：**

`RUN`命令在 image 文件构建阶段执行，执行结果都会打包进入 image 文件

`CMD`命令则是在容器启动后执行

一个Dockerfile文件可以包含多个 `RUN`命令，但是只能有一个`CMD` 命令

### 发布 image 文件

容器运行成功了之后，就说明 image 文件是有效的。那么我们可以考虑把 image 文件分享到网上， 让其他人也可以使用。

首先，我们需要去 [hub.docker.com](https://hub.docker.com/)注册账号，然后使用如下的命令登录：

```
$ docker login
```

然后输入注册的用户名和密码。如果成功登录的话，会显示

{% asset_img 8.png %}

那么接下来继续我们的发布，我们给本地的 image 标注用户名和版本：

```
$ docker tag [imageName] [userNane]/[repository]:[tag]
```

我们给自己的 `express-for-docker`标注用户名和版本：

```
$ docker tag express-for-docker:0.0.1 shuliqi/express-for-docker:0.0.1
```

标注完之后，我们就可以往 [hub.docker.com](https://hub.docker.com/)发布我们的image包了。使用如下的命令：

```
$ docker push express-for-docker:0.0.1
```

如发布成功，则会显示：

{% asset_img 9.png %}

我们到 [hub.docker.com](https://hub.docker.com/) 上就会看到已经上传的 image 文件

{% asset_img 10.png %}

这样我们就成功发布了一个 image 文件

文章使用的例子 [express-for-docker](https://github.com/shuliqi/express-for-docker)

下一节详细讲 [Dockerfile 文件](https://shuliqi.github.io/2020/10/20/Dockerfile%E6%96%87%E4%BB%B6%E7%9A%84%E4%BD%BF%E7%94%A8/)

## 参考文章

* [Docker —— 从入门到实践](https://yeasy.gitbook.io/docker_practice/image/list)
* [Docker 入门教程](https://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)
* [docker docs](https://docs.docker.com/reference/)


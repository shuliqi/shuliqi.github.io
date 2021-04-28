---
title: Dockerfile文件的使用
date: 2020-10-20 14:27:40
tags: 其他
categories: 其他
---

在上一篇文章 [Docker的必要性及基础使用](https://shuliqi.github.io/2020/09/30/Docker%E7%9A%84%E5%BF%85%E8%A6%81%E6%80%A7%E5%8F%8A%E5%9F%BA%E7%A1%80%E4%BD%BF%E7%94%A8/)，我们知道 Dockerfile 是一个文本文件。这个文件里面包含了一系列的指令，每一条指令构建一层，每一条指令的内容及就是描述该层是如何构建的。

 <!--more-->

# Dockerfile文件格式

我们可以上篇文章 [Docker的必要性及基础使用](https://shuliqi.github.io/2020/09/30/Docker%E7%9A%84%E5%BF%85%E8%A6%81%E6%80%A7%E5%8F%8A%E5%9F%BA%E7%A1%80%E4%BD%BF%E7%94%A8/)的 [Demo](https://github.com/shuliqi/express-for-docker) 的Dockerfile文件：

```dockerfile
FROM node:12.17.0
COPY . /shuliqi
WORKDIR /shuliqi
RUN ["npm", "install"]
EXPOSE 3000/tcp
CMD node app.js
```

因此我们知道`Dockerfile` 文件的格式如下:

```
# comment
INSTRUCTION arguments
```

```
# 注释
指令 参数
```

`Dockerfile`文件中的指令是不区分大小写的，但是为了更容易区分，约定使用 大写形式

Docker会依次执行`Dockerfile`文件中的指令，文件中第一条指令必须是`FROM`。

以 `#`开头的行，Docke会认为是注释，但是 `# `出现在指令参数中，则不是注释。如：

```
# Comment
RUN echo 'how old are you # my name is shuliqi'
```

# Dockerfile 的组成部分

Dockerfile 文件主要由 四部分组成，分别是：

| 部分               | 指令                                                     |
| ------------------ | -------------------------------------------------------- |
| 基础镜像信息       | FROM                                                     |
| 维护者信息         | MAINTAINER                                               |
| 镜像操作指令       | RUN，COPY，ADD，EXPOSE、WORKDIR、ONBUILD、USER、VOLUME等 |
| 容器启动时执行指令 | CMD、ENTRYPOINT                                          |

# Dockerfile中的指令

接下来我们讲指令的同时结合例子来讲，首先我们先创建一个可以使用的例子。关于怎么创建可以参考上一篇 博文[Docker的必要性及基础使用](https://shuliqi.github.io/2020/09/30/Docker%E7%9A%84%E5%BF%85%E8%A6%81%E6%80%A7%E5%8F%8A%E5%9F%BA%E7%A1%80%E4%BD%BF%E7%94%A8/#%E5%AE%9E%E4%BE%8B%EF%BC%9A%E5%88%B6%E4%BD%9C%E8%87%AA%E5%B7%B1%E7%9A%84-Docker-%E5%AE%B9)。我创建了项目 **[dockerfile-example](https://github.com/shuliqi/dockerfile-example)**。当然项目顺便你怎么弄，直接减一个Dockerfile的文件也算是个。

## FROM

`FROM`指令为后面的指令提供镜像基础。`FROM`指令必须是`Dockerfile文件的`首条命令，启动构建流程后，`Docker`将会基于该镜像构建新镜像，`FROM`后的命令也是基于这个基础镜像。

`FROM`的语法格式：

```
FROM <image>
```

或

```
FROM <image>:<tag>
```

或

```
FROM <image>:<diaest>
```

通过 `FROM`指定的镜像，可以是任何有效的基础镜像，`FROM`有以下的限制：

* `FROM`必须是 `Dockerfile`文件的第一条非注释命令
* 在一个 `Dockerfile`文件中创建多个镜像时，`FROM`是可以多次出现。只需要在每个新命令`FROM`之前，记录提交上次的镜像ID
* `tag` 和 `digest`是可选的，如果不使用这两个值的时候，会使用`latest`版本的基础镜像

举个🌰：

在我们当前的项目中， 我们使用node的基础镜像。就可以这么写

```
FROM node:8.4
```

[Docker Hub](https://hub.docker.com/search?q=&type=image&image_filter=official)有很多高质量的官方镜像。是可以直接拿来使用的。

除了选择现有的镜像为基础镜像外。Dcoker 还有一个比较特殊的镜像叫`scratch`。这个镜像是虚拟的概念。并不实际存在、表示一个空白的镜像。我们也可以使用：

````
FROM scratch
````

如果使用了 `scratch`为基础镜像的话，就说明不以任何的镜像为基础。接下来编写的指令将作为镜像的第一层存在。

## RUN 

`RUN`用来在定制镜像（image）时执行命令行命令的，有两种命令执行方式：

#### `shell` 执行

这种方式会在`shell`中执行命令，就像直接在命令行输入命令一样。

`shell`格式：

```
run <命令行命令>
# <命令行命令> 等同于在终端操作的 shell 的命令
```

整理了一份 [命令行命令](https://www.jianshu.com/p/3291de46f3ff)。可翻阅

举个🌰：

我们希望在定制镜像（image）是新建一个名字为 shuliqi.js 的文件。我们就可以这么写：

```
RUN touch shuliqi.js
```

然后我们定制镜像（image）文件`docker build -t dockerfile-example .`然后以该镜像（image）启动一个容器`docker run -t dockerfile-example `最后我们进入到这个容器看看：

```
$ docker container ls
CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS              PORTS               NAMES
0c3ac932b755        dockerfile-example   "/bin/bash"         59 seconds ago      Up 58 seconds                           compassionate_nobel
$ docker exec -t -i 0c3ac932b755 bash
root@0c3ac932b755:/# ls
bin  boot  dev	etc  home  lib	lib64  media  mnt  opt	proc  root  run  sbin  shuliq.js  srv  sys  tmp  usr  var
```

进入到容器之后， 我们使用ls 命令行命令，可以看出来，有文件名字为：shuliqi.js

#### `exec`执行

`exec`格式：

```
RUN ["可执行的额文件",“参数1”, "参数2"]
## 例如：
# RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline
```



`RUN`可以执行任何命令，然后再当前镜像上创建一个新层并且提交，提交后的结果镜像将会在 `Dockerfile`文件的下一步。

通过`RUN`执行多条命令时，可以通过`\`换行执行：

```
RUN touch shuliqi.js\
touch shuliqi2.js
```

也可以在同一行通过分好分隔命令：

```
RUN touch shuliqi.js; RUN touch shuliqi.js; \
```

**注意:**`RUN`指令创建的中间镜像会被缓存，并且在下次构建中使用。如果不想使用这些缓存镜像，可以在创建时指定`--no-cache`参数，如：

```
ocker build --no-cache .
```

例如：

{% asset_img 1.png %}

这样构建镜像的步骤就没有使用缓存，使得每一层的镜像ID都与之前的不同。

## CMD

 `CMD`用于在指定容器启动时所有执行的命令。`CMD`有以下三种格式：

```
CMD <commond> // shell 格式
CMD ["executable", "param1", "param2"] // exec格式，推荐格式
CMD ["param1", "param2"] // 为ENTRYPOINT指令提供参数
```

与`RUM`命令不同的是：

`RUM`指令在构建镜像时要执行命令。`CMD`则是用于在指定的容器启动时所要执行的命令。

`CMD`在 Dockerfile文件中仅可指定一次，指定多次时，会覆盖前面的指令。

**需要注意：**docker  run 命令会覆盖`CMD`命令。 如果 `docker run`运行容器时，使用了 `Dockerfile`中的 `CMD`命令相同的命令。就会覆盖 `Dockerfile`的`CMD`命令。

我们来举个🌰：

还是我们之前的项目，我们在 `Dockfile`文件中使用如下的命令：

```
CMD ["/bin/bash"]
```

我们使用`docker build -t dockerfile-example .`构建一个新的镜像，镜像的名字叫：dockerfile-example，构建完成之后， 我们使用这个镜像运行一个容器，运行效果如下：

```
docker run -t dockerfile-example
root@cfc0384da571:/#
```

那么容器终端将会使用 shell 。说明  `Dockfile`文件中的`CMD`起作用了。

但是我们不想使用 `Dockfile`文件中的`CMD`指定的命令。我们可以：

```
docker run -t dockerfile-example /bin/ps
  PID TTY          TIME CMD
    1 pts/0    00:00:00 ps
```

这时，`docker run`结尾指定的`/bin/ps`命令覆盖了`Dockerfile`的`CMD`中指定的命令。



## ENTRYPOINT

`ENTRYPOINT`与 `CMD`类似。

`ENTRYPOINT`有两种模式：

```
ENTRYPOINT <command> (shell模式)
ENTRYPOINT [ "executable", "param1", "param2" ] (exec模式)
```

但是`ENTRYPOINT`不会被 `docker run`中执行的命令覆盖，并且`docker run `命令中指定的任何参数都会被当成参数再次传递给 ENTRYPOINT`。如果想要覆盖 `ENTRYPOINT`，则需要在`docker run`中指定 `--entrypoint`选项。
`Dockerfile`中只允许有一个`ENTRYPOINT`，多指定时会覆盖前端的设置的`ENTRYPOINT`指令。 而只执行最后的 ENTRYPOINT指令。

举个例子：

我们重写我们的`Dockerfile`文件。添加 `ENTRYPOINT`指令：

```dockerfile
# Version: 0.0.3
FROM ubuntu:16.04
MAINTAINER 何民三 "cn.liuht@gmail.com"
RUN apt-get update
RUN apt-get install -y nginx
RUN echo 'Hello World, 我是个容器' \ 
   > /var/www/html/index.html
ENTRYPOINT ["/usr/sbin/nginx"]
EXPOSE 80
```

执行 `docker image build -t test2:0.0.1 .`构建我们的镜像、

构建完成之后， 启动一个容器：docker run -i -t  test2:0.0.1 -g "daemon off;"  ``

在运行容器时，我们使用了`-g "daemon off;"`. 这个参数会传递给 `ENTRYPOINT`。最终在容器中执行的命令为`/usr/sbin/nginx -g "daemon off;"`

## LABEL

`LABEL`用于为镜像添加元数据，元数据以键值对的形式指定：

```dockerfile
LABEL <key>=<value> <key>=<value> <key>=<value>
```

使用`LABEL`指定元数据时，一条`LABEL`指定可以指定一或多条元数据，指定多条元数据时不同元数据之间通过空格分隔。推荐将所有的元数据通过一条`LABEL`指令指定，以免生成过多的中间镜像。

如，通过`LABEL`指定一些元数据：

```dockerfile
LABEL name="shuliqi" age="23"
```

构建容器完后才能之后， 可以使用`docker inspect`查看

```dockerfile
docker inspect test3:0.0.1
```

{% asset_img 4.png %}

**注意：**`Dockerfile`中还有个 `MAINTAINER`用来指定镜像的作者。但是``MAINTAINER`并不推荐使用，更推荐使用`LATER`来指定镜像作者。如：

```dockerfile
LABEL maintainer="itbilu.com"
```



## EXPOSE

`EXPOSE`用来指定容器在运行的时监听的端口。

格式如下：

```dockerfile
EXPOSE <port> [<port> ...]
```

```dockerfile
EXPOSE 3000/tcp
```

注意：`EXPOSE`并不会让容器的端口访问到主机。要使其可以访问， 需要在`docker run`运行容器时通过 `-p`来发布这些端口。如：

```
docker run -t -p:8000:3000 express-for-docker:0.0.1
```

## ENV

`ENV`用于设置环境变量，有以下两种形式：

```
ENV <key> <value>
ENV <key>=<value> ...
```

例如：

```
ENV SHULIQI_PATH=/home/shuliqi/
```

设置完， 这个环境变量在 `ENV`命令之后都可以使用。如：

```
ENV SHULIQI_PATH=/home/shuliqi/
WORKDIR $SHULIQI_PATH
```

这个环境变量不仅可以在构建镜像中使用，使用该镜像创建的容器也可以使用。

## ADD

`ADD`指令用于复制构建环境中的文件/目录到镜像中。

有两种使用格式：

```
ADD <src>... <dest>
ADD ["<src>",... "<dest>"]
```

通过`ADD`复制文件时，需要通过<src>指定源文件位置，并通过`<dest>`来指定目标位置。<src>可以是一个构建上下文中的文件或目录，也可以是一个`URL`，但不能访问构建上下文之外的文件或目录。

如，通过`ADD`复制一个网络文件：

```
ADD http://wordpress.org/latest.zip $ITBILU_PATH
```

在上例中，`$ITBILU_PATH`是我们使用`ENV`指定的一个环境变量。

另外，如果使用的是本地归档文件（`gzip`、`bzip2`、`xz`）时，Docker会自动进行解包操作，类似使用`tar -x`。

## COPY

`COPY`同样用于复制构建环境中的文件或目录到镜像中。其有以下两种使用方式：

```
COPY <src>... <dest>
COPY ["<src>",... "<dest>"]
```

`COPY`指令非常类似于`ADD`，不同点在于`COPY`只会复制构建目录下的文件，不能使用`URL`也不会进行解压操作。

## VOLUME

`VOLUME`用于创建挂载点，即向所构建镜像创使的容器添加卷

格式：

```
VOLUME ["/data"]
```

一个卷可以存在于一个或多个容器的指定目录，该目录可以绕过联合文件系统，并具有以下功能：

- 卷可以容器间共享和重用
- 容器并不一定要和其它容器共享卷
- 修改卷后会立即生效
- 对卷的修改不会对镜像产生影响
- 卷会一直存在，直到没有任何容器在使用它

`VOLUME`让我们可以将源代码、数据或其它内容添加到镜像中，而又不并提交到镜像中，并使我们可以多个容器间共享这些内容。

如，通过`VOLUME`创建一个挂载点：

```
FROM node:12.17.0
COPY . /shuliqi
WORKDIR /shuliqi
RUN ["npm", "install"]
EXPOSE 3000/tcp
CMD ["/bin/bash"]
# `VOLUME`创建一个挂载点
ENV SHULIQI_PATH /myblog/
VOLUME [$SHULIQI_PATH]
```

构建的镜像，并指定镜像名为`express-for-docker`。构建镜像后，使用新构建的运行一个容器。运行容器时，需`-v`参将能本地目录绑定到容器的卷（挂载点）上，以使容器可以访问宿主机的数据。

```
$ docker run -i -t -v ~/myblog:/myblog/  express-for-docker:0.0.1
root@31b0fac536c4:/# cd /myblog/
root@31b0fac536c4:/myblog# ls
blog
```

{% asset_img 5.png %}

如上所示，我们已经可以容器的`/home/myblog/`目录下访问到宿主机`~/myblog目录下的数据了。

##  USER

`USER`用于指定运行镜像所使用的用户：

```
USER daemon
```

使用`USER`指定用户时，可以使用用户名、`UID`或`GID`，或是两者的组合。以下都是合法的指定试：

```
USER user
USER user:group
USER uid
USER uid:gid
USER user:gid
USER uid:group
```

使用`USER`指定用户后，`Dockerfile`中其后的命令`RUN`、`CMD`、`ENTRYPOINT`都将使用该用户。镜像构建完成后，通过`docker run`运行容器时，可以通过`-u`参数来覆盖所指定的用户。

## WORKDIR

`WORKDIR`用于在容器内设置一个工作目录：

```
WORKDIR /path/to/workdir
```

通过`WORKDIR`设置工作目录后，`Dockerfile`中其后的命令`RUN`、`CMD`、`ENTRYPOINT`、`ADD`、`COPY`等命令都会在该目录下执行。

如，使用`WORKDIR`设置工作目录：

```
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

在以上示例中，`pwd`最终将会在`/a/b/c`目录中执行。

在使用`docker run`运行容器时，可以通过`-w`参数覆盖构建时所设置的工作目录。

## ARG

`ARG`用于指定传递给构建运行时的变量：

```
ARG <name>[=<default value>]
```

如，通过`ARG`指定两个变量：

```
ARG site
ARG build_user=舒丽琦
```

以上我们指定了`site`和`build_user`两个变量，其中`build_user`指定了默认值。在使用`docker build`构建镜像时，可以通过`--build-arg <varname>=<value>`参数来指定或重设置这些变量的值。

```dockerfile
docker build --build-arg site=shuliqi.github.io -t express-for-docker .
```

这样我们构建了 express-for-docker 镜像，其中`site`会被设置为 shuliqi.github.io，由于没有指定`build_user`，其值将是默认值`舒丽琦`。

## ONBUILD

`ONBUILD`用于设置镜像触发器：

```
ONBUILD [INSTRUCTION]
```

当所构建的镜像被用做其它镜像的基础镜像，该镜像中的触发器将会被钥触发。

如，当镜像被使用时，可能需要做一些处理：

```
[...]
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
[...]
```

## STOPSIGNAL

`STOPSIGNAL`用于设置停止容器所要发送的系统调用信号：

```
STOPSIGNAL signal
```

所使用的信号必须是内核系统调用表中的合法的值，如：`9`、`SIGKILL`。

##  SHELL

`SHELL`用于设置执行命令（`shell`式）所使用的的默认`shell`类型：

```
SHELL ["executable", "parameters"]
```

`SHELL`在Windows环境下比较有用，Windows下通常会有`cmd`和`powershell`两种`shell`，可能还会有`sh`。这时就可以通过`SHELL`来指定所使用的`shell`类型：

```
FROM microsoft/windowsservercore

# Executed as cmd /S /C echo default
RUN echo default

# Executed as cmd /S /C powershell -command Write-Host default
RUN powershell -command Write-Host default

# Executed as powershell -command Write-Host hello
SHELL ["powershell", "-command"]
RUN Write-Host hello

# Executed as cmd /S /C echo hello
SHELL ["cmd", "/S"", "/C"]
RUN echo hello
```




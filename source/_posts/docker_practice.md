---
title: 《Docker 从入门到实践》学习笔记
date: 2022-04-28 23:56:21
tags: [CNCF, Docker]
categories: Docker
---


# 什么是 Docker

**Docker** 使用 `Google` 公司推出的 [Go 语言](https://golang.google.cn) 进行开发实现，基于 `Linux` 内核的 [cgroup](https://zh.wikipedia.org/wiki/Cgroups)，[namespace](https://en.wikipedia.org/wiki/Linux_namespaces)，以及 [OverlayFS](https://docs.docker.com/storage/storagedriver/overlayfs-driver/) 类的 [Union FS](https://en.wikipedia.org/wiki/Union_mount) 等技术，对进程进行封装隔离，属于 [操作系统层面的虚拟化技术](https://en.wikipedia.org/wiki/Operating-system-level_virtualization)。由于隔离的进程独立于宿主和其它的隔离的进程，因此也称其为容器。

![Docker在Linux中的架构](docker-on-linux-16488874880822.png)



## 传统虚拟机 VS Docker

传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；而容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。

![传统虚拟机](vm.png)

![Docker](docker.png)

<!-- more -->

# Docker 的三个基本概念

## 镜像（`Image`）
**Docker 镜像** 是一个特殊的root文件系统，比如官方镜像 `ubuntu:18.04` 就包含了完整的一套 Ubuntu 18.04 最小系统的 root 文件系统 (RootFS)。镜像除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。**镜像 *不包含* 任何动态数据，其内容在构建之后也不会被改变。**

### 分层存储

- 镜像并非是像一个 ISO 那样的打包文件，镜像只是一个虚拟的概念，其实际体现并非由一个文件组成，而是由一组文件系统组成，或者说，由多层文件系统联合组成。
- 镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。
- 分层存储的特征还使得镜像的复用、定制变的更为容易。甚至可以用之前构建好的镜像作为基础层，然后进一步添加新的层，以定制自己所需的内容，构建新的镜像。

![](image-20220412153434717.png)


## 容器（`Container`）
- 镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的 类 和 实例 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
- 容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 命名空间。因此容器可以拥有自己的 root 文件系统 (RootFS)、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。
- 镜像也使用分层存储。每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，我们可以称这个为容器运行时读写而准备的存储层为 **容器存储层**。容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，**任何保存于容器存储层的信息都会随容器删除而丢失。**
- **按照 Docker 最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。** 所有的文件写入操作，都应该使用 [数据卷（Volume）]()、或者 [绑定宿主目录]()，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高。数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此，使用数据卷后，容器删除或者重新运行之后，数据却不会丢失。

## 仓库（`Repository`）

[Docker Registry]() 是一个集中的存储、分发镜像的服务。

一个 Docker Registry 中可以包含多个 仓库（Repository）；每个仓库可以包含多个 标签（Tag）；每个标签对应一个镜像。

通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 `<仓库名>:<标签>` 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 `latest` 作为默认标签。

### 公有 Docker Registry
Docker Registry 公开服务是开放给用户使用、允许用户管理镜像的 Registry 服务。一般这类公开服务允许用户免费上传、下载公开的镜像，并可能提供收费服务供用户管理私有镜像。

### 私有 Docker Registry
除了使用公开服务外，用户还可以在本地搭建私有 Docker Registry。Docker 官方提供了 Docker Registry 镜像，可以直接使用做为私有 Registry 服务。

# 安装Docker

安装Docker的3种方式：
- 使用包管理器软件源安装，如`yum`、`dnf`、`apt`等；
- 下载预编译二进制安装包，如rpm，deb等；
- 使用便捷脚本（convenience scripts）安装Docker（不建议在生产环境部署时使用此方法）；

## 建立 docker 用户组
默认情况下，docker 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 root 用户和 docker 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 root 用户。因此，更好地做法是将需要使用 docker 的用户加入 docker 用户组。
建立 docker 组：

```bash
$ sudo groupadd docker
```
将当前用户加入 docker 组：
```bash
$ sudo usermod -aG docker $USER
```
退出当前终端并重新登录，进行如下测试。

## 镜像加速器

在 `/etc/docker/daemon.json` 中写入如下内容（如果文件不存在请新建该文件）：

```json
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
}
```

之后重新启动服务。

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

执行 `docker info`，如果从结果中看到了如下内容，说明配置成功。
```bash
$ docker info
Registry Mirrors:
 https://hub-mirror.c.163.com/
```

# Docker 镜像管理

## 获取镜像

从 Docker 镜像仓库获取镜像的命令是 `docker pull`。通过 `docker pull --help` 命令查看具体选项。其命令格式为：
```bash
$ docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```

镜像名称的格式：

- Docker 镜像仓库地址：地址的格式一般是 `<域名/IP>[:端口号]`。默认地址是 Docker Hub(`docker.io`)。
- 仓库名：如之前所说，这里的仓库名是两段式名称，即 `<用户名>/<软件名>`。对于 Docker Hub，如果不给出用户名，则默认为 `library`，也就是官方镜像。

实例：

```bash
$ docker pull ubuntu:18.04
18.04: Pulling from library/ubuntu
92dc2a97ff99: Pull complete
be13a9d27eb8: Pull complete
c8299583700a: Pull complete
Digest: sha256:4bc3ae6596938cb0d9e5ac51a1152ec9dcac2a1c50829c74abd9c4361e321b26
Status: Downloaded newer image for ubuntu:18.04
docker.io/library/ubuntu:18.04
```



## 运行

```bash
$ docker run -it --rm ubuntu:18.04 bash

root@e7009c6ce357:/# cat /etc/os-release
NAME="Ubuntu"
VERSION="18.04.1 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.1 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic
```

使用命令`docker run` 运行容器，具体格式我们会在 [容器]() 一节进行详细讲解，我们这里简要的说明一下上面用到的参数。

- `-it`：这是两个参数，一个是 `-i`：交互式操作，一个是 `-t` 终端。我们这里打算进入 `bash` 执行一些命令并查看返回结果，因此我们需要交互式终端。
- `--rm`：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 `docker rm`。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 `--rm` 可以避免浪费空间。
- `镜像名:标签`：这是指用特定镜像为基础来启动容器，如 `ubuntu:18.04` 。
- `bash`：放在镜像名后的是 **命令**，这里我们希望有个交互式 Shell，因此用的是 `bash`。

## 列出本地镜像

查看本地镜像可以使用：

- `docker images`
- `docker image ls`

使用`docker image ls --help`查看命令帮助：

```bash
$ docker image ls --help

Usage:  docker image ls [OPTIONS] [REPOSITORY[:TAG]]

List images

Aliases:
  ls, list

Options:
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show image IDs
```

`docker image ls`默认列出所有顶层镜像，列表包含了 `仓库名`、`标签`、`镜像 ID`、`创建时间` 以及 `所占用的空间`：

```bash
$ docker image ls
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
redis                latest              5f515359c7f8        5 days ago          183 MB
nginx                latest              05a60462f8ba        5 days ago          181 MB
mongo                3.2                 fe9198c04d62        5 days ago          342 MB
<none>               <none>              00285df0df87        5 days ago          342 MB
ubuntu               18.04               329ed837d508        3 days ago          63.3MB
ubuntu               bionic              329ed837d508        3 days ago          63.3MB
```

### 镜像体积

`docker image ls`列出的所占用空间和在 Docker Hub 上看到的镜像大小不同。比如，`ubuntu:18.04` 镜像大小，在这里是 `63.3MB`，但是在 [Docker Hub](https://hub.docker.com/layers/ubuntu/library/ubuntu/bionic/images/sha256-32776cc92b5810ce72e77aca1d949de1f348e1d281d3f00ebcc22a3adcdc9f42?context=explore) 显示的却是 `25.47 MB`。Docker Hub 中显示的体积是压缩后的体积，而 `docker image ls` 显示的是镜像下载到本地后，展开后的各层所占空间的总和。

`docker image ls` 列表中的镜像体积总和并非是所有镜像实际硬盘消耗。由于 Docker 镜像是多层存储结构，并且可以继承、复用，因此不同镜像可能会因为使用相同的基础镜像，从而拥有共同的层。由于 Docker 使用 Union FS，相同的层只需要保存一份即可，因此实际镜像硬盘占用空间很可能要比这个列表镜像大小的总和要小的多。

镜像、容器、数据卷实际所占用的空间通过 `docker system df` 或`docker system df -v`命令查看：

```bash
$ docker system df

TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              24                  0                   1.992GB             1.992GB (100%)
Containers          1                   0                   62.82MB             62.82MB (100%)
Local Volumes       9                   0                   652.2MB             652.2MB (100%)
Build Cache                                                 0B                  0B
```

### 虚悬镜像 (dangling image)

`docker pull` 和`docker build` 操作可能导致**虚悬镜像 (dangling image)**。由于新旧镜像同名，旧镜像名称被取消，从而出现仓库名、标签均为 `<none>` 的镜像。

上面的镜像列表中，还可以看到一个特殊的镜像，这个镜像既没有仓库名，也没有标签，均为 `<none>`：
```bash
<none>               <none>              00285df0df87        5 days ago          342 MB
```

这个镜像原本是有镜像名和标签的，原来为 `mongo:3.2`，随着官方镜像维护，发布了新版本后，重新 `docker pull mongo:3.2` 时，`mongo:3.2` 这个镜像名被转移到了新下载的镜像身上，而旧的镜像上的这个名称则被取消，从而成为了 `<none>`。

这类无标签镜像也被称为 **虚悬镜像(dangling image)** ，可以用下面的命令专门显示这类镜像：

```bash
$ docker image ls -f dangling=true

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              00285df0df87        5 days ago          342 MB
```

一般来说，虚悬镜像已经失去了存在的价值，可以用下面的命令删除：
```bash
$ docker image prune
```

### 中间层镜像

为了加速镜像构建、重复利用资源，Docker 会利用 **中间层镜像**。所以在使用一段时间后，可能会看到一些依赖的中间层镜像。

默认的 `docker image ls` 列表中只会显示顶层镜像，如果希望显示包括中间层镜像在内的所有镜像的话，需要加 `-a` 参数。

```bash
$ docker image ls -a
```

这样会看到很多无标签的镜像，与之前的虚悬镜像不同，这些无标签的镜像很多都是中间层镜像，是其它镜像所依赖的镜像。这些无标签镜像不应该删除，否则会导致上层镜像因为依赖丢失而出错。实际上，这些镜像也没必要删除，因为之前说过，相同的层只会存一遍，而这些镜像是别的镜像的依赖，因此并不会因为它们被列出来而多存了一份，无论如何你也会需要它们。只要删除那些依赖它们的镜像后，这些依赖的中间层镜像也会被连带删除。

### 列出部分镜像

不加任何参数的情况下，`docker image ls` 会列出所有顶层镜像，但是有时候我们只希望列出部分镜像。

根据仓库名列出镜像：
```bash
$ docker image ls ubuntu
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              18.04               329ed837d508        3 days ago          63.3MB
ubuntu              bionic              329ed837d508        3 days ago          63.3MB
```

列出特定的某个镜像，也就是说指定仓库名和标签
```bash
$ docker image ls ubuntu:18.04
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              18.04               329ed837d508        3 days ago          63.3MB
```

除此以外，`docker image ls` 还支持强大的过滤器参数 `--filter`，或者简写 `-f`。之前我们已经看到了使用过滤器来列出虚悬镜像的用法，它还有更多的用法。比如，我们希望看到在 `mongo:3.2` 之后建立的镜像，可以用下面的命令：
```bash
$ docker image ls -f since=mongo:3.2
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               latest              5f515359c7f8        5 days ago          183 MB
nginx               latest              05a60462f8ba        5 days ago          181 MB
```

想查看某个位置之前的镜像也可以，只需要把 `since` 换成 `before` 即可。

此外，如果镜像构建时，定义了 `LABEL`，还可以通过 `LABEL` 来过滤。
```bash
$ docker image ls -f label=com.example.version=0.1
```

### 只显示镜像ID

使用 `-q` 参数只显示镜像ID：

```bash
$ docker image ls -q
5f515359c7f8
05a60462f8ba
fe9198c04d62
00285df0df87
329ed837d508
329ed837d508
```

`--filter` 配合 `-q` 产生出指定范围的 ID 列表，然后送给另一个 `docker` 命令作为参数，从而针对这组实体成批的进行某种操作的做法在 Docker 命令行使用过程中非常常见，不仅仅是镜像，将来我们会在各个命令中看到这类搭配以完成很强大的功能。因此每次在文档看到过滤器后，可以多注意一下它们的用法。

### 以特定格式显示

另外一些时候，我们可能只是对表格的结构不满意，希望自己组织列；或者不希望有标题，这样方便其它程序解析结果等，这就用到了 [Go 的模板语法](https://gohugo.io/templates/introduction/)。

比如，下面的命令会直接列出镜像结果，并且只包含镜像ID和仓库名：

```bash
$ docker image ls --format "{{.ID}}: {{.Repository}}"
5f515359c7f8: redis
05a60462f8ba: nginx
fe9198c04d62: mongo
00285df0df87: <none>
329ed837d508: ubuntu
329ed837d508: ubuntu
```

或者打算以表格等距显示，并且有标题行，和默认一样，不过自己定义列：

```bash
$ docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
IMAGE ID            REPOSITORY          TAG
5f515359c7f8        redis               latest
05a60462f8ba        nginx               latest
fe9198c04d62        mongo               3.2
00285df0df87        <none>              <none>
329ed837d508        ubuntu              18.04
329ed837d508        ubuntu              bionic
```

## 删除本地镜像

如果要删除本地的镜像，可以使用以下2个命令：

- `docker rmi`命令

-  `docker image rm` 命令

其格式为：

```bash
$ docker rmi [选项] <镜像1> [<镜像2> ...]
$ docker image rm [选项] <镜像1> [<镜像2> ...]
```

### 用 ID、镜像名、摘要删除镜像
比如我们有这么一些镜像：

```bash
$ docker image ls
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
centos                      latest              0584b3d2cf6d        3 weeks ago         196.5 MB
redis                       alpine              501ad78535f0        3 weeks ago         21.03 MB
docker                      latest              cf693ec9b5c7        3 weeks ago         105.1 MB
nginx                       latest              e43d811ce2f4        5 weeks ago         181.5 MB
```

其中，`<镜像>` 可以是：

- `镜像长 ID`：镜像的完整 ID，也称为 `长 ID`，来删除镜像。
- `镜像短 ID`：`docker image ls` 默认列出的是短 ID 。通常情况下，镜像ID不需要写完整，只要短ID的前几个字符能唯一确定一个镜像即可。
- `镜像名` ：即 `<仓库名>:<标签>`。
- `镜像摘要`：镜像的sha256哈希摘要。

比如这里，如果我们要删除 `redis:alpine` 镜像，短ID`501ad78535f0`的前3个字符`501`就可以唯一确定此镜像，可以执行`docker image rm 501`删除此镜像：

```bash
$ docker image rm 501
Untagged: redis:alpine
Untagged: redis@sha256:f1ed3708f538b537eb9c2a7dd50dc90a706f7debd7e1196c9264edeea521a86d
Deleted: sha256:501ad78535f015d88872e13fa87a828425117e3d28075d0c117932b05bf189b7
Deleted: sha256:96167737e29ca8e9d74982ef2a0dda76ed7b430da55e321c071f0dbff8c2899b
Deleted: sha256:32770d1dcf835f192cafd6b9263b7b597a1778a403a109e2cc2ee866f74adf23
Deleted: sha256:127227698ad74a5846ff5153475e03439d96d4b1c7f2a449c7a826ef74a2d2fa
Deleted: sha256:1333ecc582459bac54e1437335c0816bc17634e131ea0cc48daa27d32c75eab3
Deleted: sha256:4fc455b921edf9c4aea207c51ab39b10b06540c8b4825ba57b3feed1668fa7c7
```

我们也可以用`镜像名`，也就是 `<仓库名>:<标签>`，来删除镜像。

```bash
$ docker image rm centos
Untagged: centos:latest
Untagged: centos@sha256:b2f9d1c0ff5f87a4743104d099a3d561002ac500db1b9bfa02a783a46e0d366c
Deleted: sha256:0584b3d2cf6d235ee310cf14b54667d889887b838d3f3d3033acd70fc3c48b8a
Deleted: sha256:97ca462ad9eeae25941546209454496e1d66749d53dfa2ee32bf1faabd239d38
```

当然，更精确的是使用 `镜像摘要` 删除镜像。

```bash
$ docker image ls --digests
REPOSITORY                  TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
node                        slim                sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228   6e0c4c8e3913        3 weeks ago         214 MB

$ docker image rm node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
Untagged: node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
```

可以使用 `docker image ls -q` 来配合使用 `docker image rm`，这样可以成批的删除希望删除的镜像。比如，我们需要删除所有仓库名为 `redis` 的镜像：

```bash
docker image rm $(docker image ls -q redis)
```

或者删除所有在 `mongo:3.2` 之前的镜像

```bash
$ docker image rm $(docker image ls -q -f before=mongo:3.2)
```

### 镜像的Untagged 和 Deleted

![](image-20220412153434717.png)

- 镜像由多层镜像联合组成，包含对其父镜像、父镜像的父镜像、祖先镜像等直至基础镜像的所有操作的总和。
- 对基础镜像或已有镜像进行修改，可产生新的顶层镜像。被修改的基础进行或已有镜像就成为新的顶层镜像的父镜像。新产生的顶层镜像会引用其父镜像（同时父镜像也会引用自己的父镜像），并作为一个整体形成一个完整的新镜像。
- 镜像的唯一标识是其 ID 和摘要。
- 多个不同镜像可能依赖的“层”完全相同，换句话说，多个依赖（引用）的“层”完全相同的镜像可以是不同镜像，因为它们ID不同。例如，可以使用`docker commit`将同一个容器进行`commit`为多个不同镜像，并设置不同标签。
- 删除镜像时，删除行为分为2个阶段：`Untagged`和`Deleted`。首先会对该镜像取消标签 (untag) ，然后再删除 (delete)该镜像。但当该镜像被其他镜像依赖（引用）时，只能先将镜像`Untagged`，但无法立即`Deleted`该镜像。直到没有任何层依赖当前镜像时，才会真实的删除当前镜像。
- 除了镜像依赖以外，还需要注意的是容器对镜像的依赖。如果有用这个镜像启动的容器存在（即使容器没有运行），那么同样不可以删除这个镜像。

## 由容器创建镜像

镜像是多层存储，每一层是在前一层的基础上进行的修改；而容器以镜像为基础层，在其基础上加一层可读写的容器存储层。镜像是静态的，容器是动态的。在容器运行过程中，可以动态的修改容器。

使用`docker commit`命令可以将容器的存储层保存为镜像，即将修改后的容器状态固化为镜像。

`ocker commit` 的语法格式为：

```bash
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
```

例如，运行ubuntu镜像，创建容器`ub`，在`ub`中创建文件：

```bash
[root@localhost docker]# docker run -it --name=ub ubuntu bash
root@3f67f307e836:/# touch f1.txt
root@3f67f307e836:/# echo "create file f1.txt" > f1.txt 
root@3f67f307e836:/# exit
exit
```

并将修改后的容器`ub`固化为镜像`from-ub:latest`：
```bash
[root@localhost docker]# docker commit -m 'create file f1.txt' ub from-ub:latest
sha256:ccc63c912efae74b3f547a6c0d23f0434fc3215d538f625f09fe4cc5f7d4d7d8
```

查看本地镜像：

```bash
[root@localhost docker]# docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
from-ub       latest    ccc63c912efa   12 minutes ago   72.8MB
ubuntu        latest    825d55fb6340   5 days ago       72.8MB
nginx         1.21.6    12766a6745ee   12 days ago      142MB
hello-world   latest    feb5d9fea6a5   6 months ago     13.3kB
```

使用`docker diff`检查对容器文件系统中文件或目录的修改：

```bash
[root@localhost docker]# docker diff ub
C /root
A /root/.bash_history
A /f1.txt
```

使用`docker image history`命令查看对镜像的修改历史：
```bash
[root@localhost docker]# docker image history from-ub:latest
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
ccc63c912efa   35 seconds ago   bash                                            73B       create file f1.txt
825d55fb6340   5 days ago       /bin/sh -c #(nop)  CMD ["bash"]                 0B        
<missing>      5 days ago       /bin/sh -c #(nop) ADD file:b83df51ab7caf8a4d…   72.8MB    
[root@localhost docker]# docker image history ubuntu:latest 
IMAGE          CREATED      CREATED BY                                      SIZE      COMMENT
825d55fb6340   5 days ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B        
<missing>      5 days ago   /bin/sh -c #(nop) ADD file:b83df51ab7caf8a4d…   72.8MB
```

### 慎用 `docker commit`

由于容器是动态的，在容器运行过程中，容器文件系统中的很多文件都可能被修改过。所以由`docker commit`生成镜像，可能会有大量的无关内容被添加进来，将会导致镜像极为臃肿。而且对于这些修改，我们是无法确切知道到底是做了哪些修改的。

## 使用 Dockerfile 定制镜像

使用Dockerfile允许我们以一个镜像作为基础，在其上进行定制。Dockerfile 是一个文本文件，其内包含了一条条的 **指令(Instruction)**，***每一条指令构建一层***，因此每一条指令的内容，就是描述该层应当如何构建。

在 [Docker Hub](https://hub.docker.com/search?q=&type=image&image_filter=official) 上有非常多的高质量的官方镜像，可以在其中寻找一个最符合我们最终目标的镜像为基础镜像进行定制。

下面以定制 `nginx` 镜像为例，使用 Dockerfile 来定制镜像`mynginx`。在一个空白目录中，建立一个文本文件，并命名为 `Dockerfile`：

```bash
$ mkdir mynginx
$ cd mynginx
$ touch Dockerfile
```

编辑`Dockerfile`内容如下：

```dockerfile
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

### 构建镜像

在`Dockerfile`所在目录中执行`docker build`命令构建镜像：

```bash
$ docker build -t nginx:v3 .
Sending build context to Docker daemon 2.048 kB
Step 1 : FROM nginx
 ---> e43d811ce2f4
Step 2 : RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
 ---> Running in 9cdc27646c7b
 ---> 44aa4490ce2c
Removing intermediate container 9cdc27646c7b
Successfully built 44aa4490ce2c
```

从命令的输出结果可以看到，在 `Step 2` 中，`RUN` 指令启动了一个中间容器 `9cdc27646c7b`，执行了所要求的命令，随后删除了所用到的这个中间容器 `9cdc27646c7b`，并最后提交了这一层 `44aa4490ce2c`。

### 镜像构建上下文（Context）

Docker 是基于C/S架构设计的，分为 Docker 引擎（也就是服务端守护进程）和客户端工具。表面上我们好像是在本机执行各种 `docker` 功能，但实际上，一切都是使用的远程调用形式在服务端（Docker 引擎）完成。而 `docker build` 命令构建镜像，其实并非在本地构建，而是在服务端，也就是 Docker 引擎中构建的。

刚才的命令 `docker build -t nginx:v3 .` 中的这个 `.`，实际上是在指定 ***上下文的目录***，`docker build` 命令会将该目录下的内容打包交给 Docker 引擎以帮助构建镜像。

因此，类似`COPY` 这类指令中的 ***源文件的路径*** 都是***相对路径***，引用的是打包上传后的上下文目录。 所以`COPY ../package.json /app` 或者 `COPY /opt/xxxx /app` 是无法使用的，因为这些路径已经超出了上下文的范围，Docker 引擎无法获得这些位置的文件。

如果观察 `docker build` 输出，我们其实已经看到了这个发送上下文的过程：

```bash
$ docker build -t nginx:v3 .
Sending build context to Docker daemon 2.048 kB
```

一般来说，应该会将 `Dockerfile` 置于一个空目录下，或者项目根目录下。如果该目录下没有所需文件，那么应该把所需文件复制一份过来。如果目录下有些东西确实不希望构建时传给 Docker 引擎，那么可以用 `.gitignore` 一样的语法写一个 `.dockerignore`，该文件是用于剔除不需要作为上下文传递给 Docker 引擎的。

在默认情况下，如果不额外指定 `Dockerfile` 的话，会将上下文目录下的名为 `Dockerfile` 的文件作为 Dockerfile。实际上 `Dockerfile` 的文件名并不要求必须为 `Dockerfile`，而且并不要求必须位于上下文目录中，比如可以用 `-f ../Dockerfile.php` 参数指定某个文件作为 `Dockerfile`。

### `docker build`的其他构建方式

#### 由URL构建

`docker build` 还支持从 URL 构建，比如可以直接从 Git repo 中构建：

```bash
# $env:DOCKER_BUILDKIT=0
# export DOCKER_BUILDKIT=0

$ docker build -t hello-world https://github.com/docker-library/hello-world.git#master:amd64/hello-world

Step 1/3 : FROM scratch
 --->
Step 2/3 : COPY hello /
 ---> ac779757d46e
Step 3/3 : CMD ["/hello"]
 ---> Running in d2a513a760ed
Removing intermediate container d2a513a760ed
 ---> 038ad4142d2b
Successfully built 038ad4142d2b
```

如果所给出的 URL 不是个 Git repo，而是个 `tar` 压缩包，那么 Docker 引擎会下载这个包，并自动解压缩，以其作为上下文，开始构建。

```bash
docker build http://server/context.tar.gz
```

#### 由标准输入构建

从标准输入中读取 Dockerfile 进行构建：

```bash
docker build - < Dockerfile
```

或

```bash
cat Dockerfile | docker build -
```

或者，从标准输入中读取上下文压缩包进行构建

```bash
$ docker build - < context.tar.gz
```

**这种形式由于直接从标准输入中读取 Dockerfile 的内容，它没有上下文，因此不可以像其他方法那样可以将本地文件 `COPY` 进镜像之类的事情。**

## `Dockerfile`指令详解

### `FROM` 指定基础镜像

 `FROM` 指定以一个镜像为 **基础镜像**，因此一个 `Dockerfile` 中 `FROM` 是必备的指令，并且必须是第1条指令。

除了选择现有镜像为基础镜像外，Docker 还存在一个特殊的镜像，名为 `scratch`。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。如果你以 scratch 为基础镜像的话，意味着你不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。

```dockerfile
FROM scratch
...
```

### `RUN` 执行命令

`RUN`用来执行命令行命令。由于命令行的强大能力，`RUN` 指令在定制镜像时是最常用的指令之一。其格式有2种：

- *shell* 格式：`RUN <命令>`，就像直接在命令行中输入的命令一样。
- *exec* 格式：`RUN ["可执行文件", "参数1", "参数2"]`，这更像是函数调用中的格式。

Dockerfile 中每一个指令都会建立一层，`RUN` 也不例外。**每一个 `RUN` 的行为会创建一层镜像。而下面的这种写法，创建了 7 层镜像。这是完全没有意义的，而且产生非常臃肿、非常多层的镜像。**

> **注意：Union FS 是有最大层数限制的，比如 AUFS，曾经是最大不得超过 42 层，现在是不得超过 127 层。**

```dockerfile
FROM debian:stretch

RUN apt-get update
RUN apt-get install -y gcc libc6-dev make wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN mkdir -p /usr/src/redis
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
RUN make -C /usr/src/redis
RUN make -C /usr/src/redis install
```

上面写法应改为下面这样：

```dockerfile
FROM debian:stretch

RUN set -x; buildDeps='gcc libc6-dev make wget' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

这里没有使用很多个 `RUN`指令，而是仅仅通过一个 `RUN` 指令，使用 `&&` 将各个命令串联起来，并使用`\`将行尾换行符转义。从而将之前的 7 层镜像，简化为了 1 层。

可以看到，为了尽可能减少镜像体积，避免构建出臃肿的镜像；这一组命令的最后添加了清理工作的命令，删除了为了编译构建所需要的软件，清理了所有下载、展开的文件，并且还清理了 `apt` 缓存文件。

### `COPY` 复制文件

`COPY` 指令将从构建上下文目录中 `<源路径>` 的文件/目录复制到新的一层的镜像内的 `<目标路径>` 位置。

和 `RUN` 指令一样，也有两种格式，一种类似于命令行，一种类似于函数调用：

- `COPY [--chown=<user>:<group>] <源路径>... <目标路径>`
- `COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]`

如：

```bash
COPY package.json /usr/src/app/
```

`<源路径>` 可以是多个，甚至可以是通配符，其通配符规则要满足 Go 的 [`filepath.Match`](https://golang.org/pkg/path/filepath/#Match) 规则，如：

```bash
COPY hom* /mydir/
COPY hom?.txt /mydir/
```

`<目标路径>` 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 `WORKDIR` 指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。

**使用 `COPY` 指令，源文件的各种元数据都会保留。**比如读、写、执行权限、文件变更时间等。这个特性对于镜像定制很有用。特别是构建相关文件都在使用 Git 进行管理的时候。

在使用该指令的时候还可以加上 `--chown=<user>:<group>` 选项来改变文件的所属用户及所属组。

```bash
COPY --chown=55:mygroup files* /mydir/
COPY --chown=bin files* /mydir/
COPY --chown=1 files* /mydir/
COPY --chown=10:11 files* /mydir/
```

如果源路径为文件夹，复制的时候不是直接复制该文件夹，而是将文件夹中的内容复制到目标路径。

### `ADD` 更高级的复制文件

`ADD` 指令和 `COPY` 的格式和性质基本一致。但是在 `COPY` 基础上增加了一些功能。

比如 `<源路径>` 可以是一个 `URL`，这种情况下，Docker 引擎会试图去下载这个链接的文件放到 `<目标路径>` 去。下载后的文件权限自动设置为 `600`。

如果 `<源路径>` 为一个 `tar` 压缩文件的话，压缩格式为 `gzip`, `bzip2` 以及 `xz` 的情况下，`ADD` 指令将会自动解压缩这个压缩文件到 `<目标路径>` 去。

在 `COPY` 和 `ADD` 指令中选择的时候，应遵循 Docker 官方的 [Dockerfile 最佳实践文档]() 中提出的原则，所有的文件复制均使用 `COPY` 指令，仅在需要自动解压缩的场合使用 `ADD`。

在使用该指令的时候还可以加上 `--chown=<user>:<group>` 选项来改变文件的所属用户及所属组。

```bash
ADD --chown=55:mygroup files* /mydir/
ADD --chown=bin files* /mydir/
ADD --chown=1 files* /mydir/
ADD --chown=10:11 files* /mydir/
```

### `CMD` 容器启动命令

`CMD` 指令就是用于指定默认的容器主进程的启动命令的。Docker 不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。

`CMD` 指令的格式：

- `shell` 格式：`CMD <命令>`
- `exec` 格式：`CMD ["可执行文件", "参数1", "参数2"...]`
- 参数列表格式：`CMD ["参数1", "参数2"...]`。在指定了 `ENTRYPOINT` 指令后，用 `CMD` 指定具体的参数。

在指令格式上，一般推荐使用 `exec` 格式，这类格式在解析时会被解析为 JSON 数组，因此一定要使用双引号 `"`，而不要使用单引号。

Docker 不是虚拟机，**容器中的应用都应该以前台执行**，而不是像虚拟机、物理机里面那样，用 `systemd` 去启动后台服务，容器内没有后台服务的概念。

**对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。**

一些初学者将 `CMD` 写为：

```dockerfile
CMD service nginx start
```

使用 `service nginx start` 命令，则是希望 upstart 来以后台守护进程形式启动 `nginx` 服务。而刚才说了 `CMD service nginx start` 会被理解为 `CMD [ "sh", "-c", "service nginx start"]`，因此主进程实际上是 `sh`。那么当 `service nginx start` 命令结束后，`sh` 也就结束了，`sh` 作为主进程退出了，自然就会令容器退出。

正确的做法是直接执行 `nginx` 可执行文件，并且要求以前台形式运行。比如：

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

### `ENTRYPOINT` 入口点

`ENTRYPOINT` 的目的和 `CMD` 一样，都是在指定容器启动程序及参数。`ENTRYPOINT` 在运行时也可以替代，不过比 `CMD` 要略显繁琐，需要通过 `docker run` 的参数 `--entrypoint` 来指定。

当指定了 `ENTRYPOINT` 后，`CMD` 的含义就发生了改变，不再是直接的运行其命令，而是将 `CMD` 的内容作为参数传给 `ENTRYPOINT` 指令，换句话说实际执行时，将变为：

```bash
<ENTRYPOINT> "<CMD>"
```

#### 场景1：让镜像变成像命令一样使用

假设我们要构建一个得知自己当前公网 IP 的镜像。

##### （1）使用`CMD`来实现：

```dockerfile
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
CMD [ "curl", "-s", "http://myip.ipip.net" ]
```

使用 `docker build -t myip .` 来构建镜像，然后执行：

```bash
$ docker run myip
当前 IP：61.148.226.66 来自：北京市 联通
```

假如我们修改镜像执行curl的参数：

```bash
$ docker run myip -i
docker: Error response from daemon: invalid header field value "oci runtime error: container_linux.go:247: starting container process caused \"exec: \\\"-i\\\": executable file not found in $PATH\"\n".
```

跟在镜像名后面的是 `command`，运行时会替换 Dockerfile中`CMD` 的默认值。因此这里的 `-i` 替换了一个新的 `CMD`指令。自然是找不到`-i`这个命令的。

如果我们希望加入 `-i` 这参数，我们就必须重新完整的输入这个命令：

```bash
$ docker run myip curl -s http://myip.ipip.net -i
```



##### （2）使用`ENTRYPOINT`来实现：

```dockerfile
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT [ "curl", "-s", "http://myip.ipip.net" ]
```

直接使用 `docker run myip -i`：

```bash
$ docker run myip
当前 IP：61.148.226.66 来自：北京市 联通

$ docker run myip -i
HTTP/1.1 200 OK
Server: nginx/1.8.0
Date: Tue, 22 Nov 2016 05:12:40 GMT
Content-Type: text/html; charset=UTF-8
Vary: Accept-Encoding
X-Powered-By: PHP/5.6.24-1~dotdeb+7.1
X-Cache: MISS from cache-2
X-Cache-Lookup: MISS from cache-2:80
X-Cache: MISS from proxy-2_6
Transfer-Encoding: chunked
Via: 1.1 cache-2:80, 1.1 proxy-2_6:8006
Connection: keep-alive

当前 IP：61.148.226.66 来自：北京市 联通
```

当存在 `ENTRYPOINT` 时，`CMD` 的内容将会作为参数传给（追加给） `ENTRYPOINT`，而这里 `-i` 就是新的 `CMD`，因此会作为参数传给 `curl -s http://myip.ipip.net`，从而达到了我们预期的效果。

#### 场景2：应用运行前的准备工作

有时候，启动容器（启动容器就是启动主进程）前，需要一些准备工作。比如 `mysql` 类的数据库，可能需要一些数据库配置、初始化的工作，这些工作要在最终的 mysql 服务器运行之前解决。可能希望避免使用 `root` 用户去启动服务，从而提高安全性，而在启动服务前还需要以 `root` 身份执行一些必要的准备工作，最后切换到服务用户身份启动服务。

这种情况下，可以写一个脚本，然后放入 `ENTRYPOINT` 中去执行，而这个脚本会将接到的参数（也就是 `<CMD>`）作为命令，在脚本最后执行。比如官方镜像 `redis` 中就是这么做的：

```bash
FROM alpine:3.4
...
RUN addgroup -S redis && adduser -S -G redis redis
...
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 6379
CMD [ "redis-server" ]
```

可以看到其中为了 redis 服务创建了 redis 用户，并在最后指定了 `ENTRYPOINT` 为 `docker-entrypoint.sh` 脚本。

```bash
#!/bin/sh
...
# allow the container to be started with `--user`
if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then
	find . \! -user redis -exec chown redis '{}' +
	exec gosu redis "$0" "$@"
fi

exec "$@"
```

该脚本的内容就是根据 `CMD` 的内容来判断，如果是 `redis-server` 的话，则切换到 `redis` 用户身份启动服务器，否则依旧使用 `root` 身份执行。比如：

```bash
$ docker run -it redis id
uid=0(root) gid=0(root) groups=0(root)
```

### `ENV` 设置环境变量
`ENV` 指令设置环境变量，无论是后面的其它指令，如 RUN，还是运行时的应用，都可以直接使用这里定义的环境变量。

格式：

- `ENV <key> <value>`

- `ENV <key1>=<value1> <key2>=<value2>...`


```dockerfile
ENV VERSION=1.0 DEBUG=on \
    NAME="Happy Feet"
```

定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。比如在官方 node 镜像 Dockerfile 中，就有类似这样的代码：
```dockerfile
ENV NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 \
  && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
  && ln -s /usr/local/bin/node /usr/local/bin/nodejs
```
在这里先定义了环境变量 NODE_VERSION，其后的 RUN 这层里，多次使用 $NODE_VERSION 来进行操作定制。

下列指令可以支持环境变量展开： `ADD`、`COPY`、`ENV`、`EXPOSE`、`FROM`、`LABEL`、`USER`、`WORKDIR`、`VOLUME`、`STOPSIGNAL`、`ONBUILD`、`RUN`。

### `ARG` 构建时环境变量

格式：

- `ARG <参数名>[=<默认值>]`

构建参数和 `ENV` 的效果类似，也可以设置环境变量。不同的是，`ARG` 所设置的是构建环境时的环境变量，在将来容器运行时是不会存在这些环境变量的。

> 注意：不要使用 `ARG` 保存密码之类的信息，因为 `docker history` 还是可以看到所有值的。

`Dockerfile` 中的 `ARG` 指令是定义参数名称，以及定义其默认值。该默认值可以在构建命令 `docker build` 中用 `--build-arg <参数名>=<值>` 来覆盖。灵活的使用 `ARG` 指令，能够在不修改 Dockerfile 的情况下，构建出不同的镜像。

ARG 指令有生效范围，如果在 `FROM` 指令之前指定，那么只能用于 `FROM` 指令中。

```dockerfile
ARG DOCKER_USERNAME=library

FROM ${DOCKER_USERNAME}/alpine

RUN set -x ; echo ${DOCKER_USERNAME}
```

使用上述 Dockerfile 会发现无法输出 `${DOCKER_USERNAME}` 变量的值，要想正常输出，你必须在 `FROM` 之后再次指定 `ARG`：

```dockerfile
# 只在 FROM 中生效
ARG DOCKER_USERNAME=library

FROM ${DOCKER_USERNAME}/alpine

# 要想在 FROM 之后使用，必须再次指定
ARG DOCKER_USERNAME=library

RUN set -x ; echo ${DOCKER_USERNAME}
```

对于多阶段构建，尤其要注意这个问题。下面2个 `FROM` 指令都可以使用 `${DOCKER_USERNAME}`：

```dockerfile
# 这个变量在每个 FROM 中都生效
ARG DOCKER_USERNAME=library

FROM ${DOCKER_USERNAME}/alpine

RUN set -x ; echo 1

FROM ${DOCKER_USERNAME}/alpine

RUN set -x ; echo 2
```

对于在各个阶段中使用的变量都必须在每个阶段分别指定：

```dockerfile
ARG DOCKER_USERNAME=library

FROM ${DOCKER_USERNAME}/alpine

# 在FROM 之后使用变量，必须在每个阶段分别指定
ARG DOCKER_USERNAME=library

RUN set -x ; echo ${DOCKER_USERNAME}

FROM ${DOCKER_USERNAME}/alpine

# 在FROM 之后使用变量，必须在每个阶段分别指定
ARG DOCKER_USERNAME=library

RUN set -x ; echo ${DOCKER_USERNAME}
```

### `VOLUME` 定义匿名卷

容器运行时应该尽量保持容器存储层不发生写操作，对于数据库类需要保存动态数据的应用，其数据库文件应该保存于卷(volume)中。为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在 `Dockerfile` 中，可以使用`VOLUME`指令事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。格式为：

- `VOLUME ["<路径1>", "<路径2>"...]`
- `VOLUME <路径>`

下面的 `/data` 目录就会在容器运行时自动挂载为匿名卷，任何向 `/data` 中写入的信息都不会记录进容器存储层，从而保证了容器存储层的无状态化。

```dockerfile
VOLUME /data
```

运行容器时可以覆盖这个挂载设置。比如，使用了 `mydata` 这个命名卷挂载到了 `/data` 这个位置，替代了 `Dockerfile` 中定义的匿名卷的挂载配置：

```bash
$ docker run -d -v mydata:/data xxxx
```

### `EXPOSE` 暴露端口

`EXPOSE` 指令是声明容器运行时提供服务的端口，这只是一个声明，在容器运行时并不会因为这个声明应用就会开启这个端口的服务。格式为：

- `EXPOSE <端口1> [<端口2>...]`

在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用随机端口映射时，也就是 `docker run -P` 时，会自动随机映射 `EXPOSE` 的端口。

要将 `EXPOSE` 和在运行时使用 `-p <宿主端口>:<容器端口>` 区分开来。`-p`，是映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而 `EXPOSE` 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。

### `WORKDIR` 指定工作目录

使用 `WORKDIR` 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在，`WORKDIR` 会帮你建立目录。格式为：

- `WORKDIR <工作目录路径>`

其中，*工作目录路径* 是容器的文件系统目录。

之前提到一些初学者常犯的错误是把 `Dockerfile` 等同于 Shell 脚本来书写，这种错误的理解还可能会导致出现下面这样的错误：

```dockerfile
RUN cd /app
RUN echo "hello" > world.txt
```

如果将这个 `Dockerfile` 进行构建镜像运行后，会发现找不到 `/app/world.txt` 文件，或者其内容不是 `hello`。原因其实很简单，在 Shell 中，连续两行是同一个进程执行环境，因此前一个命令修改的内存状态，会直接影响后一个命令；**而在 `Dockerfile` 中，这两行 `RUN` 命令的执行环境根本不同，是两个完全不同的容器。**这就是对 `Dockerfile` 构建分层存储的概念不了解所导致的错误。

之前说过每一个 `RUN` 都是启动一个容器、执行命令、然后提交存储层文件变更。第一层 `RUN cd /app` 的执行仅仅是当前进程的工作目录变更，一个内存上的变化而已，其结果不会造成任何文件变更。而到第二层的时候，启动的是一个全新的容器，跟第一层的容器更完全没关系，自然不可能继承前一层构建过程中的内存变化。

因此如果需要改变以后各层的工作目录的位置，那么应该使用 `WORKDIR` 指令。

```dockerfile
WORKDIR /app

RUN echo "hello" > world.txt
```

如果你的 `WORKDIR` 指令使用的相对路径，那么所切换的路径与之前的 `WORKDIR` 有关：

```dockerfile
WORKDIR /a
WORKDIR b
WORKDIR c

RUN pwd
```

`RUN pwd` 的工作目录为 `/a/b/c`。

### `USER` 指定当前用户

`USER` 指令和 `WORKDIR` 相似，都是改变环境状态并影响以后的层。`WORKDIR` 是改变工作目录，`USER` 则是改变之后层的执行 `RUN`, `CMD` 以及 `ENTRYPOINT` 这类命令的身份。格式：

 - `USER <用户名>[:<用户组>]`

注意，`USER` 只是帮助你切换到指定用户而已，这个用户必须是事先建立好的，否则无法切换。

```dockerfile
RUN groupadd -r redis && useradd -r -g redis redis
USER redis
RUN [ "redis-server" ]
```

如果以 `root` 执行的脚本，在执行期间希望改变身份，比如希望以某个已经建立好的用户来运行某个服务进程，不要使用 `su` 或者 `sudo`，这些都需要比较麻烦的配置，而且在 TTY 缺失的环境下经常出错。建议使用 [`gosu`](https://github.com/tianon/gosu)。

```dockerfile
# 建立 redis 用户，并使用 gosu 换另一个用户执行命令
RUN groupadd -r redis && useradd -r -g redis redis
# 下载 gosu
RUN wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/1.12/gosu-amd64" \
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true
# 设置 CMD，并以另外的用户执行
CMD [ "exec", "gosu", "redis", "redis-server" ]
```

### `HEALTHCHECK` 健康检查

`HEALTHCHECK` 指令是告诉 Docker 应该如何进行判断容器的状态是否正常。其格式如下：

- `HEALTHCHECK [选项] CMD <命令>`：设置检查容器健康状况的命令
- `HEALTHCHECK NONE`：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

在没有 `HEALTHCHECK` 指令前，Docker 引擎只可以通过容器内主进程是否退出来判断容器是否状态异常。很多情况下这没问题，但是如果程序进入死锁状态，或者死循环状态，应用进程并不退出，但是该容器已经无法提供服务了。在 1.12 以前，Docker 不会检测到容器的这种状态，从而不会重新调度，导致可能会有部分容器已经无法提供服务了却还在接受用户请求。

而自 1.12 之后，Docker 提供了 `HEALTHCHECK` 指令，通过该指令指定一行命令，用这行命令来判断容器主进程的服务状态是否还正常，从而比较真实的反应容器实际状态。

当在一个镜像指定了 `HEALTHCHECK` 指令后，用其启动容器，初始状态会为 `starting`，在 `HEALTHCHECK` 指令检查成功后变为 `healthy`，如果连续一定次数失败，则会变为 `unhealthy`。

`HEALTHCHECK` 支持下列选项：

- `--interval=<间隔>`：两次健康检查的间隔，默认为 30 秒；
- `--timeout=<时长>`：健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，默认 30 秒；
- `--retries=<次数>`：当连续失败指定次数后，则将容器状态视为 `unhealthy`，默认 3 次。

和 `CMD`, `ENTRYPOINT` 一样，`HEALTHCHECK` 只可以出现一次，如果写了多个，只有最后一个生效。

在 `HEALTHCHECK [选项] CMD` 后面的命令，格式和 `ENTRYPOINT` 一样，分为 `shell` 格式和 `exec` 格式。命令的返回值决定了该次健康检查的成功与否：`0`：成功；`1`：失败；`2`：保留，不要使用这个值。

假设我们有个镜像是个最简单的 Web 服务，我们希望增加健康检查来判断其 Web 服务是否在正常工作，我们可以用 `curl` 来帮助判断，其 `Dockerfile` 的 `HEALTHCHECK` 可以这么写：

```bash
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=5s --timeout=3s \
  CMD curl -fs http://localhost/ || exit 1
```

这里我们设置了每 5 秒检查一次（这里为了试验所以间隔非常短，实际应该相对较长），如果健康检查命令超过 3 秒没响应就视为失败，并且使用 `curl -fs http://localhost/ || exit 1` 作为健康检查命令。

使用 `docker build` 来构建这个镜像：

```bash
$ docker build -t myweb:v1 .
```

构建好了后，我们启动一个容器：

```bash
$ docker run -d --name web -p 80:80 myweb:v1
```



### `ONBUILD` 作为基础镜像构建新镜像时执行

`ONBUILD` 是一个特殊的指令，它后面跟的是其它指令，比如 `RUN`, `COPY` 等，而这些指令，在当前镜像构建时并不会被执行。仅当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行。格式：

- `ONBUILD <其它指令>`

`Dockerfile` 中的其它指令都是为了定制当前镜像而准备的，唯有 `ONBUILD` 是为了帮助别人定制自己而准备的。

假设我们要制作 Node.js 所写的应用的镜像。我们都知道 Node.js 使用 `npm` 进行包管理，所有依赖、配置、启动信息等会放到 `package.json` 文件里。在拿到程序代码后，需要先进行 `npm install` 才可以获得所有需要的依赖。然后就可以通过 `npm start` 来启动应用。让我们用 `ONBUILD` 写一下基础镜像的 `Dockerfile`:

```dockerfile
FROM node:slim
RUN mkdir /app
WORKDIR /app
ONBUILD COPY ./package.json /app
ONBUILD RUN [ "npm", "install" ]
ONBUILD COPY . /app/
CMD [ "npm", "start" ]
```

将项目相关的指令加上 `ONBUILD`，这样在构建基础镜像的时候，这三行并不会被执行。只有在以此镜像作为基础镜像构建新镜像时，这三行才会被执行。

假设有其他类似的Node.js项目，各个Node.js项目的 `Dockerfile` 就变成了下面简单的一行：

```dockerfile
FROM my-node
```

当在各个项目目录中，用这个只有一行的 `Dockerfile` 构建镜像时，之前基础镜像的那三行 `ONBUILD` 就会开始执行，成功的将当前项目的代码复制进镜像、并且针对本项目执行 `npm install`，生成应用镜像。

### LABEL 为镜像添加元数据

`LABEL` 指令用来给镜像以键值对的形式添加一些元数据（metadata）：

```dockerfile
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

我们还可以用一些标签来申明镜像的作者、文档地址等：

```dockerfile
LABEL org.opencontainers.image.authors="yeasy"
LABEL org.opencontainers.image.documentation="https://yeasy.gitbooks.io"
```

具体可以参考 https://github.com/opencontainers/image-spec/blob/master/annotations.md

### SHELL 指令

格式：`SHELL ["executable", "parameters"]`

`SHELL` 指令可以指定 `RUN` `ENTRYPOINT` `CMD` 指令的 shell，Linux 中默认为`["/bin/sh", "-c"]`

```dockerfile
SHELL ["/bin/sh", "-c"]

RUN lll ; ls

SHELL ["/bin/sh", "-cex"]

RUN lll ; ls
```

两个 `RUN` 运行同一命令，第二个 `RUN` 运行的命令会打印出每条命令并当遇到错误时退出。

当 `ENTRYPOINT` `CMD` 以 shell 格式指定时，`SHELL` 指令所指定的 shell 也会成为这两个指令的 shell：

```dockerfile
SHELL ["/bin/sh", "-cex"]

# /bin/sh -cex "nginx"
ENTRYPOINT nginx
```

```dockerfile
SHELL ["/bin/sh", "-cex"]

# /bin/sh -cex "nginx"
CMD nginx
```

### 参考文档

- `Dockerfie` 官方文档：https://docs.docker.com/engine/reference/builder/
- `Dockerfile` 最佳实践文档：https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
- `Docker` 官方镜像 `Dockerfile`：https://github.com/docker-library/docs

## Dockerfile 多阶段构建

Docker v17.05 开始支持多阶段构建 (`multistage builds`)。使用多阶段构建我们就可以很容易解决前面提到的问题，并且只需要编写一个 `Dockerfile`：

例如，编写 `Dockerfile` 文件：

```dockerfile
FROM golang:alpine as builder

RUN apk --no-cache add git

WORKDIR /go/src/github.com/go/helloworld/

RUN go get -d -v github.com/go-sql-driver/mysql

COPY app.go .

RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest as prod

RUN apk --no-cache add ca-certificates

WORKDIR /root/

COPY --from=0 /go/src/github.com/go/helloworld/app .

CMD ["./app"]
```

构建镜像：

```bash
$ docker build -t go/helloworld:3 .
```

#### 只构建某一阶段的镜像

我们可以使用 `as` 来为某一阶段命名，例如

```dockerfile
FROM golang:alpine as builder
```

例如当我们只想构建 `builder` 阶段的镜像时，增加 `--target=builder` 参数即可：

```bash
$ docker build --target builder -t username/imagename:tag .
```

#### 构建时从其他镜像复制文件

上面例子中我们使用 `COPY --from=0 /go/src/github.com/go/helloworld/app .` 从上一阶段的镜像中复制文件，我们也可以复制任意镜像中的文件。

```dockerfile
$ COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
```

#### 实战多阶段构建 Laravel 镜像

##### 准备

新建一个 `Laravel` 项目或在已有的 `Laravel` 项目根目录下新建 `Dockerfile` `.dockerignore` `laravel.conf` 文件。

在 `.dockerignore` 文件中写入以下内容。

```bash
.idea/
.git/
vendor/
node_modules/
public/js/
public/css/
public/mix-manifest.json
yarn-error.log
bootstrap/cache/*
storage/
# 自行添加其他需要排除的文件，例如 .env.* 文件
```

在 `laravel.conf` 文件中写入 nginx 配置。

```nginx
server {
  listen 80 default_server;
  root /app/laravel/public;
  index index.php index.html;
  location / {
      try_files $uri $uri/ /index.php?$query_string;
  }
  location ~ .*\.php(\/.*)*$ {
    fastcgi_pass   laravel:9000;
    include        fastcgi.conf;
    # fastcgi_connect_timeout 300;
    # fastcgi_send_timeout 300;
    # fastcgi_read_timeout 300;
  }
}
```

##### 前端构建

第一阶段进行前端构建。

```dockerfile
FROM node:alpine as frontend
COPY package.json /app/
RUN set -x ; cd /app \
      && npm install --registry=https://registry.npmmirror.com
COPY webpack.mix.js webpack.config.js tailwind.config.js /app/
COPY resources/ /app/resources/
RUN set -x ; cd /app \
      && touch artisan \
      && mkdir -p public \
      && npm run production
```

##### 安装 Composer 依赖

第二阶段安装 Composer 依赖。

```dockerfile
FROM composer as composer
COPY database/ /app/database/
COPY composer.json composer.lock /app/
RUN set -x ; cd /app \
      && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/ \
      && composer install \
           --ignore-platform-reqs \
           --no-interaction \
           --no-plugins \
           --no-scripts \
           --prefer-dist
```

##### 整合以上阶段所生成的文件

第三阶段对以上阶段生成的文件进行整合。

```dockerfile
FROM php:7.4-fpm-alpine as laravel
ARG LARAVEL_PATH=/app/laravel
COPY --from=composer /app/vendor/ ${LARAVEL_PATH}/vendor/
COPY . ${LARAVEL_PATH}
COPY --from=frontend /app/public/js/ ${LARAVEL_PATH}/public/js/
COPY --from=frontend /app/public/css/ ${LARAVEL_PATH}/public/css/
COPY --from=frontend /app/public/mix-manifest.json ${LARAVEL_PATH}/public/mix-manifest.json
RUN set -x ; cd ${LARAVEL_PATH} \
      && mkdir -p storage \
      && mkdir -p storage/framework/cache \
      && mkdir -p storage/framework/sessions \
      && mkdir -p storage/framework/testing \
      && mkdir -p storage/framework/views \
      && mkdir -p storage/logs \
      && chmod -R 777 storage \
      && php artisan package:discover
```

##### 最后一个阶段构建 NGINX 镜像

```dockerfile
FROM nginx:alpine as nginx
ARG LARAVEL_PATH=/app/laravel
COPY laravel.conf /etc/nginx/conf.d/
COPY --from=laravel ${LARAVEL_PATH}/public ${LARAVEL_PATH}/public
```

##### 构建 Laravel 及 Nginx 镜像

使用 `docker build` 命令构建镜像。

```bash
$ docker build -t my/laravel --target=laravel .
$ docker build -t my/nginx --target=nginx .
```

##### 启动容器并测试

新建 Docker 网络

```bash
$ docker network create laravel
```

启动 laravel 容器， `--name=laravel` 参数设定的名字必须与 `nginx` 配置文件中的 `fastcgi_pass laravel:9000;` 一致：

```bash
$ docker run -dit --rm --name=laravel --network=laravel my/laravel
```

启动 nginx 容器

```bash
$ docker run -dit --rm --network=laravel -p 8080:80 my/nginx
```

浏览器访问 `127.0.0.1:8080` 可以看到 Laravel 项目首页。

> 也许 Laravel 项目依赖其他外部服务，例如 redis、MySQL，请自行启动这些服务之后再进行测试，本小节不再赘述。

##### 生产环境优化

本小节内容为了方便测试，将配置文件直接放到了镜像中，实际在使用时 **建议** 将配置文件作为 `config` 或 `secret` 挂载到容器中，请读者自行学习 `Swarm mode` 或 `Kubernetes` 的相关内容。

由于篇幅所限本小节只是简单列出，更多内容可以参考 https://github.com/khs1994-docker/laravel-demo 项目。

##### 附录

完整的 `Dockerfile` 文件如下。

```dockerfile
FROM node:alpine as frontend
COPY package.json /app/
RUN set -x ; cd /app \
      && npm install --registry=https://registry.npmmirror.com
COPY webpack.mix.js webpack.config.js tailwind.config.js /app/
COPY resources/ /app/resources/
RUN set -x ; cd /app \
      && touch artisan \
      && mkdir -p public \
      && npm run production
FROM composer as composer
COPY database/ /app/database/
COPY composer.json /app/
RUN set -x ; cd /app \
      && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/ \
      && composer install \
           --ignore-platform-reqs \
           --no-interaction \
           --no-plugins \
           --no-scripts \
           --prefer-dist
FROM php:7.4-fpm-alpine as laravel
ARG LARAVEL_PATH=/app/laravel
COPY --from=composer /app/vendor/ ${LARAVEL_PATH}/vendor/
COPY . ${LARAVEL_PATH}
COPY --from=frontend /app/public/js/ ${LARAVEL_PATH}/public/js/
COPY --from=frontend /app/public/css/ ${LARAVEL_PATH}/public/css/
COPY --from=frontend /app/public/mix-manifest.json ${LARAVEL_PATH}/public/mix-manifest.json
RUN set -x ; cd ${LARAVEL_PATH} \
      && mkdir -p storage \
      && mkdir -p storage/framework/cache \
      && mkdir -p storage/framework/sessions \
      && mkdir -p storage/framework/testing \
      && mkdir -p storage/framework/views \
      && mkdir -p storage/logs \
      && chmod -R 777 storage \
      && php artisan package:discover
FROM nginx:alpine as nginx
ARG LARAVEL_PATH=/app/laravel
COPY laravel.conf /etc/nginx/conf.d/
COPY --from=laravel ${LARAVEL_PATH}/public ${LARAVEL_PATH}/public
```

## 构建多种系统架构支持的 Docker 镜像 -- docker manifest 命令详解

我们知道使用镜像创建一个容器，该镜像必须与 Docker 宿主机系统架构一致，例如 `Linux x86_64` 架构的系统中只能使用 `Linux x86_64` 的镜像创建容器。

> Windows、macOS 除外，其使用了 [binfmt_misc](https://docs.docker.com/docker-for-mac/multi-arch/) 提供了多种架构支持，在 Windows、macOS 系统上 (x86_64) 可以运行 arm 等其他架构的镜像。

例如我们在 `Linux x86_64` 中构建一个 `username/test` 镜像。

```docker
FROM alpine
CMD echo 1
```

构建镜像后推送到 Docker Hub，之后我们尝试在树莓派 `Linux arm64v8` 中使用这个镜像。

```bash
$ docker run -it --rm username/test
```

可以发现这个镜像根本获取不到。

要解决这个问题，通常采用的做法是通过镜像名区分不同系统架构的镜像，例如在 `Linux x86_64` 和 `Linux arm64v8` 分别构建 `username/test` 和 `username/arm64v8-test` 镜像。运行时使用对应架构的镜像即可。

这样做显得很繁琐，那么有没有一种方法让 Docker 引擎根据系统架构自动拉取对应的镜像呢？

我们发现在 `Linux x86_64` 和 `Linux arm64v8` 架构的计算机中分别使用 `golang:alpine` 镜像运行容器 `$ docker run golang:alpine go version` 时，容器能够正常的运行。

这是什么原因呢？

原因就是 `golang:alpine` 官方镜像有一个 [`manifest` 列表 (`manifest list`)](https://docs.docker.com/registry/spec/manifest-v2-2/)。

当用户获取一个镜像时，Docker 引擎会首先查找该镜像是否有 `manifest` 列表，如果有的话 Docker 引擎会按照 Docker 运行环境（系统及架构）查找出对应镜像（例如 `golang:alpine`）。如果没有的话会直接获取镜像（例如上例中我们构建的 `username/test`）。

我们可以使用 `$ docker manifest inspect golang:alpine` 查看这个 `manifest` 列表的结构。

```bash
$ docker manifest inspect golang:alpine
```

```json
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.list.v2+json",
   "manifests": [
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1365,
         "digest": "sha256:5e28ac423243b187f464d635bcfe1e909f4a31c6c8bce51d0db0a1062bec9e16",
         "platform": {
            "architecture": "amd64",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1365,
         "digest": "sha256:2945c46e26c9787da884b4065d1de64cf93a3b81ead1b949843dda1fcd458bae",
         "platform": {
            "architecture": "arm",
            "os": "linux",
            "variant": "v7"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1365,
         "digest": "sha256:87fff60114fd3402d0c1a7ddf1eea1ded658f171749b57dc782fd33ee2d47b2d",
         "platform": {
            "architecture": "arm64",
            "os": "linux",
            "variant": "v8"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1365,
         "digest": "sha256:607b43f1d91144f82a9433764e85eb3ccf83f73569552a49bc9788c31b4338de",
         "platform": {
            "architecture": "386",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1365,
         "digest": "sha256:25ead0e21ed5e246ce31e274b98c09aaf548606788ef28eaf375dc8525064314",
         "platform": {
            "architecture": "ppc64le",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1365,
         "digest": "sha256:69f5907fa93ea591175b2c688673775378ed861eeb687776669a48692bb9754d",
         "platform": {
            "architecture": "s390x",
            "os": "linux"
         }
      }
   ]
}
```

可以看出 `manifest` 列表中包含了不同系统架构所对应的镜像 `digest` 值，这样 Docker 就可以在不同的架构中使用相同的 `manifest` (例如 `golang:alpine`) 获取对应的镜像。

下面介绍如何使用 `$ docker manifest` 命令创建并推送 `manifest` 列表到 Docker Hub。

### 构建镜像

首先在 `Linux x86_64` 构建 `username/x8664-test` 镜像。并在 `Linux arm64v8` 中构建 `username/arm64v8-test` 镜像，构建好之后推送到 Docker Hub。

### 创建 `manifest` 列表

```bash
# $ docker manifest create MANIFEST_LIST MANIFEST [MANIFEST...]
$ docker manifest create username/test \
      username/x8664-test \
      username/arm64v8-test
```

当要修改一个 `manifest` 列表时，可以加入 `-a` 或 `--amend` 参数。

### 设置 `manifest` 列表

```bash
# $ docker manifest annotate [OPTIONS] MANIFEST_LIST MANIFEST
$ docker manifest annotate username/test \
      username/x8664-test \
      --os linux --arch x86_64
$ docker manifest annotate username/test \
      username/arm64v8-test \
      --os linux --arch arm64 --variant v8
```

这样就配置好了 `manifest` 列表。

### 查看 `manifest` 列表

```bash
$ docker manifest inspect username/test
```

### 推送 `manifest` 列表

最后我们可以将其推送到 Docker Hub。

```bash
$ docker manifest push username/test
```

### 测试

我们在 `Linux x86_64` `Linux arm64v8` 中分别执行 `$ docker run -it --rm username/test` 命令，发现可以正确的执行。

### 官方博客

详细了解 `manifest` 可以阅读官方博客。

* https://www.docker.com/blog/multi-arch-all-the-things/


## 其它制作镜像的方式

除了标准的使用 `Dockerfile` 生成镜像的方法外，由于各种特殊需求和历史原因，还提供了一些其它方法用以生成镜像。

### 从 rootfs 压缩包导入

格式：`docker import [选项] <文件>|<URL>|- [<仓库名>[:<标签>]]`

压缩包可以是本地文件、远程 Web 文件，甚至是从标准输入中得到。压缩包将会在镜像 `/` 目录展开，并直接作为镜像第一层提交。

比如我们想要创建一个 [OpenVZ](https://openvz.org) 的 Ubuntu 16.04 [模板](https://wiki.openvz.org/Download/template/precreated)的镜像：

```bash
$ docker import \
    http://download.openvz.org/template/precreated/ubuntu-16.04-x86_64.tar.gz \
    openvz/ubuntu:16.04
Downloading from http://download.openvz.org/template/precreated/ubuntu-16.04-x86_64.tar.gz
sha256:412b8fc3e3f786dca0197834a698932b9c51b69bd8cf49e100c35d38c9879213
```

这条命令自动下载了 `ubuntu-16.04-x86_64.tar.gz` 文件，并且作为根文件系统展开导入，并保存为镜像 `openvz/ubuntu:16.04`。

导入成功后，我们可以用 `docker image ls` 看到这个导入的镜像：

```bash
$ docker image ls openvz/ubuntu
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
openvz/ubuntu       16.04               412b8fc3e3f7        55 seconds ago      505MB
```

如果我们查看其历史的话，会看到描述中有导入的文件链接：

```bash
$ docker history openvz/ubuntu:16.04
IMAGE               CREATED              CREATED BY          SIZE                COMMENT
f477a6e18e98        About a minute ago                       214.9 MB            Imported from http://download.openvz.org/template/precreated/ubuntu-16.04-x86_64.tar.gz
```

### Docker 镜像的导入和导出 `docker save` 和 `docker load`

Docker 还提供了 `docker save` 和 `docker load` 命令，用以将镜像保存为一个文件，然后传输到另一个位置上，再加载进来。这是在没有 Docker Registry 时的做法，现在已经不推荐，镜像迁移应该直接使用 Docker Registry，无论是直接使用 Docker Hub 还是使用内网私有 Registry 都可以。

#### 保存镜像

使用 `docker save` 命令可以将镜像保存为归档文件。

比如我们希望保存这个 `alpine` 镜像。

```bash
$ docker image ls alpine
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
alpine              latest              baa5d63471ea        5 weeks ago         4.803 MB
```

保存镜像的命令为：

```bash
$ docker save alpine -o filename
$ file filename
filename: POSIX tar archive
```

这里的 filename 可以为任意名称甚至任意后缀名，但文件的本质都是归档文件

**注意：如果同名则会覆盖（没有警告）**

若使用 `gzip` 压缩：

```bash
$ docker save alpine | gzip > alpine-latest.tar.gz
```

然后我们将 `alpine-latest.tar.gz` 文件复制到了到了另一个机器上，可以用下面这个命令加载镜像：

```bash
$ docker load -i alpine-latest.tar.gz
Loaded image: alpine:latest
```

如果我们结合这两个命令以及 `ssh` 甚至 `pv` 的话，利用 Linux 强大的管道，我们可以写一个命令完成从一个机器将镜像迁移到另一个机器，并且带进度条的功能：

```bash
docker save <镜像名> | bzip2 | pv | ssh <用户名>@<主机名> 'cat | docker load'
```

## Docker镜像实现原理

Docker 镜像是怎么实现增量的修改和维护的？

每个镜像都由很多层次构成，Docker 使用 [Union FS](https://en.wikipedia.org/wiki/UnionFS) 将这些不同的层结合到一个镜像中去。

通常 Union FS 有两个用途, 一方面可以实现不借助 LVM、RAID 将多个 disk 挂到同一个目录下,另一个更常用的就是将一个只读的分支和一个可写的分支联合在一起，Live CD 正是基于此方法可以允许在镜像不变的基础上允许用户在其上进行一些写操作。

Docker 在 OverlayFS 上构建的容器也是利用了类似的原理。

# Docker 容器管理

容器是 Docker 又一核心概念。

简单的说，容器是独立运行的一个或一组应用，以及它们的运行态环境。对应的，虚拟机可以理解为模拟运行的一整套操作系统（提供了运行态环境和其他系统环境）和跑在上面的应用。

本章将具体介绍如何来管理一个容器，包括创建、启动和停止等。

## 启动容器

启动容器有2种方式：

- 基于镜像新建一个容器并启动；
- 将在终止状态（`exited`）的容器重新启动。

因为 Docker 的容器实在太轻量级了，很多时候用户都是随时删除和新创建容器。

### 新建容器并启动

当利用 `docker run` 来创建容器时，Docker 在后台运行的标准操作包括：

* 检查本地是否存在指定的镜像，不存在就从 [registry](../repository/README.md) 下载
* 利用镜像创建并启动一个容器
* 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
* 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
* 从地址池配置一个 ip 地址给容器
* 执行用户指定的应用程序
* 执行完毕后容器被终止

例如，下面的命令输出一个 “Hello World”，之后终止容器。

```bash
$ docker run ubuntu:18.04 /bin/echo 'Hello world'
Hello world
```

这跟在本地直接执行 `/bin/echo 'hello world'` 几乎感觉不出任何区别。

下面的命令则启动一个 bash 终端，允许用户进行交互。

```bash
$ docker run -t -i ubuntu:18.04 /bin/bash
root@af8bae53bdd3:/#
```

其中，`-t` 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， `-i` 则让容器的标准输入保持打开。

在交互模式下，用户可以通过所创建的终端来输入命令，例如

```bash
root@af8bae53bdd3:/# pwd
/
root@af8bae53bdd3:/# ls
bin boot dev etc home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var
```

### 启动已终止容器

可以利用 `docker container start` 命令，直接将一个已经终止（`exited`）的容器启动运行。

容器的核心为所执行的应用程序，所需要的资源都是应用程序运行所必需的。除此之外，并没有其它的资源。可以在伪终端中利用 `ps` 或 `top` 来查看进程信息。

```bash
root@ba267838cc1b:/# ps
  PID TTY          TIME CMD
    1 ?        00:00:00 bash
   11 ?        00:00:00 ps
```

可见，容器中仅运行了指定的 bash 应用。这种特点使得 Docker 对资源的利用率极高，是货真价实的轻量级虚拟化。

## 后台运行

更多的时候，需要让 Docker 在后台运行而不是直接把执行命令的结果输出在当前宿主机下。此时，可以通过添加 `-d` 参数来实现。

如果不使用 `-d` 参数运行容器。

```bash
$ docker run ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
hello world
hello world
hello world
hello world
```

容器会把输出的结果 (STDOUT) 打印到宿主机上面。

如果使用了 `-d` 参数运行容器。

```bash
$ docker run -d ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
77b2dc01fe0f3f1265df143181e7b9af5e05279a884f4776ee75350ea9d8017a
```

此时容器会在后台运行并不会把输出的结果 (STDOUT) 打印到宿主机上面(输出结果可以用 `docker logs` 查看)。

> **注：** 容器是否会长久运行，是和 `docker run` 指定的命令有关，和 `-d` 参数无关。

使用 `-d` 参数启动后会返回一个唯一的 id，也可以通过 `docker ps`命令或`docker container ls` 命令来查看容器信息。

```bash
$ docker container ls
CONTAINER ID  IMAGE         COMMAND               CREATED        STATUS       PORTS NAMES
77b2dc01fe0f  ubuntu:18.04  /bin/sh -c 'while tr  2 minutes ago  Up 1 minute        agitated_wright
```

可以通过`docker logs`命令或 `docker container logs` 命令获取容器的输出信息：

```bash
$ docker container logs [container ID or NAMES]
hello world
hello world
hello world
. . .
```

## 终止容器

可以使用 `docker stop`命令或`docker container stop` 来终止一个运行中的容器。

此外，当 Docker 容器中指定的应用终结时，容器也自动终止。

例如对于上一章节中只启动了一个终端的容器，用户通过 `exit` 命令或 `Ctrl+d` 来退出终端时，所创建的容器立刻终止。

终止状态的容器可以用 `docker container ls -a` 命令看到。例如：

```bash
$ docker container ls -a
CONTAINER ID        IMAGE                    COMMAND                CREATED             STATUS                          PORTS               NAMES
ba267838cc1b        ubuntu:18.04             "/bin/bash"            30 minutes ago      Exited (0) About a minute ago                       trusting_newton
```

处于终止状态的容器，可以通过 `docker container start` 命令来重新启动。

此外，`docker container restart` 命令会将一个运行态的容器终止，然后再重新启动它。

## 进入容器

在使用 `-d` 参数时，容器启动后会进入后台。

某些时候需要进入容器进行操作，包括使用 `docker attach` 命令或 `docker exec` 命令，推荐大家使用 `docker exec` 命令，原因会在下面说明。

### `attach` 命令

下面示例如何使用 `docker attach` 命令。

```bash
$ docker run -dit ubuntu
243c32535da7d142fb0e6df616a3c3ada0b8ab417937c853a9e1c251f499f550
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
243c32535da7        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           nostalgic_hypatia
$ docker attach 243c
root@243c32535da7:/#
```

*注意：* 如果从这个 stdin 中 exit，会导致容器的停止。

### `exec` 命令

#### `-i` `-t` 参数

`docker exec` 后边可以跟多个参数，这里主要说明 `-i` `-t` 参数。

只用 `-i` 参数时，由于没有分配伪终端，界面没有我们熟悉的 Linux 命令提示符，但命令执行结果仍然可以返回。

当 `-i` `-t` 参数一起使用时，则可以看到我们熟悉的 Linux 命令提示符。

```bash
$ docker run -dit ubuntu
69d137adef7a8a689cbcb059e94da5489d3cddd240ff675c640c8d96e84fe1f6
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
69d137adef7a        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           zealous_swirles
$ docker exec -i 69d1 bash
ls
bin
boot
dev
...
$ docker exec -it 69d1 bash
root@69d137adef7a:/#
```

如果从这个 stdin 中 exit，不会导致容器的停止。这就是为什么推荐大家使用 `docker exec` 的原因。

更多参数说明请使用 `docker exec --help` 查看。

## 导出和导入容器

### 导出容器

如果要导出本地某个容器，可以使用 `docker export` 命令。
```bash
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                    PORTS               NAMES
7691a814370e        ubuntu:18.04        "/bin/bash"         36 hours ago        Exited (0) 21 hours ago                       test
$ docker export 7691a814370e > ubuntu.tar
```

这样将导出容器快照到本地文件。

### 导入容器快照

可以使用 `docker import` 从容器快照文件中再导入为镜像，例如

```bash
$ cat ubuntu.tar | docker import - test/ubuntu:v1.0
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
test/ubuntu         v1.0                9d37a6082e97        About a minute ago   171.3 MB
```

此外，也可以通过指定 URL 或者某个目录来导入，例如

```bash
$ docker import http://example.com/exampleimage.tgz example/imagerepo
```

*注：用户既可以使用 `docker load` 来导入镜像存储文件到本地镜像库，也可以使用 `docker import` 来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。*

## 删除容器

可以使用 `docker rm`命令或`docker container rm`命令 来删除一个处于终止状态的容器。例如

```bash
$ docker container rm trusting_newton
trusting_newton
```

如果要删除一个运行中的容器，可以添加 `-f` 参数。Docker 会发送 `SIGKILL` 信号给容器。

### 清理所有处于终止状态的容器

使用 `docker ps -a`命令或`docker container ls -a` 命令可以查看所有已经创建的包括终止状态的容器，如果数量太多要一个个删除可能会很麻烦，用下面的命令可以清理掉所有处于终止状态的容器。

```bash
$ docker container prune
```

# Docker 仓库

仓库（`Repository`）是集中存放镜像的地方。

一个容易混淆的概念是注册服务器（`Registry`）。实际上注册服务器是管理仓库的具体服务器，每个服务器上可以有多个仓库，而每个仓库下面有多个镜像。从这方面来说，仓库可以被认为是一个具体的项目或目录。例如对于仓库地址 `docker.io/ubuntu` 来说，`docker.io` 是注册服务器地址，`ubuntu` 是仓库名。

大部分时候，并不需要严格区分这两者的概念。

## Docker Hub

目前 Docker 官方维护了一个公共仓库 [Docker Hub](https://hub.docker.com/)。大部分需求都可以通过在 Docker Hub 中直接下载镜像来实现。

### 注册

你可以在 https://hub.docker.com 免费注册一个 Docker 账号。

### 登录

可以通过执行 `docker login` 命令交互式的输入用户名及密码来完成在命令行界面登录 Docker Hub。

你可以通过 `docker logout` 退出登录。

### 拉取镜像

你可以通过 `docker search` 命令来查找官方仓库中的镜像，并利用 `docker pull` 命令来将它下载到本地。

例如以 `centos` 为关键词进行搜索：

```bash
$ docker search centos
NAME                               DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
centos                             The official build of CentOS.                   6449      [OK]
ansible/centos7-ansible            Ansible on Centos7                              132                  [OK]
consol/centos-xfce-vnc             Centos container with "headless" VNC session…   126                  [OK]
jdeathe/centos-ssh                 OpenSSH / Supervisor / EPEL/IUS/SCL Repos - …   117                  [OK]
centos/systemd                     systemd enabled base container.                 96                   [OK]
```

可以看到返回了很多包含关键字的镜像，其中包括镜像名字、描述、收藏数（表示该镜像的受关注程度）、是否官方创建（`OFFICIAL`）、是否自动构建 （`AUTOMATED`）。

根据是否是官方提供，可将镜像分为2类：

- 一种是类似 `centos` 这样的镜像，被称为基础镜像或根镜像。这些基础镜像由 Docker 公司创建、验证、支持、提供。这样的镜像往往使用单个单词作为名字。

- 一种是类似 `ansible/centos7-ansible` 镜像，它是由 Docker Hub 的注册用户创建并维护的，往往带有用户名称前缀。可以通过前缀 `username/` 来指定使用某个用户提供的镜像，比如 ansible 用户。

另外，在查找的时候通过 `--filter=stars=N` 参数可以指定仅显示收藏数量为 `N` 以上的镜像。

下载官方 `centos` 镜像到本地：

```bash
$ docker pull centos
Using default tag: latest
latest: Pulling from library/centos
7a0437f04f83: Pull complete
Digest: sha256:5528e8b1b1719d34604c87e11dcd1c0a20bedf46e83b5632cdeac91b8c04efc1
Status: Downloaded newer image for centos:latest
docker.io/library/centos:latest
```

### 推送镜像

用户也可以在登录后通过 `docker push` 命令来将自己的镜像推送到 Docker Hub。

以下命令中的 `username` 请替换为你的 Docker 账号用户名。

```bash
$ docker tag ubuntu:18.04 username/ubuntu:18.04
$ docker image ls
REPOSITORY                                               TAG                    IMAGE ID            CREATED             SIZE
ubuntu                                                   18.04                  275d79972a86        6 days ago          94.6MB
username/ubuntu                                          18.04                  275d79972a86        6 days ago          94.6MB
$ docker push username/ubuntu:18.04
$ docker search username
NAME                      DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
username/ubuntu
```

### 自动构建

> 2021 年 7 月 26 日之后，该项功能仅限[付费用户](https://www.docker.com/blog/changes-to-docker-hub-autobuilds/)使用。

自动构建（`Automated Builds`）功能对于需要经常升级镜像内程序来说，十分方便。

有时候，用户构建了镜像，安装了某个软件，当软件发布新版本则需要手动更新镜像。

而自动构建允许用户通过 Docker Hub 指定跟踪一个目标网站（支持 [GitHub](https://github.com) 或 [BitBucket](https://bitbucket.org)）上的项目，一旦项目发生新的提交 （`commit`）或者创建了新的标签（`tag`），Docker Hub 会自动构建镜像并推送到 Docker Hub 中。

要配置自动构建，包括如下的步骤：

* 登录 Docker Hub；

* 在 Docker Hub 点击右上角头像，在账号设置（`Account Settings`）中关联（`Linked Accounts`）目标网站；

* 在 Docker Hub 中新建或选择已有的仓库，在 `Builds` 选项卡中选择 `Configure Automated Builds`；

* 选取一个目标网站中的项目（需要含 `Dockerfile`）和分支；

* 指定 `Dockerfile` 的位置，并保存。

之后，可以在 Docker Hub 的仓库页面的 `Timeline` 选项卡中查看每次构建的状态。

## 私有仓库

有时候使用 Docker Hub 这样的公共仓库可能不方便，用户可以创建一个本地仓库供私人使用。

[`docker-registry`](https://docs.docker.com/registry/) 是官方提供的工具，可以用于构建私有的镜像仓库。本文内容基于 [`docker-registry`](https://github.com/docker/distribution) v2.x 版本。

### 安装运行 docker-registry

#### 容器运行

你可以使用官方 `registry` 镜像来运行。

```bash
$ docker run -d -p 5000:5000 --restart=always --name registry registry
```

这将使用官方的 `registry` 镜像来启动私有仓库。默认情况下，仓库会被创建在容器的 `/var/lib/registry` 目录下。你可以通过 `-v` 参数来将镜像文件存放在本地的指定路径。例如下面的例子将上传的镜像放到本地的 `/opt/data/registry` 目录。

```bash
$ docker run -d \
    -p 5000:5000 \
    -v /opt/data/registry:/var/lib/registry \
    registry
```

### 在私有仓库上传、搜索、下载镜像

创建好私有仓库之后，就可以使用 `docker tag` 来标记一个镜像，然后推送它到仓库。例如私有仓库地址为 `127.0.0.1:5000`。

先在本机查看已有的镜像。

```bash
$ docker image ls
REPOSITORY                        TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu                            latest              ba5877dc9bec        6 weeks ago         192.7 MB
```

使用 `docker tag` 将 `ubuntu:latest` 这个镜像标记为 `127.0.0.1:5000/ubuntu:latest`。

格式为 `docker tag IMAGE[:TAG] [REGISTRY_HOST[:REGISTRY_PORT]/]REPOSITORY[:TAG]`。

```bash
$ docker tag ubuntu:latest 127.0.0.1:5000/ubuntu:latest
$ docker image ls
REPOSITORY                        TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu                            latest              ba5877dc9bec        6 weeks ago         192.7 MB
127.0.0.1:5000/ubuntu:latest      latest              ba5877dc9bec        6 weeks ago         192.7 MB
```

使用 `docker push` 上传标记的镜像。

```bash
$ docker push 127.0.0.1:5000/ubuntu:latest
The push refers to repository [127.0.0.1:5000/ubuntu]
373a30c24545: Pushed
a9148f5200b0: Pushed
cdd3de0940ab: Pushed
fc56279bbb33: Pushed
b38367233d37: Pushed
2aebd096e0e2: Pushed
latest: digest: sha256:fe4277621f10b5026266932ddf760f5a756d2facd505a94d2da12f4f52f71f5a size: 1568
```

用 `curl` 查看仓库中的镜像。

```bash
$ curl 127.0.0.1:5000/v2/_catalog
{"repositories":["ubuntu"]}
```

这里可以看到 `{"repositories":["ubuntu"]}`，表明镜像已经被成功上传了。

先删除已有镜像，再尝试从私有仓库中下载这个镜像。

```bash
$ docker image rm 127.0.0.1:5000/ubuntu:latest
$ docker pull 127.0.0.1:5000/ubuntu:latest
Pulling repository 127.0.0.1:5000/ubuntu:latest
ba5877dc9bec: Download complete
511136ea3c5a: Download complete
9bad880da3d2: Download complete
25f11f5fb0cb: Download complete
ebc34468f71d: Download complete
2318d26665ef: Download complete
$ docker image ls
REPOSITORY                         TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
127.0.0.1:5000/ubuntu:latest       latest              ba5877dc9bec        6 weeks ago         192.7 MB
```

### 配置非 https 仓库地址

如果你不想使用 `127.0.0.1:5000` 作为仓库地址，比如想让本网段的其他主机也能把镜像推送到私有仓库。你就得把例如 `192.168.199.100:5000` 这样的内网地址作为私有仓库地址，这时你会发现无法成功推送镜像。

这是因为 Docker 默认不允许非 `HTTPS` 方式推送镜像。我们可以通过 Docker 的配置选项来取消这个限制，或者查看下一节配置能够通过 `HTTPS` 访问的私有仓库。

### Ubuntu 16.04+, Debian 8+, centos 7

对于使用 `systemd` 的系统，请在 `/etc/docker/daemon.json` 中写入如下内容（如果文件不存在请新建该文件）

```json
{
  "registry-mirror": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ],
  "insecure-registries": [
    "192.168.199.100:5000"
  ]
}
```

>注意：该文件必须符合 `json` 规范，否则 Docker 将不能启动。

### 其他

对于 Docker Desktop for Windows 、 Docker Desktop for Mac 在设置中的 `Docker Engine` 中进行编辑 ，增加和上边一样的字符串即可。

## 私有仓库高级配置

上一节我们搭建了一个具有基础功能的私有仓库，本小节我们来使用 `Docker Compose` 搭建一个拥有权限认证、TLS 的私有仓库。

新建一个文件夹，以下步骤均在该文件夹中进行。

### 准备站点证书

如果你拥有一个域名，国内各大云服务商均提供免费的站点证书。你也可以使用 `openssl` 自行签发证书。

这里假设我们将要搭建的私有仓库地址为 `docker.domain.com`，下面我们介绍使用 `openssl` 自行签发 `docker.domain.com` 的站点 SSL 证书。

第一步创建 `CA` 私钥。

```bash
$ openssl genrsa -out "root-ca.key" 4096
```

第二步利用私钥创建 `CA` 根证书请求文件。

```bash
$ openssl req \
          -new -key "root-ca.key" \
          -out "root-ca.csr" -sha256 \
          -subj '/C=CN/ST=Shanxi/L=Datong/O=Your Company Name/CN=Your Company Name Docker Registry CA'
```

>以上命令中 `-subj` 参数里的 `/C` 表示国家，如 `CN`；`/ST` 表示省；`/L` 表示城市或者地区；`/O` 表示组织名；`/CN` 通用名称。

第三步配置 `CA` 根证书，新建 `root-ca.cnf`。

```bash
[root_ca]
basicConstraints = critical,CA:TRUE,pathlen:1
keyUsage = critical, nonRepudiation, cRLSign, keyCertSign
subjectKeyIdentifier=hash
```

第四步签发根证书。

```bash
$ openssl x509 -req  -days 3650  -in "root-ca.csr" \
               -signkey "root-ca.key" -sha256 -out "root-ca.crt" \
               -extfile "root-ca.cnf" -extensions \
               root_ca
```

第五步生成站点 `SSL` 私钥。

```bash
$ openssl genrsa -out "docker.domain.com.key" 4096
```

第六步使用私钥生成证书请求文件。

```bash
$ openssl req -new -key "docker.domain.com.key" -out "site.csr" -sha256 \
          -subj '/C=CN/ST=Shanxi/L=Datong/O=Your Company Name/CN=docker.domain.com'
```

第七步配置证书，新建 `site.cnf` 文件。

```bash
[server]
authorityKeyIdentifier=keyid,issuer
basicConstraints = critical,CA:FALSE
extendedKeyUsage=serverAuth
keyUsage = critical, digitalSignature, keyEncipherment
subjectAltName = DNS:docker.domain.com, IP:127.0.0.1
subjectKeyIdentifier=hash
```

第八步签署站点 `SSL` 证书。

```bash
$ openssl x509 -req -days 750 -in "site.csr" -sha256 \
    -CA "root-ca.crt" -CAkey "root-ca.key"  -CAcreateserial \
    -out "docker.domain.com.crt" -extfile "site.cnf" -extensions server
```

这样已经拥有了 `docker.domain.com` 的网站 SSL 私钥 `docker.domain.com.key` 和 SSL 证书 `docker.domain.com.crt` 及 CA 根证书 `root-ca.crt`。

新建 `ssl` 文件夹并将 `docker.domain.com.key` `docker.domain.com.crt` `root-ca.crt` 这三个文件移入，删除其他文件。

### 配置私有仓库

私有仓库默认的配置文件位于 `/etc/docker/registry/config.yml`，我们先在本地编辑 `config.yml`，之后挂载到容器中。

```yaml
version: 0.1
log:
  accesslog:
    disabled: true
  level: debug
  formatter: text
  fields:
    service: registry
    environment: staging
storage:
  delete:
    enabled: true
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
auth:
  htpasswd:
    realm: basic-realm
    path: /etc/docker/registry/auth/nginx.htpasswd
http:
  addr: :443
  host: https://docker.domain.com
  headers:
    X-Content-Type-Options: [nosniff]
  http2:
    disabled: false
  tls:
    certificate: /etc/docker/registry/ssl/docker.domain.com.crt
    key: /etc/docker/registry/ssl/docker.domain.com.key
health:
  storagedriver:
    enabled: true
    interval: 10s
threshold: 3
```

### 生成 http 认证文件

```bash
$ mkdir auth
$ docker run --rm \
    --entrypoint htpasswd \
    httpd:alpine \
    -Bbn username password > auth/nginx.htpasswd
```

> 将上面的 `username` `password` 替换为你自己的用户名和密码。

### 编辑 `docker-compose.yml`

```yaml
version: '3'
services:
  registry:
    image: registry
    ports:
      - "443:443"
    volumes:
      - ./:/etc/docker/registry
      - registry-data:/var/lib/registry
volumes:
  registry-data:
```

### 修改 hosts

编辑 `/etc/hosts`

```bash
127.0.0.1 docker.domain.com
```

### 启动

```bash
$ docker-compose up -d
```

这样我们就搭建好了一个具有权限认证、TLS 的私有仓库，接下来我们测试其功能是否正常。

### 测试私有仓库功能

由于自行签发的 CA 根证书不被系统信任，所以我们需要将 CA 根证书 `ssl/root-ca.crt` 移入 `/etc/docker/certs.d/docker.domain.com` 文件夹中。

```bash
$ sudo mkdir -p /etc/docker/certs.d/docker.domain.com
$ sudo cp ssl/root-ca.crt /etc/docker/certs.d/docker.domain.com/ca.crt
```

登录到私有仓库。

```bash
$ docker login docker.domain.com
```

尝试推送、拉取镜像。

```bash
$ docker pull ubuntu:18.04
$ docker tag ubuntu:18.04 docker.domain.com/username/ubuntu:18.04
$ docker push docker.domain.com/username/ubuntu:18.04
$ docker image rm docker.domain.com/username/ubuntu:18.04
$ docker pull docker.domain.com/username/ubuntu:18.04
```

如果我们退出登录，尝试推送镜像。

```bash
$ docker logout docker.domain.com
$ docker push docker.domain.com/username/ubuntu:18.04
no basic auth credentials
```

发现会提示没有登录，不能将镜像推送到私有仓库中。

### 注意事项

如果你本机占用了 `443` 端口，你可以配置 [Nginx 代理](https://docs.docker.com/registry/recipes/nginx/)，这里不再赘述。

## Nexus3.x 的私有仓库

使用 Docker 官方的 Registry 创建的仓库面临一些维护问题。比如某些镜像删除以后空间默认是不会回收的，需要一些命令去回收空间然后重启 Registry。在企业中把内部的一些工具包放入 `Nexus` 中是比较常见的做法，最新版本 `Nexus3.x` 全面支持 Docker 的私有镜像。所以使用 [`Nexus3.x`](https://www.sonatype.com/product/repository-oss-download) 一个软件来管理 `Docker` , `Maven` , `Yum` , `PyPI` 等是一个明智的选择。

### 启动 Nexus 容器

```bash
$ docker run -d --name nexus3 --restart=always \
    -p 8081:8081 \
    --mount src=nexus-data,target=/nexus-data \
    sonatype/nexus3
```

首次运行需等待 3-5 分钟，你可以使用 `docker logs nexus3 -f` 查看日志：

```bash
$ docker logs nexus3 -f
2021-03-11 15:31:21,990+0000 INFO  [jetty-main-1] *SYSTEM org.sonatype.nexus.bootstrap.jetty.JettyServer -
-------------------------------------------------
Started Sonatype Nexus OSS 3.30.0-01
-------------------------------------------------
```

如果你看到以上内容，说明 `Nexus` 已经启动成功，你可以使用浏览器打开 `http://YourIP:8081` 访问 `Nexus` 了。

首次运行请通过以下命令获取初始密码：

```bash
$ docker exec nexus3 cat /nexus-data/admin.password
9266139e-41a2-4abb-92ec-e4142a3532cb
```

首次启动 Nexus 的默认帐号是 `admin` ，密码则是上边命令获取到的，点击右上角登录，首次登录需更改初始密码。

登录之后可以点击页面上方的齿轮按钮按照下面的方法进行设置。

### 创建仓库

创建一个私有仓库的方法： `Repository->Repositories` 点击右边菜单 `Create repository` 选择 `docker (hosted)`

* **Name**: 仓库的名称
* **HTTP**: 仓库单独的访问端口（例如：**5001**）
* **Hosted -> Deployment pollcy**: 请选择 **Allow redeploy** 否则无法上传 Docker 镜像。

其它的仓库创建方法请各位自己摸索，还可以创建一个 `docker (proxy)` 类型的仓库链接到 DockerHub 上。再创建一个 `docker (group)` 类型的仓库把刚才的 `hosted` 与 `proxy` 添加在一起。主机在访问的时候默认下载私有仓库中的镜像，如果没有将链接到 DockerHub 中下载并缓存到 Nexus 中。

### 添加访问权限

菜单 `Security->Realms` 把 Docker Bearer Token Realm 移到右边的框中保存。

添加用户规则：菜单 `Security->Roles`->`Create role`  在 `Privlleges` 选项搜索 docker 把相应的规则移动到右边的框中然后保存。

添加用户：菜单 `Security->Users`->`Create local user` 在 `Roles` 选项中选中刚才创建的规则移动到右边的窗口保存。

### NGINX 加密代理

证书的生成请参见 [`私有仓库高级配置`](registry_auth.md) 里面证书生成一节。

NGINX 示例配置如下

```nginx
upstream register
{
    server "YourHostName OR IP":5001; #端口为上面添加私有镜像仓库时设置的 HTTP 选项的端口号
    check interval=3000 rise=2 fall=10 timeout=1000 type=http;
    check_http_send "HEAD / HTTP/1.0\r\n\r\n";
    check_http_expect_alive http_4xx;
}
server {
    server_name YourDomainName;#如果没有 DNS 服务器做解析，请删除此选项使用本机 IP 地址访问
    listen       443 ssl;
    ssl_certificate key/example.crt;
    ssl_certificate_key key/example.key;
    ssl_session_timeout  5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers   on;
    large_client_header_buffers 4 32k;
    client_max_body_size 300m;
    client_body_buffer_size 512k;
    proxy_connect_timeout 600;
    proxy_read_timeout   600;
    proxy_send_timeout   600;
    proxy_buffer_size    128k;
    proxy_buffers       4 64k;
    proxy_busy_buffers_size 128k;
    proxy_temp_file_write_size 512k;
    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port $server_port;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_redirect off;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_pass http://register;
        proxy_read_timeout 900s;
    }
    error_page   500 502 503 504  /50x.html;
}
```

### Docker 主机访问镜像仓库

如果不启用 SSL 加密可以通过 [前面章节](./registry.md) 的方法添加非 https 仓库地址到 Docker 的配置文件中然后重启 Docker。

使用 SSL 加密以后程序需要访问就不能采用修改配置的方式了。具体方法如下：

```bash
$ openssl s_client -showcerts -connect YourDomainName OR HostIP:443 </dev/null 2>/dev/null|openssl x509 -outform PEM >ca.crt
$ cat ca.crt | sudo tee -a /etc/ssl/certs/ca-certificates.crt
$ systemctl restart docker
```

使用 `docker login YourDomainName OR HostIP` 进行测试，用户名密码填写上面 Nexus 中设置的。

# Docker 数据管理

这一章介绍如何在 Docker 内部以及容器之间管理数据，在容器中管理数据主要有3种方式：

- 数据卷（Volumes）
- 挂载主机目录 (Bind mounts)
- tmpfs mount

![](types-of-mounts.png)


## 挂载主机目录

### 挂载一个主机目录作为数据卷

使用 `--mount` 标记可以指定挂载一个本地主机的目录到容器中去。本地目录的路径必须是绝对路径。

> 注意：使用 `-v` 参数时如果本地目录不存在 Docker 会自动为你创建一个文件夹，使用 `--mount` 参数时如果本地目录不存在，Docker 会报错。

```bash
$ docker run -d -P \
    --name web \
    --mount type=bind,source=/src/webapp,target=/usr/share/nginx/html \
    nginx:alpine
```

上面的命令加载主机的 `/src/webapp` 目录到容器的 `/usr/share/nginx/html`目录。这个功能在进行测试的时候十分方便，比如用户可以放置一些程序到本地目录中，来查看容器是否正常工作。

Docker 挂载主机目录的默认权限是 ***读写***，用户也可以通过增加 `readonly` 指定为 ***只读***。

```bash
$ docker run -d -P \
    --name web \
    --mount type=bind,source=/src/webapp,target=/usr/share/nginx/html,readonly \
    nginx:alpine
```

加了 `readonly` 之后，就挂载为 `只读` 了。如果你在容器内 `/usr/share/nginx/html` 目录新建文件，会显示如下错误：

```bash
/usr/share/nginx/html # touch new.txt
touch: new.txt: Read-only file system
```

### 查看数据卷的具体信息

在主机里使用以下命令可以查看 `web` 容器的信息

```bash
$ docker inspect web
```

**挂载主机目录** 的配置信息在 "Mounts" 键下面，注意`Type`为`bind`：

```json
"Mounts": [
    {
        "Type": "bind",
        "Source": "/src/webapp",
        "Destination": "/usr/share/nginx/html",
        "Mode": "",
        "RW": true,
        "Propagation": "rprivate"
    }
],
```

### 挂载一个本地主机文件作为数据卷

`--mount` 标记也可以从主机挂载单个文件到容器中：

```bash
$ docker run --rm -it \
   --mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history \
   ubuntu:18.04 \
   bash
root@2affd44b4667:/# history
1  ls
2  diskutil list
```

这样就可以记录在容器输入过的命令了。


## 数据卷

**数据卷** 是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：

* **数据卷** 可以在容器之间共享和重用

* 对 **数据卷** 的修改会立马生效

* 对 **数据卷** 的更新，不会影响镜像

* **数据卷** 默认会一直存在，即使容器被删除

>注意：**数据卷** 的使用，类似于 Linux 下对目录或文件进行 mount，镜像中的被指定为挂载点的目录中的文件会复制到数据卷中（仅数据卷为空时会复制）。

### 创建一个数据卷

```bash
$ docker volume create my-vol
```

查看所有的 数据卷：

```bash
$ docker volume ls
DRIVER              VOLUME NAME
local               my-vol
```

在主机里使用以下命令可以查看指定 数据卷 的信息：

```bash
$ docker volume inspect my-vol
[
    {
        "CreatedAt": "2022-04-22T02:29:40+08:00",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]
```

### 启动一个挂载数据卷的容器

在用 `docker run` 命令的时候，使用 `-v` 参数来将 `数据卷` 挂载到容器里。在一次 `docker run` 中可以挂载多个 `数据卷`。

下面创建一个名为 `web` 的容器，并加载一个 `数据卷` 到容器的 `/usr/share/nginx/html` 目录。

```bash
$ docker run -d -P \
    --name web \
    -v my-vol:/usr/share/nginx/html \
    nginx:alpine
```

Docker 挂载数据卷的默认权限是`rw` ***读写***，用户也可以通过增加 `ro` 指定为 ***只读***。

```bash
$ docker run -d -P \
    --name web \
    -v my-vol:/usr/share/nginx/html:ro \
    nginx:alpine
```

`-v`参数也可以直接挂载宿主机上的目录或文件：

```bash
$ docker run -d -P \
    --name web \
    -v $HOME/.bash_history:/root/.bash_history \
    nginx:alpine
```


### 查看数据卷的具体信息

在主机里使用以下命令可以查看 `web` 容器的信息：

```bash
$ docker inspect web
```

**数据卷** 的配置信息在 "Mounts" 键下面，注意`Type`为`volume`：

```json
"Mounts": [
  {
    "Type": "volume",
    "Name": "my-vol",
    "Source": "/var/lib/docker/volumes/my-vol/_data",
    "Destination": "/usr/share/nginx/html",
    "Driver": "local",
    "Mode": "z",
    "RW": true,
    "Propagation": ""
  }
],
```

### 删除数据卷

```bash
$ docker volume rm my-vol
```

`数据卷` 是被设计用来持久化数据的，它的生命周期独立于容器，Docker 不会在容器被删除后自动删除 `数据卷`，并且也不存在垃圾回收这样的机制来处理没有任何容器引用的 `数据卷`。如果需要在删除容器的同时移除数据卷。可以在删除容器的时候使用 `docker rm -v` 这个命令。

无主的数据卷可能会占据很多空间，要清理请使用以下命令：

```bash
$ docker volume prune
```


## Docker bind Mounts和volume及tmpfs对比

本文总结了 bind Mounts 和 volume 及 tmpfs 三种 Docker 管理容器数据方式的区别。

### bind mounts

根据官方文档的介绍：

> bind mounts have limited functionality compared to volumes. **When you use a bind mount, a file or directory on the host machine is mounted into a container. The file or directory is referenced by its absolute path on the host machine.** By contrast, when you use a volume, a new directory is created within Docker’s storage directory on the host machine, and Docker manages that directory’s contents.

`bind mounts`相比`volume`功能较为有限，创建一个 bind mount 之后，Host 上的一个文件/文件夹就相当于被挂载到容器里了，该文件/目录就被容器通过绝对路径引用了，**意味着这个文件/文件夹必须实现存在于 Host 上**。

而使用使用`volume`时，会在 Host 主机上新建一个**Docker 存储文件夹**，文件夹里的内容由 Docker 管理。

> The file or directory does not need to exist on the Docker host already. It is created on demand if it does not yet exist. bind mounts are very performant, but they rely on the host machine’s filesystem having a specific directory structure available. If you are developing new Docker applications, consider using named volumes instead.

`volume`映射的文件/目录不需要存在于 Host 上，它们是按需生成的，即可以被自动创建，不需要事先在 Host 上创建。

**虽然`bind mounts`很高效，但是它依赖 Host 上存在特定的目录结构**，这对于部署到新主机上显然不是一件好事，所以推荐新应用使用`volume`。

### volume

> Volumes are the preferred mechanism for persisting data generated by and used by Docker containers. While bind mounts are dependent on the directory structure and OS of the host machine, volumes are completely managed by Docker.

`volume`与`bind mounts`的区别在于后者依赖于 Host 上有特定的目录结构，而前者不需要，使用 volume 时文件的管理由 Docker 主导而不是 Host。

**文档还提到了`volume`相比于`bind mounts`的优点：**

- **更易于备份/转移**
- 同时可用于 Linux/Windows 容器
- 在多个容器间共享更安全
- **`volume`驱动能够让你在远程主机或云服务器上存储`volume`，还能实现数据加密等功能**
- 新容器可以提前填充内容到`volume`里
- 在 Mac 和 Windows 主机里`volume`性能比`bind mounts`好的多

### tmpfs

若容器产生的数据不需要持久化，可以考虑使用`tmpfs`来避免持久化数据和写入容器层(writable layer)，从而提高性能。

`tmpfs`是将文件写到内存中，可以避免写数据到容器层增加容器大小。

但`tmpfs`有两个缺点：

- 只能在 Linux Host 上使用
- 不能在多个容器间共享

下图解释了三者的关系：

![bind-mounts volume tmpfs](1.png)

### 参考资料：

[Use bind mounts](https://docs.docker.com/storage/bind-mounts/)

[Use volumes](https://docs.docker.com/storage/volumes/)

[Use tmpfs mounts](https://docs.docker.com/storage/tmpfs/)

# Docker 网络管理

## 外部访问容器

可以通过 `-P` 或 `-p` 参数来指定端口映射，以使外部可以访问容器中的服务。

当使用 `-P` 标记时，Docker 会随机映射一个端口到内部容器开放的网络端口。

使用 `docker container ls` 可以看到，本地主机的 32768 被映射到了容器的 80 端口。此时访问本机的 32768 端口即可访问容器内 NGINX 默认页面。

```bash
$ docker run -d -P nginx:alpine
$ docker container ls -l
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   NAMES
fae320d08268        nginx:alpine        "/docker-entrypoint.…"   24 seconds ago      Up 20 seconds       0.0.0.0:32768->80/tcp   bold_mcnulty
```

同样的，可以通过 `docker logs` 命令来查看访问记录。

```bash
$ docker logs fa
172.17.0.1 - - [25/Aug/2020:08:34:04 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:80.0) Gecko/20100101 Firefox/80.0" "-"
```

`-p` 则可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。支持的格式有 `ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort`。

### 映射所有接口地址

使用 `hostPort:containerPort` 格式本地的 80 端口映射到容器的 80 端口，可以执行

```bash
$ docker run -d -p 80:80 nginx:alpine
```

此时默认会绑定本地所有接口上的所有地址。

### 映射到指定地址的指定端口

可以使用 `ip:hostPort:containerPort` 格式指定映射使用一个特定地址，比如 localhost 地址 127.0.0.1

```bash
$ docker run -d -p 127.0.0.1:80:80 nginx:alpine
```

### 映射到指定地址的任意端口

使用 `ip::containerPort` 绑定 localhost 的任意端口到容器的 80 端口，本地主机会自动分配一个端口。

```bash
$ docker run -d -p 127.0.0.1::80 nginx:alpine
```

还可以使用 `udp` 标记来指定 `udp` 端口

```bash
$ docker run -d -p 127.0.0.1:80:80/udp nginx:alpine
```

### 查看映射端口配置

使用 `docker port` 来查看当前映射的端口配置，也可以查看到绑定的地址

```bash
$ docker port fa 80
0.0.0.0:32768
```

注意：
* 容器有自己的内部网络和 ip 地址（使用 `docker inspect` 查看，Docker 还可以有一个可变的网络配置。）

* `-p` 标记可以多次使用来绑定多个端口

例如：

```bash
$ docker run -d \
    -p 80:80 \
    -p 443:443 \
    nginx:alpine
```

## 容器互联

如果你之前有 `Docker` 使用经验，你可能已经习惯了使用 `--link` 参数来使容器互联。随着 Docker 网络的完善，强烈建议大家将容器加入自定义的 Docker 网络来连接多个容器，而不是使用 `--link` 参数。

### 新建网络

下面先创建一个新的 Docker 网络。

```bash
$ docker network create -d bridge my-net
```

`-d` 参数指定 Docker 网络类型，有 `bridge` `overlay`。其中 `overlay` 网络类型用于 [Swarm mode](../swarm_mode/)，在本小节中你可以忽略它。

### 连接容器

运行一个容器并连接到新建的 `my-net` 网络

```bash
$ docker run -it --rm --name busybox1 --network my-net busybox sh
```

打开新的终端，再运行一个容器并加入到 `my-net` 网络

```bash
$ docker run -it --rm --name busybox2 --network my-net busybox sh
```

再打开一个新的终端查看容器信息

```bash
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b47060aca56b        busybox             "sh"                11 minutes ago      Up 11 minutes                           busybox2
8720575823ec        busybox             "sh"                16 minutes ago      Up 16 minutes                           busybox1
```

下面通过 `ping` 来证明 `busybox1` 容器和 `busybox2` 容器建立了互联关系。

在 `busybox1` 容器输入以下命令

```bash
/ # ping busybox2
PING busybox2 (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.072 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.118 ms
```

用 ping 来测试连接 `busybox2` 容器，它会解析成 `172.19.0.3`。

同理在 `busybox2` 容器执行 `ping busybox1`，也会成功连接到。

```bash
/ # ping busybox1
PING busybox1 (172.19.0.2): 56 data bytes
64 bytes from 172.19.0.2: seq=0 ttl=64 time=0.064 ms
64 bytes from 172.19.0.2: seq=1 ttl=64 time=0.143 ms
```

这样，`busybox1` 容器和 `busybox2` 容器建立了互联关系。

**如果你有多个容器之间需要互相连接，推荐使用 [Docker Compose](../compose)。**

## 配置 DNS

如何自定义配置容器的主机名和 DNS 呢？秘诀就是 Docker 利用虚拟文件来挂载容器的 3 个相关配置文件。

在容器中使用 `mount` 命令可以看到挂载信息：

```bash
$ mount
/dev/disk/by-uuid/1fec...ebdf on /etc/hostname type ext4 ...
/dev/disk/by-uuid/1fec...ebdf on /etc/hosts type ext4 ...
tmpfs on /etc/resolv.conf type tmpfs ...
```

这种机制可以让宿主主机 DNS 信息发生更新后，所有 Docker 容器的 DNS 配置通过 `/etc/resolv.conf` 文件立刻得到更新。

配置全部容器的 DNS ，也可以在 `/etc/docker/daemon.json` 文件中增加以下内容来设置。

```json
{
  "dns" : [
    "114.114.114.114",
    "8.8.8.8"
  ]
}
```

这样每次启动的容器 DNS 自动配置为 `114.114.114.114` 和 `8.8.8.8`。使用以下命令来证明其已经生效：

```bash
$ docker run -it --rm ubuntu:18.04  cat etc/resolv.conf
nameserver 114.114.114.114
nameserver 8.8.8.8
```

如果用户想要手动指定容器的配置，可以在使用 `docker run` 命令启动容器时加入如下参数：

- `-h HOSTNAME` 或者 `--hostname=HOSTNAME` 设定容器的主机名，它会被写到容器内的 `/etc/hostname` 和 `/etc/hosts`。但它在容器外部看不到，既不会在 `docker container ls` 中显示，也不会在其他的容器的 `/etc/hosts` 看到。

- `--dns=IP_ADDRESS` 添加 DNS 服务器到容器的 `/etc/resolv.conf` 中，让容器用这个服务器来解析所有不在 `/etc/hosts` 中的主机名。

- `--dns-search=DOMAIN` 设定容器的搜索域，当设定搜索域为 `.example.com` 时，在搜索一个名为 host 的主机时，DNS 不仅搜索 host，还会搜索 `host.example.com`。

>注意：如果在容器启动时没有指定最后两个参数，Docker 会默认用主机上的 `/etc/resolv.conf` 来配置容器。

# 高级网络配置

本章将介绍 Docker 的一些高级网络配置和选项。

当 Docker 启动时，会自动在主机上创建一个 `docker0` 虚拟网桥，实际上是 Linux 的一个 bridge，可以理解为一个软件交换机。它会在挂载到它的网口之间进行转发。

同时，Docker 随机分配一个本地未占用的私有网段（在 [RFC1918](https://datatracker.ietf.org/doc/html/rfc1918) 中定义）中的一个地址给 `docker0` 接口。比如典型的 `172.17.42.1`，掩码为 `255.255.0.0`。此后启动的容器内的网口也会自动分配一个同一网段（`172.17.0.0/16`）的地址。

当创建一个 Docker 容器的时候，同时会创建了一对 `veth pair` 接口（当数据包发送到一个接口时，另外一个接口也可以收到相同的数据包）。这对接口一端在容器内，即 `eth0`；另一端在本地并被挂载到 `docker0` 网桥，名称以 `veth` 开头（例如 `vethAQI2QT`）。通过这种方式，主机可以跟容器通信，容器之间也可以相互通信。Docker 就创建了在主机和所有容器之间一个虚拟共享网络。

![Docker 网络](./network.png)

接下来的部分将介绍在一些场景中，Docker 所有的网络定制配置。以及通过 Linux 命令来调整、补充、甚至替换 Docker 默认的网络配置。

## 快速配置指南

`dockerd`命令可以设置Docker网络相关的配置。

以下选项只有在 Docker 服务启动的时候才能配置，而且不能马上生效：

* `-b BRIDGE` 或 `--bridge=BRIDGE` 指定容器挂载的网桥
* `--bip=CIDR` 定制 docker0 的掩码
* `-H SOCKET...` 或 `--host=SOCKET...` Docker 服务端接收命令的通道
* `--icc=true|false` 是否支持容器之间进行通信
* `--ip-forward=true|false` 请看下文容器之间的通信
* `--iptables=true|false` 是否允许 Docker 添加 iptables 规则
* `--mtu=BYTES` 容器网络中的 MTU

下面2个命令选项既可以在启动服务时指定，也可以在启动容器时指定。在 Docker 服务启动的时候指定则会成为默认值，后面执行 `docker run` 时可以覆盖设置的默认值。

* `--dns=IP_ADDRESS...` 使用指定的DNS服务器
* `--dns-search=DOMAIN...` 指定DNS搜索域

以下这些选项只有在 `docker run` 执行时使用，因为它是针对容器的特性内容。

* `-h HOSTNAME` 或 `--hostname=HOSTNAME` 配置容器主机名
* `--link=CONTAINER_NAME:ALIAS` 添加到另一个容器的连接
* `--net=bridge|none|container:NAME_or_ID|host` 配置容器的桥接模式
* `-p SPEC` 或 `--publish=SPEC` 映射容器端口到宿主主机
* `-P or --publish-all=true|false` 映射容器所有端口到宿主主机

## 容器访问控制

容器的访问控制，主要通过 Linux 上的 `iptables` 防火墙来进行管理和实现。`iptables` 是 Linux 上默认的防火墙软件，在大部分发行版中都自带。

### 容器访问外部网络
容器要想访问外部网络，需要本地系统的转发支持。在Linux 系统中，检查转发是否打开。

```bash
$sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```
如果为 0，说明没有开启转发，则需要手动打开。
```bash
$sysctl -w net.ipv4.ip_forward=1
```
如果在启动 Docker 服务的时候设定 `--ip-forward=true`, Docker 就会自动设定系统的 `ip_forward` 参数为 1。

### 容器之间访问
容器之间相互访问，需要两方面的支持：
* 容器的网络拓扑是否已经互联。默认情况下，所有容器都会被连接到 `docker0` 网桥上。
* 本地系统的防火墙软件 -- `iptables` 是否允许通过。

#### 访问所有端口

当启动 Docker 服务（即 dockerd）的时候，默认会添加一条转发策略到本地主机 iptables 的 FORWARD 链上。策略为通过（`ACCEPT`）还是禁止（`DROP`）取决于配置`--icc=true`（缺省值）还是 `--icc=false`。当然，如果手动指定 `--iptables=false` 则不会添加 `iptables` 规则。

可见，默认情况下，不同容器之间是允许网络互通的。如果为了安全考虑，可以在 `/etc/docker/daemon.json` 文件中配置 `{"icc": false}` 来禁止它。

#### 访问指定端口

在通过 `-icc=false` 关闭网络访问后，还可以通过 `--link=CONTAINER_NAME:ALIAS` 选项来访问容器的开放端口。

例如，在启动 Docker 服务时，可以同时使用 `icc=false --iptables=true` 参数来关闭允许相互的网络访问，并让 Docker 可以修改系统中的 `iptables` 规则。

此时，系统中的 `iptables` 规则可能是类似
```bash
$ sudo iptables -nL
...
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
DROP       all  --  0.0.0.0/0            0.0.0.0/0
...
```

之后，启动容器（`docker run`）时使用 `--link=CONTAINER_NAME:ALIAS` 选项。Docker 会在 `iptables` 中为 两个容器分别添加一条 `ACCEPT` 规则，允许相互访问开放的端口（取决于 `Dockerfile` 中的 `EXPOSE` 指令）。

当添加了 `--link=CONTAINER_NAME:ALIAS` 选项后，添加了 `iptables` 规则。
```bash
$ sudo iptables -nL
...
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
ACCEPT     tcp  --  172.17.0.2           172.17.0.3           tcp spt:80
ACCEPT     tcp  --  172.17.0.3           172.17.0.2           tcp dpt:80
DROP       all  --  0.0.0.0/0            0.0.0.0/0
```

注意：`--link=CONTAINER_NAME:ALIAS` 中的 `CONTAINER_NAME` 目前必须是 Docker 分配的名字，或使用 `--name` 参数指定的名字。主机名则不会被识别。


## 映射容器端口到宿主主机

默认情况下，容器可以主动访问到外部网络的连接，但是外部网络无法访问到容器。

### 容器访问外部实现

容器所有到外部网络的连接，源地址都会被 NAT 成本地系统的 IP 地址。这是使用 `iptables` 的源地址伪装实现的。

查看主机的 NAT 规则。

```bash
$ sudo iptables -t nat -nL
...
Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  172.17.0.0/16       !172.17.0.0/16
...
```

其中，上述规则将所有源地址在 `172.17.0.0/16` 网段，目标地址为其他网段（外部网络）的流量动态伪装为从系统网卡发出。MASQUERADE 跟传统 SNAT 的好处是它能动态从网卡获取地址。

### 外部访问容器实现

容器允许外部访问，可以在 `docker run` 时候通过 `-p` 或 `-P` 参数来启用。不管用那种办法，其实也是在本地的 `iptable` 的 nat 表中添加相应的规则。

使用 `-P` 时：

```bash
$ iptables -t nat -nL
...
Chain DOCKER (2 references)
target     prot opt source               destination
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:49153 to:172.17.0.2:80
```

使用 `-p 80:80` 时：

```bash
$ iptables -t nat -nL
Chain DOCKER (2 references)
target     prot opt source               destination
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:80 to:172.17.0.2:80
```

注意：

* 这里的规则映射了 `0.0.0.0`，意味着将接受主机来自所有接口的流量。用户可以通过 `-p IP:host_port:container_port` 或 `-p IP::port` 来指定允许访问容器的主机上的 IP、接口等，以制定更严格的规则。

* 如果希望永久绑定到某个固定的 IP 地址，可以在 Docker 配置文件 `/etc/docker/daemon.json` 中添加如下内容。

```json
{
  "ip": "0.0.0.0"
}
```

## 配置 `docker0` 网桥

Docker 服务默认会创建一个 `docker0` 网桥（其上有一个 `docker0` 内部接口），它在内核层连通了其他的物理或虚拟网卡，这就将所有容器和本地主机都放到同一个物理网络。

Docker 默认指定了 `docker0` 接口 的 IP 地址、子网掩码、MTU、默认路由，让主机和容器之间可以通过网桥相互通信。**这些值都可以在服务启动的时候进行配置：**

* `--bip=CIDR` IP 地址加掩码格式，例如 192.168.1.5/24
* `--mtu=BYTES` 覆盖默认的 Docker mtu 配置

**也可以在配置文件中配置 DOCKER_OPTS，然后重启服务。**

由于目前 Docker 网桥是 Linux 网桥，用户可以使用 `brctl show` 来查看网桥和端口连接信息。

```bash
$ sudo brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.3a1d7362b4ee       no              veth65f9
                                             vethdda6
```
*注：`brctl` 命令在 Debian、Ubuntu 中可以使用 `sudo apt-get install bridge-utils` 来安装。


每次创建一个新容器的时候，Docker 从可用的地址段中选择一个空闲的 IP 地址分配给容器的 eth0 端口。使用本地主机上 `docker0` 接口的 IP 作为所有容器的默认网关。

```bash
$ sudo docker run -i -t --rm base /bin/bash
$ ip addr show eth0
24: eth0: <BROADCAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 32:6f:e0:35:57:91 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.3/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::306f:e0ff:fe35:5791/64 scope link
       valid_lft forever preferred_lft forever
$ ip route
default via 172.17.42.1 dev eth0
172.17.0.0/16 dev eth0  proto kernel  scope link  src 172.17.0.3
```


## 自定义网桥

除了默认的 `docker0` 网桥，用户也可以指定网桥来连接各个容器。

在启动 Docker 服务的时候，使用 `-b BRIDGE`或`--bridge=BRIDGE` 来指定使用的网桥。

如果服务已经运行，那需要先停止服务，并删除旧的网桥。

```bash
$ sudo systemctl stop docker
$ sudo ip link set dev docker0 down
$ sudo brctl delbr docker0
```

然后创建一个网桥 `bridge0`。

```bash
$ sudo brctl addbr bridge0
$ sudo ip addr add 192.168.5.1/24 dev bridge0
$ sudo ip link set dev bridge0 up
```

查看确认网桥创建并启动。

```bash
$ ip addr show bridge0
4: bridge0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state UP group default
    link/ether 66:38:d0:0d:76:18 brd ff:ff:ff:ff:ff:ff
    inet 192.168.5.1/24 scope global bridge0
       valid_lft forever preferred_lft forever
```

在 Docker 配置文件 `/etc/docker/daemon.json` 中添加如下内容，即可将 Docker 默认桥接到创建的网桥上。

```json
{
  "bridge": "bridge0",
}
```

启动 Docker 服务。

新建一个容器，可以看到它已经桥接到了 `bridge0` 上。

可以继续用 `brctl show` 命令查看桥接的信息。另外，在容器中可以使用 `ip addr` 和 `ip route` 命令来查看 IP 地址配置和路由信息。

## 工具和示例

在介绍自定义网络拓扑之前，你可能会对一些外部工具和例子感兴趣：

### pipework
Jérôme Petazzoni 编写了一个叫 [pipework](https://github.com/jpetazzo/pipework) 的 shell 脚本，可以帮助用户在比较复杂的场景中完成容器的连接。

### playground
Brandon Rhodes 创建了一个提供完整的 Docker 容器网络拓扑管理的 [Python库](https://github.com/brandon-rhodes/fopnp/tree/m/playground)，包括路由、NAT 防火墙；以及一些提供 `HTTP` `SMTP` `POP` `IMAP` `Telnet` `SSH` `FTP` 的服务器。

## 编辑网络配置文件

Docker 1.2.0 开始支持在运行中的容器里编辑 `/etc/hosts`, `/etc/hostname` 和 `/etc/resolv.conf` 文件。

**但是这些修改是临时的，只在运行的容器中保留，容器终止或重启后并不会被保存下来，也不会被 `docker commit` 提交。**


## 示例：创建一个点到点连接
默认情况下，Docker 会将所有容器连接到由 `docker0` 提供的虚拟子网中。

用户有时候需要两个容器之间可以直连通信，而不用通过主机网桥进行桥接。

解决办法很简单：创建一对 `peer` 接口，分别放到两个容器中，配置成点到点链路类型即可。

首先启动 2 个容器：
```bash
$ docker run -i -t --rm --net=none base /bin/bash
root@1f1f4c1f931a:/#
$ docker run -i -t --rm --net=none base /bin/bash
root@12e343489d2f:/#
```

找到进程号，然后创建网络命名空间的跟踪文件。
```bash
$ docker inspect -f '{{.State.Pid}}' 1f1f4c1f931a
2989
$ docker inspect -f '{{.State.Pid}}' 12e343489d2f
3004
$ sudo mkdir -p /var/run/netns
$ sudo ln -s /proc/2989/ns/net /var/run/netns/2989
$ sudo ln -s /proc/3004/ns/net /var/run/netns/3004
```

创建一对 `peer` 接口，然后配置路由
```bash
$ sudo ip link add A type veth peer name B
$ sudo ip link set A netns 2989
$ sudo ip netns exec 2989 ip addr add 10.1.1.1/32 dev A
$ sudo ip netns exec 2989 ip link set A up
$ sudo ip netns exec 2989 ip route add 10.1.1.2/32 dev A
$ sudo ip link set B netns 3004
$ sudo ip netns exec 3004 ip addr add 10.1.1.2/32 dev B
$ sudo ip netns exec 3004 ip link set B up
$ sudo ip netns exec 3004 ip route add 10.1.1.1/32 dev B
```
现在这 2 个容器就可以相互 ping 通，并成功建立连接。点到点链路不需要子网和子网掩码。

此外，也可以不指定 `--net=none` 来创建点到点链路。这样容器还可以通过原先的网络来通信。

利用类似的办法，可以创建一个只跟主机通信的容器。但是一般情况下，更推荐使用 `--icc=false` 来关闭容器之间的通信。

# Docker Compose 项目

`Docker Compose` 是 Docker 官方编排（Orchestration）项目之一，负责快速的部署分布式应用。

本章将介绍 `Compose` 项目情况以及安装和使用。

## Compose 简介

`Compose` 项目是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。从功能上看，跟 `OpenStack` 中的 `Heat` 十分类似。

其代码目前在 [https://github.com/docker/compose](https://github.com/docker/compose) 上开源。

`Compose` 定位是 「定义和运行多个 Docker 容器的应用（Defining and running multi-container Docker applications）」，其前身是开源项目 Fig。

通过第一部分中的介绍，我们知道使用一个 `Dockerfile` 模板文件，可以让用户很方便的定义一个单独的应用容器。然而，在日常工作中，经常会碰到需要多个容器相互配合来完成某项任务的情况。例如要实现一个 Web 项目，除了 Web 服务容器本身，往往还需要再加上后端的数据库服务容器，甚至还包括负载均衡容器等。

`Compose` 恰好满足了这样的需求。它允许用户通过一个单独的 `docker-compose.yml` 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。

`Compose` 中有两个重要的概念：

* 服务 (`service`)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。

* 项目 (`project`)：由一组关联的应用容器组成的一个完整业务单元，在 `docker-compose.yml` 文件中定义。

`Compose` 的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。

`Compose` 项目由 Python 编写，实现上调用了 Docker 服务提供的 API 来对容器进行管理。因此，只要所操作的平台支持 Docker API，就可以在其上利用 `Compose` 来进行编排管理。

## Compose V2

目前 Docker 官方用 GO 语言 [重写](https://github.com/docker/compose-cli) 了 Docker Compose，并将其作为了 docker cli 的子命令，称为 `Compose V2`。你可以参照官方文档安装，然后将熟悉的 `docker-compose` 命令替换为 `docker compose`，即可使用 Docker Compose。

### 官方文档

* [Compose V2 beta](https://docs.docker.com/compose/cli-command/)
* https://github.com/docker/compose



## 使用

### 术语

首先介绍几个术语。

* 服务 (`service`)：一个应用容器，实际上可以运行多个相同镜像的实例。

* 项目 (`project`)：由一组关联的应用容器组成的一个完整业务单元。

可见，一个项目可以由多个服务（容器）关联而成，`Compose` 面向项目进行管理。

### 场景

最常见的项目是 web 网站，该项目应该包含 web 应用和缓存。

下面我们用 `Python` 来建立一个能够记录页面访问次数的 web 网站。

#### web 应用

新建文件夹，在该目录中编写 `app.py` 文件

```python
from flask import Flask
from redis import Redis
app = Flask(__name__)
redis = Redis(host='redis', port=6379)
@app.route('/')
def hello():
    count = redis.incr('hits')
    return 'Hello World! 该页面已被访问 {} 次。\n'.format(count)
if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```

#### Dockerfile

编写 `Dockerfile` 文件，内容为

```docker
FROM python:3.6-alpine
ADD . /code
WORKDIR /code
RUN pip install redis flask
CMD ["python", "app.py"]
```

#### docker-compose.yml

编写 `docker-compose.yml` 文件，这个是 Compose 使用的主模板文件。

```yaml
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
  redis:
    image: "redis:alpine"
```

#### 运行 compose 项目

```bash
$ docker-compose up
```

此时访问本地 `5000` 端口，每次刷新页面，计数就会加 1。

## Compose 命令说明

### 命令对象与格式

对于 Compose 来说，大部分命令的对象既可以是项目本身，也可以指定为项目中的服务或者容器。如果没有特别的说明，命令对象将是项目，这意味着项目中所有的服务都会受到命令影响。

执行 `docker-compose [COMMAND] --help` 或者 `docker-compose help [COMMAND]` 可以查看具体某个命令的使用格式。

`docker-compose` 命令的基本的使用格式是

```bash
docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS...]
```

### 命令选项

* `-f, --file FILE` 指定使用的 Compose 模板文件，默认为 `docker-compose.yml`，可以多次指定。

* `-p, --project-name NAME` 指定项目名称，默认将使用所在目录名称作为项目名。

* `--verbose` 输出更多调试信息。

* `-v, --version` 打印版本并退出。

### 命令使用说明

#### `build`

格式为 `docker-compose build [options] [SERVICE...]`。

构建（重新构建）项目中的服务容器。

服务容器一旦构建后，将会带上一个标记名，例如对于 web 项目中的一个 db 容器，可能是 web_db。

可以随时在项目目录下运行 `docker-compose build` 来重新构建服务。

选项包括：

* `--force-rm` 删除构建过程中的临时容器。

* `--no-cache` 构建镜像过程中不使用 cache（这将加长构建过程）。

* `--pull` 始终尝试通过 pull 来获取更新版本的镜像。

#### `config`

验证 Compose 文件格式是否正确，若正确则显示配置，若格式错误显示错误原因。

#### `down`

此命令将会停止 `up` 命令所启动的容器，并移除网络

#### `exec`

进入指定的容器。

#### `help`

获得一个命令的帮助。

#### `images`

列出 Compose 文件中包含的镜像。

#### `kill`

格式为 `docker-compose kill [options] [SERVICE...]`。

通过发送 `SIGKILL` 信号来强制停止服务容器。

支持通过 `-s` 参数来指定发送的信号，例如通过如下指令发送 `SIGINT` 信号。

```bash
$ docker-compose kill -s SIGINT
```

#### `logs`

格式为 `docker-compose logs [options] [SERVICE...]`。

查看服务容器的输出。默认情况下，docker-compose 将对不同的服务输出使用不同的颜色来区分。可以通过 `--no-color` 来关闭颜色。

该命令在调试问题的时候十分有用。

#### `pause`

格式为 `docker-compose pause [SERVICE...]`。

暂停一个服务容器。

#### `port`

格式为 `docker-compose port [options] SERVICE PRIVATE_PORT`。

打印某个容器端口所映射的公共端口。

选项：

* `--protocol=proto` 指定端口协议，tcp（默认值）或者 udp。

* `--index=index` 如果同一服务存在多个容器，指定命令对象容器的序号（默认为 1）。

#### `ps`

格式为 `docker-compose ps [options] [SERVICE...]`。

列出项目中目前的所有容器。

选项：

* `-q` 只打印容器的 ID 信息。

#### `pull`

格式为 `docker-compose pull [options] [SERVICE...]`。

拉取服务依赖的镜像。

选项：

* `--ignore-pull-failures` 忽略拉取镜像过程中的错误。

#### `push`

推送服务依赖的镜像到 Docker 镜像仓库。

#### `restart`

格式为 `docker-compose restart [options] [SERVICE...]`。

重启项目中的服务。

选项：

* `-t, --timeout TIMEOUT` 指定重启前停止容器的超时（默认为 10 秒）。

#### `rm`

格式为 `docker-compose rm [options] [SERVICE...]`。

删除所有（停止状态的）服务容器。推荐先执行 `docker-compose stop` 命令来停止容器。

选项：

* `-f, --force` 强制直接删除，包括非停止状态的容器。一般尽量不要使用该选项。

* `-v` 删除容器所挂载的数据卷。

#### `run`
格式为 `docker-compose run [options] [-p PORT...] [-e KEY=VAL...] SERVICE [COMMAND] [ARGS...]`。

在指定服务上执行一个命令。

例如：

```bash
$ docker-compose run ubuntu ping docker.com
```

将会启动一个 ubuntu 服务容器，并执行 `ping docker.com` 命令。

默认情况下，如果存在关联，则所有关联的服务将会自动被启动，除非这些服务已经在运行中。

该命令类似启动容器后运行指定的命令，相关卷、链接等等都将会按照配置自动创建。

两个不同点：

* 给定命令将会覆盖原有的自动运行命令；

* 不会自动创建端口，以避免冲突。

如果不希望自动启动关联的容器，可以使用 `--no-deps` 选项，例如

```bash
$ docker-compose run --no-deps web python manage.py shell
```

将不会启动 web 容器所关联的其它容器。

选项：

* `-d` 后台运行容器。

* `--name NAME` 为容器指定一个名字。

* `--entrypoint CMD` 覆盖默认的容器启动指令。

* `-e KEY=VAL` 设置环境变量值，可多次使用选项来设置多个环境变量。

* `-u, --user=""` 指定运行容器的用户名或者 uid。

* `--no-deps` 不自动启动关联的服务容器。

* `--rm` 运行命令后自动删除容器，`d` 模式下将忽略。

* `-p, --publish=[]` 映射容器端口到本地主机。

* `--service-ports` 配置服务端口并映射到本地主机。

* `-T` 不分配伪 tty，意味着依赖 tty 的指令将无法运行。

#### `scale`

> **Deprecated in docker-compose v2 (use `compose up --scale` instead)**

格式为 `docker-compose scale [options] [SERVICE=NUM...]`。

设置指定服务运行的容器个数。

通过 `service=num` 的参数来设置数量。例如：

```bash
$ docker-compose scale web=3 db=2
```

将启动 3 个容器运行 web 服务，2 个容器运行 db 服务。

一般的，当指定数目多于该服务当前实际运行容器，将新创建并启动容器；反之，将停止容器。

选项：

* `-t, --timeout TIMEOUT` 停止容器时候的超时（默认为 10 秒）。

#### `start`

格式为 `docker-compose start [SERVICE...]`。

启动已经存在的服务容器。

#### `stop`

格式为 `docker-compose stop [options] [SERVICE...]`。

停止已经处于运行状态的容器，但不删除它。通过 `docker-compose start` 可以再次启动这些容器。

选项：

* `-t, --timeout TIMEOUT` 停止容器时候的超时（默认为 10 秒）。

#### `top`

查看各个服务容器内运行的进程。

#### `unpause`

格式为 `docker-compose unpause [SERVICE...]`。

恢复处于暂停状态中的服务。

#### `up`

格式为 `docker-compose up [options] [SERVICE...]`。

该命令十分强大，它将尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联服务相关容器的一系列操作。

链接的服务都将会被自动启动，除非已经处于运行状态。

可以说，大部分时候都可以直接通过该命令来启动一个项目。

默认情况，`docker-compose up` 启动的容器都在前台，控制台将会同时打印所有容器的输出信息，可以很方便进行调试。

当通过 `Ctrl-C` 停止命令时，所有容器将会停止。

如果使用 `docker-compose up -d`，将会在后台启动并运行所有的容器。一般推荐生产环境下使用该选项。

默认情况，如果服务容器已经存在，`docker-compose up` 将会尝试停止容器，然后重新创建（保持使用 `volumes-from` 挂载的卷），以保证新启动的服务匹配 `docker-compose.yml` 文件的最新内容。如果用户不希望容器被停止并重新创建，可以使用 `docker-compose up --no-recreate`。这样将只会启动处于停止状态的容器，而忽略已经运行的服务。如果用户只想重新部署某个服务，可以使用 `docker-compose up --no-deps -d <SERVICE_NAME>` 来重新创建服务并后台停止旧服务，启动新服务，并不会影响到其所依赖的服务。

选项：

* `-d` 在后台运行服务容器。

* `--no-color` 不使用颜色来区分不同的服务的控制台输出。

* `--no-deps` 不启动服务所链接的容器。

* `--force-recreate` 强制重新创建容器，不能与 `--no-recreate` 同时使用。

* `--no-recreate` 如果容器已经存在了，则不重新创建，不能与 `--force-recreate` 同时使用。

* `--no-build` 不自动构建缺失的服务镜像。

* `-t, --timeout TIMEOUT` 停止容器时候的超时（默认为 10 秒）。

#### `version`

格式为 `docker-compose version`。

打印版本信息。

### 参考资料

* [官方文档](https://docs.docker.com/compose/reference/overview/)

## Compose 模板文件

模板文件是使用 `Compose` 的核心，涉及到的指令关键字也比较多。但大家不用担心，这里面大部分指令跟 `docker run` 相关参数的含义都是类似的。

默认的模板文件名称为 `docker-compose.yml`，格式为 YAML 格式。

```yaml
version: "3"
services:
  webapp:
    image: examples/web
    ports:
      - "80:80"
    volumes:
      - "/data"
```

注意每个服务都必须通过 `image` 指令指定镜像或 `build` 指令（需要 Dockerfile）等来自动构建生成镜像。

如果使用 `build` 指令，在 `Dockerfile` 中设置的选项(例如：`CMD`, `EXPOSE`, `VOLUME`, `ENV` 等) 将会自动被获取，无需在 `docker-compose.yml` 中重复设置。

下面分别介绍各个指令的用法。

### `build`

指定 `Dockerfile` 所在文件夹的路径（可以是绝对路径，或者相对 docker-compose.yml 文件的路径）。 `Compose` 将会利用它自动构建这个镜像，然后使用这个镜像。

```yaml
version: '3'
services:
  webapp:
    build: ./dir
```

你也可以使用 `context` 指令指定 `Dockerfile` 所在文件夹的路径。

使用 `dockerfile` 指令指定 `Dockerfile` 文件名。

使用 `arg` 指令指定构建镜像时的变量。

```yaml
version: '3'
services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```

使用 `cache_from` 指定构建镜像的缓存

```yaml
build:
  context: .
  cache_from:
    - alpine:latest
    - corp/web_app:3.14
```

### `cap_add, cap_drop`

指定容器的内核能力（capacity）分配。

例如，让容器拥有所有能力可以指定为：

```yaml
cap_add:
  - ALL
```

去掉 NET_ADMIN 能力可以指定为：

```yaml
cap_drop:
  - NET_ADMIN
```

### `command`

覆盖容器启动后默认执行的命令。

```yaml
command: echo "hello world"
```

### `configs`

仅用于 `Swarm mode`，详细内容请查看 [`Swarm mode`](../swarm_mode/) 一节。

### `cgroup_parent`

指定父 `cgroup` 组，意味着将继承该组的资源限制。

例如，创建了一个 cgroup 组名称为 `cgroups_1`。

```yaml
cgroup_parent: cgroups_1
```

### `container_name`

指定容器名称。默认将会使用 `项目名称_服务名称_序号` 这样的格式。

```yaml
container_name: docker-web-container
```

>注意: 指定容器名称后，该服务将无法进行扩展（scale），因为 Docker 不允许多个容器具有相同的名称。

### `deploy`

仅用于 `Swarm mode`，详细内容请查看 [`Swarm mode`](../swarm_mode/) 一节

### `devices`

指定设备映射关系。

```yaml
devices:
  - "/dev/ttyUSB1:/dev/ttyUSB0"
```

### `depends_on`

解决容器的依赖、启动先后的问题。以下例子中会先启动 `redis` `db` 再启动 `web`

```yaml
version: '3'
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

>注意：`web` 服务不会等待 `redis` `db` 「完全启动」之后才启动。

### `dns`

自定义 `DNS` 服务器。可以是一个值，也可以是一个列表。

```yaml
dns: 8.8.8.8
dns:
  - 8.8.8.8
  - 114.114.114.114
```

### `dns_search`

配置 `DNS` 搜索域。可以是一个值，也可以是一个列表。

```yaml
dns_search: example.com
dns_search:
  - domain1.example.com
  - domain2.example.com
```

### `tmpfs`

挂载一个 tmpfs 文件系统到容器。

```yaml
tmpfs: /run
tmpfs:
  - /run
  - /tmp
```

### `env_file`

从文件中获取环境变量，可以为单独的文件路径或列表。

如果通过 `docker-compose -f FILE` 方式来指定 Compose 模板文件，则 `env_file` 中变量的路径会基于模板文件路径。

如果有变量名称与 `environment` 指令冲突，则按照惯例，以后者为准。

```bash
env_file: .env
env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```

环境变量文件中每一行必须符合格式，支持 `#` 开头的注释行。

```bash
# common.env: Set development environment
PROG_ENV=development
```

### `environment`

设置环境变量。你可以使用数组或字典两种格式。

只给定名称的变量会自动获取运行 Compose 主机上对应变量的值，可以用来防止泄露不必要的数据。

```yaml
environment:
  RACK_ENV: development
  SESSION_SECRET:
environment:
  - RACK_ENV=development
  - SESSION_SECRET
```

如果变量名称或者值中用到 `true|false，yes|no` 等表达 [布尔](https://yaml.org/type/bool.html) 含义的词汇，最好放到引号里，避免 YAML 自动解析某些内容为对应的布尔语义。这些特定词汇，包括

```bash
y|Y|yes|Yes|YES|n|N|no|No|NO|true|True|TRUE|false|False|FALSE|on|On|ON|off|Off|OFF
```

### `expose`

暴露端口，但不映射到宿主机，只被连接的服务访问。

仅可以指定内部端口为参数

```yaml
expose:
 - "3000"
 - "8000"
```

### `external_links`

>注意：不建议使用该指令。

链接到 `docker-compose.yml` 外部的容器，甚至并非 `Compose` 管理的外部容器。

```yaml
external_links:
 - redis_1
 - project_db_1:mysql
 - project_db_1:postgresql
```

### `extra_hosts`

类似 Docker 中的 `--add-host` 参数，指定额外的 host 名称映射信息。

```yaml
extra_hosts:
 - "googledns:8.8.8.8"
 - "dockerhub:52.1.157.61"
```

会在启动后的服务容器中 `/etc/hosts` 文件中添加如下两条条目。

```bash
8.8.8.8 googledns
52.1.157.61 dockerhub
```

### `healthcheck`

通过命令检查容器是否健康运行。

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
```

### `image`

指定为镜像名称或镜像 ID。如果镜像在本地不存在，`Compose` 将会尝试拉取这个镜像。

```yaml
image: ubuntu
image: orchardup/postgresql
image: a4bc65fd
```

### `labels`

为容器添加 Docker 元数据（metadata）信息。例如可以为容器添加辅助说明信息。

```yaml
labels:
  com.startupteam.description: "webapp for a startup team"
  com.startupteam.department: "devops department"
  com.startupteam.release: "rc3 for v1.0"
```

### `links`

>注意：不推荐使用该指令。

### `logging`

配置日志选项。

```yaml
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```

目前支持三种日志驱动类型。

```yaml
driver: "json-file"
driver: "syslog"
driver: "none"
```

`options` 配置日志驱动的相关参数。

```yaml
options:
  max-size: "200k"
  max-file: "10"
```

### `network_mode`

设置网络模式。使用和 `docker run` 的 `--network` 参数一样的值。

```yaml
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```

### `networks`

配置容器连接的网络。

```yaml
version: "3"
services:
  some-service:
    networks:
     - some-network
     - other-network
networks:
  some-network:
  other-network:
```

### `pid`

跟主机系统共享进程命名空间。打开该选项的容器之间，以及容器和宿主机系统之间可以通过进程 ID 来相互访问和操作。

```yaml
pid: "host"
```

### `ports`

暴露端口信息。

使用宿主端口：容器端口 `(HOST:CONTAINER)` 格式，或者仅仅指定容器的端口（宿主将会随机选择端口）都可以。

```yaml
ports:
 - "3000"
 - "8000:8000"
 - "49100:22"
 - "127.0.0.1:8001:8001"
```

*注意：当使用 `HOST:CONTAINER` 格式来映射端口时，如果你使用的容器端口小于 60 并且没放到引号里，可能会得到错误结果，因为 `YAML` 会自动解析 `xx:yy` 这种数字格式为 60 进制。为避免出现这种问题，建议数字串都采用引号包括起来的字符串格式。*

### `secrets`

存储敏感数据，例如 `mysql` 服务密码。

```yaml
version: "3.1"
services:
mysql:
  image: mysql
  environment:
    MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password
  secrets:
    - db_root_password
    - my_other_secret
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```

### `security_opt`

指定容器模板标签（label）机制的默认属性（用户、角色、类型、级别等）。例如配置标签的用户名和角色名。

```yaml
security_opt:
    - label:user:USER
    - label:role:ROLE
```

### `stop_signal`

设置另一个信号来停止容器。在默认情况下使用的是 SIGTERM 停止容器。

```yaml
stop_signal: SIGUSR1
```

### `sysctls`

配置容器内核参数。

```yaml
sysctls:
  net.core.somaxconn: 1024
  net.ipv4.tcp_syncookies: 0
sysctls:
  - net.core.somaxconn=1024
  - net.ipv4.tcp_syncookies=0
```

### `ulimits`

指定容器的 ulimits 限制值。

例如，指定最大进程数为 65535，指定文件句柄数为 20000（软限制，应用可以随时修改，不能超过硬限制） 和 40000（系统硬限制，只能 root 用户提高）。

```yaml
  ulimits:
    nproc: 65535
    nofile:
      soft: 20000
      hard: 40000
```

### `volumes`

数据卷所挂载路径设置。可以设置为宿主机路径(`HOST:CONTAINER`)或者数据卷名称(`VOLUME:CONTAINER`)，并且可以设置访问模式 （`HOST:CONTAINER:ro`）。

该指令中路径支持相对路径。

```yaml
volumes:
 - /var/lib/mysql
 - cache/:/tmp/cache
 - ~/configs:/etc/configs/:ro
```

如果路径为数据卷名称，必须在文件中配置数据卷。

```yaml
version: "3"
services:
  my_src:
    image: mysql:8.0
    volumes:
      - mysql_data:/var/lib/mysql
volumes:
  mysql_data:  
```

### 其它指令

此外，还有包括 `domainname, entrypoint, hostname, ipc, mac_address, privileged, read_only, shm_size, restart, stdin_open, tty, user, working_dir` 等指令，基本跟 `docker run` 中对应参数的功能一致。

指定服务容器启动后执行的入口文件。

```yaml
entrypoint: /code/entrypoint.sh
```

指定容器中运行应用的用户名。

```yaml
user: nginx
```

指定容器中工作目录。

```yaml
working_dir: /code
```

指定容器中搜索域名、主机名、mac 地址等。

```yaml
domainname: your_website.com
hostname: test
mac_address: 08-00-27-00-0C-0A
```

允许容器中运行一些特权命令。

```yaml
privileged: true
```

指定容器退出后的重启策略为始终重启。该命令对保持服务始终运行十分有效，在生产环境中推荐配置为 `always` 或者 `unless-stopped`。

```yaml
restart: always
```

以只读模式挂载容器的 root 文件系统，意味着不能对容器内容进行修改。

```yaml
read_only: true
```

打开标准输入，可以接受外部输入。

```yaml
stdin_open: true
```

模拟一个伪终端。

```yaml
tty: true
```

### 读取变量

Compose 模板文件支持动态读取主机的系统环境变量和当前目录下的 `.env` 文件中的变量。

例如，下面的 Compose 文件将从运行它的环境中读取变量 `${MONGO_VERSION}` 的值，并写入执行的指令中。

```yaml
version: "3"
services:
db:
  image: "mongo:${MONGO_VERSION}"
```

如果执行 `MONGO_VERSION=3.2 docker-compose up` 则会启动一个 `mongo:3.2` 镜像的容器；如果执行 `MONGO_VERSION=2.8 docker-compose up` 则会启动一个 `mongo:2.8` 镜像的容器。

若当前目录存在 `.env` 文件，执行 `docker-compose` 命令时将从该文件中读取变量。

在当前目录新建 `.env` 文件并写入以下内容。

```bash
# 支持 # 号注释
MONGO_VERSION=3.6
```

执行 `docker-compose up` 则会启动一个 `mongo:3.6` 镜像的容器。

### 参考资料

* [官方文档](https://docs.docker.com/compose/compose-file/)
* [awesome-compose](https://github.com/docker/awesome-compose)


## 使用 Django

> 本小节内容适合 `Python` 开发人员阅读。

我们现在将使用 `Docker Compose` 配置并运行一个 `Django/PostgreSQL` 应用。

在一切工作开始前，需要先编辑好三个必要的文件。

第一步，因为应用将要运行在一个满足所有环境依赖的 Docker 容器里面，那么我们可以通过编辑 `Dockerfile` 文件来指定 Docker 容器要安装内容。内容如下：

```dockerfile
FROM python:3
ENV PYTHONUNBUFFERED 1
RUN mkdir /code
WORKDIR /code
COPY requirements.txt /code/
RUN pip install -r requirements.txt
COPY . /code/
```

以上内容指定应用将使用安装了 Python 以及必要依赖包的镜像。更多关于如何编写 `Dockerfile` 文件的信息可以查看 [ Dockerfile 使用](../image/dockerfile/README.md)。

第二步，在 `requirements.txt` 文件里面写明需要安装的具体依赖包名。

```bash
Django>=2.0,<3.0
psycopg2>=2.7,<3.0
```

第三步，`docker-compose.yml` 文件将把所有的东西关联起来。它描述了应用的构成（一个 web 服务和一个数据库）、使用的 Docker 镜像、镜像之间的连接、挂载到容器的卷，以及服务开放的端口。

```yaml
version: "3"
services:
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: 'postgres'
  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - "8000:8000"
```

查看 [`docker-compose.yml` 章节](compose_file.md) 了解更多详细的工作机制。

现在我们就可以使用 `docker-compose run` 命令启动一个 `Django` 应用了。

```bash
$ docker-compose run web django-admin startproject django_example .
```

由于 web 服务所使用的镜像并不存在，所以 Compose 会首先使用 `Dockerfile` 为 web 服务构建一个镜像，接着使用这个镜像在容器里运行 `django-admin startproject django_example` 指令。

这将在当前目录生成一个 `Django` 应用。

```bash
$ ls
Dockerfile       docker-compose.yml          django_example       manage.py       requirements.txt
```

如果你的系统是 Linux,记得更改文件权限。

```bash
$ sudo chown -R $USER:$USER .
```

首先，我们要为应用设置好数据库的连接信息。用以下内容替换 `django_example/settings.py` 文件中 `DATABASES = ...` 定义的节点内容。

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'postgres',
        'USER': 'postgres',
        'HOST': 'db',
        'PORT': 5432,
        'PASSWORD': 'postgres',
    }
}
```

这些信息是在 [postgres](https://hub.docker.com/_/postgres/) 镜像固定设置好的。然后，运行 `docker-compose up` ：

```bash
$ docker-compose up
django_db_1 is up-to-date
Creating django_web_1 ...
Creating django_web_1 ... done
Attaching to django_db_1, django_web_1
db_1   | The files belonging to this database system will be owned by user "postgres".
db_1   | This user must also own the server process.
db_1   |
db_1   | The database cluster will be initialized with locale "en_US.utf8".
db_1   | The default database encoding has accordingly been set to "UTF8".
db_1   | The default text search configuration will be set to "english".
web_1  | Performing system checks...
web_1  |
web_1  | System check identified no issues (0 silenced).
web_1  |
web_1  | November 23, 2017 - 06:21:19
web_1  | Django version 1.11.7, using settings 'django_example.settings'
web_1  | Starting development server at http://0.0.0.0:8000/
web_1  | Quit the server with CONTROL-C.
```

这个 `Django` 应用已经开始在你的 Docker 守护进程里监听着 `8000` 端口了。打开 `127.0.0.1:8000` 即可看到 `Django` 欢迎页面。

你还可以在 Docker 上运行其它的管理命令，例如对于同步数据库结构这种事，在运行完 `docker-compose up` 后，在另外一个终端进入文件夹运行以下命令即可：

```bash
$ docker-compose run web python manage.py syncdb
```

## 使用 Rails

> 本小节内容适合 `Ruby` 开发人员阅读。

我们现在将使用 `Compose` 配置并运行一个 `Rails/PostgreSQL` 应用。

在一切工作开始前，需要先设置好三个必要的文件。

首先，因为应用将要运行在一个满足所有环境依赖的 Docker 容器里面，那么我们可以通过编辑 `Dockerfile` 文件来指定 Docker 容器要安装内容。内容如下：

```docker
FROM ruby
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev
RUN mkdir /myapp
WORKDIR /myapp
ADD Gemfile /myapp/Gemfile
RUN bundle install
ADD . /myapp
```

以上内容指定应用将使用安装了 Ruby、Bundler 以及其依赖件的镜像。更多关于如何编写 Dockerfile 文件的信息可以查看 [Dockerfile 使用](../image/dockerfile/README.md)。

下一步，我们需要一个引导加载 Rails 的文件 `Gemfile` 。 等一会儿它还会被 `rails new` 命令覆盖重写。

```bash
source 'https://rubygems.org'
gem 'rails', '4.0.2'
```

最后，`docker-compose.yml` 文件才是最神奇的地方。 `docker-compose.yml` 文件将把所有的东西关联起来。它描述了应用的构成（一个 web 服务和一个数据库）、每个镜像的来源（数据库运行在使用预定义的 PostgreSQL 镜像，web 应用侧将从本地目录创建）、镜像之间的连接，以及服务开放的端口。

```yaml
version: "3"
services:
  db:
    image: postgres
    ports:
      - "5432"
  web:
    build: .
    command: bundle exec rackup -p 3000
    volumes:
      - .:/myapp
    ports:
      - "3000:3000"
```

所有文件就绪后，我们就可以通过使用 `docker-compose run` 命令生成应用的骨架了。

```bash
$ docker-compose run web rails new . --force --database=postgresql --skip-bundle
```

`Compose` 会先使用 `Dockerfile` 为 web 服务创建一个镜像，接着使用这个镜像在容器里运行 `rails new ` 和它之后的命令。一旦这个命令运行完后，应该就可以看一个崭新的应用已经生成了。

```bash
$ ls
Dockerfile   app          docker-compose.yml      tmp
Gemfile      bin          lib          vendor
Gemfile.lock condocker-compose       log
README.rdoc  condocker-compose.ru    public
Rakefile     db           test
```

在新的 `Gemfile` 文件去掉加载 `therubyracer` 的行的注释，这样我们便可以使用 Javascript 运行环境：

```bash
gem 'therubyracer', platforms: :ruby
```

现在我们已经有一个新的 `Gemfile` 文件，需要再重新创建镜像。（这个会步骤会改变 Dockerfile 文件本身，所以需要重建一次）。

```bash
$ docker-compose build
```

应用现在就可以启动了，但配置还未完成。Rails 默认读取的数据库目标是 `localhost` ，我们需要手动指定容器的 `db` 。同样的，还需要把用户名修改成和 postgres 镜像预定的一致。
打开最新生成的 `database.yml` 文件。用以下内容替换：

```bash
development: &default
  adapter: postgresql
  encoding: unicode
  database: postgres
  pool: 5
  username: postgres
  password:
  host: db
test:
  <<: *default
  database: myapp_test
```

现在就可以启动应用了。

```bash
$ docker-compose up
```

如果一切正常，你应该可以看到 PostgreSQL 的输出，几秒后可以看到这样的重复信息：

```bash
myapp_web_1 | [2014-01-17 17:16:29] INFO  WEBrick 1.3.1
myapp_web_1 | [2014-01-17 17:16:29] INFO  ruby 2.0.0 (2013-11-22) [x86_64-linux-gnu]
myapp_web_1 | [2014-01-17 17:16:29] INFO  WEBrick::HTTPServer#start: pid=1 port=3000
```

最后， 我们需要做的是创建数据库，打开另一个终端，运行：

```bash
$ docker-compose run web rake db:create
```

这个 web 应用已经开始在你的 docker 守护进程里面监听着 3000 端口了。


## 使用 WordPress

> 本小节内容适合 `PHP` 开发人员阅读。

`Compose` 可以很便捷的让 `Wordpress` 运行在一个独立的环境中。

### 创建空文件夹

假设新建一个名为 `wordpress` 的文件夹，然后进入这个文件夹。

### 创建 `docker-compose.yml` 文件

[`docker-compose.yml`](https://github.com/yeasy/blob/master/compose/demo/wordpress/docker-compose.yml) 文件将开启一个 `wordpress` 服务和一个独立的 `MySQL` 实例：

```yaml
version: "3"
services:
   db:
     image: mysql:8.0
     command:
      - --default_authentication_plugin=mysql_native_password
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci     
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress
   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
volumes:
  db_data:
```

### 构建并运行项目

运行 `docker-compose up -d` Compose 就会拉取镜像再创建我们所需要的镜像，然后启动 `wordpress` 和数据库容器。 接着浏览器访问 `127.0.0.1:8000` 端口就能看到 `WordPress` 安装界面了。

## 使用 compose 搭建 LNMP 环境

本项目的维护者 [khs1994](https://github.com/khs1994) 的开源项目 [khs1994-docker/lnmp](https://github.com/khs1994-docker/lnmp) 使用 Docker Compose 搭建了一套 LNMP 环境，各位开发者可以参考该项目在 Docker 或 Kubernetes 中运行 LNMP。

# Swarm mode
Docker 1.12 Swarm mode 已经内嵌入 Docker 引擎，成为了 docker 子命令 docker swarm。请注意与旧的 Docker Swarm 区分开来。

Swarm mode 内置 kv 存储功能，提供了众多的新特性，比如：具有容错能力的去中心化设计、内置服务发现、负载均衡、路由网格、动态伸缩、滚动更新、安全传输等。使得 Docker 原生的 Swarm 集群具备与 Mesos、Kubernetes 竞争的实力。

## 基本概念

`Swarm` 是使用 [`SwarmKit`](https://github.com/docker/swarmkit/) 构建的 Docker 引擎内置（原生）的集群管理和编排工具。

 使用 `Swarm` 集群之前需要了解以下几个概念。

### 节点

运行 Docker 的主机可以主动初始化一个 `Swarm` 集群或者加入一个已存在的 `Swarm` 集群，这样这个运行 Docker 的主机就成为一个 `Swarm` 集群的节点 (`node`) 。

节点分为 ***管理 (`manager`) 节点*** 和 ***工作 (`worker`) 节点***。

管理节点用于 `Swarm` 集群的管理，`docker swarm` 命令基本只能在管理节点执行（节点退出集群命令 `docker swarm leave` 可以在工作节点执行）。一个 `Swarm` 集群可以有多个管理节点，但只有一个管理节点可以成为 `leader`，`leader` 通过 `raft` 协议实现。

工作节点是任务执行节点，管理节点将服务 (`service`) 下发至工作节点执行。管理节点默认也作为工作节点。你也可以通过配置让服务只运行在管理节点。

来自 Docker 官网的这张图片形象的展示了集群中管理节点与工作节点的关系。

![](swarm-diagram.png)

### 服务和任务

***任务 （`Task`）***是 `Swarm` 中的最小的调度单位，目前来说就是一个单一的容器。

***服务 （`Services`）*** 是指一组任务的集合，服务定义了任务的属性。服务有2种模式：

* `replicated services` 按照一定规则在各个工作节点上运行指定个数的任务。

* `global services` 每个工作节点上运行一个任务

2种模式通过 `docker service create` 的 `--mode` 参数指定。

来自 Docker 官网的这张图片形象的展示了容器、任务、服务的关系。

![](services-diagram-16509012472675.png)

## 创建 Swarm 集群

阅读 [基本概念](overview.md) 一节我们知道 `Swarm` 集群由 **管理节点** 和 **工作节点** 组成。本节我们来创建一个包含一个管理节点和两个工作节点的最小 `Swarm` 集群。

### 初始化集群

在已经安装好 Docker 的主机上执行如下命令：

```bash
$ docker swarm init --advertise-addr 192.168.99.100
Swarm initialized: current node (dxn1zf6l61qsb1josjja83ngz) is now a manager.
To add a worker to this swarm, run the following command:
    docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377
To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

如果你的 Docker 主机有多个网卡，拥有多个 IP，必须使用 `--advertise-addr` 指定 IP。

> 执行 `docker swarm init` 命令的节点自动成为管理节点。

### 增加工作节点

上一步我们初始化了一个 `Swarm` 集群，拥有了一个管理节点，下面我们继续在两个 Docker 主机中分别执行如下命令，创建工作节点并加入到集群中。

```bash
$ docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377
This node joined a swarm as a worker.
```

### 查看集群

经过上边的两步，我们已经拥有了一个最小的 `Swarm` 集群，包含一个管理节点和两个工作节点。

在管理节点使用 `docker node ls` 查看集群。

```bash
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
03g1y59jwfg7cf99w4lt0f662    worker2   Ready   Active
9j68exjopxe7wfl6yuxml7a7j    worker1   Ready   Active
dxn1zf6l61qsb1josjja83ngz *  manager   Ready   Active        Leader
```

## 部署服务

我们使用 `docker service` 命令来管理 `Swarm` 集群中的服务，该命令只能在管理节点运行。

### 新建服务

现在我们在上一节创建的 `Swarm` 集群中运行一个名为 `nginx` 服务。

```bash
$ docker service create --replicas 3 -p 80:80 --name nginx nginx:1.13.7-alpine
```

现在我们使用浏览器，输入任意节点 IP ，即可看到 nginx 默认页面。

### 查看服务

使用 `docker service ls` 来查看当前 `Swarm` 集群运行的服务。

```bash
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                 PORTS
kc57xffvhul5        nginx               replicated          3/3                 nginx:1.13.7-alpine   *:80->80/tcp
```

使用 `docker service ps` 来查看某个服务的详情。

```bash
$ docker service ps nginx
ID                  NAME                IMAGE                 NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
pjfzd39buzlt        nginx.1             nginx:1.13.7-alpine   swarm2              Running             Running about a minute ago
hy9eeivdxlaa        nginx.2             nginx:1.13.7-alpine   swarm1              Running             Running about a minute ago
36wmpiv7gmfo        nginx.3             nginx:1.13.7-alpine   swarm3              Running             Running about a minute ago
```

使用 `docker service logs` 来查看某个服务的日志。

```bash
$ docker service logs nginx
nginx.3.36wmpiv7gmfo@swarm3    | 10.255.0.4 - - [25/Nov/2017:02:10:30 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:58.0) Gecko/20100101 Firefox/58.0" "-"
nginx.3.36wmpiv7gmfo@swarm3    | 10.255.0.4 - - [25/Nov/2017:02:10:30 +0000] "GET /favicon.ico HTTP/1.1" 404 169 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:58.0) Gecko/20100101 Firefox/58.0" "-"
nginx.3.36wmpiv7gmfo@swarm3    | 2017/11/25 02:10:30 [error] 5#5: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 10.255.0.4, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "192.168.99.102"
nginx.1.pjfzd39buzlt@swarm2    | 10.255.0.2 - - [25/Nov/2017:02:10:26 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:58.0) Gecko/20100101 Firefox/58.0" "-"
nginx.1.pjfzd39buzlt@swarm2    | 10.255.0.2 - - [25/Nov/2017:02:10:27 +0000] "GET /favicon.ico HTTP/1.1" 404 169 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:58.0) Gecko/20100101 Firefox/58.0" "-"
nginx.1.pjfzd39buzlt@swarm2    | 2017/11/25 02:10:27 [error] 5#5: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 10.255.0.2, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "192.168.99.101"
```

### 服务伸缩

我们可以使用 `docker service scale` 对一个服务运行的容器数量进行伸缩。

当业务处于高峰期时，我们需要扩展服务运行的容器数量。

```bash
$ docker service scale nginx=5
```

当业务平稳时，我们需要减少服务运行的容器数量。

```bash
$ docker service scale nginx=2
```

### 删除服务

使用 `docker service rm` 来从 `Swarm` 集群移除某个服务。

```bash
$ docker service rm nginx
```

## 在 Swarm 集群中使用 compose 文件

正如之前使用 `docker-compose.yml` 来一次配置、启动多个容器，在 `Swarm` 集群中也可以使用 `compose` 文件 （`docker-compose.yml`） 来配置、启动多个服务。

上一节中，我们使用 `docker service create` 一次只能部署一个服务，使用 `docker-compose.yml` 我们可以一次启动多个关联的服务。

我们以在 `Swarm` 集群中部署 `WordPress` 为例进行说明。

```yaml
version: "3"
services:
  wordpress:
    image: wordpress
    ports:
      - 80:80
    networks:
      - overlay
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
    deploy:
      mode: replicated
      replicas: 3
  db:
    image: mysql
    networks:
       - overlay
    volumes:
      - db-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    deploy:
      placement:
        constraints: [node.role == manager]
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
volumes:
  db-data:
networks:
  overlay:
```

在 `Swarm` 集群管理节点新建该文件，其中的 `visualizer` 服务提供一个可视化页面，我们可以从浏览器中很直观的查看集群中各个服务的运行节点。

在 `Swarm` 集群中使用 `docker-compose.yml` 我们用 `docker stack` 命令，下面我们对该命令进行详细讲解。

### 部署服务

部署服务使用 `docker stack deploy`，其中 `-c` 参数指定 compose 文件名。

```bash
$ docker stack deploy -c docker-compose.yml wordpress
```

现在我们打开浏览器输入 `任一节点IP:8080` 即可看到各节点运行状态。如下图所示：

![img](wordpress.png)

在浏览器新的标签页输入 `任一节点IP` 即可看到 `WordPress` 安装界面，安装完成之后，输入 `任一节点IP` 即可看到 `WordPress` 页面。

### 查看服务

```bash
$ docker stack ls
NAME                SERVICES
wordpress           3
```

### 移除服务

要移除服务，使用 `docker stack down`

```bash
$ docker stack down wordpress
Removing service wordpress_db
Removing service wordpress_visualizer
Removing service wordpress_wordpress
Removing network wordpress_overlay
Removing network wordpress_default
```

该命令不会移除服务所使用的 `数据卷`，如果你想移除数据卷请使用 `docker volume rm`

## 在 Swarm 集群中管理敏感数据

在动态的、大规模的分布式集群上，管理和分发 `密码`、`证书` 等敏感信息是极其重要的工作。传统的密钥分发方式（如密钥放入镜像中，设置环境变量，volume 动态挂载等）都存在着潜在的巨大的安全风险。

Docker 目前已经提供了 `secrets` 管理功能，用户可以在 Swarm 集群中安全地管理密码、密钥证书等敏感数据，并允许在多个 Docker 容器实例之间共享访问指定的敏感数据。

>注意： `secret` 也可以在 `Docker Compose` 中使用。

我们可以用 `docker secret` 命令来管理敏感信息。接下来我们在上面章节中创建好的 Swarm 集群中介绍该命令的使用。

这里我们以在 Swarm 集群中部署 `mysql` 和 `wordpress` 服务为例。

### 创建 secret

我们使用 `docker secret create` 命令以管道符的形式创建 `secret`

```bash
$ openssl rand -base64 20 | docker secret create mysql_password -
$ openssl rand -base64 20 | docker secret create mysql_root_password -
```

### 查看 secret

使用 `docker secret ls` 命令来查看 `secret`

```bash
$ docker secret ls
ID                          NAME                  CREATED             UPDATED
l1vinzevzhj4goakjap5ya409   mysql_password        41 seconds ago      41 seconds ago
yvsczlx9votfw3l0nz5rlidig   mysql_root_password   12 seconds ago      12 seconds ago
```

### 创建 MySQL 服务

创建服务相关命令已经在前边章节进行了介绍，这里直接列出命令。

```bash
$ docker network create -d overlay mysql_private
$ docker service create \
     --name mysql \
     --replicas 1 \
     --network mysql_private \
     --mount type=volume,source=mydata,destination=/var/lib/mysql \
     --secret source=mysql_root_password,target=mysql_root_password \
     --secret source=mysql_password,target=mysql_password \
     -e MYSQL_ROOT_PASSWORD_FILE="/run/secrets/mysql_root_password" \
     -e MYSQL_PASSWORD_FILE="/run/secrets/mysql_password" \
     -e MYSQL_USER="wordpress" \
     -e MYSQL_DATABASE="wordpress" \
     mysql:latest
```

如果你没有在 `target` 中显式的指定路径时，`secret` 默认通过 `tmpfs` 文件系统挂载到容器的 `/run/secrets` 目录中。

```bash
$ docker service create \
     --name wordpress \
     --replicas 1 \
     --network mysql_private \
     --publish target=30000,port=80 \
     --mount type=volume,source=wpdata,destination=/var/www/html \
     --secret source=mysql_password,target=wp_db_password,mode=0444 \
     -e WORDPRESS_DB_USER="wordpress" \
     -e WORDPRESS_DB_PASSWORD_FILE="/run/secrets/wp_db_password" \
     -e WORDPRESS_DB_HOST="mysql:3306" \
     -e WORDPRESS_DB_NAME="wordpress" \
     wordpress:latest
```

查看服务

```bash
$ docker service ls
ID            NAME   MODE        REPLICAS  IMAGE
wvnh0siktqr3  mysql      replicated  1/1       mysql:latest
nzt5xzae4n62  wordpress  replicated  1/1       wordpress:latest
```

现在浏览器访问 `IP:30000`，即可开始 `WordPress` 的安装与使用。

通过以上方法，我们没有像以前通过设置环境变量来设置 MySQL 密码， 而是采用 `docker secret` 来设置密码，防范了密码泄露的风险。


## 在 Swarm 集群中管理配置数据

在动态的、大规模的分布式集群上，管理和分发配置文件也是很重要的工作。传统的配置文件分发方式（如配置文件放入镜像中，设置环境变量，volume 动态挂载等）都降低了镜像的通用性。

在 Docker 17.06 以上版本中，Docker 新增了 `docker config` 子命令来管理集群中的配置信息，以后你无需将配置文件放入镜像或挂载到容器中就可实现对服务的配置。

>注意：`config` 仅能在 Swarm 集群中使用。

这里我们以在 Swarm 集群中部署 `redis` 服务为例。

### 创建 config

新建 `redis.conf` 文件

```bash
port 6380
```

此项配置 Redis 监听 `6380` 端口

我们使用 `docker config create` 命令创建 `config`

```bash
$ docker config create redis.conf redis.conf
```

### 查看 config

使用 `docker config ls` 命令来查看 `config`

```bash
$ docker config ls
ID                          NAME                CREATED             UPDATED
yod8fx8iiqtoo84jgwadp86yk   redis.conf          4 seconds ago       4 seconds ago
```

### 创建 redis 服务

```bash
$ docker service create \
     --name redis \
     # --config source=redis.conf,target=/etc/redis.conf \
     --config redis.conf \
     -p 6379:6380 \
     redis:latest \
     redis-server /redis.conf
```

如果你没有在 `target` 中显式的指定路径时，默认的 `redis.conf` 以 `tmpfs` 文件系统挂载到容器的 `/config.conf`。

经过测试，redis 可以正常使用。

以前我们通过监听主机目录来配置 Redis，就需要在集群的每个节点放置该文件，如果采用 `docker config` 来管理服务的配置信息，我们只需在集群中的管理节点创建 `config`，当部署服务时，集群会自动的将配置文件分发到运行服务的各个节点中，大大降低了配置信息的管理和分发难度。


## Swarm mode 与滚动升级

在 [部署服务](deploy.md) 一节中我们使用 `nginx:1.13.7-alpine` 镜像部署了一个名为 `nginx` 的服务。

现在我们想要将 `NGINX` 版本升级到 `1.13.12`，那么在 Swarm mode 中如何升级服务呢？

你可能会想到，先停止原来的服务，再使用新镜像部署一个服务，不就完成服务的 “升级” 了吗。

这样做的弊端很明显，如果新部署的服务出现问题，原来的服务删除之后，很难恢复，那么在 Swarm mode 中到底该如何对服务进行滚动升级呢？

答案就是使用 `docker service update` 命令。

```bash
$ docker service update \
    --image nginx:1.13.12-alpine \
    nginx
```

以上命令使用 `--image` 选项更新了服务的镜像。当然我们也可以使用 `docker service update` 更新任意的配置。

`--secret-add` 选项可以增加一个密钥

`--secret-rm` 选项可以删除一个密钥

更多选项可以通过 `docker service update -h` 命令查看。

### 服务回退

现在假设我们发现 `nginx` 服务的镜像升级到 `nginx:1.13.12-alpine` 出现了一些问题，我们可以使用命令一键回退。

```bash
$ docker service rollback nginx
```

现在使用 `docker service ps` 命令查看 `nginx` 服务详情。

```bash
$ docker service ps nginx
ID                  NAME                IMAGE                  NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
rt677gop9d4x        nginx.1             nginx:1.13.7-alpine   VM-20-83-debian     Running             Running about a minute ago
d9pw13v59d00         \_ nginx.1         nginx:1.13.12-alpine  VM-20-83-debian     Shutdown            Shutdown 2 minutes ago
i7ynkbg6ybq5         \_ nginx.1         nginx:1.13.7-alpine   VM-20-83-debian     Shutdown            Shutdown 2 minutes ago
```

结果的输出详细记录了服务的部署、滚动升级、回退的过程。

# 安全

评估 Docker 的安全性时，主要考虑三个方面:

- 由内核的命名空间和控制组机制提供的容器内在安全
- Docker 程序（特别是服务端）本身的抗攻击性
- 内核安全性的加强机制对容器安全性的影响

## 命名空间（Namespaces）

Docker 容器和 LXC 容器很相似，所提供的安全特性也差不多。当用 `docker run` 启动一个容器时，在后台 Docker 为容器创建了一个独立的命名空间和控制组集合。

*命名空间* 提供了最基础也是最直接的隔离，在容器中运行的进程不会被运行在主机上的进程和其它容器发现和作用。

每个容器都有自己独有的网络栈，意味着它们不能访问其他容器的 sockets 或接口。不过，如果主机系统上做了相应的设置，容器可以像跟主机交互一样的和其他容器交互。当指定公共端口或使用 links 来连接 2 个容器时，容器就可以相互通信了（可以根据配置来限制通信的策略）。

从网络架构的角度来看，所有的容器通过本地主机的网桥接口相互通信，就像物理机器通过物理交换机通信一样。

那么，内核中实现命名空间和私有网络的代码是否足够成熟？

内核命名空间从 2.6.15 版本（2008 年 7 月发布）之后被引入，数年间，这些机制的可靠性在诸多大型生产系统中被实践验证。

实际上，命名空间的想法和设计提出的时间要更早，最初是为了在内核中引入一种机制来实现 [OpenVZ](https://en.wikipedia.org/wiki/OpenVZ) 的特性。
而 OpenVZ 项目早在 2005 年就发布了，其设计和实现都已经十分成熟。

## 控制组（Control groups）

控制组是 Linux 容器机制的另外一个关键组件，负责实现资源的审计和限制。

它提供了很多有用的特性；以及确保各个容器可以公平地分享主机的内存、CPU、磁盘 IO 等资源；当然，更重要的是，控制组确保了当容器内的资源使用产生压力时不会连累主机系统。

尽管控制组不负责隔离容器之间相互访问、处理数据和进程，它在防止拒绝服务（DDOS）攻击方面是必不可少的。尤其是在多用户的平台（比如公有或私有的 PaaS）上，控制组十分重要。例如，当某些应用程序表现异常的时候，可以保证一致地正常运行和性能。

控制组机制始于 2006 年，内核从 2.6.24 版本开始被引入。

## Docker服务端的防护

运行一个容器或应用程序的核心是通过 Docker 服务端。Docker 服务的运行目前需要 root 权限，因此其安全性十分关键。

首先，确保只有可信的用户才可以访问 Docker 服务。Docker 允许用户在主机和容器间共享文件夹，同时不需要限制容器的访问权限，这就容易让容器突破资源限制。例如，恶意用户启动容器的时候将主机的根目录`/`映射到容器的 `/host` 目录中，那么容器理论上就可以对主机的文件系统进行任意修改了。这听起来很疯狂？但是事实上几乎所有虚拟化系统都允许类似的资源共享，而没法禁止用户共享主机根文件系统到虚拟机系统。

这将会造成很严重的安全后果。因此，当提供容器创建服务时（例如通过一个 web 服务器），要更加注意进行参数的安全检查，防止恶意的用户用特定参数来创建一些破坏性的容器。

为了加强对服务端的保护，Docker 的 REST API（客户端用来跟服务端通信）在 0.5.2 之后使用本地的 Unix 套接字机制替代了原先绑定在 127.0.0.1 上的 TCP 套接字，因为后者容易遭受跨站脚本攻击。现在用户使用 Unix 权限检查来加强套接字的访问安全。

用户仍可以利用 HTTP 提供 REST API 访问。建议使用安全机制，确保只有可信的网络或 VPN，或证书保护机制（例如受保护的 stunnel 和 ssl 认证）下的访问可以进行。此外，还可以使用 [ HTTPS 和证书](https://docs.docker.com/engine/security/https/) 来加强保护。

最近改进的 Linux 命名空间机制将可以实现使用非 root 用户来运行全功能的容器。这将从根本上解决了容器和主机之间共享文件系统而引起的安全问题。

终极目标是改进 2 个重要的安全特性：
* 将容器的 root 用户 [映射到本地主机上的非 root 用户](https://docs.docker.com/engine/security/userns-remap/)，减轻容器和主机之间因权限提升而引起的安全问题；
* 允许 Docker 服务端在 [非 root 权限(rootless 模式)](https://docs.docker.com/engine/security/rootless/) 下运行，利用安全可靠的子进程来代理执行需要特权权限的操作。这些子进程将只允许在限定范围内进行操作，例如仅仅负责虚拟网络设定或文件系统管理、配置操作等。

最后，建议采用专用的服务器来运行 Docker 和相关的管理服务（例如管理服务比如 ssh 监控和进程监控、管理工具 nrpe、collectd 等）。其它的业务服务都放到容器中去运行。

## 内核能力机制

[能力机制（Capability）](https://man7.org/linux/man-pages/man7/capabilities.7.html) 是 Linux 内核一个强大的特性，可以提供细粒度的权限访问控制。
Linux 内核自 2.2 版本起就支持能力机制，它将权限划分为更加细粒度的操作能力，既可以作用在进程上，也可以作用在文件上。

例如，一个 Web 服务进程只需要绑定一个低于 1024 的端口的权限，并不需要 root 权限。那么它只需要被授权 `net_bind_service` 能力即可。此外，还有很多其他的类似能力来避免进程获取 root 权限。

默认情况下，Docker 启动的容器被严格限制只允许使用内核的一部分能力。

使用能力机制对加强 Docker 容器的安全有很多好处。通常，在服务器上会运行一堆需要特权权限的进程，包括有 ssh、cron、syslogd、硬件管理工具模块（例如负载模块）、网络配置工具等等。容器跟这些进程是不同的，因为几乎所有的特权进程都由容器以外的支持系统来进行管理。
* ssh 访问被主机上ssh服务来管理；
* cron 通常应该作为用户进程执行，权限交给使用它服务的应用来处理；
* 日志系统可由 Docker 或第三方服务管理；
* 硬件管理无关紧要，容器中也就无需执行 udevd 以及类似服务；
* 网络管理也都在主机上设置，除非特殊需求，容器不需要对网络进行配置。

从上面的例子可以看出，大部分情况下，容器并不需要“真正的” root 权限，容器只需要少数的能力即可。为了加强安全，容器可以禁用一些没必要的权限。
* 完全禁止任何 mount 操作；
* 禁止直接访问本地主机的套接字；
* 禁止访问一些文件系统的操作，比如创建新的设备、修改文件属性等；
* 禁止模块加载。

这样，就算攻击者在容器中取得了 root 权限，也不能获得本地主机的较高权限，能进行的破坏也有限。

默认情况下，Docker采用 [白名单](https://github.com/moby/moby/blob/master/oci/caps/defaults.go) 机制，禁用必需功能之外的其它权限。
当然，用户也可以根据自身需求来为 Docker 容器启用额外的权限。

## 其它安全特性

除了能力机制之外，还可以利用一些现有的安全机制来增强使用 Docker 的安全性，例如 TOMOYO, AppArmor, Seccomp, SELinux, GRSEC 等。

Docker 当前默认只启用了能力机制。用户可以采用多种方案来加强 Docker 主机的安全，例如：
* 在内核中启用 GRSEC 和 PAX，这将增加很多编译和运行时的安全检查；通过地址随机化避免恶意探测等。并且，启用该特性不需要 Docker 进行任何配置。
* 使用一些有增强安全特性的容器模板，比如带 AppArmor 的模板和 Redhat 带 SELinux 策略的模板。这些模板提供了额外的安全特性。
* 用户可以自定义访问控制机制来定制安全策略。

跟其它添加到 Docker 容器的第三方工具一样（比如网络拓扑和文件系统共享），有很多类似的机制，在不改变 Docker 内核情况下就可以加固现有的容器。

## 总结

总体来看，Docker 容器还是十分安全的，特别是在容器内不使用 root 权限来运行进程的话。

另外，用户可以使用现有工具，比如 [Apparmor](https://docs.docker.com/engine/security/apparmor/), [Seccomp](https://docs.docker.com/engine/security/seccomp/), SELinux, GRSEC 来增强安全性；甚至自己在内核中实现更复杂的安全机制。

# 底层实现
Docker 底层的核心技术包括 Linux 上的命名空间（Namespaces）、控制组（Control groups）、Union 文件系统（Union file systems）和容器格式（Container format）。

我们知道，传统的虚拟机通过在宿主主机中运行 hypervisor 来模拟一整套完整的硬件环境提供给虚拟机的操作系统。虚拟机系统看到的环境是可限制的，也是彼此隔离的。 这种直接的做法实现了对资源最完整的封装，但很多时候往往意味着系统资源的浪费。 例如，以宿主机和虚拟机系统都为 Linux 系统为例，虚拟机中运行的应用其实可以利用宿主机系统中的运行环境。

我们知道，在操作系统中，包括内核、文件系统、网络、PID、UID、IPC、内存、硬盘、CPU 等等，所有的资源都是应用进程直接共享的。 要想实现虚拟化，除了要实现对内存、CPU、网络IO、硬盘IO、存储空间等的限制外，还要实现文件系统、网络、PID、UID、IPC等等的相互隔离。 前者相对容易实现一些，后者则需要宿主机系统的深入支持。

随着 Linux 系统对于命名空间功能的完善实现，程序员已经可以实现上面的所有需求，让某些进程在彼此隔离的命名空间中运行。大家虽然都共用一个内核和某些运行时环境（例如一些系统命令和系统库），但是彼此却看不到，都以为系统中只有自己的存在。**这种机制就是容器（Container），利用命名空间来做权限的隔离控制，利用 cgroups 来做资源分配。**

## 基本架构

Docker 采用了 `C/S` 架构，包括客户端和服务端。Docker 守护进程 （`Daemon`）作为服务端接受来自客户端的请求，并处理这些请求（创建、运行、分发容器）。

客户端和服务端既可以运行在一个机器上，也可通过 `socket` 或者 `RESTful API` 来进行通信。

Docker 守护进程一般在宿主主机后台运行，等待接收来自客户端的消息。

Docker 客户端则为用户提供一系列可执行命令，用户用这些命令实现跟 Docker 守护进程交互。

## 命名空间（Namespaces）

命名空间是 Linux 内核一个强大的特性。每个容器都有自己单独的命名空间，运行在其中的应用都像是在独立的操作系统中运行一样。命名空间保证了容器之间彼此互不影响。

### pid 命名空间
不同用户的进程就是通过 pid 命名空间隔离开的，且不同命名空间中可以有相同 pid。所有的 LXC 进程在 Docker 中的父进程为 Docker 进程，每个 LXC 进程具有不同的命名空间。同时由于允许嵌套，因此可以很方便的实现嵌套的 Docker 容器。

### net 命名空间
有了 pid 命名空间，每个命名空间中的 pid 能够相互隔离，但是网络端口还是共享 host 的端口。网络隔离是通过 net 命名空间实现的， 每个 net 命名空间有独立的 网络设备，IP 地址，路由表，`/proc/net` 目录。这样每个容器的网络就能隔离开来。Docker 默认采用 veth 的方式，将容器中的虚拟网卡同 host 上的一 个Docker 网桥 docker0 连接在一起。

### ipc 命名空间
容器中进程交互还是采用了 Linux 常见的进程间交互方法(interprocess communication - IPC)， 包括信号量、消息队列和共享内存等。然而与 VM 不同的是，容器的进程间交互实际上还是 host 上具有相同 pid 命名空间中的进程间交互，因此需要在 IPC 资源申请时加入命名空间信息，每个 IPC 资源有一个唯一的 32 位 id。

### mnt 命名空间
类似 chroot，将一个进程放到一个特定的目录执行。mnt 命名空间允许不同命名空间的进程看到的文件结构不同，这样每个命名空间 中的进程所看到的文件目录就被隔离开了。与 chroot 不同，每个命名空间中的容器在 `/proc/mounts` 的信息只包含所在命名空间的 mount point。

### uts 命名空间
UTS("UNIX Time-sharing System") 命名空间允许每个容器拥有独立的 hostname 和 domain name， 使其在网络上可以被视作一个独立的节点而非 主机上的一个进程。

### user 命名空间
每个容器可以有不同的用户和组 id， 也就是说可以在容器内用容器内部的用户执行程序而非主机上的用户。

*注：更多关于 Linux 上命名空间的信息，请阅读 [这篇文章](https://blog.scottlowe.org/2013/09/04/introducing-linux-network-namespaces/)。

## 控制组

控制组（[cgroups](https://en.wikipedia.org/wiki/Cgroups)）是 Linux 内核的一个特性，主要用来对共享资源进行隔离、限制、审计等。只有能控制分配到容器的资源，才能避免当多个容器同时运行时的对系统资源的竞争。

控制组技术最早是由 Google 的程序员在 2006 年提出，Linux 内核自 2.6.24 开始支持。

控制组可以提供对容器的内存、CPU、磁盘 IO 等资源的限制和审计管理。

## 联合文件系统

联合文件系统（[UnionFS](https://en.wikipedia.org/wiki/UnionFS)）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。

联合文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

另外，不同 Docker 容器就可以共享一些基础的文件系统层，同时再加上自己独有的改动层，大大提高了存储的效率。

Docker 中使用的 AUFS（Advanced Multi-Layered Unification Filesystem）就是一种联合文件系统。 `AUFS` 支持为每一个成员目录（类似 Git 的分支）设定只读（readonly）、读写（readwrite）和写出（whiteout-able）权限, 同时 `AUFS` 里有一个类似分层的概念, 对只读权限的分支可以逻辑上进行增量地修改(不影响只读部分的)。

Docker 目前支持的联合文件系统包括 `OverlayFS`, `AUFS`, `Btrfs`, `VFS`, `ZFS` 和 `Device Mapper`。

各 Linux 发行版 Docker 推荐使用的存储驱动如下表。

|Linux 发行版 |	Docker 推荐使用的存储驱动 |
| :--        | :--                     |
|Docker on Ubuntu |	`overlay2` (16.04 +) |
|Docker on Debian |	`overlay2` (Debian Stretch), `aufs`, `devicemapper` |
|Docker on CentOS |	`overlay2`  |
|Docker on Fedora |	`overlay2`  |

**在可能的情况下，[推荐](https://docs.docker.com/storage/storagedriver/select-storage-driver/) 使用 `overlay2` 存储驱动，`overlay2` 是目前 Docker 默认的存储驱动，以前则是 `aufs`。**你可以通过配置来使用以上提到的其他类型的存储驱动。

## 容器格式

**最初，Docker 采用了 `LXC` 中的容器格式。从 0.7 版本以后开始去除 LXC，转而使用自行开发的 [libcontainer](https://github.com/docker/libcontainer)，从 1.11 开始，则进一步演进为使用 [runC](https://github.com/opencontainers/runc) 和 [containerd](https://github.com/containerd/containerd)。**


## Docker 网络实现

Docker 的网络实现其实就是利用了 Linux 上的 *网络命名空间* 和 *虚拟网络设备*（特别是 veth pair）。

首先，要实现网络通信，机器需要至少一个网络接口（物理接口或虚拟接口）来收发数据包；此外，如果不同子网之间要进行通信，需要路由机制。

Docker 中的网络接口默认都是虚拟的接口。虚拟接口的优势之一是转发效率较高。
Linux 通过在内核中进行数据复制来实现虚拟接口之间的数据转发，发送接口的发送缓存中的数据包被直接复制到接收接口的接收缓存中。对于本地系统和容器内系统看来就像是一个正常的以太网卡，只是它不需要真正同外部网络设备通信，速度要快很多。

Docker 容器网络就利用了这项技术。它在本地主机和容器内分别创建一个虚拟接口，并让它们彼此连通（这样的一对接口叫做 `veth pair`）。

### 创建网络参数
Docker 创建一个容器的时候，会执行如下操作：
* 创建一对虚拟接口，分别放到本地主机和新容器中；
* 本地主机一端桥接到默认的 docker0 或指定网桥上，并具有一个唯一的名字，如 veth65f9；
* 容器一端放到新容器中，并修改名字作为 eth0，这个接口只在容器的命名空间可见；
* 从网桥可用地址段中获取一个空闲地址分配给容器的 eth0，并配置默认路由到桥接网卡 veth65f9。

完成这些之后，容器就可以使用 eth0 虚拟网卡来连接其他容器和其他网络。

可以在 `docker run` 的时候通过 `--net` 参数来指定容器的网络配置，有4个可选值：
* `--net=bridge` 这个是默认值，连接到默认的网桥。
* `--net=host` 告诉 Docker 不要将容器网络放到隔离的命名空间中，即不要容器化容器内的网络。此时容器使用本地主机的网络，它拥有完全的本地主机接口访问权限。容器进程可以跟主机其它 root 进程一样可以打开低范围的端口，可以访问本地网络服务比如 D-bus，还可以让容器做一些影响整个主机系统的事情，比如重启主机。因此使用这个选项的时候要非常小心。**如果进一步的使用 `--privileged=true`，容器会被允许直接配置主机的网络堆栈。**
* `--net=container:NAME_or_ID` 让 Docker 将新建容器的进程放到一个已存在容器的网络栈中，新容器进程有自己的文件系统、进程列表和资源限制，但会和已存在的容器共享 IP 地址和端口等网络资源，两者进程可以直接通过 `lo` 环回接口通信。
* `--net=none` 让 Docker 将新容器放到隔离的网络栈中，但是不进行网络配置。之后，用户可以自己进行配置。

### 网络配置细节
用户使用 `--net=none` 后，可以自行配置网络，让容器达到跟平常一样具有访问网络的权限。通过这个过程，可以了解 Docker 配置网络的细节。

首先，启动一个 `/bin/bash` 容器，指定 `--net=none` 参数。
```bash
$ docker run -i -t --rm --net=none base /bin/bash
root@63f36fc01b5f:/#
```
在本地主机查找容器的进程 id，并为它创建网络命名空间。
```bash
$ docker inspect -f '{{.State.Pid}}' 63f36fc01b5f
2778
$ pid=2778
$ sudo mkdir -p /var/run/netns
$ sudo ln -s /proc/$pid/ns/net /var/run/netns/$pid
```
检查桥接网卡的 IP 和子网掩码信息。
```bash
$ ip addr show docker0
21: docker0: ...
inet 172.17.42.1/16 scope global docker0
...
```
创建一对 “veth pair” 接口 A 和 B，绑定 A 到网桥 `docker0`，并启用它
```bash
$ sudo ip link add A type veth peer name B
$ sudo brctl addif docker0 A
$ sudo ip link set A up
```
将B放到容器的网络命名空间，命名为 eth0，启动它并配置一个可用 IP（桥接网段）和默认网关。
```bash
$ sudo ip link set B netns $pid
$ sudo ip netns exec $pid ip link set dev B name eth0
$ sudo ip netns exec $pid ip link set eth0 up
$ sudo ip netns exec $pid ip addr add 172.17.42.99/16 dev eth0
$ sudo ip netns exec $pid ip route add default via 172.17.42.1
```
以上，就是 Docker 配置网络的具体过程。

当容器结束后，Docker 会清空容器，容器内的 eth0 会随网络命名空间一起被清除，A 接口也被自动从 `docker0` 卸载。

此外，用户可以使用 `ip netns exec` 命令来在指定网络命名空间中进行配置，从而配置容器内的网络。

# Docker Buildx
Docker Buildx 是一个 docker CLI 插件，其扩展了 docker 命令，支持 Moby BuildKit 提供的功能。提供了与 docker build 相同的用户体验，并增加了许多新功能。

> 该功能仅适用于 Docker v19.03+ 版本

## 使用 `BuildKit` 构建镜像

**BuildKit** 是下一代的镜像构建组件，在 https://github.com/moby/buildkit 开源。

> 注意：如果您的镜像构建使用的是云服务商提供的镜像构建服务（腾讯云容器服务、阿里云容器服务等），由于上述服务提供商的 Docker 版本低于 18.09，BuildKit 无法使用，将造成镜像构建失败。建议使用 BuildKit 构建镜像时使用一个新的 Dockerfile 文件（例如 Dockerfile.buildkit）

目前，Docker Hub 自动构建已经支持 buildkit，具体请参考 https://github.com/docker-practice/docker-hub-buildx

### `Dockerfile` 新增指令详解

启用 `BuildKit` 之后，我们可以使用下面几个新的 `Dockerfile` 指令来加快镜像构建。

#### `RUN --mount=type=cache`

目前，几乎所有的程序都会使用依赖管理工具，例如 `Go` 中的 `go mod`、`Node.js` 中的 `npm` 等等，当我们构建一个镜像时，往往会重复的从互联网中获取依赖包，难以缓存，大大降低了镜像的构建效率。

例如一个前端工程需要用到 `npm`：

```dockerfile
FROM node:alpine as builder
WORKDIR /app
COPY package.json /app/
RUN npm i --registry=https://registry.npm.taobao.org \
        && rm -rf ~/.npm
COPY src /app/src
RUN npm run build
FROM nginx:alpine
COPY --from=builder /app/dist /app/dist
```

使用多阶段构建，构建的镜像中只包含了目标文件夹 `dist`，但仍然存在一些问题，当 `package.json` 文件变动时，`RUN npm i && rm -rf ~/.npm` 这一层会重新执行，变更多次后，生成了大量的中间层镜像。

为解决这个问题，进一步的我们可以设想一个类似 **数据卷** 的功能，在镜像构建时把 `node_modules` 文件夹挂载上去，在构建完成后，这个 `node_modules` 文件夹会自动卸载，实际的镜像中并不包含 `node_modules` 这个文件夹，这样我们就省去了每次获取依赖的时间，大大增加了镜像构建效率，同时也避免了生成了大量的中间层镜像。

`BuildKit` 提供了 `RUN --mount=type=cache` 指令，可以实现上边的设想。

```dockerfile
# syntax = docker/dockerfile:experimental
FROM node:alpine as builder
WORKDIR /app
COPY package.json /app/
RUN --mount=type=cache,target=/app/node_modules,id=my_app_npm_module,sharing=locked \
    --mount=type=cache,target=/root/.npm,id=npm_cache \
        npm i --registry=https://registry.npm.taobao.org
COPY src /app/src
RUN --mount=type=cache,target=/app/node_modules,id=my_app_npm_module,sharing=locked \
# --mount=type=cache,target=/app/dist,id=my_app_dist,sharing=locked \
        npm run build
FROM nginx:alpine
# COPY --from=builder /app/dist /app/dist
# 为了更直观的说明 from 和 source 指令，这里使用 RUN 指令
RUN --mount=type=cache,target=/tmp/dist,from=builder,source=/app/dist \
    # --mount=type=cache,target/tmp/dist,from=my_app_dist,sharing=locked \
    mkdir -p /app/dist && cp -r /tmp/dist/* /app/dist
```

**由于 `BuildKit` 为实验特性，每个 `Dockerfile` 文件开头都必须加上如下指令**

```docker
# syntax = docker/dockerfile:experimental
```

第一个 `RUN` 指令执行后，`id` 为 `my_app_npm_module` 的缓存文件夹挂载到了 `/app/node_modules` 文件夹中。多次执行也不会产生多个中间层镜像。

第二个 `RUN` 指令执行时需要用到 `node_modules` 文件夹，`node_modules` 已经挂载，命令也可以正确执行。

第三个 `RUN` 指令将上一阶段产生的文件复制到指定位置，`from` 指明缓存的来源，这里 `builder` 表示缓存来源于构建的第一阶段，`source` 指明缓存来源的文件夹。

上面的 `Dockerfile` 中 `--mount=type=cache,...` 中指令作用如下：

|Option               |Description|
|---------------------|-----------|
|`id`                 | `id` 设置一个标志，以便区分缓存。|
|`target` (必填项)     | 缓存的挂载目标文件夹。|
|`ro`,`readonly`      | 只读，缓存文件夹不能被写入。 |
|`sharing`            | 有 `shared` `private` `locked` 值可供选择。`sharing` 设置当一个缓存被多次使用时的表现，由于 `BuildKit` 支持并行构建，当多个步骤使用同一缓存时（同一 `id`）会发生冲突。`shared` 表示多个步骤可以同时读写，`private` 表示当多个步骤使用同一缓存时，每个步骤使用不同的缓存，`locked` 表示当一个步骤完成释放缓存后，后一个步骤才能继续使用该缓存。|
|`from`               | 缓存来源（构建阶段），不填写时为空文件夹。|
|`source`             | 来源的文件夹路径。|

#### `RUN --mount=type=bind`

该指令可以将一个镜像（或上一构建阶段）的文件挂载到指定位置。

```docker
# syntax = docker/dockerfile:experimental
RUN --mount=type=bind,from=php:alpine,source=/usr/local/bin/docker-php-entrypoint,target=/docker-php-entrypoint \
        cat /docker-php-entrypoint
```

#### `RUN --mount=type=tmpfs`

该指令可以将一个 `tmpfs` 文件系统挂载到指定位置。

```docker
# syntax = docker/dockerfile:experimental
RUN --mount=type=tmpfs,target=/temp \
        mount | grep /temp
```

#### `RUN --mount=type=secret`

该指令可以将一个文件(例如密钥)挂载到指定位置。

```docker
# syntax = docker/dockerfile:experimental
RUN --mount=type=secret,id=aws,target=/root/.aws/credentials \
        cat /root/.aws/credentials
```

```bash
$ docker build -t test --secret id=aws,src=$HOME/.aws/credentials .
```

#### `RUN --mount=type=ssh`

该指令可以挂载 `ssh` 密钥。

```dockerfile
# syntax = docker/dockerfile:experimental
FROM alpine
RUN apk add --no-cache openssh-client
RUN mkdir -p -m 0700 ~/.ssh && ssh-keyscan gitlab.com >> ~/.ssh/known_hosts
RUN --mount=type=ssh ssh git@gitlab.com | tee /hello
```

```bash
$ eval $(ssh-agent)
$ ssh-add ~/.ssh/id_rsa
(Input your passphrase here)
$ docker build -t test --ssh default=$SSH_AUTH_SOCK .
```

### docker-compose build 使用 Buildkit

设置 `COMPOSE_DOCKER_CLI_BUILD=1` 环境变量即可使用。

### 官方文档

* https://github.com/moby/buildkit/blob/master/frontend/dockerfile/docs/experimental.md



## 使用 Buildx 构建镜像

### 使用

你可以直接使用 `docker buildx build` 命令构建镜像。

```bash
$ docker buildx build .
[+] Building 8.4s (23/32)
 => ...
```

Buildx 使用 [BuildKit 引擎](buildkit.md) 进行构建，支持许多新的功能，具体参考 [Buildkit](buildkit.md) 一节。

### 官方文档

* https://docs.docker.com/engine/reference/commandline/buildx/

## 使用 buildx 构建多种系统架构支持的 Docker 镜像

在之前的版本中构建多种系统架构支持的 Docker 镜像，要想使用统一的名字必须使用 [`$ docker manifest`](../image/manifest.md) 命令。

在 Docker 19.03+ 版本中可以使用 `$ docker buildx build` 命令使用 `BuildKit` 构建镜像。该命令支持 `--platform` 参数可以同时构建支持多种系统架构的 Docker 镜像，大大简化了构建步骤。

### 新建 `builder` 实例

Docker for Linux 不支持构建 `arm` 架构镜像，我们可以运行一个新的容器让其支持该特性，Docker 桌面版无需进行此项设置。

```bash
$ docker run --rm --privileged tonistiigi/binfmt:latest --install all
```

由于 Docker 默认的 `builder` 实例不支持同时指定多个 `--platform`，我们必须首先创建一个新的 `builder` 实例。同时由于国内拉取镜像较缓慢，我们可以使用配置了 [镜像加速地址](https://github.com/moby/buildkit/blob/master/docs/buildkitd.toml.md)  的 [`dockerpracticesig/buildkit:master`](https://github.com/docker-practice/buildx) 镜像替换官方镜像。

> 如果你有私有的镜像加速器，可以基于 https://github.com/docker-practice/buildx 构建自己的 buildkit 镜像并使用它。

```bash
# 适用于国内环境
$ docker buildx create --use --name=mybuilder-cn --driver docker-container --driver-opt image=dockerpracticesig/buildkit:master
# 适用于腾讯云环境(腾讯云主机、coding.net 持续集成)
$ docker buildx create --use --name=mybuilder-cn --driver docker-container --driver-opt image=dockerpracticesig/buildkit:master-tencent
# $ docker buildx create --name mybuilder --driver docker-container
$ docker buildx use mybuilder
```

### 构建镜像

新建 Dockerfile 文件。

```dockerfile
FROM --platform=$TARGETPLATFORM alpine
RUN uname -a > /os.txt
CMD cat /os.txt
```

使用 `$ docker buildx build` 命令构建镜像，注意将 `myusername` 替换为自己的 Docker Hub 用户名。

`--push` 参数表示将构建好的镜像推送到 Docker 仓库。

```bash
$ docker buildx build --platform linux/arm,linux/arm64,linux/amd64 -t myusername/hello . --push
# 查看镜像信息
$ docker buildx imagetools inspect myusername/hello
```

在不同架构运行该镜像，可以得到该架构的信息。

```bash
# arm
$ docker run -it --rm myusername/hello
Linux buildkitsandbox 4.9.125-linuxkit #1 SMP Fri Sep 7 08:20:28 UTC 2018 armv7l Linux
# arm64
$ docker run -it --rm myusername/hello
Linux buildkitsandbox 4.9.125-linuxkit #1 SMP Fri Sep 7 08:20:28 UTC 2018 aarch64 Linux
# amd64
$ docker run -it --rm myusername/hello
Linux buildkitsandbox 4.9.125-linuxkit #1 SMP Fri Sep 7 08:20:28 UTC 2018 x86_64 Linux
```

### 架构相关变量

`Dockerfile` 支持如下架构相关的变量

**TARGETPLATFORM** 

构建镜像的目标平台，例如 `linux/amd64`, `linux/arm/v7`, `windows/amd64`。

**TARGETOS** 

`TARGETPLATFORM` 的 OS 类型，例如 `linux`, `windows`

**TARGETARCH** 

`TARGETPLATFORM` 的架构类型，例如 `amd64`, `arm`

**TARGETVARIANT**

`TARGETPLATFORM` 的变种，该变量可能为空，例如 `v7`

**BUILDPLATFORM**

构建镜像主机平台，例如 `linux/amd64`

**BUILDOS** 

`BUILDPLATFORM` 的 OS 类型，例如 `linux`

**BUILDARCH** 

`BUILDPLATFORM` 的架构类型，例如 `amd64`

**BUILDVARIANT** 

`BUILDPLATFORM` 的变种，该变量可能为空，例如 `v7`

#### 使用举例

例如我们要构建支持 `linux/arm/v7` 和 `linux/amd64` 两种架构的镜像。假设已经生成了两个平台对应的二进制文件：

* `bin/dist-linux-arm`
* `bin/dist-linux-amd64`

那么 `Dockerfile` 可以这样书写：

```dockerfile
FROM scratch
# 使用变量必须申明
ARG TARGETOS
ARG TARGETARCH
COPY bin/dist-${TARGETOS}-${TARGETARCH} /dist
ENTRYPOINT ["dist"]
```



# etcd

`etcd` 是 `CoreOS` 团队发起的一个管理配置信息和服务发现（`Service Discovery`）的项目，在这一章里面，我们将基于 `etcd 3.x` 版本介绍该项目的目标，安装和使用，以及实现的技术。

## 什么是 etcd

`etcd` 是 `CoreOS` 团队于 2013 年 6 月发起的开源项目，它的目标是构建一个高可用的分布式键值（`key-value`）数据库，基于 `Go` 语言实现。我们知道，在分布式系统中，各种服务的配置信息的管理分享，服务的发现是一个很基本同时也是很重要的问题。`CoreOS` 项目就希望基于 `etcd` 来解决这一问题。

`etcd` 目前在 [github.com/etcd-io/etcd](https://github.com/etcd-io/etcd) 进行维护。

受到 [Apache ZooKeeper](https://zookeeper.apache.org/) 项目和 [doozer](https://github.com/ha/doozerd) 项目的启发，`etcd` 在设计的时候重点考虑了下面四个要素：

* 简单：具有定义良好、面向用户的 `API` ([gRPC](https://github.com/grpc/grpc))

* 安全：支持 `HTTPS` 方式的访问

* 快速：支持并发 `10 k/s` 的写操作

* 可靠：支持分布式结构，基于 `Raft` 的一致性算法

*Apache ZooKeeper 是一套知名的分布式系统中进行同步和一致性管理的工具。*

*doozer 是一个一致性分布式数据库。*

*[Raft](https://raft.github.io/) 是一套通过选举主节点来实现分布式系统一致性的算法，相比于大名鼎鼎的 Paxos 算法，它的过程更容易被人理解，由 Stanford 大学的 Diego Ongaro 和 John Ousterhout 提出。更多细节可以参考 [raftconsensus.github.io](http://raftconsensus.github.io)。*

一般情况下，用户使用 `etcd` 可以在多个节点上启动多个实例，并添加它们为一个集群。同一个集群中的 `etcd` 实例将会保持彼此信息的一致性。


## 安装

`etcd` 基于 `Go` 语言实现，因此，用户可以从 [项目主页](https://github.com/etcd-io/etcd) 下载源代码自行编译，也可以下载编译好的二进制文件，甚至直接使用制作好的 `Docker` 镜像文件来体验。

>注意：本章节内容基于 etcd `3.4.x` 版本

### 二进制文件方式下载

编译好的二进制文件都在 [github.com/etcd-io/etcd/releases](https://github.com/etcd-io/etcd/releases/) 页面，用户可以选择需要的版本，或通过下载工具下载。

例如，使用 `curl` 工具下载压缩包，并解压。

```bash
$ curl -L https://github.com/etcd-io/etcd/releases/download/v3.4.0/etcd-v3.4.0-linux-amd64.tar.gz -o etcd-v3.4.0-linux-amd64.tar.gz
# 国内用户可以使用以下方式加快下载
$ curl -L https://download.fastgit.org/etcd-io/etcd/releases/download/v3.4.0/etcd-v3.4.0-linux-amd64.tar.gz -o etcd-v3.4.0-linux-amd64.tar.gz
$ tar xzvf etcd-v3.4.0-linux-amd64.tar.gz
$ cd etcd-v3.4.0-linux-amd64
```

解压后，可以看到文件包括

```bash
$ ls
Documentation README-etcdctl.md README.md READMEv2-etcdctl.md etcd etcdctl
```

其中 `etcd` 是服务主文件，`etcdctl` 是提供给用户的命令客户端，其他文件是支持文档。

下面将 `etcd` `etcdctl` 文件放到系统可执行目录（例如 `/usr/local/bin/`）。

```bash
$ sudo cp etcd* /usr/local/bin/
```

默认 `2379` 端口处理客户端的请求，`2380` 端口用于集群各成员间的通信。启动 `etcd` 显示类似如下的信息：

```bash
$ etcd
...
2017-12-03 11:18:34.411579 I | embed: listening for peers on http://localhost:2380
2017-12-03 11:18:34.411938 I | embed: listening for client requests on localhost:2379
```

此时，可以使用 `etcdctl` 命令进行测试，设置和获取键值 `testkey: "hello world"`，检查 `etcd` 服务是否启动成功：

```bash
$ ETCDCTL_API=3 etcdctl member list
8e9e05c52164694d, started, default, http://localhost:2380, http://localhost:2379
$ ETCDCTL_API=3 etcdctl put testkey "hello world"
OK
$ etcdctl get testkey
testkey
hello world
```

说明 etcd 服务已经成功启动了。

### Docker 镜像方式运行

镜像名称为 `quay.io/coreos/etcd`，可以通过下面的命令启动 `etcd` 服务监听到 `2379` 和 `2380` 端口。

```bash
$ docker run \
-p 2379:2379 \
-p 2380:2380 \
--mount type=bind,source=/tmp/etcd-data.tmp,destination=/etcd-data \
--name etcd-gcr-v3.4.0 \
quay.io/coreos/etcd:v3.4.0 \
/usr/local/bin/etcd \
--name s1 \
--data-dir /etcd-data \
--listen-client-urls http://0.0.0.0:2379 \
--advertise-client-urls http://0.0.0.0:2379 \
--listen-peer-urls http://0.0.0.0:2380 \
--initial-advertise-peer-urls http://0.0.0.0:2380 \
--initial-cluster s1=http://0.0.0.0:2380 \
--initial-cluster-token tkn \
--initial-cluster-state new \
--log-level info \
--logger zap \
--log-outputs stderr
```

打开新的终端按照上一步的方法测试 `etcd` 是否成功启动。

### macOS 中运行

```bash
$ brew install etcd
$ etcd
$ etcdctl member list
```

## etcd 集群

下面我们使用 [Docker Compose](../compose/) 模拟启动一个 3 节点的 `etcd` 集群。

编辑 `docker-compose.yml` 文件

```yaml
version: "3.6"
services:
  node1:
    image: quay.io/coreos/etcd:v3.4.0
    volumes:
      - node1-data:/etcd-data
    expose:
      - 2379
      - 2380
    networks:
      cluster_net:
        ipv4_address: 172.16.238.100
    environment:
      - ETCDCTL_API=3
    command:
      - /usr/local/bin/etcd
      - --data-dir=/etcd-data
      - --name
      - node1
      - --initial-advertise-peer-urls
      - http://172.16.238.100:2380
      - --listen-peer-urls
      - http://0.0.0.0:2380
      - --advertise-client-urls
      - http://172.16.238.100:2379
      - --listen-client-urls
      - http://0.0.0.0:2379
      - --initial-cluster
      - node1=http://172.16.238.100:2380,node2=http://172.16.238.101:2380,node3=http://172.16.238.102:2380
      - --initial-cluster-state
      - new
      - --initial-cluster-token
      - docker-etcd
  node2:
    image: quay.io/coreos/etcd:v3.4.0
    volumes:
      - node2-data:/etcd-data
    networks:
      cluster_net:
        ipv4_address: 172.16.238.101
    environment:
      - ETCDCTL_API=3
    expose:
      - 2379
      - 2380
    command:
      - /usr/local/bin/etcd
      - --data-dir=/etcd-data
      - --name
      - node2
      - --initial-advertise-peer-urls
      - http://172.16.238.101:2380
      - --listen-peer-urls
      - http://0.0.0.0:2380
      - --advertise-client-urls
      - http://172.16.238.101:2379
      - --listen-client-urls
      - http://0.0.0.0:2379
      - --initial-cluster
      - node1=http://172.16.238.100:2380,node2=http://172.16.238.101:2380,node3=http://172.16.238.102:2380
      - --initial-cluster-state
      - new
      - --initial-cluster-token
      - docker-etcd
  node3:
    image: quay.io/coreos/etcd:v3.4.0
    volumes:
      - node3-data:/etcd-data
    networks:
      cluster_net:
        ipv4_address: 172.16.238.102
    environment:
      - ETCDCTL_API=3
    expose:
      - 2379
      - 2380
    command:
      - /usr/local/bin/etcd
      - --data-dir=/etcd-data
      - --name
      - node3
      - --initial-advertise-peer-urls
      - http://172.16.238.102:2380
      - --listen-peer-urls
      - http://0.0.0.0:2380
      - --advertise-client-urls
      - http://172.16.238.102:2379
      - --listen-client-urls
      - http://0.0.0.0:2379
      - --initial-cluster
      - node1=http://172.16.238.100:2380,node2=http://172.16.238.101:2380,node3=http://172.16.238.102:2380
      - --initial-cluster-state
      - new
      - --initial-cluster-token
      - docker-etcd
volumes:
  node1-data:
  node2-data:
  node3-data:
networks:
  cluster_net:
    driver: bridge
    ipam:
      driver: default
      config:
      -
        subnet: 172.16.238.0/24
```

使用 `docker-compose up` 启动集群之后使用 `docker exec` 命令登录到任一节点测试 `etcd` 集群。

```bash
/ # etcdctl member list
daf3fd52e3583ff, started, node3, http://172.16.238.102:2380, http://172.16.238.102:2379
422a74f03b622fef, started, node1, http://172.16.238.100:2380, http://172.16.238.100:2379
ed635d2a2dbef43d, started, node2, http://172.16.238.101:2380, http://172.16.238.101:2379
```

## 使用 etcdctl

`etcdctl` 是一个命令行客户端，它能提供一些简洁的命令，供用户直接跟 `etcd` 服务打交道，而无需基于 `HTTP API` 方式。这在某些情况下将很方便，例如用户对服务进行测试或者手动修改数据库内容。我们也推荐在刚接触 `etcd` 时通过 `etcdctl` 命令来熟悉相关的操作，这些操作跟 `HTTP API` 实际上是对应的。

`etcd` 项目二进制发行包中已经包含了 `etcdctl` 工具，没有的话，可以从 [github.com/etcd-io/etcd/releases](https://github.com/etcd-io/etcd/releases) 下载。

`etcdctl` 支持如下的命令，大体上分为数据库操作和非数据库操作两类，后面将分别进行解释。

```bash
NAME:
	etcdctl - A simple command line client for etcd3.
USAGE:
	etcdctl
VERSION:
	3.4.0
API VERSION:
	3.4
COMMANDS:
	get			Gets the key or a range of keys
	put			Puts the given key into the store
	del			Removes the specified key or range of keys [key, range_end)
	txn			Txn processes all the requests in one transaction
	compaction		Compacts the event history in etcd
	alarm disarm		Disarms all alarms
	alarm list		Lists all alarms
	defrag			Defragments the storage of the etcd members with given endpoints
	endpoint health		Checks the healthiness of endpoints specified in `--endpoints` flag
	endpoint status		Prints out the status of endpoints specified in `--endpoints` flag
	watch			Watches events stream on keys or prefixes
	version			Prints the version of etcdctl
	lease grant		Creates leases
	lease revoke		Revokes leases
	lease timetolive	Get lease information
	lease keep-alive	Keeps leases alive (renew)
	member add		Adds a member into the cluster
	member remove		Removes a member from the cluster
	member update		Updates a member in the cluster
	member list		Lists all members in the cluster
	snapshot save		Stores an etcd node backend snapshot to a given file
	snapshot restore	Restores an etcd member snapshot to an etcd directory
	snapshot status		Gets backend snapshot status of a given file
	make-mirror		Makes a mirror at the destination etcd cluster
	migrate			Migrates keys in a v2 store to a mvcc store
	lock			Acquires a named lock
	elect			Observes and participates in leader election
	auth enable		Enables authentication
	auth disable		Disables authentication
	user add		Adds a new user
	user delete		Deletes a user
	user get		Gets detailed information of a user
	user list		Lists all users
	user passwd		Changes password of user
	user grant-role		Grants a role to a user
	user revoke-role	Revokes a role from a user
	role add		Adds a new role
	role delete		Deletes a role
	role get		Gets detailed information of a role
	role list		Lists all roles
	role grant-permission	Grants a key to a role
	role revoke-permission	Revokes a key from a role
	check perf		Check the performance of the etcd cluster
	help			Help about any command
OPTIONS:
      --cacert=""				verify certificates of TLS-enabled secure servers using this CA bundle
      --cert=""					identify secure client using this TLS certificate file
      --command-timeout=5s			timeout for short running command (excluding dial timeout)
      --debug[=false]				enable client-side debug logging
      --dial-timeout=2s				dial timeout for client connections
      --endpoints=[127.0.0.1:2379]		gRPC endpoints
      --hex[=false]				print byte strings as hex encoded strings
      --insecure-skip-tls-verify[=false]	skip server certificate verification
      --insecure-transport[=true]		disable transport security for client connections
      --key=""					identify secure client using this TLS key file
      --user=""					username[:password] for authentication (prompt if password is not supplied)
  -w, --write-out="simple"			set the output format (fields, json, protobuf, simple, table)
```

### 数据库操作

数据库操作围绕对键值和目录的 CRUD （符合 REST 风格的一套操作：Create）完整生命周期的管理。

etcd 在键的组织上采用了层次化的空间结构（类似于文件系统中目录的概念），用户指定的键可以为单独的名字，如 `testkey`，此时实际上放在根目录 `/` 下面，也可以为指定目录结构，如 `cluster1/node2/testkey`，则将创建相应的目录结构。

>注：CRUD 即 Create, Read, Update, Delete，是符合 REST 风格的一套 API 操作。

#### put

```bash
$ etcdctl put /testdir/testkey "Hello world"
OK
```

#### get

获取指定键的值。例如

```bash
$ etcdctl put testkey hello
OK
$ etcdctl get testkey
testkey
hello
```

支持的选项为

`--sort`	对结果进行排序

`--consistent` 将请求发给主节点，保证获取内容的一致性

#### del

删除某个键值。例如

```bash
$ etcdctl del testkey
1
```

### 非数据库操作

#### watch

监测一个键值的变化，一旦键值发生更新，就会输出最新的值。

例如，用户更新 `testkey` 键值为 `Hello world`。

```bash
$ etcdctl watch testkey
PUT
testkey
2
```

#### member

通过 `list`、`add`、`update`、`remove` 命令列出、添加、更新、删除 etcd 实例到 etcd 集群中。

例如本地启动一个 `etcd` 服务实例后，可以用如下命令进行查看。

```bash
$ etcdctl member list
422a74f03b622fef, started, node1, http://172.16.238.100:2380, http://172.16.238.100:23
```

# Fedora CoreOS
CoreOS 是一个专门为安全和大规模运行容器化工作负载而构建的新 Fedora 版本，它继承了 Fedora Atomic Host 和 CoreOS Container Linux 的优势。

CoreOS 的安装文件和运行依赖非常小，它提供了精简的 Linux 系统。它使用 Linux 容器在更高的抽象层来管理你的服务，而不是通过常规的包管理工具 yum 或 apt 来安装包。

同时，CoreOS 几乎可以运行在任何平台：VirtualBox Amazon EC2 QEMU/KVM VMware Bare Metal 和 OpenStack 等 。

## Fedora CoreOS 介绍

[Fedora CoreOS](https://getfedora.org/coreos/) 是一个自动更新的，最小的，整体的，以容器为中心的操作系统，不仅适用于集群，而且可独立运行，并针对运行 Kubernetes 进行了优化。它旨在结合 CoreOS Container Linux 和 Fedora Atomic Host 的优点，将 Container Linux 中的 [Ignition](https://github.com/coreos/ignition) 与 [rpm-ostree](https://github.com/coreos/rpm-ostree) 和 Project Atomic 中的 SELinux 强化等技术相集成。其目标是提供最佳的容器主机，以安全，大规模地运行容器化的工作负载。

### FCOS 特性

#### 一个最小化操作系统

FCOS 被设计成一个基于容器的最小化的现代操作系统。它比现有的 Linux 安装平均节省 40% 的 RAM（大约 114M ）并允许从 PXE 或 iPXE 非常快速的启动。

#### 系统初始化

Ignition 是一种配置实用程序，可读取配置文件（JSON 格式）并根据该配置配置 FCOS 系统。可配置的组件包括存储，文件系统，systemd 和用户。

Ignition 在系统首次启动期间（在 initramfs 中）仅运行一次。由于 Ignition 在启动过程中的早期运行，因此它可以在用户空间开始启动之前重新对磁盘分区，格式化文件系统，创建用户并写入文件。当 systemd 启动时，systemd 服务已被写入磁盘，从而加快了启动时间。

#### 自动更新

FCOS 使用 rpm-ostree 系统进行事务性升级。无需像 yum 升级那样升级单个软件包，而是 rpm-ostree 将 OS 升级作为一个原子单元进行。新的 OS 部署在升级期间进行，并在下次重新引导时生效。如果升级出现问题，则一次回滚和重新启动会使系统返回到先前的状态。确保了系统升级对群集容量的影响降到最小。

#### 容器工具

对于诸如构建，复制和其他管理容器的任务，FCOS 用一组容器工具代替了 **Docker CLI**。**podman CLI** 工具支持许多容器运行时功能，例如运行，启动，停止，列出和删除容器和镜像。**skopeo CLI** 工具可以复制，认证和签名镜像。您还可以使用 **crictl CLI** 工具来处理 CRI-O 容器引擎中的容器和镜像。

### 参考文档

* [官方文档](https://docs.fedoraproject.org/en-US/fedora-coreos/)
* [openshift 官方文档](https://docs.openshift.com/container-platform/4.3/architecture/architecture-rhcos.html)

# podman

[`podman`](https://github.com/containers/podman) 是一个无守护程序与 docker 命令兼容的下一代 Linux 容器工具。

## 安装

```bash
$ sudo yum -y install podman
```

## 使用

`podman` 与 docker 命令完全兼容，只需将 `docker` 替换为 `podman` 即可，例如运行一个容器：

```bash
# $ docker run -d -p 80:80 nginx:alpine
$ podman run -d -p 80:80 nginx:alpine
```

## 参考

* https://developers.redhat.com/blog/2019/02/21/podman-and-buildah-for-docker-users/


# 常见问题总结

## 镜像相关

### 如何批量清理临时镜像文件？

答：可以使用 `docker image prune` 命令。

### 如何查看镜像支持的环境变量？

答：可以使用 `docker run IMAGE env` 命令。

### 本地的镜像文件都存放在哪里？

答：与 Docker 相关的本地资源默认存放在 `/var/lib/docker/` 目录下，以 `overlay2` 文件系统为例，其中 `containers` 目录存放容器信息，`image` 目录存放镜像信息，`overlay2` 目录下存放具体的镜像层文件。

### 构建 Docker 镜像应该遵循哪些原则？

答：整体原则上，尽量保持镜像功能的明确和内容的精简，要点包括

* 尽量选取满足需求但较小的基础系统镜像，例如大部分时候可以选择 `alpine` 镜像，仅有不足六兆大小；

* 清理编译生成文件、安装包的缓存等临时文件；

* 安装各个软件时候要指定准确的版本号，并避免引入不需要的依赖；

* 从安全角度考虑，应用要尽量使用系统的库和依赖；

* 如果安装应用时候需要配置一些特殊的环境变量，在安装后要还原不需要保持的变量值；

* 使用 Dockerfile 创建镜像时候要添加 .dockerignore 文件或使用干净的工作目录。

更多内容请查看 [Dockerfile 最佳实践](../best_practices.md)

### 碰到网络问题，无法 pull 镜像，命令行指定 http_proxy 无效？

答：在 Docker 配置文件中添加 `export http_proxy="http://<PROXY_HOST>:<PROXY_PORT>"`，之后重启 Docker 服务即可。

## 容器相关

### 容器退出后，通过 docker container ls 命令查看不到，数据会丢失么？

答：容器退出后会处于终止（exited）状态，此时可以通过 `docker container ls -a` 查看。其中的数据也不会丢失，还可以通过 `docker start` 命令来启动它。只有删除掉容器才会清除所有数据。

### 如何停止所有正在运行的容器？

答：可以使用 `docker stop $(docker container ls -q)` 命令。

### 如何批量清理已经停止的容器？

答：可以使用 `docker container prune` 命令。

### 如何获取某个容器的 PID 信息？

答：可以使用

```bash
docker inspect --format '{{ .State.Pid }}' <CONTAINER ID or NAME>
```

### 如何获取某个容器的 IP 地址？

答：可以使用
```bash
docker inspect --format '{{ .NetworkSettings.IPAddress }}' <CONTAINER ID or NAME>
```

### 如何给容器指定一个固定 IP 地址，而不是每次重启容器 IP 地址都会变？

答：使用以下命令启动容器可以使容器 IP 固定不变

```bash
$ docker network create -d bridge --subnet 172.25.0.0/16 my-net
$ docker run --network=my-net --ip=172.25.3.3 -itd --name=my-container busybox
```

### 如何临时退出一个正在交互的容器的终端，而不终止它？

答：按 `Ctrl-p Ctrl-q`。如果按 `Ctrl-c` 往往会让容器内应用进程终止，进而会终止容器。

### 使用 `docker port` 命令映射容器的端口时，系统报错“Error: No public port '80' published for xxx”？

答：

* 创建镜像时 `Dockerfile` 要通过 `EXPOSE` 指定正确的开放端口；

* 容器启动时指定 `PublishAllPort = true`。

### 可以在一个容器中同时运行多个应用进程么？

答：一般并不推荐在同一个容器内运行多个应用进程。如果有类似需求，可以通过一些额外的进程管理机制，比如 `supervisord` 来管理所运行的进程。可以参考 https://docs.docker.com/config/containers/multi-service_container/ 。

### 如何控制容器占用系统资源（CPU、内存）的份额？

答：在使用 `docker create` 命令创建容器或使用 `docker run` 创建并启动容器的时候，可以使用 `-c|--cpu-shares[=0]` 参数来调整容器使用 CPU 的权重；使用 `-m|--memory[=MEMORY]` 参数来调整容器使用内存的大小。

## 仓库相关

### 仓库（Repository）、注册服务器（Registry）、注册索引（Index） 有何关系？

首先，仓库是存放一组关联镜像的集合，比如同一个应用的不同版本的镜像。

注册服务器是存放实际的镜像文件的地方。注册索引则负责维护用户的账号、权限、搜索、标签等的管理。因此，注册服务器利用注册索引来实现认证等管理。

## 配置相关

### Docker 的配置文件放在哪里，如何修改配置？

答：使用 `systemd` 的系统（如 Ubuntu 16.04、Centos 等）的配置文件在 `/etc/docker/daemon.json`。


### 如何更改 Docker 的默认存储位置？

答：Docker 的默认存储位置是 `/var/lib/docker`，如果希望将 Docker 的本地文件存储到其他分区，可以使用 Linux 软连接的方式来完成，或者在启动 daemon 时通过 `-g` 参数指定，或者修改配置文件 `/etc/docker/daemon.json` 的 "data-root" 项 。可以使用 `docker system info | grep "Root Dir"` 查看当前使用的存储位置。

例如，如下操作将默认存储位置迁移到 /storage/docker。

```sh
[root@s26 ~]# df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root   50G  5.3G   42G  12% /
tmpfs                          48G  228K   48G   1% /dev/shm
/dev/sda1                     485M   40M  420M   9% /boot
/dev/mapper/VolGroup-lv_home  222G  188M  210G   1% /home
/dev/sdb2                     2.7T  323G  2.3T  13% /storage
[root@s26 ~]# service docker stop
[root@s26 ~]# cd /var/lib/
[root@s26 lib]# mv docker /storage/
[root@s26 lib]# ln -s /storage/docker/ docker
[root@s26 lib]# ls -la docker
lrwxrwxrwx. 1 root root 15 11月 17 13:43 docker -> /storage/docker
[root@s26 lib]# service docker start
```

### 使用内存和 swap 限制启动容器时候报警告："WARNING: Your kernel does not support cgroup swap limit. WARNING: Your kernel does not support swap limit capabilities. Limitation discarded."？

答：这是因为系统默认没有开启对内存和 swap 使用的统计功能，引入该功能会带来性能的下降。要开启该功能，可以采取如下操作：

* 编辑 `/etc/default/grub` 文件（Ubuntu 系统为例），配置 `GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"`

* 更新 grub：`$ sudo update-grub`

* 重启系统，即可。

## Docker 与虚拟化

### Docker 与 LXC（Linux Container）有何不同？

答：LXC 利用 Linux 上相关技术实现了容器。Docker 则在如下的几个方面进行了改进：
* 移植性：通过抽象容器配置，容器可以实现从一个平台移植到另一个平台；
* 镜像系统：基于 OverlayFS 的镜像系统为容器的分发带来了很多的便利，同时共同的镜像层只需要存储一份，实现高效率的存储；
* 版本管理：类似于Git的版本管理理念，用户可以更方便的创建、管理镜像文件；
* 仓库系统：仓库系统大大降低了镜像的分发和管理的成本；
* 周边工具：各种现有工具（配置管理、云平台）对 Docker 的支持，以及基于 Docker的 PaaS、CI 等系统，让 Docker 的应用更加方便和多样化。

### Docker 与 Vagrant 有何不同？

答：两者的定位完全不同。

* Vagrant 类似 Boot2Docker（一款运行 Docker 的最小内核），是一套虚拟机的管理环境。Vagrant 可以在多种系统上和虚拟机软件中运行，可以在 Windows，Mac 等非 Linux 平台上为 Docker 提供支持，自身具有较好的包装性和移植性。

* 原生的 Docker 自身只能运行在 Linux 平台上，但启动和运行的性能都比虚拟机要快，往往更适合快速开发和部署应用的场景。

简单说：Vagrant 适合用来管理虚拟机，而 Docker 适合用来管理应用环境。

### 开发环境中 Docker 和 Vagrant 该如何选择？

答：Docker 不是虚拟机，而是进程隔离，对于资源的消耗很少，但是目前需要 Linux 环境支持。Vagrant 是虚拟机上做的封装，虚拟机本身会消耗资源。

如果本地使用的 Linux 环境，推荐都使用 Docker。

如果本地使用的是 macOS 或者 Windows 环境，那就需要开虚拟机，单一开发环境下 Vagrant 更简单；多环境开发下推荐在 Vagrant 里面再使用 Docker 进行环境隔离。

## 其它

### Docker 能在非 Linux 平台（比如 Windows 或 macOS ）上运行么？

答：完全可以。安装方法请查看 [安装 Docker](../../install/) 一节

### 如何将一台宿主主机的 Docker 环境迁移到另外一台宿主主机？

答：停止 Docker 服务。将整个 Docker 存储文件夹复制到另外一台宿主主机，然后调整另外一台宿主主机的配置即可。

### 如何进入 Docker 容器的网络命名空间？

答：Docker 在创建容器后，删除了宿主主机上 `/var/run/netns` 目录中的相关的网络命名空间文件。因此，在宿主主机上是无法看到或访问容器的网络命名空间的。

用户可以通过如下方法来手动恢复它。

首先，使用下面的命令查看容器进程信息，比如这里的 1234。

```bash
$ docker inspect --format='{{.State.Pid}} ' $container_id
1234
```

接下来，在 `/proc` 目录下，把对应的网络命名空间文件链接到 `/var/run/netns` 目录。

```bash
$ sudo ln -s /proc/1234/ns/net /var/run/netns/
```

然后，在宿主主机上就可以看到容器的网络命名空间信息。例如

```bash
$ sudo ip netns show
1234
```

此时，用户可以通过正常的系统命令来查看或操作容器的命名空间了。例如修改容器的 IP 地址信息为 `172.17.0.100/16`。

```bash
$ sudo ip netns exec 1234 ifconfig eth0 172.17.0.100/16
```

### 如何获取容器绑定到本地那个 veth 接口上？

答：Docker 容器启动后，会通过 veth 接口对连接到本地网桥，veth 接口命名跟容器命名毫无关系，十分难以找到对应关系。

最简单的一种方式是通过查看接口的索引号，在容器中执行 `ip a` 命令，查看到本地接口最前面的接口索引号，如 `205`，将此值加上 1，即 `206`，然后在本地主机执行 `ip a` 命令，查找接口索引号为 `206` 的接口，两者即为连接的 veth 接口对。


# Docker 命令查询

## 基本语法

Docker 命令有两大类，客户端命令和服务端命令。前者是主要的操作接口，后者用来启动 Docker Daemon。

* 客户端命令：基本命令格式为 `docker [OPTIONS] COMMAND [arg...]`；

* 服务端命令：基本命令格式为 `dockerd [OPTIONS]`。

可以通过 `man docker` 或 `docker help` 来查看这些命令。

接下来的小节对这两个命令进行介绍。

## 客户端命令(docker)

### 客户端命令选项

* `--config=""`：指定客户端配置文件，默认为 `~/.docker`；
* `-D=true|false`：是否使用 debug 模式。默认不开启；
* `-H, --host=[]`：指定命令对应 Docker 守护进程的监听接口，可以为 unix 套接字 `unix:///path/to/socket`，文件句柄 `fd://socketfd` 或 tcp 套接字 `tcp://[host[:port]]`，默认为 `unix:///var/run/docker.sock`；
* `-l, --log-level="debug|info|warn|error|fatal"`：指定日志输出级别；
* `--tls=true|false`：是否对 Docker 守护进程启用 TLS 安全机制，默认为否；
* `--tlscacert=/.docker/ca.pem`：TLS CA 签名的可信证书文件路径；
* `--tlscert=/.docker/cert.pem`：TLS 可信证书文件路径；
* `--tlscert=/.docker/key.pem`：TLS 密钥文件路径；
* `--tlsverify=true|false`：启用 TLS 校验，默认为否。

### 客户端命令

可以通过 `docker COMMAND --help` 来查看这些命令的具体用法。

* `attach`：依附到一个正在运行的容器中；
* `build`：从一个 Dockerfile 创建一个镜像；
* `commit`：从一个容器的修改中创建一个新的镜像；
* `cp`：在容器和本地宿主系统之间复制文件中；
* `create`：创建一个新容器，但并不运行它；
* `diff`：检查一个容器内文件系统的修改，包括修改和增加；
* `events`：从服务端获取实时的事件；
* `exec`：在运行的容器内执行命令；
* `export`：导出容器内容为一个 `tar` 包；
* `history`：显示一个镜像的历史信息；
* `images`：列出存在的镜像；
* `import`：导入一个文件（典型为 `tar` 包）路径或目录来创建一个本地镜像；
* `info`：显示一些相关的系统信息；
* `inspect`：显示一个容器的具体配置信息；
* `kill`：关闭一个运行中的容器 (包括进程和所有相关资源)；
* `load`：从一个 tar 包中加载一个镜像；
* `login`：注册或登录到一个 Docker 的仓库服务器；
* `logout`：从 Docker 的仓库服务器登出；
* `logs`：获取容器的 log 信息；
* `network`：管理 Docker 的网络，包括查看、创建、删除、挂载、卸载等；
* `node`：管理 swarm 集群中的节点，包括查看、更新、删除、提升/取消管理节点等；
* `pause`：暂停一个容器中的所有进程；
* `port`：查找一个 nat 到一个私有网口的公共口；
* `ps`：列出主机上的容器；
* `pull`：从一个Docker的仓库服务器下拉一个镜像或仓库；
* `push`：将一个镜像或者仓库推送到一个 Docker 的注册服务器；
* `rename`：重命名一个容器；
* `restart`：重启一个运行中的容器；
* `rm`：删除给定的若干个容器；
* `rmi`：删除给定的若干个镜像；
* `run`：创建一个新容器，并在其中运行给定命令；
* `save`：保存一个镜像为 tar 包文件；
* `search`：在 Docker index 中搜索一个镜像；
* `service`：管理 Docker 所启动的应用服务，包括创建、更新、删除等；
* `start`：启动一个容器；
* `stats`：输出（一个或多个）容器的资源使用统计信息；
* `stop`：终止一个运行中的容器；
* `swarm`：管理 Docker swarm 集群，包括创建、加入、退出、更新等；
* `tag`：为一个镜像打标签；
* `top`：查看一个容器中的正在运行的进程信息；
* `unpause`：将一个容器内所有的进程从暂停状态中恢复；
* `update`：更新指定的若干容器的配置信息；
* `version`：输出 Docker 的版本信息；
* `volume`：管理 Docker volume，包括查看、创建、删除等；
* `wait`：阻塞直到一个容器终止，然后输出它的退出符。

### 一张图总结 Docker 的命令
![Docker 命令总结](cmd_logic.png)

### 参考

* [官方文档](https://docs.docker.com/engine/reference/commandline/cli/)

## 服务端命令(dockerd)

### dockerd 命令选项

* `--api-cors-header=""`：CORS 头部域，默认不允许 CORS，要允许任意的跨域访问，可以指定为 "*"；
* `--authorization-plugin=""`：载入认证的插件；
* `-b=""`：将容器挂载到一个已存在的网桥上。指定为 `none` 时则禁用容器的网络，与 `--bip` 选项互斥；
* `--bip=""`：让动态创建的 `docker0` 网桥采用给定的 CIDR 地址; 与 `-b` 选项互斥；
* `--cgroup-parent=""`：指定 cgroup 的父组，默认 fs cgroup 驱动为 `/docker`，systemd cgroup 驱动为 `system.slice`；
* `--cluster-store=""`：构成集群（如 `Swarm`）时，集群键值数据库服务地址；
* `--cluster-advertise=""`：构成集群时，自身的被访问地址，可以为 `host:port` 或 `interface:port`；
* `--cluster-store-opt=""`：构成集群时，键值数据库的配置选项；
* `--config-file="/etc/docker/daemon.json"`：daemon 配置文件路径；
* `--containerd=""`：containerd 文件的路径；
* `-D, --debug=true|false`：是否使用 Debug 模式。缺省为 false；
* `--default-gateway=""`：容器的 IPv4 网关地址，必须在网桥的子网段内；
* `--default-gateway-v6=""`：容器的 IPv6 网关地址；
* `--default-ulimit=[]`：默认的 ulimit 值；
* `--disable-legacy-registry=true|false`：是否允许访问旧版本的镜像仓库服务器；
* `--dns=""`：指定容器使用的 DNS 服务器地址；
* `--dns-opt=""`：DNS 选项；
* `--dns-search=[]`：DNS 搜索域；
* `--exec-opt=[]`：运行时的执行选项；
* `--exec-root=""`：容器执行状态文件的根路径，默认为 `/var/run/docker`；
* `--fixed-cidr=""`：限定分配 IPv4 地址范围；
* `--fixed-cidr-v6=""`：限定分配 IPv6 地址范围；
* `-G, --group=""`：分配给 unix 套接字的组，默认为 `docker`；
* `-g, --graph=""`：Docker 运行时的根路径，默认为 `/var/lib/docker`；
* `-H, --host=[]`：指定命令对应 Docker daemon 的监听接口，可以为 unix 套接字 `unix:///path/to/socket`，文件句柄 `fd://socketfd` 或 tcp 套接字 `tcp://[host[:port]]`，默认为 `unix:///var/run/docker.sock`；
* `--icc=true|false`：是否启用容器间以及跟 daemon 所在主机的通信。默认为 true。
* `--insecure-registry=[]`：允许访问给定的非安全仓库服务；
* `--ip=""`：绑定容器端口时候的默认 IP 地址。缺省为 `0.0.0.0`；
* `--ip-forward=true|false`：是否检查启动在 Docker 主机上的启用 IP 转发服务，默认开启。注意关闭该选项将不对系统转发能力进行任何检查修改；
* `--ip-masq=true|false`：是否进行地址伪装，用于容器访问外部网络，默认开启；
* `--iptables=true|false`：是否允许 Docker 添加 iptables 规则。缺省为 true；
* `--ipv6=true|false`：是否启用 IPv6 支持，默认关闭；
* `-l, --log-level="debug|info|warn|error|fatal"`：指定日志输出级别；
* `--label="[]"`：添加指定的键值对标注；
* `--log-driver="json-file|syslog|journald|gelf|fluentd|awslogs|splunk|etwlogs|gcplogs|none"`：指定日志后端驱动，默认为 `json-file`；
* `--log-opt=[]`：日志后端的选项；
* `--mtu=VALUE`：指定容器网络的 `mtu`；
* `-p=""`：指定 daemon 的 PID 文件路径。缺省为 `/var/run/docker.pid`；
* `--raw-logs`：输出原始，未加色彩的日志信息；
* `--registry-mirror=<scheme>://<host>`：指定 `docker pull` 时使用的注册服务器镜像地址；
* `-s, --storage-driver=""`：指定使用给定的存储后端；
* `--selinux-enabled=true|false`：是否启用 SELinux 支持。缺省值为 false。SELinux 目前尚不支持 overlay 存储驱动；
* `--storage-opt=[]`：驱动后端选项；
* `--tls=true|false`：是否对 Docker daemon 启用 TLS 安全机制，默认为否；
* `--tlscacert=/.docker/ca.pem`：TLS CA 签名的可信证书文件路径；
* `--tlscert=/.docker/cert.pem`：TLS 可信证书文件路径；
* `--tlscert=/.docker/key.pem`：TLS 密钥文件路径；
* `--tlsverify=true|false`：启用 TLS 校验，默认为否；
* `--userland-proxy=true|false`：是否使用用户态代理来实现容器间和出容器的回环通信，默认为 true；
* `--userns-remap=default|uid:gid|user:group|user|uid`：指定容器的用户命名空间，默认是创建新的 UID 和 GID 映射到容器内进程。

### 参考

* [官方文档](https://docs.docker.com/engine/reference/commandline/dockerd/)


# Dockerfile 最佳实践

本附录是笔者对 Docker 官方文档中 [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) 的理解与翻译。

## 一般性的指南和建议

### 容器应该是短暂的

通过 `Dockerfile` 构建的镜像所启动的容器应该尽可能短暂（生命周期短）。「短暂」意味着可以停止和销毁容器，并且创建一个新容器并部署好所需的设置和配置工作量应该是极小的。

### 使用 `.dockerignore` 文件

使用 `Dockerfile` 构建镜像时最好是将 `Dockerfile` 放置在一个新建的空目录下。然后将构建镜像所需要的文件添加到该目录中。为了提高构建镜像的效率，你可以在目录下新建一个 `.dockerignore` 文件来指定要忽略的文件和目录。`.dockerignore` 文件的排除模式语法和 Git 的 `.gitignore` 文件相似。

### 使用多阶段构建

在 `Docker 17.05` 以上版本中，你可以使用 [多阶段构建](../image/multistage-builds.md) 来减少所构建镜像的大小。

### 避免安装不必要的包

为了降低复杂性、减少依赖、减小文件大小、节约构建时间，你应该避免安装任何不必要的包。例如，不要在数据库镜像中包含一个文本编辑器。

### 一个容器只运行一个进程

应该保证在一个容器中只运行一个进程。将多个应用解耦到不同容器中，保证了容器的横向扩展和复用。例如 web 应用应该包含三个容器：web应用、数据库、缓存。

如果容器互相依赖，你可以使用 [Docker 自定义网络](../network/linking.md) 来把这些容器连接起来。

### 镜像层数尽可能少

你需要在 `Dockerfile` 可读性（也包括长期的可维护性）和减少层数之间做一个平衡。

### 将多行参数排序

将多行参数按字母顺序排序（比如要安装多个包时）。这可以帮助你避免重复包含同一个包，更新包列表时也更容易。也便于 `PRs` 阅读和审查。建议在反斜杠符号 `\` 之前添加一个空格，以增加可读性。

下面是来自 `buildpack-deps` 镜像的例子：

```dockerfile
RUN apt-get update && apt-get install -y \
  bzr \
  cvs \
  git \
  mercurial \
  subversion
```

### 构建缓存

在镜像的构建过程中，Docker 会遍历 `Dockerfile` 文件中的指令，然后按顺序执行。在执行每条指令之前，Docker 都会在缓存中查找是否已经存在可重用的镜像，如果有就使用现存的镜像，不再重复创建。如果你不想在构建过程中使用缓存，你可以在 `docker build` 命令中使用 `--no-cache=true` 选项。

但是，如果你想在构建的过程中使用缓存，你得明白什么时候会，什么时候不会找到匹配的镜像，遵循的基本规则如下：

* 从一个基础镜像开始（`FROM` 指令指定），下一条指令将和该基础镜像的所有子镜像进行匹配，检查这些子镜像被创建时使用的指令是否和被检查的指令完全一样。如果不是，则缓存失效。
* 在大多数情况下，只需要简单地对比 `Dockerfile` 中的指令和子镜像。然而，有些指令需要更多的检查和解释。
* 对于 `ADD` 和 `COPY` 指令，镜像中对应文件的内容也会被检查，每个文件都会计算出一个校验和。文件的最后修改时间和最后访问时间不会纳入校验。在缓存的查找过程中，会将这些校验和和已存在镜像中的文件校验和进行对比。如果文件有任何改变，比如内容和元数据，则缓存失效。
* 除了 `ADD` 和 `COPY` 指令，缓存匹配过程不会查看临时容器中的文件来决定缓存是否匹配。例如，当执行完 `RUN apt-get -y update` 指令后，容器中一些文件被更新，但 Docker 不会检查这些文件。这种情况下，只有指令字符串本身被用来匹配缓存。

一旦缓存失效，所有后续的 `Dockerfile` 指令都将产生新的镜像，缓存不会被使用。

## Dockerfile 指令

下面针对 `Dockerfile` 中各种指令的最佳编写方式给出建议。

### FROM

尽可能使用当前官方仓库作为你构建镜像的基础。推荐使用 [Alpine](https://hub.docker.com/_/alpine/) 镜像，因为它被严格控制并保持最小尺寸（目前小于 5 MB），但它仍然是一个完整的发行版。

### LABEL

你可以给镜像添加标签来帮助组织镜像、记录许可信息、辅助自动化构建等。每个标签一行，由 `LABEL` 开头加上一个或多个标签对。下面的示例展示了各种不同的可能格式。`#` 开头的行是注释内容。

>注意：如果你的字符串中包含空格，必须将字符串放入引号中或者对空格使用转义。如果字符串内容本身就包含引号，必须对引号使用转义。

```dockerfile
# Set one or more individual labels
LABEL com.example.version="0.0.1-beta"
LABEL vendor="ACME Incorporated"
LABEL com.example.release-date="2015-02-12"
LABEL com.example.version.is-production=""
```

一个镜像可以包含多个标签，但建议将多个标签放入到一个 `LABEL` 指令中。

```dockerfile
# Set multiple labels at once, using line-continuation characters to break long lines
LABEL vendor=ACME\ Incorporated \
      com.example.is-beta= \
      com.example.is-production="" \
      com.example.version="0.0.1-beta" \
      com.example.release-date="2015-02-12"
```

关于标签可以接受的键值对，参考 [Understanding object labels](https://docs.docker.com/config/labels-custom-metadata/)。关于查询标签信息，参考 [Managing labels on objects](https://docs.docker.com/config/labels-custom-metadata/)。

### RUN

为了保持 `Dockerfile` 文件的可读性，可理解性，以及可维护性，建议将长的或复杂的 `RUN` 指令用反斜杠 `\` 分割成多行。

#### apt-get

`RUN` 指令最常见的用法是安装包用的 `apt-get`。因为 `RUN apt-get` 指令会安装包，所以有几个问题需要注意。

不要使用 `RUN apt-get upgrade` 或 `dist-upgrade`，因为许多基础镜像中的「必须」包不会在一个非特权容器中升级。如果基础镜像中的某个包过时了，你应该联系它的维护者。如果你确定某个特定的包，比如 `foo`，需要升级，使用 `apt-get install -y foo` 就行，该指令会自动升级 `foo` 包。

永远将 `RUN apt-get update` 和 `apt-get install` 组合成一条 `RUN` 声明，例如：

```docker
RUN apt-get update && apt-get install -y \
        package-bar \
        package-baz \
        package-foo
```

将 `apt-get update` 放在一条单独的 `RUN` 声明中会导致缓存问题以及后续的 `apt-get install` 失败。比如，假设你有一个 `Dockerfile` 文件：

```docker
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y curl
```

构建镜像后，所有的层都在 Docker 的缓存中。假设你后来又修改了其中的 `apt-get install` 添加了一个包：

```docker
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y curl nginx
```

Docker 发现修改后的 `RUN apt-get update` 指令和之前的完全一样。所以，`apt-get update` 不会执行，而是使用之前的缓存镜像。因为 `apt-get update` 没有运行，后面的 `apt-get install` 可能安装的是过时的 `curl` 和 `nginx` 版本。

使用 `RUN apt-get update && apt-get install -y` 可以确保你的 Dockerfiles 每次安装的都是包的最新的版本，而且这个过程不需要进一步的编码或额外干预。这项技术叫作 `cache busting`。你也可以显示指定一个包的版本号来达到 `cache-busting`，这就是所谓的固定版本，例如：

```docker
RUN apt-get update && apt-get install -y \
    package-bar \
    package-baz \
    package-foo=1.3.*
```

固定版本会迫使构建过程检索特定的版本，而不管缓存中有什么。这项技术也可以减少因所需包中未预料到的变化而导致的失败。

下面是一个 `RUN` 指令的示例模板，展示了所有关于 `apt-get` 的建议。

```docker
RUN apt-get update && apt-get install -y \
    aufs-tools \
    automake \
    build-essential \
    curl \
    dpkg-sig \
    libcap-dev \
    libsqlite3-dev \
    mercurial \
    reprepro \
    ruby1.9.1 \
    ruby1.9.1-dev \
    s3cmd=1.1.* \
 && rm -rf /var/lib/apt/lists/*
```

其中 `s3cmd` 指令指定了一个版本号 `1.1.*`。如果之前的镜像使用的是更旧的版本，指定新的版本会导致 `apt-get udpate` 缓存失效并确保安装的是新版本。

另外，清理掉 apt 缓存 `var/lib/apt/lists` 可以减小镜像大小。因为 `RUN` 指令的开头为 `apt-get udpate`，包缓存总是会在 `apt-get install` 之前刷新。

> 注意：官方的 Debian 和 Ubuntu 镜像会自动运行 apt-get clean，所以不需要显式的调用 apt-get clean。

### CMD

`CMD` 指令用于执行目标镜像中包含的软件，可以包含参数。`CMD` 大多数情况下都应该以 `CMD ["executable", "param1", "param2"...]` 的形式使用。因此，如果创建镜像的目的是为了部署某个服务(比如 `Apache`)，你可能会执行类似于 `CMD ["apache2", "-DFOREGROUND"]` 形式的命令。我们建议任何服务镜像都使用这种形式的命令。

多数情况下，`CMD` 都需要一个交互式的 `shell` (bash, Python, perl 等)，例如 `CMD ["perl", "-de0"]`，或者 `CMD ["PHP", "-a"]`。使用这种形式意味着，当你执行类似 `docker run -it python` 时，你会进入一个准备好的 `shell` 中。`CMD` 应该在极少的情况下才能以 `CMD ["param", "param"]` 的形式与 `ENTRYPOINT` 协同使用，除非你和你的镜像使用者都对 `ENTRYPOINT` 的工作方式十分熟悉。

### EXPOSE

`EXPOSE` 指令用于指定容器将要监听的端口。因此，你应该为你的应用程序使用常见的端口。例如，提供 `Apache` web 服务的镜像应该使用 `EXPOSE 80`，而提供 `MongoDB` 服务的镜像使用 `EXPOSE 27017`。

对于外部访问，用户可以在执行 `docker run` 时使用一个标志来指示如何将指定的端口映射到所选择的端口。

### ENV

为了方便新程序运行，你可以使用 `ENV` 来为容器中安装的程序更新 `PATH` 环境变量。例如使用 `ENV PATH /usr/local/nginx/bin:$PATH` 来确保 `CMD ["nginx"]` 能正确运行。

`ENV` 指令也可用于为你想要容器化的服务提供必要的环境变量，比如 Postgres 需要的 `PGDATA`。

最后，`ENV` 也能用于设置常见的版本号，比如下面的示例：

```docker
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
```

类似于程序中的常量，这种方法可以让你只需改变 `ENV` 指令来自动的改变容器中的软件版本。

### ADD 和 COPY

虽然 `ADD` 和 `COPY` 功能类似，但一般优先使用 `COPY`。因为它比 `ADD` 更透明。`COPY` 只支持简单将本地文件拷贝到容器中，而 `ADD` 有一些并不明显的功能（比如本地 tar 提取和远程 URL 支持）。因此，`ADD` 的最佳用例是将本地 tar 文件自动提取到镜像中，例如 `ADD rootfs.tar.xz`。

如果你的 `Dockerfile` 有多个步骤需要使用上下文中不同的文件。单独 `COPY` 每个文件，而不是一次性的 `COPY` 所有文件，这将保证每个步骤的构建缓存只在特定的文件变化时失效。例如：

```docker
COPY requirements.txt /tmp/
RUN pip install --requirement /tmp/requirements.txt
COPY . /tmp/
```

如果将 `COPY . /tmp/` 放置在 `RUN` 指令之前，只要 `.` 目录中任何一个文件变化，都会导致后续指令的缓存失效。

为了让镜像尽量小，最好不要使用 `ADD` 指令从远程 URL 获取包，而是使用 `curl` 和 `wget`。这样你可以在文件提取完之后删掉不再需要的文件来避免在镜像中额外添加一层。比如尽量避免下面的用法：

```docker
ADD http://example.com/big.tar.xz /usr/src/things/
RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
RUN make -C /usr/src/things all
```

而是应该使用下面这种方法：

```docker
RUN mkdir -p /usr/src/things \
    && curl -SL http://example.com/big.tar.xz \
    | tar -xJC /usr/src/things \
    && make -C /usr/src/things all
```

上面使用的管道操作，所以没有中间文件需要删除。

对于其他不需要 `ADD` 的自动提取功能的文件或目录，你应该使用 `COPY`。

### ENTRYPOINT

`ENTRYPOINT` 的最佳用处是设置镜像的主命令，允许将镜像当成命令本身来运行（用 `CMD` 提供默认选项）。

例如，下面的示例镜像提供了命令行工具 `s3cmd`:

```docker
ENTRYPOINT ["s3cmd"]
CMD ["--help"]
```

现在直接运行该镜像创建的容器会显示命令帮助：

```bash
$ docker run s3cmd
```

或者提供正确的参数来执行某个命令：

```bash
$ docker run s3cmd ls s3://mybucket
```

这样镜像名可以当成命令行的参考。

`ENTRYPOINT` 指令也可以结合一个辅助脚本使用，和前面命令行风格类似，即使启动工具需要不止一个步骤。

例如，`Postgres` 官方镜像使用下面的脚本作为 `ENTRYPOINT`：

```bash
#!/bin/bash
set -e
if [ "$1" = 'postgres' ]; then
    chown -R postgres "$PGDATA"
    if [ -z "$(ls -A "$PGDATA")" ]; then
        gosu postgres initdb
    fi
    exec gosu postgres "$@"
fi
exec "$@"
```

>注意：该脚本使用了 Bash 的内置命令 exec，所以最后运行的进程就是容器的 PID 为 1 的进程。这样，进程就可以接收到任何发送给容器的 Unix 信号了。

该辅助脚本被拷贝到容器，并在容器启动时通过 `ENTRYPOINT` 执行：

```docker
COPY ./docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
```

该脚本可以让用户用几种不同的方式和 `Postgres` 交互。

你可以很简单地启动 `Postgres`：

```bash
$ docker run postgres
```

也可以执行 `Postgres` 并传递参数：

```bash
$ docker run postgres postgres --help
```

最后，你还可以启动另外一个完全不同的工具，比如 `Bash`：

```bash
$ docker run --rm -it postgres bash
```

### VOLUME

`VOLUME` 指令用于暴露任何数据库存储文件，配置文件，或容器创建的文件和目录。强烈建议使用 `VOLUME` 来管理镜像中的可变部分和用户可以改变的部分。

### USER

如果某个服务不需要特权执行，建议使用 `USER` 指令切换到非 root 用户。先在 `Dockerfile` 中使用类似 `RUN groupadd -r postgres && useradd -r -g postgres postgres` 的指令创建用户和用户组。

>注意：在镜像中，用户和用户组每次被分配的 UID/GID 都是不确定的，下次重新构建镜像时被分配到的 UID/GID 可能会不一样。如果要依赖确定的 UID/GID，你应该显式的指定一个 UID/GID。

你应该避免使用 `sudo`，因为它不可预期的 TTY 和信号转发行为可能造成的问题比它能解决的问题还多。如果你真的需要和 `sudo` 类似的功能（例如，以 root 权限初始化某个守护进程，以非 root 权限执行它），你可以使用 [gosu](https://github.com/tianon/gosu)。

最后，为了减少层数和复杂度，避免频繁地使用 `USER` 来回切换用户。

### WORKDIR

为了清晰性和可靠性，你应该总是在 `WORKDIR` 中使用绝对路径。另外，你应该使用 `WORKDIR` 来替代类似于 `RUN cd ... && do-something` 的指令，后者难以阅读、排错和维护。

## 官方镜像示例

这些官方镜像的 Dockerfile 都是参考典范：https://github.com/docker-library/docs


# 如何调试 Docker

## 开启 Debug 模式

在 dockerd 配置文件 daemon.json（默认位于 /etc/docker/）中添加

```json
{
  "debug": true
}
```

重启守护进程。

```bash
$ sudo kill -SIGHUP $(pidof dockerd)
```

此时 dockerd 会在日志中输入更多信息供分析。

## 检查内核日志

```bash
$ sudo dmesg |grep dockerd
$ sudo dmesg |grep runc
```

## Docker 不响应时处理

可以杀死 dockerd 进程查看其堆栈调用情况。

```bash
$ sudo kill -SIGUSR1 $(pidof dockerd)
```

## 重置 Docker 本地数据

*注意，本操作会移除所有的 Docker 本地数据，包括镜像和容器等。*

```bash
$ sudo rm -rf /var/lib/docker
```

# 资源链接

## 官方网站

* Docker 官方主页：https://www.docker.com
* Docker 官方博客：https://www.docker.com/blog/
* Docker 官方文档：https://docs.docker.com/
* Docker Hub：https://hub.docker.com
* Docker 的源代码仓库：https://github.com/moby/moby
* Docker 路线图 https://github.com/docker/roadmap/projects
* Docker 发布版本历史：https://docs.docker.com/release-notes/
* Docker 常见问题：https://docs.docker.com/engine/faq/
* Docker 远端应用 API：https://docs.docker.com/develop/sdk/

## 实践参考

* Dockerfile 参考：https://docs.docker.com/engine/reference/builder/
* Dockerfile 最佳实践：https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/

## 技术交流

* Docker 邮件列表： https://groups.google.com/forum/#!forum/docker-user
* Docker 的 IRC 频道：https://chat.freenode.net#docker
* Docker 的 Twitter 主页：https://twitter.com/docker

## 其它

* Docker 的 StackOverflow 问答主页：https://stackoverflow.com/search?q=docker


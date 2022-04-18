# 如何在 Ubuntu 20.04 上安装和使用 Docker

## 介绍
[Docker](https://www.docker.com/) 是一个应用程序，它简化了在容器中管理应用程序进程的过程。 容器允许您在资源分离的进程中运行您的应用程序。 类似于 VM（虚拟机），但容器更便携、资源友好且依赖于主机操作系统。

关于 Docker 容器的各个组件的详细介绍，请参见 Docker 生态系统：[通用组件介绍](https://www.digitalocean.com/community/tutorials/the-docker-ecosystem-an-introduction-to-common-components)。

本教程在 Ubuntu 20.04 上安装和使用 Docker 社区版 (CE)。 安装 Docker 本身，使用容器和图像，并将图像推送到 Docker 存储库。

## 要求
要运行本教程，您需要具备以下内容：

- Ubuntu 20.04 服务器，sudo 非 root 用户，根据 [Ubuntu 20.04 初始服务器设置指南](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04)设置防火墙。

- 如果您想创建自己的镜像并将其推送到 Docker Hub，如 `应用 Docker 映像` 和 `推送 Docker 镜像` 所示，您需要有一个 Docker Hub 帐户。

## 安装 Docker

官方 Ubuntu 存储库中提供的 Docker 安装包可能不是最新版本。 从官方 Docker 存储库安装 Docker 以确保您获得最新版本。 为此，请添加新的包源，从 Docker 添加 GPG 密钥以确保下载有效，然后安装包。

> 经确认，至少在当前（2022年4月），还是需要添加 Docker 存储库的

```shell
## 首先，更新现有包的列表
sudo apt update

## 接下来，安装一些允许 apt 通过 HTTPS 使用该软件包的必备软件包
sudo apt install apt-transport-https ca-certificates curl software-properties-common

## 然后将来自官方 Docker 存储库的 GPG 密钥添加到您的系统。
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

## 将 Docker 存储库添加到您的 APT 源。
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"

## 然后使用添加的存储库中的 Docker 包更新包数据库。
sudo apt update

## 确保从 Docker 存储库而不是默认的 Ubuntu 存储库进行安装。
apt-cache policy docker-ce

## 您应该会看到类似于以下内容的输出，但 Docker 版本号可能不同。
Output of apt-cache policy docker-ce
docker-ce:
  Installed: (none)
  Candidate: 5:19.03.9~3-0~ubuntu-focal
  Version table:
     5:19.03.9~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages

## docker-ce 未安装，但安装候选者来自 Ubuntu 20.04 (focal) 上的 Docker 存储库。
## 最后，安装 Docker。
sudo apt install docker-ce

## 现在已经安装了 Docker，启动了守护进程，并且可以在启动时启动该进程。 确保它正在运行。
sudo systemctl status docker

## 输出如下所示，表明该服务处于活动状态并正在运行。
Output
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2020-05-19 17:00:41 UTC; 17s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 24321 (dockerd)
      Tasks: 8
     Memory: 46.4M
     CGroup: /system.slice/docker.service
             └─24321 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

安装 Docker 不仅提供了 Docker 服务（守护进程），还提供了 docker 命令行实用程序或 Docker 客户端。 在本教程的后面，我们将看到如何使用 docker 命令。

## 修改 Docker 权限

在没有 Sudo 的情况下运行 Docker 命令（可选）

默认情况下，docker 命令只能由 root 用户或在 Docker 安装过程中自动创建的 docker 组中的用户运行。 如果您尝试在没有 sudo 前缀或不属于 docker 组的情况下运行 docker 命令，您将看到类似于以下内容的输出：

```shell
Output
docker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?.
See 'docker run --help'.
```

```shell
## 为避免每次运行 docker 命令时都输入 sudo，请将您的用户名添加到 docker 组。
sudo usermod -aG docker ${USER}

## 要应用新的组成员资格，请退出服务器并重新登录，或键入：
su - ${USER}
## 系统将提示您输入用户密码以继续。

## 通过键入以下内容确认用户已添加到 docker 组：
id -nG
Output
sammy sudo docker

## 如果您需要将用户添加到您未登录的 docker 组，请使用以下命令显式声明该用户名：
sudo usermod -aG docker username
```

本文的其余部分假定您以 docker 组用户身份运行 docker 命令。 如果未选中，请在命令前加上 sudo。

## 使用 Docker 命令

要使用 docker，请传递一组选项和命令，然后是参数。 语法采用以下形式：

```shell
docker [option] [command] [arguments]
```

要查看所有可用的子命令，请键入：

```shell
docker
```

从 Docker 19 开始，可用子命令的完整列表包括：

```shell
Output
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes
```

要查看可用于特定命令的选项，请键入：

```shell
docker docker-subcommand --help
```

要查看有关 Docker 的系统范围信息，请键入：

```shell
docker info
```

## 使用 Docker 镜像

Docker 容器是从 Docker 镜像构建的。 默认情况下，Docker 从 Docker 项目背后的公司获取这些镜像：Docker Hub，由 Docker 管理的 Docker 注册表。 任何人都可以在 Docker Hub 上托管 Docker 映像，因此您需要的大多数应用程序和 Linux 发行版都可以托管该映像。

要查看是否可以从 Docker Hub 访问和下载映像，请键入：

```shell
docker run hello-world
```

输出显示 Docker 工作正常。

```shell
Output
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:6a65f928fb91fcfbc963f7aa6d57c8eeb426ad9a20c7ee045538ef34847f44f1
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

...
```

最初，Docker 无法在本地找到 hello-world 映像，因此它从其默认存储库 Docker Hub 下载了该映像。 下载镜像后，Docker 从镜像创建一个容器并在容器内运行应用程序以显示消息。

要搜索 Docker Hub 中可用的图像，请使用 docker 命令和 search 子命令。 例如，要搜索 Ubuntu 映像，请键入：

```shell
docker search ubuntu
```

该脚本爬取 Docker Hub 并返回名称与搜索字符串匹配的所有图像的列表。 在这种情况下，输出将是：

```shell
Output
NAME                                                      DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
ubuntu                                                    Ubuntu is a Debian-based Linux operating sys…   10908               [OK]
dorowu/ubuntu-desktop-lxde-vnc                            Docker image to provide HTML5 VNC interface …   428                                     [OK]
rastasheep/ubuntu-sshd                                    Dockerized SSH service, built on top of offi…   244                                     [OK]
consol/ubuntu-xfce-vnc                                    Ubuntu container with "headless" VNC session…   218                                     [OK]
ubuntu-upstart                                            Upstart is an event-based replacement for th…   108                 [OK]
ansible/ubuntu14.04-ansible                               Ubuntu 14.04 LTS with
...
```

OFFICIAL 栏中的 OK 表示项目背后的公司构建和支持的图像。 确定要使用的映像后，可以使用 pull 子命令将其下载到计算机。

通过运行以下命令将官方 ubuntu 映像下载到您的计算机：

```shell
docker pull ubuntu
```

您应该看到以下输出：

```shell
Output
Using default tag: latest
latest: Pulling from library/ubuntu
d51af753c3d3: Pull complete
fc878cd0a91c: Pull complete
6154df8ff988: Pull complete
fee5db0ff82f: Pull complete
Digest: sha256:747d2dbbaaee995098c9792d99bd333c6783ce56150d1b11e333bbceed5c54d7
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

下载图像后，您可以使用 run 子命令下载的图像运行容器。 正如我们在 hello-world 示例中看到的，如果使用 run 子命令运行 docker 时没有下载镜像，则 Docker 客户端首先下载镜像并使用它来运行容器。..

要在您的计算机上查看下载的图像，请键入：

```shell
docker images
```

输出如下所示：

```shell
Output
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              1d622ef86b13        3 weeks ago         73.9MB
hello-world         latest              bf756fb1ae65        4 months ago        13.3kB
```

如本教程后面所述，您可以修改用于运行容器的镜像并使用它来生成新镜像。 然后，您可以将其上传到 Docker Hub 或任何其他 Docker 注册表（技术上称为推送）。

## 运行 Docker 容器

您在上一步中运行的 hello-world 容器是发送测试消息后运行和退出的容器示例。 容器比这方便得多，并且可以交互。 毕竟，它们类似于 VM（虚拟机），只是资源友好。

例如，让我们使用最新的 Ubuntu 映像运行容器。 -i 和 -t 开关的组合允许交互式 shell 访问容器。

```shell
docker run -it ubuntu
```

命令提示符已被修改以反映您当前在容器内工作的事实并采用以下形式：

```shell
Output
root@d9b100f2f636:/#
```

记下命令提示符容器 ID。 在此示例中，它是 d9b100f2f636。 当您删除一个容器时，您稍后将需要它的容器 ID 来识别它。

您现在可以在容器内运行任何命令。 例如，让我们更新容器内的包数据库。 由于您在容器中以 root 用户身份操作，因此您不需要在命令前加上 sudo。

```shell
apt update
```

然后安装应用程序。 让我们安装 Node.js。

```shell
apt install nodejs
```

这将从官方 Ubuntu 存储库中将 Node.js 安装到您的容器中。 安装完成后，确保安装了 Node.js。

```shell
node -v
```

版本号显示在终端

```shell
Output
v10.19.0
```

在容器内所做的更改仅适用于该容器。

要退出容器，请在提示符处键入 exit。

## 管理Docker 容器

Dockerをしばらく使用すると、コンピュータ上に多くのアクティブ（実行中）および非アクティブのコンテナができます。**アクティブなコンテナ**を表示するには、次を使用します。

```shell
docker ps
```

您应该会看到类似于以下内容的输出：

```shell
Output
CONTAINER ID        IMAGE               COMMAND             CREATED    
```

在本教程中，我们从两个容器开始。 一个来自 hello-world 映像，另一个来自 ubuntu 映像。 这些容器不再运行，但它们仍然存在于系统中。

要查看所有容器（活动和非活动），请使用 -a 开关运行 docker ps。

```shell
docker ps -a
```

您应该会看到类似于以下内容的输出：

```shell
1c08a7a0d0e4        ubuntu              "/bin/bash"         2 minutes ago       Exited (0) 8 seconds ago                       quizzical_mcnulty
a707221a5f6c        hello-world         "/hello"            6 minutes ago       Exited (0) 6 minutes ago                       youthful_curie
```

通过 -l 开关查看您创建的最新容器。

```shell
docker ps -l
```

```shell
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
1c08a7a0d0e4        ubuntu              "/bin/bash"         2 minutes ago       Exited (0) 40 seconds ago                       quizzical_mcnulty
```

要启动已停止的容器，请使用 docker start，后跟容器 ID 或容器名称。 让我们启动一个 ID 为 1c08a7a0d0e4 的基于 Ubuntu 的容器。

```shell
docker start 1c08a7a0d0e4
```

容器将启动，您可以使用 docker ps 检查其状态。

```shell
Output
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
1c08a7a0d0e4        ubuntu              "/bin/bash"         3 minutes ago       Up 5 seconds                            quizzical_mcnulty
```

要停止正在运行的容器，请使用 docker stop 后跟容器 ID 或名称。 这一次，我们将使用 Docker 分配给容器的名称 quickzical_mcnulty。

```shell
docker stop quizzical_mcnulty
```

当您不再需要容器时，使用容器 ID 或名称通过 docker rm 命令将其删除。 使用 docker ps -a 命令查找并删除与 hello-world 映像关联的容器的容器 ID 或名称。

```shell
docker rm youthful_curie
```

您可以启动一个新容器并使用 --name 开关命名容器。 您还可以使用 -rm 开关创建一个在停止时自擦除的容器。 有关这些选项的更多信息，请参阅 docker run help 命令。

容器可以转换为可用于构建新容器的镜像。 让我们看看它是如何工作的。

## 应用 Docker 映像

**将容器中的更改应用到 Docker 映像**

启动 Docker 映像时，您可以像在 VM（虚拟机）中一样创建、修改和删除文件。 您所做的更改仅适用于该容器。 您可以启动和停止它，但如果您使用 docker rm 命令丢弃它，您的更改将永远丢失。

本节介绍如何将容器的状态保存为新的 Docker 映像。

当我在 Ubuntu 容器中安装 Node.js 时，容器从映像运行，但容器与我用来创建它的映像不同。 但是，您可能希望稍后重用此 Node.js 容器作为新图像的基础。

为此，请使用以下命令结构将更改提交到新的 Docker 映像实例。

```shell
docker commit -m "What you did to the image" -a "Author Name" container_id repository/new_image_name
```

-m 开关用于提交消息，帮助您和其他人了解您在 -a 用于指定作者时所做的更改。 container_id 是在本教程前面启动交互式 Docker 会话时记下的。 除非您在 Docker Hub 中创建额外的存储库，否则该存储库通常是 Docker Hub 用户名。

例如，对于容器 ID 为 d9b100f2f636 的用户 sammy，命令将是：

```shell
docker commit -m "added Node.js" -a "sammy" d9b100f2f636 sammy/ubuntu-nodejs
```

提交图像时，新图像会本地保存在您的计算机上。 在本教程的后面部分，您将学习如何将图像推送到 Docker 注册表（如 Docker Hub）以供其他人访问。

列出 Docker 映像时，您将看到新映像和派生它的旧映像。

```shell
docker images
```

输出如下所示：

```shell
Output
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
sammy/ubuntu-nodejs   latest              7c1f35226ca6        7 seconds ago       179MB
...
```

在此示例中，ubuntu-nodejs 是从 Docker Hub 的现有 ubuntu 映像派生的新映像。 大小差异反映了所做的更改。 在此示例中，更改是安装了 NodeJS。 如果您将来需要使用预安装了 Node.js 的 Ubuntu 运行容器，则可以使用新映像。

您还可以从 Dockerfile 构建映像。 这使您可以使用新映像自动安装软件。 但是，这超出了本教程的范围。

然后与其他人共享新图像，以便您可以从中创建容器。

## 推送 Docker 镜像

**将 Docker 镜像推送到 Docker 存储库**

从现有镜像创建新镜像后，下一个合乎逻辑的步骤是与您有限的朋友、Docker Hub 世界或您可以访问的任何其他 Docker 注册表共享它。 您需要一个帐户才能将映像推送到 Docker Hub 或任何其他 Docker 注册表。

本节介绍如何将 Docker 映像推送到 Docker Hub。 有关如何创建自己的私有 Docker 注册表的信息，请参阅如何在 Ubuntu 14.04 中设置私有 Docker 注册表。

要推送镜像，首先登录 Docker Hub。

```shell
docker login -u docker-registry-username
```

系统将提示您使用 Docker Hub 密码进行身份验证。 如果您指定正确的密码，则身份验证应该会成功。

> 注意：如果 Docker 注册中心名称与您创建镜像时的本地用户名不同，则需要使用注册中心名称标记该镜像。 在上一步的示例中：

```shell
docker tag sammy/ubuntu-nodejs docker-registry-username/ubuntu-nodejs
```

然后，您可以使用以下方式推送自己的图像：

```shell
docker push docker-registry-username/docker-image-name
```

要将 ubuntu-nodejs 映像推送到 sammy 存储库，命令如下所示：

```shell
docker push sammy/ubuntu-nodejs
```

此过程会上传图像，因此可能需要一些时间才能完成，但完成后，输出应如下所示：

```shell
Output
The push refers to a repository [docker.io/sammy/ubuntu-nodejs]
e3fbbfb44187: Pushed
5f70bf18a086: Pushed
a3b5c80a4eba: Pushed
7f18b442972b: Pushed
3ce512daaf78: Pushed
7aae4540b42d: Pushed

...
```

当您将图像推送到注册表时，它应该会出现在您帐户的仪表板中，如下图所示。

![New Docker image listing on Docker Hub](https://assets.digitalocean.com/articles/docker_1804/ec2vX3Z.png)

如果您尝试推送，仍然出现以下错误，则可能是您没有登录。

```shell
Output
The push refers to a repository [docker.io/sammy/ubuntu-nodejs]
e3fbbfb44187: Preparing
5f70bf18a086: Preparing
a3b5c80a4eba: Preparing
7f18b442972b: Preparing
3ce512daaf78: Preparing
7aae4540b42d: Waiting
unauthorized: authentication required
```

使用 docker login 登录，然后重复推送尝试。 然后检查它是否存在于 Docker Hub 存储库页面上。

现在您可以使用 docker pull sammy / ubuntu-nodejs 将映像带到您的新机器并运行新容器。

## 概括
在本教程中，您安装了 **Docker**，使用图像和容器，并将修改后的图像推送到 **Docker Hub**。 了解基础知识后，请查看 **DigitalOcean** 社区中的其他 **Docker** 教程。

## 补充

- **attach** 和 **exec** 的区别

    | attach                                  | exec                                                         |
    | :-------------------------------------- | :----------------------------------------------------------- |
    | 除非SHELL在容器内运行，否则您无法连接。 | 由于 PID = 1 的进程是在正在运行的容器中执行的，因此 shell 不需要在容器中运行。 |
    | 如果使用 exit 命令退出，容器将停止。    | 即使您使用 exit 命令退出，容器也不会停止                     |

## 相关资源

- https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04-ja
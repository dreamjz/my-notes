# Docker 简单教程

Docker是一个开源的应用容器引擎，容器是一个虚拟化的独立环境，开发者可以部署应用到一个可移植的容器中，然后发布到任何流行的linux机器上，也可以实现虚拟化

Docker 从 `1.13` 版本之后采用时间线的方式作为版本号，分为社区版 `CE` 和企业版 `EE`，社区版是免费提供给个人开发者和小型团体使用的，企业版会提供额外的收费服务，比如经过官方测试认证过的基础设施、容器、插件等

社区版按照 `stable` 和 `edge` 两种方式发布，每个季度更新 `stable` 版本，如 `17.06`，`17.09`；每个月份更新 `edge` 版本，如`17.09`，`17.10`

## 安装

本文操作环境为`Manjaro`,使用`pacman`进行安装：

```
sudo pacman -S docker
```

查看版本:

```sh
docker version
Client:
 Version:           20.10.9
 API version:       1.41
 Go version:        go1.17.1
 Git commit:        c2ea9bc90b
 Built:             Mon Oct  4 19:13:02 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true
```

启动服务：

```
sudo systemctl start docker.service
```

或设置开机启动：

```
sudo systemctl enable docker.service
```

运行`hello-world`镜像，验证安装是否成功

```sh
$ sudo docker run hello-world    
```

注意默认`docker`指令需要`root`用户权限，可以将当前用户加入`docker group`

查看当前用户群组：

```sh
$ cat /etc/group|grep docker 
docker:x:961:
```

若`docker`不存在则创建：

```sh
$ sudo groupadd docker
```

将当前用户加入`docker`群组

```sh
$ sudo usermod -aG docker $USER
```

以新的组登录

```sh
$ newgrp docker
```

测试`hello-world`

```sh
$ docker run hello-world
```

## 命令

```
$ docker --help

管理命令:
  container   管理容器
  image       管理镜像
  network     管理网络
命令：
  attach      介入到一个正在运行的容器
  build       根据 Dockerfile 构建一个镜像
  commit      根据容器的更改创建一个新的镜像
  cp          在本地文件系统与容器中复制 文件/文件夹
  create      创建一个新容器
  exec        在容器中执行一条命令
  images      列出镜像
  kill        杀死一个或多个正在运行的容器    
  logs        取得容器的日志
  pause       暂停一个或多个容器的所有进程
  ps          列出所有容器
  pull        拉取一个镜像或仓库到 registry
  push        推送一个镜像或仓库到 registry
  rename      重命名一个容器
  restart     重新启动一个或多个容器
  rm          删除一个或多个容器
  rmi         删除一个或多个镜像
  run         在一个新的容器中执行一条命令
  search      在 Docker Hub 中搜索镜像
  start       启动一个或多个已经停止运行的容器
  stats       显示一个容器的实时资源占用
  stop        停止一个或多个正在运行的容器
  tag         为镜像创建一个新的标签
  top         显示一个容器内的所有进程
  unpause     恢复一个或多个容器内所有被暂停的进程
```

## 容器使用

### 启动容器

使用`docker pull`获取`centos`镜像：

```
docker pull centos:latest
```

查看镜像：

```sh
$ docker images                                       
REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
hello-world   latest    feb5d9fea6a5   5 weeks ago   13.3kB
centos        latest    5d0da3dc9764   6 weeks ago   231MB
```

使用`centos`镜像启动容器：

```sh
docker run -it centos /bin/bash
```

- `i`:交互式操作
- `t`:终端
- `centos`:镜像名
- `/bin/bash`:命令，启动bash

执行`exit`可以退出容器，此时容器会停止

查看所有容器,`docker ps`查看正在运行的容器：

```sh
docker ps -a 
CONTAINER ID   IMAGE         COMMAND       CREATED          STATUS                      PORTS     NAMES
da34e8514a0e   centos        "/bin/bash"   14 minutes ago   Exited (0) 1 second ago               admiring_buck
d03b9c3fa042   hello-world   "/hello"      19 minutes ago   Exited (0) 19 minutes ago             magical_panini
105c030cdc8c   hello-world   "/hello"      41 minutes ago   Exited (0) 41 minutes ago             eager_haslett
```

启动，停止，重启已停止的容器：

```sh
$ docker start CONTAINER_ID
CONTAINER_ID
$ docker stop CONTAINER_ID
CONTAINER_ID
$ docker restart CONTAINER_ID
CONTAINER_ID
```

删除容器：

```sh
$ docker rm -f CONTAINER_ID
```

### 后台启动容器（daemon）

启动容器时使用`-d`,可以将容器作为守护进程（daemon）在后台运行，需要使用`attach`或`exec`指令进入容器

```sh
docker run -itd centos /bin/bash
```

#### `docker attach`

```sh
$ docker attach CONTAINER_ID
```

注意在退出容器后，容器将会停止

#### `docker exec`

```sh
$ docker exec -it CONTAINER_ID COMMAND 
```

### 导入和导出容器

使用`docker export`可将容器导出到本地快照

```sh
$ docker export cb50e17da895 > ./centos.tar  
```

导入容器快照作为镜像：

```sh
$ cat centos.tar|docker import - test/centos:local
 sha256:b4e46ee456b5b7ecb4cf35bb3b3c3d5c09230020a87c5f18f71a9004cadf2668
$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
test/centos   local     b4e46ee456b5   6 seconds ago   231MB
hello-world   latest    feb5d9fea6a5   5 weeks ago     13.3kB
centos        latest    5d0da3dc9764   6 weeks ago     231MB
```

## 镜像使用

### 设置镜像加速

此处使用网易云镜像服务：`https://hub-mirror.c.163.com`

请首先执行以下命令，查看是否在 `docker.service` 文件中配置过镜像地址

```
$ systemctl cat docker | grep '\-\-registry\-mirror'
```

如果该命令有输出，那么请执行 `$ systemctl cat docker` 查看 `ExecStart=` 出现的位置，修改对应的文件内容去掉 `--registry-mirror` 参数及其值，并按接下来的步骤进行配置。

如果以上命令没有任何输出，那么就可以在 `/etc/docker/daemon.json` 中写入如下内容（如果文件不存在请新建该文件）：

```json
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
}
```

> 注意，一定要保证该文件符合 json 规范，否则 Docker 将不能启动

之后重新启动服务:

```
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

### 查看镜像

使用`docker images`可以查看本地主机的镜像，信息说明：

- REPOSITORY ： 镜像的仓库
- TAG： 镜像标签
- CREATED： 镜像创建时间
- SIZE：镜像大小

同一仓库的镜像有不同的tag，可以使用`repository:tag`来指定某一镜像

获取镜像时，不指定tag，则默认为`latest`

### 搜索镜像

可以在[Dokcer Hub](https://hub.docker.com/)查找镜像，也可以使用`docker search`:

搜索`centos`:

```sh
$docker search centos                                                                       23:48:33 
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
centos                            The official build of CentOS.                   6833      [OK]       
ansible/centos7-ansible           Ansible on Centos7                              135                  [OK]
consol/centos-xfce-vnc            Centos container with "headless" VNC session…   132                  [OK]
jdeathe/centos-ssh                OpenSSH / Supervisor / EPEL/IUS/SCL Repos - …   121                  [OK]
centos/systemd                    systemd enabled base container.                 105                  [OK]
...
```

使用`docker pull`下载镜像，`docker rmi`移除镜像：

```sh
$ docker pull centos:5.11
...
$ docker rmi centos:5.11
```

### 创建镜像

当从docker镜像仓库下载的镜像不能满足时，有两种方式创建自定义的镜像：

- 从已有的仓库中提交镜像
- 使用Docker File创建镜像

#### 使用docker commit提交镜像

```sh
$ docker commit -m "a new image" -a "me" cb50e17da895 dreamjz/centos:local
sha256:7a7a770a75a350729b9c84e512c97515e10e3075266eca8e2294b83321bd6c26
$ docker images
REPOSITORY       TAG       IMAGE ID       CREATED         SIZE
dreamjz/centos   local     7a7a770a75a3   5 seconds ago   231MB
```

- `-m`：提交的描述信息
- `-a`: 指定镜像作者
- `cb50e17da895`: 容器ID
- `dreamjz/centos:local`: 目标镜像名

#### 使用Dokerfile构建镜像

DockerFile是一个用来构建镜像的文本文件，内容包含了一条条构建镜像所需的指令和说明

下面以定制一个nginx镜像为例

创建一个空目录`\$HOME/tmp/`,创建`Dockerfile`:

```sh
$ mkdir ~/tmp
$ cd tmp
$ touch Dockerfile
```

写入内容：

```dockerfile
FROM nginx
RUN echo 'Local nginx imamge' > /usr/share/nginx/html/index.html
```

- `FROM`:指定基础镜像，这里基于nginx镜像
- `RUN`:执行命令

使用`docker build`构建镜像：

```sh
docker build -t nginx:local .
```

- `-t`:指定镜像名
- `.`:表示以当前构建打包路径

查看镜像：

```
docker images
```

## 参考

1. [Docker 入门教程](https://github.com/jaywcjlove/docker-tutorial) github repo
2.  [How to fix docker: Got permission denied issue](https://stackoverflow.com/questions/48957195/how-to-fix-docker-got-permission-denied-issue) stackoverflow
3. [Docker 教程](http://www.runoob.com/docker/docker-tutorial.html) 教程


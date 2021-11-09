# Raspberry Pi使用Docker部署静态blog

之前仅在树莓派上安装nginx并部署了blog，但是如果nginx的配置有问题或者出现错误不方便解决（所有东西都在一台主机上）。

现在使用docker容器来部署nginx，如果配置错误可以删除容器推到重来，还可以部署多个服务，非常的便利

部署之前先停止已经运行的nginx

```sh
$ sudo nginx -s stop  
```

## 安装Docker

这里不要直接用apt,apt里默认的包是docker的GUI版本

这里直接使用官网的安装脚本，其他的安装方式参见[Docker官网手册](https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script)

```sh
 $ curl -fsSL https://get.docker.com -o get-docker.sh
 $ sudo sh get-docker.sh
```

查看docker版本

```sh
$docker -v                                                                             
Docker version 20.10.10, build b485636
```

默认docker需要root权限运行，想要以非root命令运行可以参考[docker安装](https://dreamjz.github.io/zh/docker/docker-tutorial/get-started.html#%E5%AE%89%E8%A3%85)

设置docker开机自启并启动docker

```sh
$ systemctl start docker 
$ systemctl enable docker
```

## 获取镜像

### 镜像加速

此处使用网易云镜像服务：`https://hub-mirror.c.163.com`

请首先执行以下命令，查看是否在 `docker.service` 文件中配置过镜像地址

```sh
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

之后重新启动服务:

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

### 获取Nginx官方镜像

```sh
$ sudo docker pull nginx
```

查看镜像：

```sh
$ sudo docker images
```

## 部署nginx

启动容器:

```sh
$ sudo docker container run \
-d \
--rm \
-p 8897:80 \
--name mynginx \
nginx
```

 查看是否启动成功

```sh
$ sudo docker ps 
```

进入之前在本机部署的nginx资源文件夹`~/nginx`,文件结构如下（之前部署时资源已经放在`/app/blog/dist`）：

```sh
nginx
├── app
│   └── blog
│       └── dist
└── logs
```

复制nginx配置目录：

```sh
$ sudo docker container cp mynginx:/etc/nginx .
```

更改目录名：

```sh
$ mv nginx conf
```

停止之前启动的容器,停止后会被自动删除（`--rm`的作用）:

```sh
$ sudo docker stop mynginx
```

启动nginx:

```sh
$ sudo docker run \
    -d \
    -p 80:80 \
    --name my-nginx \
    -v /home/pi/nginx/app/blog/dist:/usr/share//nginx/html \
    -v /home/pi/nginx/logs:/var/log/nginx \
    -v /home/pi/nginx/conf:/etc/nginx \
    nginx  
```

查看是否启动成功：

```sh
$ sudo docker ps 
```

现在直接在浏览器访问树莓派的ip，可以直接进入blog首页了

---
本文采用[CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)许可

## 参考

1.  [Install Docker Engine on Debian](https://docs.docker.com/engine/install/debian/) docker manuals

2. [Docker 部署 vue 项目](https://juejin.cn/post/6844903837774397447#heading-0) 稀土掘金

3. [Docker 部署Nginx](https://dreamjz.github.io/zh/docker/docker-tutorial/docker-nginx.html) 今朝のブログ

4. [docker command not found even though installed with apt-get](https://stackoverflow.com/questions/30379381/docker-command-not-found-even-though-installed-with-apt-get) stackoverflow


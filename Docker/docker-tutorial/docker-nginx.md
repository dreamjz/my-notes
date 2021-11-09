# Docker 部署Nginx

Nginx 是一款面向性能设计的 HTTP 服务器，能反向代理 HTTP，HTTPS 和邮件相关(SMTP，POP3，IMAP)的协议链接。并且提供了负载均衡以及 HTTP 缓存。它的设计充分使用异步事件模型，削减上下文调度的开销，提高服务器并发能力。采用了模块化设计，提供了丰富模块的第三方模块

本文介绍docker如何部署nginx

## 快速开始

在docker hub查找镜像

```sh
$ docker search nginx 
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                             Official build of Nginx.                        15728     [OK]       
jwilder/nginx-proxy               Automated Nginx reverse proxy for docker con…   2088                 [OK]
richarvey/nginx-php-fpm           Container running Nginx + PHP-FPM capable of…   818                  [OK]
......
```

这里使用官方镜像，下载：

```sh
$ docker pull nginx
```

启动容器

```sh
$ docker container run --name my-nginx -d --rm -p 8080:80 nginx /bin/bash
24167b75db40bb32a59699f4c799b1c5009cad5853712caa7fe68ddafb85125d
```

- `--name`:指定容器名
- `-p`: host_port:container_port,主机端口和容器端口映射
- `--rm`:容器停止后文件自动删除

这里的`docker container run` 和`docker run`是一样的，`docker container run`为docker 1.13进行CLI重构时新增

查看容器日志：

```sh
docker logs my-nginx
```

在浏览器访问`localhost:8080`,可以看到nginx的初始页面了

## 挂载本地文件

资源文件都放在容器里面，不方便修改，可以将本地文件挂载到容器中

在本地创建目录：

```sh
$ mkdir -p ~/nginx/app
```

在`app`里创建一个html文件：

```sh
$ echo '<h1>Hello World</h1>' > index.html
```

启动容器，这次加上`-v host_dir:container_dir`,挂载本地文件：

```
docker container run \
-d \
-p 8080:80 \
--rm \
--name mynginx \
--volume $PWD:/usr/share/nginx/html \
nginx
```

在浏览器访问`localhost:8080`,可以看到Hello World了

## 拷贝配置 

将容器里面的Nginx配置文件拷贝到本地：

```sh
$ docker container cp mynginx:/etc/nginx/ ~/nginx
```

名称改为`conf`:

```sh
$ mv nginx conf
```

停止`mynginx`挂载配置文件：

```sh
$ docker stop mynginx
mynginx
$ docker container run \
 -d \
 -p 8080:80 \
 --rm \
 --name mynginx \
 -v $HOME/nginx/conf:/etc/nginx \
 -v $HOME/nginx/app:/usr/share/nginx/html \ 
 nginx
```

在浏览器访问`localhost:8080`,可以看到Hello World了

## 参考

1. [Docker部署nginx](https://github.com/jaywcjlove/docker-tutorial/blob/master/docker/nginx.md) github repo

2. [nginx-tutorial](https://github.com/jaywcjlove/nginx-tutorial) github repo

3. [Nginx 容器教程](https://www.ruanyifeng.com/blog/2018/02/nginx-docker.html)  阮一峰

4. [docker docs](https://docs.docker.com/engine/reference/commandline/run/) docker docs/docker run 

5. [What is the difference between docker run and docker container run](https://stackoverflow.com/questions/51247609/what-is-the-difference-between-docker-run-and-docker-container-run) stackoverflow

6.  [Introducing Docker 1.13](https://www.docker.com/blog/whats-new-in-docker-1-13/)


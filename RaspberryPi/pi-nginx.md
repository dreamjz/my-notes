# Raspberry Pi 部署静态blog

入手了一个树莓派4B，想要将其变成小型服务器，同时也把自己的博客部署上去,环境：

- Raspberry Pi : 4B+/8G
- OS: Raspios-64bit
- Nginx

## 配置静态IP

由于树莓派连接家里的路由器，默认使用DHCP[^1]

要做为服务器的话，设置静态方便配置连接

修改`cat /etc/dhcpcd.conf`:

```
interface wlan0
static ip_address=192.168.***.***/24 # IP地址
static routers=192.168.***.*** # 路由器地址
static domain_name=192.168.***.*** # dns解析地址
```

## 安装Nginx

快速学习使用Nginx可以看这篇[Nginx教程](../../../nginx/nginx.md)

可以使用apt安装：

```sh
sudo apt install nginx
```

在$HOME目录下创建`nginx`目录，目录结构如下：

```
nginx
├── app
│   └── blog
├── logs
│   ├── access.log
│   ├── error.log
│   ├── info.log
│   └── notice.log
├── nginx.conf
└── start-nginx.sh
```

- app: 存放静态资源，打包好的静态网页放在此处,需要在配置文件中设置

- logs：nginx日志文件

- nginx.conf :  nginx 配置文件

- start-nginx.sh ： 启动脚本

  ```sh
  #!/bin/zsh                                                        
  CONFIG=$HOME/nginx/nginx.conf
  echo "Config path: "$CONFIG" "
  #  如果启动前已经启动nginx并记录下pid文件，会kill指定进程
  sudo nginx -s stop
  
  # 测试配置文件语法正确性
  sudo nginx -t -c $CONFIG
  
  # 显示版本信息
  sudo nginx -v
  
  # 按照指定配置去启动nginx
  sudo nginx -c $CONFIG
  ```

### 修改配置文件

 `$HOME/nginx/nginx.conf`:

```
#运行用户                                                                                                                                                                                                         
user pi;
    
#启动进程,通常设置成和cpu的数量相等
worker_processes  1;

#全局错误日志
error_log  /home/username/nginx/logs/error.log;
error_log  /home/username/nginx/logs/notice.log  notice;
error_log  /home/username/nginx/logs/info.log  info;

#PID文件，记录当前启动的nginx的进程ID
pid        /run/nginx.pid;

#工作模式及连接数上限
events {
    worker_connections 1024;    #单个后台worker process进程的最大并发链接数
}


http {
    #设定mime类型(邮件支持类型),类型由mime.types文件定义
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;                                                                                                                                                                       

    #设定日志
    log_format  main  '[$remote_addr] - [$remote_user] [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log    /home/username/nginx/logs/access.log main;
    rewrite_log     on; 

    #sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用，
    #必须设为 on,如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime.
    sendfile        on; 
    #tcp_nopush     on;

    #连接超时时间
    keepalive_timeout  120;
    tcp_nodelay        on; 

   #gzip压缩开关
   #gzip  on;
server {
    listen 80;
    location / {
    	# 设置跨域
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Credentials' 'true';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
        root /home/username/nginx/app/blog/dist;
        index index.html;
    }
}

```

## 上传静态资源

此处以vue为例,将打包好的`dist`目录压缩上传至服务器，将其解压至`$HOME/nginx/app/blog/`即可

在客户端浏览器总输入服务器IP地址即可访问：

![](./image/blog.png)


---
本文采用[CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)许可

## 参考

1. [vue部署至nginx](https://segmentfault.com/a/1190000038672615) lxcan blog
2. [动态主机设置协议](https://zh.wikipedia.org/wiki/%E5%8A%A8%E6%80%81%E4%B8%BB%E6%9C%BA%E8%AE%BE%E7%BD%AE%E5%8D%8F%E8%AE%AE) wiki
3. [Nginx 极简教程](https://github.com/dunwu/nginx-tutorial) github repo



[^1]: **动态主机设置协议**（英语：**D**ynamic **H**ost **C**onfiguration **P**rotocol，缩写：**DHCP**），又称**动态主机组态协定**，是一个用于[IP](https://zh.wikipedia.org/wiki/网际协议)网络的[网络协议](https://zh.wikipedia.org/wiki/网络协议)，位于[OSI模型](https://zh.wikipedia.org/wiki/OSI模型)的[应用层](https://zh.wikipedia.org/wiki/应用层)，使用[UDP](https://zh.wikipedia.org/wiki/用户数据报协议)协议工作
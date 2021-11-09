# RspiOS 配置使用 Clash

## 环境

平常开发使用manjaro和windows，配置clash一直使用的GUI客户端，没有去折腾使用底层的clash（主要是懒...）

刚好最近入手了一个树莓派，这次就在树莓派上研究下clash的使用（主要是想整活）

本文环境：

- Raspi OS - 64bit

## 安装 

在[clash](https://github.com/Dreamacro/clash) github 仓库中选择 [clash-linux-armv8-v1.7.1.gz](https://github.com/Dreamacro/clash/releases/download/v1.7.1/clash-linux-armv8-v1.7.1.gz) 下载（因为树莓派是ARM架构，本文使用的系统为64bit，需选择armv8）

直接使用`wget`下载：

```sh
wget -O clash-linux-armv8-v1.7.1.gz https://github.com/Dreamacro/clash/releases/download/v1.7.1/clash-linux-armv8-v1.7.1.gz
```

解压并添加执行权限：

```sh
gzip -f clash-linux-armv8-v1.7.1.gz -d 
chmod +x clash-linux-armv8-v1.7.1
```

启动clash，会自动生成默认配置`config.yml`和下载全球IP地址库`Country.mmdb`

##  配置

将服务商提供的配置文件写到`config.yml`即可

还可以使用[Web工具](：http://clash.razord.top/#/proxies[)

## 环境变量

设置环境变量：

```sh
export http_proxy=http://host:port
export https_proxy=$http_proxy
```

## 启动

再次启动clash，即可使用

可以编写启动脚本在后台执行:

```sh
#!/bin/zsh
nohup $HOME/MyApps/clash/clash-linux-armv8-v1.7.1 > $HOME/MyApps/clash/log/clash.log 2>&1 &
```

## 参考

1. [clash](https://github.com/Dreamacro/clash) github repo

2. [ubuntu 20.04 配置使用 clash for linux](http://www.ptbird.cn/ubuntu-2004-clash-for-linux.html) postbird blog

3. [nohup 详解](https://www.cnblogs.com/jinxiao-pu/p/9131057.html) jinxiao-pu blog

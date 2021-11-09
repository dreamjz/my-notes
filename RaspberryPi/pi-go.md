#  Raspberry Pi 搭建 golang 开发环境

​	本文主要介绍如何在树莓派搭建golang开发环境，涉及硬件及软件版本：

- Raspberry Pi : 4B+/8G
- OS: Raspios-64bit
- Golang: 1.15

## 安装 Raspi OS

首先为树莓派安装操作系统，树莓派可以采用官方OS或三方系统（ubuntu，manjaro，centos等），这里我们使用官方的[Raspi OS 64-bit](https://downloads.raspberrypi.org/raspios_arm64/images/)

注意正式版Raspi OS位32bit，64bit版本仅有测试版(需自行下载)

安装软件时 appimage 类型是无法执行的(树莓派是ARM架构的，而appimage基于x86)

### 使用 [Raspi imager](https://www.raspberrypi.com/software/)

​	进入官网后下载[Raspi imager](https://www.raspberrypi.com/software/)，安装后启动：

![](./image/raspi-imager.png)

选择下载好的系统镜像(本文使用镜像为：2021-05-07-raspios-buster-arm64.zip)，之后选择存储卡进行写入，等待片刻

## 网络配置(可选)

在下载和更新软件包之前，可以配置代理（拯救巨慢的下载速度....）

 若想用clash可以参照文章[RaspiOS 使用clash](../clash/clash-linux.md)

## 更改软件源

RaspiOS默认软件源为`Debian buster`,软件版本较低，这里改为`Debian Bullseye` 

为后续的vim插件做准备（YCM需要vim8.2）

`/etc/apt/sources.list`中使用bullseye同时换成清华源

```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-backports main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-backports main contrib non-free

deb https://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free
```

执行update和upgrade 

```sh
sudo apt upadte
sudo apt upgrade
```

## 安装 zsh & OMZ(oh-my-zsh)

### zsh

查看当前系统有哪些shell:

```sh
$ cat /etc/shells
...
/bin/sh
/bin/bash
/usr/bin/bash
/bin/rbash
/usr/bin/rbash
/bin/dash
/usr/bin/dash
```

没有则安装zsh：

```sh
$ sudo apt install zsh
```

查看和更改默认shell：

```sh
$ echo $SHELL #当前默认shell
$ chsh -s /bin/zsh # 修改默认shell
```

重新登录即可生效

### omz

执行官网安装脚本：

```sh
$ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

在`~/.zshrc`中修改主题：

```sh
ZSH_THEME="random" # 随机主题
```

## 安装golang

这里使用apt安装，也可以手动安装

```sh
sudo apt install golang
```

安装gopls

```sh
GO111MODULE=on go get golang.org/x/tools/gopls@latest
```

## 安装 vim

使用apt安装：

```sh
sudo apt intall vim-nox
```

注意：这里安装`vim-nox`，默认的vim没有python3支持，无法安装后续插件

后续的`vim-go`,`spf-vim`及插件的安装参考[spf-vim 安装及插件配置](../vim/spf13-vim-plugin.md)

---
本文采用[CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)许可

## 参考

1. respi os](https://www.raspberrypi.com/software/)  Raspberry Pi official site
2. [Mac、Linux 安装zsh & ohmyzsh](https://segmentfault.com/a/1190000013857738)  segmentfault/mayihetu
3. [Appimage install problem - cannot execute binary file: Exec format error](https://www.reddit.com/r/Crostini/comments/c76e0b/appimage_install_problem_cannot_execute_binary/) reddit
4. [oh-my-zsh](https://ohmyz.sh/) omz official site
5. [oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh) omz github repo
6. [debian tsinghua mirror](https://mirrors.tuna.tsinghua.edu.cn/help/debian/) 清华镜像
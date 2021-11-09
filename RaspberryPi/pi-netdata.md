# Raspberry Pi 安装 NetData 轻量服务器性能监控工具

在树莓派上搭建了Blog之后，想要在监控下服务器的各项参数（一开始只是想看看CPU，内存和网速的），google下发现了一个好用的工具NetData

## 简介

**[Netdata](https://github.com/netdata/netdata)** 是一个用于系统和应用的分布式实时性能和健康监控工具。它提供了对系统中实时发生的所有事情的全面检测。你可以在高度互动的 Web 仪表板中查看结果。使用 Netdata，你可以清楚地了解现在发生的事情，以及之前系统和应用中发生的事情。你无需成为专家即可在 Linux 系统中部署此工具。NetData 开箱即用，零配置、零依赖。只需安装它然后坐等，之后 NetData 将负责其余部分。

它有自己的内置 Web 服务器，以图形形式显示结果。NetData 非常快速高效，安装后可立即开始分析系统性能。它是用 C 编程语言编写的，所以它非常轻量。它占用的单核 CPU 使用率不到 3％，内存占用 10-15MB。我们可以轻松地在任何现有网页上嵌入图表，并且它还有一个插件 API，以便你可以监控任何应用。

## 特点

![](./image/NETDATA.gif)

- interactive bootstrap dashboards, 酷炫
- 所有请求每个metreic都在0.5ms内响应，即便是一台烂机器
- 非常高效，每秒采集数千个指标，但仅占cpu单核1%，少量MB的内存以及完全没有磁盘IO
- 提供复杂的、各种类型的告警，支持动态阈值、告警模板、多种通知方式等
- 可扩展，使用自带的插件API（比如bash, python, perl, node.js, java, go, ruby等）来收集任何可以衡量的数据
- 零配置：安装后netdata会自动的监测一切
- 零依赖：netdata有自己的web server， 提供静态web文件和web API 零维护：只管跑上！
- 支撑多种时间序列后端服务，比如graphite, opentsdb, prometheus, json document DBs

###  NetData在Linux中的监控列表

- CPU 使用率
- RAM 使用率
- 交换内存使用率
- 内核内存使用率
- 硬盘及其使用率
- 网络接口
- IPtables
- Netfilter
- DDoS 保护
- 进程
- 应用
- NFS 服务器
- Web 服务器 （Apache 和 Nginx）
- 数据库服务器 （MySQL），
- DHCP 服务器
- DNS 服务器
- 电子邮件服务
- 代理服务器
- Tomcat
- PHP
- SNP 设备

## 部署

在Rspbian 下直接使用apt安装即可：

```sh
sudo apt install netdata
```

查看配置文件`/etc/netdata/netdata.conf`：

```
# NetData Configuration

# The current full configuration can be retrieved from the running
# server at the URL
#
#   http://localhost:19999/netdata.conf
#
# for example:
#
#   wget -O /etc/netdata/netdata.conf http://localhost:19999/netdata.conf
#

[global]
   run as user = netdata
   web files owner = root
   web files group = root
   # Netdata is not designed to be exposed to potentially hostile
   # networks. See https://github.com/netdata/netdata/issues/164
   bind socket to IP = 127.0.0.1
   port = 19901           
```

可以修改默认的19901端口为自己想要的端口

## Nngix配置



## 参考

1. [netdata](https://github.com/netdata/netdata) github repo
2. [超详细的NetData-轻量的性能监控工具安装教程](https://www.gitiu.com/journal/netdata-install/)  gitiu blog


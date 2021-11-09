# Problem

```shell
正在读取软件包列表... 完成
正在分析软件包的依赖关系树       
正在读取状态信息... 完成       
下列软件包将被【卸载】：
  jd-gui
升级了 0 个软件包，新安装了 0 个软件包，要卸载 1 个软件包，有 1 个软件包未被升级。
有 1 个软件包没有被完全安装或卸载。
解压缩后将会空出 1,533 kB 的空间。
您希望继续执行吗？ [Y/n] Y
(正在读取数据库 ... 系统当前共安装有 326530 个文件和目录。)
正在卸载 jd-gui (1.6.6-0) ...
xdg-desktop-menu: No writable system menu directory found.
dpkg: 处理软件包 jd-gui (--remove)时出错：
 已安装 jd-gui 软件包 pre-removal 脚本 子进程返回错误状态 3
在处理时有错误发生：
 jd-gui
E: Sub-process /usr/bin/dpkg returned an error code (1)

```

# Solution

```shell
sudo mkdir /usr/share/desktop-directories/
```

https://askubuntu.com/questions/405800/installation-problem-xdg-desktop-menu-no-writable-system-menu-directory-found
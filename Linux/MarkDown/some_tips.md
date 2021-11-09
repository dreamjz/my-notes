# Some Tips

记录一些Linux小技巧

## 1.1 Tree 命令列出文件结构

安装tree,以manjaro为例:

```sh
sudo pacman -S tree
```

常用参数:

```
tree:显示目录树 
-a：显示所有文件目录
-d:只显示目录 
-L:选择显示的目录深度 
1：只显示一层深度，即不递归子目录
```

使用:

```sh
tree -d
.
├── A
├── B
├── C
│   └── A
├── D
├── gitemp
├── grep_test
└── sort_test
```




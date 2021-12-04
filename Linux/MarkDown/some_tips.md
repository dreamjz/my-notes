# Tips

记录一些Linux小技巧

## 1. Tree 命令列出文件结构

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

## 2. AUR 一个或多个文件没有通过有效性检查

1. 编辑pkgbuild文件（可以用pamac里搜索对应的aur软件进入软件详情界面，点击构建文件，直接修改）
2. 找到对应的验证部分，把里面的验证的码修改为SKIP，SKIP一定要是大写（md5sums sha1sums sha256sums sha224sums, sha384sums, sha512sums b2sums）
3. 点击构建就能够成功了




---
tags: [Linux, 权限, 用户]
---

# 权限、用户与 sudo

> [!abstract] 本页目录
> - [[#读懂权限]]
> - [[#chmod：修改权限]]
> - [[#所有者和组]]
> - [[#sudo 与 root]]
> - [[#umask]]
> - [[00-Linux基础学习地图|← 返回总目录]]

## 读懂权限

```text
-rwxr-xr-- 1 alice dev 1234 Aug 8 10:00 script.sh
│└┬┘└┬┘└┬┘   │    │
│ │   │  │    │    └─ 所属组
│ │   │  │    └─ 所有者
│ │   │  └─ 其他人：r--
│ │   └─ 所属组：r-x
│ └─ 所有者：rwx
└─ 文件类型
```

- `r`：read，读
- `w`：write，写
- `x`：execute，执行
- `-`：没有该权限

目录权限与文件略有不同：

- 目录的 `r`：列出名称
- 目录的 `w`：创建、删除、改名目录项
- 目录的 `x`：进入并访问其中对象

删除文件主要看**父目录**权限，不只看文件自身权限。

## chmod：修改权限

符号写法：

```bash
chmod u+x script.sh       # 所有者增加执行权限
chmod g-w file.txt        # 组移除写权限
chmod o-r secret.txt      # 其他人移除读权限
chmod a+r readme.txt      # 所有人增加读权限
```

数字写法：`r=4`、`w=2`、`x=1`，相加得到每组权限。

```bash
chmod 755 script.sh       # rwx r-x r-x
chmod 644 file.txt        # rw- r-- r--
chmod 600 private.txt     # rw- --- ---
```

## 所有者和组

```bash
id                        # 查看当前用户 UID、GID 和组
whoami                    # 当前用户名
groups                    # 所属组
chown alice file.txt      # 修改所有者，通常需要管理员权限
chown alice:dev file.txt  # 同时修改所有者和组
chgrp dev file.txt        # 修改所属组
```

## sudo 与 root

- `root` 是超级用户，几乎不受普通权限限制。
- `sudo command` 以授权身份执行一条命令。
- `sudo` 不是“遇到报错就加”的万能前缀。

```bash
sudo -v                 # 刷新 sudo 凭据
sudo apt update         # 示例：只用管理员权限更新一次软件包索引
sudoedit /etc/hosts     # 更安全地编辑管理员文件
```

> [!warning]
> 用 `sudo` 创建的文件可能归 root 所有，导致之后普通用户无法修改。不要把密码写进命令或脚本。

## umask

`umask` 决定新文件默认要去掉哪些权限。常见 `022` 通常产生文件 `644`、目录 `755`；文件默认一般不自动带执行位。

关联：[[10-Bash脚本入门]] · [[12-高频命令速查表]]

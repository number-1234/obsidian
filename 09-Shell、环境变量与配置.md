---
tags: [Linux, Bash, 环境变量]
---

# Shell、环境变量与配置

> [!abstract] 本页目录
> - [[#Shell 是什么]]
> - [[#变量]]
> - [[#常见环境变量]]
> - [[#PATH]]
> - [[#Bash 配置文件]]
> - [[#alias 与 function]]
> - [[00-Linux基础学习地图|← 返回总目录]]

## Shell 是什么

Shell 负责读取命令、展开变量和通配符、处理重定向与管道，然后启动相应程序。

```bash
echo "$SHELL"            # 登录 Shell
ps -p $$ -o comm=        # 当前 Shell 进程
```

## 变量

```bash
name='Linux'             # 等号两边不能有空格
echo "$name"
unset name
```

默认情况下，Shell 变量不会自动传给子进程；导出后成为环境变量：

```bash
export EDITOR=nano
printenv EDITOR
env | sort
```

## 常见环境变量

| 变量 | 用途 |
|---|---|
| `HOME` | 家目录 |
| `USER` | 当前用户名 |
| `SHELL` | 登录 Shell 路径 |
| `PATH` | 查找可执行命令的目录列表 |
| `PWD` | 当前目录 |
| `OLDPWD` | 上一次目录 |
| `LANG` | 语言和区域设置 |
| `EDITOR` | 默认文本编辑器（约定） |

## PATH

输入 `python` 时，Shell 会按 `PATH` 中从左到右的目录查找可执行文件。

```bash
echo "$PATH" | tr ':' '\n'
command -v python
export PATH="$HOME/.local/bin:$PATH"
```

通常把个人目录加在 PATH 前面，但不要随便加入当前目录 `.`，以免误运行恶意同名程序。

## Bash 配置文件

常见文件：

- `~/.bashrc`：交互式非登录 Bash 常读取
- `~/.bash_profile` 或 `~/.profile`：登录时常读取
- `/etc/profile`：系统级登录配置

不同发行版和启动方式有差异。修改后可新开终端，或在确认内容安全后运行：

```bash
source ~/.bashrc
```

## alias 与 function

```bash
alias ll='ls -lah'
alias grep='grep --color=auto'

mkcd() {
  mkdir -p "$1" && cd "$1"
}
```

把常用定义写进 `~/.bashrc` 才会长期生效。不要给危险操作设置让人放松警惕的别名。

关联：[[02-终端、命令格式与帮助]] · [[10-Bash脚本入门]]

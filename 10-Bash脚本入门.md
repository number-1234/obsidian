---
tags: [Linux, Bash, 脚本]
---

# Bash 脚本入门

> [!abstract] 本页目录
> - [[#第一个脚本]]
> - [[#参数]]
> - [[#条件判断]]
> - [[#循环]]
> - [[#函数]]
> - [[#更稳妥的开头]]
> - [[#调试与检查]]
> - [[00-Linux基础学习地图|← 返回总目录]]

## 第一个脚本

创建 `hello.sh`：

```bash
#!/usr/bin/env bash

name=${1:-朋友}
echo "你好，$name！"
```

运行方式：

```bash
bash hello.sh Alice
# 或
chmod u+x hello.sh
./hello.sh Alice
```

`#!` 叫 shebang，用于指定直接执行脚本时的解释器。

## 参数

| 写法 | 含义 |
|---|---|
| `$0` | 脚本名 |
| `$1`、`$2` | 第 1、2 个参数 |
| `$#` | 参数数量 |
| `"$@"` | 所有参数，逐个保留边界 |
| `$?` | 上一条命令退出状态 |
| `$$` | 当前 Shell PID |

引用变量时通常写 `"$variable"`，避免空格和通配符导致意外拆分。

## 条件判断

```bash
if [[ -f "$1" ]]; then
  echo "这是普通文件"
elif [[ -d "$1" ]]; then
  echo "这是目录"
else
  echo "路径不存在"
fi
```

常见判断：

- `-e path`：存在
- `-f path`：普通文件
- `-d path`：目录
- `-r/-w/-x path`：可读/可写/可执行
- `-z "$s"`：字符串为空
- `"$a" == "$b"`：字符串相等
- `(( a > b ))`：整数比较

## 循环

```bash
for file in *.txt; do
  echo "处理：$file"
done
```

```bash
count=1
while (( count <= 3 )); do
  echo "$count"
  ((count++))
done
```

## 函数

```bash
greet() {
  local name=${1:-朋友}
  echo "你好，$name"
}

greet Alice
```

## 更稳妥的开头

```bash
#!/usr/bin/env bash
set -Eeuo pipefail
```

- `-e`：未处理的命令失败时退出
- `-u`：使用未定义变量时报错
- `pipefail`：管道任一环节失败会反映到整体
- `-E`：让 ERR trap 在更多上下文继承

这些选项不是万能安全保证；脚本仍要正确处理预期失败和清理逻辑。

## 调试与检查

```bash
bash -n script.sh       # 只检查语法
bash -x script.sh       # 显示执行轨迹
shellcheck script.sh    # 若已安装，做静态检查
```

关联：[[05-权限、用户与sudo]] · [[06-重定向、管道与命令组合]]

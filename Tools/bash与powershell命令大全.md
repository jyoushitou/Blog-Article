---
title: Bash 与 PowerShell 命令大全（语法 + 命令 + 对照）
author: 示例作者
date: 2026-01-01
tags: [bash, shell, powershell, 终端, 命令, 脚本, 大全]
---

# Bash 与 PowerShell 命令大全（语法 + 命令 + 对照）

> 本文分两大块：**上篇 = Bash**（Linux/macOS/WSL/Git Bash），**下篇 = PowerShell**（Windows 原生 / PowerShell 7 跨平台），
> 最后是**命令对照表**、**引号与转义坑**、**实战片段**。
> 约定：`<...>` 是占位符，`[...]` 是可选项；`$` 表示普通用户提示符，`#` 表示 root；`PS>` 表示 PowerShell 提示符。
> 一条铁律：**Bash 的管道里流的是文本，PowerShell 的管道里流的是对象。** 这句话能解释两边 90% 的写法差异。

[TOC]

---

## 〇、怎么用这份文档

| 你的情况 | 直接看 |
| --- | --- |
| 刚进 Linux 终端，命令看不懂 | 上篇 一、二 |
| 想知道「这个符号干嘛的」 | 上篇 三、下篇 三（符号表） |
| 写 Shell 脚本 / 定时任务 | 上篇 四、十一 |
| 处理日志、改配置文件 | 上篇 三（grep/sed/awk） |
| 刚打开 Windows 终端，想替代 cmd | 下篇 一、二 |
| 想把 Linux 命令翻译成 PowerShell | 对照表 A、B |
| 报错「引号/转义/编码」 | 附录 E |
| 只想抄一条能干活的命令 | 附录 B 速查表 |

**先记住这几条**

```bash
man ls              # bash/linux：查手册，q 退出，/ 搜索
ls --help           # 快速看选项
type ls             # 这个命令是别名/内置/外部程序？
```

```powershell
Get-Help Get-ChildItem -Examples   # PS：查帮助（首次可先 Update-Help）
Get-Command ls                     # PS：这个命令到底是什么
Get-Member                         # PS：管他什么东西，先看它是什么对象
```

---

# 上篇 · Bash

## 一、Bash 基础

### 1.1 什么是 Shell / Bash

- **Shell**：用户和内核之间的翻译层，负责解析你敲的命令行。
- **Bash**（Bourne Again SHell）：Linux 默认 Shell，兼容 `sh`，是脚本的通用语言。
- 其他：`sh`、`zsh`（macOS 默认）、`fish`、`dash`（Ubuntu 的 `/bin/sh`，更快但功能少）。
- 查当前 Shell：`echo $SHELL`、`echo $0`、`ps -p $$`。

### 1.2 命令行结构

```bash
命令   [选项]        [参数]
ls     -l -h -a      /var/log
git    commit -m     "fix bug"
```

- **短选项**：`-l`，可合并：`-lh`、`-alh`。
- **长选项**：`--help`、`--color=auto`。
- `--` 表示「选项结束」，后面全当参数：`rm -- -weird-file`。
- 空格分隔一切；**路径有空格必须加引号**：`cd "My Docs"`。

### 1.3 帮助与查找

```bash
man ls                 # 完整手册
man 5 passwd           # 第 5 类（文件格式）手册
ls --help              # 简要选项
info coreutils         # GNU 详细文档
type -a python3        # 命令类型与全部匹配
which python3          # 在 PATH 里的位置
whereis ls             # 二进制/源码/man 位置
whatis ls              # 一行说明
command -v docker      # 脚本里判断命令是否存在（推荐）
apropos "disk usage"   # 按关键词搜 man
compgen -c | grep git  # 列出所有可执行命令
```

### 1.4 目录与文件浏览

```bash
pwd                    # 当前目录
cd /var/log            # 切换
cd ~                   # 回家目录
cd -                   # 回上一个目录（来回跳）
cd ..                  # 上一级
cd ../..               # 上两级
ls                     # 列出
ls -l                  # 长格式（权限/属主/大小/时间）
ls -lh                 # 人类可读大小（K/M/G）
ls -la                 # 含隐藏文件
ls -lt                 # 按修改时间排序（新→旧）
ls -ltr                # 最旧在上（看最新日志常用）
ls -lS                 # 按大小排序
ls -R                  # 递归
ls -d */               # 只列目录
ls -i                  # 显示 inode
tree -L 2              # 树状展示两层（需安装 tree）
stat file.txt          # 详细元信息（inode、权限、三个时间）
file photo.jpg         # 判断真实文件类型
du -sh *               # 每个子项占多大
du -sh * | sort -h     # 按大小排序（找谁在吃磁盘）
df -h                  # 各分区使用率
```

### 1.5 文件与目录操作

```bash
touch a.txt                       # 建空文件 / 更新时间戳
mkdir dir                         # 建目录
mkdir -p a/b/c                    # 递归建，已存在不报错
cp src dst                        # 复制文件
cp -r srcdir dstdir               # 递归复制目录
cp -a srcdir dstdir               # 保留权限/时间/链接（归档式）
cp -i a b                         # 覆盖前询问
mv old new                        # 移动 / 重命名
mv -i a b                         # 覆盖前询问
mv -n a b                         # 不覆盖
rename 's/\.txt$/.md/' *.txt      # 批量改名（perl 版 rename）
rm a.txt                          # 删文件
rm -r dir                         # 递归删目录
rm -rf dir                        # 强制递归删（危险！确认三遍）
rm -i *.log                       # 逐个确认
rmdir emptydir                    # 删空目录
ln -s /path/target linkname       # 软链接（类似快捷方式）
ln target hardlink                # 硬链接
readlink -f link                  # 看软链接指向哪
realpath ./a/../b                 # 规范化路径
basename /a/b/c.txt               # → c.txt
dirname /a/b/c.txt                # → /a/b
mktemp -d                         # 建临时目录（脚本里必用）
mktemp                           # 建临时文件
```

### 1.6 查看文件内容

```bash
cat file                     # 全部输出（小文件）
cat -n file                  # 带行号
tac file                     # 倒序输出
less file                    # 分页查看（推荐；q 退出，/ 搜索，G 末尾，g 开头）
head -n 20 file              # 前 20 行
head -c 100 file             # 前 100 字节
tail -n 20 file              # 后 20 行
tail -f app.log              # 实时跟踪（看日志神器）
tail -F app.log              # 跟踪并处理日志轮转（文件被重建也能跟）
tail -f a.log | grep ERROR   # 只看错误
wc -l file                   # 行数
wc -w -c file                # 词数、字节数
nl file                      # 加行号输出
column -t -s, data.csv       # 按分隔符对齐成表格
xxd file | head              # 十六进制查看
od -c file                   # 八进制/字符查看
```

### 1.7 命令历史与补全

```bash
history                 # 历史列表
history 20              # 最近 20 条
history | grep ssh      # 搜历史
!!                      # 重跑上一条
sudo !!                 # 上一条加 sudo（经典救场）
!123                    # 跑第 123 条
!ssh                    # 跑最近一条以 ssh 开头的
!$                      # 上一条命令的最后一个参数
!*                      # 上一条命令的所有参数
Alt+.                   # 循环插入上一条命令的参数
Ctrl+R                  # 反向搜索历史（最实用），再按 Ctrl+R 继续往前
Ctrl+G                  # 退出历史搜索
Ctrl+L                  # 清屏（等价 clear）
Ctrl+C                  # 中断当前命令
Ctrl+D                  # 结束输入 / 退出 Shell
Ctrl+Z                  # 挂起到后台（bg 继续，fg 拉回）
Ctrl+A / Ctrl+E         # 光标到行首 / 行尾
Ctrl+U / Ctrl+K         # 删到行首 / 删到行尾
Ctrl+W                  # 删掉前一个单词
Tab / Tab Tab           # 补全 / 列出候选
history -c              # 清空当前会话历史
history -d 500          # 删除第 500 条
```

相关环境变量：

```bash
export HISTSIZE=10000            # 内存中历史条数
export HISTFILESIZE=20000        # 文件里保留条数
export HISTTIMEFORMAT='%F %T '   # 历史带时间
export HISTIGNORE='ls:cd:pwd'    # 不记录这些
export HISTCONTROL=ignoreboth    # 忽略重复与前置空格开头的命令
```

### 1.8 命令连接与流程控制

```bash
a; b            # 顺序执行，不管 a 成败
a && b          # a 成功才执行 b（短路与）
a || b          # a 失败才执行 b（兜底）
a && b || c     # 三态：成功走 b，失败走 c
! a             # 取反退出码
a &             # a 放后台
a & b & wait    # 并行跑两个，等它们都结束
```

```bash
mkdir -p /tmp/x && cd /tmp/x && echo ok
grep -q ERROR app.log || echo "没有错误"
command -v docker >/dev/null || { echo "请先装 docker"; exit 1; }
```

### 1.9 分组与子 Shell

```bash
( cd /tmp && ls )        # 子 Shell：cd 不影响当前环境
{ cd /tmp; ls; }         # 当前 Shell 执行（注意 } 前要有 ; 和空格）
(cmd1; cmd2) > out.txt   # 整组输出重定向到文件
diff <(sort a) <(sort b) # 进程替换：把命令当文件用（Bash 专有）
```

### 1.10 后台与作业控制

```bash
long_task &              # 后台运行
jobs -l                  # 看后台作业
fg %1                    # 把 1 号作业拉回前台
bg %1                    # 让停住的作业后台继续
kill %1                  # 杀作业
disown -h %1             # 退出终端时不杀它
nohup ./run.sh > run.log 2>&1 &   # 关掉终端也继续跑
setsid ./run.sh &        # 另起会话（更彻底）
```

---

## 二、Bash 语法：变量、引号、扩展

### 2.1 变量

```bash
name="Alice"             # 定义（= 两边不能有空格！）
echo "$name"             # 使用
echo "${name}"           # 推荐写法，边界清晰
readonly NAME="x"        # 只读
unset name               # 删除
declare -i n=10          # 声明整数
declare -a arr           # 数组
declare -A map           # 关联数组（字典，Bash 4+）
declare -r c=1           # 只读
declare -x E=1           # 导出为环境变量
local v=1                # 函数内局部变量
```

**位置参数与特殊变量**

| 变量 | 含义 |
| --- | --- |
| `$0` | 脚本名 / Shell 名 |
| `$1` `$2` … | 第 1、2 个参数 |
| `${10}` | 第 10 个参数（必须带花括号） |
| `$#` | 参数个数 |
| `$@` | 所有参数（`"$@"` 保留每个参数独立，推荐） |
| `$*` | 所有参数当成一个词（`"$*"` 用 IFS 连接） |
| `$?` | 上一条命令退出码（0 成功，非 0 失败） |
| `$$` | 当前 Shell PID |
| `$!` | 最近一个后台进程 PID |
| `$_` | 上一条命令的最后一个参数 |
| `$-` | 当前 Shell 的选项标志 |

### 2.2 引号三种，区别要说清

```bash
echo '$HOME'      # 单引号：原样输出，什么都不解释
echo "$HOME"      # 双引号：解释变量、命令替换、转义
echo $HOME        # 不加引号：会分词、通配符展开（路径有空格就炸）
echo "a\tb"       # 双引号里 \t 不转义，要 printf "a\tb"
echo -e "a\tb"    # -e 开启转义
```

**规则**：变量、`$(...)`、路径，**一律用双引号包起来**：`rm -rf "$dir"`。
反引号 `` `cmd` `` 等于 `$(cmd)`，但后者可嵌套、可读性更好，请用后者。

### 2.3 参数扩展（Bash 最实用的语法）

```bash
${var}                 # 基本
${var:-default}        # var 未设置或空 → 用 default（不赋值）
${var:=default}        # 同上，并赋值给 var
${var:?msg}            # 未设置则报错退出（必填参数校验）
${var:+alt}            # var 有值 → 用 alt
${#var}                # 字符串长度
${var:0:3}             # 截取：从 0 起 3 个字符
${var#pattern}         # 从左边删最短匹配
${var##pattern}        # 从左边删最长匹配
${var%pattern}         # 从右边删最短匹配
${var%%pattern}        # 从右边删最长匹配
${var/old/new}         # 替换第一处
${var//old/new}        # 替换全部
${var/#old/new}        # 只替换开头
${var/%old/new}        # 只替换结尾
${var^^}               # 全大写
${var,,}               # 全小写
${!prefix*}            # 列出所有以 prefix 开头的变量名
${!arr[@]}             # 数组下标
```

实战：

```bash
file="/path/to/report.tar.gz"
echo "${file##*/}"      # report.tar.gz   （相当于 basename）
echo "${file%/*}"       # /path/to        （相当于 dirname）
echo "${file%.*}"       # /path/to/report.tar
echo "${file##*.}"      # gz              （取扩展名）
echo "${file//\//_}"    # _path_to_report.tar.gz
: "${DB_HOST:?必须设置 DB_HOST}"
```

### 2.4 算术

```bash
n=$((3 + 4 * 2))        # 推荐：算术展开
((n++))                 # 自增，也可作条件
let "n = n + 1"         # let 形式
echo $[1+1]             # 老写法，不推荐
echo "scale=2; 10/3" | bc       # 浮点（bc）
awk 'BEGIN{printf "%.2f\n", 10/3}'   # 浮点（awk，更常用）
```

运算符：`+ - * / % **`、比较 `> < >= <= == !=`、位运算 `& | ^ ~ << >>`、逻辑 `&& || !`。
十六进制：`$((16#FF))` → 255。

### 2.5 条件判断 `test` / `[ ]` / `[[ ]]`

```bash
if [ -f "$f" ]; then echo 存在; fi
[ -d "$d" ] && echo 是目录
[[ "$s" == *.log ]] && echo 日志文件       # [[ ]] 支持通配符，推荐
[[ "$s" =~ ^[0-9]+$ ]] && echo 纯数字     # 正则匹配
```

**文件测试**

| 表达式 | 含义 |
| --- | --- |
| `-e f` | 存在 |
| `-f f` | 存在且是普通文件 |
| `-d f` | 是目录 |
| `-l f` | 是符号链接 |
| `-r / -w / -x` | 可读 / 可写 / 可执行 |
| `-s f` | 存在且非空 |
| `-L f` | 符号链接 |
| `f1 -nt f2` | f1 比 f2 新 |
| `f1 -ot f2` | f1 比 f2 旧 |
| `f1 -ef f2` | 同一文件（同 inode） |

**字符串测试**

| 表达式 | 含义 |
| --- | --- |
| `-z "$s"` | 空串为真 |
| `-n "$s"` | 非空为真 |
| `"$a" = "$b"` | 相等（`==` 在 `[[ ]]` 里也行） |
| `"$a" != "$b"` | 不等 |
| `"$a" \< "$b"` | 字典序小于（`[ ]` 里要转义） |

**数字比较**（`[ ]` 里不能用 `>`、`<`，那是重定向！）

| 运算符 | 含义 |
| --- | --- |
| `-eq` | 等于 |
| `-ne` | 不等于 |
| `-gt` / `-ge` | 大于 / 大于等于 |
| `-lt` / `-le` | 小于 / 小于等于 |

**逻辑**

```bash
[ -f a ] && [ -f b ]        # 与
[ -f a ] || [ -f b ]        # 或
[ ! -f a ]                  # 非
[[ -f a && -f b ]]          # [[ ]] 内可直接写
```

### 2.6 流程控制

```bash
# if / elif / else
if [[ -f "$f" ]]; then
  echo 文件存在
elif [[ -d "$f" ]]; then
  echo 是目录
else
  echo 都不是
fi

# case（模式匹配分支）
case "$1" in
  start)   echo 启动 ;;
  stop)    echo 停止 ;;
  restart) echo 重启 ;;
  *.log)   echo 日志 ;;
  *)       echo "用法: $0 {start|stop|restart}" ; exit 1 ;;
esac

# while
i=0
while (( i < 5 )); do
  echo "$i"; ((i++))
done

# 逐行读文件（标准写法）
while IFS= read -r line; do
  echo "行: $line"
done < file.txt

# 命令输出逐行读
grep ERROR app.log | while read -r line; do echo ">> $line"; done

# until（条件为假时循环）
until [[ -f /tmp/done ]]; do sleep 1; done

# for 列表
for f in *.log; do echo "$f"; done
for i in {1..5}; do echo "$i"; done
for i in $(seq 1 2 10); do echo "$i"; done          # 1 3 5 7 9
for ((i=0; i<10; i++)); do echo "$i"; done

# C 风格无限循环 + break/continue
while :; do
  read -r x || break
  [[ "$x" == q ]] && break
  [[ "$x" == skip ]] && continue
  echo "$x"
done

# select 菜单
select opt in 启动 停止 退出; do
  case $opt in 启动|停止) echo "选了 $opt";; 退出) break;; esac
done
```

### 2.7 函数

```bash
# 定义（两种等价）
greet() {
  local name="$1"           # 参数就是 $1 $2 ...
  echo "Hello, $name"
}
function greet2 { echo hi; }

# 调用
greet "World"

# 返回值：return 只能返回 0-255 的退出码，数据用 echo 输出 + 命令替换
is_number() {
  [[ "$1" =~ ^[0-9]+$ ]]
}
if is_number 123; then echo 是数字; fi

get_version() { echo "1.2.3"; }
v=$(get_version)

# 传数组（用 "$@" 或 nameref）
log() { echo "[$(date +%F' '%T)] $*"; }
log 服务启动
```

### 2.8 数组

```bash
arr=(a b c)
arr+=(d)                  # 追加
echo "${arr[0]}"          # 第一个（下标从 0 开始）
echo "${arr[-1]}"         # 最后一个（Bash 4.3+）
echo "${arr[@]}"          # 全部元素
echo "${#arr[@]}"         # 元素个数
echo "${!arr[@]}"         # 所有下标
echo "${arr[@]:1:2}"      # 切片：从 1 起 2 个
unset 'arr[1]'            # 删除某个元素

# 遍历（永远用 "${arr[@]}" 加引号）
for x in "${arr[@]}"; do echo "$x"; done

# 关联数组（字典）
declare -A m
m[host]="127.0.0.1"
m[port]=8080
echo "${m[host]}"
for k in "${!m[@]}"; do echo "$k=${m[$k]}"; done
```

### 2.9 花括号与通配符展开

```bash
echo {a,b,c}              # a b c
echo file{1..3}.txt       # file1.txt file2.txt file3.txt
echo {01..10}             # 01 02 ... 10
echo {a..e}               # a b c d e
mkdir -p project/{src,test,docs}          # 一次建多个目录
cp config.yml{,.bak}                      # 备份（等价 cp config.yml config.yml.bak）
```

**通配符（Glob）**

| 模式 | 含义 |
| --- | --- |
| `*` | 任意多个字符（不含 `/`） |
| `?` | 一个字符 |
| `[abc]` | 其中任一字符 |
| `[a-z]` | 范围 |
| `[!abc]` / `[^abc]` | 排除 |
| `**` | 递归匹配（需 `shopt -s globstar`） |
| `{a,b}` | 花括号展开（不是 glob，是 Bash 做的） |

```bash
shopt -s globstar nullglob dotglob   # 开递归/无匹配为空/含隐藏文件
shopt -u globstar                    # 关闭
ls **/*.md                           # 递归找（开了 globstar 后）
```

### 2.10 命令替换与进程替换

```bash
now=$(date +%F)                 # 命令替换（推荐）
files=$(ls | wc -l)
echo "共 $files 个文件"

diff <(sort a.txt) <(sort b.txt)        # 进程替换
while read -r l; do echo "$l"; done < <(grep ERROR app.log)   # 避免子 Shell 变量丢失
```

### 2.11 退出码与 `set` 严格模式

```bash
echo $?                  # 上条命令退出码：0=成功，1/2/126/127/130/137 常见失败
exit 0                   # 脚本正常退出
exit 1                   # 脚本失败退出
false; echo $?           # 1
command_not_found_xyz    # 127
```

脚本开头**强烈推荐**：

```bash
#!/usr/bin/env bash
set -euo pipefail
# -e 任一命令失败立刻退出
# -u 使用未定义变量报错（防手滑）
# -o pipefail 管道中任一环失败就算失败
IFS=$'\n\t'              # 可选：收紧分词
```

调试：

```bash
bash -x script.sh        # 跟踪每条命令
set -x / set +x          # 脚本内开关跟踪
set -n script.sh         # 只做语法检查
bash -n script.sh        # 语法检查
shellcheck script.sh     # 静态检查（强烈推荐安装）
PS4='+${BASH_SOURCE}:${LINENO}: '   # 调试时显示文件与行号
trap 'echo "出错于第 $LINENO 行"' ERR
trap 'rm -f "$tmp"' EXIT            # 退出时清理（必用）
```

---

## 三、Bash 核心命令与文本处理

### 3.1 查找文件 `find`

```bash
find . -name "*.log"                     # 按名字
find . -iname "*.LOG"                    # 忽略大小写
find /var -type f -size +100M            # 大于 100M 的文件
find . -type d -name node_modules        # 找目录
find . -mtime -7                         # 7 天内修改过
find . -mtime +30 -name "*.log"          # 30 天前
find . -mmin -10                         # 10 分钟内
find . -user root                        # 属主
find . -perm 644                         # 权限
find . -empty                            # 空文件/空目录
find . -name "*.tmp" -delete             # 找到即删
find . -name "*.log" -exec gzip {} \;    # 逐个处理（\; 结尾）
find . -name "*.log" -exec gzip {} +     # 批量传参（+ 结尾，更快）
find . -name "*.bak" -print0 | xargs -0 rm -f    # 文件名含空格也安全
find . -type f -newer ref.txt            # 比 ref.txt 新
find . -maxdepth 2 -type f               # 限制深度
find . -path ./node_modules -prune -o -name "*.js" -print   # 排除目录
```

### 3.2 文本搜索 `grep`

```bash
grep "error" app.log                 # 基本搜索
grep -i "error" app.log              # 忽略大小写
grep -v "error" app.log              # 反选（不含）
grep -r "TODO" ./src                 # 递归
grep -rn "TODO" ./src                # 递归 + 行号
grep -rl "TODO" ./src                # 只列文件名
grep -c "error" app.log              # 计数
grep -w "cat" file                   # 全词匹配
grep -x "exact line" file            # 整行匹配
grep -A 3 -B 2 "panic" app.log       # 前后各带 3/2 行上下文
grep -C 2 "panic" app.log            # 前后各 2 行
grep -E "err(or|ors)" file           # 扩展正则（=egrep）
grep -F "a.b[c]" file                # 固定字符串，不当正则（=fgrep）
grep -o "[0-9]\+" file               # 只输出匹配部分
grep -q "ok" file && echo yes        # 静默，只看退出码
grep -n --color=auto "x" file        # 高亮 + 行号
grep -e "a" -e "b" file              # 多个模式
grep -h "x" *.log                    # 不显示文件名
grep -L "x" *.log                    # 列出不含匹配的文件名
grep -m 1 "x" file                   # 最多输出 1 条
```

正则要点：`.` 任意字符、`*` 前一个重复 0+、`.*` 贪婪、`\+` 1+（BRE 里要转义，ERE 里写 `+`）、`^` 行首、`$` 行尾、`[0-9]`、`\b` 词边界、`\(...\)` 分组（BRE）。

### 3.3 `sed` 流编辑

```bash
sed 's/old/new/' file            # 每行替换第一处
sed 's/old/new/g' file           # 全局替换
sed -i 's/old/new/g' file        # 直接改文件（-i.bak 先备份）
sed -i.bak 's/old/new/g' file    # 改并生成 file.bak
sed -n '5,10p' file              # 只打印 5-10 行（-n 抑制默认输出）
sed -n '/start/,/end/p' file     # 打印两个模式之间的行
sed '/^#/d' file                 # 删注释行
sed '/^$/d' file                 # 删空行
sed '2d' file                    # 删第 2 行
sed '1i\新首行' file             # 第 1 行前插入
sed '$a\新尾行' file             # 末尾追加
sed -E 's/([0-9]{4})-([0-9]{2})/\2\/\1/' file   # 分组反向引用
sed 's#/old/path#/new/path#' f   # 用 # 当分隔符（路径友好）
sed 's/\s\+$//' file             # 去行尾空白
sed -n '=' file                  # 输出行号
sed 's/"/\\"/g' file             # 转义双引号
```

### 3.4 `awk` 按列处理

```bash
awk '{print $1, $3}' file              # 第 1、3 列（默认空白分隔）
awk -F: '{print $1}' /etc/passwd       # 指定分隔符
awk -F, '{print $2}' data.csv          # CSV
awk 'NR==5' file                       # 第 5 行
awk 'NR>=2 && NR<=10' file             # 2-10 行
awk '$3 > 100 {print $1}' file         # 条件过滤
awk '/ERROR/ {print $0}' file          # 模式匹配
awk '{sum += $2} END {print sum}' f    # 求和
awk '{c++} END {print c}' f            # 计数
awk '!seen[$0]++' file                 # 去重但保持顺序（神器）
awk 'NF' file                          # 去掉空行
awk '{print NR": "$0}' file            # 加行号
awk 'BEGIN{FS=","; OFS="\t"} {print $1,$2}' data.csv   # 设置分隔符
awk -v limit=100 '$2 > limit' file     # 传外部变量
awk '{printf "%-10s %5d\n", $1, $2}' f # 格式化输出
awk 'length($0) > 80' file             # 超过 80 列的行
awk '{for(i=1;i<=NF;i++) print i, $i}' f   # 遍历字段
```

内置变量：`$0` 整行、`$1..$n` 字段、`NF` 字段数、`NR` 当前行号、`FNR` 文件内行号、`FS` 输入分隔符、`OFS` 输出分隔符、`RS` 记录分隔符、`FILENAME`。

### 3.5 其他文本工具

```bash
cut -d: -f1 /etc/passwd          # 取第 1 列（单字符分隔）
cut -c1-10 file                  # 取前 10 个字符
paste a.txt b.txt                # 按行横向拼接
join -t, a.csv b.csv             # 按第一列连接（需先排序）
sort file                        # 字典序
sort -n numbers.txt              # 按数值
sort -h sizes.txt                # 按人类可读大小（K/M/G）
sort -r file                     # 逆序
sort -k2 -n file                 # 按第 2 列数值
sort -t, -k3 file                # 指定分隔符的列排序
sort -u file                     # 排序去重
uniq file                        # 去相邻重复（先 sort）
uniq -c file                     # 统计出现次数（最常用）
uniq -d file                     # 只显示重复行
uniq -u file                     # 只显示不重复行
tr 'a-z' 'A-Z' < file            # 小写转大写
tr -d '\n' < file                # 删换行
tr -s ' ' < file                 # 压缩连续空格
tr -c '[:alnum:]' '\n' < f       # 非字母数字换成换行
rev file                         # 每行反转
fold -w 80 file                  # 折行
diff a.txt b.txt                 # 比较差异
diff -u a.txt b.txt              # 统一格式（补丁用）
diff -y a.txt b.txt              # 并排对比
diff -r dir1 dir2                # 递归比较目录
sdiff a b                        # 交互式并排
patch -p1 < fix.patch            # 打补丁
comm -12 <(sort a) <(sort b)     # 两文件交集（-23 独有，-3 只在 a）
tee out.txt                      # 输出到屏幕同时写文件
tee -a out.txt                   # 追加
xargs -n1 echo                   # 每次传 1 个参数
xargs -I{} cp {} /backup/        # 占位符替换
xargs -P4 -n1 cmd                # 4 路并行
xargs -0 rm -f                   # 配合 find -print0
```

**经典组合（背下来就能干活）**

```bash
# 统计访问量 Top 10 IP
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10

# 统计各状态码数量
awk '{print $9}' access.log | sort | uniq -c | sort -rn

# 找出占磁盘最大的 10 个目录
du -sh */ 2>/dev/null | sort -h | tail -10

# 找出占用 8080 端口的进程并杀掉
lsof -ti:8080 | xargs -r kill -9

# 批量把 CRLF 换成 LF
sed -i 's/\r$//' *.sh

# 提取 JSON 里某字段（无 jq 时）
grep -o '"name":"[^"]*"' data.json | cut -d'"' -f4
```

### 3.6 重定向与管道

```bash
cmd > out.txt            # 标准输出覆盖写入
cmd >> out.txt           # 追加
cmd 2> err.txt           # 标准错误写入
cmd 2>&1                 # 错误并入标准输出
cmd > out.txt 2>&1       # 两者都进文件（顺序不能反）
cmd &> out.txt           # 同上（Bash 简写）
cmd > /dev/null 2>&1     # 全都丢掉
cmd 2>/dev/null          # 只丢错误（只看结果）
cmd < in.txt             # 标准输入来自文件
cmd <<< "字符串"         # here-string
cmd <<EOF                # here-doc
第一行 $var
EOF
cmd <<'EOF'              # 引号包 EOF：内部不做变量替换
cmd | tee out.txt | less # 一边看一边存
exec > log.txt 2>&1      # 整个脚本后续输出都进文件
cmd > file 2>&1 &        # 后台并把输出写文件
: > file                 # 清空文件
```

**文件描述符**：0=stdin、1=stdout、2=stderr；自定义：`exec 3>file`、`echo x >&3`、`exec 3>&-`。

### 3.7 归档与压缩

```bash
tar -czvf backup.tar.gz /data            # 打包并 gzip（c 建 z gzip v 显示 f 文件名）
tar -xzvf backup.tar.gz                  # 解包
tar -xzvf backup.tar.gz -C /target       # 解到指定目录
tar -tzvf backup.tar.gz                  # 只列内容不解
tar -cjvf b.tar.bz2 dir                  # bzip2（更小更慢）
tar -cJvf b.tar.xz dir                   # xz（最小最慢）
tar -czvf b.tgz --exclude='*.log' dir    # 排除
tar -czvf b.tgz -C /base sub             # 去掉路径前缀
tar -xzvf b.tar.gz path/in/archive       # 只解其中某个文件
gzip file / gunzip file.gz               # 单文件压缩/解压
zip -r a.zip dir                         # zip
unzip a.zip -d target                    # 解 zip
unzip -l a.zip                           # 列内容
7z x a.7z                                # 7z
zstd file                                # 现代高速压缩
```

### 3.8 权限与属主

```bash
chmod 755 script.sh        # rwxr-xr-x
chmod 644 file.txt         # rw-r--r--
chmod +x script.sh         # 加执行权限
chmod -R 750 dir           # 递归
chmod u+x,g-w,o-r file     # 符号法
chmod --reference=a b      # 照抄 a 的权限
chown user:group file      # 改属主属组
chown -R user:group dir    # 递归
chgrp group file           # 只改属组
umask                      # 看默认权限掩码
umask 022                  # 新文件 644、新目录 755
getfacl file / setfacl -m u:alice:rw file    # ACL 精细授权
lsattr / chattr +i file    # 查看/设置不可变属性（+i 后 root 也删不掉）
sudo -u other cmd          # 以他人身份执行
sudo -i                    # 切到 root 登录式 Shell
su - user                  # 切换用户
id / whoami / groups       # 我是谁、属哪些组
```

权限数字对照：`r=4 w=2 x=1`；`7=rwx 6=rw- 5=r-x 4=r-- 0=---`。

### 3.9 进程与系统

```bash
ps aux                     # 所有进程（BSD 风格）
ps -ef                     # 所有进程（System V 风格）
ps -ef | grep nginx        # 找进程（grep 自身也会出现）
pgrep -af nginx            # 按名字找（推荐）
pstree -p                  # 进程树
top                        # 实时监控（M 按内存，P 按 CPU，q 退出）
htop                       # 更好用的 top
kill 1234                  # 温和终止（SIGTERM=15）
kill -9 1234               # 强杀（SIGKILL，不可捕获，慎用）
kill -HUP 1234             # 让服务重载配置
pkill -f "python app.py"   # 按命令行匹配杀
killall nginx              # 按名字杀
nice -n 10 cmd             # 低优先级启动（-20 最高，19 最低）
renice -n 5 -p 1234        # 调整已运行进程优先级
nohup cmd &                # 免挂断后台
jobs / fg / bg / disown    # 作业控制
lsof -i:8080               # 谁占用端口
lsof -p 1234               # 进程打开的文件
fuser -k 8080/tcp          # 杀掉占用端口的进程
uptime                     # 运行时长与负载
free -h                    # 内存
vmstat 1 5                 # 每秒采样，共 5 次
iostat -x 1                # 磁盘 IO
sar -u 1 3                 # 历史性能（sysstat）
dmesg -T | tail            # 内核日志（OOM 排查）
journalctl -u nginx -f     # systemd 日志跟踪
journalctl --since "1 hour ago" -p err
systemctl status nginx / start / stop / restart / reload / enable / disable
service nginx restart      # 老式 SysV
crontab -e / -l            # 编辑/列出定时任务
at 23:00                   # 一次性定时（echo cmd | at 23:00）
watch -n 2 'df -h'         # 每 2 秒重复执行
time cmd                   # 计时
date '+%F %T' / date -d '-1 day' '+%F'
timedatectl                # 时区与时间同步
hostnamectl                # 主机信息
uname -a / hostname -I     # 内核 / IP
who / w / last / lastlog   # 登录情况
```

### 3.10 网络

```bash
ip a                       # 网卡与地址（替代 ifconfig）
ip r                       # 路由表
ip -br a                   # 简洁输出
ss -tulnp                  # 监听端口（替代 netstat）
ss -tan state established  # 已建立连接
ping -c 4 8.8.8.8          # 连通性
traceroute host            # 路由跟踪（Windows 是 tracert）
mtr host                   # 持续 traceroute + 丢包统计
dig +short example.com     # DNS 查询
nslookup example.com
host example.com
curl -I https://example.com              # 只看响应头
curl -sS -o out.json https://api/x      # 保存到文件
curl -X POST -H "Content-Type: application/json" -d '{"a":1}' https://api/x
curl -u user:pass https://api/x          # Basic 认证
curl -L https://x                        # 跟随重定向
curl -k https://x                        # 忽略证书
curl -w "%{http_code} %{time_total}\n" -o /dev/null -s https://x   # 只要状态码与耗时
curl --retry 3 --max-time 10 https://x
wget -c https://x/file.tar.gz            # 断点续传下载
scp file user@host:/path                 # 传文件
scp -r dir user@host:/path
rsync -avz --progress src/ user@host:/dst/     # 增量同步（推荐）
rsync -avz --delete src/ user@host:/dst/       # 目标多余文件删除（危险）
ssh user@host                            # 登录
ssh -p 2222 user@host                    # 指定端口
ssh -i key.pem user@host                 # 指定私钥
ssh -L 8080:localhost:80 user@host       # 本地端口转发
ssh -N -f -L 5432:db:5432 jump@host      # 后台隧道
ssh-keygen -t ed25519 -C "me@x.com"      # 生成密钥
ssh-copy-id user@host                    # 上传公钥免密
~/.ssh/config                            # 别名/跳板机配置
```

`~/.ssh/config` 示例：

```sshconfig
Host prod
    HostName 10.0.0.5
    User deploy
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
```

之后直接 `ssh prod`、`scp f prod:/tmp/`。

### 3.11 包管理与服务（速查）

```bash
# Debian / Ubuntu
apt update && apt upgrade -y
apt install -y nginx
apt remove --purge nginx
apt search keyword
apt list --installed
dpkg -l | grep nginx
dpkg -i pkg.deb

# RHEL / CentOS / Rocky
dnf install -y nginx        # 或 yum
rpm -qa | grep nginx
rpm -ivh pkg.rpm

# 其他
snap install code --classic
pip install requests / pipx install tool
npm i -g pnpm
```

---

## 四、Bash 脚本模板

```bash
#!/usr/bin/env bash
# ============================================================
# 用途：示例脚本（备份 + 清理）
# 用法：./backup.sh -s /data -d /backup -k 7
# ============================================================
set -euo pipefail
IFS=$'\n\t'

readonly SCRIPT_NAME="$(basename "$0")"
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

log()  { printf '[%s] [INFO ] %s\n'  "$(date '+%F %T')" "$*"; }
warn() { printf '[%s] [WARN ] %s\n'  "$(date '+%F %T')" "$*" >&2; }
die()  { printf '[%s] [ERROR] %s\n'  "$(date '+%F %T')" "$*" >&2; exit 1; }

usage() {
  cat <<EOF
用法: $SCRIPT_NAME -s <源目录> [-d <备份目录>] [-k <保留天数>]

选项:
  -s  源目录（必填）
  -d  备份目录（默认 /backup）
  -k  保留天数（默认 7）
  -h  显示帮助
EOF
}

SRC=""; DST="/backup"; KEEP=7
while getopts ":s:d:k:h" opt; do
  case "$opt" in
    s) SRC="$OPTARG" ;;
    d) DST="$OPTARG" ;;
    k) KEEP="$OPTARG" ;;
    h) usage; exit 0 ;;
    \?) die "未知选项: -$OPTARG" ;;
    :)  die "选项 -$OPTARG 需要参数" ;;
  esac
done
shift $((OPTIND - 1))

[[ -n "$SRC" ]]      || { usage; die "必须指定 -s"; }
[[ -d "$SRC" ]]      || die "源目录不存在: $SRC"
[[ "$KEEP" =~ ^[0-9]+$ ]] || die "-k 必须是数字"

mkdir -p "$DST"
TMP="$(mktemp -d)"
trap 'rm -rf "$TMP"' EXIT                       # 无论如何都清理
trap 'die "脚本在第 $LINENO 行异常退出"' ERR

STAMP="$(date +%Y%m%d-%H%M%S)"
ARCHIVE="$DST/$(basename "$SRC")-$STAMP.tar.gz"

log "开始打包 $SRC → $ARCHIVE"
tar -czf "$ARCHIVE" -C "$(dirname "$SRC")" "$(basename "$SRC")"
log "打包完成，大小 $(du -h "$ARCHIVE" | cut -f1)"

log "清理 $KEEP 天前的备份"
find "$DST" -name "*.tar.gz" -mtime "+$KEEP" -print -delete

log "全部完成"
```

常用骨架片段：

```bash
# 必须 root
[[ $EUID -eq 0 ]] || die "请用 root 运行"

# 交互确认
read -r -p "确定继续? [y/N] " ans
[[ "$ans" =~ ^[Yy]$ ]] || exit 1

# 读密码不回显
read -r -s -p "密码: " pass; echo

# 并发控制
for f in *.csv; do
  while (( $(jobs -r | wc -l) >= 4 )); do sleep 0.1; done
  process "$f" &
done
wait

# 重试
retry() { local n=3 i=1; until "$@"; do ((i++ > n)) && return 1; sleep 2; done; }
retry curl -sSf https://api/health

# 锁，避免重复运行
exec 9>/var/lock/myscript.lock
flock -n 9 || die "已有实例在运行"
```

---

# 下篇 · PowerShell

## 五、PowerShell 基础

### 五.0 版本与入口

- **Windows PowerShell 5.1**：系统自带，`powershell.exe`，仅 Windows，只有 `ConvertFrom-Json` 等基础能力。
- **PowerShell 7.x**（原 Core）：跨平台，命令是 `pwsh`，推荐日常用；支持三元 `? :`、`??`、`&&`、`||`、并行 `ForEach-Object -Parallel`。
- 查看版本：`$PSVersionTable`。
- 执行策略（脚本被拦时）：

```powershell
Get-ExecutionPolicy -List
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned   # 常用安全档
pwsh -ExecutionPolicy Bypass -File .\x.ps1            # 临时绕过
```

### 5.1 命名规范：动词-名词

所有 cmdlet 都是 `Verb-Noun`：`Get-ChildItem`、`Set-Content`、`Remove-Item`、`New-Item`、`Start-Process`。

常用动词：`Get`（读）、`Set`（改）、`New`（建）、`Remove`（删）、`Add`、`Copy`、`Move`、`Rename`、`Start`/`Stop`、`Test`、`Select`、`Where`、`ForEach`、`Sort`、`Group`、`Measure`、`ConvertTo`/`ConvertFrom`、`Import`/`Export`、`Invoke`、`Out`、`Write`、`Enable`/`Disable`、`Register`/`Unregister`。

### 5.2 别名（Linux 习惯可直接用）

| 别名 | 真实命令 | 说明 |
| --- | --- | --- |
| `ls` / `dir` / `gci` | `Get-ChildItem` | 列目录 |
| `cd` / `chdir` | `Set-Location` | 切目录 |
| `pwd` | `Get-Location` | 当前目录 |
| `cat` / `type` / `gc` | `Get-Content` | 读文件 |
| `cp` / `copy` | `Copy-Item` | 复制 |
| `mv` / `move` | `Move-Item` | 移动 |
| `rm` / `del` / `erase` / `rd` | `Remove-Item` | 删除 |
| `mkdir` / `md` | `New-Item -ItemType Directory` | 建目录 |
| `ni` | `New-Item` | 建文件/目录 |
| `ps` | `Get-Process` | 进程 |
| `kill` | `Stop-Process` | 杀进程 |
| `curl` / `wget` | `Invoke-WebRequest` | 下载（**不是真 curl**） |
| `echo` / `write` | `Write-Output` | 输出对象 |
| `cls` / `clear` | `Clear-Host` | 清屏 |
| `man` / `help` | `Get-Help` | 帮助 |
| `grep` | `Select-String` | 文本搜索 |
| `sort` | `Sort-Object` | 排序 |
| `where` | `Where-Object` | 过滤 |
| `foreach` | `ForEach-Object` | 遍历 |

脚本里**建议写全名**（别名在不同版本/平台上可能缺失）。查别名：`Get-Alias ls`、`Get-Alias -Definition Get-ChildItem`。

### 5.3 帮助系统

```powershell
Get-Help Get-ChildItem                 # 基本帮助
Get-Help Get-ChildItem -Examples       # 示例（最有用）
Get-Help Get-ChildItem -Full           # 全部
Get-Help Get-ChildItem -Parameter Path # 参数说明
Get-Help about_Operators               # 概念文档：about_*
Update-Help -Force                     # 首次更新帮助（需联网/管理员）
Get-Command *service*                  # 按名字找命令
Get-Command -Noun Service              # 按名词找
Get-Command -Verb Get                  # 按动词找
Get-Command -Module Microsoft.PowerShell.Management
(Get-Command Get-ChildItem).Parameters  # 看参数表
Get-Member                             # 看对象的属性/方法（管道后必用）
```

### 5.4 对象管道：PowerShell 的灵魂

```powershell
Get-Process | Where-Object CPU -gt 100 | Sort-Object CPU -Descending | Select-Object -First 5 Name, CPU, Id
```

拆开看每一步：

1. `Get-Process` 返回 **Process 对象数组**，不是文本。
2. `Where-Object` 按**属性**过滤（不是按行文本匹配）。
3. `Sort-Object` 按**属性**排序。
4. `Select-Object` 挑**属性**，不是"取第几列字符"。
5. 输出时才由格式化系统渲染成表格。

所以：

- **不要用 `grep` 的思路**。要过滤对象用 `Where-Object`；要过滤文本才用 `Select-String`。
- 想看对象里有什么：`... | Get-Member` 或 `... | Select-Object -First 1 | Format-List *`。
- 想从对象里掏值：`$p.Name`、`$p.CPU`，或用 `Select-Object -ExpandProperty Name`。

### 5.5 变量与类型

```powershell
$name = "Alice"                        # 定义（= 两边可有空格，宽松）
$n = 42                                # Int32
$d = 3.14                              # Double
$b = $true / $false                    # 布尔
$null                                  # 空
$arr = 1, 2, 3                         # 数组
$arr = @(1, 2, 3)                      # 显式数组
$empty = @()                           # 空数组
$h = @{ Name = "a"; Port = 80 }        # 哈希表（字典）
$o = [pscustomobject]@{ Name = "a"; Port = 80 }   # 自定义对象（推荐）
$list = [System.Collections.Generic.List[int]]::new()

"$name 有 $($h.Port) 端口"              # 插值；表达式要用 $( )
'单引号不做插值 $name'
$name.Length                           # 属性
$name.ToUpper()                        # 方法
[int]"42" ; [string]42 ; [double]"3.14"   # 类型转换
$n.GetType().FullName                  # 查看类型
[Math]::Round(3.14159, 2)              # 调用 .NET 静态方法
$PSVersionTable.PSVersion              # 版本对象

# 变量作用域
$global:x = 1 ; $script:x = 2 ; $local:x = 3 ; $private:x = 4
New-Variable -Name y -Value 5
Remove-Variable y
Get-Variable | Select-Object -First 10
```

自动变量：`$_`（管道当前对象）、`$PSItem`（同上）、`$args`、`$input`、`$?`（上次成功与否）、`$LASTEXITCODE`（外部程序退出码）、`$Error`、`$PSHOME`、`$PWD`、`$HOME`、`$PROFILE`、`$env:NAME`（环境变量）、`$PSDefaultParameterValues`。

### 5.6 运算符

**比较**（默认大小写不敏感；前缀 `c` 变敏感，`i` 变不敏感）

| 运算符 | 含义 |
| --- | --- |
| `-eq` `-ne` | 等于 / 不等于 |
| `-gt` `-ge` `-lt` `-le` | 大于 / ≥ / 小于 / ≤ |
| `-like` / `-notlike` | 通配符匹配（`*` `?` `[a-z]`） |
| `-match` / `-notmatch` | 正则匹配（会填充 `$Matches`） |
| `-contains` / `-notcontains` | 集合包含某值 |
| `-in` / `-notin` | 值在集合中 |
| `-replace` | 正则替换：`"a1" -replace '\d','X'` |
| `-split` / `-join` | 拆分 / 连接 |
| `-is` / `-isnot` / `-as` | 类型判断 / 转换 |
| `-ceq` `-clike` `-cmatch` | 大小写敏感版本 |

**逻辑与空值**

```powershell
$a -and $b ; $a -or $b ; -not $a ; !$a
$x = $null
$y = $x ?? "默认值"              # null 合并（PS7）
$x ??= "赋值"                    # null 合并赋值（PS7）
$v = $cond ? "yes" : "no"        # 三元（PS7）
$obj?.Name                       # 空条件访问（PS7）

# 范围与成员
1..10                            # 范围数组
(1..10) -contains 5
5 -in 1..10
$h.Keys ; $h["Name"] ; $h.Name
$h['new'] = 1
$h.Remove('new')

# 数组操作
$arr + @(4,5)                    # 拼接
$arr[0] ; $arr[-1] ; $arr[1..2]  # 索引（负数为倒数）
$arr.Count
$arr -contains 2
$arr | Where-Object { $_ -gt 1 }
$arr.Where({ $_ -gt 1 })         # 方法式（更快，PS4+）
$arr.ForEach({ $_ * 2 })
```

### 5.7 条件与循环

```powershell
if ($n -gt 10) { "大" } elseif ($n -gt 5) { "中" } else { "小" }

# switch（能匹配正则、通配符、范围，且不会 fall-through）
switch ($status) {
  200      { "OK" }
  { $_ -lt 300 } { "成功类" }
  default  { "其他: $_" }
}
switch -Wildcard ($file) { "*.log" { "日志" } "*.txt" { "文本" } }
switch -Regex ($line)    { '^ERROR' { "错误行" } }

for ($i = 0; $i -lt 5; $i++) { $i }
foreach ($f in Get-ChildItem) { $f.Name }
Get-ChildItem | ForEach-Object { $_.Name }
Get-ChildItem | ForEach-Object -Parallel { $_.Name } -ThrottleLimit 8   # PS7
while ($true) { break }
do { } while ($false)
do { } until ($true)
1..10 | ForEach-Object { if ($_ % 2) { return } ; $_ }   # return 相当于 continue
break / continue / return
```

### 5.8 函数与脚本

```powershell
function Get-Square {
    [CmdletBinding()]                 # 开启高级函数特性（-Verbose 等）
    param(
        [Parameter(Mandatory, Position = 0)]
        [ValidateRange(0, 1000)]
        [int]$Number,

        [ValidateSet('KB','MB','GB')]
        [string]$Unit = 'MB',

        [switch]$Detailed
    )
    process {                          # 支持管道输入
        $r = $Number * $Number
        if ($Detailed) { [pscustomobject]@{ Input = $Number; Result = $r; Unit = $Unit } }
        else { $r }
    }
}
Get-Square 12
Get-Square -Number 12 -Detailed

# 返回多个值：输出多个对象即可
function Get-Info { "a"; "b" }
$r = Get-Info              # $r 是数组

# 参数校验
[ValidateNotNullOrEmpty()][string]$Path
[ValidatePattern('^\d{3}-\d{4}$')][string]$Phone
[ValidateScript({ Test-Path $_ })][string]$Path

# 注释帮助（Get-Help 能读到）
<#
.SYNOPSIS  一句话说明
.DESCRIPTION 详细说明
.PARAMETER Number 参数说明
.EXAMPLE  Get-Square 3
#>

# 脚本参数（.ps1 开头）
param(
  [Parameter(Mandatory)][string]$Path,
  [switch]$Force
)
```

### 5.9 错误处理

```powershell
# 两类错误：终止性 / 非终止性
$ErrorActionPreference = 'Stop'        # 全部当终止性错误（脚本里推荐）
Get-Item nope -ErrorAction SilentlyContinue     # 忽略
Get-Item nope -ErrorAction Stop                 # 提升为终止性

try {
  Get-Content "nope.txt" -ErrorAction Stop
} catch [System.IO.FileNotFoundException] {
  "文件不存在: $($_.Exception.Message)"
} catch {
  "其他错误: $($_.Exception.Message)"
  $_.ScriptStackTrace
} finally {
  "总是执行（清理）"
}

throw "自定义错误"                      # 主动抛出
$LASTEXITCODE                          # 外部程序（exe）退出码
if ($LASTEXITCODE -ne 0) { throw "命令失败" }
trap { "捕获到: $_" ; continue }        # 脚本级 trap

# 输出流（写日志用）
Write-Output "进管道"        # 1
Write-Host  "只显示"         # 不进管道（PS5+ 可被捕获）
Write-Verbose "详细" -Verbose
Write-Debug "调试" -Debug
Write-Warning "警告"         # 3
Write-Error "错误"           # 2
Write-Information "信息"     # 6
Write-Progress -Activity "处理中" -PercentComplete 50
$VerbosePreference = 'Continue' ; $DebugPreference = 'Continue'
```

### 5.10 引号、转义与字符串

```powershell
'单引号：不插值，$name 原样输出'
"双引号：会插值 $name，子表达式用 $($obj.Prop)"
"`t 制表  `n 换行  `r 回车  `` 反引号  `" 双引号  `$ 美元符  `0 空"
"路径 C:\temp\file"        # 反斜杠不是转义符，正常写
'包含 ''单引号'''
"包含 `"双引号`""

# here-string（@ 后必须紧跟换行）
$sql = @"
SELECT * FROM t WHERE name = '$name'
"@
$raw = @'
不插值 $name
'@

# 常用字符串操作
"abc".Length ; "abc".ToUpper() ; "  a  ".Trim()
"a,b,c" -split ','                 # 数组
"a,b,c".Split(',')
@('a','b') -join '-'
"hello" -replace 'l', 'L'
"hello" -like "h*"
"hello" -match '^h(.*)o$' ; $Matches[1]
"path/to" -replace '/', '\'
[String]::IsNullOrWhiteSpace($s)
"{0} 有 {1} 个" -f $name, $count
$s.PadLeft(5,'0') ; $s.Substring(0,3) ; $s.Contains('x')
[System.Uri]::EscapeDataString("a b")
```

**双引号里必须 `$()` 的场景**：属性访问、方法调用、索引、表达式。

```powershell
"$obj.Name"        # 错：只会插值 $obj，然后输出字面量 .Name
"$($obj.Name)"     # 对
"共 $($list.Count) 条"
```

### 5.11 重定向与管道

```powershell
Get-Process > out.txt            # 重定向（其实是 Out-File 的简写，带格式化）
Get-Process | Out-File out.txt -Encoding utf8
Get-Process | Set-Content out.txt          # 纯文本，无表格填充
"text" >> out.txt                          # 追加
cmd 2> err.txt                             # 错误流
cmd *> all.txt                             # 所有流（PS3+）
cmd 2>&1                                   # 错误并入输出
cmd > $null                                # 丢弃
Get-Content in.txt | ForEach-Object { $_ }
Get-Content a.txt, b.txt                   # 多文件
Get-Content a.txt -TotalCount 10           # 前 10 行
Get-Content a.txt -Tail 10                 # 后 10 行
Get-Content a.txt -Tail 5 -Wait            # 类似 tail -f（PS3+）
Get-Content a.txt -Raw                     # 整块读成一个字符串
Select-String -Path *.log -Pattern 'ERROR' -Context 2,2
Get-ChildItem | Tee-Object -FilePath list.txt
$data | Export-Csv out.csv -NoTypeInformation -Encoding utf8
$data | ConvertTo-Json -Depth 5 | Set-Content out.json -Encoding utf8
Import-Csv in.csv | Where-Object Status -eq 'Active'
$json = Get-Content d.json -Raw | ConvertFrom-Json
```

### 5.12 输出与格式

```powershell
# 格式化（只为显示，不要用于后续处理）
Format-Table Name, CPU -AutoSize
Format-List *                    # 看对象全部属性
Format-Wide Name -Column 4
Format-Custom

# 输出
Out-Host / Out-String / Out-GridView / Out-Printer
Write-Output "值"                # 进管道
Write-Host "彩色" -ForegroundColor Green -BackgroundColor Black

# 选取与展开
Select-Object Name, CPU                    # 挑属性
Select-Object -First 5 / -Last 5 / -Skip 2
Select-Object -Unique                      # 去重
Select-Object -ExpandProperty Name         # 掏成纯字符串数组
Select-Object @{N='大小MB';E={[math]::Round($_.Length/1MB,1)}}   # 计算属性

# 排序 / 分组 / 统计 / 比较
Sort-Object CPU -Descending
Sort-Object @{E={$_.Length}; Descending=$true}
Group-Object Extension | Sort-Object Count -Descending
Measure-Object -Property Length -Sum -Average -Maximum -Minimum
Compare-Object $a $b -PassThru
Get-ChildItem | Where-Object { $_.Length -gt 1MB }
Get-ChildItem | Where-Object Name -like '*.log'      # 简化语法（PS3+）
Get-ChildItem | Where-Object { $_.Name -match '^\d+' }

# 进度与计时
Measure-Command { Get-ChildItem -Recurse }
```

---

## 六、PowerShell 核心命令

### 6.1 文件与目录

```powershell
Get-Location                       # pwd
Set-Location C:\Temp               # cd
Push-Location / Pop-Location       # 压栈/弹栈（临时切目录）
Get-ChildItem                      # ls
Get-ChildItem -Force               # 含隐藏
Get-ChildItem -Recurse -File -Filter *.log
Get-ChildItem -Directory
Get-ChildItem C:\ -Depth 1
Get-ChildItem | Where-Object Length -gt 1MB | Sort-Object Length -Descending
New-Item -ItemType Directory -Path .\a\b -Force
New-Item -ItemType File -Path .\a.txt -Force
ni .\n.txt                          # 别名
Copy-Item a.txt b.txt
Copy-Item -Recurse -Path .\src -Destination .\dst
Move-Item a.txt .\sub\
Rename-Item a.txt b.txt
Rename-Item -Path .\x -NewName { $_.Name -replace ' ','_' }   # 批量改名
Remove-Item a.txt
Remove-Item -Recurse -Force .\dir
Get-Item a.txt | Select-Object *
Test-Path .\a.txt                   # 存在与否（返回布尔）
Test-Path -PathType Container .\dir
Resolve-Path .\a\..\b               # 规范路径
(Get-Item .\a.txt).FullName         # 绝对路径
Split-Path C:\a\b.txt -Leaf         # b.txt
Split-Path C:\a\b.txt -Parent       # C:\a
Join-Path $HOME 'docs' 'x.txt'
Get-Content a.txt                   # cat
Set-Content a.txt "内容"            # 覆盖写
Add-Content a.txt "追加"            # 追加
Clear-Content a.txt                 # 清空
Out-File a.txt -Encoding utf8
Get-Item a.txt | Format-List *
Get-FileHash a.txt -Algorithm SHA256
(Get-Content a.txt -Raw) -replace 'old','new' | Set-Content a.txt   # 改文件
Get-ChildItem -Recurse *.log | Remove-Item -WhatIf                 # 先预演！
New-Item -ItemType SymbolicLink -Path link -Target target          # 软链接（需权限）
Get-Acl a.txt ; Set-Acl ...
Get-Volume ; Get-PSDrive                    # 磁盘
```

**编码坑（重点）**：PS 5.1 的 `>`、`Out-File`、`Set-Content` 默认写 **UTF-16LE** 或系统 ANSI，容易产生乱码/带 BOM。
建议：**统一写 `-Encoding utf8`**（PS7 里 `utf8` = 无 BOM；PS5.1 里 `utf8` = 带 BOM，需要 `[IO.File]::WriteAllText($p,$s,[Text.UTF8Encoding]::new($false))` 才能无 BOM）。

### 6.2 进程、服务、计划任务

```powershell
Get-Process                                   # 进程列表
Get-Process -Name chrome
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10 Name, Id, CPU, WS
Get-Process | Where-Object WS -gt 500MB
Stop-Process -Id 1234
Stop-Process -Name notepad -Force
Start-Process notepad
Start-Process -FilePath cmd.exe -ArgumentList '/c','dir' -Wait -NoNewWindow
Start-Process powershell -Verb RunAs          # 提权
Wait-Process -Id 1234
Get-Process | Where-Object {$_.Path -like '*python*'} | Select-Object Id, Path

Get-Service                                   # 服务
Get-Service -Name w32time | Select-Object *
Start-Service / Stop-Service / Restart-Service -Name spooler
Set-Service -Name spooler -StartupType Automatic
Get-Service | Where-Object Status -eq 'Running'
Get-CimInstance Win32_Service | Where-Object State -eq 'Running' | Select-Object Name, StartMode, PathName

Get-ScheduledTask                             # 计划任务
Get-ScheduledTask -TaskName 'X' | Get-ScheduledTaskInfo
Start-ScheduledTask -TaskName 'X'
Register-ScheduledTask -TaskName 'Backup' -Trigger (New-ScheduledTaskTrigger -Daily -At 3am) `
  -Action (New-ScheduledTaskAction -Execute 'pwsh.exe' -Argument '-File C:\b.ps1') -RunLevel Highest
Unregister-ScheduledTask -TaskName 'Backup' -Confirm:$false

# 提权与登录信息
whoami ; whoami /groups ; $env:USERNAME
[Security.Principal.WindowsIdentity]::GetCurrent().Name
([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole(
  [Security.Principal.WindowsBuiltInRole]::Administrator)
```

### 6.3 系统信息与注册表

```powershell
Get-ComputerInfo | Select-Object WindowsProductName, OsVersion, CsName, CsTotalPhysicalMemory
Get-CimInstance Win32_OperatingSystem | Select-Object Caption, Version, LastBootUpTime
(Get-CimInstance Win32_OperatingSystem).LastBootUpTime
Get-CimInstance Win32_LogicalDisk -Filter "DriveType=3" | Select-Object DeviceID, @{N='FreeGB';E={[math]::Round($_.FreeSpace/1GB,1)}}
Get-CimInstance Win32_Processor | Select-Object Name, NumberOfCores
Get-CimInstance Win32_PhysicalMemory | Select-Object Capacity, Speed
Get-CimInstance Win32_NetworkAdapterConfiguration -Filter "IPEnabled=1" | Select-Object Description, IPAddress, DefaultIPGateway
Get-CimInstance Win32_Product          # 慢，慎用
Get-HotFix                             # 补丁
Get-EventLog -LogName System -Newest 20            # 老式（5.1）
Get-WinEvent -LogName System -MaxEvents 20         # 推荐
Get-WinEvent -FilterHashtable @{LogName='Application'; Level=2; StartTime=(Get-Date).AddHours(-1)}
Get-EventLog -LogName System -EntryType Error -Newest 10

# 注册表（像文件系统一样操作）
Get-ChildItem HKCU:\Software
Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion' | Select-Object ProductName, DisplayVersion
Get-ItemPropertyValue 'HKCU:\Control Panel\Desktop' -Name Wallpaper
New-Item -Path HKCU:\Software\MyApp -Force
New-ItemProperty -Path HKCU:\Software\MyApp -Name Ver -Value '1.0' -PropertyType String
Set-ItemProperty -Path HKCU:\Software\MyApp -Name Ver -Value '1.1'
Remove-ItemProperty -Path HKCU:\Software\MyApp -Name Ver
Remove-Item -Path HKCU:\Software\MyApp -Recurse
Get-PSDrive                            # 含 HKLM/HKCU 两个注册表驱动器
Test-Path 'HKLM:\SOFTWARE\Microsoft'
```

### 6.4 网络与 Web

```powershell
Test-Connection 8.8.8.8 -Count 4          # ping
Test-NetConnection example.com -Port 443  # 端口连通性（很好用）
Test-NetConnection -ComputerName x -InformationLevel Detailed
Resolve-DnsName example.com               # DNS
Get-NetIPAddress ; Get-NetIPConfiguration ; Get-NetRoute
Get-NetTCPConnection -State Listen | Select-Object LocalAddress, LocalPort, OwningProcess
Get-NetTCPConnection -LocalPort 8080 | Select-Object OwningProcess
Get-Process -Id (Get-NetTCPConnection -LocalPort 8080).OwningProcess   # 谁占了端口
Get-NetFirewallRule -Enabled True | Select-Object DisplayName, Direction, Action
New-NetFirewallRule -DisplayName 'App 8080' -Direction Inbound -LocalPort 8080 -Protocol TCP -Action Allow
Remove-NetFirewallRule -DisplayName 'App 8080'

# 下载与 API
Invoke-WebRequest https://example.com -OutFile index.html
Invoke-RestMethod https://api.github.com/repos/PowerShell/PowerShell      # 自动解析 JSON
$r = Invoke-RestMethod -Uri $u -Method Post -ContentType 'application/json' -Body (@{a=1} | ConvertTo-Json)
Invoke-WebRequest -Uri $u -Headers @{ Authorization = "Bearer $tok" }
iwr https://x -OutFile f.zip ; (Get-Item f.zip).Length
# 真 curl（Windows 10+ 自带 curl.exe，注意要写扩展名）
curl.exe -sSL -o f.zip https://x
# SSH / SCP（Win10+ 自带 OpenSSH）
ssh user@host ; scp file user@host:/tmp/ ; ssh-keygen -t ed25519
```

### 6.5 远程管理

```powershell
# 1) PowerShell 远程（WSMan，Windows 默认 5985/5986）
Enable-PSRemoting -Force                 # 在被控端启用（管理员）
Enter-PSSession -ComputerName SRV1 -Credential (Get-Credential)   # 交互
Invoke-Command -ComputerName SRV1 -ScriptBlock { Get-Service spooler }
Invoke-Command -ComputerName SRV1,SRV2 -ScriptBlock { $env:COMPUTERNAME }
$s = New-PSSession -ComputerName SRV1
Invoke-Command -Session $s -ScriptBlock { Get-Date }
Copy-Item -ToSession $s .\a.txt -Destination C:\tmp\
Remove-PSSession $s

# 2) SSH 远程（跨平台，PS7 推荐）
$s = New-PSSession -HostName host -UserName user -KeyFilePath ~/.ssh/id_ed25519
Invoke-Command -Session $s -ScriptBlock { uname -a }

# 3) 其他
Get-CimInstance -ClassName Win32_Service -ComputerName SRV1
Restart-Computer -ComputerName SRV1 -Force
```

### 6.6 作业与并行

```powershell
Start-Job { Get-Process } | Wait-Job | Receive-Job        # 后台作业（新进程，独立）
Get-Job ; Remove-Job -Id 1
Start-ThreadJob { 1..3 }                                  # 线程作业（更快，需模块）
ForEach-Object -Parallel { $_ * 2 } -ThrottleLimit 8      # PS7
1..20 | ForEach-Object -Parallel {
  Invoke-RestMethod "https://api/item/$_"
} -ThrottleLimit 5
$using:var                                                # 并行块里引用外部变量
```

### 6.7 模块与包管理

```powershell
Get-Module -ListAvailable | Select-Object Name, Version
Import-Module ActiveDirectory
Install-Module PSReadLine -Scope CurrentUser -Force
Update-Module
Find-Module *azure*
Install-PSResource / Find-PSResource          # PS7.4+ 新一代（替代 *-Module）
Get-InstalledModule

# Windows 包管理
winget search vscode ; winget install Microsoft.VisualStudioCode ; winget upgrade --all
choco install git -y                          # Chocolatey
scoop install curl                            # Scoop

# 自己的模块
New-ModuleManifest -Path .\My.psd1 -RootModule .\My.psm1
$env:PSModulePath -split ';'
```

### 6.8 配置文件与提示符

```powershell
$PROFILE                                # 当前用户配置文件路径
Test-Path $PROFILE                      # 可能不存在
New-Item -Path $PROFILE -ItemType File -Force
notepad $PROFILE                        # 编辑
. $PROFILE                              # 重新加载
```

`$PROFILE` 示例：

```powershell
Set-PSReadLineOption -PredictionSource History
Set-PSReadLineOption -PredictionViewStyle ListView
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete
Set-Alias ll Get-ChildItem
function ll { Get-ChildItem -Force @args }
function gs { git status -sb }
function .. { Set-Location .. }
function which($n) { (Get-Command $n).Source }
Import-Module PSReadLine
$OutputEncoding = [Console]::OutputEncoding = [Text.Encoding]::UTF8   # 中文乱码常见修复
Set-Location D:\WebService_Project\BlogData
```

---

## 七、对照表

### A. 日常操作对照

| 作用 | Bash | PowerShell |
| --- | --- | --- |
| 当前目录 | `pwd` | `Get-Location`（`pwd`） |
| 切换目录 | `cd /x` | `Set-Location C:\x`（`cd`） |
| 列文件 | `ls -la` | `Get-ChildItem -Force`（`ls`） |
| 递归列表 | `ls -R` / `find .` | `Get-ChildItem -Recurse` |
| 找文件 | `find . -name '*.log'` | `Get-ChildItem -Recurse -Filter *.log` |
| 建目录 | `mkdir -p a/b` | `New-Item -ItemType Directory a\b -Force` |
| 建空文件 | `touch a.txt` | `New-Item -ItemType File a.txt` |
| 复制 | `cp -r a b` | `Copy-Item -Recurse a b` |
| 移动/改名 | `mv a b` / `rename` | `Move-Item` / `Rename-Item` |
| 删除 | `rm -rf d` | `Remove-Item -Recurse -Force d` |
| 读文件 | `cat a.txt` | `Get-Content a.txt` |
| 写文件 | `echo x > a` | `"x" \| Set-Content a` |
| 追加 | `echo x >> a` | `"x" \| Add-Content a` |
| 前 10 行 | `head -10 a` | `Get-Content a -TotalCount 10` |
| 后 10 行 | `tail -10 a` | `Get-Content a -Tail 10` |
| 实时跟踪 | `tail -f a` | `Get-Content a -Tail 5 -Wait` |
| 存在判断 | `[ -f a ]` | `Test-Path a` |
| 文件信息 | `stat a` / `ls -l` | `Get-Item a \| Format-List *` |
| 绝对路径 | `realpath a` | `(Resolve-Path a).Path` |
| 文件名/目录名 | `basename` / `dirname` | `Split-Path -Leaf` / `-Parent` |
| 拼路径 | `"$a/$b"` | `Join-Path $a $b` |
| 当前时间 | `date +%F` | `Get-Date -Format 'yyyy-MM-dd'` |
| 时间计算 | `date -d '-1 day'` | `(Get-Date).AddDays(-1)` |
| 计时 | `time cmd` | `Measure-Command { cmd }` |
| 环境变量 | `echo $PATH` / `export A=1` | `$env:PATH` / `$env:A=1` |
| 查看进程 | `ps aux` | `Get-Process` |
| 找进程 | `pgrep -af nginx` | `Get-Process nginx` |
| 杀进程 | `kill -9 123` | `Stop-Process -Id 123 -Force` |
| 服务状态 | `systemctl status x` | `Get-Service x` |
| 服务重启 | `systemctl restart x` | `Restart-Service x` |
| 磁盘 | `df -h` | `Get-Volume` / `Get-PSDrive` |
| 目录大小 | `du -sh *` | `Get-ChildItem \| Measure-Object Length -Sum` |
| 网络配置 | `ip a` | `Get-NetIPAddress` |
| 监听端口 | `ss -tulnp` | `Get-NetTCPConnection -State Listen` |
| ping | `ping -c 4 h` | `Test-Connection h -Count 4` |
| 端口测试 | `nc -zv h 443` | `Test-NetConnection h -Port 443` |
| DNS | `dig h` | `Resolve-DnsName h` |
| 下载 | `curl -O url` | `Invoke-WebRequest url -OutFile f` |
| 调 API | `curl -s url \| jq` | `Invoke-RestMethod url` |
| 哈希 | `sha256sum f` | `Get-FileHash f -Algorithm SHA256` |
| 清屏 | `clear` | `Clear-Host` |
| 历史 | `history` | `Get-History`（`h`）；`Ctrl+R` 需 PSReadLine |
| 上条参数 | `!$` / `Alt+.` | `Alt+.`（PSReadLine 支持） |
| 权限 | `chmod 755 f` | `icacls f /grant ...` / `Set-Acl` |
| 属主 | `chown u:g f` | `Set-Acl` / `takeown` |
| 包管理 | `apt install x` | `winget install x` |
| 管道内容 | **文本** | **对象** |

### B. 文本处理对照

| 作用 | Bash | PowerShell |
| --- | --- | --- |
| 搜文本 | `grep -n "x" f` | `Select-String -Path f -Pattern x` |
| 递归搜 | `grep -rn x .` | `Get-ChildItem -Recurse \| Select-String x` |
| 忽略大小写 | `grep -i x f` | `Select-String x -CaseSensitive:$false`（默认不敏感） |
| 上下文 | `grep -C 2 x f` | `Select-String x -Context 2,2` |
| 计数匹配 | `grep -c x f` | `(Select-String x f).Count` |
| 反选 | `grep -v x f` | `Select-String x f -NotMatch` |
| 过滤行 | `grep x f` | `Get-Content f \| Where-Object { $_ -match 'x' }` |
| 替换 | `sed -i 's/a/b/g' f` | `(Get-Content f -Raw) -replace 'a','b' \| Set-Content f` |
| 取列 | `awk '{print $1}'` / `cut` | `Select-Object Prop` / `-split` / `-replace` |
| 去重 | `sort -u` | `Sort-Object -Unique` |
| 计数统计 | `sort \| uniq -c` | `Group-Object \| Sort-Object Count` |
| 排序 | `sort -k2 -n` | `Sort-Object Prop -Descending` |
| 求和/均值 | `awk '{s+=$1} END{print s}'` | `Measure-Object Prop -Sum -Average` |
| 取前 N | `head -5` | `Select-Object -First 5` |
| 取片段 | `cut -c1-10` | `.Substring(0,10)` |
| 分割 | `awk -F,` / `cut -d,` | `-split ','` |
| 连接 | `paste` | `-join ','` |
| 大小写 | `tr a-z A-Z` | `.ToUpper()` |
| 正则替换 | `sed -E 's/(\d+)/[\1]/'` | `-replace '(\d+)','[$1]'` |
| JSON | `jq .a.b` | `ConvertFrom-Json` → `$o.a.b` |
| CSV | `awk -F,` | `Import-Csv` / `Export-Csv` |
| Base64 | `base64 -w0 f` | `[Convert]::ToBase64String([IO.File]::ReadAllBytes($p))` |
| 十六进制 | `xxd f` | `Format-Hex f` |
| 逐行处理 | `while read l; do` | `ForEach-Object { }` |
| 输出同时看+存 | `tee f` | `Tee-Object -FilePath f` |

> 迁移提醒：Bash 里 `uniq -c`、`awk '{print $2}'` 这类"按列"操作，在 PowerShell 里要换成"按属性"：`Group-Object`、`Select-Object Prop`。而 `grep` 的语义要分清——想过滤**对象**用 `Where-Object`，想过滤**文本行**才用 `Select-String`。

### C. 符号对照

| 语法 | Bash | PowerShell |
| --- | --- | --- |
| 命令分隔 | `;` / `&&` / `\|\|` | `;` / `&&` / `\|\|`（**PS7 才有 && \|\|**，5.1 用 `-and`/`if`） |
| 管道 | `\|` | `\|` |
| 输出重定向 | `>` `>>` | `>` `>>`（= Out-File） |
| 错误重定向 | `2>` `2>&1` | `2>` `*> ` `2>&1` |
| 丢弃输出 | `> /dev/null` | `> $null` / `\| Out-Null` |
| 变量 | `$name` / `${name}` | `$name` / `${name}` |
| 变量插值 | `"$name"`（变量直接插） | `"$name"`，**属性必须 `$($o.P)`** |
| 字面字符串 | `'...'` | `'...'` |
| 可插值字符串 | `"..."` | `"..."` |
| 命令替换 | `$(cmd)` / `` `cmd` `` | `$(cmd)` / `(cmd)` 作为参数 |
| 转义符 | `\` | `` ` ``（反引号）；单引号内用 `''` |
| 取上条退出码 | `$?` | `$?`（布尔）/ `$LASTEXITCODE`（数值） |
| 正则 | `grep -E` / `sed -E` | `-match` / `-replace` / `-like` |
| 通配符 | `*` `?` `[a-z]` | `*` `?` `[a-z]`（`-like`） |
| 当前对象 | 无 | `$_` / `$PSItem` |
| 注释 | `#` / `: <<'EOF'` | `#` / `<# ... #>` |
| 续行 | 行尾 `\` | 行尾 `` ` `` 或管道/括号自然续行 |
| 空值 | `""` / `: ${x:=}` | `$null` |
| 布尔 | `true` / `false`（外部命令） | `$true` / `$false` |
| 数组字面量 | `(a b c)` | `@('a','b','c')` 或 `'a','b','c'` |
| 字典 | `declare -A m` | `@{ k = 'v' }` |
| 对象字面量 | 无 | `[pscustomobject]@{ A = 1 }` |
| 后台执行 | `cmd &` | `Start-Job { cmd }` / `Start-Process` |
| 变量作用域 | `local` / `export` | `$local:` / `$env:` / `$script:` / `$global:` |

---

## 八、踩坑清单

### 8.1 Bash 常见坑

1. **`=` 两边加空格** → `name = 1` 被当成执行 `name` 命令。
2. **不加引号的分词**：`rm $files` 遇到含空格的路径直接炸，写 `rm "$f"` / `"${arr[@]}"`。
3. **`[ ]` 里用 `>` `<`**：那是重定向，数字比较必须用 `-gt` / `-lt`。
4. **`cd` 在管道里失效**：`ls | while read d; do cd $d; done` 里的 cd 在子 Shell，出循环就丢。
5. **`set -e` 与 `grep`**：`grep` 无匹配返回 1，会把脚本整死，写 `grep x f || true`。
6. **`rm -rf $DIR/`**，`DIR` 为空时变成 `rm -rf /`。永远写 `rm -rf "${DIR:?}/"`。
7. **`tar -czf a.tar.gz *`** 不包含隐藏文件，用 `.` 或 `--exclude` 配合。
8. **`find | xargs` 遇到文件名带空格**：用 `-print0` + `xargs -0`。
9. **CRLF**：Windows 编辑过的 `.sh` 报 `bad interpreter: ^M`，用 `sed -i 's/\r$//' x.sh` 或 `dos2unix`。
10. **Ubuntu 的 `/bin/sh` 是 dash**：`[[ ]]`、`function`、数组都会失败，脚本头写 `#!/usr/bin/env bash`。
11. **`$(( ))` 只支持整数**，小数用 `awk` 或 `bc`。
12. **通配符没匹配时原样保留**：`shopt -s nullglob` 可让 `*.log` 无匹配时展开为空。
13. **反引号嵌套困难**，统一用 `$( )`。
14. **在 `sudo` 后丢失环境变量**：用 `sudo -E` 或 `sudo VAR=1 cmd`。

### 8.2 PowerShell 常见坑

1. **`>` 不是字节级重定向**：它走 `Out-File`，带格式化、带编码（5.1 常出 UTF-16），二进制/精确输出要避开。
2. **编码乱码**：中文文件读写统一显式 `-Encoding utf8`；控制台乱码用 `[Console]::OutputEncoding = [Text.Encoding]::UTF8`。
3. **`&&` 在 5.1 不存在**：会报「`&&` 不是有效语句分隔符」，改用 `if ($?) { }` 或装 PS7。
4. **`"$obj.Name"` 只插值 `$obj`**，属性要 `"$($obj.Name)"`。
5. **比较不用 `==`**：用 `-eq`；`==` 在 PS 里不是运算符（`-eq` 对字符串默认不区分大小写，要严格用 `-ceq`）。
6. **`-eq` 遇数组会过滤**：`@(1,2,3) -eq 2` 返回 `2` 而不是 `$false`，判空/判断存在要用 `-contains` / `-in`。
7. **`$null` 与空数组**：`@() -eq $null` 结果是数组，判断用 `$null -eq $x`（把 `$null` 放左边）。
8. **`Where-Object { $_.X = 1 }`** 赋值写成了比较，应该 `-eq`。
9. **`Select-Object` 只是投影**，`$sel.Count` 之类的属性要先 `-ExpandProperty` 或取 `.Count` 前确认类型。
10. **`Remove-Item -Recurse -Force` 没有回收站**，先 `-WhatIf` 预演。
11. **执行策略**：脚本被拦是 `Get-ExecutionPolicy` 问题，不是文件坏了。
12. **路径含空格**：参数直接给字符串即可 `Get-Item 'C:\My Docs\a.txt'`；调用外部 exe 用 `--%` 停止解析或 `&` 调用运算符。
13. **调用外部命令传参**：`& curl.exe @args`；参数里有 `-`、`=` 时注意 PS 会先解析。
14. **`Get-Content` 默认逐行返回数组**，字符串操作用 `-Raw`。
15. **`-Filter` 比 `-Include` 快得多**，且 `-Include` 需配合 `-Recurse` 或通配路径才生效。
16. **`$_` 在嵌套管道里会被覆盖**：先存 `$item = $_` 再进内层。
17. **`Measure-Object` 要指定 `-Property`**，否则只统计行数/字符数。
18. **`Write-Host` 不进管道**，想让下游拿到值用 `Write-Output`。

### 8.3 跨平台 / WSL 建议

- Windows 上用 **Windows Terminal + pwsh 7 + PSReadLine**，中文与颜色体验最好。
- Linux 命令想原样跑：装 **WSL2**（`wsl --install`），在里面用 bash；Git Bash 也能跑大部分 bash 脚本。
- 路径转换：`wsl wslpath -u 'C:\Temp'` → `/mnt/c/Temp`；`wslpath -w /mnt/c/Temp` → `C:\Temp`。
- 换行符：Git 配置 `core.autocrlf=input`（Linux 仓库）避免 CRLF 污染脚本。
- 想同时用：`wsl bash -lc "命令"` 从 PowerShell 直接调 bash。

---

## 九、实战片段

### 9.1 日志排查

```bash
# 过去 1 小时错误 Top 10 消息
journalctl --since "1 hour ago" -p err --no-pager | awk '{$1=$2=$3="";print}' | sort | uniq -c | sort -rn | head
# 跟踪某接口的 5xx
tail -f access.log | grep -E '" (5[0-9]{2}) '
# 找最慢的请求（假设最后一段是耗时 ms）
awk '{print $NF, $7}' access.log | sort -rn | head -20
```

```powershell
Get-WinEvent -FilterHashtable @{LogName='System'; Level=1,2; StartTime=(Get-Date).AddHours(-1)} |
  Group-Object ProviderName | Sort-Object Count -Descending | Select-Object -First 10 Count, Name
Get-Content .\access.log -Tail 200 -Wait | Select-String ' 5\d\d '
```

### 9.2 批量改名 / 批量处理

```bash
# 把当前目录 png 改成 001.png 形式
i=1; for f in *.png; do printf -v n '%03d.png' "$i"; mv -- "$f" "$n"; ((i++)); done
# 批量去空格
for f in *\ *; do mv -- "$f" "${f// /_}"; done
```

```powershell
$i = 0
Get-ChildItem *.png | Sort-Object Name | ForEach-Object {
  $i++
  Rename-Item $_.FullName -NewName ('{0:d3}.png' -f $i) -WhatIf
}
Get-ChildItem * | Where-Object Name -match ' ' | Rename-Item -NewName { $_.Name -replace ' ','_' }
```

### 9.3 磁盘 / 大文件清理

```bash
du -x -h --max-depth=1 / 2>/dev/null | sort -h | tail -15
find /var/log -type f -name '*.log' -mtime +14 -print -delete
find . -type f -size +1G -exec ls -lh {} \;
```

```powershell
Get-ChildItem C:\ -Directory -Force -ErrorAction SilentlyContinue | ForEach-Object {
  $s = (Get-ChildItem $_.FullName -Recurse -File -Force -ErrorAction SilentlyContinue |
        Measure-Object Length -Sum).Sum
  [pscustomobject]@{ Dir = $_.FullName; GB = [math]::Round($s/1GB, 2) }
} | Sort-Object GB -Descending | Select-Object -First 15

Get-ChildItem C:\Logs -Recurse -File -Include *.log |
  Where-Object LastWriteTime -lt (Get-Date).AddDays(-14) | Remove-Item -WhatIf
```

### 9.4 JSON / CSV 处理

```bash
# 无 jq 时
grep -o '"name":"[^"]*"' d.json | cut -d'"' -f4
# 有 jq
jq -r '.items[] | [.id, .name] | @tsv' d.json
# CSV 第 2 列求和
awk -F, 'NR>1 {s+=$2} END {print s}' d.csv
```

```powershell
$d = Get-Content d.json -Raw | ConvertFrom-Json
$d.items | Select-Object id, name | Format-Table
$d.items | Where-Object status -eq 'open' | Export-Csv open.csv -NoTypeInformation -Encoding utf8

$csv = Import-Csv d.csv
($csv | Measure-Object 金额 -Sum).Sum
$csv | Group-Object 部门 | Select-Object Name, Count
$csv | ConvertTo-Json -Depth 3 | Set-Content d.json -Encoding utf8
```

### 9.5 服务健康检查 / 定时任务

```bash
# crontab -e：每 5 分钟检查一次，失败发邮件
*/5 * * * * /usr/bin/curl -sf http://localhost:8080/health || echo "服务异常 $(date)" >> /var/log/health.log
0 3 * * * /opt/backup.sh >> /var/log/backup.log 2>&1
```

```powershell
# 注册每天 3 点的计划任务
$a = New-ScheduledTaskAction -Execute 'pwsh.exe' -Argument '-NoProfile -File C:\ops\backup.ps1'
$t = New-ScheduledTaskTrigger -Daily -At 3:00AM
$p = New-ScheduledTaskPrincipal -UserId 'SYSTEM' -RunLevel Highest
Register-ScheduledTask -TaskName 'DailyBackup' -Action $a -Trigger $t -Principal $p -Force
Get-ScheduledTask DailyBackup | Get-ScheduledTaskInfo
```

### 9.6 循环重试 / 幂等脚本

```bash
retry() {
  local tries=${RETRIES:-5} delay=${DELAY:-3} i=1
  until "$@"; do
    ((i++ > tries)) && { echo "重试 $tries 次仍失败: $*" >&2; return 1; }
    echo "第 $((i-1)) 次失败，${delay}s 后重试…" >&2
    sleep "$delay"
  done
}
retry curl -sSf https://api/health
```

```powershell
function Invoke-WithRetry {
  param([scriptblock]$Action, [int]$Retries = 5, [int]$DelaySec = 3)
  for ($i = 1; $i -le $Retries; $i++) {
    try { return & $Action }
    catch {
      if ($i -eq $Retries) { throw }
      Write-Warning "第 $i 次失败：$($_.Exception.Message)，${DelaySec}s 后重试"
      Start-Sleep -Seconds $DelaySec
    }
  }
}
Invoke-WithRetry { Invoke-RestMethod https://api/health }
```

---

## 附录

### A. 快捷键总表

| 快捷键 | Bash（readline） | PowerShell（PSReadLine） |
| --- | --- | --- |
| 中断命令 | `Ctrl+C` | `Ctrl+C` |
| 结束输入/退出 | `Ctrl+D` | `Ctrl+D`（空行时退出） |
| 清屏 | `Ctrl+L` | `Ctrl+L` |
| 搜索历史 | `Ctrl+R` | `Ctrl+R` |
| 历史下一条 | `Ctrl+S`（常被终端占用） | `Ctrl+S` / 方向键 |
| 行首 / 行尾 | `Ctrl+A` / `Ctrl+E` | `Home` / `End` |
| 删到行首 / 行尾 | `Ctrl+U` / `Ctrl+K` | `Ctrl+Home` / `Ctrl+End` |
| 删前一个词 | `Ctrl+W` | `Ctrl+Backspace` |
| 上条命令参数 | `Alt+.` | `Alt+.` |
| 补全 | `Tab` | `Tab`（可配 MenuComplete） |
| 挂起 | `Ctrl+Z` | 无（用 `Start-Job`） |
| 撤销 | `Ctrl+_` | `Ctrl+Z`（PSReadLine） |
| 反向删除 | `Ctrl+H` | `Backspace` |

### B. 一页速查（最常用的 40 条）

```bash
pwd ; ls -lah ; cd - ; tree -L 2
cat f ; less f ; head -20 f ; tail -f f ; wc -l f
grep -rn "x" . ; sed -i 's/a/b/g' f ; awk '{print $1}' f
find . -name '*.log' -mtime -7 ; du -sh * | sort -h ; df -h
ps aux | grep x ; kill -9 PID ; lsof -i:8080 ; top
ss -tulnp ; curl -I url ; dig +short h ; ssh user@host
tar -czvf a.tgz dir ; tar -xzvf a.tgz ; rsync -avz a/ h:/b/
chmod +x f ; chown u:g f ; sudo !! ; history | grep x
```

```powershell
Get-Location ; Get-ChildItem -Force ; Set-Location - ; Get-ChildItem -Recurse -File
Get-Content f ; Get-Content f -Tail 20 -Wait ; Measure-Object -Line
Select-String -Path f -Pattern x ; (Get-Content f -Raw) -replace 'a','b' | Set-Content f
Get-ChildItem -Recurse -Filter *.log | Where-Object LastWriteTime -gt (Get-Date).AddDays(-7)
Get-Process | Sort-Object WS -Descending | Select-Object -First 10
Stop-Process -Id PID -Force ; Get-NetTCPConnection -State Listen
Test-NetConnection h -Port 443 ; Resolve-DnsName h ; Invoke-RestMethod url
Compress-Archive dir a.zip ; Expand-Archive a.zip -DestinationPath out
Get-FileHash f ; Get-Help cmd -Examples ; Get-Member
```

### C. 常用环境变量

| 变量 | Bash | PowerShell |
| --- | --- | --- |
| PATH | `$PATH` | `$env:PATH` |
| 家目录 | `$HOME` | `$HOME` / `$env:USERPROFILE` |
| 当前用户 | `$USER` / `whoami` | `$env:USERNAME` / `whoami` |
| 主机名 | `$HOSTNAME` / `hostname` | `$env:COMPUTERNAME` |
| 语言/区域 | `$LANG` / `$LC_ALL` | `Get-Culture` |
| 临时目录 | `$TMPDIR` / `/tmp` | `$env:TEMP` |
| 编辑器 | `$EDITOR` | `$env:EDITOR` |
| 设置变量 | `export A=1`（会话内） | `$env:A = 1`（仅当前会话/子进程） |
| 永久设置（用户） | `~/.bashrc`、`~/.profile` | `[Environment]::SetEnvironmentVariable('A','1','User')` |
| 永久设置（系统） | `/etc/environment`、`/etc/profile.d/*.sh` | `[Environment]::SetEnvironmentVariable('A','1','Machine')` |

### D. 退出码速查

| 码 | 含义 |
| --- | --- |
| 0 | 成功 |
| 1 | 一般错误（grep 无匹配、`false`） |
| 2 | 用法错误 / Shell 内置误用 |
| 126 | 找到但无法执行（无 x 权限） |
| 127 | 命令不存在 |
| 128+n | 被信号 n 终止（130 = Ctrl+C，137 = `kill -9` / OOM） |
| PowerShell | `$?`（布尔）、`$LASTEXITCODE`（外部程序数值） |

### E. 引号与转义速查

| 需求 | Bash | PowerShell |
| --- | --- | --- |
| 字面量字符串 | `'a$b'` | `'a$b'` |
| 插值字符串 | `"a$b"` | `"a$b"`，属性用 `"$($o.P)"` |
| 字符串里放同类引号 | `"it's"` / `'say "hi"'` | `"it's"` / `'say "hi"'` |
| 转义双引号 | `"say \"hi\""` | `"say `"hi`""` |
| 转义单引号 | `'it'\''s'` | `'it''s'` |
| 换行 | `$'a\nb'` / `printf` | `"a`nb"` |
| 制表 | `$'a\tb'` | `"a`tb"` |
| 字面 `$` | `\$` 或 `'$'` | `` `$ `` 或 `'$'` |
| 字面反斜杠 | `\\` | `\`（直接写） |
| 字面反引号 | `` \` `` | ` `` ` |

### F. 进一步学习

- Bash：`man bash`、`help <内置>`、ShellCheck（shellcheck.net / `shellcheck` 命令）、《Bash 指南》（TLDP ABS）。
- PowerShell：`about_*` 概念文档（`Get-Help about_Operators`）、`Get-Help <cmd> -Examples`、微软官方文档 learn.microsoft.com/powershell。
- 对照迁移：官方 `PowerShell` 别名表（`Get-Alias`）、`about_Comparison_Operators`。
- 通用：`tldr`（人话版 man：`tldr tar`）、`cheat.sh`、`explainshell.com`（粘贴命令逐段解释）。

---

> 维护建议：这份文档按"能抄就用"写，遇到新坑就补进第八章；两条主线别混——
> **Bash 想文本，PowerShell 想对象**，剩下都是查表。

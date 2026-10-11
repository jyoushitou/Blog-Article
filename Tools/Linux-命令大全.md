---
title: Linux 命令大全（详解手册）
author: 示例作者
date: 2026-01-01
tags: [linux, 命令, shell, 运维, 大全]
---

# Linux 命令大全（详解手册）

> 本文档系统汇总 Linux 日常使用、运维、排障中会用到的命令，按功能分类，每条命令给出**用途、常用选项、可直接复制的示例**。
> 约定：`$` 开头表示普通用户执行，`#` 开头表示需要 root 权限；`<...>` 为占位符，`[...]` 为可选项。
> 提示：几乎所有命令都能用 `man <命令>`、`<命令> --help`、`info <命令>` 查询官方文档。

[TOC]

---

## 〇、如何使用本文档

| 需求 | 看哪一章 |
| --- | --- |
| 刚接触 Linux，想跑起来 | 一、二、三 |
| 处理文本/日志/配置文件 | 四、五 |
| 权限、用户、sudo 问题 | 六、十三 |
| 进程卡死、CPU/内存高 | 七、八 |
| 磁盘满、分区、挂载 | 九、十 |
| 网络不通、端口、抓包 | 十一 |
| 装软件、服务启停 | 十二、十四 |
| 写脚本、定时任务 | 十五、十六、十七 |
| 内核参数、容器、安全 | 十八、十九、二十 |

**命令的通用结构**

```bash
command [选项] [参数]
# 例：
ls -lh /var/log
```

**三条自救命令**

```bash
man ls          # 查看手册，q 退出，/ 搜索
ls --help       # 快速查看选项
type ls         # 查看这个命令是什么类型（内置/别名/外部程序）
```

---

## 一、基础与入门

### 1.1 命令的类型与查找

```bash
type -a ls              # 查看命令类型（alias / builtin / file / keyword）
which python3           # 显示可执行文件路径（查 PATH）
whereis ls              # 显示二进制、源码、man 手册位置
whatis ls               # 一行说明
command -v docker       # 判断命令是否存在（脚本里最常用）
apropos "disk usage"    # 按关键词搜 man 手册
```

### 1.2 命令历史

```bash
history                 # 查看历史命令
history 20              # 最近 20 条
history | grep ssh      # 搜索历史
!!                      # 执行上一条命令
!123                    # 执行历史中编号 123 的命令
!ssh                    # 执行最近一条以 ssh 开头的命令
sudo !!                 # 用上条命令加 sudo 重跑（经典救场）
Ctrl+R                  # 反向增量搜索历史（最实用）
Ctrl+G                  # 退出历史搜索
history -c              # 清空当前会话历史
history -d 500          # 删除第 500 条
```

历史相关环境变量：

```bash
export HISTSIZE=10000          # 内存中保存条数
export HISTFILESIZE=20000      # 历史文件保存条数
export HISTTIMEFORMAT='%F %T ' # history 显示时间戳
export HISTCONTROL=ignoreboth  # 忽略重复与空格开头的命令
export HISTIGNORE='ls:cd:pwd'  # 不记录这些命令
```

### 1.3 关于“一切皆文件”与目录规范

| 路径 | 含义 |
| --- | --- |
| `/` | 根目录 |
| `/bin`、`/sbin` | 基础命令（多数发行版已软链到 `/usr/bin`、`/usr/sbin`） |
| `/etc` | 配置文件 |
| `/home` | 普通用户家目录 |
| `/root` | root 家目录 |
| `/var` | 可变数据：日志 `/var/log`、缓存、数据库 |
| `/tmp` | 临时文件，重启可能清空 |
| `/usr` | 用户程序与库（`/usr/local` 为手工安装） |
| `/opt` | 第三方大型软件 |
| `/proc` | 内核与进程的虚拟文件系统 |
| `/sys` | 内核设备与驱动的虚拟文件系统 |
| `/dev` | 设备文件 |
| `/mnt`、`/media` | 临时挂载点 / 可移动介质 |

### 1.4 开关机与运行级别

```bash
shutdown -h now            # 立刻关机
shutdown -h +10 "10分钟后维护"   # 10 分钟后关机并广播
shutdown -r now            # 立刻重启
shutdown -c                # 取消已计划的关机
reboot                     # 重启
poweroff                   # 关机
halt                       # 停止 CPU（一般等价关机）
systemctl reboot           # systemd 重启
systemctl poweroff         # systemd 关机
systemctl isolate rescue.target    # 进入救援模式
```

---

## 二、文件与目录操作

### 2.1 ls — 列出目录内容

```bash
ls                    # 列出当前目录
ls -l                 # 长格式（权限、属主、大小、时间）
ls -a                 # 显示隐藏文件（. 开头）
ls -lh                # 人类可读大小（K/M/G）
ls -ltr               # 按修改时间倒序，最新在最下（排障必备）
ls -R                 # 递归列出
ls -i                 # 显示 inode 号
ls -d */              # 只列出目录
ls -lS                # 按大小排序
ls --color=auto       # 彩色输出
ls -l --time-style=full-iso   # 完整时间格式
```

长格式含义：`-rw-r--r-- 1 root root 4096 Jan 1 10:00 file`

```text
-rw-r--r--  1  root  root  4096  Jan 1 10:00  file
│└┬┘└┬┘└┬┘  │   │     │     │        │        └ 文件名
│ │  │  └── 其他人权限         │        └ 修改时间
│ │  └───── 所属组权限         └ 大小（字节）
│ └──────── 属主权限
└────────── 文件类型：- 普通文件 d 目录 l 链接 c 字符设备 b 块设备 s socket p 管道
```

### 2.2 cd、pwd — 切换与显示目录

```bash
cd /etc              # 进入绝对路径
cd ..                # 上级目录
cd -                 # 返回上一次所在目录
cd ~                 # 家目录
cd                   # 同 cd ~
pwd                  # 显示当前绝对路径
pwd -P               # 显示物理路径（解析软链接）
```

### 2.3 创建与删除

```bash
mkdir dir                     # 创建目录
mkdir -p a/b/c                # 递归创建（父目录不存在也建）
mkdir -m 755 dir              # 指定权限创建
rmdir dir                     # 删除空目录
rmdir -p a/b/c                # 递归删除空目录链

rm file                       # 删除文件
rm -f file                    # 强制删除，不提示
rm -r dir                     # 递归删除目录
rm -rf dir                    # 递归强制删除（危险！）
rm -i *.log                   # 逐个确认
rm -- -weird.txt              # 删除以 - 开头的文件
rm ./-weird.txt               # 同上，另一种写法
```

> ⚠️ 危险命令清单：`rm -rf /`、`rm -rf /*`、`rm -rf $VAR/`（VAR 为空时等于 `/`）、`chmod -R 777 /`、`dd of=/dev/sda`、`> /dev/sda`、`mkfs.* /dev/sda`。生产环境禁止手抖，脚本里删变量路径前先 `echo` 确认。

### 2.4 cp、mv — 复制与移动

```bash
cp src dst                # 复制文件
cp -r dir1 dir2           # 递归复制目录
cp -a dir1 dir2           # 归档复制：保留权限/属主/时间/软链接（备份首选）
cp -p file dst            # 保留属性
cp -i file dst            # 覆盖前提示
cp -u src dst             # 仅当源更新时才复制
cp -v src dst             # 显示过程
cp file{,.bak}            # 快速备份：生成 file.bak
cp --parents src/a.txt /tmp/   # 复制时保留目录结构

mv old new                # 重命名
mv file /tmp/             # 移动
mv -i file /tmp/          # 覆盖前提示
mv -n file /tmp/          # 不覆盖已存在文件
mv -v *.log /tmp/         # 显示过程
mv dir1 dir2              # dir2 不存在则重命名；存在则移入 dir2 内
```

### 2.5 touch、stat、file — 元信息

```bash
touch file                       # 创建空文件 / 更新时间戳
touch -a file                    # 只改访问时间
touch -m file                    # 只改修改时间
touch -t 202601011200 file       # 指定时间 YYYYMMDDhhmm
touch -d "2 days ago" file       # 相对时间
stat file                        # 详细元信息（三种时间、inode、块数）
stat -c '%n %s %U %G %a' file    # 自定义输出：名 大小 属主 属组 权限
file image.bin                   # 判断文件真实类型
file -i file                     # 显示 MIME 类型
```

三种时间：`atime` 访问时间、`mtime` 内容修改时间、`ctime` 元数据（权限/属主）变更时间。

### 2.6 ln — 链接

```bash
ln file hardlink          # 硬链接（同一 inode，不能跨分区、不能链目录）
ln -s /opt/app/bin/run /usr/local/bin/run   # 软链接（符号链接，可跨分区）
ln -sfn /new/target /path/link              # 更新已有软链接（常用）
ls -l link                # 查看链接指向
readlink -f link          # 解析最终真实路径
```

### 2.7 通配符与花括号展开

```bash
*            # 任意多个字符（不含 /）
?            # 单个字符
[abc]        # a、b、c 中任一
[a-z]        # 范围
[!0-9]       # 非数字
{a,b,c}.txt  # 展开为 a.txt b.txt c.txt
{1..5}       # 1 2 3 4 5
f{01..03}    # f01 f02 f03
**/*.log     # 递归匹配（需 globstar，bash 4+：shopt -s globstar）
```

### 2.8 重定向与管道

```bash
cmd > file            # 标准输出覆盖写入
cmd >> file           # 标准输出追加
cmd 2> err.log        # 标准错误写入文件
cmd > out 2>&1        # 输出与错误都写 out（顺序不能反）
cmd &> out            # bash 简写，等价上一条
cmd > /dev/null 2>&1  # 丢弃全部输出（静默）
cmd < input           # 标准输入来自文件
cmd <<< "字符串"       # here-string
cmd1 | cmd2           # 管道：cmd1 的输出作为 cmd2 的输入
cmd | tee file        # 同时输出到屏幕和文件
cmd | tee -a file     # 追加模式
set -o pipefail       # 管道中任一命令失败则整体失败
```

Here Document（脚本中写多行内容）：

```bash
cat > /tmp/app.conf <<'EOF'
listen = 8080
debug = false
$PATH 不会被展开（因为引号包住了 EOF）
EOF

# 不阻止变量展开
cat <<EOF
当前用户：$USER
EOF
```

### 2.9 后台运行与作业控制

```bash
cmd &                 # 后台运行
jobs -l               # 查看当前 shell 的作业
fg %1                 # 把 1 号作业调到前台
bg %1                 # 让暂停的作业继续后台跑
Ctrl+Z                # 暂停当前前台进程
Ctrl+C                # 中断当前前台进程
nohup cmd &           # 退出终端后仍继续运行
nohup cmd > out.log 2>&1 &
disown -h %1          # 从作业表中移除，避免收到 HUP
setsid cmd            # 新会话运行，完全脱离终端（更彻底）
```

---

## 三、文件内容查看与编辑

### 3.1 cat、tac

```bash
cat file              # 输出全部内容
cat -n file           # 显示行号
cat -A file           # 显示不可见字符（排查 Windows 换行/隐藏字符）
cat file1 file2 > all # 合并文件
tac file              # 反向输出（从最后一行开始）
```

### 3.2 less / more — 分页查看

```bash
less file             # 打开分页器
less +F app.log       # 类似 tail -f，按 Ctrl+C 退出跟踪
```

less 内部快捷键：

| 按键 | 作用 |
| --- | --- |
| `空格` / `b` | 下一页 / 上一页 |
| `g` / `G` | 跳到文件头 / 文件尾 |
| `/关键词` | 向下搜索，`n` 下一个，`N` 上一个 |
| `?关键词` | 向上搜索 |
| `F` | 进入实时跟踪模式 |
| `-N` | 切换行号显示 |
| `q` | 退出 |

### 3.3 head、tail

```bash
head file             # 前 10 行
head -n 50 file       # 前 50 行
head -c 100 file      # 前 100 字节
tail file             # 后 10 行
tail -n 100 file      # 后 100 行
tail -f app.log       # 实时跟踪新增内容（最常用）
tail -F app.log       # 文件被轮转/重建后仍能跟踪（推荐）
tail -f app.log | grep ERROR
tail -n +2 file       # 从第 2 行开始输出到结尾（跳过表头）
```

### 3.4 wc

```bash
wc -l file            # 行数
wc -w file            # 单词数
wc -c file            # 字节数
wc -m file            # 字符数
wc -L file            # 最长行长度
cat file | wc -l      # 统计行数（管道常用）
```

### 3.5 diff、cmp、patch — 比较与打补丁

```bash
diff a.txt b.txt                  # 比较两文件
diff -u a.txt b.txt               # 统一格式（生成补丁）
diff -y a.txt b.txt               # 并排显示
diff -r dir1 dir2                 # 递归比较目录
diff -q dir1 dir2                 # 只报告是否不同
diff -w a b                       # 忽略空白差异
diff -i a b                       # 忽略大小写
diff a b > fix.patch              # 生成补丁
patch < fix.patch                 # 应用补丁
patch -R < fix.patch              # 回滚补丁
cmp a b                           # 二进制逐字节比较
```

### 3.6 vim 速查

```bash
vim file        # 打开
vi file         # 老版本编辑器（一般即 vim）
```

| 场景 | 按键 |
| --- | --- |
| 进入编辑 | `i` 光标前插入、`a` 后插入、`o` 下行新开 |
| 退出编辑 | `Esc` |
| 保存退出 | `:wq` 或 `ZZ` |
| 不保存退出 | `:q!` |
| 保存 | `:w` |
| 另存为 | `:w newfile` |
| 显示行号 | `:set nu` |
| 跳到行 | `:100` 或 `100G` |
| 搜索 | `/word`，`n` 下一个 |
| 替换当前行 | `:s/old/new/g` |
| 全文替换 | `:%s/old/new/g` |
| 全文替换并确认 | `:%s/old/new/gc` |
| 删除整行 | `dd`，`5dd` 删 5 行 |
| 复制/粘贴 | `yy` / `p` |
| 撤销/重做 | `u` / `Ctrl+r` |
| 可视块编辑 | `Ctrl+v` 选列，`I` 插入，`Esc` 生效 |
| 分屏 | `:sp` 水平、`:vsp` 垂直，`Ctrl+w` 切窗 |

```bash
vim -R file           # 只读打开
vim +100 file         # 打开并跳到第 100 行
vim +/ERROR file      # 打开并搜索 ERROR
vim -d a b            # diff 模式
```

### 3.7 nano / emacs

```bash
nano file             # 简单编辑器，Ctrl+O 保存，Ctrl+X 退出
emacs file            # Emacs，Ctrl+X Ctrl+S 保存，Ctrl+X Ctrl+C 退出
```

### 3.8 其他实用查看命令

```bash
nl file                     # 带行号输出（可控制格式）
column -t file              # 对齐成表格
rev file                    # 每行字符反转
strings binary              # 提取二进制中的可打印字符串
xxd file | head             # 十六进制查看
od -c file                  # 八进制/字符查看
hexdump -C file             # 十六进制 + ASCII
```

---

## 四、文本处理三剑客与查找

### 4.1 grep — 文本搜索

```bash
grep "error" app.log              # 搜索包含 error 的行
grep -i "error" app.log           # 忽略大小写
grep -r "TODO" ./src              # 递归搜索目录
grep -rn "TODO" ./src             # 递归 + 显示行号
grep -v "debug" app.log           # 反向匹配（排除）
grep -w "cat" file                # 全词匹配
grep -c "error" app.log           # 统计匹配行数
grep -l "error" *.log             # 只列出匹配的文件名
grep -L "error" *.log             # 列出不匹配的文件名
grep -A 3 "panic" app.log         # 匹配行及后 3 行
grep -B 3 "panic" app.log         # 匹配行及前 3 行
grep -C 3 "panic" app.log         # 前后各 3 行
grep -E "err(or|no)" file         # 扩展正则（等价 egrep）
grep -F "a.b" file                # 按字面量匹配，不当正则（快）
grep -o "[0-9]\+" file            # 只输出匹配部分
grep -n --color=always "x" file   # 高亮
grep -e pat1 -e pat2 file         # 多个模式
grep -f patterns.txt file         # 从文件读取模式
grep -q "x" file && echo yes      # 静默模式，只看退出码
grep -P "\d{3}" file              # PCRE 正则（需支持）
grep -z "x" file                  # 以 NUL 分隔处理
grep --exclude-dir=.git -rn "x" . # 排除目录
```

常用正则速记：

```text
^      行首          $      行尾
.      任意单字符     *      前一项 0 次或多次
\+     1 次或多次     \?     0 次或 1 次
[abc]  字符集         [^abc] 取反
\{3\}  恰好 3 次      \{2,5\} 2 到 5 次
\< \>  词首词尾       \|     或（BRE 中转义，ERE 中写 |）
[[:digit:]] 数字      [[:alpha:]] 字母   [[:space:]] 空白
```

### 4.2 sed — 流编辑器

```bash
sed 's/old/new/' file             # 每行替换第一个
sed 's/old/new/g' file            # 每行替换全部
sed -i 's/old/new/g' file         # 直接改文件（-i.bak 会先备份）
sed -n '5,10p' file               # 只打印 5-10 行
sed -n '/start/,/end/p' file      # 打印两个模式之间的内容
sed '3d' file                     # 删除第 3 行
sed '/^#/d' file                  # 删除注释行
sed '/^$/d' file                  # 删除空行
sed -n '1~2p' file                # 打印奇数行
sed 's/^/PREFIX /' file           # 每行前加内容
sed 's/$/ SUFFIX/' file           # 每行后加内容
sed -i '2i\新插入的行' file        # 第 2 行前插入
sed -i '2a\追加的行' file          # 第 2 行后追加
sed 's/[0-9]\+/N/g' file          # 正则替换
sed -E 's/(\w+)@(\w+)/\2:\1/' f   # 分组捕获与反向引用
sed -n '/ERROR/=' file            # 打印匹配行的行号
sed 'y/abc/xyz/' file             # 字符一一替换
```

### 4.3 awk — 按列处理

```bash
awk '{print $1}' file                  # 打印第 1 列
awk '{print $1, $NF}' file             # 第 1 列和最后一列
awk -F: '{print $1}' /etc/passwd       # 以 : 分隔
awk -F, '{print $2}' data.csv
awk 'NR==5' file                       # 第 5 行
awk 'NR>=2 && NR<=4' file              # 2-4 行
awk '/error/ {print $0}' app.log       # 匹配后打印
awk '$3 > 100 {print $1}' file         # 条件过滤
awk '{sum+=$1} END {print sum}' file   # 求和
awk '{sum+=$1} END {print sum/NR}' f   # 求平均
awk 'END {print NR}' file              # 行数
awk '{print NF}' file                  # 每行的字段数
awk '!seen[$0]++' file                 # 去重且保持顺序
awk '{a[$1]+=$2} END {for(k in a) print k, a[k]}' f   # 分组求和
awk 'BEGIN {printf "%-10s %s\n", "NAME", "SIZE"} {printf "%-10s %d\n", $1, $2}' f
awk -v OFS='\t' '{print $1,$2}' file   # 指定输出分隔符
awk '/start/,/end/' file               # 范围匹配
awk 'length($0) > 80' file             # 找长行
awk 'NF' file                          # 去掉空行
```

内置变量：`$0` 整行、`$1..$n` 各字段、`NF` 字段数、`NR` 当前行号、`FNR` 文件内行号、`FS` 输入分隔符、`OFS` 输出分隔符、`RS` 记录分隔符、`FILENAME` 文件名。

### 4.4 cut、paste、tr、sort、uniq — 行列加工

```bash
cut -d: -f1 /etc/passwd        # 以 : 分隔取第 1 列
cut -d: -f1,3 /etc/passwd      # 取 1 和 3 列
cut -c1-10 file                # 取每行 1-10 字符
cut -c5- file                  # 从第 5 字符到结尾
paste a.txt b.txt              # 按行左右合并
paste -d, a.txt b.txt          # 用逗号连接
paste -s file                  # 多行合并为一行
tr 'a-z' 'A-Z' < file          # 小写转大写
tr -d '\n' < file              # 删除换行
tr -s ' ' < file               # 压缩连续空格
tr -cd '[:print:]\n' < file    # 只保留可打印字符
tr ',' '\t' < in.csv           # 逗号转换 tab
```

```bash
sort file                      # 字典序排序
sort -n file                   # 按数字排序
sort -h file                   # 按人类可读大小排序（1K < 1M）
sort -r file                   # 逆序
sort -u file                   # 排序并去重
sort -k2 file                  # 按第 2 列排序
sort -t: -k3 -n /etc/passwd    # 以 : 分隔，按第 3 列数字排序
sort -k2,2 -k1,1 file          # 先按第 2 列再按第 1 列
sort -f file                   # 忽略大小写
sort -M file                   # 按月份名排序
sort -V file                   # 版本号排序
sort -o file file              # 原地排序（安全）
```

```bash
uniq file                      # 去掉相邻重复行
sort file | uniq               # 先排序再去重
uniq -c file                   # 统计每行出现次数
uniq -d file                   # 只显示重复过的行
uniq -u file                   # 只显示没重复过的行
sort file | uniq -c | sort -rn | head    # 出现次数 Top 10（日志分析神器）
```

### 4.5 find — 文件查找

```bash
find . -name "*.log"                       # 按名字（区分大小写）
find . -iname "*.log"                      # 忽略大小写
find /var -type f -name "*.log"            # 类型：f 文件 d 目录 l 链接
find . -maxdepth 2 -name "*.conf"          # 限制递归深度
find . -size +100M                         # 大于 100M
find . -size -1k                           # 小于 1K
find . -empty                              # 空文件或空目录
find . -mtime -7                           # 7 天内修改过
find . -mtime +30                          # 30 天前修改过
find . -mmin -10                           # 10 分钟内修改过
find . -atime +30                          # 30 天未访问
find . -newer ref.txt                      # 比 ref.txt 新
find . -user root                          # 属主为 root
find . -group www-data                     # 属组
find . -perm 644                           # 权限恰好 644
find . -perm -u+w                          # 属主有写权限
find . -perm /o+w                          # 其他人有写权限（安全审计）
find . -nouser -o -nogroup                 # 无有效属主/属组
find . -name "*.tmp" -delete               # 找到并删除
find . -name "*.log" -exec gzip {} \;      # 逐个执行
find . -name "*.log" -exec gzip {} +       # 批量执行（更高效）
find . -type f -exec chmod 644 {} +
find . -name "*.bak" -print0 | xargs -0 rm -f    # 处理含空格文件名
find . -type f | xargs grep -l "TODO"      # 对结果批量 grep
find . -type d -name node_modules -prune -o -name "*.js" -print   # 排除目录
find / -xdev -type f -size +1G 2>/dev/null  # 不跨文件系统找大文件
```

`find` 常用动作：`-print`、`-print0`、`-delete`、`-exec`、`-ok`（执行前确认）、`-ls`。

### 4.6 xargs — 参数传递

```bash
cmd | xargs rm -f                # 把标准输入变成参数
cmd | xargs -n1 echo             # 每次传 1 个参数
cmd | xargs -I{} cp {} /tmp/     # 占位符替换
cmd | xargs -0 ...               # 输入以 NUL 分隔（配合 find -print0）
cmd | xargs -P4 -n1 ...          # 4 个并行进程
cmd | xargs -t ...               # 执行前打印命令
cmd | xargs -r ...               # 输入为空时不执行
```

### 4.7 文本合并与拆分

```bash
split -l 1000 big.log part_        # 按 1000 行拆分
split -b 100M big.iso chunk_       # 按 100M 拆分
split -n 5 file part_              # 拆成 5 份
csplit file '/^CHAPTER/' '{*}'     # 按模式拆分
join -1 1 -2 1 a.txt b.txt         # 按共同字段连接两个文件
comm a.txt b.txt                   # 比较两个已排序文件（三列：仅A、仅B、共有）
comm -12 a.txt b.txt               # 只显示共同行
```

### 4.8 编码转换与加解密

```bash
iconv -f GBK -t UTF-8 in.txt -o out.txt      # 编码转换
iconv -l | grep -i utf                       # 列出支持的编码
dos2unix file                                # CRLF 转 LF
unix2dos file                                # LF 转 CRLF
base64 file                                  # 编码
base64 -d file.b64                           # 解码
base64 -w0 file                              # 不换行输出
openssl enc -aes-256-cbc -salt -in f -out f.enc    # 对称加密
openssl enc -d -aes-256-cbc -in f.enc -out f       # 解密
md5sum file                                  # MD5 校验值
sha256sum file                               # SHA256 校验值
sha256sum -c checksums.txt                   # 校验文件完整性
```

---

## 五、权限与所有权

### 5.1 chmod — 修改权限

数字表示法：

```text
r = 4   w = 2   x = 1
755 = rwxr-xr-x   文件：属主全权，其他人读+执行
644 = rw-r--r--   普通文件默认
600 = rw-------   私钥文件
700 = rwx------   私有目录
777 = rwxrwxrwx   所有人可读写执行（危险，别用）
```

```bash
chmod 755 script.sh            # 数字设置
chmod +x script.sh             # 加执行权限
chmod -x script.sh             # 去掉执行权限
chmod u+x,g-w,o-r file         # 符号方式：u 属主 g 属组 o 其他 a 全部
chmod -R 755 dir               # 递归
chmod -R u+rwX dir             # X：目录才有执行权限（递归修权限的正确姿势）
chmod --reference=ref file     # 照抄另一个文件的权限
chmod 4755 file                # SUID：执行时以属主身份运行
chmod 2755 dir                 # SGID：目录内新文件继承属组
chmod 1777 /tmp                # 粘滞位：只能删自己的文件
find / -perm -4000 -type f 2>/dev/null     # 查找所有 SUID 程序（安全审计）
```

特殊位含义：

| 位 | 数字 | 文件 | 目录 |
| --- | --- | --- | --- |
| SUID | 4000 | 以文件属主身份执行 | 无意义 |
| SGID | 2000 | 以文件属组身份执行 | 新文件继承目录属组 |
| Sticky | 1000 | 无意义 | 只能删除自己的文件 |

### 5.2 chown、chgrp — 修改属主属组

```bash
chown user file                # 改属主
chown user:group file          # 同时改属主和属组
chown :group file              # 只改属组
chown -R www-data:www-data /var/www    # 递归
chown --reference=ref file     # 参照其他文件
chown -h user symlink          # 只改链接本身，不跟随
chgrp group file               # 改属组
```

### 5.3 umask — 默认权限掩码

```bash
umask                  # 查看当前掩码（常见 022 或 002）
umask -S               # 符号形式显示
umask 027              # 设置：新文件 640，新目录 750
umask 077              # 私密：新文件 600，新目录 700
```

计算方式：文件基准 `666`、目录基准 `777` 减去 umask。`umask 022` → 文件 `644`、目录 `755`。

### 5.4 lsattr、chattr — 文件属性

```bash
lsattr file                  # 查看扩展属性
chattr +i file               # 不可修改（连 root 也不能改，除非先去掉）
chattr -i file               # 取消
chattr +a file               # 只能追加，不能删除/覆盖
chattr +e file               # 使用 extents（ext4 默认）
```

### 5.5 getfacl、setfacl — 精细权限（ACL）

```bash
getfacl file                                  # 查看 ACL
setfacl -m u:alice:rw file                    # 给 alice 读写
setfacl -m g:devs:rx dir                      # 给组权限
setfacl -m d:u:alice:rwx dir                  # 目录默认 ACL，新文件自动继承
setfacl -x u:alice file                       # 删除某条
setfacl -b file                               # 清空 ACL
setfacl -R -m u:alice:rX dir                  # 递归设置
```

---

## 六、用户与组管理

### 6.1 用户

```bash
useradd -m -s /bin/bash tom        # 创建用户并建家目录
useradd -m -G sudo,docker tom      # 附加组
useradd -r -s /sbin/nologin svc    # 系统账号，禁止登录
passwd tom                          # 设置密码
passwd -l tom                       # 锁定账号
passwd -u tom                       # 解锁
passwd -e tom                       # 强制下次登录改密码
passwd -S tom                       # 查看密码状态
usermod -aG sudo tom                # 追加到 sudo 组（-a 必须写，否则覆盖）
usermod -g devs tom                 # 改主组
usermod -s /bin/zsh tom             # 改登录 shell
usermod -d /new/home -m tom         # 改家目录并迁移
usermod -L tom / -U tom             # 锁定 / 解锁
userdel tom                         # 删除用户
userdel -r tom                      # 删除用户及家目录邮件池
chage -l tom                        # 查看密码有效期
chage -M 90 tom                     # 最长 90 天
chage -d 0 tom                      # 强制下次登录改密码
id tom                              # 查看 uid/gid/所属组
id -u tom                           # 只要 uid
whoami                              # 当前用户
groups tom                          # 用户所属组
finger tom                          # 用户详细信息（需安装）
```

### 6.2 组

```bash
groupadd devs                  # 创建组
groupadd -g 1500 devs          # 指定 GID
groupmod -n newname oldname    # 重命名
groupdel devs                  # 删除组
gpasswd -a tom devs            # 加入组
gpasswd -d tom devs            # 移出组
gpasswd -A tom devs            # 设为组管理员
newgrp devs                    # 临时切换主组（新 shell）
```

### 6.3 相关文件

| 文件 | 内容 |
| --- | --- |
| `/etc/passwd` | 用户信息：`用户名:x:UID:GID:说明:家目录:shell` |
| `/etc/shadow` | 密码哈希与有效期（仅 root 可读） |
| `/etc/group` | 组信息 |
| `/etc/gshadow` | 组密码 |
| `/etc/skel` | 新用户家目录模板 |
| `/etc/login.defs` | 用户与密码策略默认值 |
| `/etc/default/useradd` | useradd 默认行为 |

### 6.4 登录状态与切换

```bash
who                    # 当前登录用户
who -b                 # 上次启动时间
w                      # 登录用户及正在执行的命令
last                   # 登录历史（读取 /var/log/wtmp）
last -n 20             # 最近 20 条
lastb                  # 失败登录记录
lastlog                # 每个用户最后登录时间
users                  # 简洁列出登录用户名
su - tom               # 切换用户并加载其环境（- 很重要）
su -                   # 切换到 root
sudo -i                # 以 root 交互式登录 shell
sudo -s                # 以 root 启动 shell（不加载 profile）
sudo -u www-data cmd   # 以其他用户执行
sudo -l                # 查看当前用户可用的 sudo 规则
sudo -k                # 清除 sudo 密码缓存
visudo                 # 安全编辑 /etc/sudoers（带语法检查）
```

sudoers 常用写法：

```text
tom   ALL=(ALL:ALL) ALL                 # tom 可执行所有命令
%sudo ALL=(ALL:ALL) ALL                 # sudo 组成员可用
deploy ALL=(root) NOPASSWD: /bin/systemctl restart nginx
Defaults timestamp_timeout=15           # 15 分钟内免密
Defaults !requiretty                    # 允许非 tty（脚本/自动化）
```

---

## 七、进程管理

### 7.1 ps — 查看进程

```bash
ps aux                       # 所有进程（BSD 风格，最常用）
ps -ef                       # 所有进程（System V 风格，含 PPID）
ps -ef --forest              # 树状显示父子关系
ps -u tom                    # 指定用户的进程
ps -p 1234 -o pid,ppid,cmd   # 指定 PID 的自定义列
ps aux --sort=-%mem | head   # 按内存降序
ps aux --sort=-%cpu | head   # 按 CPU 降序
ps -eLf                      # 显示线程
ps -eo pid,ppid,user,%cpu,%mem,etime,cmd --sort=-%cpu | head
pgrep -u tom nginx           # 按名字找 PID
pgrep -af python             # 名字+完整命令
pstree -p                    # 进程树带 PID
pstree -u                    # 显示用户切换
```

`ps aux` 列含义：`USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND`

| STAT | 含义 |
| --- | --- |
| R | 运行中 |
| S | 可中断睡眠（等待事件） |
| D | 不可中断睡眠（多为 IO 等待） |
| T | 已停止（Ctrl+Z） |
| Z | 僵尸进程 |
| s | 会话首进程 |
| l | 多线程 |
| + | 前台进程组 |
| < | 高优先级 |
| N | 低优先级 |

### 7.2 top / htop — 实时监控

```bash
top                # 打开实时监控
top -u tom         # 只看某用户
top -p 1234,5678   # 只看指定 PID
top -b -n1 > snap.txt   # 批处理模式抓一次快照（脚本用）
htop               # 更友好的交互式监控（需安装）
btop / glances     # 更现代的资源监控（需安装）
```

top 交互键：`P` 按 CPU 排序、`M` 按内存排序、`T` 按运行时间、`k` 杀进程、`r` renice、`1` 展开每个 CPU、`c` 显示完整命令、`H` 显示线程、`q` 退出。

### 7.3 kill、pkill、killall — 终止进程

```bash
kill 1234              # 发送 SIGTERM（15，优雅退出）
kill -9 1234           # SIGKILL，强杀（最后手段）
kill -1 1234           # SIGHUP，常用于让服务重载配置
kill -l                # 列出所有信号
kill -0 1234           # 探测进程是否存在（不发信号）
pkill nginx            # 按名字杀
pkill -f "python app.py"   # 按完整命令行匹配
pkill -u tom           # 杀某用户所有进程
killall nginx          # 按精确进程名杀
killall -9 -r 'worker.*'   # 正则匹配
```

常用信号：

| 信号 | 编号 | 说明 |
| --- | --- | --- |
| SIGHUP | 1 | 挂起/重载配置 |
| SIGINT | 2 | Ctrl+C 中断 |
| SIGQUIT | 3 | 退出并 core dump |
| SIGKILL | 9 | 强制杀死，不可捕获 |
| SIGSEGV | 11 | 段错误 |
| SIGTERM | 15 | 默认终止信号，可被捕获 |
| SIGSTOP | 19 | 暂停，不可捕获 |
| SIGCONT | 18 | 继续运行 |

### 7.4 优先级与资源限制

```bash
nice -n 10 cmd             # 以低优先级启动（-20 最高，19 最低）
nice -n -5 cmd             # 提高优先级（需 root）
renice -n 5 -p 1234        # 调整运行中进程
renice -n 5 -u tom         # 调整某用户所有进程
ionice -c3 -p 1234         # IO 优先级：1 实时 2 尽力 3 空闲
taskset -c 0,1 cmd         # 绑定到指定 CPU 核
taskset -cp 2 1234         # 修改运行中进程的绑核
ulimit -a                  # 查看所有资源限制
ulimit -n 65535            # 最大打开文件数（当前 shell）
ulimit -u 4096             # 最大进程数
ulimit -c unlimited        # 允许 core dump
prlimit --pid 1234         # 查看进程限制
chrt -f 50 cmd             # 以实时调度策略运行
```

### 7.5 进程信息深入

```bash
ls -l /proc/1234/          # 进程的虚拟目录
cat /proc/1234/cmdline | tr '\0' ' '; echo    # 启动命令
cat /proc/1234/environ | tr '\0' '\n'         # 环境变量
ls -l /proc/1234/cwd       # 工作目录
ls -l /proc/1234/exe       # 可执行文件
ls -l /proc/1234/fd        # 打开的文件描述符（排查文件句柄泄漏）
cat /proc/1234/limits      # 资源限制
cat /proc/1234/status      # 状态汇总
pmap 1234                  # 内存映射
lsof -p 1234               # 进程打开的所有文件
strace -p 1234             # 跟踪系统调用（排障利器）
strace -f -e trace=network -p 1234
ltrace -p 1234             # 跟踪库函数调用
```

### 7.6 后台服务化与进程守护

```bash
# 用 nohup 简单常驻
nohup ./app > app.log 2>&1 &

# 用 systemd 管理（推荐，见第十四章）
sudo systemctl enable --now myapp

# 用 supervisor 管理（需安装）
supervisorctl status
```

---

## 八、系统信息与性能监控

### 8.1 基础信息

```bash
uname -a                   # 内核与系统全部信息
uname -r                   # 内核版本
uname -m                   # 架构（x86_64 / aarch64）
hostname                   # 主机名
hostnamectl                # systemd 主机信息（发行版、内核、虚拟化）
hostnamectl set-hostname newname    # 修改主机名
cat /etc/os-release        # 发行版信息
lsb_release -a             # 发行版信息（需安装）
uptime                     # 运行时长与负载
uptime -p                  # 人类可读格式
date                       # 当前时间
date +"%Y-%m-%d %H:%M:%S"  # 格式化输出
date -d "yesterday" +%F    # 昨天的日期
date -d "+7 days" +%F      # 7 天后
cal                        # 日历
timedatectl                # 时间与时区状态
timedatectl set-timezone Asia/Shanghai
timedatectl set-ntp true   # 启用 NTP 同步
locale                     # 语言环境
arch                       # 架构
nproc                      # CPU 核数
getconf LONG_BIT           # 位数
```

### 8.2 CPU

```bash
lscpu                      # CPU 架构详情
cat /proc/cpuinfo          # CPU 详细信息
nproc --all                # 逻辑核数
mpstat -P ALL 1            # 每个核的使用率（sysstat 包）
mpstat 1 5                 # 每秒采样，共 5 次
sar -u 1 5                 # CPU 历史/实时统计
vmstat 1 5                 # 虚拟内存与 CPU 概览
```

### 8.3 内存

```bash
free -h                    # 内存使用（人类可读）
free -m                    # 以 MB 显示
free -s 2                  # 每 2 秒刷新
cat /proc/meminfo          # 内存详情
vmstat -s                  # 内存统计摘要
slabtop                    # 内核 slab 缓存占用
```

`free -h` 列含义：`total` 总量、`used` 已用、`free` 空闲、`shared` 共享、`buff/cache` 缓存（可回收）、`available` **真正可用**（关注这个）。

### 8.4 综合监控

```bash
vmstat 1 10                # 每秒采样 10 次：进程、内存、swap、IO、CPU
iostat -xz 1               # 磁盘 IO 详细（sysstat）
iostat -d -m 1             # 每秒磁盘吞吐（MB）
sar -n DEV 1 5             # 网络统计
sar -r 1 5                 # 内存统计
sar -q 1 5                 # 负载队列
sar -f /var/log/sa/sa15    # 查看历史某天数据
pidstat 1                  # 按进程统计 CPU/IO
pidstat -d 1               # 进程 IO
pidstat -r 1               # 进程内存
pidstat -w 1               # 进程上下文切换
dstat -tcmnd --disk-util   # 综合监控（需安装 dstat）
nmon                       # 交互式性能监控
```

### 8.5 负载与上下文切换

```bash
uptime                     # 1/5/15 分钟平均负载
cat /proc/loadavg          # 负载原始值
vmstat 1                   # cs 列：上下文切换次数，in 列：中断次数
```

负载解读：单核 CPU 负载持续大于 1 说明有排队；4 核机器负载 4 表示刚好跑满。用 `nproc` 对比着看。

### 8.6 内核与硬件信息

```bash
dmesg                      # 内核环形缓冲区（开机日志、硬件错误）
dmesg -T                   # 带可读时间戳
dmesg -w                   # 实时跟踪
dmesg -l err,crit          # 只看错误级别
dmesg | grep -i "oom"      # 查 OOM 杀进程记录
lsblk                      # 块设备树
lspci                      # PCI 设备（显卡、网卡）
lspci -nnk                 # 含驱动信息
lsusb                      # USB 设备
lsmod                      # 已加载内核模块
modinfo e1000e             # 模块信息
lshw -short                # 硬件总览（需安装）
dmidecode -t memory        # 内存条信息（需 root）
hwinfo --short             # 硬件信息（需安装）
```

### 8.7 日志查看

```bash
journalctl -xe                   # 最近日志与错误说明
journalctl -u nginx              # 某服务日志
journalctl -u nginx -f           # 实时跟踪服务日志
journalctl -u nginx --since "1 hour ago"
journalctl --since today --until "2026-01-01 12:00"
journalctl -p err -b             # 本次启动以来的错误
journalctl -b -1                 # 上次启动的日志
journalctl -k                    # 内核日志（同 dmesg）
journalctl --disk-usage          # 日志占用空间
journalctl --vacuum-size=500M    # 清理到 500M
journalctl --vacuum-time=7d      # 只保留 7 天
journalctl _PID=1234             # 按 PID 过滤
journalctl -n 100                # 最近 100 行
journalctl -o json-pretty        # JSON 格式

# 传统日志文件
tail -f /var/log/syslog                 # Debian/Ubuntu
tail -f /var/log/messages               # RHEL/CentOS
tail -f /var/log/auth.log               # 认证日志
tail -f /var/log/secure                 # RHEL 认证日志
tail -f /var/log/nginx/access.log
last -f /var/log/wtmp                   # 登录历史
cat /var/log/dpkg.log | tail            # 软件包操作历史
/var/log/boot.log                       # 启动日志
/var/log/cron                           # 定时任务日志
/var/log/dmesg                          # 内核启动日志
```

日志轮转配置：`/etc/logrotate.conf`、`/etc/logrotate.d/*`。

```bash
logrotate -d /etc/logrotate.conf    # 干跑测试
logrotate -f /etc/logrotate.conf    # 强制执行
```

---

## 九、磁盘与文件系统

### 9.1 容量查看

```bash
df -h                  # 各分区使用情况
df -hT                 # 带文件系统类型
df -i                  # inode 使用情况（小文件过多会先耗尽 inode）
df -h /var             # 指定路径所在分区
du -sh dir             # 目录总大小
du -sh *               # 当前目录各项大小
du -sh * | sort -h     # 按大小排序
du -h --max-depth=1 /var | sort -h    # 只看一层子目录
du -ah dir | sort -h | tail -20       # 目录内最大的 20 个文件
ncdu /                 # 交互式磁盘占用分析（需安装）
```

### 9.2 查找大文件与清理

```bash
find / -xdev -type f -size +500M 2>/dev/null -exec ls -lh {} \; | sort -k5 -h
find /var/log -type f -name "*.gz" -mtime +30 -delete
du -sh /var/* | sort -h
lsof +L1                            # 查找已删除但仍被占用的文件（空间没释放）
truncate -s 0 file.log              # 清空文件但保留 inode（服务还在写日志时用）
: > file.log                        # 同上
> file.log                          # 同上
journalctl --vacuum-size=200M       # 清 journal 日志
docker system prune -a              # 清理 Docker 占用
```

### 9.3 分区与格式化

```bash
lsblk                       # 查看块设备与分区
lsblk -f                    # 带文件系统类型与 UUID
blkid                       # 设备 UUID 与类型
fdisk -l                    # 列出所有磁盘分区表
fdisk /dev/sdb              # MBR 分区交互（n 新建、d 删除、w 写入、q 退出）
gdisk /dev/sdb              # GPT 分区
parted /dev/sdb print       # 查看分区
parted -a optimal /dev/sdb mklabel gpt
parted -a optimal /dev/sdb mkpart primary ext4 0% 100%
partprobe /dev/sdb          # 让内核重读分区表

mkfs.ext4 /dev/sdb1                 # 格式化为 ext4
mkfs.xfs /dev/sdb1                  # XFS
mkfs.vfat -F32 /dev/sdb1            # FAT32
mkswap /dev/sdb2                    # 创建 swap
swapon /dev/sdb2                    # 启用 swap
swapon --show                       # 查看 swap
swapoff /dev/sdb2                   # 关闭 swap
```

### 9.4 挂载

```bash
mount                                  # 查看已挂载
mount | column -t
mount /dev/sdb1 /mnt/data              # 挂载
mount -o ro /dev/sdb1 /mnt             # 只读挂载
mount -o remount,rw /                   # 根分区改读写
mount --bind /src /dst                 # 目录绑定挂载
mount -t nfs 10.0.0.5:/share /mnt/nfs  # NFS
mount -o loop image.iso /mnt           # 挂载镜像
umount /mnt/data                       # 卸载
umount -l /mnt/data                    # 懒卸载（设备忙时）
fuser -mv /mnt/data                    # 查看谁占用挂载点
lsof +D /mnt/data                      # 同上
findmnt /mnt/data                      # 查询挂载信息
findmnt -T /var -o TARGET,SOURCE,FSTYPE,OPTIONS
```

`/etc/fstab` 格式（开机自动挂载）：

```text
UUID=xxxx-xxxx  /data  ext4  defaults,noatime  0  2
# <设备>        <挂载点> <类型> <选项>          <dump> <fsck顺序>
```

推荐用 UUID，用 `blkid` 获取。修改后务必用 `mount -a` 验证，否则可能开不了机。

### 9.5 文件系统检查与修复

```bash
fsck /dev/sdb1              # 检查修复（必须先卸载！）
fsck -y /dev/sdb1           # 自动回答 yes
e2fsck -f /dev/sdb1         # ext 系列强制检查
xfs_repair /dev/sdb1        # XFS 修复
tune2fs -l /dev/sdb1        # ext 文件系统参数
tune2fs -c 30 /dev/sdb1     # 每挂载 30 次检查
dumpe2fs -h /dev/sdb1       # 超级块信息
resize2fs /dev/sdb1         # 调整 ext4 大小（扩容前先扩分区）
xfs_growfs /data            # XFS 扩容（只能扩）
```

### 9.6 LVM 逻辑卷

```bash
pvs; vgs; lvs                        # 查看 PV/VG/LV 概览
pvcreate /dev/sdb1                   # 创建物理卷
vgcreate vgdata /dev/sdb1            # 创建卷组
lvcreate -L 50G -n lvdata vgdata     # 创建 50G 逻辑卷
lvcreate -l 100%FREE -n lvdata vgdata   # 用尽剩余空间
lvextend -L +10G /dev/vgdata/lvdata  # 扩容 10G
lvextend -l +100%FREE /dev/vgdata/lvdata
resize2fs /dev/vgdata/lvdata         # ext4 扩容文件系统
xfs_growfs /mountpoint               # XFS 扩容
lvreduce -L -10G /dev/vgdata/lvdata  # 缩容（ext4 需先 resize2fs 缩小，危险）
lvremove /dev/vgdata/lvdata          # 删除
vgextend vgdata /dev/sdc1            # 卷组扩容
lvdisplay / vgdisplay / pvdisplay    # 详细信息
```

### 9.7 RAID 与磁盘健康

```bash
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
cat /proc/mdstat                     # RAID 状态
mdadm --detail /dev/md0
smartctl -a /dev/sda                 # SMART 健康详情（需 smartmontools）
smartctl -H /dev/sda                 # 健康自评
smartctl -t short /dev/sda           # 开始短自检
badblocks -sv /dev/sdb               # 坏道扫描
hdparm -tT /dev/sda                  # 读写速度测试
fio -name=test -rw=randread -size=1G -filename=/tmp/fio.test   # IO 压测
```

### 9.8 dd — 镜像与压测

```bash
dd if=/dev/zero of=/tmp/test bs=1M count=1024    # 写入 1G 测试文件
dd if=/dev/sda of=/backup/sda.img bs=4M status=progress   # 磁盘镜像
dd if=image.iso of=/dev/sdb bs=4M status=progress conv=fsync   # 写 U 盘
dd if=/dev/sda of=mbr.bak bs=512 count=1         # 备份 MBR
dd if=/dev/zero of=file bs=1M count=0 seek=1024  # 创建稀疏文件
```

> ⚠️ `dd` 的 `of=` 若写错设备名会直接毁数据，执行前用 `lsblk` 反复确认。

---

## 十、压缩与归档

### 10.1 tar — 打包

```bash
tar -cvf archive.tar dir/            # 打包（不压缩）
tar -xvf archive.tar                 # 解包
tar -tvf archive.tar                 # 查看内容
tar -czvf a.tar.gz dir/              # gzip 压缩打包
tar -xzvf a.tar.gz                   # 解压 .tar.gz / .tgz
tar -cjvf a.tar.bz2 dir/             # bzip2
tar -xjvf a.tar.bz2
tar -cJvf a.tar.xz dir/              # xz（压缩率高）
tar -xJvf a.tar.xz
tar -czvf a.tar.gz -C /path dir/     # 切到 /path 再打包
tar -xzvf a.tar.gz -C /target        # 解压到指定目录
tar -xzvf a.tar.gz path/to/file      # 只解压其中一个文件
tar -czvf a.tgz --exclude='*.log' --exclude='.git' dir/
tar -xzvf a.tar.gz --strip-components=1   # 去掉第一层目录
tar -czf - dir/ | ssh host 'cat > /tmp/a.tar.gz'   # 跨机传目录
```

参数速记：`c` 创建、`x` 解压、`t` 列表、`v` 详细、`f` 指定文件名（必须紧跟文件名）、`z` gzip、`j` bzip2、`J` xz、`C` 切换目录、`p` 保留权限。

### 10.2 各类压缩工具

```bash
gzip file                 # 压缩为 file.gz（原文件消失）
gzip -k file              # 保留原文件
gzip -9 file              # 最高压缩率
gzip -d file.gz           # 解压（= gunzip）
zcat file.gz              # 不解压直接查看
zgrep pattern file.gz     # 直接搜索压缩文件
bzip2 file / bunzip2 file.bz2 / bzcat file.bz2
xz file / unxz file.xz / xzcat file.xz
xz -T0 -9 file            # 多线程最高压缩
zip -r a.zip dir/         # 创建 zip
unzip a.zip               # 解压
unzip -l a.zip            # 查看内容
unzip -d /target a.zip    # 解压到目录
unzip -O GBK a.zip        # 解决中文乱码（Windows 生成的 zip）
7z x a.7z                 # 7-Zip 解压（需安装 p7zip）
7z a a.7z dir/            # 创建
zstd file                 # 高速压缩（现代方案）
zstd -d file.zst
lz4 file                  # 极速压缩
```

### 10.3 跨机传输与同步

```bash
scp file user@host:/path/              # 上传
scp user@host:/path/file .             # 下载
scp -r dir user@host:/path/            # 递归
scp -P 2222 file user@host:/path/      # 指定端口（大写 P）
scp -i key.pem file user@host:/path/   # 指定密钥
rsync -avz src/ user@host:/dst/        # 增量同步（推荐）
rsync -avz --delete src/ user@host:/dst/   # 镜像同步，删除目标多余文件
rsync -avz --exclude='.git' src/ dst/
rsync -avz --progress -e "ssh -p 2222" src/ host:/dst/
rsync -avzn src/ dst/                  # 干跑（-n）预览会做什么
rsync -avz --bwlimit=5000 src/ dst/    # 限速 5MB/s
rsync -aHAX --numeric-ids src/ dst/    # 保留硬链接/ACL/xattr（完整备份）
```

注意 `src/` 结尾的斜杠：带斜杠表示同步目录**内容**，不带表示同步目录**本身**。

---

## 十一、网络命令

### 11.1 网卡与地址

```bash
ip a                        # 查看所有网卡与 IP（替代 ifconfig）
ip -4 a                     # 只看 IPv4
ip a show eth0              # 指定网卡
ip link set eth0 up/down    # 启用/禁用网卡
ip addr add 192.168.1.10/24 dev eth0    # 临时加 IP
ip addr del 192.168.1.10/24 dev eth0
ip -s link                  # 网卡收发统计
ip -br a                    # 简洁输出
ifconfig                    # 老命令（net-tools，可能未安装）
ifconfig eth0 192.168.1.10 netmask 255.255.255.0
ethtool eth0                # 网卡速率/双工/驱动
ethtool -S eth0             # 网卡详细计数
nmcli device status         # NetworkManager 状态
nmcli con show              # 连接列表
nmcli con up "Wired connection 1"
nmtui                       # NetworkManager 文本界面
hostname -I                 # 本机所有 IP
```

### 11.2 路由

```bash
ip route                    # 路由表
ip route show default        # 默认网关
ip route add 10.0.0.0/8 via 192.168.1.1
ip route del 10.0.0.0/8
ip route add default via 192.168.1.1 dev eth0
route -n                    # 老命令
traceroute 8.8.8.8          # 路由追踪（需安装）
tracepath 8.8.8.8           # 无需 root 的追踪
mtr 8.8.8.8                 # 持续追踪 + 丢包统计（排障最好用）
mtr -r -c 100 8.8.8.8       # 报告模式，100 次
```

### 11.3 连通性测试与 DNS

```bash
ping -c 4 8.8.8.8              # 发 4 个包
ping -i 0.2 -c 100 host        # 快速 100 包（测丢包）
ping -s 1472 -M do host        # 测 MTU
ping6 ::1                      # IPv6
fping -a -g 192.168.1.0/24     # 批量扫描存活主机
arping -I eth0 192.168.1.1     # 二层探测
nslookup example.com           # DNS 查询
nslookup example.com 8.8.8.8   # 指定 DNS 服务器
dig example.com                # 详细 DNS 查询
dig +short example.com         # 只要结果
dig @8.8.8.8 example.com MX    # 查 MX 记录
dig -x 8.8.8.8                 # 反向解析
dig +trace example.com         # 从根域追踪解析过程
host example.com               # 简洁查询
getent hosts example.com       # 走系统 NSS 解析（含 /etc/hosts）
cat /etc/resolv.conf           # DNS 配置
resolvectl status              # systemd-resolved 状态
```

### 11.4 端口与连接

```bash
ss -tulnp                     # 查看监听端口（-t TCP -u UDP -l 监听 -n 数字 -p 进程）
ss -tan                       # 所有 TCP 连接
ss -tan state established     # 已建立连接
ss -s                         # 连接状态汇总
ss -tnp dst 10.0.0.5          # 到某地址的连接
netstat -tulnp                # 老命令（net-tools）
lsof -i :8080                 # 谁占用了 8080
lsof -i -P -n | grep LISTEN   # 所有监听
fuser -n tcp 8080             # 查占用端口的 PID
nc -zv host 80                # 测试端口是否可达
nc -zv host 20-30             # 测试端口范围
nc -l 9000                    # 监听 9000（简易服务端）
nc -u host 53                 # UDP
telnet host 80                # 测试端口（也可手工发 HTTP）
curl -v telnet://host:80      # 现代替代
```

### 11.5 下载与 HTTP 调试

```bash
curl https://example.com                     # GET 输出到屏幕
curl -o file.html https://example.com        # 保存
curl -O https://example.com/a.tar.gz         # 用远端文件名保存
curl -L https://example.com                  # 跟随重定向
curl -I https://example.com                  # 只看响应头
curl -i https://example.com                  # 头 + 体
curl -s -o /dev/null -w "%{http_code}\n" url # 只要状态码
curl -X POST -d 'a=1&b=2' https://api/       # 表单 POST
curl -X POST -H 'Content-Type: application/json' -d '{"k":"v"}' https://api/
curl -u user:pass https://api/               # Basic 认证
curl -H "Authorization: Bearer $TOKEN" https://api/
curl -b cookies.txt -c cookies.txt url       # 读/写 cookie
curl -k https://self-signed/                 # 忽略证书校验
curl --cacert ca.pem https://host/           # 指定 CA
curl -x http://proxy:8080 https://x/         # 走代理
curl --resolve api.local:443:10.0.0.5 https://api.local/   # 本地解析
curl -w "\n%{http_code} %{time_total}s\n" -o /dev/null -s url
curl --limit-rate 200k -O https://x/big.iso  # 限速
curl -T file ftp://host/                     # 上传
curl -C - -O https://x/big.iso               # 断点续传
curl --retry 3 --retry-delay 2 url           # 失败重试
curl -fsSL https://get.docker.com | sh      # 常见安装写法
wget https://example.com/file.iso            # 下载
wget -c https://example.com/big.iso          # 断点续传
wget -r -np -k https://site/docs/            # 递归镜像
wget --limit-rate=1m url
wget -b url                                  # 后台下载
wget -O name url                             # 指定文件名
wget --spider url                            # 只检查链接是否存在
aria2c -x16 -s16 url                         # 多线程下载（需安装）
```

### 11.6 抓包与流量分析

```bash
tcpdump -i eth0                       # 抓包（需 root）
tcpdump -i any -nn                    # 所有网卡，不解析名称
tcpdump -i eth0 port 80               # 指定端口
tcpdump -i eth0 host 10.0.0.5         # 指定主机
tcpdump -i eth0 src 10.0.0.5 and dst port 443
tcpdump -i eth0 -c 100 -w cap.pcap    # 抓 100 个包存文件
tcpdump -r cap.pcap                   # 读包文件
tcpdump -i eth0 -A port 80            # ASCII 显示内容
tcpdump -i eth0 -X port 80            # 十六进制 + ASCII
tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'   # 只看 SYN
tshark -i eth0 -Y "http.request"      # Wireshark 命令行
iftop -i eth0                         # 实时连接流量（需安装）
nethogs                               # 按进程看流量（需安装）
iptraf-ng                             # 交互式流量统计
nload                                 # 网卡速率实时图
bmon                                  # 带宽监控
```

### 11.7 防火墙

```bash
# iptables（传统）
iptables -L -n -v                     # 查看规则
iptables -L -n --line-numbers
iptables -A INPUT -p tcp --dport 80 -j ACCEPT      # 允许 80
iptables -A INPUT -s 10.0.0.5 -j DROP              # 拒绝来源
iptables -D INPUT 3                   # 删除第 3 条
iptables -F                           # 清空规则（注意默认策略）
iptables -P INPUT DROP                # 默认拒绝（危险，先确保放行 SSH）
iptables-save > rules.v4              # 备份
iptables-restore < rules.v4           # 恢复
# 持久化：Debian 系 iptables-persistent；RHEL 系 service iptables save 或 firewalld

# nftables（现代默认）
nft list ruleset
nft add rule inet filter input tcp dport 80 accept

# firewalld（RHEL 系）
firewall-cmd --state
firewall-cmd --list-all
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --add-service=https --permanent
firewall-cmd --reload
firewall-cmd --zone=public --add-source=10.0.0.0/24

# ufw（Ubuntu 简化版）
ufw status verbose
ufw allow 22/tcp
ufw allow from 10.0.0.0/24 to any port 3306
ufw deny 23
ufw enable
ufw delete allow 22/tcp
```

### 11.8 SSH 与远程访问

```bash
ssh user@host                     # 登录
ssh -p 2222 user@host             # 指定端口
ssh -i ~/.ssh/id_rsa user@host    # 指定私钥
ssh user@host "df -h"            # 执行远程命令
ssh -L 8080:localhost:80 user@host    # 本地端口转发：本机8080 → 远端80
ssh -R 9000:localhost:3000 user@host  # 远程转发：远端9000 → 本机3000
ssh -D 1080 user@host             # 动态 SOCKS5 代理
ssh -N -f -L 3306:db.internal:3306 jump   # 后台静默隧道
ssh -J jumpuser@jump host         # 跳板机
ssh -o StrictHostKeyChecking=no host     # 跳过指纹确认（自动化）
ssh -o ConnectTimeout=5 host      # 连接超时
ssh -v user@host                  # 调试模式（-vvv 更详细）
ssh-keygen -t ed25519 -C "me@mail"    # 生成密钥（推荐 ed25519）
ssh-keygen -t rsa -b 4096         # RSA 4096
ssh-keygen -R host                # 删除旧指纹（重装机器后）
ssh-copy-id user@host             # 上传公钥免密登录
ssh-add ~/.ssh/id_ed25519         # 加入 agent
ssh-agent bash                    # 启动 agent
ssh-keyscan host >> ~/.ssh/known_hosts   # 预置主机指纹

# 服务与配置文件
systemctl status sshd / ssh
# /etc/ssh/sshd_config 常用项：
# Port 2222
# PermitRootLogin no
# PasswordAuthentication no
# PubkeyAuthentication yes
# AllowUsers deploy

# 客户端配置 ~/.ssh/config
# Host prod
#   HostName 10.0.0.5
#   User deploy
#   Port 2222
#   IdentityFile ~/.ssh/id_ed25519
#   ProxyJump jump
ssh prod                          # 配置好之后直接这样连
```

### 11.9 文件共享与远程挂载

```bash
# NFS
showmount -e server               # 查看服务端导出
mount -t nfs server:/share /mnt
# /etc/exports (服务端)： /data 10.0.0.0/24(rw,sync,no_subtree_check)

# SMB / CIFS
smbclient -L //server -U user     # 列出共享
mount -t cifs //server/share /mnt -o username=user,vers=3.0

# SSHFS
sshfs user@host:/remote/path /mnt/local
fusermount -u /mnt/local          # 卸载
```

### 11.10 网络测试工具

```bash
iperf3 -s                          # 服务端
iperf3 -c server                   # 客户端测带宽
iperf3 -c server -u -b 100M        # UDP 100M
speedtest-cli                      # 测外网带宽
ab -n 1000 -c 100 http://host/     # ApacheBench 压测
wrk -t4 -c100 -d30s http://host/   # 现代压测工具
hey -n 1000 -c 100 http://host/    # 另一款压测
curl -o /dev/null -s -w '%{time_total}\n' url   # 单次响应耗时
```

---

## 十二、软件包管理

### 12.1 Debian / Ubuntu（apt、dpkg）

```bash
apt update                     # 更新软件源索引
apt upgrade                    # 升级已装软件
apt full-upgrade               # 升级并处理依赖变化
apt install nginx              # 安装
apt install -y nginx vim curl  # 批量非交互安装
apt remove nginx               # 卸载（保留配置）
apt purge nginx                # 卸载并删配置
apt autoremove                 # 清理无用依赖
apt clean                      # 清理下载的 deb 缓存
apt search keyword             # 搜索
apt show nginx                 # 查看包详情
apt list --installed           # 已安装列表
apt list --upgradable          # 可升级列表
apt-cache policy nginx         # 版本与来源
apt-mark hold nginx            # 锁定版本（禁止升级）
apt-mark unhold nginx
add-apt-repository ppa:xxx/yyy # 添加 PPA
apt-get -f install             # 修复依赖问题
dpkg -i pkg.deb                # 安装本地 deb
dpkg -l | grep nginx           # 查询已安装包
dpkg -L nginx                  # 包安装了哪些文件
dpkg -S /usr/sbin/nginx        # 文件属于哪个包
dpkg -r nginx                  # 卸载
dpkg --configure -a            # 重新配置未完成的包
```

### 12.2 RHEL / CentOS / Rocky（dnf、yum、rpm）

```bash
dnf install nginx              # 安装（yum 为旧别名）
dnf remove nginx               # 卸载
dnf update                     # 升级全部
dnf update nginx               # 升级指定包
dnf search keyword
dnf info nginx
dnf list installed
dnf provides "*/bin/nginx"     # 哪个包提供该文件
dnf clean all                  # 清理缓存
dnf group install "Development Tools"
dnf module list                # 模块流（RHEL8+）
dnf config-manager --add-repo url
rpm -ivh pkg.rpm               # 安装 rpm
rpm -Uvh pkg.rpm               # 升级安装
rpm -e nginx                   # 卸载
rpm -qa                        # 所有已安装包
rpm -qa | grep nginx
rpm -ql nginx                  # 包内文件列表
rpm -qf /usr/sbin/nginx        # 文件属于哪个包
rpm -qi nginx                  # 包信息
rpm -V nginx                   # 校验文件是否被改
rpm --import RPM-GPG-KEY       # 导入 GPG key
```

### 12.3 Arch / openSUSE / Alpine

```bash
# Arch
pacman -Syu                    # 同步并升级全系统
pacman -S nginx                # 安装
pacman -Rs nginx               # 卸载含依赖
pacman -Ss keyword             # 搜索
pacman -Qs keyword             # 搜索已安装
pacman -Ql nginx               # 包内文件
pacman -Qo /usr/bin/nginx      # 文件归属
pacman -Sc                     # 清理缓存

# openSUSE
zypper refresh
zypper install nginx
zypper remove nginx
zypper search keyword
zypper up

# Alpine
apk add nginx
apk del nginx
apk update && apk upgrade
apk search keyword
apk info -L nginx
```

### 12.4 源码编译安装（通用三步）

```bash
./configure --prefix=/usr/local/app     # 生成 Makefile
make -j$(nproc)                         # 编译
sudo make install                       # 安装
sudo make uninstall                     # 卸载（若支持）
```

### 12.5 通用安装方式

```bash
# 二进制 + 软链
wget https://x/tool.tar.gz && tar -xzf tool.tar.gz -C /opt
ln -s /opt/tool/bin/tool /usr/local/bin/tool

# 本地 deb 自动装依赖
apt install ./pkg.deb
# 本地 rpm 自动装依赖
dnf install ./pkg.rpm

# 通用包管理器
snap install code --classic
flatpak install flathub com.spotify.Client
brew install jq                 # Linuxbrew
```

---

## 十三、服务与 systemd

### 13.1 systemctl 基础

```bash
systemctl start nginx           # 启动
systemctl stop nginx            # 停止
systemctl restart nginx         # 重启
systemctl reload nginx          # 重载配置（不中断服务）
systemctl reload-or-restart nginx
systemctl status nginx          # 状态（含最近日志）
systemctl enable nginx          # 开机自启
systemctl disable nginx         # 取消自启
systemctl enable --now nginx    # 设为自启并立即启动
systemctl is-active nginx       # 是否运行中
systemctl is-enabled nginx      # 是否自启
systemctl is-failed nginx
systemctl mask nginx            # 彻底屏蔽（连依赖也无法拉起）
systemctl unmask nginx
systemctl list-units --type=service              # 所有服务
systemctl list-units --type=service --state=running
systemctl list-unit-files --state=enabled
systemctl list-units --failed                    # 失败单元
systemctl daemon-reload         # 修改 unit 文件后重载
systemctl reset-failed nginx    # 清除失败状态
systemctl cat nginx             # 查看 unit 文件内容
systemctl edit nginx            # 创建覆盖片段（推荐改配置方式）
systemctl show nginx -p MainPID  # 查看某属性
systemctl list-dependencies nginx
journalctl -u nginx -f          # 跟踪服务日志
```

### 13.2 目标（target）与电源

```bash
systemctl get-default                        # 当前默认目标
systemctl set-default multi-user.target      # 默认命令行模式
systemctl set-default graphical.target       # 默认图形模式
systemctl isolate multi-user.target          # 立即切换
systemctl list-units --type=target
systemctl rescue / emergency                 # 救援 / 紧急模式
```

### 13.3 自定义服务单元

`/etc/systemd/system/myapp.service`：

```ini
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=appuser
Group=appuser
WorkingDirectory=/opt/myapp
Environment="ENV=prod"
EnvironmentFile=-/etc/myapp/env
ExecStart=/opt/myapp/bin/myapp --config /etc/myapp/app.conf
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
StandardOutput=journal
StandardError=journal
# 安全加固
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ReadWritePaths=/var/lib/myapp

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
```

`Restart` 可选值：`no`、`on-success`、`on-failure`、`on-abnormal`、`always`。`Type` 常见值：`simple`、`forking`、`oneshot`、`notify`、`dbus`。

### 13.4 定时器（systemd timer）

`/etc/systemd/system/backup.timer`：

```ini
[Unit]
Description=Daily backup

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
systemctl list-timers --all
systemctl enable --now backup.timer
systemd-analyze calendar "Mon *-*-* 04:00:00"   # 验证时间表达式
```

### 13.5 传统 SysV 与开机管理

```bash
service nginx start / stop / restart / status    # SysV 兼容
/etc/init.d/nginx restart
chkconfig --list                                 # 老式开机自启管理
chkconfig nginx on
update-rc.d nginx defaults                       # Debian 系
systemd-analyze blame                            # 开机各服务耗时
systemd-analyze critical-chain                   # 关键启动链
systemd-analyze plot > boot.svg                  # 启动时序图
```

---

## 十四、Shell 与命令行技巧

### 14.1 Shell 家族与配置文件

```bash
cat /etc/shells            # 系统可用 shell
echo $SHELL                # 当前 shell
chsh -s /bin/zsh           # 修改默认 shell
bash                       # 启动子 shell
exec bash                  # 用 bash 替换当前 shell
sh -x script.sh            # 调试模式，打印执行的每条命令
bash -n script.sh          # 只做语法检查
```

加载顺序（bash 登录 shell）：`/etc/profile` → `~/.bash_profile` → `~/.bashrc`（通常被前者 source）。非登录交互式 shell 只读 `~/.bashrc`。

| 文件 | 用途 |
| --- | --- |
| `/etc/profile` | 全局环境变量，登录时加载 |
| `/etc/bash.bashrc`、`/etc/bashrc` | 全局交互式配置 |
| `~/.bash_profile` | 用户登录环境 |
| `~/.bashrc` | 用户交互式配置（别名、函数、提示符） |
| `~/.bash_logout` | 退出登录时执行 |
| `~/.profile` | 通用登录配置（Debian 系常用） |
| `~/.zshrc` | zsh 配置 |

```bash
source ~/.bashrc           # 立即生效（等价 . ~/.bashrc）
```

### 14.2 变量与环境

```bash
NAME=value                 # 赋值（等号两侧不能有空格）
export NAME=value          # 导出为环境变量
echo $NAME / echo ${NAME}  # 取值
echo "${NAME}_suffix"      # 明确变量边界（推荐加引号）
unset NAME                 # 删除变量
env                        # 所有环境变量
printenv PATH              # 查看单个
export PATH="$PATH:/opt/bin"
echo $?                    # 上条命令退出码（0 成功）
echo $$                    # 当前 shell PID
echo $!                    # 最后一个后台进程 PID
echo $# / $@ / $0 / $1     # 参数个数 / 全部参数 / 脚本名 / 第 1 个参数
read -p "输入：" var       # 读取输入
read -s -p "密码：" pw     # 隐藏输入
declare -i n=10            # 整数变量
declare -r CONST=1         # 只读
local x=1                  # 函数内局部变量
```

常用内置变量：

| 变量 | 含义 |
| --- | --- |
| `PATH` | 命令搜索路径 |
| `HOME` | 家目录 |
| `USER` / `LOGNAME` | 当前用户 |
| `PWD` / `OLDPWD` | 当前/上次目录 |
| `LANG` / `LC_ALL` | 语言与编码 |
| `PS1` | 主提示符 |
| `IFS` | 字段分隔符（默认空格/Tab/换行） |
| `EDITOR` | 默认编辑器 |
| `HISTSIZE` / `HISTFILE` | 历史条数与文件 |

### 14.3 别名与函数

```bash
alias ll='ls -alhF'                       # 定义别名
alias gs='git status'
unalias ll                                # 取消
alias                                     # 查看所有别名
\ls                                       # 绕过别名执行原命令
command ls                                # 同上

myfunc() {
  local msg="$1"
  echo "hello $msg"
}
myfunc world
declare -f myfunc                         # 查看函数定义
```

持久化：写进 `~/.bashrc`。

### 14.4 提示符 PS1

```bash
export PS1='\u@\h:\w\$ '        # 用户@主机:路径$
export PS1='[\t] \u@\h \W \$ '  # 带时间
```

| 转义 | 含义 |
| --- | --- |
| `\u` | 用户名 |
| `\h` / `\H` | 短/完整主机名 |
| `\w` / `\W` | 完整/当前目录名 |
| `\t` / `\d` | 时间 / 日期 |
| `\$` | 普通用户 `$`，root 为 `#` |
| `\n` | 换行 |
| `\[ \]` | 包裹非打印字符（颜色码） |

### 14.5 快捷键与技巧

| 快捷键 | 作用 |
| --- | --- |
| `Tab` | 补全（连按两次列出候选） |
| `Ctrl+A` / `Ctrl+E` | 行首 / 行尾 |
| `Ctrl+U` / `Ctrl+K` | 删除光标前 / 后全部 |
| `Ctrl+W` | 删除光标前一个单词 |
| `Ctrl+Y` | 粘贴刚删除的内容 |
| `Ctrl+L` | 清屏（同 clear） |
| `Ctrl+R` | 搜索历史 |
| `Ctrl+C` | 中断当前命令 |
| `Ctrl+Z` | 挂起当前命令 |
| `Ctrl+D` | 结束输入 / 退出 shell |
| `Alt+.` | 粘贴上条命令的最后一个参数 |
| `!!:gs/a/b/` | 上条命令中 a 替换为 b 再执行 |
| `cd -` | 回到上一个目录 |
| `^old^new^` | 上条命令替换后重跑 |

```bash
# 花括号与命令替换
echo "今天是 $(date +%F)"       # $() 命令替换（推荐）
echo "今天是 `date +%F`"        # 反引号（老式，不推荐）
echo $(( 3 * 7 ))               # 算术运算
echo $[3*7]                     # 老式算术
(( i++ ))                       # 算术求值
for i in {1..3}; do echo $i; done

# 历史扩展
sudo !!
!$                              # 上条命令最后一个参数
!*                              # 上条命令所有参数

# 常用组合
grep -rn "TODO" . | head
ps aux | grep -v grep | grep nginx
find . -name "*.py" | xargs wc -l | tail -1
tar -czf - dir/ | ssh host 'tar -xzf - -C /dst'
for h in host1 host2 host3; do echo "== $h =="; ssh $h uptime; done
```

### 14.6 终端复用（screen / tmux）

```bash
screen -S work              # 新建会话
screen -ls                  # 列出会话
screen -r work              # 恢复会话
screen -d -r work           # 强制接管
# Ctrl+A D 脱离，Ctrl+A C 新窗口，Ctrl+A N 下一窗口

tmux                        # 新会话
tmux new -s work            # 命名会话
tmux ls                     # 列出
tmux attach -t work         # 接入
tmux attach -d -t work      # 接管
tmux kill-session -t work
# Ctrl+B D 脱离；Ctrl+B C 新窗口；Ctrl+B N/P 切窗口；Ctrl+B % 垂直分屏；Ctrl+B " 水平分屏
```

SSH 断线不丢任务的三种做法：`tmux`、`screen`、`nohup ... &`。

---

## 十五、Shell 脚本编程

### 15.1 脚本骨架

```bash
#!/usr/bin/env bash
# 用法：./deploy.sh <环境>
set -euo pipefail        # 遇错退出 / 用未定义变量报错 / 管道任一失败即失败
IFS=$'\n\t'              # 更安全的字段分隔

readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
log() { printf '[%s] %s\n' "$(date +'%F %T')" "$*"; }

usage() {
  cat <<'EOF'
用法: deploy.sh [-e env] [-n]
  -e  环境 (dev|prod)
  -n  干跑
EOF
  exit 1
}

while getopts ":e:nh" opt; do
  case "$opt" in
    e) ENV="$OPTARG" ;;
    n) DRY_RUN=1 ;;
    h|*) usage ;;
  esac
done
shift $((OPTIND - 1))
```

### 15.2 条件判断

```bash
if [[ -f "$file" ]]; then
  echo "文件存在"
elif [[ -d "$file" ]]; then
  echo "是目录"
else
  echo "不存在"
fi

# 测试运算符
# 文件：-e 存在 -f 普通文件 -d 目录 -L 链接 -r 可读 -w 可写 -x 可执行
#       -s 非空 -nt 更新 -ot 更旧
# 字符串：-z 空 -n 非空 = 相等 != 不等 < >
# 数字：-eq -ne -gt -ge -lt -le
# 逻辑：&& || !

[[ -f a && -f b ]] && echo both
[ "$a" -gt 10 ] || exit 1
test -d /tmp && echo yes
```

`[ ]` 与 `[[ ]]` 区别：`[[ ]]` 是 bash 关键字，支持 `&&`、`||`、正则 `=~`，不做分词，更安全，推荐使用。

### 15.3 循环

```bash
for i in 1 2 3; do echo $i; done
for i in {1..5}; do echo $i; done
for ((i=0; i<10; i++)); do echo $i; done
for f in *.log; do gzip "$f"; done
for f in $(find . -name "*.tmp"); do rm -f "$f"; done

while read -r line; do echo "$line"; done < file.txt
while IFS=: read -r u _ uid _; do echo "$u $uid"; done < /etc/passwd
while true; do date; sleep 5; done

until ping -c1 host &>/dev/null; do sleep 1; done

# 循环控制
break        # 跳出循环
continue     # 进入下一轮
break 2      # 跳出两层
```

### 15.4 案例分支与函数

```bash
case "$1" in
  start)   start_service ;;
  stop)    stop_service ;;
  restart) stop_service; start_service ;;
  *)       echo "用法: $0 {start|stop|restart}"; exit 1 ;;
esac

# 函数
backup() {
  local src="$1" dst="$2"
  [[ -d "$src" ]] || { echo "源不存在: $src" >&2; return 1; }
  tar -czf "$dst/backup-$(date +%F).tar.gz" -C "$src" . && echo "完成"
}
backup /var/www /backup
```

### 15.5 数组与字符串

```bash
arr=(a b c)
echo "${arr[0]}"          # 第一个
echo "${arr[@]}"          # 全部
echo "${#arr[@]}"         # 长度
arr+=(d)                  # 追加
unset 'arr[1]'            # 删除元素
for x in "${arr[@]}"; do echo "$x"; done

declare -A map            # 关联数组
map[key]=value
echo "${map[key]}"
for k in "${!map[@]}"; do echo "$k=${map[$k]}"; done

s="hello world"
echo "${#s}"              # 长度 11
echo "${s:0:5}"           # 截取 hello
echo "${s/world/linux}"   # 替换第一个
echo "${s//o/0}"          # 替换全部
echo "${s^^}"             # 转大写
echo "${s,,}"             # 转小写
echo "${s#hello }"        # 去掉前缀（最短匹配）
echo "${s##* }"           # 去掉最长前缀
echo "${s%world}"         # 去掉后缀（最短匹配）
echo "${s%% *}"           # 去掉最长后缀
echo "${var:-默认值}"      # 未设置或空时用默认值
echo "${var:=默认值}"      # 并赋值
echo "${var:?错误信息}"    # 未设置则报错退出
f="/a/b/c.txt"; echo "${f##*/}"    # 文件名 c.txt
echo "${f%/*}"                      # 目录 /a/b
```

### 15.6 参数处理与退出码

```bash
$0           # 脚本名
$1..$9       # 位置参数
${10}        # 第 10 个及以后需加花括号
$#           # 参数个数
$@           # 所有参数（"$@" 保留各自边界，推荐）
$*           # 所有参数（"$*" 合并为一个字符串）
shift        # 丢弃第一个参数
shift 2      # 丢弃前两个
exit 0       # 正常退出
exit 1       # 异常退出
trap 'echo 清理; rm -f /tmp/$$.tmp' EXIT INT TERM    # 捕获信号做清理
```

退出码约定：`0` 成功、`1` 一般错误、`2` 用法错误、`126` 不可执行、`127` 命令未找到、`128+n` 被信号 n 杀死（`130` = Ctrl+C）。

### 15.7 并发与调试

```bash
# 简单并发
for i in {1..10}; do
  ( heavy_task "$i" ) &
done

# 限制并发数
max=4
for i in {1..100}; do
  while (( $(jobs -r | wc -l) >= max )); do sleep 0.1; done
  ( work "$i" ) &
done
wait
```

```bash
bash -n script.sh            # 语法检查
bash -x script.sh            # 打印每条命令（调试）
set -x / set +x              # 局部开启/关闭调试
shellcheck script.sh         # 静态检查（强烈推荐）
trap 'echo "第 $LINENO 行出错"' ERR
```

---

## 十六、定时任务

### 16.1 crontab

```bash
crontab -e            # 编辑当前用户任务
crontab -l            # 列出
crontab -r            # 删除全部（危险）
crontab -u tom -l     # 查看他人（root）
crontab file          # 从文件导入
```

时间字段格式：

```text
*  *  *  *  *  command
│  │  │  │  └── 星期 (0-7, 0和7都是周日)
│  │  │  └───── 月 (1-12)
│  │  └──────── 日 (1-31)
│  └─────────── 小时 (0-23)
└────────────── 分钟 (0-59)
```

```bash
0 3 * * *        /opt/backup.sh                    # 每天 3:00
*/5 * * * *      /opt/check.sh                     # 每 5 分钟
0 */2 * * *      /opt/sync.sh                      # 每 2 小时
30 4 * * 1       /opt/weekly.sh                    # 每周一 4:30
0 2 1 * *        /opt/monthly.sh                   # 每月 1 号 2:00
0 9-18 * * 1-5   /opt/report.sh                    # 工作日 9-18 点整点
@reboot          /opt/start.sh                     # 开机执行
@daily / @hourly / @weekly / @monthly / @yearly    # 快捷写法
```

生产写法（务必重定向输出并显式设置环境）：

```bash
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MAILTO=""
0 3 * * * /opt/backup.sh >> /var/log/backup.log 2>&1
```

### 16.2 管理与其他调度

```bash
systemctl status cron        # Debian 系服务名
systemctl status crond       # RHEL 系服务名
grep CRON /var/log/syslog    # 查看执行记录
journalctl -u cron -f
cat /etc/crontab             # 系统级任务（多一个用户名字段）
ls /etc/cron.d/ /etc/cron.daily/ /etc/cron.hourly/
run-parts /etc/cron.daily    # 手工运行目录内脚本
crontab -l | crontab -       # 备份还原技巧
```

`/etc/crontab` 与用户 crontab 的差异：系统级格式多一列**用户名**：

```text
0 3 * * * root /opt/backup.sh
```

一次性任务：

```bash
at 23:00                     # 交互输入，Ctrl+D 结束
echo "/opt/script.sh" | at now + 1 hour
at -l / atq                  # 列出待执行
atrm 3                       # 删除 3 号任务
batch                        # 系统负载低时执行
```

---

## 十七、内核与系统调优

### 17.1 sysctl — 运行时内核参数

```bash
sysctl -a                          # 所有参数
sysctl net.ipv4.ip_forward         # 查一个
sysctl -w net.ipv4.ip_forward=1    # 临时修改
sysctl -p                          # 重新加载 /etc/sysctl.conf
sysctl -p /etc/sysctl.d/99-tune.conf
cat /proc/sys/net/ipv4/ip_forward  # 直接读写 /proc
```

常用调优项（`/etc/sysctl.d/99-tune.conf`）：

```ini
# 网络
net.ipv4.ip_forward = 1
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_keepalive_time = 600
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216
net.ipv4.ip_local_port_range = 10240 65000

# 文件与进程
fs.file-max = 2097152
fs.inotify.max_user_watches = 524288
fs.inotify.max_user_instances = 512
kernel.pid_max = 4194304

# 内存
vm.swappiness = 10
vm.dirty_ratio = 15
vm.dirty_background_ratio = 5
vm.overcommit_memory = 1
vm.max_map_count = 262144

# 安全
kernel.sysrq = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.rp_filter = 1
```

### 17.2 内核模块

```bash
lsmod                       # 已加载模块
modprobe br_netfilter       # 加载模块（含依赖）
modprobe -r br_netfilter    # 卸载
modinfo br_netfilter        # 模块信息
insmod ./my.ko              # 加载指定文件（不处理依赖）
rmmod my                    # 卸载
depmod -a                   # 重建依赖关系
# 开机加载：/etc/modules-load.d/my.conf 里写模块名（每行一个）
# 模块参数：/etc/modprobe.d/my.conf 写 options my param=1
```

### 17.3 时间同步

```bash
timedatectl status
timedatectl set-timezone Asia/Shanghai
timedatectl set-ntp true
chronyc sources -v          # chrony 时间源
chronyc tracking            # 同步状态
chronyc makestep            # 立即校正
ntpq -p                     # 传统 ntpd
hwclock --show              # 硬件时钟
hwclock --systohc           # 系统时间写入硬件时钟
date -s "2026-01-01 12:00:00"   # 手工设置（需关闭 NTP）
```

### 17.4 交换与内存行为

```bash
swapon --show
cat /proc/sys/vm/swappiness
sysctl -w vm.swappiness=10
swapoff -a && swapon -a     # 重建 swap
# 增大 swap 文件
fallocate -l 4G /swapfile && chmod 600 /swapfile && mkswap /swapfile && swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
drop_caches:
sync && echo 3 > /proc/sys/vm/drop_caches    # 释放页缓存（谨慎，影响性能）
```

---

## 十八、容器与虚拟化

### 18.1 Docker 常用命令

```bash
# 镜像
docker images                          # 本地镜像
docker pull nginx:1.25                 # 拉取
docker build -t myapp:1.0 .            # 构建
docker build -f Dockerfile.prod -t myapp .
docker tag myapp:1.0 registry/myapp:1.0
docker push registry/myapp:1.0
docker rmi nginx:1.25                  # 删除镜像
docker image prune -a                  # 清理未使用镜像
docker history myapp:1.0               # 镜像层
docker inspect nginx                   # 详情 JSON
docker save myapp:1.0 -o myapp.tar     # 导出镜像
docker load -i myapp.tar               # 导入镜像

# 容器
docker ps                              # 运行中容器
docker ps -a                           # 全部容器（含已退出）
docker run -d --name web -p 8080:80 nginx
docker run -it --rm ubuntu:22.04 bash  # 交互式，退出即删
docker run -d -v /host/data:/data -e ENV=prod --restart=always nginx
docker run --network host nginx        # 使用宿主网络
docker start/stop/restart web
docker rm -f web                       # 强制删除
docker exec -it web bash               # 进入容器
docker exec web nginx -t               # 容器内执行命令
docker logs -f --tail 100 web          # 日志
docker logs --since 1h web
docker inspect -f '{{.State.Status}}' web
docker stats                           # 实时资源占用
docker top web                         # 容器内进程
docker cp web:/etc/nginx/nginx.conf .  # 拷文件
docker port web                        # 端口映射
docker diff web                        # 文件系统改动
docker commit web myimage:v1           # 保存容器为镜像（不推荐长期用）
docker rename web web2

# 网络与卷
docker network ls
docker network create mynet
docker network inspect mynet
docker volume ls
docker volume create myvol
docker volume inspect myvol
docker volume prune

# 编排与清理
docker compose up -d / down / logs -f / ps / restart
docker compose -f docker-compose.prod.yml up -d
docker system df                       # 空间占用
docker system prune -a --volumes       # 彻底清理（危险）
docker events                          # 实时事件
```

### 18.2 Kubernetes（kubectl）

```bash
kubectl get nodes / pods / svc / deploy -A
kubectl get pods -o wide
kubectl get pod mypod -o yaml
kubectl describe pod mypod             # 排障第一命令
kubectl logs -f mypod                  # 日志
kubectl logs mypod -c container -f
kubectl exec -it mypod -- bash
kubectl apply -f deploy.yaml
kubectl delete -f deploy.yaml
kubectl scale deploy/myapp --replicas=3
kubectl rollout status deploy/myapp
kubectl rollout undo deploy/myapp
kubectl set image deploy/myapp app=myapp:2.0
kubectl port-forward svc/myapp 8080:80
kubectl cp mypod:/path/file ./file
kubectl top pods / nodes
kubectl config get-contexts
kubectl config use-context prod
kubectl get events --sort-by=.metadata.creationTimestamp
```

### 18.3 虚拟机

```bash
# KVM / libvirt
virsh list --all
virsh start vm1 / shutdown vm1 / destroy vm1
virsh console vm1
virsh dominfo vm1
virsh snapshot-create-as vm1 snap1
virsh edit vm1
virt-install --name vm1 --memory 2048 --vcpus 2 --disk size=20 \
  --cdrom /iso/ubuntu.iso --network network=default

# 其他
qemu-img info disk.qcow2
qemu-img resize disk.qcow2 +10G
VBoxManage list vms
vagrant up / halt / ssh
```

---

## 十九、安全相关

### 19.1 认证与提权

```bash
passwd                          # 修改自己密码
sudo -l                         # 查看自己的 sudo 权限
id                              # 当前身份
sudo -i                         # root 交互 shell
visudo -c                       # 检查 sudoers 语法
last / lastb / lastlog          # 登录 / 失败 / 各用户最后登录
faillock --user tom             # 查看失败计数（pam_faillock）
faillock --user tom --reset     # 重置
pam_tally2 --user tom           # 老系统
```

### 19.2 加密与证书

```bash
# GPG
gpg --gen-key                             # 生成密钥对
gpg --list-keys / --list-secret-keys
gpg --export -a "Name" > pub.asc          # 导出公钥
gpg --import pub.asc
gpg -e -r "Name" file                     # 加密
gpg -d file.gpg                           # 解密
gpg --sign file / gpg --verify file.sig
gpg --detach-sign -a file
gpg --full-generate-key
gpg --delete-secret-keys KEYID

# SSH 密钥
ssh-keygen -t ed25519 -a 100 -C "note"
ssh-keygen -p -f ~/.ssh/id_ed25519        # 修改口令
ssh-keygen -y -f id_ed25519 > id_ed25519.pub   # 从私钥导出公钥
ssh-keygen -lf ~/.ssh/id_ed25519.pub      # 指纹

# openssl
openssl genrsa -out key.pem 4096
openssl req -new -key key.pem -out csr.pem
openssl x509 -in cert.pem -text -noout   # 查看证书
openssl x509 -in cert.pem -noout -dates  # 有效期
openssl x509 -in cert.pem -noout -subject -issuer
openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null | openssl x509 -noout -dates
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout k.pem -out c.pem  # 自签
openssl verify -CAfile ca.pem cert.pem
openssl dgst -sha256 file
openssl rand -base64 32                   # 生成随机密码
openssl passwd -6 'mypassword'            # 生成 SHA-512 密码哈希
```

### 19.3 安全审计

```bash
# 查找危险权限
find / -perm -4000 -type f 2>/dev/null              # SUID 文件
find / -perm -0002 -type f 2>/dev/null              # 任何人可写文件
find / -nouser -o -nogroup 2>/dev/null              # 无主文件
find /home -name ".ssh" -exec ls -ld {} \;          # 检查密钥目录权限

# 端口与连接
ss -tulnp | grep -v "127.0.0.1"      # 对外暴露的端口
last -a | head -20                    # 最近登录
grep -i "Failed password" /var/log/auth.log | tail

# 系统完整性
rpm -Va                               # RHEL 校验所有包文件（需谨慎解读）
debsums -c                            # Debian 校验
aide --init / aide --check            # 文件完整性数据库（需安装）
tripwire --check

# 扫描工具
nmap -sV 192.168.1.1                  # 端口与服务识别
nmap -p 1-65535 -T4 host              # 全端口
nmap -sn 192.168.1.0/24               # 存活主机
lynis audit system                    # 系统安全基线检查
chkrootkit / rkhunter --check         # 后门检测
```

### 19.4 SELinux 与 AppArmor

```bash
# SELinux（RHEL/CentOS/Fedora）
getenforce                    # 当前模式
sestatus                      # 详细状态
setenforce 0                  # 临时宽容模式（排障用）
# 永久修改：/etc/selinux/config 中 SELINUX=permissive|enforcing|disabled
ls -Z file                    # 查看安全上下文
ps -eZ                        # 进程上下文
restorecon -Rv /var/www       # 恢复默认上下文（最常见修复）
chcon -t httpd_sys_content_t file       # 临时改上下文
semanage fcontext -a -t httpd_sys_content_t "/data(/.*)?"   # 永久规则
ausearch -m avc -ts recent    # 查看拒绝日志
sealert -a /var/log/audit/audit.log
getsebool -a | grep httpd     # 布尔值
setsebool -P httpd_can_network_connect on

# AppArmor（Ubuntu/SUSE）
aa-status                     # 状态
aa-enforce /etc/apparmor.d/usr.sbin.nginx
aa-complain /etc/apparmor.d/usr.sbin.nginx
apparmor_parser -r /etc/apparmor.d/profile   # 重载
```

---

## 二十、常见故障排查套路

### 20.1 CPU 高

```bash
uptime                                   # 先看负载
top -o %CPU                              # 找占用最高的进程
ps -eo pid,ppid,cmd,%cpu --sort=-%cpu | head
pidstat -u 1 5                           # 按进程持续采样
top -H -p 1234                           # 看线程级占用
# Java：jstack 1234 > /tmp/stack.txt 分析线程栈
# 通用：perf top / perf record -p 1234 -g -- sleep 10
strace -c -p 1234                        # 统计系统调用耗时
```

### 20.2 内存高 / OOM

```bash
free -h                                  # 关注 available
ps aux --sort=-%mem | head
smem -rs rss | head                      # 更准确的内存统计（需安装）
dmesg -T | grep -i "killed process"      # 查 OOM 记录
journalctl -k | grep -i oom
cat /proc/meminfo
slabtop -s c | head                      # 内核缓存大户
pmap -x 1234 | tail -1                   # 进程总内存
```

### 20.3 磁盘满

```bash
df -h                                    # 哪个分区满
df -i                                    # inode 是否耗尽
du -h --max-depth=1 / | sort -h         # 逐层定位
lsof +L1                                 # 已删除但未释放的文件（关键）
find / -xdev -size +1G -type f 2>/dev/null -exec ls -lh {} \;
journalctl --vacuum-size=200M
docker system df                         # 容器占用
truncate -s 0 /var/log/big.log           # 应急清空
```

### 20.4 服务起不来

```bash
systemctl status mysvc -l                # 状态与最近日志
journalctl -u mysvc -n 100 --no-pager
systemctl cat mysvc                      # 检查 unit 定义
systemctl show mysvc -p ExecStart
ss -tulnp | grep 8080                    # 端口是否被占
ls -l /path/to/binary                    # 权限与存在性
sudo -u appuser /path/to/binary          # 以服务用户手工执行，看报错
dmesg -T | tail -20                      # 是否被 OOM / SELinux 拦
ausearch -m avc -ts recent               # SELinux 拒绝
```

### 20.5 网络不通（按层排查）

```bash
ip a                                     # 1. 网卡与 IP 是否存在
ip link                                 # 2. 网卡是否 UP
ip route                                 # 3. 路由与网关
ping 网关                                # 4. 二层/三层连通
ping 8.8.8.8                             # 5. 出口是否通
ping 域名                                # 6. DNS 是否正常
cat /etc/resolv.conf                     # 7. DNS 配置
ss -tulnp                                # 8. 服务是否在监听
nc -zv host port                         # 9. 端口是否可达
iptables -L -n -v / firewall-cmd --list-all   # 10. 防火墙
tcpdump -i any -nn port 8080             # 11. 抓包看包到没到
mtr -r host                              # 12. 哪一跳开始丢
```

### 20.6 进程 D 状态 / 卡住

```bash
ps -eo pid,stat,wchan:30,cmd | awk '$2 ~ /D/'
cat /proc/1234/stack 2>/dev/null         # 内核栈（需 root）
cat /proc/1234/wchan                     # 卡在哪个内核函数
iostat -xz 1                             # 是否 IO 瓶颈
dmesg -T | tail -50                      # 硬件/文件系统错误
mount | grep "sdb"                       # 是否只读重挂载（IO 错误后常见）
```

### 20.7 SSH 连不上

```bash
systemctl status sshd                    # 服务在跑吗
ss -tlnp | grep :22                      # 监听吗
iptables -L -n | grep 22                 # 防火墙
telnet host 22                           # 网络可达吗
tail -50 /var/log/auth.log               # 拒绝原因
cat ~/.ssh/known_hosts | grep host       # 指纹是否变化
ssh -vvv user@host                       # 客户端详细日志
ls -ld ~/.ssh ~/.ssh/authorized_keys     # 权限必须是 700 / 600
restorecon -Rv ~/.ssh                    # SELinux 上下文
```

---

## 二十一、命令速查索引

### 21.1 按功能索引

| 功能 | 命令 |
| --- | --- |
| 列目录 | `ls`、`tree`、`find`、`exa`/`eza` |
| 看文件 | `cat`、`less`、`head`、`tail`、`bat` |
| 编辑 | `vim`、`nano`、`emacs`、`sed` |
| 搜索内容 | `grep`、`rg`（ripgrep）、`ag` |
| 按列处理 | `awk`、`cut`、`column` |
| 排序去重 | `sort`、`uniq`、`comm` |
| 比较 | `diff`、`cmp`、`vimdiff`、`meld` |
| 权限 | `chmod`、`chown`、`umask`、`setfacl`、`chattr` |
| 用户 | `useradd`、`usermod`、`passwd`、`su`、`sudo` |
| 进程 | `ps`、`top`、`htop`、`pgrep`、`kill`、`pkill` |
| 系统信息 | `uname`、`hostnamectl`、`lscpu`、`lsblk`、`dmidecode` |
| 监控 | `vmstat`、`iostat`、`sar`、`pidstat`、`mpstat` |
| 磁盘 | `df`、`du`、`fdisk`、`mkfs`、`mount`、`fsck` |
| LVM | `pvcreate`、`vgcreate`、`lvcreate`、`lvextend` |
| 归档 | `tar`、`gzip`、`xz`、`zip`、`zstd` |
| 传输 | `scp`、`rsync`、`sftp`、`nc` |
| 网络 | `ip`、`ss`、`ping`、`dig`、`curl`、`wget` |
| 抓包 | `tcpdump`、`tshark`、`iftop`、`nethogs` |
| 防火墙 | `iptables`、`nft`、`firewall-cmd`、`ufw` |
| 包管理 | `apt`、`dpkg`、`dnf`、`rpm`、`pacman`、`apk` |
| 服务 | `systemctl`、`journalctl`、`service` |
| 定时 | `crontab`、`at`、`systemd timer` |
| 内核 | `sysctl`、`lsmod`、`modprobe`、`dmesg` |
| 容器 | `docker`、`kubectl`、`podman`、`nerdctl` |
| 安全 | `openssl`、`gpg`、`nmap`、`lynis`、`getenforce` |
| 调试 | `strace`、`ltrace`、`lsof`、`perf`、`gdb` |

### 21.2 高频组合速查

```bash
# 找占端口进程
ss -tulnp | grep :8080
lsof -i :8080

# 按大小找文件
du -ah . | sort -h | tail -20
find / -xdev -type f -size +1G 2>/dev/null

# 日志 Top 错误
grep -i error app.log | awk '{print $5}' | sort | uniq -c | sort -rn | head

# 统计某接口访问量
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -20

# 统计状态码分布
awk '{print $9}' access.log | sort | uniq -c | sort -rn

# 实时看某服务错误
journalctl -u myapp -f | grep -i --color error

# 批量改名
for f in *.txt; do mv "$f" "${f%.txt}.md"; done

# 批量替换内容
grep -rl "old" . | xargs sed -i 's/old/new/g'

# 清理 30 天前日志
find /var/log -name "*.log" -mtime +30 -delete

# 目录大小排行
du -sh */ | sort -h

# 找出重复文件（按 md5）
find . -type f -exec md5sum {} + | sort | uniq -w32 -dD

# 查看某进程的网络连接
lsof -p 1234 -i -n -P

# 对比两台机器某文件
diff <(ssh host1 cat /etc/nginx/nginx.conf) /etc/nginx/nginx.conf

# 快速共享当前目录
python3 -m http.server 8000

# 生成随机密码
tr -dc 'A-Za-z0-9!@#$%' < /dev/urandom | head -c 20; echo

# 查看命令属于哪个包
dpkg -S $(which nginx) || rpm -qf $(which nginx)

# 递归查看目录树
find . -print | sed -e 's;[^/]*/;|____;g;s;____|; |;g'
```

### 21.3 现代替代工具（可选安装）

| 传统命令 | 现代替代 | 优势 |
| --- | --- | --- |
| `ls` | `eza` / `exa` | 彩色、图标、git 状态 |
| `cat` | `bat` | 语法高亮、行号 |
| `grep` | `ripgrep (rg)` | 极快、默认递归、忽略 .gitignore |
| `find` | `fd` | 更短语法、默认忽略隐藏 |
| `sed/awk` | `sd` | 更直观的替换语法 |
| `du` | `dust` / `ncdu` | 可视化 |
| `df` | `duf` | 美观表格 |
| `ps/top` | `procs` / `btop` | 更友好 |
| `curl` | `httpie (http)` | 类自然语言 |
| `diff` | `delta` | 彩色 diff |
| `man` | `tldr` | 只给常用示例（强烈推荐） |
| `jq` | — | 处理 JSON 必备 |
| `yq` | — | 处理 YAML |
| `fzf` | — | 模糊查找，配合任何命令 |
| `watch` | — | 定时重复执行命令 |
| `tree` | — | 目录树可视化 |
| `htop` | — | 交互式进程管理 |

`tldr` 用法（比 man 更实用）：

```bash
tldr tar
tldr rsync
tldr find
```

---

## 二十二、附录

### 22.1 通配符、正则、引号对照

| 符号 | 在 Shell 中 | 在正则中 |
| --- | --- | --- |
| `*` | 任意多个字符 | 前一项重复 0+ 次 |
| `?` | 单个字符 | 前一项 0 或 1 次 |
| `.` | 字面点号 | 任意单字符 |
| `[abc]` | 字符集 | 字符集 |
| `^` | 无特殊含义 | 行首 |
| `$` | 变量引用 | 行尾 |

引号规则：

```bash
echo '$HOME'      # 单引号：完全字面量，不展开
echo "$HOME"      # 双引号：展开变量、命令替换、转义 \ $ ` "
echo \$HOME       # 反斜杠转义
echo `date`       # 反引号：命令替换
echo $(date)      # 推荐写法
```

### 22.2 常见文件权限建议

| 文件 | 建议权限 |
| --- | --- |
| 普通文件 | `644` |
| 可执行脚本 | `755` |
| 目录 | `755` |
| 私钥、`~/.ssh/id_*` | `600` |
| `~/.ssh` 目录、`authorized_keys` | `700` / `600` |
| 密码文件、`/etc/shadow` | `600` / `000`（root 专属） |
| Web 目录（属主为部署用户） | `755` 文件 `644` |
| `/tmp` | `1777`（粘滞位） |
| 日志文件 | `640` |

### 22.3 常见端口

| 端口 | 服务 |
| --- | --- |
| 20/21 | FTP |
| 22 | SSH |
| 23 | Telnet |
| 25 / 465 / 587 | SMTP / SMTPS / 提交 |
| 53 | DNS |
| 80 / 443 | HTTP / HTTPS |
| 110 / 995 | POP3 / POP3S |
| 143 / 993 | IMAP / IMAPS |
| 123 | NTP |
| 161/162 | SNMP |
| 3306 | MySQL / MariaDB |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 27017 | MongoDB |
| 9200/9300 | Elasticsearch |
| 8080 / 8443 | HTTP 备用 / HTTPS 备用 |
| 11211 | Memcached |
| 2181 | ZooKeeper |
| 5672 | RabbitMQ |
| 9092 | Kafka |
| 2375/2376 | Docker API |

### 22.4 退出码与信号速查

```text
退出码：0 成功 | 1 一般错误 | 2 用法错误 | 126 无执行权限
        127 命令未找到 | 128+n 被信号 n 终止（130 = Ctrl+C）
        137 = 128+9（被 SIGKILL，常见于 OOM）| 143 = 128+15
信号：1 HUP 2 INT 3 QUIT 9 KILL 11 SEGV 15 TERM 18 CONT 19 STOP
```

### 22.5 学习建议

1. **先练会用**：`man`、`tldr`、`--help`、`Tab` 补全、`Ctrl+R`，比背命令重要。
2. **理解管道哲学**：每个命令只做一件事，用 `|` 组合出复杂能力。
3. **记住三个万能命令**：`grep` 找内容、`find` 找文件、`awk` 处理列。
4. **排障顺序固定**：先看日志（`journalctl`/`tail`），再看资源（`top`/`df`/`free`），最后看网络（`ss`/`tcpdump`）。
5. **危险操作先 `echo`**：破坏性命令在脚本里先打印再执行，确认无误再放开。
6. **写脚本必加 `set -euo pipefail`**，并用 `shellcheck` 检查。

---

> 文档说明：本文汇总的命令覆盖 GNU/Linux 主流发行版（Debian/Ubuntu、RHEL/CentOS/Rocky、Arch、Alpine）。
> 部分命令需要额外安装（如 `htop`、`sysstat`、`tree`、`tldr`、`ripgrep`），部分参数在不同版本间略有差异，请以 `man` 手册与 `--help` 为准。

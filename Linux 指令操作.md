# Linux 常用指令速查

> 以 Ubuntu/Debian 为例，大部分命令在所有发行版通用。

---

## 目录与路径

### `pwd` — 显示当前目录的绝对路径

```bash
pwd
# /home/alice/work
```

### `ls` — 列出目录内容

```bash
ls          # 平铺显示
ls -l       # 详细信息（权限、大小、时间）
ls -a       # 包含隐藏文件（. 开头）
ls -lh      # 人类可读的文件大小（K/M/G）
ls -lt      # 按修改时间排序
```

常用组合：`ls -lah`

### `cd` — 切换目录

```bash
cd /var/log      # 进入指定目录
cd ..            # 返回上一级
cd ~             # 回到用户家目录
cd -             # 回到上一次所在的目录
```

---

## 文件操作

### `touch` — 创建空文件，或更新文件时间戳

```bash
touch note.txt
```

### `mkdir` — 创建目录

```bash
mkdir project
mkdir -p a/b/c   # 递归创建多级目录
```

### `cp` — 复制文件/目录

```bash
cp a.txt b.txt           # 复制并重命名
cp file.txt /target/     # 复制到指定目录
cp -r dir1/ dir2/        # 递归复制整个目录
```

### `mv` — 移动或重命名

```bash
mv old.txt new.txt       # 重命名
mv file.txt /target/     # 移动到指定目录
```

### `rm` — 删除

```bash
rm file.txt              # 删除文件
rm -r dir/               # 递归删除目录
rmdir dir/               # 仅删除空目录
```

> `rm -rf` 极其危险，不会确认，无法恢复。慎用。

### `cat` — 查看文件内容（一次性输出全部）

```bash
cat file.txt
cat -n file.txt          # 带行号
```

### `file` — 识别文件类型

```bash
file unknown.bin
# unknown.bin: ELF 64-bit LSB executable
```

---

## 文件搜索

### `find` — 按文件名/属性查找文件

```bash
find . -name "*.log"           # 当前目录下搜 .log 文件
find /var -size +100M          # 大于 100MB 的文件
find . -mtime -7               # 最近 7 天修改的文件
```

### `grep` — 搜索文件内容

```bash
grep "error" app.log           # 查找含 error 的行
grep -r "TODO" .               # 递归搜索当前目录
grep -i "warning" app.log      # 忽略大小写
grep -n "main" main.c          # 带行号
```

---

## 磁盘与存储

### `df` — 查看磁盘分区空间

```bash
df -h     # 人类可读格式
```

### `du` — 查看文件/目录占用空间

```bash
du -sh .          # 当前目录总大小
du -sh *          # 每个子目录的大小
```

### `sync` — 将内存缓冲区的数据写入磁盘

```bash
sync
```

---

## 系统状态

### `uname` — 系统/内核信息

```bash
uname -a     # 全部信息（内核版本、架构等）
uname -r     # 仅内核版本号
```

### `ps` — 查看进程快照

```bash
ps aux       # 所有用户的所有进程
```

### `top` — 实时进程监控

```bash
top          # 按 q 退出
```

### `man` — 查看命令帮助手册

```bash
man ls       # 按 q 退出
```

### `sudo` — 以 root 身份执行单条命令

```bash
sudo apt update
sudo su       # 切换到 root 用户
```

### `reboot` / `poweroff` — 重启 / 关机

```bash
sudo reboot
sudo poweroff
```

---

## 网络

### `ifconfig` — 查看/配置网络接口

```bash
ifconfig                    # 显示所有网卡信息
sudo ifconfig eth0 down     # 关闭网卡
sudo ifconfig eth0 up       # 启用网卡
```

> 新发行版推荐使用 `ip` 命令替代：`ip addr`、`ip link set eth0 up`

---

## 终端

### `clear` — 清屏

```bash
clear
# 等效快捷键：Ctrl + L
```

---

## 速查表

| 场景 | 命令 |
|------|------|
| 我在哪个目录 | `pwd` |
| 这目录里有什么 | `ls -lah` |
| 切到另一个目录 | `cd /path` |
| 创建文件 | `touch a.txt` |
| 创建目录 | `mkdir -p a/b/c` |
| 复制 | `cp -r source target` |
| 移动/重命名 | `mv old new` |
| 删除文件 | `rm file` |
| 删除目录 | `rm -r dir` |
| 看文件内容 | `cat file` |
| 搜文件名 | `find . -name "*.log"` |
| 搜文件内容 | `grep "keyword" file` |
| 磁盘空间 | `df -h` |
| 目录大小 | `du -sh .` |
| 进程列表 | `ps aux` |
| 实时监控 | `top` |
| 看帮助 | `man command` |
| 临时提权 | `sudo command` |

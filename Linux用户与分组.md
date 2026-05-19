# Linux 用户与分组

## 用户信息文件

| 文件 | 内容 |
|------|------|
| `/etc/passwd` | 用户列表（用户名:UID:GID:描述:家目录:shell） |
| `/etc/shadow` | 用户密码（加密存储，仅 root 可读） |
| `/etc/group` | 组列表（组名:GID:组成员） |

## 用户管理

| 命令 | 作用 | 常用选项 |
|------|------|----------|
| `useradd alice` | 创建用户 | `-m` 创建家目录，`-s /bin/bash` 指定 shell |
| `userdel alice` | 删除用户 | `-r` 同时删除家目录 |
| `usermod` | 修改用户 | `-aG sudo alice` 加入 sudo 组 |
| `passwd alice` | 设置/修改密码 | |
| `id alice` | 查看用户 UID/GID/所属组 | |
| `whoami` | 查看当前用户名 | |
| `su - alice` | 切换用户 | `-` 同时加载环境变量 |
| `who` | 查看当前登录用户 | |

## 组管理

| 命令 | 作用 | 常用选项 |
|------|------|----------|
| `groupadd dev` | 创建组 | |
| `groupdel dev` | 删除组 | |
| `usermod -aG dev alice` | 将用户加入组 | `-a` 追加，不加会覆盖 |
| `gpasswd -d alice dev` | 将用户移出组 | |
| `groups alice` | 查看用户所属组 | |

## 用户类型

| 类型 | UID 范围 | 说明 |
|------|----------|------|
| root | 0 | 超级管理员 |
| 系统用户 | 1~999 | 运行服务，不能登录 |
| 普通用户 | 1000+ | 日常使用 |

## /etc/sudoers

| 配置 | 含义 |
|------|------|
| `alice ALL=(ALL) ALL` | alice 可执行所有 sudo 命令 |
| `%dev ALL=(ALL) ALL` | dev 组成员可执行所有 sudo 命令 |
| `alice ALL=(ALL) NOPASSWD: ALL` | alice sudo 不需要密码 |

> 用 `visudo` 编辑，会语法检查后再保存，比直接改 `/etc/sudoers` 安全。

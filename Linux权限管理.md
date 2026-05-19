# Linux 权限管理

## 权限表示

```
-rwxr-xr-x  1 alice dev  4096 May 19 10:00 script.sh
 └┬┘└┬┘└┬┘
  │  │   └─ 其他用户 (o)   r-x = 读+执行
  │  └───── 所属组 (g)     r-x = 读+执行
  └──────── 所有者 (u)     rwx = 读+写+执行
```

| 类型 | 标识 | 含义 |
|------|------|------|
| `-` | 普通文件 | |
| `d` | 目录 | |
| `l` | 软链接 | |
| `r` | 读 | 文件：可查看内容 / 目录：可 `ls` |
| `w` | 写 | 文件：可修改 / 目录：可创建删除文件 |
| `x` | 执行 | 文件：可运行 / 目录：可 `cd` 进入 |

## chmod — 修改权限

### 符号模式

| 命令 | 作用 |
|------|------|
| `chmod u+x file` | 所有者加执行 |
| `chmod g-w file` | 组去写权限 |
| `chmod o+r file` | 其他加读权限 |
| `chmod a+x file` | 全部加执行 |
| `chmod u=rwx,g=rx,o= file` | 精确设置 |

### 数字模式

| 数字 | 权限 |
|------|------|
| 4 | r（读） |
| 2 | w（写） |
| 1 | x（执行） |

| 命令 | 含义 |
|------|------|
| `chmod 755 file` | rwxr-xr-x（可执行文件） |
| `chmod 644 file` | rw-r--r--（普通文件） |
| `chmod 777 file` | rwxrwxrwx（全开，危险） |
| `chmod 600 file` | rw-------（私密文件，如 SSH 私钥） |

> `chmod -R 755 dir/` 递归修改整个目录

## chown — 修改所有者

| 命令 | 作用 |
|------|------|
| `chown alice file` | 改所有者为 alice |
| `chown alice:dev file` | 改所有者和组 |
| `chown :dev file` | 仅改组 |
| `chown -R alice:dev dir/` | 递归修改 |

## 特殊权限

| 权限 | 数字 | 文件 | 目录 |
|------|------|------|------|
| SUID | 4 | 以文件所有者身份运行 | 无意义 |
| SGID | 2 | 以文件所属组身份运行 | 目录内新建文件继承目录的组 |
| Sticky | 1 | 无意义 | 只有文件所有者能删除（如 `/tmp`） |

| 命令 | 作用 |
|------|------|
| `chmod u+s file` | 设置 SUID |
| `chmod g+s dir/` | 设置 SGID |
| `chmod +t dir/` | 设置 Sticky bit |
| `chmod 1777 dir/` | Sticky + 777（/tmp 默认权限） |

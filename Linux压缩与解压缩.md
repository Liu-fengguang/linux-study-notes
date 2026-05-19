# Linux 压缩与解压缩

## tar

| 命令 | 作用 |
|------|------|
| `tar -cvf a.tar dir/` | 打包（不压缩） |
| `tar -xvf a.tar` | 解包 |
| `tar -czvf a.tar.gz dir/` | gzip 压缩打包 |
| `tar -xzvf a.tar.gz` | 解压 .tar.gz |
| `tar -cjvf a.tar.bz2 dir/` | bzip2 压缩打包 |
| `tar -xjvf a.tar.bz2` | 解压 .tar.bz2 |
| `tar -cJvf a.tar.xz dir/` | xz 压缩打包 |
| `tar -xJvf a.tar.xz` | 解压 .tar.xz |

> 参数记忆：`c` 创建，`x` 解压，`v` 显示过程，`f` 指定文件名，`z` gzip，`j` bzip2，`J` xz

## gzip / bzip2 / xz（单文件）

| 命令 | 作用 |
|------|------|
| `gzip file` | 压缩为 file.gz（原文件删除） |
| `gunzip file.gz` | 解压 |
| `bzip2 file` | 压缩为 file.bz2 |
| `bunzip2 file.bz2` | 解压 |
| `xz file` | 压缩为 file.xz |
| `unxz file.xz` | 解压 |

## zip（跨平台兼容）

| 命令 | 作用 |
|------|------|
| `zip -r a.zip dir/` | 压缩目录 |
| `unzip a.zip` | 解压 |
| `unzip -l a.zip` | 仅查看内容，不解压 |

## 对比

| 格式 | 压缩率 | 速度 |
|------|--------|------|
| gzip (.gz) | 中 | 快 |
| bzip2 (.bz2) | 高 | 慢 |
| xz (.xz) | 最高 | 最慢 |
| zip (.zip) | 中 | 快 |

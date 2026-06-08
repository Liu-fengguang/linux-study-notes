# 问题手册：M 系列芯片裸机开发踩坑记录

> MacBook M4 + i.MX6ULL 开发板裸机实战中遇到的每一个报错及解决。

---

## 第一阶段：编译与链接原理

### 1. 为什么 M4 和开发板都是 ARM，还需要交叉编译？

| 平台 | 架构 | 位宽 |
|------|------|------|
| Mac M4 / Ubuntu 虚拟机 | AArch64 | 64 位 |
| i.MX6ULL (Cortex-A7) | ARMv7 | 32 位 |

> 必须用 `arm-linux-gnueabihf-gcc` 把代码翻译成 32 位 ARM 机器码。系统自带 gcc 生成的是 64 位指令，开发板不认识。

### 2. 编译四大步

```
.c/.s ─①─→ .o ─②─→ .elf ─③─→ .bin
                   └④─→ .dis
```

| 步骤 | 做了什么 | 关键点 |
|------|----------|--------|
| ① 编译 | 每个源文件翻译成机器码 | 此时地址悬空，不可运行 |
| ② 链接 | 拼合所有 .o + 分配绝对地址 | `-Ttext 0x87800000` 指定运行地址 |
| ③ 提纯 | 剥掉 ELF 元数据 | `objcopy -O binary`，只留纯机器码 |
| ④ 反汇编 | 逆向为汇编 + 地址对照 | 崩溃时用地址反查定位代码行 |

### 3. Boot ROM 机制：先有鸡还是先有蛋？

| 问题 | 答案 |
|------|------|
| DDR 需要代码初始化才能用 | 对 |
| 初始化代码又得放 DDR 里 | 对 |
| 这不矛盾吗？ | Boot ROM 破了这个循环 |

**流程：**

```
芯片上电 → Boot ROM 运行 → 读 SD 卡第 1024 字节处
→ 读取 DCD 头部（DDR 初始化参数）→ 初始化 DDR
→ 把 bin 搬运到 0x87800000 → 跳转执行
```

> Boot ROM 是芯片出厂固化的，不需外部代码就能跑。DCD 头部就是给它看的"DDR 初始化说明书"。

---

## 第二阶段：SD 卡与文件传输

### 4. Mac 格式化旧 SD 卡报错

**问题**：磁盘工具格式化失败，SD 卡曾是树莓派系统盘（含 ext4 分区）。

| 步骤 | 操作 |
|------|------|
| ① | 磁盘工具 → 显示 → 显示所有设备 |
| ② | 选中**物理读卡器**（最顶层），不是分区 |
| ③ | 抹掉 → 格式 MS-DOS (FAT32) → 方案 主引导记录 (MBR) |

### 5. 错误码 -69772

**原因**：SD 卡物理写保护开关锁上了。

**解决**：卡槽左侧拨片往上拨（解锁），重新格式化。

### 6. 搭建 vsftpd（Ubuntu → Mac 文件传输）

```bash
sudo apt install vsftpd
sudo vim /etc/vsftpd.conf
```

| 配置 | 操作 |
|------|------|
| `local_enable=YES` | 去掉注释 `#` |
| `write_enable=YES` | 去掉注释 `#` |

```bash
sudo systemctl restart vsftpd
```

### 7. FileZilla 553 错误

| 问题 | 原因 | 解决 |
|------|------|------|
| 上传到 `/home/` 失败 | `/home` 是公共大堂，没权限 | 双击进入 `/home/tim`（你的用户目录）再上传 |

---

## 第三阶段：Mac M 系列特有"天坑"

### 8. 虚拟机找不到 SD 卡（PCIe 隔离墙）

**原理**：Mac 自带 SD 卡槽走 **PCIe 总线**，不是 USB 总线。虚拟机只能接管 USB 设备，无法直接访问 PCIe 设备。

**破局策略**：

```
Ubuntu 虚拟机 → 编译 + 加工生成 load.imx
         │
Mac 终端    → 物理烧录 dd 命令写入 SD 卡
```

> 混合双打模式：虚拟机做软件，Mac 做硬件。

### 9. imxdownload 报 Exec format error

**问题**：教程提供的 `imxdownload` 无法运行。

**原因**：教程编译的是 **x86_64** 架构，M4 的 Ubuntu 虚拟机是 **ARM64**。

```
$ file imxdownload
imxdownload: ELF 64-bit LSB executable, x86-64   ← Intel 版

$ uname -m
aarch64                                          ← 我们是 ARM64
```

**解决**：拿源码 `imxdownload.c` + `.h`，在虚拟机里现场重新编译：

```bash
gcc imxdownload.c -o imxdownload
```

> 忽略 VS Code 里 `.h` 找不到的假报错，终端编译正常就行。

### 10. /dev/null 黑洞欺骗术

**问题**：虚拟机里没有物理 SD 卡，`imxdownload` 烧录会报错退出。

**解决**：把输出指向 `/dev/null`（只进不出的数据黑洞），让它安静生成 `load.imx`：

```bash
./imxdownload led.bin /dev/null
```

> 只取副产品 `load.imx`，不需要真烧录。

### 11. Mac 终端 dd 写入 SD 卡

```bash
sudo dd if=load.imx of=/dev/rdisk4 bs=512 seek=2 conv=sync
```

| 参数 | 含义 |
|------|------|
| `rdisk4` | Raw 模式，绕过文件系统缓冲，速度极快 |
| `bs=512` | 每次读写 512 字节（一个扇区） |
| `seek=2` | **跳过前 2 个扇区（1024 字节）**，保留分区表，写到 Boot ROM 约定的位置 |
| `conv=sync` | 数据不足 512 的倍数时，用 `00` 补全尾巴 |

### 12. 终极隐蔽坑：Invalid argument

**问题**：dd 报 `Invalid argument`。

**原因**：Mac 的 `rdisk` 要求写入数据必须是 **512 字节的完美倍数**。多出来的碎片字节会导致拒绝写入。

**解决**：加上 `conv=sync`，用空白 `00` 填充尾巴碎片。最终完美写入 3584 字节。

---

## 装备建议

| 问题 | 方案 |
|------|------|
| 自带 SD 卡槽走 PCIe，虚拟机无法访问 | 买 Type-C 外接 SD 读卡器（10 元） |
| 外接读卡器走 USB 协议 | 虚拟机直接接管，实现编译+烧录一键完成 |

---

## 问题速查表

| 序号 | 症状 | 原因 | 解决 |
|------|------|------|------|
| 1 | gcc 编的 bin 开发板不跑 | M4 是 ARM64，开发板是 ARM32 | 用 `arm-linux-gnueabihf-gcc` |
| 2 | 磁盘工具格式化失败 | SD 卡有 ext4 分区 | 显示所有设备 → 抹掉顶层物理读卡器 |
| 3 | 错误 -69772 | 写保护锁 | 拨片解锁 |
| 4 | FileZilla 553 | 传到 `/home/` 没权限 | 进 `/home/tim` 再传 |
| 5 | 虚拟机看不到 SD 卡 | 自带卡槽是 PCIe 不是 USB | 混合双打 / 买外接读卡器 |
| 6 | imxdownload 报 Exec format error | 教程是 x86_64，虚拟机是 ARM64 | 拿源码重编译 |
| 7 | imxdownload 找不到 SD 卡 | 虚拟机没物理 SD 卡 | 指向 `/dev/null` 只生成 load.imx |
| 8 | dd 报 Invalid argument | 数据不是 512 整数倍 | 加 `conv=sync` |

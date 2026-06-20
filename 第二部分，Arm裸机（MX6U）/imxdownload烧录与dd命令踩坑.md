# imxdownload 烧录与 dd 命令踩坑

> imxdownload 源码编译、/dev/null 虚空生成、Mac dd 物理扇区烧录全流程。

---

## 一、imxdownload 是什么

给 `led.bin` 前面粘上 IVT + Boot Data + DCD 头部，生成 chip 可启动的 `load.imx`，并烧到 SD 卡指定扇区。

---

## 二、源码本地编译（ARM64 Ubuntu）

教程提供的 `imxdownload` 是 x86_64 版本，M4 Mac 的 Ubuntu 虚拟机是 ARM64，跑不了。

```bash
# 本地编译（不是交叉编译！imxdownload 是在电脑上跑的工具）
gcc imxdownload.c -o imxdownload
chmod +x imxdownload
```

> 本地编译用系统自带 `gcc`，不是 `arm-linux-gnueabihf-gcc`。

---

## 三、参数规则（V1.1 版本）

| 参数个数 | 格式 | 效果 |
|----------|------|------|
| 2 个 | `./imxdownload led.bin /dev/sdb` | 兼容模式，默认 512MB |
| 3 个 | `./imxdownload -512m led.bin /dev/null` | 明确指定内存大小 |

> 只给 1 个参数会触发错误提示，显示完整 Usage。

---

## 四、不插 SD 卡只生成 load.imx

用 `/dev/null` 黑洞骗过烧录步骤：

```bash
./imxdownload -512m led.bin /dev/null
# 或兼容模式（两参数）
./imxdownload led.bin /dev/null
```

工具会：
1. 读取 `led.bin`，加上 IVT 头部
2. 生成 `load.imx`（留在当前目录）
3. 尝试写入 `/dev/null`（数据丢弃，无副作用）

> 不用 `sudo`，因为没碰真实设备。

---

## 五、为什么不能直接拖拽文件到 SD 卡

| 方式 | 原理 | 芯片能读吗 |
|------|------|------------|
| 拖拽复制 | 文件系统层操作，存在 FAT 目录分配的任意位置 | ❌ |
| dd 扇区烧录 | 绕过文件系统，直接写物理扇区 | ✅ |

芯片上电 Boot ROM 不看文件系统，只会去**固定物理偏移 1024 字节**处找代码。

> 文件复制 = 放哪里由文件系统定。dd 烧录 = 精确放到指定扇区。

---

## 六、Mac dd 烧录流程

### 1. 找到 SD 卡

```bash
diskutil list
# 根据容量找到 /dev/disk4（示例）
```

> `/dev/disk0` 是 Mac 内置硬盘，看错就重装系统。

### 2. 卸载

```bash
diskutil unmountDisk /dev/disk4
```

> 不卸载文件系统就烧录会被拒绝。

### 3. 烧录

```bash
sudo dd if=load.imx of=/dev/rdisk4 bs=1024 seek=1 conv=sync
```

| 参数 | 含义 |
|------|------|
| `rdisk4` | Raw 模式，绕过文件系统缓冲，速度快 |
| `bs=1024` | 每次写 1024 字节（= 2 扇区） |
| `seek=1` | **跳过第 1 个 1024 字节**，从第 2 个 1024 字节（偏移 1024）开始写 |
| `conv=sync` | 数据不足 1024 倍数时，用 `00` 补齐尾巴 |

---

## 七、3+1 vs 3+0 数据丢失

**症状**：

```
3+1 records in
3+0 records out
Invalid argument
```

| 显示 | 含义 |
|------|------|
| `3+1 in` | 读了 3 个完整块 + 1 个不足块（尾数） |
| `3+0 out` | 只写入了 3 个完整块，**尾数被丢弃** |
| `Invalid argument` | Mac rdisk 拒绝非整数倍写入 |

尾数块正是你的 `led.bin` 核心代码。丢掉它 = SD 卡只有头部没有代码体。

**解决**：`conv=sync` 自动用 `00` 补齐尾数，满足 Mac 的整数倍强迫症。

```
4+0 records in
4+0 records out    ← 全写进去了
```

---

## 八、Resource busy 错误

**症状**：`Resource busy`

**原因**：Mac 后台进程（访达预览、Spotlight）偷偷重新挂载了 SD 卡。

**解决**：

```bash
# 关掉所有访达窗口，再卸载
diskutil unmountDisk /dev/disk4

# 还不行就强踢
diskutil unmountDisk force /dev/disk4

# 立即执行烧录（趁系统没反应过来）
sudo dd if=load.imx of=/dev/rdisk4 bs=1024 seek=1 conv=sync
```

---

## 九、完整工作流

```
Ubuntu 虚拟机（编译）
  make                         → led.bin
  ./imxdownload led.bin /dev/null → load.imx
         │
    VS Code Remote-SSH 下载 load.imx 到 Mac
         │
Mac（烧录）
  diskutil list                 → 确认 /dev/disk4
  diskutil unmountDisk /dev/disk4
  sudo dd if=load.imx of=/dev/rdisk4 bs=1024 seek=1 conv=sync
         │
  拔卡 → 插开发板 → 上电 → LED 亮
```

---

## 十、速查

| 问题 | 解决 |
|------|------|
| imxdownload 跑不了 (Exec format error) | `gcc imxdownload.c -o imxdownload` 源码重编 |
| 不想烧录只生成 load.imx | 目标填 `/dev/null` |
| 文件拖到 SD 卡不启动 | 用 dd 物理扇区写入，不能文件系统拷贝 |
| dd 报 Invalid argument | 加 `conv=sync` |
| dd 报 Resource busy | 关访达 → `unmountDisk force` → 立即 dd |

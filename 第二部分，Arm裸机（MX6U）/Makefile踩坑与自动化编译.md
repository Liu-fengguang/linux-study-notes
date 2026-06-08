# Makefile 踩坑与自动化编译

## 一、概念澄清：Makefile ≠ make

| | Makefile | make |
|------|----------|------|
| 是什么 | 文本文件，写编译规则 | 终端命令，执行工具 |
| 比喻 | 菜谱 | 厨师 |
| 操作 | 编辑保存 | 终端敲 `make` |

> 把 Makefile 的代码直接粘贴到终端里运行，Bash 不认识 `CROSS_COMPILE ?=` 这种语法，会报 `未找到命令`。

---

## 二、安装 make

```bash
sudo apt-get install make
```

---

## 三、第一个 Makefile

```makefile
CROSS_COMPILE ?= arm-linux-gnueabihf-

all: led.bin

led.bin: led.elf
	$(CROSS_COMPILE)objcopy -O binary -g -S led.elf led.bin
	$(CROSS_COMPILE)objdump -D led.elf > led.dis

led.elf: led.o
	$(CROSS_COMPILE)ld -Ttext 0x87800000 led.o -o led.elf

led.o: led.s
	$(CROSS_COMPILE)gcc -g -c led.s -o led.o

clean:
	rm -rf *.o led.elf led.bin led.dis
```

---

## 四、致命坑：Tab 缩进

| 问题 | 原因 | 解决 |
|------|------|------|
| `*** missing separator` | 命令前用了空格 | 退格删掉，按 Tab 键补上 |

Makefile 硬性规定：**命令行的缩进必须用 Tab 键，空格不行**。

从网页复制代码最容易踩这个坑——粘贴后空格还在，看着一样，make 不认。

---

## 五、-Ttext 与链接脚本

| 方式 | 写法 | 适合 |
|------|------|------|
| 内联地址 | `ld -Ttext 0x87800000` | 单文件，初学直观 |
| 链接脚本 | `ld -Tled.lds` | 多文件，需要控制段布局 |

`-Ttext` 直接指定代码段的起始地址，等于告诉链接器"代码将来在 `0x87800000` 处运行"。

---

## 六、make 自动推导

make 通过比较文件时间戳决定是否重新编译：

```
led.bin 比 led.elf 旧？ → 重新生成 led.bin
led.elf 比 led.o 旧？   → 重新链接
led.o 比 led.s 旧？     → 重新编译
```

只改了一个源文件，make 只重编受影响的环节，不会全量重来。

---

## 七、日常开发节奏

```bash
# 编辑源码 → 保存
make                          # 一键编译
./imxdownload led.bin /dev/null  # 加 DCD 头部
# load.imx 拖回 Mac → dd 烧录到 SD 卡
```

> 外接 USB 读卡器直连虚拟机后，编译+烧录全在虚拟机完成，不用再两头传文件。

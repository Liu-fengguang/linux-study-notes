# 通用 Makefile 详解

> 逐行拆解工业级 Makefile：每条代码的意思，每个函数的用法。

---

## 第一部分：基础变量与路径

```makefile
CROSS_COMPILE ?= arm-linux-gnueabihf-
TARGET        ?= ledc

CC      := $(CROSS_COMPILE)gcc
LD      := $(CROSS_COMPILE)ld
OBJCOPY := $(CROSS_COMPILE)objcopy
OBJDUMP := $(CROSS_COMPILE)objdump

INCUDIRS := imx6u \
            bsp/clk \
            bsp/led \
            bsp/delay

SRCDIRS  := project \
            bsp/clk \
            bsp/led \
            bsp/delay
```

### 逐行解释

| 代码 | 含义 |
|------|------|
| `?=` | 条件赋值：变量没定义才赋，定义了就不覆盖。用户可 `make CROSS_COMPILE=xxx` 覆盖 |
| `:=` | 立即展开：定义时就求值，不等用到再展开 |
| `$(CROSS_COMPILE)gcc` | 拼接：`arm-linux-gnueabihf-` + `gcc` = `arm-linux-gnueabihf-gcc` |
| `\` | Makefile 换行符，纯为了让代码不超出一屏 |
| `INCUDIRS` | 头文件所在的文件夹列表 |
| `SRCDIRS` | 源文件（`.c` `.S`）所在的文件夹列表 |

### 三种赋值对比

| 赋值符 | 行为 | 时机 |
|--------|------|------|
| `=` | 延迟展开 | 用到时才求值 |
| `:=` | 立即展开 | 定义时就求值 |
| `?=` | 条件赋值 | 变量为空才赋值 |
| `+=` | 追加 | 往已有值后面加 |

---

## 第二部分：函数魔法 — 自动搜集文件

```makefile
INCLUDE := $(patsubst %, -I %, $(INCUDIRS))

SFILES := $(foreach dir, $(SRCDIRS), $(wildcard $(dir)/*.S))
CFILES := $(foreach dir, $(SRCDIRS), $(wildcard $(dir)/*.c))
```

### `patsubst` — 模式替换

```
$(patsubst 原模式, 目标模式, 文本列表)
```

```makefile
INCLUDE := $(patsubst %, -I %, $(INCUDIRS))
```

| 输入 | 输出 |
|------|------|
| `imx6u` | `-I imx6u` |
| `bsp/clk` | `-I bsp/clk` |
| `bsp/led` | `-I bsp/led` |

> `%` = 通配符。拿着列表里每个元素替换到 `%` 的位置。
> 效果等同于手写 `-I imx6u -I bsp/clk -I bsp/led`。

### `foreach` — 循环

```
$(foreach 临时变量, 列表, 要执行的表达式)
```

```makefile
$(foreach dir, $(SRCDIRS), $(wildcard $(dir)/*.S))
```

Makefile 版 for 循环：

```
dir = project  →  执行 wildcard(project/*.S)
dir = bsp/clk  →  执行 wildcard(bsp/clk/*.S)
dir = bsp/led  →  执行 wildcard(bsp/led/*.S)
```

### `wildcard` — 通配符展开

```
$(wildcard 路径/*.c)
```

自动把指定目录下所有匹配文件找出来。等价于 shell 的 `ls bsp/led/*.c`。

三个函数合起来的效果：

```
SFILES = project/start.S
CFILES = bsp/clk/bsp_clk.c bsp/led/bsp_led.c bsp/delay/bsp_delay.c main.c ...
```

---

## 第三部分：路径剥离

```makefile
SFILENDIR := $(notdir $(SFILES))
CFILENDIR := $(notdir $(CFILES))

SOBJS := $(patsubst %, obj/%, $(SFILENDIR:.S=.o))
COBJS := $(patsubst %, obj/%, $(CFILENDIR:.c=.o))
OBJS  := $(SOBJS) $(COBJS)
```

### `notdir` — 去掉路径

```
$(notdir 文件列表)
```

| 输入 | 输出 |
|------|------|
| `project/start.S` | `start.S` |
| `bsp/led/bsp_led.c` | `bsp_led.c` |

### 变量后缀替换

```
$(变量名:.旧后缀=.新后缀)
```

```
$(SFILENDIR:.S=.o)   →   start.S 变成 start.o
$(CFILENDIR:.c=.o)   →   bsp_led.c 变成 bsp_led.o
```

### 再加 obj/ 前缀

```
$(patsubst %, obj/%, $(SFILENDIR:.S=.o))
```

| 输入 | 输出 |
|------|------|
| `start.o` | `obj/start.o` |
| `bsp_led.o` | `obj/bsp_led.o` |

最终结果：

```makefile
OBJS = obj/start.o obj/bsp_clk.o obj/bsp_led.o obj/bsp_delay.o obj/main.o
```

---

## 第四部分：VPATH 寻宝雷达

```makefile
VPATH := $(SRCDIRS)
```

| 问题 | 解决 |
|------|------|
| `notdir` 扒掉了路径，Makefile 找不到 `bsp_led.c` 在哪 | `VPATH` 告诉 make："去这些目录找" |

VPATH 是 make 内置变量。当 make 在当前目录找不到源文件时，自动去 VPATH 列出的路径挨个搜索。

> 这就是为什么分文件夹放源文件，`notdir` 去掉路径后还能编译成功的终极原因。

---

## 第五部分：总装配线

```makefile
.PHONY: clean

$(TARGET).bin: $(OBJS)
	$(LD) -Timx6u.lds -o $(TARGET).elf $^
	$(OBJCOPY) -O binary -S $(TARGET).elf $@
	$(OBJDUMP) -D -m arm $(TARGET).elf > $(TARGET).dis
```

### `.PHONY` — 伪目标

告诉 make：`clean` 不是文件名，每次都要执行。防止工程里有同名 `clean` 文件导致 `make clean` 失效。

### 自动化变量

| 变量 | 含义 | 此处展开为 |
|------|------|-----------|
| `$^` | 所有依赖 | `obj/start.o obj/bsp_clk.o ...` |
| `$@` | 目标文件名 | `ledc.bin` |
| `$<` | 第一个依赖 | `start.S` 或 `bsp_clk.c` |
| `$(@D)` | 目标所在目录 | `obj` |
| `$(@F)` | 目标文件名（去目录） | `ledc.bin` |

### 三条命令

| 命令 | 做什么 |
|------|--------|
| `ld ... $^` | 链接所有 `.o` → `.elf` |
| `objcopy ... $@` | `.elf` → 纯 `.bin` |
| `objdump ...` | `.elf` → `.dis` 反汇编 |

---

## 第六部分：静态模式规则

```makefile
$(SOBJS): obj/%.o : %.S
	$(CC) -Wall -nostdlib -c -O2 $(INCLUDE) -o $@ $<

$(COBJS): obj/%.o : %.c
	$(CC) -Wall -nostdlib -c -O2 $(INCLUDE) -o $@ $<
```

### 语法

```
目标集合 : 目标模式 : 依赖模式
    命令
```

### 拆解过程

拿出 `$(SOBJS)` 里的第一个目标 `obj/start.o`：

```
obj/start.o  匹配  obj/%.o  →  提取出核心词 % = start
套入依赖模式  %.S          →  依赖文件 = start.S
结合 VPATH 找到 project/start.S
执行命令：把 start.S 编译成 obj/start.o
```

然后自动循环处理 `$(SOBJS)` 里的下一个目标。

> 一条规则处理全部文件。1 个汇编文件和 100 个汇编文件，Makefile 不用改一行。

### 编译选项

| 选项 | 含义 |
|------|------|
| `-Wall` | 显示所有警告 |
| `-nostdlib` | 不链接标准库（裸机不需要） |
| `-c` | 只编译不链接 |
| `-O2` | 二级优化 |
| `$(INCLUDE)` | 头文件搜索路径（`-I xxx`） |

---

## 第七部分：清理

```makefile
clean:
	rm -rf $(TARGET).elf $(TARGET).dis $(TARGET).bin $(OBJS)
```

---

## 全文速查表

| Makefile 元素 | 是什么 | 例子 |
|--------------|--------|------|
| `$@` | 当前目标 | `ledc.bin` |
| `$^` | 所有依赖 | `obj/a.o obj/b.o obj/c.o` |
| `$<` | 第一个依赖 | `start.S` |
| `patsubst` | 模式替换 | `$(patsubst %, -I %, dirs)` |
| `foreach` | 循环 | `$(foreach d, $(dirs), ...)` |
| `wildcard` | 通配符展开 | `$(wildcard *.c)` |
| `notdir` | 去掉路径 | `$(notdir bsp/led/bsp_led.c)` → `bsp_led.c` |
| `VPATH` | 自动搜索路径 | make 找不到文件时去这找 |
| `.PHONY` | 伪目标声明 | 强制每次执行 |
| `?=` | 条件赋值 | 没定义才赋 |
| `:=` | 立即赋值 | 定义时就求值 |

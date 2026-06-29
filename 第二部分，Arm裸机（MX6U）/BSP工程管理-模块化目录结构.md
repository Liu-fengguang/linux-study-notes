# BSP 工程管理：模块化目录结构

> 同一属性的文件放同一个目录。代码多了不散架。

---

## 一、为什么需要 BSP 工程管理

### 没有工程管理

```
1_leds/
├── main.c
├── startup.s
├── led.lds
├── Makefile
├── fsl_gpio.c      ← 混乱
├── fsl_gpio.h
├── fsl_iomuxc.h
├── fsl_common.h
├── fsl_clock.c
└── ...              ← 20 个文件全堆在一起
```

### 有 BSP 工程管理

```
1_leds/
├── bsp/             ← 板级驱动（你写的）
│   ├── led/
│   │   ├── bsp_led.c
│   │   └── bsp_led.h
│   ├── clk/
│   │   ├── bsp_clk.c
│   │   └── bsp_clk.h
│   └── delay/
│       ├── bsp_delay.c
│       └── bsp_delay.h
├── imx6ull/         ← 芯片 SDK（原厂/第三方）
│   ├── inc/         ← 头文件
│   └── src/         ← 源文件
├── obj/             ← 编译中间产物
├── project/          ← 启动+链接+Makefile
│   ├── startup.s
│   ├── led.lds
│   └── Makefile
└── main.c
```

---

## 二、每个文件夹是什么、放什么

| 文件夹 | 名字含义 | 放什么 | 修改频率 |
|--------|----------|--------|----------|
| `bsp/` | Board Support Package | 你自己写的板级驱动（led、按键、蜂鸣器） | 经常改 |
| `imx6ull/` | 芯片 SDK | NXP 提供的寄存器头文件和外设驱动（gpio、iomuxc、clock） | 几乎不改 |
| `obj/` | object | 编译生成的 `.o` 临时文件 | 自动生成 |
| `project/` | 项目配置 | 启动文件 `startup.s`、链接脚本 `.lds`、`Makefile` | 偶尔改 |
| 根目录 | — | `main.c` + 业务逻辑 | 经常改 |

### 详细拆解

**`bsp/`** — 板级支持包。你针对自己这块开发板写的外设驱动。按外设分目录（led/、clk/、delay/），每个目录一对 `.c` + `.h`。文件命名加 `bsp_` 前缀区分于 SDK。

**`imx6ull/`** — 芯片 SDK 文件。分成 `inc/`（头文件）和 `src/`（源文件），全部来自 NXP 官方，不自己修改。头文件包含所有寄存器的结构体定义。

**`obj/`** — 编译中间产物。`.o` 文件的存放处。Makefile 里指定 `-o obj/xxx.o`，编译完这个目录里就是一堆 `.o`，用 `make clean` 清掉。

**`project/`** — 三个核心文件：启动（`startup.s`）负责设栈、清 BSS 并跳 C；链接（`.lds`）规定内存布局；Makefile 负责自动化编译。

---

## 三、对应 Makefile 怎么写

目录分层后，Makefile 需要告诉编译器去哪找文件：

```makefile
CROSS_COMPILE ?= arm-linux-gnueabihf-

CC      = $(CROSS_COMPILE)gcc
LD      = $(CROSS_COMPILE)ld
OBJCOPY = $(CROSS_COMPILE)objcopy

TARGET  = obj/led

# 头文件搜索路径（每个目录的 .h 都要指到）
INCLUDE = -I bsp/led -I bsp/clk -I bsp/delay -I imx6ull/inc

# 所有 .c 源文件
SRCS  = main.c
SRCS += bsp/led/bsp_led.c
SRCS += bsp/clk/bsp_clk.c
SRCS += bsp/delay/bsp_delay.c

# 源文件换成 .o
OBJS = $(patsubst %.c, obj/%.o, $(notdir $(SRCS)))

all: $(TARGET).bin

$(TARGET).bin: $(TARGET).elf
	$(OBJCOPY) -O binary -g -S $< $@

$(TARGET).elf: $(OBJS)
	$(LD) -T project/led.lds $^ -o $@

obj/%.o: %.c
	$(CC) -g -c $< -o $@ $(INCLUDE)

obj/%.o: bsp/%.c
	$(CC) -g -c $< -o $@ $(INCLUDE)

obj/%.o: imx6ull/src/%.c
	$(CC) -g -c $< -o $@ $(INCLUDE)

.PHONY: clean
clean:
	rm -rf obj/*
```

---

## 四、工程管理前后对比

| | 全堆在根目录 | BSP 工程管理 |
|------|------------|-------------|
| 找文件 | 在 20 个文件里翻 | 按文件夹定位，秒找 |
| 新增外设 | 根目录多加 2 个文件 | `bsp/` 下新建子目录 |
| 换芯片 | 全部重写 | 只换 `imx6ull/` 目录 |
| 团队协作 | 容易互相覆盖 | 各改各的 bsp 目录 |
| Makefile | 简单但重复 | 复杂但可复用 |

# SDK 介绍与使用

> SDK（Software Development Kit，软件开发工具包）= 芯片原厂提前写好的底层代码。从手搓寄存器升级到调用现成 API。

---

## 一、什么是 SDK

| 对比 | 纯寄存器手搓 | 使用 SDK |
|------|------------|----------|
| 比喻 | 砍树、刨木板、打钉子 | 宜家套件，切割好、打好孔 |
| 寄存器定义 | 自己查手册写 `#define` | 原厂已写好结构体指针 |
| 点灯 | `GPIO5_DR &= ~(1 << 1)` | `GPIO_PinWrite(GPIO5, 1, 0)` |
| 出错率 | 高，地址写错不报 | 低，函数名写错编译器直接报 |
| 新增外设 | 重新翻手册查寄存器 | 找到对应 `.c/.h`，调用 API |

---

## 二、SDK 里有什么

以 NXP i.MX6ULL SDK 为例：

```
sdk/
├── devices/                  ← 芯片头文件
│   ├── MK64F12.h             ←   CPU 寄存器定义
│   ├── fsl_common.h          ←   通用宏（BIT、ARRAY_SIZE 等）
│   └── fsl_device_registers.h ←   外设寄存器结构体定义
│
├── drivers/                  ← 外设驱动（每个外设一对 .c + .h）
│   ├── fsl_gpio.c / .h       ←   GPIO 操作函数
│   ├── fsl_iomuxc.c / .h     ←   IO 复用配置
│   ├── fsl_clock.c / .h      ←   时钟树配置
│   ├── fsl_uart.c / .h       ←   串口
│   ├── fsl_i2c.c / .h        ←   I2C
│   ├── fsl_spi.c / .h        ←   SPI
│   └── fsl_enet.c / .h       ←   以太网
│
├── boards/                   ← 示例工程（每个开发板一套）
│   └── evkmimxrt1020/
│       ├── led_blinky/        ←   点灯示例
│       ├── hello_world/       ←   串口打印示例
│       └── ...
│
├── middleware/                ← 中间件
│   ├── sdmmc/                 ←   SD 卡协议栈
│   ├── usb/                   ←   USB 协议栈
│   └── fatfs/                 ←   文件系统
│
├── CMSIS/                     ← ARM 标准接口
│   ├── startup_MK64F12.S     ←   官方启动文件
│   └── system_MK64F12.c      ←   系统初始化
│
└── tools/                     ← 辅助工具
```

### `fsl_` 前缀的含义

NXP 的前身是 **Freescale Semiconductor**。`fsl` = **F**ree**s**ca**l**e。NXP 收购后沿用至今。看到 `fsl_gpio.h` 就知道：这是 NXP 官方的 GPIO 驱动。

---

## 三、SDK 函数内部是怎么操作寄存器的？

SDK 不是魔法，它底层还是操作寄存器。打开 `fsl_gpio.c` 看看：

```c
// SDK 提供的函数（你调用的）
void GPIO_PinWrite(GPIO_Type *base, unsigned int pin, unsigned char output)
{
    if (output == 0U) {
        base->DR &= ~(1U << pin);    // 清零 → 低电平
    } else {
        base->DR |= (1U << pin);     // 置位 → 高电平
    }
}
```

SDK 只是把你手写的位操作**封装进函数**。本质没变，但：
- 函数名告诉你意图（`PinWrite` = 写引脚）
- 参数名告诉你含义（`output` = 0 还是 1）
- 不需要记寄存器叫 DR 还是 ODR

---

## 四、SDK 的三层结构

| 层次 | 是什么 | 举例 | 你需不需要改 |
|------|--------|------|------------|
| 寄存器层 | `#define` 或结构体定义地址 | `GPIO5_DR *(volatile...)0x020AC000` | 不动，原厂写好 |
| 驱动层 | 封装寄存器操作的函数 | `GPIO_PinWrite(GPIO5, 1, 0)` | 不动，原厂写好 |
| 应用层 | 你写的业务逻辑 | 蜂鸣器每隔 500ms 响一次 | 你写的 |

---

## 五、怎么使用 SDK：完整流程

### 第 1 步：获取

NXP 官网 → 选芯片 i.MX6ULL → 下载 SDK 压缩包（几百 MB）→ 解压。

### 第 2 步：移植（摘菜）

SDK 几百 MB，不能全塞进工程。按需挑选：

| 你需要什么功能 | 去哪个目录拿什么文件 |
|--------------|-------------------|
| 点灯 / 蜂鸣器 | `drivers/fsl_gpio.c` `fsl_gpio.h` |
| 配置 IO 复用 | `drivers/fsl_iomuxc.c` `fsl_iomuxc.h` |
| 配置时钟 | `drivers/fsl_clock.c` `fsl_clock.h` |
| 通用宏定义 | `devices/fsl_common.h` |
| 寄存器基地址 | `devices/MK64F12.h` 或 `imx6ull.h` |
| 启动文件 | `CMSIS/startup_MK64F12.S` |
| 系统初始化 | `CMSIS/system_MK64F12.c` |

放入工程后的目录结构：

```
蜂鸣器实验/
├── bsp/beep/           ← 你自己写的板级驱动
├── bsp/clk/            ← 你自己写的时钟（或直接调 SDK）
├── bsp/delay/
├── sdk/                ← 从 NXP SDK 摘过来的文件（不动）
│   ├── fsl_gpio.c
│   ├── fsl_gpio.h
│   ├── fsl_iomuxc.h
│   ├── fsl_common.h
│   └── imx6ull.h
└── project/
```

### 第 3 步：告诉编译器

修改 Makefile，加 `-I` 让编译器知道去哪找 SDK 的头文件：

```makefile
INCUDIRS := imx6ull/inc \
            sdk \
            bsp/clk \
            bsp/delay \
            bsp/beep
```

### 第 4 步：包含并调用

```c
#include "fsl_gpio.h"
#include "fsl_iomuxc.h"
#include "fsl_common.h"

int main(void)
{
    CLOCK_EnableClock(kCLOCK_Iomuxc);   /* SDK：使能 IOMUXC 时钟 */
    CLOCK_EnableClock(kCLOCK_Gpio5);    /* SDK：使能 GPIO5 时钟 */

    IOMUXC_SetPinMux(IOMUXC_SNVS_TAMPER1, 5);  /* SDK：复用为 GPIO5_IO01 */

    gpio_pin_config_t config = { kGPIO_DigitalOutput, 0 };
    GPIO_PinInit(GPIO5, 1, &config);     /* SDK：初始化为输出 */

    GPIO_PinWrite(GPIO5, 1, 0);          /* SDK：输出低电平 → 蜂鸣器响 */
}
```

> `GPIO_PinWrite(GPIO5, 1, 0)` 一眼看懂：GPIO5 的第 1 脚，输出 0。比 `GPIO5_DR &= ~(1 << 1)` 直观得多。

---

## 六、SDK 文件的依赖链（移植时容易踩坑）

SDK 文件之间有依赖关系。例如 `fsl_gpio.c` 第一行就有：

```c
#include "fsl_common.h"
#include "fsl_gpio.h"
```

而 `fsl_gpio.h` 又包含 `fsl_device_registers.h`。

**移植时必须满足完整依赖链**，否则编译报 `file not found`。

常见依赖：

```
fsl_gpio.c
  ├── fsl_gpio.h
  │     ├── fsl_common.h
  │     └── imx6ull.h          ← 寄存器基地址定义
  └── fsl_clock.h              ← GPIO 时钟控制
```

> 如果只拷贝 `fsl_gpio.c` 不拷 `fsl_common.h`，编译立刻报错。

---

## 七、纯寄存器 vs SDK（实战对比）

### 蜂鸣器响（你现在的写法）

```c
/* 纯寄存器：4 行代码，每行都要知道寄存器名和 bit */
IOMUXC_SW_MUX_CTL_PAD_SNVS_TAMPER1 = 0x5;
IOMUXC_SW_PAD_CTL_PAD_SNVS_TAMPER1 = 0x10B0;
GPIO5_GDIR |= (1 << 1);
GPIO5_DR   &= ~(1 << 1);
```

### 蜂鸣器响（SDK 写法）

```c
/* SDK：4 行代码，每行都是英语句子 */
IOMUXC_SetPinMux(IOMUXC_SNVS_TAMPER1, 5);
IOMUXC_SetPinConfig(IOMUXC_SNVS_TAMPER1, 0x10B0);

gpio_pin_config_t config = { kGPIO_DigitalOutput, 0 };
GPIO_PinInit(GPIO5, 1, &config);
GPIO_PinWrite(GPIO5, 1, 0);
```

---

## 八、SDK 与你目前学习的关系

| 阶段 | 做什么 | 价值 |
|------|--------|------|
| 现在 | 纯寄存器手搓 | 理解底层，有透视眼 |
| 之后 | 引入 SDK | 工业效率，代码可靠 |

> 学寄存器是为了看懂芯片怎么工作。用 SDK 是为了实战不拖进度。两个都要会。

# SDK 介绍与使用

> SDK = 芯片原厂提前写好的底层代码大礼包。从手搓寄存器升级到调用现成 API。

---

## 一、什么是 SDK

SDK（Software Development Kit，软件开发工具包）。

| 对比 | 纯寄存器（你现在） | SDK |
|------|------------------|-----|
| 比喻 | 砍树、刨木板、打钉子 | 宜家套件，切割好、打好孔 |
| 寄存器定义 | 自己查手册写 `#define` | 原厂已写好结构体头文件 |
| 点灯 | `*GPIO1_DR &= ~(1<<3)` | `GPIO_PinWrite(GPIO1, 3, 0)` |
| 出错率 | 高，地址写错不报 | 低，函数名写错编译器报 |

---

## 二、SDK 里有什么

| 组件 | 内容 | 例子 |
|------|------|------|
| 芯片头文件 | 所有寄存器定义（结构体指针） | `imx6u.h` |
| 外设驱动库 | GPIO/I2C/SPI/UART 的操作函数 | `fsl_gpio.c` `fsl_iomuxc.h` |
| 示例工程 | 点灯、串口、屏幕等现成代码 | `boards/` 目录 |
| 启动与链接 | 写好的 `startup.s` 和 `.lds` | 开箱即用 |

---

## 三、纯寄存器 vs SDK（代码对比）

```c
// 纯寄存器 — 你必须知道 bit3 清零 = 低电平 = 亮
GPIO1->DR &= ~(1 << 3);

// SDK — 函数名自己会说话
GPIO_PinWrite(GPIO1, 3, 0);
```

> SDK 的 API 封装了寄存器细节，你不再需要记住 DR 清零还是置位，函数名本身就表达了意图。

---

## 四、怎么使用 SDK

### 第 1 步：获取

NXP 官网 → 选芯片 i.MX6ULL → 下载 SDK 压缩包 → 解压。标准目录结构：

```
sdk/
├── devices/      ← 芯片头文件
├── drivers/      ← 外设驱动（.c + .h）
├── boards/       ← 示例工程
└── middleware/   ← 中间件
```

### 第 2 步：移植（摘菜）

SDK 动辄几百 MB，不能全塞进工程。只挑需要的文件：

| 你需要什么 | 去哪个目录找 |
|-----------|------------|
| 点灯 | `drivers/fsl_gpio.c` `fsl_gpio.h` |
| IO 复用 | `drivers/fsl_iomuxc.h` |
| 通用宏定义 | `devices/fsl_common.h` |

### 第 3 步：告诉编译器

修改 Makefile，加 `-I` 指定头文件搜索路径：

```makefile
INCLUDES = -I ./sdk/devices -I ./sdk/drivers
$(CC) -g -c main.c -o main.o $(INCLUDES)
```

### 第 4 步：包含并调用

```c
#include "fsl_gpio.h"
#include "fsl_iomuxc.h"

int main(void) {
    GPIO_PinInit(GPIO1, 3, OUTPUT);   // 初始化
    GPIO_PinWrite(GPIO1, 3, 0);       // 点亮
}
```

---

## 五、SDK 与你目前学习的关系

| 阶段 | 做什么 | 价值 |
|------|--------|------|
| 现在 | 纯寄存器手搓 | 理解底层，有透视眼 |
| 之后 | 引入 SDK | 工业效率，代码可靠 |

> 学寄存器是为了看懂芯片怎么工作。用 SDK 是为了实战不拖进度。两个都要会。

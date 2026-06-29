# 跨文件夹编译与 .c/.h 文件关系

> 电脑怎么找到不同文件夹下的头文件？.c 和 .h 有什么区别？为什么不能把 .c 直接 include？

---

## 一、电脑怎么找到其他文件夹的 .h？

编译器默认**只在当前目录**找头文件。看到 `#include "bsp_led.h"` 就搜当前文件夹，找不到立即报 `No such file or directory`。

**答案**：Makefile 里的 `-I` 参数。

```makefile
INCLUDE = -I bsp/led -I bsp/clk -I bsp/delay -I imx6ull/inc
```

实际编译命令变成：

```bash
gcc -c main.c -I bsp/led -I bsp/clk -I bsp/delay -I imx6ull/inc
```

编译器拿着 `-I` 这张寻宝地图，在 `#include "bsp_led.h"` 时依次去地图上的路径翻找，在 `bsp/led/` 里找到后把内容复制进 `main.c`。

---

## 二、不同文件夹的代码怎么连在一起？

两个独立阶段：**编译**（各干各的）+ **链接**（总拼装）。

### 编译阶段：各自造零件

```
main.c ──gcc──→ main.o     （调用 led_init，留一个空洞）
bsp_led.c ──gcc──→ bsp_led.o （提供 led_init 的真实机器码）
bsp_clk.c ──gcc──→ bsp_clk.o
```

每个 `.c` 独立编译，互不知道对方存在。`main.o` 里有 `led_init` 的空洞（外部符号），`bsp_led.o` 里对外宣布"我有 led_init 的实现"。

### 链接阶段：总拼装

链接器按 Makefile 指令把散落的 `.o` 全部抓到一起：

```
main.o + bsp_clk.o + bsp_delay.o + bsp_led.o
        │
   链接器 ld
        │
   填补空洞，算出绝对地址
        │
   led.elf（完整可执行文件）
```

> 编译时 `.c` 各自为政，链接时 `.o` 才焊在一起。

---

## 三、.c 和 .h 的区别

| | .h（头文件） | .c（源文件） |
|------|------------|------------|
| 比喻 | 餐厅菜单 | 后厨菜谱 |
| 内容 | 函数声明、宏定义、typedef | 函数实现、真实逻辑 |
| 占内存 | ❌ 不生成机器码 | ✅ 编译后占内存 |
| 被谁 include | 谁想用这个功能就 include | 只被自己的 .h include |

```c
// bsp_led.h — 菜单（只承诺不实现）
#ifndef __BSP_LED_H
#define __BSP_LED_H
void led_init(void);
void led_on(void);
void led_off(void);
#endif

// bsp_led.c — 菜谱（真正干活的代码）
#include "bsp_led.h"
void led_init(void) { /* 配置 GPIO */ }
void led_on(void)  { /* 输出低电平 */ }
void led_off(void) { /* 输出高电平 */ }
```

---

## 四、为什么不能直接 #include .c？

**#include 的本质是复制粘贴**。`#include "xxx"` = 把 xxx 文件的内容全选、复制、粘贴到当前位置。

### 如果 main.c 直接 include bsp_led.c

```
main.c  #include "bsp_led.c"  →  bsp_led.c 的代码被复制到 main.c
bsp_delay.c  #include "bsp_led.c"  →  同一份代码又被复制到 bsp_delay.c
```

编译结果：
- `main.o` 里有一份 `led_init` 的机器码
- `bsp_delay.o` 里也有一份**一模一样**的 `led_init` 机器码

链接器：`Error: multiple definition of 'led_init'`

> 两份同样的实体代码，链接器不知道用哪个，直接罢工。

### 用 .h 为什么没事？

`.h` 里只有声明（承诺），没有实体代码。被 include 一百次，也只是多了一百张菜单，不产生重复机器码。

真正实体代码只在 `bsp_led.c` 编译出的**唯一一份** `bsp_led.o` 里。链接器把所有人的空洞和这唯一实体一一对上。

---

## 五、必须拆分的两个现实原因

| 原因 | 不拆分 | 拆分 |
|------|--------|------|
| 编译速度 | 改一行重编全部（可能几小时） | Makefile 只重改被修改的那个 `.c`（0.1 秒） |
| 代码保密 | 源码全暴露 | 编译成 `.a` / `.so` 库 + `.h` 菜单卖给你 |

> 芯片原厂卖 SDK 只给 `.h`（菜单）和编译好的 `.a` 库（黑盒）。你能调用，看不到源码。

---

## 六、总结

```
main.c                 bsp_led.h              bsp_led.c
#include "bsp_led.h" →  void led_init();  ←  #include "bsp_led.h"
                              ↑               void led_init() { ... }
                        只声明，不占内存         真实代码，占内存

编译：main.c 和 bsp_led.c 各自独立 → main.o + bsp_led.o
链接：空洞 + 实体 = led.elf
```

# C 语言版 LED 踩坑记录

> 从汇编过渡到 C 语言裸机开发，编译、链接、烧录全流程遇到的问题与解决。

---

## 1. make 找不到 Makefile

**症状**：`make: *** No targets specified and no makefile found. Stop.`

**原因**：终端当前路径不在项目目录里。make 只在当前目录找 Makefile。

**检查**：`pwd` 看当前路径，确保提示符是 `~/Desktop/.../C语言版LED驱动实验$`

**解决**：
```bash
cd C语言版LED驱动实验/
```

---

## 2. 交叉编译器未安装

**症状**：`arm-linux-gnueabihf-gcc: command not found`

**解决**：
```bash
sudo apt install gcc-arm-linux-gnueabihf binutils-arm-linux-gnueabihf
```

---

## 3. void main 为什么没报错？

**症状**：教程说 main 必须返回 int，但我写 `void main` 编译通过了。

**原因**：裸机 vs 操作系统，规则不同。

| 环境 | main 返回值 | 原因 |
|------|------------|------|
| 有 OS | 必须 `int` | OS 要接收退出状态码（0=正常） |
| 裸机 | `void` 也可以 | 没有 OS 接收返回值；代码 `while(1)` 不退出 |

GCC 默认宽容，不加 `-Wall` 时只报 Error 不报 Warning。

> 加上 `-Wall` 编译器会弹警告 `warning: return type of 'main' is not 'int'`，但也只是个黄字，不影响生成 .o。

---

## 4. make 成功后 start.s 的两个警告

### 警告①：`newline inserted`

**原因**：汇编文件最后一行没有回车换行。GNU 汇编器要求文件末尾有空行。

**解决**：打开 `start.s`，光标移到最后，按一下回车保存。

### 警告②：`cannot find entry symbol _start`

**原因**：链接器默认找 `_start` 符号作为程序入口，找不到就强行用 `0x87800000`。

两种可能的根因：

| 根因 | 表现 | 解决 |
|------|------|------|
| 拼写错误 | 写成了 `__start`（双下划线） | 改为 `_start`（单下划线） |
| start.s 是空文件 | 忘记写代码或忘保存 | 补全汇编代码并保存 |

> 对最终 `.bin` 无影响，但修正后反汇编更清晰。

---

## 5. 教程反汇编第一行是 main，我的是 _start

**教程为什么错**：讲师的 `start.s` 是空文件（`touch` 创建后忘了写内容或忘保存）。`start.o` 体积为 0 字节，链接器只能把 `main.o` 排在 `0x87800000`。

**你为什么对**：你的 `start.s` 有真实汇编代码，`start.o` 有体积，`_start` 被正确安排在入口地址。

> **你的是标准答案，教程的结果是错误示范。**

---

## 6. -O2 优化导致 main 抢占入口（核心坑）

**真正原因**：`-O2` 优化让编译器给 `main` 开了 VIP 通道。

正常（`-O0`）：所有代码放 `.text`，按 Makefile 顺序拼接（start.o 在前）。

`-O2` 模式：

| 代码 | 放进哪个段 | 优先级 |
|------|-----------|--------|
| start.s 的汇编 | `.text`（普通段） | 低 |
| main() 函数 | `.text.startup`（VIP 段） | 高 |

链接器默认规则：`.text.startup` 的优先级高于 `.text`。

结果：`main` 插队抢占了 `0x87800000`。CPU 上电直接跑 C 代码，此时栈指针未初始化，程序瞬间崩溃。

**裸机致命之处**：CPU 只会从 `0x87800000` 执行，不管放的是 `_start` 还是 `main`。

**解决**：用链接脚本（`.lds`）强制规定代码排版顺序，无视编译器的段命名花样。

```lds
SECTIONS {
    . = 0x87800000;
    .text : {
        startup.o (.text)     /* 必须排第一个 */
        *(.text)
    }
}
```

---

## 速查表

| 序号 | 报错/症状 | 原因 | 解决 |
|------|----------|------|------|
| 1 | `make: No targets...` | 目录不对 | `cd` 到项目目录 |
| 2 | `arm-linux-gnueabihf-gcc: not found` | 未装交叉编译器 | `sudo apt install gcc-arm-linux-gnueabihf` |
| 3 | `void main` 没报错 | 裸机允许，GCC 默认宽容 | 加 `-Wall` 会提示 |
| 4 | `newline inserted` | 文件末尾没回车 | 最后一行按回车 |
| 5 | `cannot find entry symbol _start` | `__start` 双下划线或 start.s 为空 | 改为 `_start`，补全代码 |
| 6 | 反汇编第一行是 main | -O2 优化，`.text.startup` 插队 | 用链接脚本锁死顺序 |

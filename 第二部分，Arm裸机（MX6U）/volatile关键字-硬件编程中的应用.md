# volatile 关键字

## 一、是什么

`volatile` 告诉编译器：**这个变量的值可能随时被外部改变，不要优化它，每次使用都老老实实从内存重新读。**

```c
volatile unsigned int *p = (unsigned int *)0x0209C000;
```

---

## 二、为什么裸机编程必须用它

编译器会自作聪明地优化代码。看一个反例：

### 不加 volatile（被优化掉）

```c
unsigned int *GPIO1_DR = (unsigned int *)0x0209C000;

*GPIO1_DR &= ~(1 << 3);     // 点亮 LED
delay();
*GPIO1_DR |= (1 << 3);      // 熄灭 LED
delay();
*GPIO1_DR &= ~(1 << 3);     // 点亮 LED
```

编译器看到的是：

> "同一地址被连续重复读写，都是废话，直接保留最后一个结果就行。"

于是编译器把你精心写的亮→灭→亮，优化成了只执行最后一步。LED 不闪，你的代码"消失"了。

### 加 volatile（禁止优化）

```c
volatile unsigned int *GPIO1_DR = (volatile unsigned int *)0x0209C000;

*GPIO1_DR &= ~(1 << 3);     // 1. 必须读-改-写
delay();
*GPIO1_DR |= (1 << 3);      // 2. 必须读-改-写
delay();
*GPIO1_DR &= ~(1 << 3);     // 3. 必须读-改-写
```

`volatile` 告诉编译器："这个地址的每一步操作都必须忠实执行，不准省略。"

---

## 三、编译器做了什么优化？

| 优化类型 | 不加 volatile | 加 volatile |
|----------|--------------|-------------|
| 删除重复写入 | 同一地址连续写，只保留最后一次 | 每次写入都执行 |
| 缓存到寄存器 | 第一次读后存寄存器，后面直接用寄存器值 | 每次都从内存重读 |
| 删除无用读 | 读出来没用的值，直接优化掉 | 保留每次读取 |

---

## 四、哪些场景必须用 volatile

| 场景 | 原因 |
|------|------|
| 硬件寄存器 | 寄存器的值由硬件改变，编译器不知道 |
| 中断服务函数修改的变量 | 主循环和 ISR 共享的变量，编译器看不出 ISR 会改它 |
| 多线程共享变量 | 另一个线程可能随时改 |
| 内存映射 IO | 读写操作本身就有副作用（清除中断标志等） |

---

## 五、示例：LED 闪烁代码对比

```c
/* ❌ 没加 volatile — 被优化后 LED 不闪 */
#define GPIO1_DR (*(unsigned int *)0x0209C000)

/* ✅ 加了 volatile — 每次读写都真实发生 */
#define GPIO1_DR (*(volatile unsigned int *)0x0209C000)

while (1) {
    GPIO1_DR &= ~(1 << 3);    // 真实读寄存器 → 改 → 写回
    delay(0xFFFFF);
    GPIO1_DR |= (1 << 3);     // 同上
    delay(0xFFFFF);
}
```

---

## 六、容易混淆的三个用词

| 写法 | 含义 |
|------|------|
| `volatile int *p` | p 指向的值是 volatile（数据可能被外部改变） |
| `int *volatile p` | p 本身是 volatile（指针变量本身可能被外部改变） |
| `volatile int *volatile p` | 指针本身和指向的值都是 volatile |

裸机寄存器操作用的都是第一种：

```c
#define GPIO1_DR (*(volatile unsigned int *)0x0209C000)
```

> `volatile` 不是锁，不保证原子性。它只管"别优化掉"，不管"别被中断打断"。

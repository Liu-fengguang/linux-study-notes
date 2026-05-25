# Makefile 基本语法

## 从简单到标准，逐步演进

假设有 `main.c`、`input.c`、`calcu.c` 三个源文件。

---

## 阶段一：不用 Makefile（最原始）

```bash
gcc main.c input.c calcu.c -o main
```

每次改一个文件也要全部重新编译，效率最低。

---

## 阶段二：最简单的 Makefile（单文件）

```makefile
main: main.c input.c calcu.c
	gcc -o main main.c input.c calcu.c
```

运行 `make`，如果任何源文件比 `main` 新，就重新编译。

---

## 阶段三：多文件分别编译（教材写法）

每个 `.c` 先编译成 `.o`，再链接：

```makefile
main: main.o input.o calcu.o
	gcc -o main main.o input.o calcu.o

main.o: main.c
	gcc -c main.c

input.o: input.c
	gcc -c input.c

calcu.o: calcu.c
	gcc -c calcu.c

clean:
	rm *.o main
```

> 规则格式：`目标: 依赖` 下一行 Tab 缩进 + 命令。命令前**必须是 Tab**，不能是空格。

分开编译的好处：改了 `input.c` 只重新编译 `input.o`，`main.o` 和 `calcu.o` 不动，省时间。

---

## 阶段四：引入变量

常用部分抽成变量，改一处全生效：

```makefile
CC = gcc
TARGET = main
OBJS = main.o input.o calcu.o

$(TARGET): $(OBJS)
	$(CC) -o $(TARGET) $(OBJS)

main.o: main.c
	$(CC) -c main.c

input.o: input.c
	$(CC) -c input.c

calcu.o: calcu.c
	$(CC) -c calcu.c

clean:
	rm $(OBJS) $(TARGET)
```

> 引用变量用 `$(变量名)`。

### 变量赋值运算符

| 运算符 | 行为 | 时机 |
|--------|------|------|
| `=` | 延迟展开 | 使用时才求值 |
| `:=` | 立即展开 | 定义时就求值 |
| `+=` | 追加 | 给已有变量追加值 |
| `?=` | 条件赋值 | 变量未定义时才赋值 |

**`=` vs `:=` 的区别：**

```makefile
A = $(B)       # 延迟：定义时不管 B 是什么，用的时候再展开
B = hello
# 此时 $(A) 展开为 hello
```

```makefile
C := $(D)      # 立即：定义时 D 还是空，所以 C 就是空
D = hello
# 此时 $(C) 为空，因为定义 C 时 D 还未定义
```

**`+=` 追加：**

```makefile
OBJS = main.o
OBJS += input.o
# OBJS = main.o input.o
```

```makefile
CFLAGS = -Wall
CFLAGS += -g
# CFLAGS = -Wall -g
```

**`?=` 条件赋值：**

```makefile
CC ?= gcc        # 如果 CC 没定义过，赋值为 gcc
CC ?= clang      # CC 已有 gcc，这条被忽略
```

> 实际项目里 `CC ?= gcc` 很常用：允许用户通过命令行覆盖 `make CC=clang`，不指定就默认 gcc。

---

## 阶段五：自动变量

不再写死文件名，用自动变量代替：

| 自动变量 | 含义 |
|----------|------|
| `$@` | 目标名 |
| `$<` | 第一个依赖 |
| `$^` | 所有依赖 |

```makefile
CC = gcc
TARGET = main
OBJS = main.o input.o calcu.o

$(TARGET): $(OBJS)
	$(CC) -o $@ $^

main.o: main.c
	$(CC) -c $<

input.o: input.c
	$(CC) -c $<

calcu.o: calcu.c
	$(CC) -c $<

clean:
	rm $(OBJS) $(TARGET)
```

---

## 阶段六：模式规则（最终标准写法）

`%.o: %.c` 一条规则替代所有 `.o` 规则，新增 `.c` 也无需改 Makefile：

```makefile
CC = gcc
TARGET = main
OBJS = main.o input.o calcu.o

$(TARGET): $(OBJS)
	$(CC) -o $@ $^

%.o: %.c
	$(CC) -c $<

.PHONY: clean
clean:
	rm $(OBJS) $(TARGET)
```

---

## 演进总结

| 阶段 | 特点 | 适合 |
|------|------|------|
| 一 | 无 Makefile | 单文件测试 |
| 二 | 单条规则 | 多文件但无增量编译 |
| 三 | 每个 `.o` 单写规则 | 教程讲解，清晰直观 |
| 四 | 引入变量 | 改编译选项方便 |
| 五 | 自动变量 `$@ $^ $<` | 减少硬编码文件名 |
| 六 | 模式规则 `%.o: %.c` | 实际项目标准写法 |

---

## 重点提醒

| 要点 | 说明 |
|------|------|
| Tab 缩进 | 命令前必须是 Tab，空格会报 `*** missing separator` |
| `.PHONY` | `clean` 必须声明，防止目录下出现同名文件不执行 |
| 增量编译 | make 只在依赖比目标新时才重新编译 |
| 默认目标 | `make` 不加参数执行第一个目标 |

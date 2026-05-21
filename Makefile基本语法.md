# Makefile 基本语法

## 规则结构

| 部分 | 说明 |
|------|------|
| 目标 (target) | 要生成的文件名 |
| 依赖 (prerequisites) | 生成目标需要的文件 |
| 命令 (recipe) | shell 命令，必须以 **Tab** 开头 |

```makefile
target: prerequisites
	command
```

## 自动变量

| 变量 | 展开为 |
|------|--------|
| `$@` | 目标名 |
| `$<` | 第一个依赖 |
| `$^` | 所有依赖 |
| `$?` | 所有比目标新的依赖 |
| `$(@D)` | 目标所在目录 |
| `$(@F)` | 目标文件名（去掉目录） |

## 常用场景

```makefile
# 简单单文件
hello: hello.c
	gcc hello.c -o hello

# 多文件编译
app: main.o utils.o
	gcc $^ -o $@

main.o: main.c
	gcc -c $<

utils.o: utils.c
	gcc -c $<

clean:
	rm -f *.o app
```

## 变量

```makefile
CC = gcc
CFLAGS = -Wall -g -O2
TARGET = app
SRCS = main.c utils.c
OBJS = $(SRCS:.c=.o)

$(TARGET): $(OBJS)
	$(CC) $^ -o $@
```

| 变量类型 | 语法 | 示例 |
|------|------|------|
| 显式定义 | `VAR = value` | `CC = gcc` |
| 引用 | `$(VAR)` | `$(CC)` |
| 替换后缀 | `$(SRCS:.c=.o)` | main.c → main.o |
| 追加 | `VAR += value` | `CFLAGS += -O2` |
| 条件赋值 | `VAR ?= value` | 未定义才赋值 |

## 模式规则

| 写法 | 含义 |
|------|------|
| `%.o: %.c` | 任意 `.c` 生成同名 `.o` |

```makefile
%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@
```

一个 `%.o: %.c` 规则替代了每个 `.c` 文件都要单独写一条规则的重复劳动。

## 伪目标

```makefile
.PHONY: clean install all

all: app

install:
	cp app /usr/local/bin/

clean:
	rm -f *.o app
```

> `.PHONY` 告诉 make 这些目标不是文件名，总是执行。否则目录下有 `clean` 文件时 `make clean` 会跳过。

## 内置变量

| 变量 | 默认值 |
|------|--------|
| `$(CC)` | cc（即 gcc） |
| `$(CXX)` | g++ |
| `$(CFLAGS)` | 空 |
| `$(CXXFLAGS)` | 空 |
| `$(RM)` | rm -f |

## 条件判断

```makefile
ifeq ($(DEBUG), 1)
    CFLAGS += -g -O0
else
    CFLAGS += -O2
endif
```

使用：`make`（发布模式）、`make DEBUG=1`（调试模式）

## 使用建议

| 建议 | 原因 |
|------|------|
| 用模式规则代替逐个写 | 新增 `.c` 不用改 Makefile |
| 变量放文件顶部 | 改编译器/选项只改一处 |
| `clean` 必加 `.PHONY` | 避免被同名文件干扰 |
| 不写 Makefile 就用 `gcc *.c -o app` | 单文件或简单项目不必过度工程化 |

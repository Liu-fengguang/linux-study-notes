# make 与 Makefile

## 核心概念

| 概念 | 说明 |
|------|------|
| 目标 (target) | 要生成的文件 |
| 依赖 (prerequisites) | 生成目标所需的文件 |
| 命令 (recipe) | 生成目标的 shell 命令 |
| 规则 | 目标 + 依赖 + 命令 |

> Makefile 本质：告诉 make 如何从依赖生成目标，依赖更新时自动重新构建。

## 基本语法

```makefile
目标: 依赖
	命令     # 注意：必须是 Tab 缩进，不能用空格
```

## 单文件示例

```makefile
hello: hello.c
	gcc hello.c -o hello
```

运行：`make` 或 `make hello`

## 多文件示例

```makefile
app: main.o utils.o
	gcc main.o utils.o -o app

main.o: main.c
	gcc -c main.c

utils.o: utils.c
	gcc -c utils.c

clean:
	rm -f *.o app
```

运行：`make`（构建）、`make clean`（清理）

## 变量

| 变量 | 含义 |
|------|------|
| `CC` | 编译器，默认 `gcc` |
| `CFLAGS` | 编译选项 |
| `LDFLAGS` | 链接选项 |
| `$@` | 目标名 |
| `$^` | 所有依赖 |
| `$<` | 第一个依赖 |

```makefile
CC = gcc
CFLAGS = -Wall -g -O2

app: main.o utils.o
	$(CC) $^ -o $@

%.o: %.c
	$(CC) $(CFLAGS) -c $<
```

## 模式规则

| 写法 | 含义 |
|------|------|
| `%.o: %.c` | 所有 `.o` 由同名 `.c` 生成 |
| `.PHONY: clean` | 声明 clean 不是文件名，总是执行 |

## 常用选项

| 命令 | 作用 |
|------|------|
| `make` | 构建第一个目标 |
| `make target` | 构建指定目标 |
| `make -j4` | 4 核并行构建 |
| `make -n` | 预览命令，不实际执行 |
| `make -C dir` | 进入 dir 执行 make |
| `make clean` | 执行 clean 目标 |

## 使用建议

| 建议 | 原因 |
|------|------|
| `clean` 必须声明 `.PHONY` | 防止目录下有 `clean` 文件时不执行 |
| 变量放 Makefile 顶部 | 方便修改编译器、选项 |
| 用模式规则 `%.o: %.c` | 减少重复，新增 `.c` 无需加规则 |
| Tab 不是空格 | 命令前必须是 Tab，编辑器切换后注意 |

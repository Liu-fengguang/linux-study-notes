# Shell 脚本与简单命令

## 1. 是什么

Shell 脚本是一系列 Linux 命令写在一个文件里，一次性执行。后缀 `.sh`，本质是纯文本。

| 概念 | 说明 |
|------|------|
| Shell | 命令行解释器，连接用户和内核 |
| 常用 Shell | bash（Linux 默认）、zsh（macOS 默认）、sh（最基础） |
| 脚本 | 命令按顺序写进文件，省去逐条输入 |

## 2. 怎么用

```bash
# 创建并写入
vim hello.sh

# 内容
#!/bin/bash
echo "hello world"

# 加执行权限 — 然后运行
chmod +x hello.sh
./hello.sh
```

> `#!/bin/bash` 叫 shebang，告诉系统用哪个解释器。必须放在第一行。

| 运行方式 | 说明 |
|----------|------|
| `./hello.sh` | 需要执行权限，开子进程运行 |
| `bash hello.sh` | 不需要执行权限，开子进程运行 |
| `source hello.sh` | 当前 shell 运行，脚本里的变量会留在当前环境 |

## 3. 基本语法

### 变量

```bash
name="alice"          # 定义，= 两边不能有空格
echo $name            # 使用，加 $ 前缀
echo ${name}          # 用花括号明确边界
readonly name         # 设为只读
unset name            # 删除
```

### 字符串

```bash
str='hello'           # 单引号：所有字符原样输出
str="hello $name"     # 双引号：$ 和 反引号 会解析
name="hello"$name"!"  # 拼接
echo ${#name}         # 字符串长度
```

### 数组（bash）

```bash
arr=(a b c d)
echo ${arr[0]}        # 第一个元素
echo ${arr[@]}        # 所有元素
echo ${#arr[@]}       # 元素个数
```

### 条件判断 (if)

```bash
if [ $a -gt $b ]; then
    echo "a > b"
elif [ $a -eq $b ]; then
    echo "a = b"
else
    echo "a < b"
fi                     # if 反写，结束标志
```

### 循环

```bash
for i in 1 2 3; do
    echo $i
done

while [ $n -le 10 ]; do
    echo $n
    n=$((n + 1))
done

case $var in
    start)  echo "启动" ;;
    stop)   echo "停止" ;;
    *)      echo "其他" ;;
esac
```

### 函数

```bash
myfunc() {
    echo "第一个参数: $1"
    return 0              # 返回值 0-255，0 表示成功
}
myfunc hello
```

## 4. 交互式 Shell

| 命令 | 作用 |
|------|------|
| `read name` | 等待用户输入，存入变量 name |
| `read -p "提示:" name` | 带提示的输入 |
| `read -s password` | 隐藏输入（输入密码） |
| `read -t 5 name` | 5 秒超时，超时后跳过 |

```bash
read -p "请输入姓名:" name
echo "你好, $name"
```

## 5. 数值计算

| 写法 | 说明 |
|------|------|
| `$((a + b))` | 整数运算（常用） |
| `$[a + b]` | 同上，老写法 |
| `$(expr $a + $b)` | 调用 expr 命令，`*` 需要转义 `\*` |

```bash
a=10; b=3
echo $((a + b))        # 13
echo $((a - b))        # 7
echo $((a * b))        # 30
echo $((a / b))        # 3（整数除法）
echo $((a % b))        # 1（取余）
```

> 仅支持整数，浮点数需要 `bc` 或 `awk`。

## 6. test 命令 && 与 ||

### test（两种等价写法）

| 写法一 | 写法二 | 含义 |
|--------|--------|------|
| `test -f file` | `[ -f file ]` | 判断文件是否存在 |
| `test $a -eq $b` | `[ $a -eq $b ]` | 判断两数相等 |

> `test` 和 `[]` 是同一个命令，`[]` 只是更好读的写法。

### && 与 ||

| 写法 | 含义 |
|------|------|
| `cmd1 && cmd2` | cmd1 成功才执行 cmd2 |
| `cmd1 \|\| cmd2` | cmd1 失败才执行 cmd2 |
| `cmd1 && cmd2 \|\| cmd3` | cmd1 成功→cmd2，失败→cmd3 |

```bash
make && ./app             # 编译成功才运行
gcc main.c || echo "失败"  # 编译失败才提示
```

## 7. [] 判断符

### 文件判断

| 写法 | 含义 |
|------|------|
| `[ -e file ]` | 存在 |
| `[ -f file ]` | 是普通文件 |
| `[ -d dir ]` | 是目录 |
| `[ -r file ]` | 可读 |
| `[ -w file ]` | 可写 |
| `[ -x file ]` | 可执行 |

### 整数比较

| 写法 | 含义 |
|------|------|
| `[ $a -eq $b ]` | 等于 |
| `[ $a -ne $b ]` | 不等于 |
| `[ $a -gt $b ]` | 大于 |
| `[ $a -lt $b ]` | 小于 |
| `[ $a -ge $b ]` | 大于等于 |
| `[ $a -le $b ]` | 小于等于 |

### 字符串比较

| 写法 | 含义 |
|------|------|
| `[ "$a" = "$b" ]` | 相等 |
| `[ "$a" != "$b" ]` | 不等 |
| `[ -z "$a" ]` | 为空 |
| `[ -n "$a" ]` | 非空 |

> 变量加双引号 `"$a"` 防止空值导致语法错误。

### 逻辑运算

| 写法 | 含义 |
|------|------|
| `[ 条件1 -a 条件2 ]` | 与 |
| `[ 条件1 -o 条件2 ]` | 或 |
| `[ ! 条件 ]` | 非 |

```bash
if [ -f "$file" -a -r "$file" ]; then
    cat "$file"
fi
```

## 8. 默认变量

| 变量 | 含义 |
|------|------|
| `$0` | 脚本自身名字 |
| `$1` ~ `$9` | 第 1~9 个参数 |
| `${10}` | 第 10 个参数（两位数要加花括号） |
| `$#` | 参数个数 |
| `$@` | 所有参数（每个独立成词） |
| `$*` | 所有参数（合并成一个字符串） |
| `$?` | 上一条命令的退出码（0=成功） |
| `$$` | 当前脚本的进程 ID |
| `$!` | 最近后台进程的 PID |

```bash
#!/bin/bash
echo "脚本名: $0"
echo "第一个参数: $1"
echo "参数个数: $#"
echo "所有参数: $@"
echo "上一条命令退出码: $?"
```

运行：`./test.sh a b c` 打印各参数值。

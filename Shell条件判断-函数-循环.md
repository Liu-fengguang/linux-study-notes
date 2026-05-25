# Shell 条件判断、函数与循环

## 条件判断

### if 语句

```bash
if [ 条件 ]; then
    命令
elif [ 条件 ]; then
    命令
else
    命令
fi
```

`then` 可以换行写（省略 `;`）：

```bash
if [ 条件 ]
then
    命令
fi
```

### 整数比较

| 写法 | 含义 |
|------|------|
| `[ $a -eq $b ]` | 等于 |
| `[ $a -ne $b ]` | 不等于 |
| `[ $a -gt $b ]` | 大于 |
| `[ $a -lt $b ]` | 小于 |
| `[ $a -ge $b ]` | 大于等于 |
| `[ $a -le $b ]` | 小于等于 |

```bash
score=85
if [ $score -ge 90 ]; then
    echo "优秀"
elif [ $score -ge 60 ]; then
    echo "及格"
else
    echo "不及格"
fi
```

### 字符串比较

| 写法 | 含义 |
|------|------|
| `[ "$a" = "$b" ]` | 相等 |
| `[ "$a" != "$b" ]` | 不等 |
| `[ -z "$a" ]` | 为空 |
| `[ -n "$a" ]` | 非空 |

```bash
name=""
if [ -z "$name" ]; then
    echo "name 为空"
fi
```

### 文件判断

| 写法 | 含义 |
|------|------|
| `[ -e file ]` | 存在 |
| `[ -f file ]` | 是普通文件 |
| `[ -d dir ]` | 是目录 |
| `[ -r file ]` | 可读 |
| `[ -w file ]` | 可写 |
| `[ -x file ]` | 可执行 |
| `[ -s file ]` | 非空 |
| `[ file1 -nt file2 ]` | file1 比 file2 新 |
| `[ file1 -ot file2 ]` | file1 比 file2 旧 |

```bash
if [ -f "/etc/passwd" ]; then
    echo "passwd 文件存在"
fi
```

### 逻辑组合

| 写法一 | 写法二 | 含义 |
|--------|--------|------|
| `[ 条件1 -a 条件2 ]` | `[ 条件1 ] && [ 条件2 ]` | 与 |
| `[ 条件1 -o 条件2 ]` | `[ 条件1 ] \|\| [ 条件2 ]` | 或 |
| `[ ! 条件 ]` | | 非 |

```bash
if [ -f "$file" -a -r "$file" ]; then
    cat "$file"
fi
```

> 优先用 `&&` `||` 写法，POSIX 兼容性更好。

### case 多分支

```bash
case $变量 in
    模式1)
        命令 ;;
    模式2)
        命令 ;;
    *)
        命令 ;;    # 默认分支
esac
```

```bash
read -p "输入 start / stop / restart: " cmd
case $cmd in
    start)
        echo "启动服务" ;;
    stop)
        echo "停止服务" ;;
    restart)
        echo "重启服务" ;;
    *)
        echo "无效输入" ;;
esac
```

| 模式 | 含义 |
|------|------|
| `abc)` | 精确匹配 |
| `[a-z])` | 匹配一个字符（方括号） |
| `*.txt)` | 通配符匹配 |
| `start\|begin)` | 或，匹配 start 或 begin |

---

## 函数

### 定义与调用

```bash
# 定义
函数名() {
    命令
    return 0
}

# 调用（直接写函数名）
函数名
```

```bash
hello() {
    echo "hello, $1"
}
hello alice        # 输出: hello, alice
```

### 参数

| 变量 | 含义 |
|------|------|
| `$1` ~ `$9` | 函数第 1~9 个参数 |
| `$#` | 参数个数 |
| `$@` | 所有参数 |
| `$0` | 仍然是脚本名，不是函数名 |

```bash
add() {
    echo $(($1 + $2))
}
result=$(add 3 5)   # result = 8
```

### 返回值

| 方式 | 说明 |
|------|------|
| `return 0` | 返回退出码 0~255，0 成功，其他失败 |
| `echo` | 返回字符串，调用方用 `$(函数)` 捕获 |

```bash
check_file() {
    if [ -f "$1" ]; then
        return 0
    else
        return 1
    fi
}

check_file "/etc/passwd" && echo "存在"

get_sum() {
    echo $(($1 + $2))
}
total=$(get_sum 10 20)   # total = 30
```

> 函数用 `return` 返回状态码（0~255），用 `echo` 返回字符串。

### 局部变量

```bash
myfunc() {
    local name="alice"      # 仅函数内可见
    age=25                  # 全局可见
}
myfunc
echo $name      # 空
echo $age       # 25
```

---

## 循环

### for 循环

| 写法 | 场景 |
|------|------|
| `for i in 列表` | 遍历列表 |
| `for i in $(cmd)` | 遍历命令输出 |
| `for i in {1..10}` | 遍历数值范围 |
| `for ((i=0; i<10; i++))` | C 风格循环 |

```bash
# 遍历列表
for name in alice bob carol; do
    echo $name
done

# 遍历命令输出
for file in $(ls *.sh); do
    echo $file
done

# 数值范围
for i in {1..5}; do
    echo $i
done

# C 风格
for ((i=0; i<5; i++)); do
    echo $i
done
```

### while 循环

```bash
while [ 条件 ]
do
    命令
done
```

```bash
n=1
while [ $n -le 5 ]; do
    echo $n
    n=$((n + 1))
done
```

### until 循环

和 while 相反：条件为**假**时执行，为真时退出。

```bash
n=1
until [ $n -gt 5 ]; do
    echo $n
    n=$((n + 1))
done
```

### 循环控制

| 命令 | 作用 |
|------|------|
| `break` | 跳出整个循环 |
| `continue` | 跳过本次，进入下一次 |

```bash
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        break
    fi
    echo $i
done
# 输出 1 2 3 4
```

### 读取文件逐行

```bash
while IFS= read -r line; do
    echo $line
done < file.txt
```

---

## 综合示例

```bash
#!/bin/bash

is_odd() {
    if [ $(($1 % 2)) -eq 0 ]; then
        return 1
    else
        return 0
    fi
}

if [ $# -eq 0 ]; then
    echo "用法: $0 数字..."
    exit 1
fi

for num in "$@"; do
    if is_odd $num; then
        echo "$num 是奇数"
    else
        echo "$num 是偶数"
    fi
done
```

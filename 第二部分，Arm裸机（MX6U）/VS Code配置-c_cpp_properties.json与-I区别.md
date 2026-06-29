# VS Code 配置：c_cpp_properties.json

> 两个世界：Makefile 的 `-I` 给编译器用，VS Code 的配置给编辑器用。

---

## 一、为什么配置了 Makefile -I，VS Code 还是红色波浪线？

**两个独立的系统**，互不知对方的存在。

| | Makefile `-I` | VS Code `c_cpp_properties.json` |
|------|--------------|-------------------------------|
| 给谁用 | gcc 编译器（真实世界） | VS Code 编辑器（视觉世界） |
| 作用 | 决定代码能不能编译通过 | 决定有没有红色波浪线、Tab 能不能自动补全 |
| 配错后果 | `make` 失败 | 不影响编译，但写代码体验极差 |

> 配了 Makefile → `make` 能过。配了 VS Code → 编辑体验顺滑。**两个都得配。**

---

## 二、.vscode 目录是什么

VS Code 在工程根目录下创建的**隐藏文件夹**（`.` 开头），存放当前工程的私人设置，不影响其他工程。

```
LEDC/
├── .vscode/                     ← 隐藏，存放 VS Code 专属配置
│   └── c_cpp_properties.json
├── bsp/
├── main.c
└── Makefile
```

---

## 三、c_cpp_properties.json 是什么

VS Code 的 C/C++ 插件读取这个文件来知道去哪找头文件。相当于给 VS Code 的"监工"发寻宝地图。

```json
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**",
                "${workspaceFolder}/bsp/led",
                "${workspaceFolder}/bsp/clk",
                "${workspaceFolder}/imx6ull/inc"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/arm-linux-gnueabihf-gcc",
            "intelliSenseMode": "gcc-arm"
        }
    ]
}
```

| 字段 | 作用 |
|------|------|
| `includePath` | 头文件搜索路径（= Makefile 的 `-I`） |
| `compilerPath` | 编译器路径 |
| `intelliSenseMode` | 智能提示模式 |
| `defines` | 宏定义（= Makefile 的 `-D`） |

### `${workspaceFolder}/**` 的含义

| 写法 | 含义 |
|------|------|
| `${workspaceFolder}` | 当前工程根目录 |
| `/**` | 递归搜索所有子文件夹 |

> 最省事的写法：`"${workspaceFolder}/**"` 把整个工程翻个底朝天。

---

## 四、标准配置流程

新工程建好后，两步都做：

| 步骤 | 操作 | 目的 |
|------|------|------|
| ① | 修改 Makefile，加 `-I` 路径 | 保证 `make` 编译通过 |
| ② | 生成 `c_cpp_properties.json`，填 `includePath` | 消除红色波浪线，开启自动补全 |

### 生成方法

VS Code 里按 `Ctrl+Shift+P` → 搜索 `C/C++: Edit Configurations (JSON)` → 自动在 `.vscode/` 下生成 `c_cpp_properties.json` → 填 `includePath`。

---

## 五、配与不配的区别

| | 没配 VS Code | 配了 VS Code |
|------|------------|------------|
| `#include` 下面 | 红色波浪线 | 干净 |
| 敲函数名 | 无提示 | Tab 自动补全 |
| Ctrl+点击函数名 | 跳不过去 | 跳转到定义 |
| `make` 编译 | 能过（因为 Makefile 配了） | 一样能过 |

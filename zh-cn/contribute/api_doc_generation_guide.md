# 如何生成 API 参考文档

[ [English](../../en/contribute/api_doc_generation_guide.md) | 简体中文 ]

本文介绍如何在本地构建 openvela API 参考文档。该文档基于 Doxygen + Breathe + Sphinx 工具链，从源码头文件中的注释自动提取 API 信息，结合手写的 Markdown 页面生成完整的 HTML 文档站点。

## 工具链概览

文档生成流程分为两个阶段：

1. Doxygen 解析源码头文件（`.h`）中的注释，生成 XML 中间文件
2. Sphinx 通过 Breathe 插件读取 XML，结合 `docs/doxygen/api/` 下的 Markdown 页面，生成最终的 HTML 文档

```
源码头文件 (.h)
    |
    v
Doxygen --> XML 中间文件 (docs/doxygen/doxygen/xml/)
                |
                v
Markdown 页面 + Sphinx + Breathe --> HTML 文档站点
(docs/doxygen/api/)                  (docs/doxygen/_build/html/)
```

## 环境准备

### 系统要求

- 操作系统：Linux（推荐 Ubuntu 22.04）
- Python：3.8+

### 安装 Doxygen

```bash
# Ubuntu/Debian
sudo apt-get install doxygen

# 验证安装
doxygen --version
```

### 安装 Python 依赖

```bash
cd docs/doxygen
pip3 install -r requirements.txt
```

`requirements.txt` 中包含以下核心依赖：

| 工具             | 版本   | 说明                       |
| ---------------- | ------ | -------------------------- |
| Sphinx           | 4.4.0  | 文档生成引擎               |
| breathe          | 4.33.1 | Sphinx 的 Doxygen 桥接插件 |
| myst-parser      | 0.17.0 | Markdown 解析支持          |
| sphinx-rtd-theme | 1.0.0  | 文档主题                   |

## 构建文档

### 基本构建命令

```bash
cd docs/doxygen
make html type=openvela
```

`type` 参数控制构建范围：

| 参数值     | 说明                                                |
| ---------- | --------------------------------------------------- |
| `openvela` | 仅构建开源部分的 API 文档（使用 `Doxyfile.public`） |

构建成功后，HTML 文件输出到 `docs/doxygen/_build/html/` 目录。

### 预览文档

用浏览器打开生成的文档：

```bash
# 直接打开
xdg-open docs/doxygen/_build/html/index.html

# 或启动本地 HTTP 服务
cd docs/doxygen/_build/html
python3 -m http.server 8080
# 然后访问 http://localhost:8080
```

### 清理构建产物

```bash
cd docs/doxygen
make clean
```

## 目录结构说明

```
docs/doxygen/
├── Makefile                  # Sphinx 构建入口
├── conf.py                   # Sphinx 配置文件
├── Doxyfile.public           # Doxygen 配置（开源版本）
├── requirements.txt          # Python 依赖
├── index.md                  # 文档站点首页
├── api/                      # API 文档页面
│   ├── index.md              # API 总索引
│   ├── kernel/               # 内核接口
│   ├── network/              # 网络接口
│   ├── framework/            # 应用框架接口
│   │   ├── bluetooth/        # 蓝牙
│   │   ├── media/            # 多媒体
│   │   ├── telephony/        # 电话服务
│   │   ├── services/         # 系统服务 (AMS/PMS)
│   │   ├── feature/          # Feature Framework
│   │   ├── kvdb              # 键值存储
│   │   ├── security.md       # 安全框架
│   │   ├── uorb              # uORB 消息总线
│   │   └── ...
│   └── external/             # 第三方库接口
├── doxygen/xml/              # Doxygen 生成的 XML（构建产物）
└── _build/html/              # Sphinx 生成的 HTML（构建产物）
```

## 如何新增 API 文档模块

以新增一个名为 `mymodule` 的框架模块为例：

### 第一步：确认头文件路径

确保头文件已在 `Doxyfile.public` 的 `INPUT` 配置中覆盖。例如头文件位于 `frameworks/mymodule/include/mymodule.h`，需要添加：

```
INPUT = ... \
        ../../frameworks/mymodule/include
```

### 第二步：编写 Markdown 页面

在 `docs/doxygen/api/framework/` 下创建文档文件，例如 `mymodule.md`：

````markdown
# MyModule API 参考

MyModule 提供 xxx 能力，支持 xxx 功能。

## 接口说明

```eval_rst

.. doxygenfile:: mymodule.h
    :project: doxygen
```
````

> **说明**：上面的 `` ```eval_rst `` 和 `` ``` `` 是 MyST-Parser 的特殊代码围栏语法，用于在 Markdown 中嵌入 reStructuredText 指令。请确保使用三个反引号（`` ` ``）包裹，而非普通代码块。

其中 `doxygenfile` 指令会自动从 Doxygen XML 中提取该头文件的所有 API 文档。

### 第三步：注册到上层索引

在上层 `index.md` 的 `toctree` 中添加引用：

````markdown
```eval_rst

.. toctree::
    :maxdepth: 2

    existing_module
    mymodule
```
````

> **说明**：同样使用 `` ```eval_rst `` 围栏语法嵌入 Sphinx 的 `toctree` 指令。

### 第四步：构建验证

```bash
cd docs/doxygen
make clean && make html type=openvela
```

检查构建输出中是否有 `Cannot find file` 或其他警告。

## 头文件注释规范

Doxygen 从头文件注释中提取 API 信息。推荐使用以下注释风格：

```c
/**
 * @brief Create a new task.
 *
 * This function creates a new task with the specified parameters.
 *
 * @param name    Task name string.
 * @param priority Task priority (0-255).
 * @param stack_size Stack size in bytes.
 * @param entry   Task entry function.
 * @param arg     Argument passed to entry function.
 *
 * @return Task ID on success, negative errno on failure.
 *
 * @note The task name should not exceed CONFIG_TASK_NAME_SIZE characters.
 *
 * Example:
 * @code
 * int task_id = task_create("mytask", 100, 2048, my_entry, NULL);
 * if (task_id < 0) {
 *     printf("Failed to create task: %d\n", task_id);
 * }
 * @endcode
 */
int task_create(const char *name, int priority, int stack_size,
                main_t entry, char * const argv[]);
```

常用 Doxygen 标签：

| 标签                 | 说明         |
| -------------------- | ------------ |
| `@brief`             | 简要描述     |
| `@param`             | 参数说明     |
| `@return`            | 返回值说明   |
| `@note`              | 注意事项     |
| `@warning`           | 警告信息     |
| `@code` / `@endcode` | 代码示例     |
| `@see`               | 参考链接     |
| `@deprecated`        | 标记废弃接口 |

## 常见问题

### 构建报错 "Cannot find file xxx.h"

说明 `doxygenfile` 指令引用的头文件不在 `Doxyfile.public` 的 `INPUT` 路径覆盖范围内。解决方法：

1. 确认头文件在工作区中存在
2. 将头文件所在目录添加到 `Doxyfile.public` 的 `INPUT` 配置中
3. 如果头文件确实不存在（闭源模块），应移除对应的文档页面

### 构建警告 "doxygenfile: Cannot find file"

与上述问题相同，通常是 `INPUT` 路径未覆盖或头文件不存在。

### API 文档中出现 `FAR` 等未知宏

NuttX 使用 `FAR`、`CODE` 等宏标注指针类型。在 `Doxyfile.public` 的 `PREDEFINED` 中添加：

```
PREDEFINED = FAR= CODE=
```

### 中英文混杂问题

API 签名和参数说明来自头文件中的英文注释，模块概述来自手写的中文 Markdown 页面。这是业界通用做法（Android、Linux 内核等项目均采用此模式），保持 API 描述与源码一致，降低维护成本。

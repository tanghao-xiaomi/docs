# How to Generate API Reference Documentation

\[ English | [简体中文](./../../zh-cn/contribute/api_doc_generation_guide.md) \]

This guide describes how to build the openvela API reference documentation locally. The documentation is built on the Doxygen + Breathe + Sphinx toolchain, which automatically extracts API information from source header file comments and combines it with hand-written Markdown pages to generate a complete HTML documentation site.

## Toolchain Overview

The documentation generation process consists of two stages:

1. Doxygen parses comments in source header files (`.h`) and generates intermediate XML files
2. Sphinx reads the XML via the Breathe plugin, combines it with Markdown pages under `docs/doxygen/api/`, and generates the final HTML documentation

```
Source Header Files (.h)
    |
    v
Doxygen --> Intermediate XML (docs/doxygen/doxygen/xml/)
                |
                v
Markdown Pages + Sphinx + Breathe --> HTML Documentation Site
(docs/doxygen/api/)                   (docs/doxygen/_build/html/)
```

## Prerequisites

### System Requirements

- Operating System: Linux (Ubuntu 22.04 recommended)
- Python: 3.8+

### Install Doxygen

```bash
# Ubuntu/Debian
sudo apt-get install doxygen

# Verify installation
doxygen --version
```

### Install Python Dependencies

```bash
cd docs/doxygen
pip3 install -r requirements.txt
```

`requirements.txt` includes the following core dependencies:

| Tool             | Version | Description                      |
| ---------------- | ------- | -------------------------------- |
| Sphinx           | 4.4.0   | Documentation generation engine  |
| breathe          | 4.33.1  | Doxygen bridge plugin for Sphinx |
| myst-parser      | 0.17.0  | Markdown parsing support         |
| sphinx-rtd-theme | 1.0.0   | Documentation theme              |

## Building the Documentation

### Basic Build Command

```bash
cd docs/doxygen
make html type=openvela
```

The `type` parameter controls the build scope:

| Value      | Description                                                           |
| ---------- | --------------------------------------------------------------------- |
| `openvela` | Build only the open-source API documentation (uses `Doxyfile.public`) |

After a successful build, HTML files are output to the `docs/doxygen/_build/html/` directory.

### Preview the Documentation

Open the generated documentation in a browser:

```bash
# Open directly
xdg-open docs/doxygen/_build/html/index.html

# Or start a local HTTP server
cd docs/doxygen/_build/html
python3 -m http.server 8080
# Then visit http://localhost:8080
```

### Clean Build Artifacts

```bash
cd docs/doxygen
make clean
```

## Directory Structure

```
docs/doxygen/
├── Makefile                  # Sphinx build entry point
├── conf.py                   # Sphinx configuration file
├── Doxyfile.public           # Doxygen configuration (open-source version)
├── requirements.txt          # Python dependencies
├── index.md                  # Documentation site homepage
├── api/                      # API documentation pages
│   ├── index.md              # API master index
│   ├── kernel/               # Kernel interfaces
│   ├── network/              # Network interfaces
│   ├── framework/            # Application framework interfaces
│   │   ├── bluetooth/        # Bluetooth
│   │   ├── media/            # Multimedia
│   │   ├── telephony/        # Telephony services
│   │   ├── services/         # System services (AMS/PMS)
│   │   ├── feature/          # Feature Framework
│   │   ├── kvdb              # Key-value storage
│   │   ├── security.md       # Security framework
│   │   ├── uorb              # uORB message bus
│   │   └── ...
│   └── external/             # Third-party library interfaces
├── doxygen/xml/              # Doxygen-generated XML (build artifact)
└── _build/html/              # Sphinx-generated HTML (build artifact)
```

## How to Add a New API Documentation Module

The following example demonstrates adding a new framework module named `mymodule`:

### Step 1: Verify Header File Path

Ensure the header file is covered by the `INPUT` configuration in `Doxyfile.public`. For example, if the header file is located at `frameworks/mymodule/include/mymodule.h`, add:

```
INPUT = ... \
        ../../frameworks/mymodule/include
```

### Step 2: Write the Markdown Page

Create a documentation file under `docs/doxygen/api/framework/`, for example `mymodule.md`:

````markdown
# MyModule API Reference

MyModule provides xxx capabilities and supports xxx features.

## API Description

```eval_rst

.. doxygenfile:: mymodule.h
    :project: doxygen
```
````

> **Note**: The `` ```eval_rst `` and `` ``` `` above use MyST-Parser's special fenced code block syntax to embed reStructuredText directives within Markdown. Make sure to use three backticks (`` ` ``) as delimiters, not a regular code block.

The `doxygenfile` directive automatically extracts all API documentation from the Doxygen XML for the specified header file.

### Step 3: Register in the Parent Index

Add a reference in the parent `index.md`'s `toctree`:

````markdown
```eval_rst

.. toctree::
    :maxdepth: 2

    existing_module
    mymodule
```
````

> **Note**: This also uses the `` ```eval_rst `` fenced block syntax to embed Sphinx's `toctree` directive.

### Step 4: Build and Verify

```bash
cd docs/doxygen
make clean && make html type=openvela
```

Check the build output for `Cannot find file` or other warnings.

## Header File Comment Conventions

Doxygen extracts API information from header file comments. The following comment style is recommended:

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

Common Doxygen tags:

| Tag                  | Description              |
| -------------------- | ------------------------ |
| `@brief`             | Brief description        |
| `@param`             | Parameter description    |
| `@return`            | Return value description |
| `@note`              | Notes                    |
| `@warning`           | Warnings                 |
| `@code` / `@endcode` | Code examples            |
| `@see`               | See also references      |
| `@deprecated`        | Mark deprecated APIs     |

## FAQ

### Build Error: "Cannot find file xxx.h"

This means the header file referenced by the `doxygenfile` directive is not within the `INPUT` path coverage in `Doxyfile.public`. To resolve:

1. Verify the header file exists in the workspace
2. Add the header file's directory to the `INPUT` configuration in `Doxyfile.public`
3. If the header file does not exist (closed-source module), remove the corresponding documentation page

### Build Warning: "doxygenfile: Cannot find file"

Same as the issue above. This is typically caused by an uncovered `INPUT` path or a missing header file.

### Unknown Macros Like `FAR` in API Documentation

NuttX uses macros such as `FAR` and `CODE` to annotate pointer types. Add the following to the `PREDEFINED` section in `Doxyfile.public`:

```
PREDEFINED = FAR= CODE=
```

### Mixed Chinese and English Content

API signatures and parameter descriptions come from English comments in header files, while module overviews come from hand-written Chinese Markdown pages. This is a common industry practice (adopted by projects like Android and the Linux kernel) that keeps API descriptions consistent with source code and reduces maintenance costs.

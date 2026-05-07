# Keystore Client API

openvela Keystore 方案移植自 Android Keystore 服务框架，遵循 Keystore/Keymaster 标准接口。其中 Keymaster 层支持多种实现方式，包括对接 MiTee、纯软件实现以及为安全芯片（SE）定制的实现。

openvela Keystore 向上层以 Keystore C API 的方式，为账号 SDK 等应用场景提供密钥管理与安全存储能力，使用方无需关注底层的硬件差异和具体存储细节。

## 头文件

详细的 API 声明请参考源码中的头文件：`keystore/client.h`。

> **说明**：本模块 API 文档尚待手写整改。当前阶段请直接查阅头文件中的函数声明和注释。

\[ English | [简体中文](../../../../zh-cn/api/framework/feature/feature_framework.md) \]

# Feature Framework Overview

## Feature Framework Introduction

In QuickApp development, new capabilities need to be added to QuickApps, and these capabilities are written in C/C++. The Feature framework is a framework, SDK, and toolset that helps system developers extend functionality for QuickApps.

<img src="./figures/feature_framework.png" alt="Feature Framework Architecture Diagram" style="zoom: 80%;" />

## Feature Framework Capabilities

The Feature framework consists of a runtime framework, APIs, and the JIDL language and tools:

- Provides an execution framework for JS layer to call Native code
- The Feature framework API provides a set of interfaces for Native code to interact with JS
- JIDL is an interface description language used to automatically generate interfaces for mutual invocation between JS and Native

<img src="./figures/JIDL.png" alt="JIDL Interface Description Language Workflow" style="zoom:50%;" />

### Feature Concept Model

#### Static Concept Model

Since Features provide interfaces from Native to JS, the Feature concepts also follow JS conventions.

Features have 3 conceptual layers:

- Module: A Feature is a module, equivalent to a program module in C. It has no instances and exists globally.
- Prototype: Equivalent to a prototype object in JS, similar to a class in C++ but with differences. One QuickApp instance produces one Prototype. All functions and properties on a Feature are managed on the Prototype.
- Instance: One app contains multiple instances (each `require` produces an instance). Instances hold all processing data.

Specifically in QuickApps:

- A QuickApp instance has only one Prototype.
- A QuickApp page typically contains only one Feature Instance.

From a system perspective, the Feature concepts:

<img src="./figures/feature_static.png" alt="Feature Static Concept Model" style="zoom: 67%;" />

#### Runtime Concept Model

Each Feature can associate Native data, with different associated content. The runtime concept model is as follows:

<img src="./figures/feature_running.png" alt="Feature Runtime Concept Model" style="zoom: 67%;" />

#### Feature Lifecycle

<img src="./figures/feature_life.png" alt="Feature Lifecycle Diagram" style="zoom:80%;" />

### Feature Framework Interface Capabilities

#### Automatic Generation of Feature Prototype and Instance

The Feature framework helps developers create Feature Prototypes and Instances.

1. Feature developers need to provide a FeatureDescription that describes the Feature information, including:

    - Feature name
    - Feature member composition
        - Methods supported by the Feature, including method name, parameter list, return value, and implementation callback function
        - Property name, type, and implementation function
        - Others

2. Based on the FeatureDescription, FeaturePrototype and FeatureInstance are generated, and provided to developers as `FeatureProtoHandle` and `FeatureInstanceHandle`.

    <img src="./figures/feature_instance.png" alt="Feature Instance Creation Flow" style="zoom: 50%;" />

#### Parameter Conversion

From JS to Native, the Feature framework provides parameter conversion capabilities, converting JS parameters to plain parameters. The following table shows the basic conversion capabilities:

| JS Type             | C Type                   | Description                                                                                                                          |
| ------------------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------ |
| number/boolean      | int, float, double, bool | JS number types exist as floating point. Based on JIDL description, they can be converted to compatible types like int, float. Converting to int/bool causes loss of decimal part |
| string              | FtString                 | `const char*` typedef                                                                                                                |
| object              | struct pointer / FtAny pointer | If a struct is defined in JIDL, converts to the corresponding C struct pointer; if defined as object/any type in JIDL, defined as FtAny pointer |
| array               | FtArray pointer          | Converts to a C structure FtArray                                                                                                    |
| function            | FtCallbackId             | Converts to an integer representing CallbackId                                                                                       |
| promise             | FtPromiseId              | Converts to an integer representing PromiseId                                                                                        |

---

- Pointer objects have built-in reference counting and can be released via `FeatureDupValue` and `FeatureFreeValue`.
- Pointers passed through parameters do not need additional release.

The following diagram shows the management mechanism for Callbacks and Promises:

<img src="./figures/callback_promise_manager.png" alt="Callback and Promise Management Mechanism" style="zoom: 50%;" />

The Feature framework achieves two goals by hiding details:

- Feature developers do not need to care about details or manage the lifecycle of Callbacks and Promises.
- FeatureInstance provides fallback memory management methods.

#### Asynchronous Programming Model

Feature code and JS code run in the same uvloop. Feature developers need to be mindful of call duration. Blocking is not allowed in regular functions.

<img src="./figures/Asynchronous_model.png" alt="Asynchronous Programming Model Diagram" style="zoom:50%;" />

- A task can be added to the worker queue.
- Any thread can call `FeaturePost` to add a task to the main loop queue.

### JIDL Interface Description

JIDL is used to describe Feature interfaces. Below is a simple Feature file:

```C++
// Module name
module test@1.0

callback cb(int a, int b);

void foo(int a, float b, string c);

void goo(int a, cb cb1);

property string name;
property int age;
```

- Uses C++-style comments.
- Always starts with `module`, including module name and version.
- Can define properties, functions, interfaces, etc.

File naming conventions:

- File names end with `.jidl`.
- File names are typically `<feature name>_<version>.jidl`, but this is not mandatory.

Module naming allows the use of `.` separator, such as `system.fetch`.

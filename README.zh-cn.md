[Read this in English](./README.md)

这份文档旨在帮助维护者理解 `INIDictionary.ini` 的结构与语法。该文件是 **RA2 INI IntelliSense** 插件的核心数据库，定义了《红色警戒2》及其 Mod（Ares, Phobos 等）的所有配置规则、类型检查逻辑和自动补全数据。

---

# RA2/YR INI Dictionary 通用规范指南

## 项目介绍

这是一个通用的、专注于《红色警戒2》及其资料片《尤里的复仇》游戏配置（INI 文件）的模板字典库。

本项目系 **星轨工作室 (Starry Orbit Studio)** 为 **星轨妙妙屋** 旗下程序设计研发的一个统一、权威且易于读取和识别的 INI 配置模板。通过将复杂的 INI 结构映射为规范化的字典格式，它为所有依赖于解析 RA2/YR 配置文件的工具和程序提供了一个稳定、便捷的数据基础。

---

## 第一章：基础架构与注册语法

字典文件的核心由几个顶级的 **索引节 (Index Sections)** 组成。这些节的作用是声明“字典里包含了哪些类型”。

在以下索引节中，均支持三种灵活的注册语法：
1.  **`[Sections]`** (逻辑对象索引)
2.  **`[Globals]`** (全局节索引)
3.  **`[NumberLimits]`** (数值限制索引)
4.  **`[Limits]`** (字符串限制索引)
5.  **`[Lists]`** (列表结构索引)

### 通用注册语法
为了方便维护和追加定义，上述索引节支持以下三种条目书写方式：

1.  **标准索引式** (`Index=ID`)：
    ```ini
    0=InfantryType
    1=VehicleType
    ```
2.  **追加式** (`+=ID`)：
    *   推荐用于扩展字典，无需关心具体索引号。
    ```ini
    +=AircraftType
    ```
3.  **直接枚举式** (`ID`)：
    *   **推荐**。最简洁的方式，直接写名字，解析器会自动识别。
    ```ini
    BuildingType
    TechnoType
    ```

---

## 第二章：详细定义规范

本章将详细说明各类定义的内部语法。为了保持一致性，所有类型的描述均遵循统一格式。

### 2.1 逻辑对象 (Sections)

这是字典中占比最大的部分，用于定义游戏中可被实例化的对象类型（如具体的兵种、建筑、弹头等）及其可用的属性规范。

#### 2.1.1 继承机制
字典支持类似 C++ 的单继承语法。通过继承，子类将自动拥有父类定义的所有属性，无需重复编写。这极大地简化了字典的维护工作。

*   **语法**：`[SubTypeName]:[BaseTypeName]`
*   **原则**：尽量将通用属性（如 `Strength`, `Owner`）定义在顶层基类（如 `TechnoType`）中。

#### 2.1.2 属性定义规范 (Keys)
在类型节内部，每一行定义了一个允许出现的 `Key` 及其值的约束条件。

**完整语法：**
`KeyName = ValueType [, DefaultValue] [, FileCategory]`

> **注意**：参数之间使用英文逗号 `,` 分隔。如果中间某个参数为空（例如没有默认值但有文件过滤），逗号不能省略（如 `string,,art`）。

**参数详解：**

| 参数 | 必填 | 说明 | 示例 |
| :--- | :--- | :--- | :--- |
| **KeyName** | 是 | 属性名称 (不区分大小写)。 | `Strength`, `Primary` |
| **ValueType** | 是 | 值的类型。可以是基础类型、限制类型或注册表名。 | `int`, `bool`, `ColorStruct`, `InfantryTypes` |
| **DefaultValue** | 否 | 默认值。主要用于 IDE 悬停提示，帮助用户了解默认行为。 | `0`, `no`, `1.0` |
| **FileCategory** | 否 | **文件作用域过滤**。指定该属性只在特定的文件类型中生效。需配合插件的 `fileCategories` 设置使用。 | `art` (只在 art.ini 提示)<br>`rules` (只在 rules.ini 提示)<br>`ai` (只在 ai.ini 提示) |

#### 2.1.3 综合示例

```ini
; 1. 定义基类
[TechnoType]
; [基础类型] 整数，无默认值，所有文件通用
Strength=int
; [带默认值] 浮点数，默认为 1.0
Armor=float,1.0
; [注册表引用] 值必须是 Countries 列表中的 ID
Owner=Countries

; 2. 定义子类
[InfantryType]:[TechnoType]
; InfantryType 自动继承了 Strength, Armor, Owner

; [枚举类型] 值必须是 PipEnum 中定义的值
Pip=PipEnum,white
; [文件过滤] Cameo 属性只在 Art 文件中提示，且没有默认值 (注意中间的双逗号)
Cameo=string,,art
; [文件过滤] Category 属性只在 Rules 文件中提示
Category=Soldier,,rules
```

### 2.2 全局设置 (Globals)
定义游戏中全局唯一的配置节（如 `[General]`, `[CombatDamage]`），这些节不需要实例化，直接存在于文件中。

*   **索引节**：`[Globals]`
*   **定义语法**：直接列出该节下的属性。
*   **示例**：

    ```ini
    [Globals]
    General
    AudioVisual

    [General]
    WindDirection=int,0
    MinLowPowerProductionSpeed=float,.5
    ```

### 2.3 数值限制 (NumberLimits)
定义特定整数类型的取值范围。

*   **索引节**：`[NumberLimits]`
*   **定义标签**：
    *   `Range`：`最小值,最大值`
*   **示例**：

    ```ini
    [NumberLimits]
    int8
    uint8

    [int8]
    ; 允许负数
    Range=-128,127

    [uint8]
    Range=0,255
    ```

### 2.4 字符串限制 (Limits)
定义字符串的枚举、前缀或后缀要求。

*   **索引节**：`[Limits]`
*   **定义标签**：
    *   `LimitIn`：逗号分隔的枚举列表。
    *   `LimitIn.Ext`：**追加枚举**（推荐用于扩展字典）。
    *   `StartWith`：值必须以指定字符串开头。
    *   `EndWith`：值必须以指定字符串结尾。
    *   `CaseSensitive`：是否区分大小写（默认为 false）。
*   **示例**：

    ```ini
    [Limits]
    bool
    BuildCat

    [bool]
    StartWith=1,1,0,t,f,y,n

    [BuildCat]
    LimitIn=Combat,Infrastructure,Resource,Power,Tech,DontCare
    ```

### 2.5 列表结构 (Lists)
定义逗号分隔的数组结构（如颜色值、坐标点）。

*   **索引节**：`[Lists]`
*   **定义标签**：
    *   `Type`：列表中每个元素的类型。
    *   `Range`：`最小长度,最大长度`。
*   **示例**：

    ```ini
    [Lists]
    ColorStruct
    AnimList

    [ColorStruct]
    ; RGB颜色，必须是3个整数
    Type=uint8
    Range=3,3

    [AnimList]
    ; 包含任意多个动画类型的列表
    Type=AnimType
    ```

### 2.6 ID 注册表映射 (Registries)
定义游戏引擎中的 ID 列表与逻辑类型的映射关系。这告诉工具：“在 `[InfantryTypes]` 列表里注册的 ID，都属于 `InfantryType` 类型”。

*   **索引节**：`[Registries]`
*   **注意**：本节**不使用**第一章提到的三种通用注册语法，而是使用键值对映射。
*   **语法**：`RegistrySectionName = LogicTypeName`
*   **示例**：

    ```ini
    [Registries]
    ; 意味着 [InfantryTypes] 列表下的所有值（如 E1, SNIPE）都是 InfantryType
    InfantryTypes=InfantryType
    
    ; 意味着 [Countries] 列表下的所有值都是 Country
    Countries=Country
    ```

---

## 第三章：模块化与扩展

为了支持多平台（原版、Ares、Phobos）并保持字典整洁，我们支持 `#include` 语法。

### 3.1 包含语法
创建一个名为 `[#include]` 的特殊节。解析器会递归读取这些文件，并将内容合并到主字典中。

```ini
[#include]
; 同样支持第一章的灵活语法，但通常直接写文件名或使用索引
1=Standard.ini
2=Ares.ini
3=Phobos.ini
```

### 3.2 来源追踪与覆盖
*   **来源追踪**：工具应当记录每个属性是从哪个文件读取的（例如 `Phobos.ini`），以便向用户展示该属性的来源平台。
*   **覆盖规则**：后加载的定义会覆盖先加载的定义（例如 Ares 字典可以修改原版字典中某个属性的类型）。
*   **追加规则**：对于 `LimitIn` 等支持扩展的标签，子字典的内容会被追加到列表中，而不是覆盖。

---

## 维护最佳实践

1.  **保持原子性**：如果一个属性只在 Ares 平台可用，请将其定义在 `Ares.ini` 中，并使用 `#include` 引入，而不要直接修改 `INIDictionary.ini`。
2.  **继承优先**：尽量将通用属性放在基类（如 `TechnoType`, `AbstractType`）中，避免在每个子类重复定义。
3.  **文件分类**：利用 `FileCategory` 参数区分逻辑属性（`rules`）和美术属性（`art`），这能极大减少干扰项。
4.  **命名规范**：类型名建议使用 **PascalCase**（如 `InfantryType`），属性名保持与官方 INI 一致。
5.  **注释**：对于复杂的逻辑或不确定的属性，请使用 `;` 添加注释。

---

## 贡献与反馈

欢迎所有对 RA2/YR INI 结构有深刻理解的开发者和爱好者加入贡献：

*   **提交 Issue**: 报告标签错误、缺失的官方标签，或提出新的扩展平台支持建议。
*   **提交 Pull Request**: 贡献缺失的官方标签定义，或实现 Ares/Phobos 等扩展平台的标签支持。

让我们共同维护和完善这个通用的 INI 字典，为整个《红色警戒 2/尤里的复仇》工具生态提供一个坚实的基础！

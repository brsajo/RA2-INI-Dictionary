[Read this in Chinese (简体中文)](./README.zh-cn.md)

This document aims to help maintainers understand the structure and syntax of `INIDictionary.ini`. This file acts as the core database for the **RA2 INI IntelliSense** extension, defining all configuration rules, type checking logic, and auto-completion data for *Command & Conquer: Red Alert 2* and its Mods (Ares, Phobos, etc.).

---

# RA2/YR INI Dictionary General Specification Guide

## Project Introduction

This is a universal template dictionary library focused on game configuration files (INI files) for *Command & Conquer: Red Alert 2* and its expansion *Yuri's Revenge*.

Developed by **Starry Orbit Studio** (under **Starry Orbit Wonder House**) for program design, this project provides a unified, authoritative, and easily machine-readable INI configuration template. By mapping complex INI structures into a standardized dictionary format, it offers a stable and convenient data foundation for all tools and programs that rely on parsing RA2/YR configuration files.

---

## Chapter 1: Infrastructure & Registration Syntax

The core of the dictionary consists of several top-level **Index Sections**. The purpose of these sections is to declare "what types are contained in the dictionary."

The following index sections support three flexible registration syntaxes:
1.  **`[Sections]`** (Logical Object Index)
2.  **`[Globals]`** (Global Section Index)
3.  **`[NumberLimits]`** (Numeric Limit Index)
4.  **`[Limits]`** (String Limit Index)
5.  **`[Lists]`** (List Structure Index)

### General Registration Syntax
To facilitate maintenance and appending definitions, the above index sections support the following three entry styles:

1.  **Standard Index Style** (`Index=ID`):
    ```ini
    0=InfantryType
    1=VehicleType
    ```
2.  **Append Style** (`+=ID`):
    *   Recommended for extension dictionaries, as there is no need to worry about specific index numbers.
    ```ini
    +=AircraftType
    ```
3.  **Direct Enumeration Style** (`ID`):
    *   **Recommended**. The most concise method; simply write the name, and the parser will identify it automatically.
    ```ini
    BuildingType
    TechnoType
    ```

---

## Chapter 2: Detailed Definition Specifications

This chapter details the internal syntax for various definitions. To maintain consistency, descriptions of all types follow a unified format.

### 2.1 Logical Objects (Sections)

This constitutes the largest part of the dictionary, used to define object types that can be instantiated in the game (such as specific units, buildings, warheads, etc.) and their available property specifications.

#### 2.1.1 Inheritance Mechanism
The dictionary supports single inheritance syntax similar to C++. Through inheritance, a subclass automatically possesses all properties defined in the parent class, eliminating the need for repetitive writing. This greatly simplifies dictionary maintenance.

*   **Syntax**: `[SubTypeName]:[BaseTypeName]`
*   **Principle**: Try to define common properties (like `Strength`, `Owner`) in the top-level base class (like `TechnoType`).

#### 2.1.2 Property Definition Specifications (Keys)
Inside a type section, each line defines an allowed `Key` and constraints on its value.

**Full Syntax:**
`KeyName = ValueType [, DefaultValue] [, FileCategory]`

> **Note**: Parameters are separated by commas `,`. If a parameter in the middle is empty (e.g., no default value but has a file filter), the comma cannot be omitted (e.g., `string,,art`).

**Parameter Details:**

| Parameter | Required | Description | Example |
| :--- | :--- | :--- | :--- |
| **KeyName** | Yes | The name of the property (case-insensitive). | `Strength`, `Primary` |
| **ValueType** | Yes | The data type of the value. Can be a primitive type, limit type, or registry name. | `int`, `bool`, `ColorStruct`, `InfantryTypes` |
| **DefaultValue** | No | The default value. Mainly used for IDE hover tooltips to help users understand default behavior. | `0`, `no`, `1.0` |
| **FileCategory** | No | **File Scope Filter**. Specifies that this property only takes effect in specific file types. Must be used in conjunction with the extension's `fileCategories` setting. | `art` (only prompts in art.ini)<br>`rules` (only prompts in rules.ini)<br>`ai` (only prompts in ai.ini) |

#### 2.1.3 Comprehensive Example

```ini
; 1. Define Base Class
[TechnoType]
; [Primitive Type] Integer, no default value, valid in all files
Strength=int
; [With Default] Float, default is 1.0
Armor=float,1.0
; [Registry Reference] Value must be an ID from the Countries list
Owner=Countries

; 2. Define Subclass
[InfantryType]:[TechnoType]
; InfantryType automatically inherits Strength, Armor, and Owner

; [Enum Type] Value must be one of the values defined in PipEnum
Pip=PipEnum,white
; [File Filter] Cameo property only prompts in Art files, no default value (note the double comma)
Cameo=string,,art
; [File Filter] Category property only prompts in Rules files
Category=Soldier,,rules
```

### 2.2 Global Settings (Globals)
Defines configuration sections that are globally unique in the game (such as `[General]`, `[CombatDamage]`). These sections do not need instantiation and exist directly in the file.

*   **Index Section**: `[Globals]`
*   **Definition Syntax**: List the properties under the section directly.
*   **Example**:

    ```ini
    [Globals]
    General
    AudioVisual

    [General]
    WindDirection=int,0
    MinLowPowerProductionSpeed=float,.5
    ```

### 2.3 Numeric Limits (NumberLimits)
Defines the value range for specific integer types.

*   **Index Section**: `[NumberLimits]`
*   **Definition Tag**:
    *   `Range`: `MinValue,MaxValue`
*   **Example**:

    ```ini
    [NumberLimits]
    int8
    uint8

    [int8]
    ; Negative numbers allowed
    Range=-128,127

    [uint8]
    Range=0,255
    ```

### 2.4 String Limits (Limits)
Defines enumeration, prefix, or suffix requirements for strings.

*   **Index Section**: `[Limits]`
*   **Definition Tags**:
    *   `LimitIn`: Comma-separated list of enumerations.
    *   `LimitIn.Ext`: **Append Enumeration** (Recommended for extension dictionaries).
    *   `StartWith`: Value must start with the specified string.
    *   `EndWith`: Value must end with the specified string.
    *   `CaseSensitive`: Whether it is case-sensitive (default is false).
*   **Example**:

    ```ini
    [Limits]
    bool
    BuildCat

    [bool]
    StartWith=1,1,0,t,f,y,n

    [BuildCat]
    LimitIn=Combat,Infrastructure,Resource,Power,Tech,DontCare
    ```

### 2.5 List Structures (Lists)
Defines comma-separated array structures (such as color values, coordinates).

*   **Index Section**: `[Lists]`
*   **Definition Tags**:
    *   `Type`: The type of each element in the list.
    *   `Range`: `MinLength,MaxLength`.
*   **Example**:

    ```ini
    [Lists]
    ColorStruct
    AnimList

    [ColorStruct]
    ; RGB color, must be 3 integers
    Type=uint8
    Range=3,3

    [AnimList]
    ; List containing any number of animation types
    Type=AnimType
    ```

### 2.6 ID Registry Mapping (Registries)
Defines the mapping relationship between ID lists in the game engine and logical types. This tells the tool: "All IDs registered in the `[InfantryTypes]` list belong to the `InfantryType` type."

*   **Index Section**: `[Registries]`
*   **Note**: This section **does not use** the three general registration syntaxes mentioned in Chapter 1; instead, it uses key-value mapping.
*   **Syntax**: `RegistrySectionName = LogicTypeName`
*   **Example**:

    ```ini
    [Registries]
    ; Means all values under the [InfantryTypes] list (e.g., E1, SNIPE) are InfantryType
    InfantryTypes=InfantryType
    
    ; Means all values under the [Countries] list are Country
    Countries=Country
    ```

---

## Chapter 3: Modularity & Extension

To support multiple platforms (Vanilla, Ares, Phobos) and keep the dictionary clean, we support `#include` syntax.

### 3.1 Include Syntax
Create a special section named `[#include]`. The parser will recursively read these files and merge the content into the main dictionary.

```ini
[#include]
; Also supports the flexible syntax from Chapter 1, but typically direct filenames or indexes are used
1=Standard.ini
2=Ares.ini
3=Phobos.ini
```

### 3.2 Source Tracking & Overriding
*   **Source Tracking**: The tool should record which file each property was read from (e.g., `Phobos.ini`) to display the source platform of the property to the user.
*   **Overriding Rules**: Definitions loaded later will override definitions loaded earlier (e.g., the Ares dictionary can modify the type of a property in the vanilla dictionary).
*   **Appending Rules**: For tags that support extension like `LimitIn`, the content of the sub-dictionary will be appended to the list rather than overwriting it.

---

## Maintenance Best Practices

1.  **Atomicity**: If a property is only available on the Ares platform, define it in `Ares.ini` and include it via `#include`, rather than modifying `INIDictionary.ini` directly.
2.  **Inheritance First**: Try to place general properties in base classes (like `TechnoType`, `AbstractType`) to avoid repetitive definitions in every subclass.
3.  **File Categorization**: Use the `FileCategory` parameter to distinguish between logic properties (`rules`) and art properties (`art`), which significantly reduces distractions.
4.  **Naming Conventions**: Type names are recommended to use **PascalCase** (e.g., `InfantryType`), and property names should remain consistent with the official INI.
5.  **Comments**: Use `;` to add comments for complex logic or uncertain properties.

---

## Contribution & Feedback

We welcome all developers and enthusiasts with a deep understanding of the RA2/YR INI structure to contribute:

*   **Submit Issue**: Report tag errors, missing official tags, or suggest support for new extension platforms.
*   **Submit Pull Request**: Contribute missing official tag definitions or implement tag support for extension platforms like Ares/Phobos.

Let's work together to maintain and improve this universal INI dictionary, providing a solid foundation for the entire *Red Alert 2 / Yuri's Revenge* tool ecosystem!

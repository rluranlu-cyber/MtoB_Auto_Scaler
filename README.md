# Auto_Scaler

> **Blender 5.0.1 兼容插件** | 为导入模型快速创建参考空物体并完成英寸→米单位换算
>
> **Blender 5.0.1 Compatible Add-on** | Quickly create a reference empty for imported models and perform inches→meters unit conversion

[![Version](https://img.shields.io/badge/version-1.4.0-blue)](https://github.com/) [![Blender](https://img.shields.io/badge/Blender-5.0.1-orange)](https://www.blender.org/) [![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

---

## 📖 目录 | Table of Contents

- [中文版](#中文版)
  - [简介](#简介)
  - [功能概览](#功能概览)
  - [安装方法](#安装方法)
  - [使用步骤](#使用步骤)
  - [参数说明](#参数说明)
  - [工作原理](#工作原理)
  - [语言适配](#语言适配)
  - [常见问题](#常见问题)
- [English Version](#english-version)
  - [Overview](#overview)
  - [Features](#features)
  - [Installation](#installation)
  - [Usage](#usage)
  - [Parameters](#parameters)
  - [How It Works](#how-it-works)
  - [Language Adaptation](#language-adaptation)
  - [FAQ](#faq)

---

# 中文版

## 简介

**Auto_Scaler** 是一款适配 Blender 5.0.1 的轻量级插件，专为 3D 动画工作流设计。它解决了导入模型（特别是来自3ds Max等以英寸为默认单位的软件）后常见的两个痛点：

1. **单位换算**：导入的**英寸**单位模型需要转换为**米制**
2. **层级整理**：导入文件自带的空物体层级需要清理或重组（原3ds max里模型不干净的情况）

插件提供两个独立按钮，按需使用，互不干扰。UI 文本会根据 Blender 界面语言自动切换（简体中文 / English）。

## 功能概览

### 按钮 A：解除父级并删除空物体

- ✅ 对选中物体执行 **Alt+P → 清除父级并保留变换**
- ✅ 删除选中集合中的所有空物体
- ✅ 保留 Mesh / Armature / Curve 等几何体（不破坏蒙皮）

### 按钮 B：创建空物体并父级化

- ✅ 计算选中物体合并包围盒的底部中心点
- ✅ 在底部中心创建**空物体（Empty）**
- ✅ 识别根物体，以 **保留变换** 方式父级化到新建的空物体
- ✅ 保留导入文件原有父子层级
- ✅ 新空物体的 **变换** 和 **缩放** xyz轴都乘以 **0.0254（英寸→米）**

## 安装方法

1. 打开 Blender 5.0.1
2. 进入 `编辑 > 偏好设置 > 插件`
3. 点击右上角下拉菜单 → `从磁盘安装`
4. 选择 `auto_scaler.py` 文件
5. 在插件列表中勾选启用 **Auto_Scaler**

**位置**：3D Viewport 侧栏（按 `N` 键）> `工具` 标签 > `Auto Scaler` 面板

## 使用步骤

### 场景一：清理导入层级（假如导入的模型带有父级且你的项目要求网格体上无多余层级的情况）

```
1. 导入模型（OBJ / ABC / glTF 等）
2. 选中需要清理的物体（比如含父级空物体的网格体）
3. 点击「解除父级并删除 Empty」按钮
4. 完成：所有父级关系被解除，空物体被删除，仅保留几何体
```

### 场景二：单位换算 + 创建参考空物体

```
1. 导入模型（导入后默认自动选中相关物体）
2. 保持选中状态不变
3. 点击「创建 Cube 形态空物体并父级化」按钮
4. 完成：模型底部中心出现 Cube 形态空物体，整体缩放为原 2.54%
```

### 场景三：组合使用

```
1. 导入模型
2. 用按钮 A 清理多余层级
3. 重新选中需要的网格体
4. 用按钮 B 创建参考空物体并完成单位换算
```

## 参数说明

按钮 B 支持以下可调参数（按 `F6` 或在 Redo Panel 中修改）：

| 参数 | 默认值 | 说明 |
|---|---|---|
| **换算系数** | `0.0254` | 乘以空物体位移和缩放的系数（英寸→米） |
| **空物体名称** | `Empty_Parent` | 新建空物体的名字 |
| **空物体显示样式** | `CUBE` | 视口显示形态（Cube / Arrows / Plain Axes 等） |
| **空物体显示尺寸** | `0.5` | 视口中线框的大小 |

## 工作原理

### 按钮 A 的工作流程

```
选中物体 → 筛选有父级的物体 → Alt+P (Clear and Keep Transform)
       → 删除选中集合中所有 Empty → 恢复剩余物体选中状态
```

**关键点**：
- Alt+P 解除所有父级关系（不管父级类型，完全打散层级）
- 仅删除 **空物体**，保留 Armature / Curve 等几何体（避免破坏蒙皮）
- **清除并保留变换** 保证解除后世界变换不变

### 按钮 B 的工作流程

```
选中物体 → 收集所有顶点世界坐标 → 计算合并包围盒底部中心
       → 创建 Cube 形态空物体 → 识别根物体
       → Ctrl+P (Keep Transform) 父级化 → 三轴 × 0.0254
```

**关键点**：
- **底部中心** = (X中心, Y中心, Z最小值)
- **根物体** = 父级为 None 或父级不在选中集合中的物体
- 只父级化根物体，保留导入文件原有父子层级
- Cube 是 Empty 的显示形态（`empty_display_type='CUBE'`），不是真正的 Mesh Cube
- 父级化后修改父级变换，子物体世界变换自动联动：新世界位置 = 0.0254 × 原位置

### 包围盒计算

| 物体类型 | 计算方式 | 精度 |
|---|---|---|
| **Mesh** | 直接读取顶点世界坐标 | 最高 |
| **Armature / Curve / Surface / Meta / Lattice / Font** | `bound_box` 8 角点经 `matrix_world` 变换 | 良好 |
| **Empty** | 不参与计算 | — |

## 语言适配

插件仅支持 **简体中文 (zh_CN)** 和 **英文 (en_US)** 两种界面语言，会根据 Blender 的界面语言设置自动切换：

- **实时跟随**：Panel 内的按钮文案、使用步骤、状态提示、report 消息
- **需重启插件**：Property 的 name/description（如"换算系数"等，在 F6 面板中显示）

**回退规则**：
- 繁体中文 (zh_TW) → 回退到简体中文 (zh_CN)
- 其他未支持语言 → 回退到英文 (en_US)

切换 Blender 语言后，如需让 Property 文本也同步切换，请在 Add-ons 管理器中先禁用再启用本插件。

## 常见问题

<details>
<summary><b>Q: 创建的空物体是真正的 Cube 吗？</b></summary>

**A: 不是。** 创建的是 Empty 对象（`obj.type == 'EMPTY'`），只是显示形态设为 Cube 线框（`empty_display_type='CUBE'`）。它没有 mesh data，不参与渲染，仅在视口中显示为立方体线框作为参考。
</details>

<details>
<summary><b>Q: 为什么按钮 B 后模型变小了很多？</b></summary>

**A: 这是预期行为。** 0.0254 是英寸→米的换算系数。如果你的原始模型是英寸单位（如来自 3ds Max），按钮 B 会把整体缩放为原 2.54%，等效于把单位从英寸转换为米。如果你的模型本来就是米单位，请将"换算系数"改为 `1.0`。
</details>

<details>
<summary><b>Q: 按钮 A 会破坏骨骼蒙皮吗？</b></summary>

**A: 不会。** 按钮 A 仅删除 Empty，保留 Armature 和 Mesh。Armature Modifier 的引用是属性级，不依赖父子层级，所以解除父子关系不会破坏顶点权重变形。
</details>

<details>
<summary><b>Q: 导入文件自带的父子层级会被保留吗？</b></summary>

**A: 会。** 按钮 B 只父级化"根物体"（父级为 None 或不在选中集合中的物体），保留导入文件原有的父子层级。例如导入 `Empty_A → Mesh_1` 层级，只会把 Empty_A 挂到新 Cube 下，Mesh_1 仍保持为 Empty_A 的子级。
</details>

<details>
<summary><b>Q: 支持哪些 Blender 版本？</b></summary>

**A: 适配 Blender 5.0.1。** 作为传统 legacy add-on，通过 `Install from Disk` 安装。如需 4.2+ 的 Extension 形式（带 `manifest.toml`），可另行改造。
</details>

---

# English Version

## Overview

**Auto_Scaler** is a lightweight add-on for Blender 5.0.1, designed for 3D animation workflows. It addresses two common pain points when importing models (especially from software like 3ds Max that originally use **inches**):

1. **Unit conversion**: Imported inch-unit models need to be converted to meters
2. **Hierarchy cleanup**: Empty objects from imported files need to be cleaned up or reorganized

The add-on provides two independent buttons that can be used separately. UI text automatically switches between Simplified Chinese and English based on Blender's interface language.

## Features

### Button A: Clear Parents & Delete Empties

- ✅ Executes **Alt+P → Clear and Keep Transformation** on selected objects
- ✅ Deletes all **Empties** in the selection
- ✅ Preserves Mesh / Armature / Curve and other geometry (won't break skinning)

### Button B: Create Cube Empty & Parent

- ✅ Computes the bottom-center of the combined bounding box of selected objects
- ✅ Creates a Cube-shaped Empty at the bottom-center
- ✅ Identifies root objects and parents them to the new Empty with Keep Transform
- ✅ Preserves the original hierarchy from the imported file
- ✅ Multiplies the new Empty's location and scale (all axes) by 0.0254 (inches→meters)

## Installation

1. Open Blender 5.0.1
2. Go to `Edit > Preferences > Add-ons`
3. Click the dropdown menu in the top-right → `Install from Disk`
4. Select the `auto_scaler.py` file
5. Enable **Auto_Scaler** by checking the box in the add-ons list

**Location**: 3D Viewport Sidebar (press `N`) > `Tool` tab > `Auto Scaler` panel

## Usage

### Scenario 1: Clean Up Imported Hierarchy

```
1. Import a model (OBJ / FBX / glTF, etc.)
2. Select the objects to clean up (including Empties and meshes)
3. Click the "Clear Parents & Delete Empties" button
4. Done: All parent relationships are cleared, Empties are deleted, only geometry remains
```

### Scenario 2: Unit Conversion + Create Reference Empty

```
1. Import a model (relevant objects are auto-selected after import)
2. Keep the selection as-is
3. Click the "Create Cube Empty & Parent" button
4. Done: A Cube-shaped Empty appears at the bottom-center, overall scale becomes 2.54% of original
```

### Scenario 3: Combined Workflow (Recommended)

```
1. Import the model
2. Use Button A to clean up unnecessary hierarchy
3. Re-select the needed meshes
4. Use Button B to create a reference Empty and complete unit conversion
```

## Parameters

Button B supports the following adjustable parameters (press `F6` or modify in the Redo Panel):

| Parameter | Default | Description |
|---|---|---|
| **Factor** | `0.0254` | Multiplier for the empty's location and scale (inches→meters) |
| **Empty Name** | `Empty_Parent` | Name of the new empty object |
| **Empty Display Type** | `CUBE` | Viewport display form (Cube / Arrows / Plain Axes, etc.) |
| **Empty Display Size** | `0.5` | Size of the wireframe in the viewport |

## How It Works

### Button A Workflow

```
Selected objects → Filter objects with parents → Alt+P (Clear and Keep Transform)
                → Delete all Empties in selection → Restore remaining selection
```

**Key points**:
- Alt+P clears all parent relationships (regardless of parent type, fully breaks hierarchy)
- Only deletes Empties, preserves Armature / Curve and other geometry (won't break skinning)
- Clear and Keep Transformation ensures world transform remains unchanged after clearing

### Button B Workflow

```
Selected objects → Collect all vertex world coordinates → Compute combined bbox bottom-center
                → Create Cube-shaped Empty → Identify root objects
                → Ctrl+P (Keep Transform) parent → All axes × 0.0254
```

**Key points**:
- **Bottom-center** = (X-center, Y-center, Z-min)
- **Root objects** = objects with parent=None or parent not in selection
- Only roots are parented, preserving the original hierarchy from the imported file
- Cube is the Empty's display form (`empty_display_type='CUBE'`), not a real Mesh Cube
- After parenting, modifying the parent's transform automatically propagates to children: new world position = 0.0254 × original position

### Bounding Box Calculation

| Object Type | Method | Accuracy |
|---|---|---|
| **Mesh** | Directly reads vertex world coordinates | Highest |
| **Armature / Curve / Surface / Meta / Lattice / Font** | `bound_box` 8 corners transformed by `matrix_world` | Good |
| **Empty** | Not参与计算 | — |

## Language Adaptation

The add-on supports both **Simplified Chinese (zh_CN)** and **English (en_US)** interface languages, switching automatically based on Blender's interface language setting:

- **Real-time follow**: Panel button text, usage steps, status hints, report messages
- **Requires plugin restart**: Property name/description (e.g., "Factor", shown in F6 panel)

**Fallback rules**:
- Traditional Chinese (zh_TW) → falls back to Simplified Chinese (zh_CN)
- Other unsupported languages → falls back to English (en_US)

After switching Blender's language, if you want Property text to sync as well, disable and re-enable this add-on in the Add-ons manager.

## FAQ

<details>
<summary><b>Q: Is the created empty a real Cube?</b></summary>

**A: No.** What's created is an Empty object (`obj.type == 'EMPTY'`) with its display form set to Cube wireframe (`empty_display_type='CUBE'`). It has no mesh data, doesn't participate in rendering, and only appears as a cube wireframe in the viewport as a reference.
</details>

<details>
<summary><b>Q: Why does the model become much smaller after Button B?</b></summary>

**A: This is expected behavior.** 0.0254 is the inches→meters conversion factor. If your original model is in inches (e.g., from 3ds Max), Button B will scale the whole thing to 2.54% of the original, equivalent to converting the unit from inches to meters. If your model is already in meters, change the "Factor" to `1.0`.
</details>

<details>
<summary><b>Q: Will Button A break skeletal skinning?</b></summary>

**A: No.** Button A only deletes Empties, preserving Armatures and Meshes. The Armature Modifier reference is property-level and doesn't depend on parent-child hierarchy, so clearing parent relationships won't break vertex weight deformation.
</details>

<details>
<summary><b>Q: Will the original hierarchy from the imported file be preserved?</b></summary>

**A: Yes.** Button B only parents "root objects" (objects with parent=None or parent not in the selection), preserving the original hierarchy from the imported file. For example, if you import an `Empty_A → Mesh_1` hierarchy, only Empty_A is attached under the new Cube, and Mesh_1 remains a child of Empty_A.
</details>

<details>
<summary><b>Q: Which Blender versions are supported?</b></summary>

**A: Adapted for Blender 5.0.1.** Installed as a traditional legacy add-on via `Install from Disk`. For the 4.2+ Extension format (with `manifest.toml`), separate conversion is needed.
</details>

---

## 📋 技术规格 | Technical Specifications

| 项 | 值 |
|---|---|
| **版本 | Version** | 1.4.0 |
| **Blender 版本 | Blender Version** | 5.0.1 |
| **类别 | Category** | Object |
| **位置 | Location** | View3D > Sidebar (N) > Tool > Auto Scaler |
| **支持语言 | Supported Languages** | 简体中文 (zh_CN), English (en_US) |
| **许可证 | License** | MIT |

## 📝 版本历史 | Version History

| 版本 | 日期 | 主要变更 |
|---|---|---|
| **v1.4.0** | 2026-06-22 | 新增 i18n 自动语言适配 (zh_CN / en_US) |
| **v1.3.0** | 2026-06-22 | 新增按钮 A (解除父级并删除 Empty)；永久封装版本 |
| **v1.2.1** | 2026-06-22 | 措辞澄清：明确"Cube 形态空物体"非 Mesh Cube |
| **v1.2.0** | 2026-06-22 | 空物体改为 Cube 显示；识别根物体保留导入层级 |
| **v1.1.0** | 2026-06-22 | 扩展支持 Armature/Curve 等非网格组件 |
| **v1.0.0** | 2026-06-22 | 初始版本：基本功能实现 |

| Version | Date | Major Changes |
|---|---|---|
| **v1.4.0** | 2026-06-22 | Added i18n auto language adaptation (zh_CN / en_US) |
| **v1.3.0** | 2026-06-22 | Added Button A (Clear parents & delete Empties); permanently archived |
| **v1.2.1** | 2026-06-22 | Wording clarification: "Cube-shaped Empty" is not a Mesh Cube |
| **v1.2.0** | 2026-06-22 | Changed empty display to Cube; identify roots to preserve imported hierarchy |
| **v1.1.0** | 2026-06-22 | Extended support for Armature/Curve and other non-mesh components |
| **v1.0.0** | 2026-06-22 | Initial version: basic functionality |

## 📄 许可证 | License

MIT License - 详见 [LICENSE](LICENSE) 文件。

MIT License - See the [LICENSE](LICENSE) file for details.

---

<p align="center">
  <sub>Built with ❤️ by WorkBuddy · Powered by Blender Python API</sub>
</p>

---
tags: [source, opencv, contour, hierarchy, note]
created: 2026-07-03
updated: 2026-07-03
status: budding
domain: robot-learning
source_file: Robot_Learning/raw/OpenCV 轮廓提取与层级拓扑完全指南.md
source_type: note
source_author: unknown
ingest_date: 2026-07-03
---

# OpenCV 轮廓提取与层级拓扑完全指南 源

> **原文**：[[Robot_Learning/raw/OpenCV 轮廓提取与层级拓扑完全指南.md]]
> **摄入日期**：2026-07-03
> **源类型**：技术笔记（对上一份 OpenCV 课程笔记的专项补充）

## 摘要

这是一篇对 `cv::findContours` 和 `hierarchy` 的深度技术解析，约 1200 行。与 [[Robot_Learning/sources/4h上手C++版Opencv 源|第一份 OpenCV 笔记]] 不同的是，上一份覆盖了 OpenCV 的广度（从图像 I/O 到人脸检测），这一份聚焦于**轮廓提取这一条线的深度**——从数据结构的内存布局到层级拓扑的父子遍历，再到四个完整的工业实战项目。

## 详细提取

### 1. 轮廓在视觉流水线中的定位

> `findContours` 不是一个"检测算法"，它是"像素边界 → 几何结构"的转换步骤。

```
原始图像 → 灰度化 → 滤波 → 阈值/边缘 → 形态学 → findContours → 几何分析 → 输出
```

输入必须是**二值图**（白色前景 + 黑色背景），而非彩色原图。

### 2. 数据结构的内存布局

这是第一份笔记没有深入的部分：

**Point** → 单点坐标 `(x, y)`，原点在左上

**一条轮廓** = `vector<Point>`，一串连续边界点

**多条轮廓** = `vector<vector<Point>>`，双击 vector 结构：
```
contours[0] → 第 0 条的所有点
contours[1] → 第 1 条的所有点
...
```

**层级记录** = `Vec4i`，固定长度为 4 的整数向量，每条轮廓对应一个：
```
hierarchy[i] = { next, previous, first_child, parent }
```

`hierarchy` 与 `contours` 严格一一对应：`contours[i]` ↔ `hierarchy[i]`。

### 3. 输入图像要求

`findContours` 的输入必须满足：
- 单通道
- 8 位图像
- 前景白色、背景黑色
- 已经阈值分割或边缘检测

标准预处理代码：
```cpp
cv::cvtColor(src, gray, cv::COLOR_BGR2GRAY);
cv::GaussianBlur(gray, gray, cv::Size(5, 5), 0);
cv::threshold(gray, binary, 100, 255, cv::THRESH_BINARY);
```

**防御性编程**：传入 `binary.clone()` 防止原图被修改。

### 4. 轮廓近似方法对比

| 方法 | 行为 | 内存 | 典型输出 |
|------|------|------|---------|
| `CHAIN_APPROX_NONE` | 保存每个边界点 | 最大 | 100 像素水平线 → 100 个点 |
| `CHAIN_APPROX_SIMPLE` | 压缩冗余，只留端点 | 小得多 | 完美矩形 → 仅 4 个角点 |

工程默认选择：`CHAIN_APPROX_SIMPLE`。

### 5. Hierarchy：四字段拓扑（本文核心）

这是第一份笔记**完全未涉及**的内容。

```
hierarchy[i][0] = Next         同一层级的后一个轮廓（兄弟）
hierarchy[i][1] = Previous     同一层级的前一个轮廓（兄弟）
hierarchy[i][2] = First_Child  直接包含的第一个子轮廓
hierarchy[i][3] = Parent       直接外层轮廓（父亲）
```

不存在的关系值为 `-1`。

**三种基本判断**：
```cpp
hierarchy[i][3] == -1    // 是外层轮廓（无父亲）
hierarchy[i][2] != -1    // 有子轮廓（内部有孔或嵌套物）
hierarchy[i][3] != -1    // 是内部轮廓（被其他轮廓包裹）
```

**手工推演示例**（大正方形包含一圆一三角形）：

```
轮廓 0：大正方形   → hierarchy[0] = [-1, -1,  1, -1]
轮廓 1：内部圆     → hierarchy[1] = [ 2, -1, -1,  0]
轮廓 2：内部三角形 → hierarchy[2] = [-1,  1, -1,  0]
```

兄弟之间通过 `Next` / `Previous` 形成链表，可通过循环遍历所有直接子轮廓。

### 6. 四种检索模式的精确行为

| 模式 | 层级表现 | 行为 | 典型场景 |
|------|---------|------|---------|
| `RETR_EXTERNAL` | 单层 | 只取最外层，内部忽略 | 外形检测、目标计数 |
| `RETR_LIST` | 平铺 | 所有轮廓但不建父子关系 | 面积筛选、统计总数 |
| `RETR_CCOMP` | 双层 | 外边界 + 孔洞两层 | 零件孔洞判断、简化嵌套 |
| `RETR_TREE` | 完整树 | 父子孙全部保留 | 二维码定位、复杂嵌套 |

选择指南：
- 只要外边界 → `RETR_EXTERNAL`
- 需要孔洞但不想无限嵌套 → `RETR_CCOMP`
- 需要完整拓扑 → `RETR_TREE`

### 7. 轮廓几何特征（与第一份笔记互补）

**圆度（Circularity）**——第一份笔记未涉及：

```cpp
double circularity = 4.0 * CV_PI * area / (perimeter * perimeter);
// 完美圆 ≈ 1.0，偏离越多越不圆
```

**最小外接旋转矩形（minAreaRect）**——第一份只提了 `boundingRect`：

```cpp
cv::RotatedRect rotated = cv::minAreaRect(contours[i]);
// 返回带角度的矩形，适合倾斜目标
```

### 8. 四个实战项目

#### 项目一：未打孔缺陷检测

用 `hierarchy` 检测垫片是否有孔：
- `hierarchy[i][3] == -1` 且 `hierarchy[i][2] != -1` → 有孔 → 合格
- `hierarchy[i][3] == -1` 且 `hierarchy[i][2] == -1` → 无孔 → 缺陷

关键注意：
- 必须用 `RETR_TREE` 或 `RETR_CCOMP`，`RETR_EXTERNAL` 拿不到孔洞
- 需配合面积过滤排除噪声
- 判断"合格"应结合孔洞面积、圆度、位置，不只依赖 `hasChild`

#### 项目二：二维码定位图案

二维码角点的三层嵌套（黑-白-黑）通过 `RETR_TREE` 的父子孙关系定位：
```cpp
int child = hierarchy[i][2];      // 子（白块）
int grandson = hierarchy[child][2]; // 孙（内层黑块）
```

仅判断三层嵌套不够鲁棒，实际需结合面积比、正方形判断、三定位图案间的几何关系。

#### 项目三：字符/零件孔洞统计

通过遍历兄弟链表统计孔洞数量：
```cpp
int countDirectChildren(hierarchy, parentId) {
    int count = 0;
    int child = hierarchy[parentId][2];
    while (child != -1) {
        count++;
        child = hierarchy[child][0];  // 下一个兄弟
    }
    return count;
}
```
应用：区分字母 O(1 孔) vs C(0 孔)、数字 8(2 孔) vs 0(1 孔)。

#### 项目四：形状识别增强

在之前的三角形/矩形/圆形判断基础上增加了**圆度**公式和 `minAreaRect` 的应用。

### 9. 高频错误

| 错误 | 表现 | 根因 | 解决 |
|------|------|------|------|
| 传入三通道彩色图 | 运行时异常 | `src` 是 BGR 图 | 先 cvtColor + threshold |
| 数组越界 | crash | 未检查 contours.size() | `for(size_t i=0; i<contours.size(); ++i)` |
| 轮廓数量异常多 | 画面密密麻麻 | 未滤波/阈值低/纹理干扰 | GaussianBlur + 形态学开运算 + 面积过滤 |
| 使用了错误检索模式 | 内部孔洞检测不到 | 用了 RETR_EXTERNAL | 改为 RETR_TREE 或 RETR_CCOMP |
| 前景背景方向反了 | 提取背景而非目标 | 默认白色=前景 | THRESH_BINARY_INV 或 bitwise_not |

### 10. 工程实践原则

- **先可视化，再调参数**：依次显示灰度图→二值图→形态学后→轮廓图→过滤后
- **拆开条件，不要一行写死**：`bool isOuter = ...` 优于 `if(h[3]==-1 && h[2]!=-1 && area>500)`
- **单一特征不可靠**：合格判断应组合面积、周长、宽高比、圆度、层级、孔洞数、面积比

### 11. 完整模板

源末尾提供了一份可直接复用的 C++ 轮廓分析模板（约 60 行代码），覆盖完整的预处理→轮廓提取→几何计算→层级判断→输出流程。

## 关键引用

| 引用 | 上下文 |
|------|--------|
| "findContours 不是检测算法，是像素边界 → 几何结构的转换步骤" | 正确定位它在流水线中的角色 |
| "如果只做简单目标计数，只需要 contours；如果要识别嵌套结构，必须理解 hierarchy" | 两套数据的定位 |
| "轮廓分析核心三层：像素→几何、几何→拓扑、拓扑→业务判断" | 知识体系总结 |
| "先可视化，再调参数" | 工程调试原则 |

## 对已有 Wiki 的影响

| 已有页面 | 影响 |
|---------|------|
| [[Robot_Learning/concepts/OpenCV/第7章 - 形状轮廓检测|第7章 - 形状/轮廓检测]] | 已整合：findContours 几何分析 + Hierarchy 拓扑分析合并为一页 |
| [[Robot_Learning/concepts/OpenCV/第7章 - 形状轮廓检测|第7章 - 形状/轮廓检测]]（第二部分） | 本源的 hierarchy 内容是该章节的主要支撑 |
| [[Robot_Learning/sources/4h上手C++版Opencv 源|4h 上手 C++ 版 OpenCV 源]] | 本源的源头——这份源是对其轮廓检测部分的深化 |

## 相关页面

- [[Robot_Learning/concepts/OpenCV/第7章 - 形状轮廓检测|第7章 - 形状/轮廓检测]] — 本源的几何分析 + Hierarchy 理论展开
- [[Robot_Learning/sources/4h上手C++版Opencv 源|4h 上手 C++ 版 OpenCV 源]] — 前一份 OpenCV 源，本源的母体
- [[Robot_Learning/concepts/OpenCV/第2章 - 基本功能|第2章 - 基本功能]] — 轮廓提取的上游管道（灰度化→模糊→Canny→形态学）

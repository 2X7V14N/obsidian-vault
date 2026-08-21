---
title: OpenCV 轮廓提取与层级拓扑完全指南
created: 2026-07-03
tags:
  - OpenCV
  - Robot-Perception
  - Computer-Vision
  - Contours
  - Hierarchy
---

# OpenCV 轮廓提取与层级拓扑（Hierarchy）完全指南

在计算机视觉流水线中，边缘检测（如 Canny）或阈值分割（Threshold）通常只能得到一张由前景像素和背景像素组成的二值图。二值图告诉我们“哪里可能有目标边界”，但它本身并不直接告诉我们：

- 图像中有多少个独立物体；
- 每个物体的边界由哪些点组成；
- 目标之间是否存在嵌套、孔洞或包含关系；
- 哪些轮廓是外边界，哪些轮廓是内部孔洞；
- 如何把像素级结果转化为可计算的几何结构。

`cv::findContours` 的作用，就是把二值图中的白色连通边界转换为结构化的几何数据。它输出两类信息：

- `contours`：轮廓点集，回答“每条边界由哪些点组成”；
- `hierarchy`：层级拓扑，回答“这些轮廓之间是什么关系”。

如果只做简单目标计数，可能只需要 `contours`；如果要识别垫片孔洞、二维码定位图案、字符内外环、零件嵌套结构等，就必须理解 `hierarchy`。

---

## 1. 轮廓提取在视觉流水线中的位置

典型的轮廓分析流程如下：

```text
原始图像
  ↓
灰度化 / 色彩空间转换
  ↓
滤波去噪
  ↓
阈值分割 / 边缘检测
  ↓
形态学处理
  ↓
findContours 提取轮廓与层级
  ↓
轮廓筛选、测量、分类、识别
  ↓
绘制结果 / 输出检测结论
```

需要注意的是，`findContours` 通常不直接处理彩色原图，而是处理已经分割好的二值图。它更像是一个“像素边界转几何结构”的步骤，而不是完整的检测算法本身。

---

## 2. 核心数据结构

### 2.1 `cv::Point`

`cv::Point` 表示图像中的一个二维像素坐标：

```cpp
cv::Point p(120, 80);
```

其中：

- `p.x = 120` 表示横向坐标；
- `p.y = 80` 表示纵向坐标；
- OpenCV 图像坐标原点位于左上角；
- `x` 向右增大，`y` 向下增大。

### 2.2 一条轮廓：`std::vector<cv::Point>`

一条轮廓本质上是一串连续边界点：

```cpp
std::vector<cv::Point> contour;
```

可以理解为：

```text
contour = [
  (x0, y0),
  (x1, y1),
  (x2, y2),
  ...
]
```

这代表一条闭合或近似闭合的几何边界。比如一个矩形的边界、一个圆的边界，或者一个零件外沿的边界。

### 2.3 多条轮廓：`std::vector<std::vector<cv::Point>>`

一张图中往往会有多个目标，每个目标可能又有多个边界。因此 OpenCV 使用双层 `vector` 存储所有轮廓：

```cpp
std::vector<std::vector<cv::Point>> contours;
```

结构可以理解为：

```text
contours
├── contours[0] -> 第 0 条轮廓的所有点
├── contours[1] -> 第 1 条轮廓的所有点
├── contours[2] -> 第 2 条轮廓的所有点
└── ...
```

如果图像中找到了 5 条轮廓，那么 `contours.size()` 就是 5。

### 2.4 层级记录：`cv::Vec4i`

`cv::Vec4i` 是 OpenCV 中一个固定长度为 4 的整数向量：

```cpp
cv::Vec4i h;
```

它等价于一个包含 4 个 `int` 的小数组。用于轮廓层级时，每条轮廓对应一个 `Vec4i`：

```cpp
std::vector<cv::Vec4i> hierarchy;
```

`hierarchy` 与 `contours` 严格一一对应：

```text
contours[0]  <-> hierarchy[0]
contours[1]  <-> hierarchy[1]
contours[2]  <-> hierarchy[2]
...
```

第 `i` 条轮廓的拓扑关系，就保存在 `hierarchy[i]` 中。

---

## 3. 寻找轮廓：`cv::findContours`

基本用法：

```cpp
std::vector<std::vector<cv::Point>> contours;
std::vector<cv::Vec4i> hierarchy;

cv::findContours(
    binary,
    contours,
    hierarchy,
    cv::RETR_TREE,
    cv::CHAIN_APPROX_SIMPLE
);
```

### 3.1 参数说明

| 参数 | 含义 | 关键要求 |
| --- | --- | --- |
| `binary` | 输入图像 | 通常是 8 位单通道二值图 |
| `contours` | 输出轮廓点集 | 每个元素是一条轮廓 |
| `hierarchy` | 输出层级关系 | 与 `contours` 一一对应 |
| `mode` | 轮廓检索模式 | 决定是否建立父子关系 |
| `method` | 轮廓近似方法 | 决定保存多少边界点 |

### 3.2 输入图像要求

`findContours` 的输入通常应满足：

- 单通道；
- 8 位图像；
- 前景为白色，背景为黑色；
- 已经经过阈值分割、边缘检测或形态学处理。

常见预处理示例：

```cpp
cv::Mat src = cv::imread("part.png");
cv::Mat gray, binary;

cv::cvtColor(src, gray, cv::COLOR_BGR2GRAY);
cv::GaussianBlur(gray, gray, cv::Size(5, 5), 0);
cv::threshold(gray, binary, 100, 255, cv::THRESH_BINARY);
```

如果源图后续还要使用，建议传入副本：

```cpp
cv::findContours(binary.clone(), contours, hierarchy,
                 cv::RETR_TREE, cv::CHAIN_APPROX_SIMPLE);
```

在 OpenCV 旧版本中，`findContours` 可能修改输入图像；使用 `clone()` 是一个稳妥习惯。

---

## 4. 轮廓近似方法：`CHAIN_APPROX_*`

轮廓点可能非常多。一个大圆或不规则边缘可能包含上千个像素点。OpenCV 允许通过近似方法控制保存点的数量。

### 4.1 `cv::CHAIN_APPROX_NONE`

保存轮廓上的每一个点：

```cpp
cv::findContours(binary, contours, hierarchy,
                 cv::RETR_TREE, cv::CHAIN_APPROX_NONE);
```

特点：

- 信息最完整；
- 内存占用最大；
- 后续计算更慢；
- 适合需要精确边界轨迹的场景。

例如一条 100 像素长的水平线，可能会保存 100 个点。

### 4.2 `cv::CHAIN_APPROX_SIMPLE`

压缩水平、垂直、对角线方向上的冗余点，只保留关键端点：

```cpp
cv::findContours(binary, contours, hierarchy,
                 cv::RETR_TREE, cv::CHAIN_APPROX_SIMPLE);
```

特点：

- 保存点更少；
- 计算更快；
- 对矩形、多边形等规则目标尤其高效；
- 是工程中最常用的选择。

例如一个完美矩形的边界，使用 `CHAIN_APPROX_NONE` 可能保存几百个点，而 `CHAIN_APPROX_SIMPLE` 通常只保留 4 个角点。

---

## 5. 绘制轮廓：`cv::drawContours`

提取轮廓后，通常需要将结果画回图像上，用于调试和可视化。

```cpp
cv::drawContours(
    src,
    contours,
    -1,
    cv::Scalar(255, 0, 255),
    2
);
```

### 5.1 参数说明

| 参数 | 含义 |
| --- | --- |
| `src` | 绘制目标图像 |
| `contours` | 轮廓数据 |
| `-1` | 绘制所有轮廓 |
| `cv::Scalar(255, 0, 255)` | BGR 颜色，表示洋红色 |
| `2` | 线宽为 2 像素 |

如果只想绘制第 `i` 条轮廓：

```cpp
cv::drawContours(src, contours, static_cast<int>(i),
                 cv::Scalar(0, 255, 0), 2);
```

如果想填充轮廓内部：

```cpp
cv::drawContours(mask, contours, static_cast<int>(i),
                 cv::Scalar(255), cv::FILLED);
```

填充轮廓常用于生成掩膜（Mask）。

---

## 6. `hierarchy` 的核心含义

当图像中存在嵌套结构时，例如：

- 大圆内部有小圆；
- 金属垫片有外圈和内孔；
- 字母 `O` 有外边界和内边界；
- 二维码定位图案有黑白黑三层嵌套；
- 一个零件外壳内部有多个孔洞。

仅有 `contours` 是不够的，因为 `contours` 只告诉你“有哪些轮廓”，不告诉你“谁包含谁”。这时就需要 `hierarchy`。

对于第 `i` 条轮廓：

```cpp
hierarchy[i] = { next, previous, first_child, parent };
```

也就是：

```text
hierarchy[i][0] = Next
hierarchy[i][1] = Previous
hierarchy[i][2] = First_Child
hierarchy[i][3] = Parent
```

如果某个关系不存在，对应值为 `-1`。

### 6.1 四个字段

| 字段 | 含义 | 通俗理解 |
| --- | --- | --- |
| `Next` | 同一层级中的下一个轮廓 | 下一个兄弟 |
| `Previous` | 同一层级中的上一个轮廓 | 上一个兄弟 |
| `First_Child` | 当前轮廓直接包含的第一个子轮廓 | 第一个孩子 |
| `Parent` | 当前轮廓的直接外层轮廓 | 父轮廓 |

### 6.2 关键判断

判断是否为最外层轮廓：

```cpp
if (hierarchy[i][3] == -1) {
    // 没有父轮廓，是外层轮廓
}
```

判断是否包含内部孔洞或子轮廓：

```cpp
if (hierarchy[i][2] != -1) {
    // 有子轮廓
}
```

判断是否为内部孔洞：

```cpp
if (hierarchy[i][3] != -1) {
    // 有父轮廓，说明它被其他轮廓包裹
}
```

---

## 7. 层级关系推演示例

假设图像中有 3 条轮廓：

- `0` 号轮廓：一个外部大正方形；
- `1` 号轮廓：大正方形内部左侧的圆；
- `2` 号轮廓：大正方形内部右侧的三角形。

拓扑关系如下：

```text
轮廓 0：大正方形
├── 轮廓 1：内部圆
└── 轮廓 2：内部三角形
```

那么可能得到：

```text
hierarchy[0] = [-1, -1,  1, -1]
hierarchy[1] = [ 2, -1, -1,  0]
hierarchy[2] = [-1,  1, -1,  0]
```

### 7.1 `hierarchy[0] = [-1, -1, 1, -1]`

含义：

- `Next = -1`：外层同级没有下一个轮廓；
- `Previous = -1`：外层同级没有上一个轮廓；
- `First_Child = 1`：第一个子轮廓是 1 号；
- `Parent = -1`：它没有父轮廓，是最外层轮廓。

### 7.2 `hierarchy[1] = [2, -1, -1, 0]`

含义：

- `Next = 2`：同级下一个兄弟是 2 号；
- `Previous = -1`：同级前面没有兄弟；
- `First_Child = -1`：它没有子轮廓；
- `Parent = 0`：它被 0 号轮廓包裹。

### 7.3 `hierarchy[2] = [-1, 1, -1, 0]`

含义：

- `Next = -1`：同级后面没有兄弟；
- `Previous = 1`：同级前一个兄弟是 1 号；
- `First_Child = -1`：它没有子轮廓；
- `Parent = 0`：它也被 0 号轮廓包裹。

---

## 8. 四种常用轮廓检索模式

`findContours` 的第四个参数决定如何检索轮廓，以及如何构建 `hierarchy`。

| 检索模式 | 层级表现 | 行为 | 适用场景 |
| --- | --- | --- | --- |
| `cv::RETR_EXTERNAL` | 单层 | 只提取最外层轮廓，忽略所有内部轮廓 | 只关心目标外形、目标计数、外接框检测 |
| `cv::RETR_LIST` | 平铺 | 提取所有轮廓，但不建立父子关系 | 只关心轮廓数量、面积、周长，不关心嵌套 |
| `cv::RETR_CCOMP` | 双层 | 建立两层结构：外边界和孔洞 | 分析带孔目标、简化孔洞结构 |
| `cv::RETR_TREE` | 完整树 | 提取所有轮廓并建立完整父子孙关系 | 复杂嵌套、二维码定位、结构识别 |

### 8.1 `cv::RETR_EXTERNAL`

只保留最外层轮廓：

```cpp
cv::findContours(binary, contours, hierarchy,
                 cv::RETR_EXTERNAL, cv::CHAIN_APPROX_SIMPLE);
```

特点：

- 内部孔洞不会被提取；
- 所有返回轮廓都没有父轮廓；
- `hierarchy[i][2]` 和 `hierarchy[i][3]` 通常为 `-1`；
- 适合外形检测。

典型用途：

- 统计画面中有多少个独立目标；
- 获取每个目标的外接矩形；
- 忽略内部纹理、孔洞、反光造成的小轮廓。

### 8.2 `cv::RETR_LIST`

提取所有轮廓，但不组织父子关系：

```cpp
cv::findContours(binary, contours, hierarchy,
                 cv::RETR_LIST, cv::CHAIN_APPROX_SIMPLE);
```

特点：

- 所有轮廓被视为同一层；
- 不记录谁包含谁；
- `First_Child` 和 `Parent` 通常为 `-1`。

适合场景：

- 只做面积筛选；
- 只统计轮廓总数；
- 不需要孔洞或包含关系。

### 8.3 `cv::RETR_CCOMP`

建立双层结构：

```cpp
cv::findContours(binary, contours, hierarchy,
                 cv::RETR_CCOMP, cv::CHAIN_APPROX_SIMPLE);
```

特点：

- 第一层是外边界；
- 第二层是孔洞边界；
- 如果孔洞中还有物体，该物体会被提升为第一层。

适合场景：

- 检测零件是否有孔；
- 分离外边界和内部空洞；
- 需要孔洞信息，但不想处理无限深嵌套。

### 8.4 `cv::RETR_TREE`

建立完整树状结构：

```cpp
cv::findContours(binary, contours, hierarchy,
                 cv::RETR_TREE, cv::CHAIN_APPROX_SIMPLE);
```

特点：

- 所有轮廓都会被提取；
- 父子、兄弟、孙子关系都会保留；
- 是层级信息最完整的模式。

适合场景：

- 二维码定位图案；
- 多层嵌套结构分析；
- 复杂零件内部结构检测；
- 根据包含层数识别目标。

---

## 9. 常用轮廓几何特征

提取轮廓后，通常会继续计算几何特征，用于筛选或分类。

### 9.1 面积：`cv::contourArea`

```cpp
double area = cv::contourArea(contours[i]);
```

用途：

- 过滤小噪声；
- 判断目标大小；
- 对零件面积做质量检测。

示例：

```cpp
if (cv::contourArea(contours[i]) < 100.0) {
    continue;
}
```

### 9.2 周长：`cv::arcLength`

```cpp
double perimeter = cv::arcLength(contours[i], true);
```

第二个参数表示轮廓是否闭合：

- `true`：闭合轮廓；
- `false`：非闭合曲线。

### 9.3 多边形近似：`cv::approxPolyDP`

```cpp
std::vector<cv::Point> approx;
double epsilon = 0.02 * cv::arcLength(contours[i], true);

cv::approxPolyDP(contours[i], approx, epsilon, true);
```

用途：

- 判断矩形、三角形、多边形；
- 减少轮廓点；
- 提取角点结构。

例如：

```cpp
if (approx.size() == 3) {
    // 三角形
} else if (approx.size() == 4) {
    // 四边形
}
```

### 9.4 外接矩形：`cv::boundingRect`

```cpp
cv::Rect box = cv::boundingRect(contours[i]);
cv::rectangle(src, box, cv::Scalar(0, 255, 0), 2);
```

用途：

- 快速定位目标；
- 裁剪 ROI；
- 计算宽高比。

### 9.5 最小外接旋转矩形：`cv::minAreaRect`

```cpp
cv::RotatedRect rotated = cv::minAreaRect(contours[i]);
```

用途：

- 获取倾斜目标的方向；
- 检测长条零件姿态；
- 工业抓取中的角度估计。

---

## 10. 实战一：检测未打孔的缺陷零件

### 10.1 业务背景

流水线上有一批金属垫片。合格垫片应该具有：

- 一个外圈轮廓；
- 一个内孔轮廓；
- 内孔轮廓是外圈轮廓的子轮廓。

如果某个零件没有打孔，它可能仍然拥有相似的外部形状和面积，仅靠外轮廓无法判断是否合格。因此需要通过 `hierarchy` 判断外轮廓是否拥有子轮廓。

### 10.2 判断逻辑

对每条轮廓 `i`：

```cpp
bool isOuterContour = hierarchy[i][3] == -1;
bool hasHole = hierarchy[i][2] != -1;
```

那么：

- `isOuterContour == true` 且 `hasHole == true`：可能是合格垫片；
- `isOuterContour == true` 且 `hasHole == false`：可能是未打孔缺陷件；
- `isOuterContour == false`：它是内部孔洞或内部结构，不直接作为独立零件判断。

### 10.3 示例代码

```cpp
#include <opencv2/opencv.hpp>
#include <iostream>

int main() {
    cv::Mat src = cv::imread("factory_parts.png");
    if (src.empty()) {
        std::cout << "无法读取图像" << std::endl;
        return -1;
    }

    cv::Mat gray, binary;
    cv::cvtColor(src, gray, cv::COLOR_BGR2GRAY);
    cv::GaussianBlur(gray, gray, cv::Size(5, 5), 0);
    cv::threshold(gray, binary, 100, 255, cv::THRESH_BINARY_INV);

    std::vector<std::vector<cv::Point>> contours;
    std::vector<cv::Vec4i> hierarchy;

    cv::findContours(binary, contours, hierarchy,
                     cv::RETR_TREE, cv::CHAIN_APPROX_SIMPLE);

    if (hierarchy.empty()) {
        std::cout << "未找到有效轮廓" << std::endl;
        return 0;
    }

    for (size_t i = 0; i < contours.size(); ++i) {
        double area = cv::contourArea(contours[i]);
        if (area < 500.0) {
            continue;
        }

        bool isOuterContour = hierarchy[i][3] == -1;
        bool hasChild = hierarchy[i][2] != -1;

        if (isOuterContour && !hasChild) {
            cv::drawContours(src, contours, static_cast<int>(i),
                             cv::Scalar(0, 0, 255), 3);
            std::cout << "发现疑似未打孔缺陷件，轮廓索引: "
                      << i << std::endl;
        } else if (isOuterContour && hasChild) {
            cv::drawContours(src, contours, static_cast<int>(i),
                             cv::Scalar(0, 255, 0), 2);
        }
    }

    cv::imshow("Inspection Result", src);
    cv::waitKey(0);
    return 0;
}
```

### 10.4 工程注意点

- 阈值方向要根据前景和背景亮度选择，必要时使用 `THRESH_BINARY_INV`；
- 需要使用 `RETR_TREE` 或 `RETR_CCOMP`，否则内部孔洞信息可能丢失；
- 要用面积过滤小噪声，避免把划痕、污点当成孔洞；
- 判断“是否合格”时，最好结合孔洞面积、圆度、位置等特征，而不只判断 `hasChild`。

---

## 11. 实战二：识别二维码定位图案

### 11.1 业务背景

二维码角点的定位图案在二值图中通常表现为多层嵌套结构：

```text
外层黑方块
  ↓
中间白方块
  ↓
内层黑方块
```

这类结构可以通过 `RETR_TREE` 形成的父子孙关系快速定位。

### 11.2 判断逻辑

对某条轮廓 `i`：

```cpp
int child = hierarchy[i][2];
if (child != -1) {
    int grandson = hierarchy[child][2];
}
```

如果 `i` 有子轮廓，子轮廓又有自己的子轮廓，则说明存在连续三层嵌套。

### 11.3 示例代码

```cpp
#include <opencv2/opencv.hpp>
#include <iostream>

int main() {
    cv::Mat src = cv::imread("qr_code_scan.jpg");
    if (src.empty()) {
        return -1;
    }

    cv::Mat gray, binary;
    cv::cvtColor(src, gray, cv::COLOR_BGR2GRAY);

    cv::adaptiveThreshold(gray, binary, 255,
                          cv::ADAPTIVE_THRESH_GAUSSIAN_C,
                          cv::THRESH_BINARY,
                          51,
                          2);

    std::vector<std::vector<cv::Point>> contours;
    std::vector<cv::Vec4i> hierarchy;

    cv::findContours(binary, contours, hierarchy,
                     cv::RETR_TREE, cv::CHAIN_APPROX_SIMPLE);

    if (hierarchy.empty()) {
        return 0;
    }

    for (size_t i = 0; i < contours.size(); ++i) {
        double area = cv::contourArea(contours[i]);
        if (area < 50.0) {
            continue;
        }

        int child = hierarchy[i][2];
        if (child == -1) {
            continue;
        }

        int grandson = hierarchy[child][2];
        if (grandson == -1) {
            continue;
        }

        cv::drawContours(src, contours, static_cast<int>(i),
                         cv::Scalar(0, 255, 0), 2);

        std::cout << "发现疑似二维码定位图案，外层轮廓索引: "
                  << i << std::endl;
    }

    cv::imshow("QR Finder Pattern Candidates", src);
    cv::waitKey(0);
    return 0;
}
```

### 11.4 进一步增强

仅判断三层嵌套还不够鲁棒，实际工程中还应结合：

- 外层、中层、内层的面积比例；
- 轮廓是否接近正方形；
- 三个定位图案之间的几何关系；
- 候选区域的宽高比；
- 是否存在明显透视变形。

---

## 12. 实战三：提取字符或零件中的孔洞

字符识别和零件检测中，经常要判断一个目标内部是否有孔：

- 字母 `O`、`D`、`P`、`A` 有内部孔洞；
- 数字 `0`、`6`、`8`、`9` 有内部孔洞；
- 垫片、法兰盘、机械连接件可能有一个或多个孔。

### 12.1 统计每个外轮廓的直接子轮廓数量

```cpp
int countDirectChildren(const std::vector<cv::Vec4i>& hierarchy, int parentId) {
    int count = 0;
    int child = hierarchy[parentId][2];

    while (child != -1) {
        ++count;
        child = hierarchy[child][0];
    }

    return count;
}
```

逻辑说明：

- `hierarchy[parentId][2]` 找到第一个孩子；
- 每个孩子通过 `hierarchy[child][0]` 找到下一个兄弟；
- 一直遍历到 `-1`。

### 12.2 应用示例

```cpp
for (size_t i = 0; i < contours.size(); ++i) {
    if (hierarchy[i][3] != -1) {
        continue;
    }

    int holeCount = countDirectChildren(hierarchy, static_cast<int>(i));

    if (holeCount > 0) {
        std::cout << "外轮廓 " << i
                  << " 包含孔洞数量: "
                  << holeCount << std::endl;
    }
}
```

这比只判断 `hierarchy[i][2] != -1` 更细，因为它可以统计一个外轮廓内部有多少个直接孔洞。

---

## 13. 实战四：从轮廓中提取形状

### 13.1 形状识别流程

```text
findContours
  ↓
面积过滤
  ↓
计算周长 arcLength
  ↓
多边形近似 approxPolyDP
  ↓
根据角点数、宽高比、圆度分类
```

### 13.2 示例代码

```cpp
for (size_t i = 0; i < contours.size(); ++i) {
    double area = cv::contourArea(contours[i]);
    if (area < 100.0) {
        continue;
    }

    double perimeter = cv::arcLength(contours[i], true);

    std::vector<cv::Point> approx;
    cv::approxPolyDP(contours[i], approx, 0.02 * perimeter, true);

    cv::Rect box = cv::boundingRect(approx);
    double aspectRatio = static_cast<double>(box.width) / box.height;

    std::string shape;
    if (approx.size() == 3) {
        shape = "Triangle";
    } else if (approx.size() == 4) {
        if (aspectRatio > 0.95 && aspectRatio < 1.05) {
            shape = "Square";
        } else {
            shape = "Rectangle";
        }
    } else {
        shape = "Circle or Polygon";
    }

    cv::drawContours(src, contours, static_cast<int>(i),
                     cv::Scalar(0, 255, 0), 2);
    cv::putText(src, shape, box.tl(),
                cv::FONT_HERSHEY_SIMPLEX,
                0.6,
                cv::Scalar(255, 0, 0),
                2);
}
```

### 13.3 圆度判断

圆度常用公式：

```text
circularity = 4 * pi * area / perimeter^2
```

代码：

```cpp
double circularity = 4.0 * CV_PI * area / (perimeter * perimeter);
```

圆越标准，`circularity` 越接近 1。

---

## 14. 高频错误与避坑指南

### 14.1 错误一：输入了三通道彩色图

错误表现：

```text
cv::Exception: Unsupported format or combination of formats in function 'findContours'
```

常见原因：

```cpp
cv::findContours(src, contours, hierarchy,
                 cv::RETR_TREE, cv::CHAIN_APPROX_SIMPLE);
```

其中 `src` 是 BGR 彩色图。

正确做法：

```cpp
cv::Mat gray, binary;
cv::cvtColor(src, gray, cv::COLOR_BGR2GRAY);
cv::threshold(gray, binary, 100, 255, cv::THRESH_BINARY);

cv::findContours(binary, contours, hierarchy,
                 cv::RETR_TREE, cv::CHAIN_APPROX_SIMPLE);
```

### 14.2 错误二：遍历时数组越界

错误表现：

```text
vector subscript out of range
```

常见原因：

```cpp
for (int i = 0; i < 10; ++i) {
    cv::drawContours(src, contours, i, cv::Scalar(0, 255, 0), 2);
}
```

如果实际只找到 3 条轮廓，访问第 4 条就会越界。

正确做法：

```cpp
for (size_t i = 0; i < contours.size(); ++i) {
    cv::drawContours(src, contours, static_cast<int>(i),
                     cv::Scalar(0, 255, 0), 2);
}
```

访问 `hierarchy` 前也要确认它不为空：

```cpp
if (!hierarchy.empty()) {
    // 安全访问 hierarchy[i]
}
```

### 14.3 错误三：轮廓数量异常多

表现：

- 画面上密密麻麻都是轮廓；
- 结果被噪声、纹理、反光干扰；
- `contours.size()` 远超预期。

常见原因：

- 没有滤波；
- 阈值过低；
- 背景纹理被识别为前景；
- 反光、阴影、划痕形成碎片轮廓。

解决方法：

```cpp
cv::GaussianBlur(gray, gray, cv::Size(5, 5), 0);
cv::threshold(gray, binary, 100, 255, cv::THRESH_BINARY);

cv::Mat kernel = cv::getStructuringElement(cv::MORPH_RECT, cv::Size(3, 3));
cv::morphologyEx(binary, binary, cv::MORPH_OPEN, kernel);
```

并在轮廓遍历时做面积过滤：

```cpp
if (cv::contourArea(contours[i]) < 100.0) {
    continue;
}
```

### 14.4 错误四：使用了错误的检索模式

例如需要检测垫片孔洞，却使用：

```cpp
cv::RETR_EXTERNAL
```

这样只能拿到最外层轮廓，内部孔洞会被忽略。

如果需要父子关系，应使用：

```cpp
cv::RETR_TREE
```

如果只需要外边界和孔洞两层结构，可以使用：

```cpp
cv::RETR_CCOMP
```

### 14.5 错误五：前景背景方向反了

`findContours` 会把白色区域视为前景。如果目标是黑色、背景是白色，直接阈值分割可能会提取背景外边界，而不是目标。

可以使用反向阈值：

```cpp
cv::threshold(gray, binary, 100, 255, cv::THRESH_BINARY_INV);
```

或者反转图像：

```cpp
cv::bitwise_not(binary, binary);
```

---

## 15. 工程实践建议

### 15.1 先可视化，再调参数

轮廓结果强烈依赖预处理。调试时建议依次显示：

- 灰度图；
- 二值图；
- 形态学处理后的图；
- 轮廓绘制结果；
- 面积过滤后的结果。

只有看到每一步的图像，才能准确判断问题出在阈值、噪声、背景、形态学核大小，还是轮廓检索模式。

### 15.2 把筛选条件拆开写

不要把复杂判断直接写成一行：

```cpp
if (hierarchy[i][3] == -1 && hierarchy[i][2] != -1 && area > 500) {
    // ...
}
```

更推荐：

```cpp
bool isOuterContour = hierarchy[i][3] == -1;
bool hasChild = hierarchy[i][2] != -1;
bool areaIsValid = area > 500.0;

if (isOuterContour && hasChild && areaIsValid) {
    // ...
}
```

这样便于调试，也便于后续加入更多规则。

### 15.3 避免只依赖单一特征

真实项目中，不建议只根据一个条件下结论。例如：

- 不能只根据 `hasChild` 判断垫片合格；
- 不能只根据三层嵌套判断二维码；
- 不能只根据角点数判断矩形。

更稳健的方式是组合多个特征：

- 面积；
- 周长；
- 宽高比；
- 圆度；
- 层级关系；
- 孔洞数量；
- 子轮廓面积比例；
- 目标相对位置。

---

## 16. 一份完整的轮廓分析模板

下面是一份较通用的 C++ 模板，可作为后续项目起点。

```cpp
#include <opencv2/opencv.hpp>
#include <iostream>

int main() {
    cv::Mat src = cv::imread("input.png");
    if (src.empty()) {
        std::cout << "无法读取图像" << std::endl;
        return -1;
    }

    cv::Mat gray;
    cv::cvtColor(src, gray, cv::COLOR_BGR2GRAY);

    cv::Mat blurred;
    cv::GaussianBlur(gray, blurred, cv::Size(5, 5), 0);

    cv::Mat binary;
    cv::threshold(blurred, binary, 100, 255, cv::THRESH_BINARY);

    cv::Mat kernel = cv::getStructuringElement(cv::MORPH_RECT, cv::Size(3, 3));
    cv::morphologyEx(binary, binary, cv::MORPH_OPEN, kernel);

    std::vector<std::vector<cv::Point>> contours;
    std::vector<cv::Vec4i> hierarchy;

    cv::findContours(binary.clone(), contours, hierarchy,
                     cv::RETR_TREE, cv::CHAIN_APPROX_SIMPLE);

    for (size_t i = 0; i < contours.size(); ++i) {
        double area = cv::contourArea(contours[i]);
        if (area < 100.0) {
            continue;
        }

        double perimeter = cv::arcLength(contours[i], true);
        cv::Rect box = cv::boundingRect(contours[i]);

        bool hasHierarchy = !hierarchy.empty();
        bool isOuterContour = hasHierarchy && hierarchy[i][3] == -1;
        bool hasChild = hasHierarchy && hierarchy[i][2] != -1;

        cv::Scalar color = isOuterContour ? cv::Scalar(0, 255, 0)
                                          : cv::Scalar(255, 0, 0);

        cv::drawContours(src, contours, static_cast<int>(i), color, 2);
        cv::rectangle(src, box, cv::Scalar(0, 255, 255), 1);

        std::cout << "轮廓 " << i
                  << " | 面积: " << area
                  << " | 周长: " << perimeter
                  << " | 外层: " << isOuterContour
                  << " | 有子轮廓: " << hasChild
                  << std::endl;
    }

    cv::imshow("Binary", binary);
    cv::imshow("Contours", src);
    cv::waitKey(0);

    return 0;
}
```

---

## 17. 总结

OpenCV 轮廓分析的核心可以概括为三层：

第一层是像素到几何：  
`findContours` 把二值图中的前景边界转换成 `vector<vector<Point>>`，让我们可以计算面积、周长、外接框、多边形近似等几何特征。

第二层是几何到拓扑：  
`hierarchy` 使用 `[Next, Previous, First_Child, Parent]` 描述轮廓之间的兄弟关系和父子关系，解决“谁包含谁”的问题。

第三层是拓扑到业务判断：  
在工业检测、二维码识别、字符分析和零件孔洞检测中，真正有价值的不是单独一条轮廓，而是轮廓的结构关系。掌握 `hierarchy` 后，就可以把图像中的嵌套、孔洞、外边界和内部结构转化为稳定的算法判断。

最常用的经验选择是：

- 只要外边界：用 `RETR_EXTERNAL`；
- 所有轮廓但不关心嵌套：用 `RETR_LIST`；
- 外边界和孔洞两层：用 `RETR_CCOMP`；
- 完整父子孙关系：用 `RETR_TREE`；
- 工程默认近似方法：用 `CHAIN_APPROX_SIMPLE`。

真正写项目时，建议先把预处理结果可视化，再选择合适的检索模式，并结合面积、形状、位置和层级关系综合判断。这样得到的轮廓系统才会稳定、可解释，也更接近工业应用中的可靠视觉算法。

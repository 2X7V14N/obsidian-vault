---
tags: [source, opencv, cpp, note]
created: 2026-06-04
updated: 2026-06-04
status: budding
domain: robot-learning
source_file: Robot_Learning/raw/4h上手C++版Opencv.md
source_type: note
source_author: Murtaza's Workshop (Bilibili)
ingest_date: 2026-06-04
---

# 4h 上手 C++ 版 OpenCV 源

> **原文**：[[Robot_Learning/raw/4h上手C++版Opencv.md]]
> **来源**：[Bilibili - Murtaza's Workshop](https://www.bilibili.com/video/BV11A411T7rL/)
> **摄入日期**：2026-06-04
> **源类型**：课程笔记（基于视频教程整理）

## 摘要

这是一份面向 C++ 初学者的 OpenCV 实战教程笔记，覆盖从环境搭建到三个完整项目的全流程。特点是以代码示例驱动教学，略去理论推导，强调"看到效果"。内容包括图像 I/O、基本处理函数、几何变换、颜色检测、轮廓/形状识别、Haar 级联目标检测，以及虚拟画笔、文档扫描、车牌检测三个项目实战。

## 详细提取

### 1. 图像的数字表示

**像素与分辨率**：图像由像素（pixel）组成。分辨率如 VGA (640×480)、HD (1280×720) 决定了图像的像素数量。

**三种图像类型**：
- **二值图像**：仅含黑色(0)和白色(1)
- **灰度图像**：每个像素用一个 8 位无符号整数表示，范围 0(黑)∼255(白)，共 256 个灰度级
- **彩色图像 (BGR)**：由蓝(Blue)、绿(Green)、红(Red)三个通道叠加而成。OpenCV 默认使用 **BGR** 而非 RGB 颜色顺序

> ⚠️ OpenCV 的 BGR 惯例是其最常被忽视的坑——当与其他库（如 matplotlib 的 RGB）交互时，需要手动转换通道顺序。

### 2. Core 代码结构

所有 OpenCV C++ 程序需要的基础导入：

```cpp
#include <opencv2/imgcodecs.hpp>   // 图像读写
#include <opencv2/highgui.hpp>     // GUI 窗口与显示
#include <opencv2/imgproc.hpp>     // 图像处理函数
#include <iostream>

using namespace cv;
using namespace std;
```

- `imgcodecs.hpp`：`imread()`、`imwrite()`
- `highgui.hpp`：`imshow()`、`namedWindow()`、`createTrackbar()`
- `imgproc.hpp`：`cvtColor()`、`GaussianBlur()`、`Canny()`、`findContours()` 等核心处理函数

### 3. 图像与视频 I/O

**读取并显示图像**：
```cpp
string path = "resources/test.png";
Mat image = imread(path);       // Mat 是 OpenCV 核心矩阵类
imshow("Image", image);
waitKey(0);                     // 0 = 无限等待按键
```

**播放视频**（视频 = 连续图像帧）：
```cpp
VideoCapture cap("resources/test_video.mp4");
Mat image;
while (true) {
    cap.read(image);            // 逐帧读取
    imshow("Video", image);
    waitKey(1);                 // 延迟 ms，越大播放越慢
}
```

**摄像头捕获**：
```cpp
VideoCapture cap(0);            // 0 = 默认摄像头，1/2 = 外接
```

### 4. 基本图像处理函数

**灰度化**：
```cpp
cvtColor(image, imageGray, COLOR_BGR2GRAY);
```

**高斯模糊**（降噪、减少细节）：
```cpp
GaussianBlur(imageGray, imageBlur, Size(7, 7), 5, 0);
// Size(7,7)：核大小，必须为奇数。sigmaX=5, sigmaY=0（自动计算）
```

**Canny 边缘检测**（通常在模糊后使用）：
```cpp
Canny(imageBlur, imageCanny, 50, 150);
// 阈值比通常为 1:2 或 1:3
```

**形态学操作——膨胀与侵蚀**：
```cpp
Mat kernel = getStructuringElement(MORPH_RECT, Size(5, 5));

dilate(imageCanny, imageDilate, kernel);    // 膨胀：加粗/闭合边缘
erode(imageDilate, imageErode, kernel);     // 侵蚀：细化边缘
```

典型管道：`灰度化 → 高斯模糊 → Canny → 膨胀`，用于让边缘连续闭合，为后续轮廓检测做准备。

### 5. 几何变换

**缩放**：
```cpp
resize(image, imageResize, Size(640, 480));           // 指定尺寸（可能变形）
resize(image, imageResize, Size(), 0.5, 0.5);         // 按比例（推荐）
```

**裁剪 (ROI)**：
```cpp
Rect roi(100, 100, 300, 250);        // x, y, width, height
Mat imageCrop = image(roi);
```

### 6. 绘图函数

在空白画布或现有图像上绘制：

```cpp
Mat img(512, 512, CV_8UC3, Scalar(255, 255, 255));    // 白色背景
// Scalar(B, G, R) — OpenCV 颜色始终是 BGR 顺序

circle(img, Point(256, 256), 155, Scalar(0, 69, 255), FILLED);
rectangle(img, Rect(130, 226, 382, 286), Scalar(255, 255, 255), FILLED);
line(img, Point(130, 296), Point(382, 296), Scalar(255, 255, 255), 2);
putText(img, "Text", Point(137, 262), FONT_HERSHEY_PLAIN, 0.75, Scalar(0, 69, 255), 2);
```

### 7. 透视变换

从图像中提取平面物体并转换为鸟瞰视角。四步操作：

```cpp
// 1. 定义源四角点（从原始图像中选取）
vector<Point2f> src = { {529,142}, {771,190}, {405,395}, {674,457} };

// 2. 定义目标四角点（输出矩形的四个角）
float w = 250, h = 350;
vector<Point2f> dst = { {0.0f,0.0f}, {w,0.0f}, {0.0f,h}, {w,h} };

// 3. 计算透视变换矩阵
Mat matrix = getPerspectiveTransform(src, dst);

// 4. 应用变换
warpPerspective(image, imageWarp, matrix, Point(w, h));
```

### 8. HSV 颜色检测

**为什么用 HSV 而非 BGR**：HSV 将颜色信息（Hue）与亮度（Value）分离，使得对光照变化更鲁棒。同一颜色在不同光照下的 H 值基本不变。

**检测流程**：
```cpp
cvtColor(image, imageHSV, COLOR_BGR2HSV);           // 转换到 HSV

Scalar lower(h_min, s_min, v_min);                  // 颜色下界
Scalar upper(h_max, s_max, v_max);                  // 颜色上界
inRange(imageHSV, lower, upper, mask);              // 二值掩膜
```

**用 Trackbar 实时调参**（OpenCV 的实用技巧）：
```cpp
namedWindow("Trackbars", Size(640, 200));
createTrackbar("Hue Min", "Trackbars", &h_min, 179);
createTrackbar("Hue Max", "Trackbars", &h_max, 179);
// ... Sat Min/Max, Val Min/Max ...
// 在 while 循环中动态更新 lower/upper 并重新生成 mask
```

HSV 的 H 范围是 0∼179（OpenCV 中为 0∼180 的半开区间），S 和 V 为 0∼255。

### 9. 轮廓检测与形状识别

这是计算机视觉中最常用的操作之一。

**完整流程**：

```cpp
// Step 1: 预处理
cvtColor → GaussianBlur → Canny → dilate

// Step 2: 查找轮廓
vector<vector<Point>> contours;
vector<Vec4i> hierarchy;
findContours(imageDilate, contours, hierarchy, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE);

// Step 3: 遍历并过滤
for (int i = 0; i < contours.size(); i++) {
    int area = contourArea(contours[i]);
    if (area > 1000) {                          // 过滤噪声
        // 计算周长
        float peri = arcLength(contours[i], true);
        
        // 多边形近似
        vector<vector<Point>> conPoly(contours.size());
        approxPolyDP(contours[i], conPoly[i], 0.02 * peri, true);
        
        // 获取边界框
        Rect boundRect = boundingRect(conPoly[i]);
        
        // 形状判断
        int objCor = conPoly[i].size();          // 角点数量
        if (objCor == 3)      { /* 三角形 */ }
        else if (objCor == 4) {
            float aspRatio = (float)boundRect.width / boundRect.height;
            if (aspRatio > 0.95 && aspRatio < 1.05) { /* 正方形 */ }
            else                                     { /* 矩形 */ }
        }
        else if (objCor > 4) { /* 圆形 */ }
    }
}
```

**关键参数**：
- `approxPolyDP` 的第三个参数（0.02 × peri）：越大 → 约简越激进，角点越少
- `RETR_EXTERNAL`：只取最外层轮廓；`RETR_TREE`：取所有层级轮廓
- `CHAIN_APPROX_SIMPLE`：压缩水平/垂直/对角线线段，只保留端点

### 10. 人脸检测（基于 Haar 级联）

使用 OpenCV 内置的 Viola-Jones Haar 级联分类器：

```cpp
#include <opencv2/objdetect.hpp>

CascadeClassifier faceCascade;
faceCascade.load("resources/haarcascade_frontalface_default.xml");

vector<Rect> faces;
faceCascade.detectMultiScale(image, faces, 1.1, 10);
// 参数：输入图像, 输出矩形, 缩放因子(每次缩小比例), 最小邻居数(越高越严格)

for (int i = 0; i < faces.size(); i++) {
    rectangle(image, faces[i].tl(), faces[i].br(), Scalar(255, 0, 255), 3);
}
```

**参数含义**：
- **scaleFactor = 1.1**：检测窗口每次放大 10%。越小 → 检测越精细但越慢
- **minNeighbors = 10**：一个候选窗口需要至少 10 个相邻窗口支持才被保留。越高 → 误检越少

### 11. 项目一：虚拟画笔

**核心技术链**：颜色检测 (HSV) → 轮廓检测 → 取轮廓顶部中心点 → 在画布上绘制

关键设计：
- 用 `vector<vector<int>>` 存储多组 HSV 范围（每种颜色对应一组）
- 用 `vector<Scalar>` 存储对应的绘制颜色
- `findColor()` 遍历所有颜色，为每种颜色创建 mask 并查找轮廓
- `getContours()` 返回轮廓顶部中心点 `Point(x, y)`
- 全局 `newPoints` 向量持久存储所有历史点，实现"不擦除"效果

### 12. 项目二：文档扫描器

**核心技术链**：预处理 → 轮廓检测 → 找最大四边形 → 角点重排序 → 透视变换

**角点重排序算法**（项目亮点）：
```
输入：任意顺序的四个角点
1. 对每个点计算 x+y：和最小的是左上(0)，和最大的是右下(3)
2. 对每个点计算 x-y（或 y-x）：差最小的是右上(1)，差最大的是左下(2)
输出：[左上, 右上, 左下, 右下]
```

这是透视变换前的关键步骤——`getPerspectiveTransform` 要求源点和目标点**一一对应**，顺序不对则变换结果不可预期。

### 13. 项目三：车牌检测器

**核心技术**：Haar 级联分类器 + 图像裁剪保存

```cpp
CascadeClassifier plateCascade;
plateCascade.load("resources/haarcascade_russian_plate_number.xml");

// 在 while(true) 循环中对每帧：
plateCascade.detectMultiScale(img, plates, 1.1, 10);
for (Rect plate : plates) {
    rectangle(img, plate.tl(), plate.br(), Scalar(255, 0, 255), 5);
    Mat imgCrop = img(plate);
    imwrite("resources/plates/" + to_string(count++) + ".png", imgCrop);
}
```

## 关键引用

| 引用 | 上下文 |
|------|--------|
| "OpenCV 默认使用的颜色空间是 BGR" | 最常被忽视的坑 |
| "视频是一系列连续的图像帧" | 视频处理的核心理解 |
| "模糊通常在边缘检测前使用，以减少噪声" | 预处理管道设计原则 |
| "HSV 使颜色检测对光照变化更鲁棒" | 为什么不用 BGR 做颜色检测 |

## 知识缺口（本源的局限）

- **Canny 算法的理论基础**：笔记只给出了阈值参数，未解释双阈值滞后（hysteresis thresholding）机制
- **高斯模糊的数学**：只给核大小和 sigma，未解释高斯核的生成和卷积过程
- **Viola-Jones 原理**：使用 Haar 级联但不解释积分图（integral image）、AdaBoost 训练、级联结构
- **缺少性能考量**：没有讨论各操作的复杂度或实时性优化
- **所有检测基于传统 CV**：未涉及深度学习方案（YOLO、SSD 等）

> 上述缺口在对应的 Wiki 概念页面中做了补充拓展。

## 相关页面

- [[Robot_Learning/entities/OpenCV|OpenCV]] — 本源的母体库
- [[Robot_Learning/concepts/OpenCV/图像简介|图像简介]] — 像素、BGR、Mat 类型
- [[Robot_Learning/concepts/OpenCV/第6章 - 颜色检测|第6章 - 颜色检测]] — HSV、inRange 深入
- [[Robot_Learning/concepts/OpenCV/第2章 - 基本功能|第2章 - 基本功能]] — 灰度化、高斯模糊、Canny
- [[Robot_Learning/concepts/OpenCV/第7章 - 形状轮廓检测|第7章 - 形状/轮廓检测]] — findContours、approxPolyDP、Hierarchy
- [[Robot_Learning/concepts/OpenCV/第5章 - 透视变形|第5章 - 透视变形]] — 单应性矩阵、透视变换
- [[Robot_Learning/concepts/OpenCV/第8章 - 人脸检测|第8章 - 人脸检测]] — Haar 级联、Viola-Jones

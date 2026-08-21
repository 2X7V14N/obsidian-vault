---
tags: [entity, library, opencv, cpp, computer-vision]
created: 2026-06-04
updated: 2026-06-04
status: budding
domain: robot-learning
---

# OpenCV

**OpenCV**（Open Source Computer Vision Library）是 Intel 于 1999 年启动、2000 年首次发布的开源计算机视觉库。它是机器人感知领域最核心的工具库之一，提供从图像 I/O 到底层处理、特征提取到目标检测的完整管线。

## 核心架构

OpenCV 按功能模块组织，每个模块对应一组头文件和库文件：

| 模块 | 头文件示例 | 主要功能 |
|------|-----------|---------|
| **core** | `<opencv2/core.hpp>` | 核心数据结构（`Mat`、`Point`、`Scalar`、`Rect`）、矩阵运算 |
| **imgproc** | `<opencv2/imgproc.hpp>` | 图像处理（滤波、边缘检测、形态学、轮廓、颜色转换、几何变换） |
| **imgcodecs** | `<opencv2/imgcodecs.hpp>` | 图像文件读写（`imread`、`imwrite`） |
| **highgui** | `<opencv2/highgui.hpp>` | GUI 窗口、滑动条（`imshow`、`createTrackbar`） |
| **video** | `<opencv2/video.hpp>` | 视频分析（光流、背景减除、目标跟踪） |
| **objdetect** | `<opencv2/objdetect.hpp>` | 目标检测（Haar 级联、HOG + SVM） |
| **calib3d** | `<opencv2/calib3d.hpp>` | 相机标定、3D 重建、立体视觉 |
| **features2d** | `<opencv2/features2d.hpp>` | 2D 特征检测与描述（SIFT、ORB、AKAZE） |
| **dnn** | `<opencv2/dnn.hpp>` | 深度学习推理（加载 ONNX/TensorFlow/Caffe 模型） |

## 核心数据类型

### Mat — 图像容器

`Mat` 是 OpenCV 最核心的类，表示一个多维稠密数组（通常为 2D 图像矩阵）。

```cpp
Mat image = imread("path/to/image.jpg");
// image.rows — 高度
// image.cols — 宽度
// image.channels() — 通道数（灰度 = 1, 彩色 = 3）
// image.type() — 数据类型（如 CV_8UC3 = 8位无符号 × 3通道）
```

`Mat` 使用**引用计数**智能内存管理——赋值和拷贝只复制头部，数据共享，避免大图像频繁深拷贝。

### 其他关键类型

| 类型 | 说明 |
|------|------|
| `Point` / `Point2f` | 2D 点（整数/浮点） |
| `Scalar(B, G, R)` | 颜色值，始终 BGR 顺序 |
| `Rect(x, y, w, h)` | 矩形区域 |
| `Size(w, h)` | 尺寸 |
| `vector<Point>` | 轮廓（一组点） |
| `vector<vector<Point>>` | 多轮廓集合 |

### BGR 颜色顺序

这是 OpenCV 最常被忽视的坑——内部使用 **BGR** 而非 RGB：
- `Scalar(255, 0, 0)` = 蓝色
- `Scalar(0, 255, 0)` = 绿色
- `Scalar(0, 0, 255)` = 红色

当与 matplotlib（默认 RGB）或其他库交互时，需用 `cvtColor(image, image, COLOR_BGR2RGB)` 转换。

## OpenCV 在机器人感知中的角色

在典型的机器人感知流水线中，OpenCV 承担：

1. **数据采集**：从摄像头、深度传感器获取图像帧
2. **预处理**：去噪（高斯模糊）、归一化、颜色空间转换
3. **特征提取**：边缘检测（Canny）、角点检测（Harris）、关键点（SIFT/ORB）
4. **目标检测与跟踪**：Haar 级联、HOG+SVM、或加载 DNN 模型
5. **几何处理**：透视变换（文档扫描）、相机标定（calib3d）
6. **可视化**：绘制检测框、关键点、路径

在 [[wiki/concepts/LLM Wiki|LLM Wiki]] 的方法论视角下，OpenCV 是机器人感知领域的"基础设施级"实体——几乎所有更高级的感知概念（SLAM、3D 重建、物体位姿估计）都依赖于 OpenCV 提供的底层操作。

## 安装与版本

- **C++ 版本**：下载预编译包或从源码构建，Visual Studio 中配置 include/lib 路径和链接器依赖
- **Python 版本**：`pip install opencv-python`（最常用）
- **主要版本**：OpenCV 3.x → 4.x → 5.x（当前主流为 4.x）
- **contrib 模块**：`opencv-contrib-python` 包含非官方但实用的扩展（如 SIFT、SURF 等专利算法在 4.x 后移入 contrib）

## 相关页面

- [[Robot_Learning/concepts/OpenCV/图像简介|图像简介]] — Mat、BGR 基础
- [[Robot_Learning/concepts/OpenCV/第2章 - 基本功能|第2章 - 基本功能]] — OpenCV 的 imgproc 模块核心应用
- [[Robot_Learning/concepts/OpenCV/第7章 - 形状轮廓检测|第7章 - 形状/轮廓检测]] — findContours、Hierarchy 深入分析
- [[Robot_Learning/concepts/OpenCV/第8章 - 人脸检测|第8章 - 人脸检测]] — OpenCV 内置 Haar 级联分类器
- [[Robot_Learning/sources/4h上手C++版Opencv 源|OpenCV 课程笔记]] — 本页面的主要源

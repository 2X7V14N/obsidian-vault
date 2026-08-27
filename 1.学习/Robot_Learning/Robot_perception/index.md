---
tags: [domain, index, robot-learning]
created: 2026-06-04
updated: 2026-07-10
status: budding
domain: robot-learning
---

# Robot Learning · 学习计划与知识索引

> 机器人视觉与控制学习路线——从 C++ 基础到具身智能。
>
> 📋 详细要求与说明见 [[Robot_Learning/Robot Learning Plan|学习计划详情页]]

---

## 学习进度总览

### 语言与工程基础

| 任务 | 优先级 | 状态 | 进度 |
|------|:------:|:----:|------|
| C++ 基础复习 | P0 | 🔄 | `██████░░░░` 60% |
| C++11/14 常用特性 | P1 | ⬜ | `░░░░░░░░░░` 0% |
| CMake | P0 | 🔄 | `███░░░░░░░` 30% |
| Git / GitHub | P0 | ⬜ | `░░░░░░░░░░` 0% |
| Ubuntu / Linux 基础 | P0 | ⬜ | `░░░░░░░░░░` 0% |

### 视觉算法

| 任务 | 优先级 | 状态 | 进度 |
|------|:------:|:----:|------|
| OpenCV C++ | P0 | ✅ | `████████░░` 80% |
| 相机标定 | P1 | ⬜ | `░░░░░░░░░░` 0% |
| ArUco / AprilTag | P1 | ⬜ | `░░░░░░░░░░` 0% |

### 项目实战

| 任务 | 优先级 | 状态 | 进度 |
|------|:------:|:----:|------|
| OpenCV 抓取位姿估计 V1 | P0 | 🔄 | `██░░░░░░░░` 20% |
| GitHub 项目整理 | P0 | ⬜ | `░░░░░░░░░░` 0% |
| OpenCV 抓取位姿估计 V2 | P1 | ⬜ | `░░░░░░░░░░` 0% |
| 视觉引导机械臂仿真抓取 V3 | P1 | ⬜ | `░░░░░░░░░░` 0% |
| 深度学习目标检测版抓取 V4 | P2 | ⬜ | `░░░░░░░░░░` 0% |

### 机器人学

| 任务 | 优先级 | 状态 | 进度 |
|------|:------:|:----:|------|
| 坐标系与齐次变换 | P0 | ⬜ | `░░░░░░░░░░` 0% |
| 姿态表达 | P0 | ⬜ | `░░░░░░░░░░` 0% |
| 运动学快速复习 | P0 | ⬜ | `░░░░░░░░░░` 0% |
| 雅可比与视觉伺服 | P1 | ⬜ | `░░░░░░░░░░` 0% |

### 数学与控制

| 任务 | 优先级 | 状态 | 进度 |
|------|:------:|:----:|------|
| 线性代数复习 | P1 | ⬜ | `░░░░░░░░░░` 0% |
| PID 与闭环控制 | P1 | ⬜ | `░░░░░░░░░░` 0% |
| 概率论基础 | P2 | ⬜ | `░░░░░░░░░░` 0% |

### 机器人软件（ROS2）

| 任务 | 优先级 | 状态 | 进度 |
|------|:------:|:----:|------|
| ROS2 基础 | P1 | 🔄 | `█░░░░░░░░░` 15% |
| TF2 坐标变换 | P1 | ⬜ | `░░░░░░░░░░` 0% |
| MoveIt2 | P1 | ⬜ | `░░░░░░░░░░` 0% |

### 算法与深度学习

| 任务 | 优先级 | 状态 | 进度 |
|------|:------:|:----:|------|
| LeetCode 热题 100 | P1 | 🔄 | `█░░░░░░░░░` 10% |
| PyTorch 基础 | P2 | 🔄 | `██░░░░░░░░` 20% |
| 目标检测 / 分割 | P2 | ⬜ | `░░░░░░░░░░` 0% |

### 仿真与 3D 感知

| 任务 | 优先级 | 状态 | 进度 |
|------|:------:|:----:|------|
| Gazebo / MuJoCo / Isaac Sim | P2 | ⬜ | `░░░░░░░░░░` 0% |
| PCL 点云处理 | P2 | ⬜ | `░░░░░░░░░░` 0% |

### 扩展与前沿

| 任务 | 优先级 | 状态 | 进度 |
|------|:------:|:----:|------|
| SLAM 十四讲 | P2 | ⬜ | `░░░░░░░░░░` 0% |
| LeRobot | P3 | ⬜ | `░░░░░░░░░░` 0% |
| Diffusion Policy | P3 | ⬜ | `░░░░░░░░░░` 0% |
| OpenVLA / RT 系列 | P3 | ⬜ | `░░░░░░░░░░` 0% |
| 机器人抓取/视觉伺服论文 | P3 | ⬜ | `░░░░░░░░░░` 0% |

> **图例**：✅ 基本完成 🔄 进行中 ⬜ 未开始 `████████░░` = 已完成/总计

---

## OpenCV 课程笔记

按课程章节组织的系统化学习笔记：

| 章节 | 页面 | 内容 |
|------|------|------|
| 概述 | [[Robot_Learning/concepts/OpenCV/介绍\|介绍]] | OpenCV 是什么、课程覆盖范围 |
| 基础 | [[Robot_Learning/concepts/OpenCV/图像简介\|图像简介]] | 像素、分辨率、BGR、Mat 类型 |
| 安装 | [[Robot_Learning/concepts/OpenCV/安装 - Windows\|安装 - Windows]] | Visual Studio + 预编译包配置 |
| 安装 | [[Robot_Learning/concepts/OpenCV/安装 - Mac\|安装 - Mac]] | Homebrew / 源码编译 |
| 第1章 | [[Robot_Learning/concepts/OpenCV/第1章 - 读取图像、视频和摄像头\|读取图像、视频和摄像头]] | imread、imshow、VideoCapture |
| 第2章 | [[Robot_Learning/concepts/OpenCV/第2章 - 基本功能\|基本功能]] | 灰度化、高斯模糊、Canny、形态学 |
| 第3章 | [[Robot_Learning/concepts/OpenCV/第3章 - 调整大小和裁剪\|调整大小和裁剪]] | resize、ROI 裁剪 |
| 第4章 | [[Robot_Learning/concepts/OpenCV/第4章 - 绘制形状和文字\|绘制形状和文字]] | circle、rectangle、line、putText |
| 第5章 | [[Robot_Learning/concepts/OpenCV/第5章 - 透视变形\|透视变形]] | 单应性矩阵、透视变换、角点重排序 |
| 第6章 | [[Robot_Learning/concepts/OpenCV/第6章 - 颜色检测\|颜色检测]] | HSV 颜色空间、inRange、Trackbar 调参 |
| 第7章 | [[Robot_Learning/concepts/OpenCV/第7章 - 形状轮廓检测\|形状/轮廓检测]] | findContours + 轮廓层级拓扑（Hierarchy） |
| 第8章 | [[Robot_Learning/concepts/OpenCV/第8章 - 人脸检测\|人脸检测]] | Haar 级联 + FaceDetectorYN / YuNet DNN |
| 项目1 | [[Robot_Learning/concepts/OpenCV/项目1 - 虚拟画师\|虚拟画师]] | 颜色检测 + 轮廓追踪 + 实时绘画 |
| 项目2 | [[Robot_Learning/concepts/OpenCV/项目2 - 文档扫描仪\|文档扫描仪]] | 轮廓检测 + 透视变换 + 文档矫正 |
| 项目3 | [[Robot_Learning/concepts/OpenCV/项目3 - 车牌检测器\|车牌检测器]] | Haar 级联 + 车牌裁剪保存 |

**工程手册**：[[Robot_Learning/concepts/OpenCV/OpenCV_CheatSheet\|OpenCV CheatSheet]] — C++ 函数速查 + 任务流程 + 排错

---

## 深度学习课程笔记

基于《动手学深度学习》（李沐）的系统学习笔记：

| 章节 | 页面 | 内容 |
|------|------|------|
| Ch01 | [[Ch01 深度学习概论\|深度学习概论]] | ML/DL 定义、四大核心组件、各类问题、历史与成功案例 |
| Ch02 | [[Ch02 深度学习的基础设施\|深度学习的基础设施]] | 张量操作、广播机制、pandas 预处理、矩阵乘法、范数 |
| Ch03 | [[Ch03 线性神经网络\|线性神经网络]] | 线性回归模型、解析解与 SGD、从零实现+框架实现、矢量化 |
| Ch04 | [[Ch04 多层感知机\|多层感知机]] | 隐藏层+非线性、ReLU/Sigmoid/Tanh、过拟合与正则化（权重衰减/Dropout）、前向/反向传播、Xavier初始化、分布偏移 |
| Ch05 | [[Ch05 深度学习计算\|深度学习计算]] | nn.Module 块抽象、参数管理（访问/初始化/绑定）、延后初始化、自定义层、模型持久化、GPU 计算 |
| Ch06 | [[Ch06 卷积神经网络\|卷积神经网络]] | 平移不变性+局部性、互相关/填充/步幅、多通道与 $1 \times 1$ 卷积、汇聚层、LeNet |
| Ch07 | [[Ch07 现代卷积神经网络\|现代卷积神经网络]] | AlexNet→VGG→GoogLeNet→BN→ResNet→DenseNet：更深、更宽、跳跃连接、通道拼接 |
| Ch08 | [[Ch08 循环神经网络\|循环神经网络]] | 序列模型、隐状态、RNN 公式、困惑度、BPTT、梯度裁剪、字符级语言模型 |
| Ch09 | [[Ch09 现代循环神经网络\|现代循环神经网络]] | GRU/LSTM 门控、深度/双向 RNN、编码器-解码器、seq2seq、BLEU、束搜索 |
| Ch10 | [[Ch10 注意力机制\|注意力机制]] | 查询-键-值、评分函数、Bahdanau 注意力、多头注意力、自注意力+位置编码、Transformer |
| Ch13 | [[Ch13 计算机视觉\|计算机视觉]] | 图像增广/微调、锚框/IoU/NMS、SSD/R-CNN 系列、语义分割/FCN/转置卷积 |

**学习规划**：
- [[Deep Learning Plan\|深度学习课程学习计划]] — 完整学习路线 + 阶段性完成标准
- [[1_学习/Robot_Learning/Robot_perception/concepts/深度学习/深度学习课程学习清单\|深度学习课程学习清单]] — 197 集课程分类清单

---

## ROS2 课程笔记

基于《ROS2 入门 21 讲》（古月居）的系统学习笔记：

| 章节 | 页面 | 内容 |
|------|------|------|
| Ch01 | [[concepts/ROS2/Ch01 认识 ROS 与 ROS2\|认识 ROS 与 ROS2]] | 机器人三个时代、ROS 诞生（PR2）、发展历程、核心目标与社区 |
| Ch02 | [[concepts/ROS2/Ch02 ROS2 与 ROS1 的对比\|ROS2 与 ROS1 的对比]] | ROS1 局限、ROS2 六大需求、推倒重来、DDS 通信与 RMW、架构对比 |
| Ch03 | [[concepts/ROS2/Ch03 ROS2 环境安装与配置\|ROS2 环境安装与配置]] | Linux/Ubuntu、虚拟机安装、ROS2 Humble 安装、talker/listener 测试、Git+VSCode |
| Ch04 | [[concepts/ROS2/Ch04 ROS2 命令行操作\|ROS2 命令行操作]] | Linux 命令、ros2 run/node/topic/service/action/bag 实战 |
| Ch05 | [[concepts/ROS2/Ch05 核心概念：工作空间与功能包\|核心概念：工作空间与功能包]] | 概念总览、工作空间四子空间、colcon 编译、rosdep、功能包结构 |

---

## 实体 (Entities)

- [[Robot_Learning/entities/OpenCV\|OpenCV]] — 开源计算机视觉库，核心架构、数据类型与模块概览

## 源 (Sources)

- [[Robot_Learning/sources/4h上手C++版Opencv 源\|4h 上手 C++ 版 OpenCV 源]] — 基于 Murtaza's Workshop 课程的系统笔记
- [[Robot_Learning/sources/OpenCV 轮廓提取与层级拓扑完全指南 源\|轮廓层级拓扑完全指南 源]] — 对轮廓检测和 Hierarchy 的深度解析

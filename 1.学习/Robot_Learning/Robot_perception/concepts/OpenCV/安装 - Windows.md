---
tags: [opencv, setup, windows, installation]
created: 2026-07-04
updated: 2026-07-04
status: budding
domain: robot-learning
---

# 安装 - Windows

在 Windows 上使用 OpenCV C++ 开发，推荐 Visual Studio + 预编译 OpenCV 库的方案。

## 步骤概览

1. 安装 Visual Studio（含 C++ 桌面开发工作负载）
2. 下载 OpenCV 预编译包
3. 配置系统环境变量
4. 在 Visual Studio 中配置项目的 include/lib 路径
5. 验证安装

## 详细步骤

### 1. 安装 Visual Studio

下载 [Visual Studio Community](https://visualstudio.microsoft.com/)（免费），安装时勾选"**使用 C++ 的桌面开发**"工作负载。这会安装 MSVC 编译器、Windows SDK 和必要的构建工具。

### 2. 下载 OpenCV

从 [OpenCV 官方 Release 页面](https://github.com/opencv/opencv/releases) 下载 Windows 预编译包（`opencv-4.x.x-windows.exe`）。运行后解压到 `C:\opencv\`。

解压后目录结构：
```
C:\opencv\
  build\
    include\          ← 头文件
    x64\vc16\         ← Visual Studio 2022 的库文件
      bin\            ← DLL 文件
      lib\            ← .lib 文件
```

### 3. 配置环境变量

将 `C:\opencv\build\x64\vc16\bin` 添加到系统 `PATH` 环境变量，使程序运行时能找到 OpenCV 的 DLL 文件。

### 4. 配置 Visual Studio 项目

在项目中需要做两处配置（每个新项目都要配置一次）：

**包含目录**（让编译器找到头文件）：
```
项目属性 → C/C++ → 常规 → 附加包含目录
→ 添加 C:\opencv\build\include
```

**库目录 + 链接器依赖**（让链接器找到库文件）：
```
项目属性 → 链接器 → 常规 → 附加库目录
→ 添加 C:\opencv\build\x64\vc16\lib

项目属性 → 链接器 → 输入 → 附加依赖项
→ 添加 opencv_world4xx.lib（Release）或 opencv_world4xxd.lib（Debug）
```

> `opencv_world` 模块将所有 OpenCV 功能打包在一个库文件中。如果只用部分功能，也可以链接单独的模块（如 `opencv_core4xx.lib`、`opencv_imgproc4xx.lib` 等）。

### 5. 验证安装

创建新项目，运行以下测试代码：

```cpp
#include <opencv2/opencv.hpp>
#include <iostream>

int main() {
    cv::Mat img(200, 300, CV_8UC3, cv::Scalar(0, 255, 0));
    std::cout << "OpenCV version: " << CV_VERSION << std::endl;
    cv::imshow("Test", img);
    cv::waitKey(0);
    return 0;
}
```

如果能显示一个绿色窗口并输出 OpenCV 版本号，安装成功。

## 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| `无法找到 opencv_world4xx.dll` | PATH 未配置 | 添加 bin 目录到系统 PATH |
| `无法解析的外部符号` | 链接器依赖未配置 | 检查 .lib 文件名和附加依赖项 |
| Debug/Release 混用 | Debug 程序链接了 Release 的 .lib | Debug 用 `opencv_world4xxd.lib`（注意结尾的 `d`） |

## 相关页面

- [[Robot_Learning/concepts/OpenCV/安装 - Mac|安装 - Mac]] — macOS 平台安装指南
- [[Robot_Learning/concepts/OpenCV/介绍|OpenCV 介绍]] — 回到课程总览

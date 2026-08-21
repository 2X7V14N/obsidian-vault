---
tags: [opencv, setup, mac, installation]
created: 2026-07-04
updated: 2026-07-04
status: budding
domain: robot-learning
---

# 安装 - Mac

在 macOS 上使用 OpenCV C++ 开发，推荐 Homebrew 安装（最简单）或从源码编译（需要更多控制时）。

## 方案一：Homebrew 安装（推荐）

### 1. 安装 Homebrew

如果还没有 Homebrew，先安装：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 2. 安装 OpenCV

```bash
brew install opencv
```

Homebrew 会自动安装 OpenCV 及其依赖。安装后，头文件和库文件位于：

```
/usr/local/include/opencv4/          ← 头文件（Intel Mac）
/opt/homebrew/include/opencv4/       ← 头文件（Apple Silicon Mac）
/usr/local/lib/                       ← 库文件（Intel Mac）
/opt/homebrew/lib/                    ← 库文件（Apple Silicon Mac）
```

### 3. 编译与运行

使用 `pkg-config` 自动获取编译和链接参数：

```bash
# 编译
g++ -std=c++17 main.cpp -o main $(pkg-config --cflags --libs opencv4)

# 运行
./main
```

### 4. 验证安装

```cpp
// test.cpp
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

编译运行后应显示绿色窗口和版本号。

## 方案二：从源码编译

适合需要自定义模块（如 contrib、CUDA 支持）的场景。

```bash
git clone https://github.com/opencv/opencv.git
cd opencv
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j$(nproc)
sudo make install
```

## 使用 Xcode

如果偏好 IDE 开发，可在 Xcode 中创建 Command Line Tool 项目，然后在 Build Settings 中配置：

- **Header Search Paths**: `/usr/local/include/opencv4`（或 `/opt/homebrew/include/opencv4`）
- **Library Search Paths**: `/usr/local/lib`（或 `/opt/homebrew/lib`）
- **Other Linker Flags**: 添加 `-lopencv_core -lopencv_imgproc -lopencv_highgui -lopencv_imgcodecs`

## 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| `opencv2/opencv.hpp not found` | 头文件路径未配置 | 确认 `pkg-config --cflags opencv4` 输出 |
| `Library not loaded` | 动态库路径问题 | 设置 `DYLD_LIBRARY_PATH` 或使用 `install_name_tool` |
| Apple Silicon 路径不同 | Homebrew 在 M1/M2/M3 上使用 `/opt/homebrew` | 替换对应的路径 |

## 相关页面

- [[Robot_Learning/concepts/OpenCV/安装 - Windows|安装 - Windows]] — Windows 平台安装指南
- [[Robot_Learning/concepts/OpenCV/介绍|OpenCV 介绍]] — 回到课程总览

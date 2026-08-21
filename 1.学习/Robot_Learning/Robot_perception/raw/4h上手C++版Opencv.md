> 来源链接：https://www.bilibili.com/video/BV11A411T7rL/?spm_id_from=333.1391.0.0&vd_source=3236ff85e9caddefb17778de051e9c33

## OpenCV C++ 课程笔记

### 课程概述
本课程是基于讲师之前广受欢迎的Python版OpenCV课程的C++版本，旨在教授使用C++进行计算机视觉和图像处理的基础知识与实践技能。课程内容涵盖图像、视频、网络摄像头的处理，以及形状、颜色、人脸和车牌检测等。课程包含三个实战项目，侧重于实际应用，略去了大量枯燥的理论。

**讲师:** 来自频道 "Murtaza's Workshop"。
**课程页面与资源:** 课程相关的代码、资源文件和详细步骤说明可在讲师网站上找到（链接在视频描述中）。

---

## 1. 图像基础与OpenCV安装配置

### 1.1 图像的数字表示
- **像素的概念**: 图像由称为像素的小方格组成。分辨率（如VGA: 640x480, HD: 1280x720）代表了图像的像素数量（宽度 x 高度）。
- **二值图像**: 仅包含黑色(0)和白色(1)两种颜色的图像。
- **灰度图像**: 每个像素用一个8位无符号整数表示，范围是0(黑)到255(白)，共256个灰度级别。
- **彩色图像 (RGB/BGR)**: 由红(Red)、绿(Green)、蓝(Blue)三个通道的灰度图像叠加而成。
    - **注意:** OpenCV默认使用的颜色空间是BGR（蓝, 绿, 红），而不是常见的RGB。

### 1.2 Windows环境配置 (Visual Studio)
1.  **安装 Visual Studio**: 下载并安装Visual Studio社区版，安装时务必勾选“使用C++的桌面开发”工作负载。
2.  **下载 OpenCV**: 从OpenCV的GitHub发布页面下载可执行文件（.exe）并安装到指定目录（如 `D:\`）。
3.  **设置环境变量**: 将OpenCV的 `bin` 文件夹路径（例如 `D:\opencv\build\x64\vc15\bin`）添加到系统环境变量 `Path` 中，并重启电脑。
4.  **创建并配置Visual Studio项目**:
    - 创建一个新的空C++项目。
    - 将解决方案平台从 `x86` 改为 `x64`。
    - 进入项目属性 (`Project` -> `Properties`)：
        - **VC++ 目录 (`VC++ Directories`)**:
            - **包含目录 (`Include Directories`)**: 添加OpenCV的`include`目录路径。
            - **库目录 (`Library Directories`)**: 添加OpenCV的`lib`目录路径。
        - **链接器 (`Linker`) -> 输入 (`Input`)**:
            - 在`附加依赖项`中添加 `opencv_world<版本号>.lib` (例如 `opencv_world453.lib`)。
            - **注意:** 文件名后缀带 `d`（如 `opencv_world453d.lib`）的是调试（Debug）版本，不带的是发布（Release）版本。

### 1.3 macOS环境配置 (Xcode)
1.  **安装 Xcode**: 从App Store安装Xcode。
2.  **安装 Homebrew**: 在终端中运行官网提供的安装命令。
3.  **安装 OpenCV**: 在终端运行 `brew install opencv`。
4.  **创建并配置Xcode项目**:
    - 创建一个新的macOS命令行工具项目，语言选择C++。
    - 右键项目，选择“Add Files to...”，使用快捷键 `/` 导航到 `/usr/local/Cellar/opencv/<版本号>/lib`，选中所有 `.dylib` 动态库文件并添加。
    - 在项目 `Build Settings` 中搜索 `Header Search Paths`，添加路径 `/usr/local/Cellar/opencv/<版本号>/include/opencv4` 并将其设置为递归(`recursive`)。
5.  **资源访问权限**:
    - 将包含测试图片等资料的 `resources` 文件夹拖拽到Xcode项目目录中。
    - 为了使用摄像头，需创建一个 `Info.plist` 文件，添加 `Privacy - Camera Usage Description` 键和相应的描述字符串，并将该文件拖拽到 `Products` 目录下的可执行文件同级位置。

---

## 2. 基础功能：读取、显示与处理

### 2.1 核心代码结构
所有C++代码都需要包含以下基础头文件和命名空间声明：
```cpp
#include <opencv2/imgcodecs.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/imgproc.hpp>
#include <iostream>

using namespace cv;
using namespace std;
```
- `imgcodecs.hpp`: 用于读取和写入图像。
- `highgui.hpp`: 用于创建窗口和显示图像。
- `imgproc.hpp`: 包含图像处理函数。
- `iostream`: C++标准输入输出流。
- 一个文件中只能有一个 `main` 函数。

### 2.2 导入与显示图像
```cpp
string path = "resources/test.png";
Mat image = imread(path); // Mat是OpenCV用于存储图像的基本数据类型
imshow("Image", image);
waitKey(0); // 0表示无限等待按键，否则窗口会立即关闭
```

### 2.3 导入与播放视频
视频是一系列连续的图像帧。需要通过循环逐帧读取和显示。
```cpp
string path = "resources/test_video.mp4";
VideoCapture cap(path);
Mat image;

while (true) {
    cap.read(image); // 从视频对象cap读取当前帧到image
    imshow("Video", image);
    waitKey(1); // 延迟1毫秒。数值越大，播放越慢
}
```

### 2.4 使用网络摄像头
原理与导入视频类似，但 `VideoCapture` 的参数是摄像头的设备ID。
```cpp
VideoCapture cap(0); // 0通常代表笔记本自带摄像头，1, 2...代表外接摄像头
Mat image;

while (true) {
    cap.read(image);
    imshow("Webcam", image);
    waitKey(1);
}
```

---

## 3. 基本图像处理函数

在读取图像后，可以对 `Mat` 对象进行一系列处理。

### 3.1 灰度化 (CvtColor)
将彩色图像转换为灰度图像。
```cpp
Mat imageGray;
cvtColor(image, imageGray, COLOR_BGR2GRAY);
```

### 3.2 高斯模糊 (Gaussian Blur)
用于减少图像噪声和细节。
```cpp
Mat imageBlur;
GaussianBlur(imageGray, imageBlur, Size(7, 7), 5, 0); 
// Size(7,7)是核大小，必须是奇数。sigmaX=5, sigmaY=0（自动计算）
```

### 3.3 Canny边缘检测
检测图像中的边缘。通常在对图像进行模糊后使用效果更佳。
```cpp
Mat imageCanny;
Canny(imageBlur, imageCanny, 50, 150); // 阈值1和阈值2，比值通常为1:2或1:3
```

### 3.4 图像膨胀 (Dilation) 与侵蚀 (Erosion)
常用于处理边缘检测后的图像，以闭合断裂的线条或加粗/细化边缘。需要先用 `getStructuringElement` 创建一个结构元素（核）。
```cpp
Mat kernel = getStructuringElement(MORPH_RECT, Size(5, 5)); // 矩形核，大小5x5

Mat imageDilate;
dilate(imageCanny, imageDilate, kernel);

Mat imageErode;
erode(imageDilate, imageErode, kernel);
```

---

## 4. 调整大小与裁剪

### 4.1 调整图像大小 (Resize)
```cpp
Mat imageResize;
resize(image, imageResize, Size(640, 480)); // 指定确切尺寸，可能改变宽高比

// 或按比例缩放，更常用
resize(image, imageResize, Size(), 0.5, 0.5); // fx, fy = 0.5 表示缩小一半
```
可以通过 `image.size()` 获取当前图像尺寸并打印： `cout << image.size() << endl;`

### 4.2 裁剪图像 (Crop)
利用矩形区域（Region of Interest，ROI）在图像上实现裁剪。
```cpp
Rect roi(100, 100, 300, 250); // x, y, width, height
Mat imageCrop = image(roi);
```

---

## 5. 绘制形状与文本

可以在一个空白画布或现有图像上进行绘制。

### 5.1 创建空白图像
```cpp
// 创建一个512x512像素，3通道，8位无符号整数的彩色图像
Mat img(512, 512, CV_8UC3, Scalar(255, 255, 255)); // Scalar(B, G, R) = 白色背景
```

### 5.2 绘制圆形 (Circle)
```cpp
Point center(256, 256); // 圆心
circle(img, center, 155, Scalar(0, 69, 255), FILLED); // 填充的橙色圆。FILLED 或 -1 表示填充
```
- `Scalar(B, G, R)` 用于定义颜色。

### 5.3 绘制矩形 (Rectangle)
有两种方法：
1.  使用 `Rect` 数据定义：`rectangle(img, Rect(130, 226, 382, 286), Scalar(255, 255, 255), FILLED);`
2.  使用两个对角点定义：`rectangle(img, Point(130, 226), Point(382, 286), Scalar(255, 255, 255), FILLED);`
    - **注意:** 前者是x, y, width, height；后者是两个顶点的坐标。

### 5.4 绘制直线 (Line)
```cpp
Point p1(130, 296);
Point p2(382, 296);
line(img, p1, p2, Scalar(255, 255, 255), 2);
```

### 5.5 添加文本 (PutText)
```cpp
Point textOrg(137, 262);
putText(img, "Murtaza's Workshop", textOrg, FONT_HERSHEY_PLAIN, 0.75, Scalar(0, 69, 255), 2);
```

---

## 6. 图像透视变换 (Warp Perspective)

用于从图像中提取一个平面（如卡片、纸张），并将其转换为鸟瞰视角。

1.  **定义源点**: 从图像中找到要提取物体的四个角点坐标，存储在 `vector<Point2f>` 中。
    ```cpp
    vector<Point2f> src = { {529,142}, {771,190}, {405,395}, {674,457} };
    ```
2.  **定义目标点**: 定义最终输出矩形图像的四个角点，顺序与源点一一对应。
    ```cpp
    float width = 250, height = 350;
    vector<Point2f> dst = { {0.0f,0.0f}, {width,0.0f}, {0.0f,height}, {width,height} };
    ```
3.  **计算变换矩阵**:
    ```cpp
    Mat matrix = getPerspectiveTransform(src, dst);
    ```
4.  **应用透视变换**:
    ```cpp
    Mat imageWarp;
    warpPerspective(image, imageWarp, matrix, Point(width, height));
    ```
5.  **可视化源点**: 可在原图上绘制圆形以便检查选点是否正确。
    ```cpp
    for (int i = 0; i < src.size(); i++) {
        circle(image, src[i], 10, Scalar(0, 0, 255), FILLED);
    }
    ```

---

## 7. 颜色检测

一种快速检测特定颜色物体的方法。在HSV颜色空间中操作比在BGR中更容易。

### 7.1 基本流程
1.  **转换为HSV**: `cvtColor(image, imageHSV, COLOR_BGR2HSV);`
2.  **定义目标颜色范围**: 为Hue(H)，Saturation(S)，Value(V)通
3.  **创建掩膜 (Mask)**: 使用 `inRange` 函数。
    ```cpp
    Scalar lower(h_min, s_min, v_min);
    Scalar upper(h_max, s_max, v_max);
    Mat mask;
    inRange(imageHSV, lower, upper, mask);
    ```
    掩膜是一幅二值图像，白色区域表示颜色在指定范围内的像素。

### 7.2 使用滑动条 (Trackbars) 实时调色
手动寻找最佳HSV范围值很困难。通过创建滑动条可以实时调整并观察效果。
- **创建窗口**: `namedWindow("Trackbars", Size(640, 200));`
- **创建滑动条**:
  ```cpp
  int h_min = 0, s_min = 0, v_min = 0;
  int h_max = 179, s_max = 255, v_max = 255;
  createTrackbar("Hue Min", "Trackbars", &h_min, 179);
  createTrackbar("Sat Min", "Trackbars", &s_min, 255);
  createTrackbar("Val Min", "Trackbars", &v_min, 255);
  createTrackbar("Hue Max", "Trackbars", &h_max, 179);
  createTrackbar("Sat Max", "Trackbars", &s_max, 255);
  createTrackbar("Val Max", "Trackbars", &v_max, 255);
  ```
- 在 `while(true)` 循环中，不断用滑动条变量的当前值去更新 `lower` 和 `upper` Scaler，并生成 `mask`，即可实现实时调节。

---

## 8. 形状/轮廓检测

用于检测图像中的形状（三角形、矩形、圆形等）。

### 8.1 检测流程
1.  **图像预处理**:
    - 转换为灰度图 -> 高斯模糊 -> Canny边缘检测 -> 膨胀处理（使边缘闭合）
2.  **查找轮廓**: 使用 `findContours` 函数。
    ```cpp
    vector<vector<Point>> contours;
    vector<Vec4i> hierarchy; // Vec4i 是包含4个整数的向量
    findContours(imageDilate, contours, hierarchy, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE);
    ```
3.  **遍历轮廓并过滤**: 基于面积过滤掉噪声。
    ```cpp
    for (int i = 0; i < contours.size(); i++) {
        int area = contourArea(contours[i]);
        if (area > 1000) { // 过滤掉面积小于1000的轮廓
            // ... 进一步处理
        }
    }
    ```
4.  **绘制轮廓或边界框**:
    - 绘制轮廓： `drawContours(image, contours, i, Scalar(255, 0, 255), 2);`
5.  **获取轮廓的边界框**:
    ```cpp
    vector<Rect> boundRect(contours.size());
    // ...
    boundRect[i] = boundingRect(contours[i]);
    rectangle(image, boundRect[i].tl(), boundRect[i].br(), Scalar(0, 255, 0), 5);
    ```

### 8.2 形状识别
通过近似多边形来推测形状。
```cpp
vector<vector<Point>> conPoly(contours.size()); // 存储近似多边形
float peri = arcLength(contours[i], true); // 计算轮廓周长
approxPolyDP(contours[i], conPoly[i], 0.02 * peri, true); // 进行多边形近似

int objCor = (int)conPoly[i].size(); // 获取角点数量
if (objCor == 3) { /* 三角形 */ }
else if (objCor == 4) { 
    // 进一步判断是矩形还是正方形
    float aspRatio = (float)boundRect[i].width / (float)boundRect[i].height;
    if (aspRatio > 0.95 && aspRatio < 1.05) { /* 正方形 */ }
    else { /* 矩形 */ }
}
else if (objCor > 4) { /* 圆形 */ }
```
可以在检测到的形状附近添加文本 `string objectType = "Triangle";` 进行标注。

---

## 9. 人脸检测

使用OpenCV内置的基于Viola-Jones方法的Haar级联分类器。

1.  **包含头文件**: `#include <opencv2/objdetect.hpp>`
2.  **加载分类器**:
    ```cpp
    CascadeClassifier faceCascade;
    faceCascade.load("resources/haarcascade_frontalface_default.xml");
    ```
3.  **执行检测**:
    ```cpp
    vector<Rect> faces;
    faceCascade.detectMultiScale(image, faces, 1.1, 10);
    // 参数: 图像, 输出矩形向量, 缩放因子, 最小邻居数
    ```
4.  **绘制结果**: 遍历 `faces` 向量，在图像上画出矩形。
    ```cpp
    for (int i = 0; i < faces.size(); i++) {
        rectangle(image, faces[i].tl(), faces[i].br(), Scalar(255, 0, 255), 3);
    }
    ```

---

## 10. 项目实战

### 10.1 项目一：虚拟画笔 (Virtual Painter)
**目标**: 使用彩色物体（如笔帽）作为画笔，在摄像头画面中实时绘制。
**核心技术**: 颜色检测 (HSV) + 轮廓检测。
**实现步骤**:
1.  **定义颜色和对应的显示颜色**: 创建 `vector<vector<int>>` 存储要检测颜色的HSV范围，创建 `vector<Scalar>` 存储对应的绘制颜色。
    ```cpp
    vector<vector<int>> myColors = { {0, 0, 0, 179, 255, 255}, /* 紫色范围 */ 
                                     {38, 76, 0, 70, 255, 255} /* 绿色范围 */ };
    vector<Scalar> myColorValues = { {255, 0, 255}, /* 紫色 */ {0, 255, 0} /* 绿色 */ };
    ```
2.  **创建 `findColor` 函数**: 输入图像，遍历 `myColors`，为每个颜色创建掩膜，并在该掩膜上调用 `getContours` 函数。
3.  **创建 `getContours` 函数**: 输入掩膜图像，查找轮廓，找到轮廓顶部中心点并返回。
    ```cpp
    Point getContours(Mat imgDil) {
        // ... 查找轮廓 ...
        // 假设找到了符合面积条件的轮廓
        int x = boundRect[i].x + boundRect[i].width / 2;
        int y = boundRect[i].y; // 顶部y坐标
        return Point(x, y);
    }
    ```
4.  **创建全局点向量**: `vector<vector<int>> newPoints;` 用于存储所有绘制点及其颜色索引（X, Y, ColorIndex）。
5.  **创建 `drawOnCanvas` 函数**: 遍历 `newPoints`，根据其颜色索引从 `myColorValues` 获取颜色，并在图像上画填充圆。

### 10.2 项目二：文档扫描器 (Document Scanner)
**目标**: 从图像中检测出A4纸，并将其矫正为平整的扫描件效果。
**核心技术**: 预处理 (Canny) -> 轮廓检测 -> 透视变换 (Warp Perspective)。
**实现步骤**:
1.  **预处理 (`PreProcessing`)**:
    - 灰度化 -> 高斯模糊 -> Canny边缘检测 -> 膨胀
2.  **获取轮廓 (`GetContours`)**: 从预处理后的二值图像中查找所有轮廓，通过 `contourArea` 和 `approxPolyDP` 找到面积最大且具有4个顶点的矩形轮廓，返回其四个角点。
3.  **角点重排序 (`ReOrder`)**: 由于检测到的四个角点顺序可能不统一，需要将其排序为 `[左上, 右上, 左下, 右下]`。
    - **方法**:
        - 将所有点的 `x+y` 值求和，和最小的点为左上角(0)，和最大的点为右下角(3)。
        - 将所有点的 `x-y` 值(或 `y-x`)求差，差最小的点为右上角(1)，差最大的点为左下角(2)。
4.  **应用透视变换 (`GetWarp`)**: 使用重排序后的源点和定义好的目标点（A4纸比例，如420x596），计算变换矩阵并执行 `warpPerspective`。
5.  **裁剪边缘**: 因为透视变换后的图像边缘可能不完美，可再次使用 `Rect` 裁剪掉几个像素的边缘，得到最终扫描图像。

### 10.3 项目三：车牌检测器 (Number Plate Detector)
**目标**: 使用网络摄像头从实时视频或从图像中检测并保存车辆车牌。
**核心技术**: Haar级联分类器检测 + 图像裁剪。
**实现步骤**:
1.  **使用摄像头**: 将人脸检测中的代码改为从 `VideoCapture` 读取帧。
2.  **加载车牌分类器**: 加载 `haarcascade_russian_plate_number.xml` 文件。
    ```cpp
    CascadeClassifier plateCascade;
    plateCascade.load("resources/haarcascade_russian_plate_number.xml");
    ```
3.  **循环检测**:
    - 在 `while(true)` 循环中，对每一帧图像进行检测，得到 `plates` 矩形向量。
    - 遍历检测到的 `plates`：
        - 在原图上绘制矩形框。
        - **裁剪与保存**：使用 `Rect` 从原始图像中裁剪出车牌区域 `Mat imgCrop = img(plates[i]);`。使用 `imwrite("resources/plates/" + to_string(i) + ".png", imgCrop);`  将裁剪出的车牌图像保存到本地文件夹。为避免覆盖，文件名可使用时间戳等方式。
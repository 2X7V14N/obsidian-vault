---
tags: [opencv, cheatsheet, cpp, reference, engineering]
created: 2026-07-04
updated: 2026-07-04
status: budding
domain: robot-learning
---

# OpenCV CheatSheet

> C++ OpenCV 工程手册。不是教程，不是函数大全——是项目开发时能快速查阅、复用和排错的工具。
>
> **使用原则**：查得到、看得懂、拿来能用。

---

## 0. 使用说明

**怎么看这份手册**：
- 搜函数名 → 找到函数速查条目，看参数怎么选、有什么坑
- 面对任务 → 找对应任务流程，看完整步骤和常见失败原因
- 遇到报错 → 翻"常见错误与调试方法"或"自己踩过的坑"
- 每次解决新问题 → 补充到对应段落

**更新规则**：每次写代码后，把用到的新函数、遇到的错误、可复用的流程补进来。不要攒。

---

## 1. C++ OpenCV 项目模板

### 最小可编译项目

```cpp
#include <opencv2/imgcodecs.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/imgproc.hpp>
#include <iostream>

int main() {
    cv::Mat img = cv::imread("test.jpg");
    if (img.empty()) {
        std::cerr << "Failed to load image." << std::endl;
        return -1;
    }
    cv::imshow("Image", img);
    cv::waitKey(0);
    return 0;
}
```

### 常用头文件

| 头文件 | 用途 |
|--------|------|
| `<opencv2/imgcodecs.hpp>` | `imread`, `imwrite` |
| `<opencv2/highgui.hpp>` | `imshow`, `waitKey`, `createTrackbar` |
| `<opencv2/imgproc.hpp>` | `cvtColor`, `GaussianBlur`, `Canny`, `findContours`, `warpPerspective` ... |
| `<opencv2/objdetect.hpp>` | `CascadeClassifier`（Haar 级联，OpenCV 4.x） |
| `<opencv2/objdetect/face.hpp>` | `FaceDetectorYN`（YuNet DNN，OpenCV 5 推荐） |
| `<opencv2/video.hpp>` | `VideoCapture`, `VideoWriter` |

---

## 2. cv::Mat 基础

### cv::Mat 创建

```cpp
cv::Mat img;                                         // 空矩阵
cv::Mat img(480, 640, CV_8UC3);                      // 480行×640列，3通道8位无符号
cv::Mat img(480, 640, CV_8UC3, cv::Scalar(0,0,0));  // 同上，初始化为黑色
cv::Mat img = cv::imread("test.jpg");                // 从文件读取
```

### cv::Mat 属性与状态

```cpp
img.rows;        // 高度（行数）
img.cols;        // 宽度（列数）
img.channels();  // 通道数（灰度=1，彩色=3）
img.type();      // 数据类型（如 CV_8UC3）
img.empty();     // 是否为空（读图失败时返回 true）
img.size();      // cv::Size(w, h)，注意 width=cols, height=rows
```

### cv::Mat::clone

**作用**：创建图像数据的深拷贝（独立内存，修改不影响原图）。

```cpp
cv::Mat copy = img.clone();
```

> ⚠️ `cv::Mat b = a;` 只是共享数据头——修改 b 会同时改变 a。需要独立副本时一定要 `.clone()`。

### ROI 区域裁剪

```cpp
cv::Rect roi(x, y, width, height);
cv::Mat crop = img(roi);             // 引用原图数据（修改crop会影响img）
cv::Mat cropCopy = img(roi).clone(); // 独立副本
```

### 常见坑

- `cv::Mat b = a` 是浅拷贝，`b.clone()` / `a.copyTo(b)` 才是深拷贝
- ROI 裁剪默认是引用，传给会修改输入的函数（如 `findContours`）前要 `.clone()`
- `img.size()` 返回 `Size(cols, rows)`，注意顺序

---

## 3. 图像读取、显示与保存

### cv::imread

**作用**：从文件路径读取图像。

```cpp
cv::Mat img = cv::imread("data/test.jpg");
if (img.empty()) {
    std::cerr << "Failed to read image." << std::endl;
    return -1;
}
```

- 输入：文件路径（相对或绝对）
- 输出：`cv::Mat`

| 关键参数 | 说明 |
|---------|------|
| `cv::IMREAD_COLOR` | 按 BGR 彩色图读取（**默认**） |
| `cv::IMREAD_GRAYSCALE` | 按灰度图读取 |
| `cv::IMREAD_UNCHANGED` | 保留原图所有通道（含 alpha） |

常见坑：
- 路径写错不报编译错误，只返回空图 → **所有 imread 后面都要加 `empty()` 检查**
- Windows 路径建议用 `/` 或 `\\`，不要用 `\`
- 读进来的彩色图是 BGR，不是 RGB

### cv::imshow

**作用**：在窗口中显示图像。

```cpp
cv::imshow("Window Name", img);
cv::waitKey(0);   // 0 = 无限等待按键；正数 = 等待指定毫秒
```

> `waitKey` 是必需的——不调用它，窗口不会显示或立即消失。

### cv::imwrite

**作用**：将图像保存到文件。

```cpp
cv::imwrite("output/result.jpg", img);
```

- 格式由扩展名决定（`.jpg`, `.png`, `.bmp` 等）
- JPEG 可加第三个参数控制压缩质量：`cv::imwrite("out.jpg", img, {cv::IMWRITE_JPEG_QUALITY, 95})`

### cv::waitKey

| 值 | 行为 |
|----|------|
| `0` | 无限等待，直到按下任意键 |
| `1` | 等待 1ms（视频实时显示） |
| `>0` | 等待指定毫秒数 |

> `waitKey(0)` 用于单张图片查看；`waitKey(1)` 用于视频/摄像头循环。

---

## 4. 颜色空间转换

### cv::cvtColor

**作用**：转换图像的颜色空间。

```cpp
cv::cvtColor(src, dst, cv::COLOR_BGR2GRAY);    // BGR → 灰度
cv::cvtColor(src, dst, cv::COLOR_BGR2HSV);     // BGR → HSV
cv::cvtColor(src, dst, cv::COLOR_BGR2RGB);     // BGR → RGB（与其他库交互时用）
cv::cvtColor(src, dst, cv::COLOR_GRAY2BGR);    // 灰度 → BGR（画彩色标注时用）
```

- 输入：源图像
- 输出：转换后的图像

关键参数（常用转换码）：
| 转换码 | 场景 |
|--------|------|
| `COLOR_BGR2GRAY` | 边缘检测/轮廓检测/人脸检测的前置步骤 |
| `COLOR_BGR2HSV` | 颜色检测/分割（`inRange` 的前置） |
| `COLOR_BGR2RGB` | 与 matplotlib/PIL/TensorFlow 交互时转换 |

常见坑：
- OpenCV 默认彩色图是 **BGR**，不是 RGB
- 与其他库交互时忘了转换 → 红蓝互换
- 灰度图转 BGR 后三个通道值相同，视觉上仍是灰色

---

## 5. 图像预处理

### cv::GaussianBlur

**作用**：高斯滤波去噪——边缘检测前的标准前置步骤。

```cpp
cv::GaussianBlur(src, dst, cv::Size(5, 5), 0);
//                                  核大小    sigmaX
```

- 输入：任意图像
- 输出：模糊后的图像

| 关键参数 | 怎么选 |
|---------|--------|
| 核大小 `Size(k,k)` | 必须是正奇数（3,5,7,9）。Canny 前常用 `5×5` 或 `7×7` |
| `sigmaX` | 设为 `0` 时从核大小自动计算。手动设时越大越模糊 |

常见坑：
- 核太大 → 边缘也被模糊掉 → Canny 输出残缺
- 核太小 → 噪声残留 → Canny 输出大量伪边缘

### cv::medianBlur

**作用**：中值滤波——适合椒盐噪声。

```cpp
cv::medianBlur(src, dst, 5);  // 核大小必须为正奇数
```

### cv::bilateralFilter

**作用**：双边滤波——去噪的同时保留边缘，但速度较慢。

```cpp
cv::bilateralFilter(src, dst, 9, 75, 75);
//                             d  sigmaColor  sigmaSpace
```

---

## 6. 阈值分割

### cv::threshold

**作用**：将灰度图转为二值图。

```cpp
cv::Mat binary;
cv::threshold(gray, binary, 0, 255, cv::THRESH_BINARY | cv::THRESH_OTSU);
//                          阈值 最大值 二值化方法        自动阈值
```

- 输入：灰度图（`CV_8UC1`）
- 输出：二值图

| 关键参数 | 怎么选 |
|---------|--------|
| `cv::THRESH_BINARY` | 大于阈值→255，小于→0（目标为白色） |
| `cv::THRESH_BINARY_INV` | 反过来（目标为黑色，背景白色时用） |
| `cv::THRESH_OTSU` | 自动计算最佳阈值（前景/背景灰度差异明显时用） |

常见坑：
- 输入必须是灰度图，传入彩色图结果不可预期
- 光照不均匀时 Otsu 效果差 → 换 `adaptiveThreshold`
- 二值图中目标要白色、背景黑色 → 否则 `findContours` 提取的是背景

### cv::adaptiveThreshold

**作用**：自适应阈值——光照不均匀时的选择。

```cpp
cv::adaptiveThreshold(gray, binary, 255,
    cv::ADAPTIVE_THRESH_GAUSSIAN_C, cv::THRESH_BINARY, 11, 2);
//                                 块大小（必须奇数）      C
```

### cv::inRange

**作用**：HSV 颜色范围过滤，输出二值掩膜。

```cpp
cv::Mat mask;
cv::Scalar lower(h_min, s_min, v_min);
cv::Scalar upper(h_max, s_max, v_max);
cv::inRange(imgHSV, lower, upper, mask);
```

- 输入：HSV 图像
- 输出：二值掩膜（范围内=255，范围外=0）

常见坑：
- 必须先转 HSV（`COLOR_BGR2HSV`），不能直接用 BGR 图
- HSV 的 H 范围是 0~179（不是 0~360）
- 红色在 H 的两端（0~10 和 170~179），需要两个 `inRange` + `|` 合并

---

## 7. 形态学操作

### cv::getStructuringElement + cv::dilate / cv::erode

```cpp
cv::Mat kernel = cv::getStructuringElement(cv::MORPH_RECT, cv::Size(3, 3));
cv::dilate(binary, dilated, kernel);    // 膨胀：白色区域变大
cv::erode(binary, eroded, kernel);      // 腐蚀：白色区域变小
```

| 结构元素 | 场景 |
|---------|------|
| `MORPH_RECT` | 通用 |
| `MORPH_ELLIPSE` | 圆形物体 |
| `MORPH_CROSS` | 保留细线 |

### cv::morphologyEx — 开闭运算

```cpp
cv::morphologyEx(src, dst, cv::MORPH_OPEN, kernel);    // 开：先去小噪点
cv::morphologyEx(src, dst, cv::MORPH_CLOSE, kernel);   // 闭：先填小孔洞
```

| 操作 | 数学 | 效果 |
|------|------|------|
| 开运算 | 先侵蚀再膨胀 | 消除小噪点、平滑边界 |
| 闭运算 | 先膨胀再侵蚀 | 填充小孔、闭合断裂边缘 |

常见坑：
- 核太大 → 把不想连的东西连起来了
- 核太小 → 没起到闭合/去噪效果
- 操作顺序记反：开=清理（先缩再胀），闭=修补（先胀再缩）

---

## 8. 边缘检测

### cv::Canny

**作用**：从灰度图中提取边缘（输出二值图）。

```cpp
cv::Canny(blurred, edges, 50, 150);
//                        低阈值 高阈值
```

- 输入：灰度图（通常先 `GaussianBlur`）
- 输出：二值边缘图（边缘=255白，非边缘=0黑）

| 关键参数 | 怎么选 |
|---------|--------|
| 低阈值 | 低于它的像素一定不是边缘。推荐用 Trackbar 调试 |
| 高阈值 | 高于它的像素一定是边缘。中间值需与强边缘连通才保留 |
| 阈值比 | 通常 1:2 或 1:3（如 50:150 或 75:200） |

常见坑：
- OpenCV 的 `Canny()` 内部**不包含**高斯模糊——必须自己在前面调用 `GaussianBlur`
- 阈值太高 → 边缘太少，漏检
- 阈值太低 → 全是噪声导致的伪边缘
- 输出是二值图，可直接作为 `findContours` 的输入

---

## 9. 轮廓检测

### cv::findContours

**作用**：从二值图中提取所有轮廓。

```cpp
std::vector<std::vector<cv::Point>> contours;
std::vector<cv::Vec4i> hierarchy;

cv::findContours(
    binary.clone(),        // 必须.clone()！会修改输入
    contours,
    hierarchy,
    cv::RETR_EXTERNAL,     // 检索模式
    cv::CHAIN_APPROX_SIMPLE
);
```

- 输入：单通道二值图（白色=前景）
- 输出 `contours`：`vector<vector<Point>>`，每条轮廓是一组连续边界点
- 输出 `hierarchy`：`vector<Vec4i>`，`hierarchy[i] = {Next, Previous, First_Child, Parent}`

| 检索模式 | 行为 |
|---------|------|
| `RETR_EXTERNAL` | 只取最外层（不关心嵌套时用这个） |
| `RETR_LIST` | 所有轮廓，不建层级关系 |
| `RETR_CCOMP` | 两层：外边界+孔洞 |
| `RETR_TREE` | 完整父子树（需要分析嵌套关系时） |

常见坑：
- **必须传 `.clone()`**，否则原二值图被破坏
- 输入必须是二值图，不能是灰度图或彩色图
- 目标为黑色、背景为白色时需要反二值化或 `bitwise_not`
- 噪声太多 → 先形态学开运算或按面积过滤

### cv::drawContours

**作用**：在图像上绘制轮廓。

```cpp
cv::drawContours(img, contours, -1, cv::Scalar(0, 255, 0), 2);
//              画布        索引(-1=全部) 颜色              线宽
```

### cv::contourArea

```cpp
double area = cv::contourArea(contour);
// 过滤噪声：if (area > 1000) { /* 有效轮廓 */ }
```

### cv::arcLength

```cpp
double peri = cv::arcLength(contour, true);  // true=闭合曲线
```

### cv::approxPolyDP

**作用**：用最少点近似轮廓（Douglas-Peucker 算法），用于形状分类。

```cpp
std::vector<cv::Point> approx;
double peri = cv::arcLength(contour, true);
cv::approxPolyDP(contour, approx, 0.02 * peri, true);
//                                 ε（允许的最大误差）
```

| 关键参数 | 怎么选 |
|---------|--------|
| ε | `0.02 * peri` 是经验值。越大 → 点越少，形状越简化 |

常见坑：
- ε 太大 → 正方形被近似为三角形
- ε 太小 → 噪点导致角点数不准确

---

## 10. 几何测量

### cv::boundingRect — 正立外接矩形

```cpp
cv::Rect rect = cv::boundingRect(contour);
// rect.x, rect.y, rect.width, rect.height
```

### cv::minAreaRect — 最小旋转外接矩形

```cpp
cv::RotatedRect rotated = cv::minAreaRect(contour);
// rotated.size.width, rotated.size.height, rotated.angle
```

> 与 `boundingRect` 不同：`minAreaRect` 可以旋转，给出任意角度下面积最小的包围矩形。

### cv::boxPoints

**作用**：将 `RotatedRect` 转为 4 个角点（用于绘制）。

```cpp
cv::Point2f vertices[4];
rotated.points(vertices);   // 获取 4 个角点到数组
// 或用 boxPoints：
// cv::boxPoints(rotated, vertices);
```

### cv::moments — 计算轮廓矩（用于找质心）

```cpp
cv::Moments m = cv::moments(contour);
double cx = m.m10 / m.m00;  // 质心 x
double cy = m.m01 / m.m00;  // 质心 y
```

### 圆度公式

```cpp
double circularity = 4.0 * CV_PI * area / (perimeter * perimeter);
// 完美圆 ≈ 1.0，正方形 ≈ 0.79
```

---

## 11. 几何变换

### cv::resize

**作用**：缩放图像。

```cpp
cv::resize(src, dst, cv::Size(), 0.5, 0.5, 0, 0, cv::INTER_AREA);   // 按比例缩小
cv::resize(src, dst, cv::Size(), 2.0, 2.0, 0, 0, cv::INTER_CUBIC);  // 按比例放大
```

| 插值方式 | 场景 |
|---------|------|
| `INTER_AREA` | 缩小图像（质量最好） |
| `INTER_CUBIC` | 放大图像 |
| `INTER_LINEAR` | 默认，速度与质量平衡 |

### cv::warpAffine — 仿射变换（旋转/平移/缩放）

```cpp
cv::Mat M = cv::getRotationMatrix2D(cv::Point2f(w/2, h/2), angle, scale);
cv::warpAffine(src, dst, M, cv::Size(w, h));
```

### cv::getPerspectiveTransform + cv::warpPerspective — 透视变换

**作用**：将平面物体从透视角度矫正为正视。

```cpp
// 源四角点（顺序：左上、右上、左下、右下）
std::vector<cv::Point2f> src = { {x0,y0}, {x1,y1}, {x2,y2}, {x3,y3} };
// 目标四角点
std::vector<cv::Point2f> dst = { {0,0}, {w,0}, {0,h}, {w,h} };

cv::Mat matrix = cv::getPerspectiveTransform(src, dst);
cv::warpPerspective(img, warped, matrix, cv::Point(w, h));
```

- 最少需要 4 对对应点
- 角点顺序必须对应一致（src[0]↔dst[0]...），否则结果扭曲

### 角点重排序算法

轮廓检测返回的角点顺序是任意的，使用前必须排序为 [左上, 右上, 左下, 右下]：

```
1. 按 x+y 排序：和最小→左上角，和最大→右下角
2. 剩余两点按 x-y 排序：差最小→右上角，差最大→左下角
```

---

## 14. 视频与摄像头

### cv::VideoCapture

```cpp
cv::VideoCapture cap(0);                          // 默认摄像头
cv::VideoCapture cap("video.mp4");                // 视频文件

if (!cap.isOpened()) {
    std::cerr << "Cannot open camera." << std::endl;
    return -1;
}

cv::Mat frame;
while (true) {
    cap.read(frame);
    if (frame.empty()) break;

    // 逐帧处理...

    cv::imshow("Video", frame);
    if (cv::waitKey(1) == 'q') break;     // 按 q 退出
}
cap.release();
```

常见坑：
- 摄像头编号从 0 开始，被占用时 `isOpened()` 返回 false
- 每帧处理太慢 → 卡顿 → 考虑降低分辨率或跳帧
- 退出循环后要 `release()` 释放资源

---

## 15. 常见项目流程

### 15.1 颜色检测 + 轮廓追踪

```
BGR → HSV → inRange → 形态学开运算 → findContours → 取质心/边界框
```

10 行核心代码：

```cpp
cv::Mat hsv, mask;
cv::cvtColor(frame, hsv, cv::COLOR_BGR2HSV);

cv::Scalar lower(0, 100, 100), upper(10, 255, 255);
cv::inRange(hsv, lower, upper, mask);

cv::morphologyEx(mask, mask, cv::MORPH_OPEN, cv::getStructuringElement(cv::MORPH_RECT, cv::Size(5,5)));

std::vector<std::vector<cv::Point>> contours;
cv::findContours(mask, contours, cv::RETR_EXTERNAL, cv::CHAIN_APPROX_SIMPLE);

for (auto& cnt : contours) {
    if (cv::contourArea(cnt) > 500) {
        cv::Rect r = cv::boundingRect(cnt);
        cv::rectangle(frame, r, cv::Scalar(0,255,0), 2);
    }
}
```

### 15.2 边缘检测 → 轮廓提取 → 形状识别

```
灰度 → 高斯模糊 → Canny → 膨胀 → findContours → approxPolyDP → 角点判断
```

判断三角形/矩形/圆形的逻辑：

```cpp
for (auto& cnt : contours) {
    double area = cv::contourArea(cnt);
    if (area < 1000) continue;

    double peri = cv::arcLength(cnt, true);
    std::vector<cv::Point> approx;
    cv::approxPolyDP(cnt, approx, 0.02 * peri, true);

    int corners = approx.size();
    cv::Rect r = cv::boundingRect(cnt);

    if (corners == 3)       { /* 三角形 */ }
    else if (corners == 4)  {
        float ar = (float)r.width / r.height;
        if (ar > 0.95 && ar < 1.05) { /* 正方形 */ }
        else                        { /* 矩形 */ }
    }
    else if (corners > 4)   {
        double circ = 4.0 * CV_PI * area / (peri * peri);
        if (circ > 0.85)    { /* 圆形 */ }
    }
}
```

### 15.3 透视矫正（文档扫描）

```
灰度 → 高斯模糊 → Canny → 膨胀 → findContours → 找最大四边形 → 角点重排序 → getPerspectiveTransform → warpPerspective
```

关键步骤——找最大四边形：

```cpp
std::vector<cv::Point> biggest;
int maxArea = 0;
for (auto& cnt : contours) {
    int area = cv::contourArea(cnt);
    if (area > 1000) {
        double peri = cv::arcLength(cnt, true);
        std::vector<cv::Point> approx;
        cv::approxPolyDP(cnt, approx, 0.02 * peri, true);
        if (approx.size() == 4 && area > maxArea) {
            biggest = approx;
            maxArea = area;
        }
    }
}
```

### 15.4 人脸检测（传统方法 — OpenCV 4.x）

```
灰度 → CascadeClassifier.detectMultiScale → 绘制检测框
```

```cpp
cv::CascadeClassifier cascade;
cascade.load("haarcascade_frontalface_default.xml");

std::vector<cv::Rect> objects;
cascade.detectMultiScale(gray, objects, 1.1, 10);

for (auto& obj : objects) {
    cv::rectangle(frame, obj, cv::Scalar(255, 0, 255), 2);
}
```

### 15.5 人脸检测（现代 DNN 方法 — OpenCV 5 推荐）

YuNet `.onnx` 模型，BGR 彩色图直接输入，输出含矩形框 + 5 个面部关键点 + 置信度。

```cpp
#include <opencv2/objdetect/face.hpp>

auto detector = cv::FaceDetectorYN::create(
    "face_detection_yunet_2023mar.onnx", "", img.size(), 0.6f, 0.3f, 5000);

cv::Mat faces;
detector->detect(img, faces);

for (int i = 0; i < faces.rows; i++) {
    int x = int(faces.at<float>(i, 0));
    int y = int(faces.at<float>(i, 1));
    int w = int(faces.at<float>(i, 2));
    int h = int(faces.at<float>(i, 3));
    float score = faces.at<float>(i, 14);
    cv::rectangle(img, cv::Rect(x, y, w, h), cv::Scalar(0, 255, 0), 2);
}
```

> `Ptr<FaceDetectorYN>` 是智能指针，用 `->` 调用方法；`faces` 是输出参数（空 Mat 传入，由 detect 填入结果）。详见 [[Robot_Learning/concepts/OpenCV/第8章 - 人脸检测|第8章]]。

---

## 16. 常见错误与调试方法

| 现象 | 可能原因 | 检查方法 |
|------|---------|---------|
| imshow 空窗口 | `imread` 读图失败 | 检查 `img.empty()` |
| 轮廓数量为 0 | 二值图方向反了/阈值不当 | 先 imshow 确认二值图是否正确 |
| 轮廓太多/全是噪声 | 没做模糊/Canny 阈值太低 | 检查预处理管道每一步的输出 |
| findContours crash | 输入是三通道图 | 确认传入的是灰度二值图 |
| 透视变换结果扭曲 | 角点顺序错乱 | 检查重排序逻辑 |
| 程序编译通过但运行黑屏 | 路径错误/DLL 缺失 | 检查 PATH 环境变量 + `img.empty()` |
| 红色物体检测不到 | HSV H 环绕（红在两端） | 用两个 inRange + 按位或 |
| `hierarchy` 越界 crash | 没检查 `hierarchy.empty()` | `if (!hierarchy.empty())` |

### 调试标准流程

```
问题发生 → 打印中间图像（imshow 每步输出）→ 定位是哪一步出了问题 → 调整参数 → 确认修复 → 记录到本手册
```

---

## 17. 自己踩过的坑

### BGR vs RGB

**现象**：用 matplotlib 显示 OpenCV 读取的图像，颜色不对。

**原因**：OpenCV 默认 BGR，matplotlib 默认 RGB。

**解决**：`cv::cvtColor(img, img, cv::COLOR_BGR2RGB)` 后再给 matplotlib。

**以后如何避免**：跨库使用图像时，先确认颜色通道顺序。

### findContours 修改输入图

**现象**：对 `findContours` 输入的二值图做后续处理，结果不对。

**原因**：`findContours` 内部会修改输入图像。

**解决**：传入 `binary.clone()` 而不是原始 `binary`。

**以后如何避免**：所有传给 `findContours` 的图像都用 `.clone()` 保护原图。

### imread 路径不报编译错误

**现象**：程序编译通过，运行后窗口一片空白，后续函数报 `!_src.empty()` 断言失败。

**原因**：图片路径写错，`imread` 不报异常，只返回空 Mat。

**解决**：所有 `imread` 后加 `if (img.empty())` 检查。

**以后如何避免**：写成固定模式——`imread` 后永远紧接 `empty()` 判断。

### HSV 红色检测需要两个范围

**现象**：用 `inRange` 检测红色，只能检测到一部分红色区域。

**原因**：HSV 的 Hue 是环形——红色在 0°（H=0~10）和 360°（H=170~179）两端。

**解决**：两个 `inRange` + `mask1 | mask2` 合并。

**以后如何避免**：检测红色、紫色等 Hue 边界附近的颜色时，检查是否需要两端覆盖。

---

## 18. 待验证内容

> 以下功能和流程尚未在实际项目中使用和验证，仅记录函数名和典型用法供后续参考。

### 相机标定

```cpp
cv::findChessboardCorners(gray, patternSize, corners);
cv::cornerSubPix(gray, corners, winSize, zeroZone, criteria);
cv::calibrateCamera(objectPoints, imagePoints, imageSize, cameraMatrix, distCoeffs, rvecs, tvecs);
cv::undistort(distorted, undistorted, cameraMatrix, distCoeffs);
```

待填充：完整标定流程、棋盘格角点数量定义、重投影误差评估。

### 位姿估计

```cpp
cv::solvePnP(objectPoints, imagePoints, cameraMatrix, distCoeffs, rvec, tvec);
cv::Rodrigues(rvec, R);  // 旋转向量 → 旋转矩阵
```

待填充：PnP 的输入要求、rvec/tvec 的含义、与相机标定的衔接。

---

## 快速问答索引

| 问题 | 翻到哪 |
|------|--------|
| 怎么读一张图并显示？ | §3 `imread` + `imshow` |
| 怎么转二值图？ | §6 `threshold` / `inRange` |
| 怎么提取工件轮廓？ | §9 `findContours` + §15.2 流程 |
| 怎么测矩形工件长宽？ | §10 `minAreaRect` + §15.2 |
| 怎么矫正倾斜的纸？ | §11 透视变换 + §15.3 |
| 读图失败怎么办？ | §17 "imread 路径" |
| 轮廓很乱/太多噪声？ | §16 常见错误表 |
| 怎么搭建项目模板？ | §1 |
| OpenCV 5 怎么检测人脸？ | §15.5 DNN FaceDetectorYN / [[Robot_Learning/concepts/OpenCV/第8章 - 人脸检测\|第8章]] |
| CascadeClassifier 找不到了？ | [[Robot_Learning/concepts/OpenCV/第8章 - 人脸检测#第一部分：传统方法 — Haar 级联分类器\|第8章 — OpenCV 5 已移除]] |

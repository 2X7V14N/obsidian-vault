# 项目上下文整合报告：ROS2 + YOLO + RGB-D 桌面目标检测与三维定位系统

> 文档用途：项目学习、源码交接、问题追踪和后续 Agent 初始化。
> 整合范围：当前仓库源码、配置、文档、Git 状态，以及本项目相关对话记录。
> 生成时点：2026-09-12。
> 说明：没有材料支持的内容标记为“待确认”，不把虚拟数据结果表述为真实硬件能力。

## 材料地图

| 材料类别 | 主要来源 | 作用 |
| --- | --- | --- |
| 项目代码 | `ros2_ws/src/vision_interfaces/`、`ros2_ws/src/vision_pipeline/` | ROS2 消息、节点、几何计算和 RViz 可视化 |
| 离线脚本 | `scripts/` | 推理、训练、评估、数据整理、实验和基准测试 |
| 配置 | `configs/`、`ros2_ws/src/vision_pipeline/config/pipeline.yaml` | 模型、数据集、虚拟相机和 ROS2 参数 |
| 模型 | `models/final/desktop_detection_v3.pt` | 当前正式 YOLO 权重 |
| 文档 | `README.md`、`docs/` | 架构、实验、路线、接口和 Demo 说明 |
| 工程元数据 | `requirements.txt`、`package.xml`、`setup.py`、`.github/workflows/ci.yml` | 依赖、构建、测试和 CI |
| 对话记录 | 用户提供的训练指标、命令输出、报错和学习问答 | 项目历史、运行证据、边界和学习状态 |
| 当前 Git 状态 | `main`、`origin/main`、工作区差异 | 版本和未提交修改 |

# 0. 一页速览

## 项目一句话总结

【事实】这是一个使用 ROS2、YOLO 和 RGB-D 数据识别桌面 `cup`、`book`、`pen`，并输出目标在相机光学坐标系下三维位置的视觉原型系统。

## 项目目的

【事实】建立一条可替换输入、可测试、可可视化的机器人视觉感知链路，为后续接入真实 RGB-D 相机、桌面坐标标定和机械臂抓取做准备。来源：`README.md`、`docs/roadmap_no_arm.md`、项目对话。

## 当前状态

【事实】ROS2 虚拟 RGB-D 主链路已经实现：虚拟输入 → YOLO 二维检测 → RGB-D 深度反投影 → 自定义三维检测消息 → RViz Marker。

【事实】当前没有真实 RGB-D 相机、没有机械臂，深度图是虚拟数据；项目不能宣称已经完成真实三维定位或抓取。

【事实】当前模型为 `desktop_detection_v3.pt`，离线测试记录为 `mAP50=0.9826`、`mAP50-95=0.8546`。来源：`docs/experiments/baseline_v3.md`。

【事实】Git 分支为 `main`，远程为 `https://github.com/2X7V14N/ros2-yolo-rgbd-desktop-localization.git`，最近提交为 `d092e70`。当前工作区存在未提交的 `geometry.py` 修改。

## 核心原理

【事实】使用针孔相机反投影：

```text
X = (u - cx) * Z / fx
Y = (v - cy) * Z / fy
Z = depth(u, v)
```

深度从检测框中心附近窗口中筛选有效值后取中值，再与 `CameraInfo.K` 中的内参结合计算相机坐标。

## 核心流程

```text
虚拟彩色图/深度图/CameraInfo
    -> Detection2DArray
    -> 有效深度窗口
    -> 像素反投影
    -> LocalizedDetectionArray
    -> RViz MarkerArray
```

## 当前最大阻塞

【事实】真实 RGB-D 相机尚未到位，无法完成真实深度测量、深度对齐验证、真实内外参标定和真实定位误差实验。来源：多次项目对话和 `README.md`。

【风险】当前 `geometry.py` 的练习代码被追加在模块顶层，导入模块时会执行打印并产生副作用；需要在下一次代码整理时移入独立练习脚本或 `if __name__ == "__main__":` 块。

## 下一步最优先事项

1. 保留并完成当前学习练习，但不要把练习变量放在正式库模块顶层。
2. 修复 `scripts/inference_json.py` 中缺失的 `PROJECT_ROOT` 定义。
3. 将 ROS2 参数文件中的 `device: "0"` 改为可移植的 `auto`，或明确使用参数覆盖。
4. 为四节点链路增加可复现的 ROS2 集成测试和输入异常测试。
5. 相机到位后优先验证深度单位、彩色/深度对齐、内参和坐标系，再做真实误差评估。

# 1. 项目目的与背景

## 1.1 为什么做这个项目

【事实】用户希望面向机器人视觉、计算机视觉和 ROS2 相关秋招，系统学习并完善一个可解释、可展示的项目。来源：项目对话。

【事实】项目最初由 AI 辅助生成，用户目前希望从零理解源码、数据流、技术难点和不足，而不是只会运行命令。来源：用户学习目标说明。

## 1.2 解决的问题

【事实】系统识别桌面上的三类目标，并将二维像素检测与深度信息结合，输出相机坐标系下的三维点。

【推断】长期应用场景是为机器人提供目标候选位置，未来可能扩展到抓取位姿和机械臂闭环；这些后续功能目前尚未实现。

## 1.3 目标用户和场景

【事实】当前直接用户是项目作者本人，用于学习、求职展示和后续机器人视觉开发。

【推断】未来使用场景为桌面物体感知和机器人操作，但没有材料证明已经有具体机器人型号、机械臂型号或生产部署需求。

## 1.4 成功标准和评价指标

### 已有指标

| 指标 | 数值 | 适用范围 |
| --- | ---: | --- |
| Precision | 0.9857 | V3 离线二维检测测试集 |
| Recall | 0.9489 | V3 离线二维检测测试集 |
| mAP50 | 0.9826 | V3 离线二维检测测试集 |
| mAP50-95 | 0.8546 | V3 离线二维检测测试集 |
| 平均推理延迟 | 48.70 ms | RTX 3050 Ti、640 输入、单图重复推理 |
| 平均 FPS | 20.54 | 同上，仅模型推理，不是完整 ROS2 延迟 |
| 几何测试 | 6 passed | 上一阶段测试，当前练习代码加入后需重跑 |

### 尚缺指标

- 真实深度定位 MAE、RMSE、P50、P95；
- 彩色/深度对齐误差；
- 深度有效率和空洞比例；
- 完整 ROS2 端到端延迟；
- 桌面坐标系转换误差；
- 目标姿态误差；
- 抓取成功率。

## 1.5 非目标

【事实】当前明确不包含：真实 RGB-D 相机验证、机械臂控制、抓取规划、抓取执行、抓取闭环和真实机器人坐标变换。来源：`README.md`、`docs/roadmap_no_arm.md`。

## 1.6 约束

- 当前没有真实相机，必须使用虚拟输入或公开/离线数据。
- 当前没有机械臂，只做感知、定位、评估和可视化。
- 用户是初学者，学习必须按小模块展开，不能只给最终代码。
- 项目需要保持事实边界，不能把虚拟深度写成真实测量。
- 主要环境为 Windows + VS Code + WSL2 + Ubuntu 22.04 + ROS2 Humble + Python 3.10。

# 2. 核心原理

## 2.1 YOLO 二维目标检测

【事实】`yolo_detector.py` 将彩色图转换为 OpenCV/NumPy 图像，调用 Ultralytics YOLO 模型，读取检测框、类别和置信度，发布 `vision_msgs/msg/Detection2DArray`。

模型类别：

```text
0: cup
1: book
2: pen
```

【事实】V3 的离线检测指标来自固定测试集，不等于三维定位精度。

## 2.2 RGB-D 数据

RGB-D 表示：

- RGB：彩色图像；
- D：每个像素对应的深度值。

【事实】真实系统要求深度图已经对齐到彩色图坐标系，否则 YOLO 得到的 `(u,v)` 不能在深度图中取到相同物理位置的深度。

## 2.3 针孔相机模型

相机内参矩阵：

```text
K = [ fx  0  cx ]
    [ 0  fy  cy ]
    [ 0   0   1 ]
```

投影关系：

```text
u = fx * X / Z + cx
v = fy * Y / Z + cy
```

反投影关系：

```text
X = (u - cx) * Z / fx
Y = (v - cy) * Z / fy
Z = depth(u, v)
```

【事实】当前 `pixel_to_camera()` 在 `geometry.py` 中实现了上述反投影，并检查焦距和深度是否为正。

## 2.4 深度窗口中值

【事实】`valid_depth_median()` 会删除 NaN、无穷值、0、负值和超出 `[min_depth_m, max_depth_m]` 的值，再取中值。

【理由】中值比平均值更不容易受到离群点影响；窗口比单像素更能抵抗深度空洞和噪声。

## 2.5 坐标系和 `frame_id`

【事实】当前输出坐标属于 `camera_color_optical_frame`，约定为 X 向右、Y 向下、Z 向前。

【事实】`frame_id` 只是坐标系标签，不会自动完成坐标变换。

如果未来要变换到机器人坐标系，需要：

```text
p_robot = R_robot_camera * p_camera + t_robot_camera
```

并且需要真实标定得到的 `R` 和 `t`，不能使用虚拟或拍脑袋参数代替。

## 2.6 ROS2 事件驱动机制

ROS2 节点不自动执行所有函数，只自动执行已经注册的回调：

- `__init__()`：Python 创建对象时自动调用；
- `create_timer(period, callback)`：定时器到期时调用 callback；
- `create_subscription(..., callback, ...)`：收到消息时调用 callback；
- `registerCallback()`：消息同步成功时调用同步回调。

普通辅助函数必须由其他代码显式调用，例如 `publish_frame()` 调用 `create_camera_info()`，`synced_callback()` 调用 `valid_depth_median()` 和 `pixel_to_camera()`。

## 2.7 时间戳同步

【事实】虚拟输入节点为彩色图、深度图和 CameraInfo 使用同一个时间戳；YOLO 检测结果复制输入彩色图的 Header。

【事实】定位节点使用 `ApproximateTimeSynchronizer`，当前配置为 `queue_size=10`、`slop=0.1`。

【含义】它将时间相同或足够接近的检测、深度和相机参数绑定成一组，避免使用不同时间的观测计算三维位置。

# 3. 系统架构与模块

## 3.1 总体架构

```mermaid
flowchart LR
    A[离线彩色图片] --> B[virtual_rgbd_publisher]
    B -->|Image| C[/camera/color/image_raw]
    B -->|Depth Image| D[/camera/aligned_depth_to_color/image_raw]
    B -->|CameraInfo| E[/camera/color/camera_info]
    C --> F[yolo_detector]
    F -->|Detection2DArray| G[/detections]
    G --> H[rgbd_localizer]
    D --> H
    E --> H
    H -->|LocalizedDetectionArray| I[/detections_3d]
    I --> J[rviz_markers]
    J --> K[/detections_3d_markers]
    K --> L[RViz2]
```

## 3.2 ROS2 包

### `vision_interfaces`

【事实】自定义接口包，使用 `ament_cmake` 和 `rosidl_default_generators` 生成：

- `LocalizedDetection.msg`；
- `LocalizedDetectionArray.msg`。

### `vision_pipeline`

【事实】Python ROS2 功能包，使用 `ament_python`，包含四个可执行节点：

```text
virtual_rgbd_publisher
yolo_detector
rgbd_localizer
rviz_markers
```

## 3.3 关键文件地图

| 文件 | 入口/对象 | 作用 | 类型 |
| --- | --- | --- | --- |
| `ros2_ws/src/vision_pipeline/vision_pipeline/geometry.py` | `pixel_to_camera`、`valid_depth_median` | 几何和深度纯函数 | 核心 |
| `.../virtual_rgbd_publisher.py` | `VirtualRgbdPublisher` | 生成虚拟输入 | 核心 |
| `.../yolo_detector.py` | `YoloDetector` | YOLO 二维检测 | 核心 |
| `.../rgbd_localizer.py` | `RgbdLocalizer` | 同步、深度处理、反投影 | 核心 |
| `.../rviz_markers.py` | `RvizMarkers` | 三维可视化 | 辅助/展示 |
| `vision_interfaces/msg/*.msg` | 消息定义 | 节点间接口契约 | 核心接口 |
| `launch/pipeline.launch.py` | `generate_launch_description` | 一次启动四节点 | 工程入口 |
| `config/pipeline.yaml` | ROS2 参数 | 话题和算法配置 | 配置 |
| `scripts/inference_json.py` | `main` | 单图推理 JSON | 离线工具 |
| `scripts/evaluate_test.py` | `main` | 测试集评估 | 离线工具 |
| `scripts/train_yolo.py` | `train` | 训练入口 | 离线工具 |
| `scripts/benchmark_inference.py` | `main` | 延迟/FPS 基准 | 实验工具 |
| `scripts/virtual_rgbd_pipeline.py` | 顶层脚本 | 离线虚拟定位实验 | 实验工具 |
| `test/test_geometry.py` | pytest 测试 | 几何回归测试 | 测试 |
| `.github/workflows/ci.yml` | GitHub Actions | Python 编译和几何测试 | CI |
| `docs/` | 多份 Markdown | 文档和实验记录 | 辅助 |

## 3.4 关键消息接口

| 话题 | 消息 | 发布者 | 订阅者 |
| --- | --- | --- | --- |
| `/camera/color/image_raw` | `sensor_msgs/msg/Image` | 虚拟输入/真实相机 | YOLO |
| `/camera/aligned_depth_to_color/image_raw` | `sensor_msgs/msg/Image` | 虚拟输入/真实相机 | 定位 |
| `/camera/color/camera_info` | `sensor_msgs/msg/CameraInfo` | 虚拟输入/真实相机 | 定位 |
| `/detections` | `vision_msgs/msg/Detection2DArray` | YOLO | 定位 |
| `/detections_3d` | `vision_interfaces/msg/LocalizedDetectionArray` | 定位 | RViz |
| `/detections_3d_markers` | `visualization_msgs/msg/MarkerArray` | RViz 节点 | RViz2 |

## 3.5 调用关系

```mermaid
sequenceDiagram
    participant Main as main()
    participant ROS as rclpy.spin()
    participant Pub as VirtualRgbdPublisher
    participant Yolo as YoloDetector
    participant Loc as RgbdLocalizer
    participant Viz as RvizMarkers

    Main->>Pub: VirtualRgbdPublisher()
    Pub->>Pub: __init__()
    Pub->>ROS: 注册 Timer(publish_frame)
    ROS->>Pub: 定时调用 publish_frame()
    Pub->>Yolo: 发布彩色 Image
    Pub->>Loc: 发布深度 Image + CameraInfo
    Yolo->>Yolo: image_callback()
    Yolo->>Loc: 发布 Detection2DArray
    Loc->>Loc: ApproximateTimeSynchronizer 匹配
    Loc->>Loc: synced_callback()
    Loc->>Loc: valid_depth_median() + pixel_to_camera()
    Loc->>Viz: 发布 LocalizedDetectionArray
    Viz->>Viz: callback()
    Viz->>ROS: 发布 MarkerArray
```

# 4. 端到端流程

## 4.1 业务流程

1. 输入一张桌面图片。
2. 检测杯子、书和笔。
3. 为每个二维目标关联深度。
4. 输出相机坐标系三维位置。
5. 在 RViz 中展示目标点和坐标。
6. 将来接入真实相机，再评估真实定位误差。

## 4.2 数据流程

```mermaid
flowchart TD
    A[本地 JPG/PNG] --> B[OpenCV BGR NumPy]
    B --> C[ROS Image bgr8]
    D[np.full 虚拟深度图] --> E[ROS Image 32FC1]
    F[K 矩阵 + frame_id + stamp] --> G[CameraInfo]
    C --> H[YOLO]
    H --> I[类别、置信度、bbox 中心]
    I --> J[Detection2DArray]
    J --> K[时间同步]
    E --> K
    G --> K
    K --> L[7x7 深度窗口]
    L --> M[过滤无效值并取中值]
    M --> N[针孔反投影]
    N --> O[LocalizedDetectionArray]
    O --> P[球体和文字 Marker]
```

## 4.3 技术流程和异常处理

| 步骤 | 负责人 | 工具/代码 | 输入 | 输出 | 异常处理 |
| --- | --- | --- | --- | --- | --- |
| 读取图片 | 输入节点 | OpenCV `imread` | 图片路径 | BGR NumPy 数组 | 文件不存在/不可读则启动失败 |
| 发布彩色图 | 输入节点 | `CvBridge.cv2_to_imgmsg` | BGR 数组 | `Image(bgr8)` | 转换失败记录异常 |
| 创建深度 | 输入节点 | `np.full` | 图像尺寸、`depth_m` | `float32` 深度图 | `depth_m<=0` 拒绝 |
| 发布内参 | 输入节点 | `CameraInfo.K` | `fx/fy/cx/cy` | `CameraInfo` | 焦距非正则拒绝 |
| 目标检测 | YOLO 节点 | Ultralytics | 彩色 `Image` | `Detection2DArray` | 推理异常记录日志 |
| 消息匹配 | 定位节点 | `ApproximateTimeSynchronizer` | 三类消息 | 同步回调参数 | 时间差过大时不触发回调 |
| 深度读取 | 定位节点 | `imgmsg_to_cv2` | ROS 深度图 | 米制 NumPy 数组 | `uint16` 除以 1000，其它编码需确认 |
| 深度过滤 | 定位节点 | `valid_depth_median` | 深度窗口 | 中值、有效数 | 无有效值标记 `no_valid_depth` |
| 三维计算 | 定位节点 | `pixel_to_camera` | `(u,v,Z,K)` | `(X,Y,Z)` | 非法焦距/深度抛错 |
| 消息输出 | 定位节点 | 自定义消息 | 检测和三维结果 | `LocalizedDetectionArray` | 无效点不作为有效坐标使用 |
| 可视化 | RViz 节点 | `MarkerArray` | 有效三维点 | 球体、标签 | 只显示 `valid_3d=true` |

## 4.4 开发、调试、测试、部署流程

```text
安装 WSL/ROS2/Python 依赖
    -> VS Code 打开 D:\robot_vision_project
    -> WSL 终端 source 环境
    -> colcon 编译两个 ROS2 包
    -> launch 四节点
    -> echo /detections 和 /detections_3d
    -> RViz 查看 Marker
    -> pytest 几何测试
    -> benchmark 统计延迟
    -> 相机到位后替换输入并标定
```

# 5. 当前进展与里程碑

## 5.1 已完成

- V3 模型已放入 `models/final/desktop_detection_v3.pt`。
- 三类检测类别和固定测试集基线已记录。
- 正确针孔反投影函数已实现。
- 有效深度中值和无效深度拒绝已实现。
- `vision_interfaces` 自定义消息已实现。
- 四节点 ROS2 虚拟链路已实现。
- RViz Marker 配置和节点已实现。
- 性能基准、静态 Demo 脚本、基础 CI 和技术文档已加入。
- 上一阶段 ROS2 构建成功，几何测试曾达到 `6 passed`。来源：前序工作记录。
- 代码和文档已推送到 GitHub，远程 `main` 对应 `d092e70`。来源：Git push 记录。

## 5.2 正在进行

- 用户正在从 Python、相机模型和 ROS2 回调机制开始源码学习。
- 当前正在学习 `VirtualRgbdPublisher`、时间戳、`frame_id` 和函数调用关系。
- 当前 `geometry.py` 中有学习练习代码，尚未整理回正式结构。

## 5.3 未开始或待硬件

- 真实 RGB-D 相机接入。
- 深度单位和对齐验证。
- 真实内参检查和标定。
- 相机到桌面/机器人坐标系的真实外参。
- ROS2 TF2 变换。
- 真实定位误差报告。
- 目标姿态和抓取位姿。
- 机械臂执行和成功率。
- 完整 ROS2 集成测试和 rosbag 回放。

## 5.4 版本和环境

| 项目 | 当前信息 |
| --- | --- |
| 工作目录 | `D:\\robot_vision_project` / WSL `/mnt/d/robot_vision_project` |
| 分支 | `main` |
| 远程 | `https://github.com/2X7V14N/ros2-yolo-rgbd-desktop-localization.git` |
| 最近提交 | `d092e70 补充系统技术设计与 ROS2 接口文档` |
| ROS | ROS2 Humble |
| Python | 3.10 |
| 系统 | WSL2 Ubuntu 22.04 |
| GPU 记录 | NVIDIA RTX 3050 Ti Laptop GPU |
| 当前工作树 | `geometry.py` 有未提交修改 |

## 5.5 常用命令

### 建立环境

```bash
cd /mnt/d/robot_vision_project
source /opt/ros/humble/setup.bash
source .venv/bin/activate
python -m pip install -r requirements.txt
```

### 编译

```bash
cd /mnt/d/robot_vision_project/ros2_ws
/home/wanhaoran/robot_vision_project/.venv/bin/python \
  /usr/bin/colcon build --symlink-install
source install/setup.bash
```

### 启动虚拟链路

```bash
ros2 launch vision_pipeline pipeline.launch.py \
  image_path:=/mnt/d/path/to/test.jpg \
  model_path:=/mnt/d/robot_vision_project/models/final/desktop_detection_v3.pt
```

### 检查话题

```bash
ros2 topic echo /detections --once
ros2 topic echo /detections_3d --once
ros2 topic echo /detections_3d_markers --once
```

### 几何测试

```bash
python -m pytest -q ros2_ws/src/vision_pipeline/test/test_geometry.py
```

### 单图推理

```bash
python scripts/inference_json.py \
  --source "/path/to/image.jpg" \
  --model models/final/desktop_detection_v3.pt \
  --config configs/inference.json \
  --output-dir runs/inference
```

# 6. 关键决策与理由

【事实】当前使用 desktop_detection_v3.pt，原因是已有对话材料中的 V3 离线指标优于 V2。

| 决策 | 背景 | 选择 | 理由 | 替代方案 | 风险 | 来源 |
| --- | --- | --- | --- | --- | --- | --- |
| 使用 desktop_detection_v3.pt | V3 比 V2 的整体检测指标更好 | V3 作为当前正式权重 | V3 的 Precision、Recall、mAP50 和 mAP50-95 均有改善；pen 类提升尤其明显 | 继续使用 V2 或重新训练 | 指标来自离线数据，尚未证明真实场景泛化 | docs/experiments/baseline_v3.md、用户提供的 V2/V3 对比 |
| 使用 ROS2 Topic 解耦节点 | 检测、定位和可视化的输入输出不同 | 各节点通过 Topic 通信 | 便于替换输入源、单独调试和接入真实相机 | 单进程脚本、直接函数调用 | Topic、QoS 或时间同步配置错误会导致无数据或错配 | docs/architecture.md、ROS2 包源码 |
| 使用 vision_msgs/Detection2DArray | 需要表达二维框、类别和置信度 | YOLO 节点输出标准消息 | 与 ROS2 视觉生态兼容，减少自定义消息负担 | 自定义二维检测消息 | 当前只填充了部分标准字段，语义完整性仍需检查 | yolo_detector.py、docs/interface_spec.md |
| 使用 LocalizedDetectionArray | 标准二维检测消息不包含深度和三维位置 | 自定义 vision_interfaces 消息 | 同时保存类别、置信度、像素中心、深度、相机坐标和有效性 | Detection3DArray 或 PointCloud2 | 自定义接口增加维护和构建依赖 | ros2_ws/src/vision_interfaces/msg/ |
| 深度窗口使用中值 | 中心像素可能无效或有噪声 | 在检测中心附近取窗口，过滤后求中值 | 对离群点比均值更稳健，适合当前原型 | 单像素、均值、平面拟合 | 恒定虚拟深度掩盖真实深度问题；窗口可能跨物体边界 | geometry.py、rgbd_localizer.py |
| 使用针孔模型反投影 | 已知像素、深度和内参，需要得到相机坐标 | X=(u-cx)Z/fx、Y=(v-cy)Z/fy、Z=depth | 简单、可解释，是 RGB-D 相机常用基础模型 | OpenCV reprojectImageTo3D、点云库 | 未处理畸变；内参必须来自真实标定 | geometry.py、docs/technical_design.md |
| 当前先使用虚拟 RGB-D 输入 | 真实深度相机尚未到位 | 离线图片 + 虚拟深度图 + CameraInfo | 先验证 ROS2 数据流、消息接口和反投影逻辑 | 等硬件到位后再开发 | 深度没有真实空间信息，不能报告真实定位精度 | virtual_rgbd_publisher.py、用户对话 |
| 使用 ApproximateTimeSynchronizer | 三类消息由独立 Topic 到达 | 允许时间差在 slop=0.1 内组成一组 | 适合非严格同步的原型输入 | TimeSynchronizer、硬件同步 | slop 过大增加错配风险 | rgbd_localizer.py |
| 输出相机光学坐标系 | 目前没有机器人 TF、外参和机械臂 | position_camera_m + camera_color_optical_frame | 这是当前证据支持的坐标结果 | 转换到 base_link 或输出抓取位姿 | 不能直接作为机械臂目标 | LocalizedDetection.msg、学习对话 |
| 暂不提交数据集 | 数据集在项目外且体积/许可证未完全确认 | 仓库保留模型、配置和代码 | 减少仓库体积和版权风险 | 提交小型样例或使用 Git LFS | 克隆者无法直接复现实验 | .gitignore、configs/datasets/ |

# 7. 当前遇到的问题

以下问题按材料确认程度记录；根因假设没有当作确定结论。

| 编号 | 问题 | 类型 | 现象/复现 | 影响 | 已尝试方案 | 根因假设 | 待确认 | 建议下一步 | 优先级 | 关联材料 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| P-01 | 学习练习代码位于正式模块顶层 | 代码质量/教学 | 导入 geometry.py 会执行 result2 打印 | ROS2 节点或测试产生额外输出，库模块有副作用 | 用户直接追加了中值练习 | 已确认是顶层执行 | 是否还需要保留练习结果 | 移到单独教学脚本，或放入 if __name__ == "__main__" | P0 | geometry.py、Git 状态 |
| P-02 | inference_json.py 的相对模型路径分支引用未定义变量 | 代码缺陷 | 使用相对模型路径时可能出现 NameError: PROJECT_ROOT is not defined | 单图推理不能按 README 的相对路径稳定运行 | 尚未修复 | 源码使用 PROJECT_ROOT，但文件顶部未定义 | 绝对路径运行不会触发该分支 | 定义 PROJECT_ROOT = Path(__file__).resolve().parents[1]，补回归测试 | P0 | scripts/inference_json.py |
| P-03 | ROS2 配置把 GPU 写死为 device: "0" | 部署/可移植性 | 无 CUDA 时仍可能尝试 GPU 0 | CPU 环境或不同 GPU 机器启动失败或行为不一致 | 代码支持 auto，但 YAML 覆盖默认值 | 配置覆盖了自动选择逻辑 | 无 GPU 启动的实际日志 | 改为 auto，并允许 launch/命令行覆盖 | P1 | pipeline.yaml、yolo_detector.py |
| P-04 | 数据集未纳入仓库 | 复现/数据 | 克隆仓库后没有完整测试集和标注 | 无法直接复现评估指标 | 使用本机外部数据集测试 | .gitignore 忽略数据，许可证和体积未确认 | 数据集来源、许可证和公开子集 | README 写明数据获取步骤，提供小型可公开样例 | P1 | .gitignore、configs/datasets、对话路径 |
| P-05 | 虚拟深度是整张图的常量 | 算法有效性 | virtual_rgbd_publisher.py 用 np.full 生成统一深度 | 所有物体的 Z 由参数决定，X/Y 也随 Z 缩放 | 用不同虚拟深度做过公式演示 | 这是模拟输入，不是推理错误 | 是否要逐目标真值深度场 | 增加可控深度场和真值文件，单独报告几何链路测试 | P0 | virtual_rgbd_publisher.py、experiments |
| P-06 | 深度图编码和尺寸校验不完整 | 鲁棒性 | uint16 按毫米转米，其他类型直接转 float；未严格比对尺寸 | 编码单位、分辨率或对齐错误时会生成错误坐标 | 支持常见 16UC1/32FC1 路径 | 输入约定未显式验证 | 真实相机编码、单位、对齐状态 | 检查 encoding、维度、frame_id、时间戳和内参尺寸 | P1 | rgbd_localizer.py |
| P-07 | YOLO 推理运行在订阅回调中 | 性能/实时性 | image_callback 内同步执行模型推理 | 推理慢时阻塞 executor，产生积压 | 当前低频虚拟输入可运行 | 单线程回调与模型推理耦合 | 实际 FPS、回调耗时、队列积压 | 增加延迟/吞吐基准，再考虑丢帧或专用线程 | P1 | yolo_detector.py、benchmark_inference.py |
| P-08 | CI 没有完整构建 ROS2 workspace | 工程化/测试 | 未证明 Humble 环境下 colcon build 和最小 launch | 接口生成、依赖和 launch 问题可能只在本机暴露 | 本机手动处理过 colcon/环境 | CI 环境未执行完整 ROS2 检查 | 远程 CI 最新日志 | 增加 ROS2 runner/container，构建、测试和最小启动检查 | P1 | .github/workflows/ci.yml、历史报错 |
| P-09 | 离线脚本与 ROS2 主链路的坐标语义曾不一致 | 设计/文档 | 离线脚本曾演示桌面平面坐标，ROS2 输出相机坐标 | 容易把相机坐标当桌面或机器人坐标 | 学习中已澄清 z=0 只是平面假设 | 两套示例和命名是否完全同步 | 统一坐标系命名，输出带 frame_id | P1 | scripts/virtual_rgbd_pipeline.py、消息定义 |
| P-10 | 缺少真实定位误差与抓取成功率 | 实验/产品 | 有检测指标和虚拟公式演示，没有真实深度真值或抓取闭环 | 不能量化定位准不准，也不能声称可抓取 | 已规划无机械臂路线 | 没有相机、标定板和机械臂 | 相机到位时间、标定条件 | 先做合成真值误差测试，硬件到位后做标定和重复测量 | P0 | docs/experiments、用户对话 |

# 8. 我提问过的问题及解答

下表提取主要学习问题，不是逐字记录全部聊天；摘要保留可交接结论。

| 编号 | 我的原问题 | 问题意图 | 回答摘要 | 结论 | 是否已解决 | 关联文件/对话 | 后续动作 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Q-01 | 这个项目能写到简历里吗，含金量如何？ | 评估项目价值 | 框架和技术链路有价值，但缺真实相机、误差指标和闭环，不能夸大 | 可作为机器人视觉原型，必须说明边界 | 部分解决 | 项目评价对话 | 完成能力分层、补实验指标 |
| Q-02 | 使用真实深度相机后含金量会怎样？ | 了解升级路径 | 真实 RGB-D、标定、TF2、误差评估会提高可信度，但不等于抓取系统 | 硬件是必要条件之一，不是全部 | 已理解，未实施 | 项目评价对话 | 相机到位后接入并标定 |
| Q-03 | V3 模型效果如何？ | 选择正式模型 | V3 比 V2 更好；pen 类 mAP50-95 从约 0.336 提升到约 0.808 | 当前使用 V3 | 已解决 | baseline_v3.md、用户指标 | 记录类别级和跨域测试 |
| Q-04 | NumPy/OpenCV/cv_bridge 报错为什么出现？ | 排查 ROS2 失败 | ROS2 节点和虚拟环境 NumPy/OpenCV ABI 不匹配；重建后 shebang 指向虚拟环境，解决了一部分 | 构建和运行解释器、二进制 ABI 必须一致 | 部分解决 | _ARRAY_API、KeyError: 16 报错 | 固化环境文档和版本矩阵 |
| Q-05 | colcon not found、shebang 不对怎么办？ | 构建 ROS2 包 | 系统提供 /usr/bin/colcon；用目标 Python 调 colcon 重建后生成虚拟环境 shebang | 构建工具和运行解释器可能不同 | 当时已解决 | 历史构建输出 | 增加可重复构建说明 |
| Q-06 | vision_msgs 找不到怎么办？ | 排查消息依赖 | source ROS2 环境和 workspace，确保 package 依赖和消息生成存在 | ROS2 overlay 的 source 顺序很重要 | 当时已解决 | ModuleNotFoundError 历史输出 | 固定 source 顺序 |
| Q-07 | 为什么输出位置都是 0？ | 理解消息字段 | /detections 只是二维检测，pose 默认零值；三维结果由 localizer 写入自定义消息 | 2D pose 零值不等于物体在原点 | 已解决 | 历史 /detections 输出、rgbd_localizer.py | 查看 /detections_3d |
| Q-08 | 为什么 Z 从 0.60 到 0.80 时 X/Y 也按比例变大？ | 理解反投影 | X/Y 都乘 Z；同一像素射线在更远深度处按比例外扩 | 是针孔几何结果 | 已解决 | geometry.py、学习对话 | 用真实深度验证 |
| Q-09 | fx 到底是什么，焦距为什么是像素单位？ | 理解内参 | 物理焦距除以像素尺寸后形成像素单位焦距，是投影尺度参数 | fx 不是图像宽度 | 已解决基础概念 | 相机模型学习对话 | 学习标定和畸变 |
| Q-10 | 为什么要乘 Z/fx？ | 推导公式 | 相似三角形给出 (u-cx)/fx=X/Z，整理得到 X=(u-cx)Z/fx | 需要深度才能恢复实际长度 | 已解决 | 反投影对话 | 手算并测试 |
| Q-11 | 为什么深度窗口取中值？ | 理解鲁棒性 | 过滤 0、NaN、超范围值后取中值，不易被离群点拉偏 | 当前实现是鲁棒估计 | 已解决 | valid_depth_median 学习练习 | 增加边界测试 |
| Q-12 | 为什么三类消息要有相同时间戳？ | 理解同步 | 共同时间语义让同步器知道哪些数据属于同一帧；近似同步允许小误差 | 同一帧不等于时间戳必须完全相等 | 已解决基础概念 | virtual_rgbd_publisher.py、rgbd_localizer.py | 观察真实相机时间戳 |
| Q-13 | frame_id 是什么？ | 理解坐标系标签 | 标识消息所属坐标系；camera_color_optical_frame 表示相机光学坐标系 | 数值必须和坐标系一致 | 已解决基础概念 | ROS Header、LocalizedDetection.msg | 学习 TF2 和外参 |
| Q-14 | declare_parameter 语法是什么意思？ | 学习 ROS2 参数 | 声明参数并给默认值，get_parameter 读取；YAML/launch 可以覆盖 | 参数把代码和配置解耦 | 已解决基础概念 | virtual_rgbd_publisher.py | 练习修改参数 |
| Q-15 | VirtualRgbdPublisher 里的函数没显式调用为什么能运行？ | 理解初始化和回调 | main 实例化类时执行 __init__；create_timer 注册 publish_frame；spin 后由 executor 定时调用 | 回调由事件循环触发 | 已解决基础概念 | virtual_rgbd_publisher.py | 学习 subscription callback |
| Q-16 | ROS2 会自动调用节点里的函数吗？ | 理解回调机制 | 只有注册为 timer、subscription、service、action 等回调的函数才由 executor 调用 | “自动”来自注册和事件循环 | 已解决基础概念 | ROS2 节点源码 | 画调用链 |
| Q-17 | VirtualRgbdPublisher 声明和调用了几个函数？ | 梳理调用关系 | __init__ 初始化并注册资源；create_camera_info 由 publish_frame 调用；publish_frame 由 timer 调用；main 管生命周期 | 沿实例化、注册和 spin 跟踪 | 已解决基础概念 | virtual_rgbd_publisher.py | 写函数输入输出表 |
| Q-18 | 最小完整运行文件夹在哪，为什么有很多文件夹？ | 理解仓库结构 | .venv、缓存和构建产物不是最小源码；核心是模型、配置、ROS2 包、脚本和文档 | 源码仓库应排除环境缓存 | 部分解决 | 目录讨论 | 用干净克隆验证 |

# 9. 未解决问题与下一步

| 优先级 | 任务 | 验收标准 |
| --- | --- | --- |
| P0 | 清理 geometry.py 顶层学习练习 | 导入无额外打印；几何测试仍通过 |
| P0 | 修复 inference_json.py 相对路径 | 任意工作目录都能用相对模型路径运行 |
| P0 | 建立有真值的虚拟深度实验 | 报告深度和反投影误差，并标注不是传感器实验 |
| P0 | 明确项目能力边界 | README 区分二维检测、虚拟 RGB-D、相机坐标和未实现抓取 |
| P1 | 处理 device 配置 | 有/无 CUDA 都能用 auto 启动 |
| P1 | 加强深度输入校验 | 编码、尺寸、内参无效时明确报错或标记无效 |
| P1 | 增加 ROS2 集成测试 | 构建消息包并验证关键 Topic 有数据 |
| P1 | 增加性能基线 | 记录预处理、推理、后处理、端到端延迟和吞吐 |
| P2 | 统一离线脚本和 ROS2 坐标语义 | 所有坐标带 frame，移除或重命名平面模拟输出 |
| P2 | 真实相机接入 | 获取内参、对齐彩色/深度、验证编码和时间戳 |
| P2 | 相机坐标到桌面/机器人坐标 | 用标定板、TF2 和外参发布 table/base_link 下的位置 |
| P2 | 抓取前置接口 | 先输出目标选择和抓取候选位姿；机械臂到位后再接 MoveIt2 |

## 建议排查路径

1. 修复 P0 代码问题并运行 Python 单元测试。
2. 用相对路径和绝对路径分别运行单图推理。
3. 没有真实相机时，用“已知深度场 + 已知内参 + 已知像素”的合成测试验证公式、边界和错误处理。
4. 再做 ROS2 workspace 构建、节点启动和 Topic 检查。
5. 真实相机到位后，先验证原始 Topic 和 CameraInfo，再启用定位节点，最后做 TF2 与误差实验。

## 需要别人协助的事项

- 数据集公开来源、许可证和可否提供小型复现子集：待确认。
- 真实相机型号、SDK、彩色/深度对齐能力和内参获取方式：待确认。
- 若要声称抓取能力，需要机械臂型号、末端执行器、基座 TF 和控制接口：当前未提供。

# 10. 风险、假设与依赖

| 类别 | 项目 | 当前假设/事实 | 不成立时的后果 | 应对方式 |
| --- | --- | --- | --- | --- |
| 技术 | 针孔模型适用 | 内参有效，畸变可忽略或已校正 | 三维点产生系统误差 | 真实相机标定并使用畸变参数/去畸变 |
| 技术 | 深度和彩色已对齐 | 深度像素与彩色像素处于同一成像坐标 | 取到错误物体深度 | 使用 SDK 的 aligned depth，检查分辨率和 frame_id |
| 技术 | 深度单位为米 | uint16 路径按毫米转米 | Z 放大或缩小 1000 倍 | 根据设备文档和实测值配置单位 |
| 技术 | 同步消息属于同一帧 | 使用 ApproximateTimeSynchronizer | 检测框和深度错配 | 调整 queue/slop，检查时间戳 |
| 数据 | V3 测试集代表部署场景 | 指标来自既有测试集 | 真实环境性能下降 | 增加跨域、遮挡、光照和距离分层指标 |
| 数据 | 标注质量足够 | V3 相比 V2 已改善 | 指标和体验不一致 | 抽样审查标注并报告混淆 |
| 部署 | ROS2、Python、NumPy、OpenCV、PyTorch ABI 兼容 | 历史上出现过不兼容报错 | 节点启动失败或崩溃 | 固定环境、记录版本、干净环境 CI |
| 性能 | 同步回调满足帧率 | 当前是低频虚拟输入 | 实时相机下积压和延迟 | 测量端到端延迟，必要时丢帧/异步推理 |
| 坐标 | 输出仅是相机坐标 | 没有机器人 TF/外参 | 不能直接驱动机械臂 | 明确 frame_id，使用 TF2 |
| 产品 | 三维点可代表抓取位姿 | 当前没有姿态、碰撞和抓取验证 | 误用坐标导致抓取失败 | 建模抓取候选、姿态和安全约束 |

# 11. 术语表

| 术语 | 含义 | 关联模块 | 备注 |
| --- | --- | --- | --- |
| YOLO | 单阶段目标检测模型族 | yolo_detector.py、V3 权重 | 输出二维框、类别和置信度 |
| RGB-D | 同时包含彩色图像和深度图的数据 | 虚拟发布器、定位器 | 当前深度是虚拟数据 |
| CameraInfo | ROS2 相机内参消息 | 发布器、定位器 | k[0]=fx、k[4]=fy、k[2]=cx、k[5]=cy |
| fx/fy | 像素单位的水平/垂直焦距参数 | 反投影 | 不是图像宽高 |
| cx/cy | 主点的像素坐标 | 反投影 | 通常接近图像中心但应来自标定 |
| 针孔模型 | 用相似三角形描述三维点投影 | geometry.py | 当前几何计算基础 |
| 反投影 | 从像素和深度恢复相机坐标 | pixel_to_camera | 需要深度和内参 |
| 光学坐标系 | 相机视觉约定的坐标系 | camera_color_optical_frame | 方向遵循设备/TF 约定 |
| frame_id | ROS Header 中的数据坐标系名称 | 图像、检测、Marker | 是 TF 语义的一部分 |
| Topic | ROS2 节点间的发布/订阅通道 | 四节点链路 | 类型和 QoS 必须匹配 |
| Publisher | 发布消息的 ROS2 对象 | 各节点 | 通过 publish 发送 |
| Subscriber | 接收消息并触发回调的对象 | YOLO、RViz | 通过 callback 消费 |
| Timer | 定时触发回调的对象 | 虚拟发布器 | create_timer 注册 |
| Executor / spin | ROS2 事件循环和回调调度 | main | 不是任意函数都自动执行 |
| ApproximateTimeSynchronizer | 按时间近似配对多路消息 | RGB-D 定位 | slop 控制时间差 |
| QoS | ROS2 通信质量策略 | Topic | 队列和可靠性必须匹配 |
| Detection2DArray | 标准二维检测数组消息 | YOLO 输出 | pose 默认零值不是三维位置 |
| LocalizedDetectionArray | 自定义三维检测数组 | 定位输出 | 包含深度、相机点和有效性 |
| TF2 | ROS2 坐标变换系统 | 真实相机后续 | 当前尚未完成机器人坐标转换 |
| mAP50 / mAP50-95 | 目标检测评估指标 | V2/V3 评估 | 与三维误差不是同一指标 |
| colcon | ROS2 工作空间构建工具 | ros2_ws | 构建消息和 Python 包 |
| cv_bridge | OpenCV/Numpy 与 ROS Image 的转换桥 | 发布器、检测器、定位器 | 版本和 ABI 兼容重要 |

# 12. 冲突与不一致

| 冲突点 | 说法 A | 来源 A | 说法 B | 来源 B | 建议处理方式 |
| --- | --- | --- | --- | --- | --- |
| 项目名称与当前能力 | 仓库名含 rgbd、localization，容易理解为真实 RGB-D 定位 | GitHub 仓库名、历史命名 | 当前只有虚拟 RGB-D 和相机坐标输出 | virtual_rgbd_publisher.py、对话记录 | README 首屏明确 virtual RGB-D prototype，真实相机和机器人坐标列为后续 |
| 三维定位的含义 | 早期讨论中曾把平面线性映射称为位置输出 | 历史离线脚本/讨论 | 正式 ROS2 主链路使用针孔反投影 | geometry.py、rgbd_localizer.py | 统一区分平面模拟坐标、相机坐标、桌面坐标、机器人坐标 |
| 数据是否可复现 | 仓库有模型和配置 | 仓库内容 | 数据集在外部且被忽略 | .gitignore、用户数据路径 | 提供数据获取、许可证和校验信息 |
| 环境解释器 | ROS2 系统包使用系统 Python | 早期 shebang 和 NumPy 报错 | 重建后节点 shebang 可指向项目 .venv | 历史 colcon 输出 | 固化 source 顺序、构建解释器和版本矩阵 |
| 输出 Topic 命名 | 历史对话查看过 /object_poses 或 /detections | 历史运行命令 | 当前源码主要输出 /detections_3d 和 /detections_3d_markers | pipeline.yaml、节点源码 | 以当前源码和 ros2 topic list 为准，并记录旧命名 |
| 训练指标与真实部署 | V3 测试集指标很高 | baseline_v3.md、用户指标 | 真实相机和新场景尚未测量 | 用户没有相机 | 面试只报告测试集指标并说明域差异风险 |

# 13. 待补充信息清单

按重要性排序：

1. 真实相机品牌、型号、SDK 和预计接入方式。
2. 数据集原始来源、许可证、目录结构和是否允许公开。
3. V3 训练配置：基础模型、输入尺寸、增强、轮数、随机种子和验证命令。
4. V3 测试集是否与训练/验证集严格隔离，是否存在同源图片泄漏风险。
5. ROS2、Ubuntu/WSL、Python、NumPy、OpenCV、PyTorch、Ultralytics 和 CUDA 版本清单。
6. colcon build、单元测试和完整 launch 最新成功日志。
7. .github/workflows/ci.yml 最近一次远程运行结果。
8. 虚拟深度实验中每个目标的真值，以及希望验证的误差阈值。
9. 桌面平面坐标定义、桌面相对相机姿态，以及未来是否需要 base_link。
10. 机械臂型号、末端执行器、目标选择策略和抓取成功率定义。

# 14. 给下一个 Agent 的初始化 Prompt


```text
你正在接手项目：ROS2 + YOLO + RGB-D 桌面目标检测与三维定位系统。

项目路径：D:\robot_vision_project（WSL: /mnt/d/robot_vision_project）
GitHub：https://github.com/2X7V14N/ros2-yolo-rgbd-desktop-localization
当前分支：main。当前模型：models/final/desktop_detection_v3.pt。
类别映射：0=cup，1=book，2=pen。V3 已知离线指标：Precision=0.9857，Recall=0.9489，mAP50=0.9826，mAP50-95=0.8546；这些指标只代表已有测试集，不代表真实相机部署效果。

项目目标：用 ROS2 组织一条可替换的桌面视觉感知链路：彩色图像 -> YOLO 二维检测 -> 对齐深度和 CameraInfo -> 深度有效性检查 -> 针孔模型反投影 -> 相机光学坐标系三维点 -> RViz Marker。当前没有真实 RGB-D 相机和机械臂，深度来自 virtual_rgbd_publisher.py 的虚拟图，输出不是机器人坐标，也不是抓取位姿。

主链路：
virtual_rgbd_publisher -> /camera/color/image_raw、/camera/aligned_depth_to_color/image_raw、/camera/color/camera_info
-> yolo_detector -> /detections (vision_msgs/Detection2DArray)
-> rgbd_localizer -> /detections_3d (vision_interfaces/LocalizedDetectionArray)
-> rviz_markers -> /detections_3d_markers (visualization_msgs/MarkerArray)

关键文件：
- ros2_ws/src/vision_pipeline/vision_pipeline/virtual_rgbd_publisher.py：离线图片和常量虚拟深度发布。
- ros2_ws/src/vision_pipeline/vision_pipeline/yolo_detector.py：YOLO 推理并发布二维框。
- ros2_ws/src/vision_pipeline/vision_pipeline/rgbd_localizer.py：同步检测、深度、CameraInfo，取有效深度中值并反投影。
- ros2_ws/src/vision_pipeline/vision_pipeline/geometry.py：pixel_to_camera 和 valid_depth_median。
- ros2_ws/src/vision_pipeline/vision_pipeline/rviz_markers.py：发布三维 Marker。
- ros2_ws/src/vision_interfaces/msg/LocalizedDetection*.msg：自定义三维检测接口。
- ros2_ws/src/vision_pipeline/launch/pipeline.launch.py 和 config/pipeline.yaml：启动和参数。
- scripts/inference_json.py：离线单图推理；当前需要检查相对路径分支。

核心公式：X=(u-cx)Z/fx，Y=(v-cy)Z/fy，Z=depth(u,v)。u/v 是像素中心，fx/fy/cx/cy 来自 CameraInfo，坐标结果的 frame_id 当前是 camera_color_optical_frame。

当前已知问题：
1. geometry.py 文件末尾有用户学习练习代码，导入时会打印 result2；不要擅自删除用户修改，先说明并经用户确认后清理到独立教学脚本。
2. scripts/inference_json.py 使用了未定义的 PROJECT_ROOT；相对模型路径可能 NameError。
3. pipeline.yaml 将 device 写死为 "0"，应评估改为 auto。
4. 虚拟深度为全图常量，不能证明真实三维定位精度。
5. 深度编码、尺寸、对齐和时间戳校验还不够严格。
6. YOLO 推理在订阅回调内同步运行，实时场景可能积压。
7. CI 尚未证明能在干净 ROS2 环境构建和运行完整链路。

常用命令（WSL）：
source /opt/ros/humble/setup.bash
source /mnt/d/robot_vision_project/.venv/bin/activate
cd /mnt/d/robot_vision_project/ros2_ws
/mnt/d/robot_vision_project/.venv/bin/python /usr/bin/colcon build --symlink-install
source install/setup.bash
ros2 launch vision_pipeline pipeline.launch.py image_path:=/path/to/test.jpg model_path:=/mnt/d/robot_vision_project/models/final/desktop_detection_v3.pt
ros2 topic echo /detections --once
ros2 topic echo /detections_3d --once
python -m pytest -q ros2_ws/src/vision_pipeline/test/test_geometry.py

下一步优先级：先检查并修复 inference_json.py 的 PROJECT_ROOT；单独处理 geometry.py 的练习代码；运行测试和单图推理；把 pipeline.yaml 的 device 变成可移植配置；为虚拟深度建立有真值的误差实验；然后再加 ROS2 集成测试和性能基线。真实相机到位后才做内参验证、对齐、TF2、桌面/机器人坐标和抓取接口。

用户是 Python、ROS2、计算机视觉初学者，目标是达到机器人视觉/ROS2 秋招面试水平。教学要求：每次只讲一个小主题但可以稍微多讲；先讲基础，再定位到具体文件和函数，解释 Python 语法、ROS2 概念、输入输出、调用关系，用小例子和练习验证，最后给面试式问题和参考回答。必须严格区分事实、推断、建议和冲突，不要把虚拟深度说成真实传感器能力，不要把相机坐标说成机器人坐标，不要代替用户完成全部学习过程。```

# 15. 最后

本上下文包已经把当前可从源码、文档、运行输出和历史对话确认的项目背景集中到一份文档中，并对未知信息、冲突和风险作了显式标记。它适合作为新 Agent 的启动材料、项目复盘底稿和后续实验索引。

本次整理没有修改业务代码。当前工作区仍有用户对 ros2_ws/src/vision_pipeline/vision_pipeline/geometry.py 的未提交学习练习；该修改应在用户确认后再决定是保留、移到教学脚本，还是用 if __name__ == "__main__": 隔离。scripts/inference_json.py 的相对路径缺陷也已记录，但本次没有擅自修复。

建议下次工作从 P0 项目开始，并在修复后补充新的运行日志、测试结果和提交号，使本文件保持可追溯。

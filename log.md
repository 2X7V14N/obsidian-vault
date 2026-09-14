# 操作日志

## [2026-06-04] init | 知识库初始化
- 创建目录结构：`raw/`、`wiki/concepts/`、`wiki/entities/`、`wiki/sources/`、`wiki/comparisons/`
- 创建 [[CLAUDE]] — Wiki Schema（操作手册 v1.0）
- 创建 [[wiki/index]] — Wiki 内容总目录
- 创建 [[log]] — 本操作日志
- 摄入第一篇源：[[wiki/sources/LLM Wiki 源]]（方法论原文，从 vault 根目录复制到 `raw/`）
- 创建概念页：[[wiki/concepts/LLM Wiki]]（~150 行，budding）、[[wiki/concepts/RAG]]（~120 行，budding）
- 创建实体页：[[wiki/entities/Memex]]（~110 行，budding）、[[wiki/entities/Obsidian]]（~120 行，budding）、[[wiki/entities/Anthropic Claude]]（~100 行，budding）
- 关键决策：中文优先作为写作语言；页面采用散文而非 bullet points 风格；frontmatter 使用 seedling/budding/evergreen 三级状态

## [2026-06-04] lint | 初次巡检（自我审查）
- 发现问题：初始版本的实体页（Memex、Obsidian、Anthropic Claude）内容过浅，只有骨架
- 已修复：重写所有核心页面，增加了实质性内容和交叉引用密度
- 已修复：CLAUDE.md 从"泛泛的规范列表"升级为"包含决策框架和具体操作步骤的操作手册"
- 已修复：LLM Wiki 源.md 源摘要从 bullet points 升级为"详细提取"（12 个主题段 + 关键引用表 + 对本 Wiki 的影响分析）
- 状态：所有核心页面 status 从 seedling 提升至 budding

## [2026-06-04] fix | 修复文件结构
- 问题：用户误操作导致文件被移入 `welcome/` 目录，产生大量陈旧副本
- 操作：删除 `welcome/` 全部内容（v1 陈旧副本 + 链接损坏的版本）
- 恢复：`raw/LLM Wiki.md`（源文件）、`LLM Wiki.md`（原始文章）、`Robot_Learning/`（研究目录）、`wiki/index.md`、`wiki/comparisons/`
- 验证：所有根级 Wiki 页面链接已统一为完整路径格式（`[[wiki/category/PageName]]`），无 `welcome/` 前缀残留或损坏链接

## [2026-06-04] restructure | 引入领域（Domain）组织
- 变更：从"纯按页面类型分类"改为"领域 + 类型"两级结构
- 新增领域：[[Robot_Learning/index|Robot_Learning]]（机器人感知），包含 concepts/、entities/、sources/、raw/
- 更新：[[CLAUDE]] Schema 已更新至适应领域组织（v3）
  - 目录结构新增领域模板
  - Frontmatter 新增 `domain` 字段
  - Ingest 流程适配领域路径
  - 链接规范明确跨领域引用格式
- 修复：[[log]] 中 3 处断裂/不规范链接（[[2_科研/index]]→[[wiki/index]]，[[CLAUDE.md]]→[[CLAUDE]]，[[log.md]]→[[log]]）
- 关键决策：`wiki/` 保留为跨领域元知识区；各领域在顶级目录下拥有独立子 wiki；领域模板可复制用于未来新领域

## [2026-06-04] ingest | [Robot_Learning] 4h上手C++版Opencv
- 创建源摘要：[[Robot_Learning/sources/4h上手C++版Opencv 源|4h上手C++版Opencv 源]]（~280 行，详细提取 + 知识缺口标注）
- 创建实体页：[[Robot_Learning/entities/OpenCV|OpenCV]]（核心架构、数据类型、imgproc/imgcodecs/highgui/objdetect 等模块概览、在机器人感知中的角色）
- 创建概念页：
  - [[Robot_Learning/concepts/图像的数字表示与颜色空间|图像的数字表示与颜色空间]]（像素、二值/灰度/BGR、HSV 原理 + Hue 环绕问题 + Trackbar 调参）
  - [[Robot_Learning/concepts/图像预处理：模糊与边缘检测|图像预处理：模糊与边缘检测]]（标准管道四步、高斯核数学、Canny 四阶段原理、形态学操作、开闭运算）
  - [[Robot_Learning/concepts/轮廓检测与形状识别|轮廓检测与形状识别]]（findContours 算法原理、Douglas-Peucker 多边形近似、形状分类判断、面积/周长/边界框/凸包）
  - [[Robot_Learning/concepts/透视变换与图像矫正|透视变换与图像矫正]]（单应性矩阵数学、角点重排序算法、仿射 vs 透视对比、AR/拼接/标定扩展应用）
  - [[Robot_Learning/concepts/基于Haar级联的目标检测|基于Haar级联的目标检测]]（Viola-Jones 三大创新：积分图/AdaBoost/级联结构、scaleFactor 和 minNeighbors 参数原理、Haar 与深度学习的定位对比）
- 更新：[[Robot_Learning/index]]（填充概念、实体、源三个分类）
- 知识拓展：每个概念页在源笔记基础上补充了算法原理（如 Canny 的双阈值滞后、Viola-Jones 的级联漏斗）、数学基础（如高斯核、单应性矩阵）、常见陷阱（如 BGR→RGB 转换、HSV Hue 环绕、findContours 破坏输入）、以及在机器人感知中的应用场景
- 略过：环境配置部分（VS/Xcode 安装步骤）
- 关键决策：OpenCV 作为机器人感知的基础设施级实体；概念页之间通过预处理管道形成上下游关系链（颜色空间→预处理→轮廓检测→透视变换）

## [2026-07-03] ingest | [Robot_Learning] OpenCV 轮廓提取与层级拓扑完全指南
- 创建源摘要：[[Robot_Learning/sources/OpenCV 轮廓提取与层级拓扑完全指南 源|轮廓层级拓扑完全指南 源]]（~180 行，12 个提取段 + 对本 Wiki 影响矩阵）
- 创建概念页：[[Robot_Learning/concepts/轮廓层级拓扑（Hierarchy）|轮廓层级拓扑（Hierarchy）]]（Vec4i 四字段详解、手工推演示例、四种检索模式行为差异、兄弟链表遍历、三个基于 hierarchy 的实战 + 5 个常见陷阱）
- 更新概念页：[[Robot_Learning/concepts/轮廓检测与形状识别|轮廓检测与形状识别]]（收紧 findContours 基础内容 + 添加 hierarchy 页交叉引用 + 新增圆度公式 + 补充四个实战场景入口 + 更新相关页面列出新源）
- 更新：[[Robot_Learning/index]]（新增 Hierarchy 概念页 + 第二篇源）
- 整合策略：第一篇 OpenCV 源覆盖"广度"（从 I/O 到人脸检测），第二篇覆盖"深度"（聚焦轮廓检测一条线）——两页分工明确：轮廓检测页 = 几何分析，Hierarchy 页 = 拓扑分析，互相链接形成"几何↔拓扑"互补
- 关键决策：Hierarchy 独立成页而非合并到轮廓检测页，因为"谁包含谁"的拓扑问题与"面积周长"的几何问题在概念上属于两个维度，独立成一页更聚焦、更可搜索

## [2026-07-04] restructure | [Robot_Learning] Concepts 按 OpenCV 课程章节重组
- 问题：原有 6 个概念页呈扁平列表，缺乏系统性——读者看不出技术间的依赖关系和管道顺序
- 解决：在 `concepts/` 下新建 `OpenCV/` 文件夹，将所有笔记按课程章节重新组织为 15 页：
  - 概述：[[Robot_Learning/concepts/OpenCV/介绍|介绍]]、[[Robot_Learning/concepts/OpenCV/图像简介|图像简介]]
  - 安装：[[Robot_Learning/concepts/OpenCV/安装 - Windows|Windows]]、[[Robot_Learning/concepts/OpenCV/安装 - Mac|Mac]]
  - 第1-4章：读取图像视频摄像头、基本功能、调整大小和裁剪、绘制形状和文字
  - 第5-8章：透视变形、颜色检测、形状/轮廓检测（合并了原轮廓检测几何分析页和 Hierarchy 拓扑页）、人脸检测
  - 项目1-3：虚拟画师、文档扫描仪、车牌检测器
- 旧页面处理：6 个原概念页转为跳转页（seedling 状态），保留历史链接不丢失
- 内容合并：[[Robot_Learning/concepts/OpenCV/第7章 - 形状轮廓检测|第7章]] 整合了原"轮廓检测与形状识别"（几何）和"轮廓层级拓扑"（拓扑）两页，以"第一部分/第二部分"分节
- 内容拆分：原"图像的数字表示与颜色空间"拆分为[[Robot_Learning/concepts/OpenCV/图像简介|图像简介]]（像素/BGR）和[[Robot_Learning/concepts/OpenCV/第6章 - 颜色检测|第6章]]（HSV/inRange）
- 更新：[[Robot_Learning/index]] — Concepts 区改为课程章节表格式

## [2026-07-04] fix | 删除旧概念页 + 创建 OpenCV CheatSheet
- 删除：6 个已迁移的旧概念页（跳转页）——内容已完整迁移至 `concepts/OpenCV/` 下各章节
- 修复：`Robot_Learning/sources/4h上手C++版Opencv 源`、`OpenCV 轮廓提取与层级拓扑完全指南 源`、`Robot_Learning/entities/OpenCV` 中的断裂链接，全部更新为指向新章节
- 创建：[[Robot_Learning/concepts/OpenCV/OpenCV_CheatSheet|OpenCV CheatSheet]]（~300 行）——基于 `raw/OpenCV_CheatSheet.md` 规范制作的 C++ 工程手册
  - 18 个板块：项目模板、Mat基础、I/O、颜色空间、预处理、阈值、形态学、边缘检测、轮廓检测、几何测量、几何变换、视频、任务流程、错误调试、踩坑记录、待验证内容
  - 每条函数按 6 部分模板（作用/代码/输入输出/参数/常见坑/场景）
  - 4 个任务流程（颜色检测+轮廓追踪、形状识别、透视矫正、人脸检测）
  - 4 条真实踩坑记录 + 快速问答索引
- 更新：[[Robot_Learning/index]] 和 [[Robot_Learning/concepts/OpenCV/介绍]] 加入 CheatSheet 链接

## [2026-07-07] update | [Robot_Learning] 第8章 人脸检测 — 接入 FaceDetectorYN (YuNet)
- 源材料：[[Robot_Learning/raw/OpenCV_Study_Notes|OpenCV_Study_Notes]]（OpenCV 5 适配 + DNN 人脸检测学习笔记）
- 更新：[[Robot_Learning/concepts/OpenCV/第8章 - 人脸检测|第8章 - 人脸检测]] — 从单一 Haar 级联方案重构为双方案结构：
  - **第一部分**：传统 Haar 级联（Viola-Jones）— 保留完整原理和用法，标注为 OpenCV 4.x 历史方案
  - **第二部分**：现代 DNN FaceDetectorYN / YuNet — 新增完整代码模板、faces 输出矩阵结构（15列）、C++ 语法要点（Ptr 智能指针、:: 作用域解析符、-> vs .、输出参数模式）、create() 6 参数详解
  - 页面顶部添加 OpenCV 5 版本警告，新增对比表格
- 同步更新：[[Robot_Learning/concepts/OpenCV/OpenCV_CheatSheet|CheatSheet]] — 头文件表新增 face.hpp；§15 新增 DNN 人脸检测流程模板；快速问答索引新增两条

## [2026-07-10] restructure | 重命名 Robot Perception → Robot_Learning + 整合学习计划
- 重命名：`Robot Perception/` → `Robot_Learning/`，全部 23 个文件 148 处链接已更新
- 整合：基于 `raw/Robot_Learning_Table.md` 创建 [[Robot_Learning/Robot Learning Plan|Robot Learning Plan]]（~180 行）
  - 34 个任务按 9 大类分组（语言工程基础、视觉算法、项目、机器人学、数学控制、ROS2、算法DL、仿真3D、扩展前沿）
  - 每个任务附带：优先级（P0–P3）、状态图标（✅🔄⬜）、视觉进度条（████░░）、学习要求说明
  - 当前进度：OpenCV C++ 80%✅、C++ 基础 60%🔄、CMake 30%🔄、抓取V1 20%🔄、LeetCode 10%🔄
- 更新：[[Robot_Learning/index]] — 页面顶部嵌入学习计划总览进度表，并链接到详情页

## [2026-07-23] ingest | [1.学习/Robot_perception] 动手学深度学习 预备知识
- 源材料：`1.学习/Robot_Learning/Robot_perception/raw/` 中的 4 份网页剪藏（第 1 章引言 + 第 2 章预备知识）
- 创建概念页（2 篇，按章节合并）：
  - [[Ch01 深度学习概论|Ch01 深度学习概论]] — 第 1 章引言：ML/DL 定义、四大核心组件、监督/无监督/强化学习分类、历史脉络、DL 核心特点
  - [[Ch02 数据操作、预处理与线性代数|Ch02 预备知识]] — 第 2 章合并（2.1+2.2+2.3）：张量操作与广播、pandas 数据预处理、矩阵乘法与范数
- 补充内容：广播机制的维度对齐推理、独热编码 → Embedding 的扩展讨论、矩阵乘法的几何视角（变换复合）、范数与损失函数的深层联系
- 更新：[[1.学习/Robot_Learning/Robot_perception/index]] — 深度学习课程笔记改为章节表格（Ch01 + Ch02 + 2 个规划页）
- 更新：[[Deep Learning Plan|深度学习课程学习计划]] — 阶段 1 状态 ⬜ → 🔄
- 关键决策：按 D2L 原书章节结构组织笔记（Ch01=第1章，Ch02=合并2.1~2.3）；同一章的节内容合并为单篇笔记以保持知识连贯性；命名对齐 OpenCV 和相机标定的 `Ch0X` 格式
- 上一条日志中记录的 4 篇初始笔记已删除（被 Ch01 + Ch02 替代）
- 更新标题：Ch02 重命名为 [[Ch02 深度学习的基础设施|Ch02 深度学习的基础设施]]

## [2026-07-23] ingest | [1.学习/Robot_perception] 动手学深度学习 Ch03 线性神经网络
- 源材料：`Clippings/` 中的 3 份网页剪藏
  - `3.1. 线性回归.md` — 理论：线性模型、平方损失、解析解、SGD、矢量化、正态分布与 MLE
  - `3.2. 线性回归的从零开始实现.md` — 从零实现六步：合成数据→迭代器→初始化→模型→SGD→训练循环
  - `3.3. 线性回归的简洁实现.md` — 框架实现：DataLoader + nn.Linear + MSELoss + optim.SGD
- 创建概念页（1 篇）：
  - [[Ch03 线性神经网络|Ch03 线性神经网络]] — 理论（线性模型/平方损失/解析解/SGD/矢量化的概率解释）→ 从零实现六步 → 框架简洁实现，三段递进
- 补充内容：MSE 的概率解释（高斯噪声假设 → MLE 等价性）、L1 vs L2 损失的选择依据、权重不能全初始化为 0 的原因、梯度清零的必要性、每轮打乱数据的原理
- 更新：[[1.学习/Robot_Learning/Robot_perception/index]] — 深度学习课程笔记新增 Ch03
- 更新：[[Deep Learning Plan|深度学习课程学习计划]] — 阶段 2 状态 ⬜ → 🔄
- 关键决策：将 3.1~3.3 合并为一篇 Ch03，按"理论→从零→框架"递进；代码以 PyTorch 为主线（Paddle/TF/MXNet 略去）；强调训练循环模板的通用性（ResNet/Transformer 也用同套流程）
- 补充：整合 `PyTorch_QA_Notes.md` 的核心问答（TensorDataset 订书机比喻、书与书签迭代器比喻、下山优化器比喻、梯度清零底层逻辑）到 3.2 和 3.3 各对应位置
- 新增：整合 `3.4. softmax回归.md` 为 Ch03 的 3.4 节 — Softmax 回归（独热编码、多输出架构、Softmax 运算与温度参数、交叉熵损失与梯度推导、信息论视角的熵与惊异度、精度评估）
- 新增：整合 `3.5~3.7` 为 Ch03 的 3.5~3.7 节，与线性回归形成对称结构：
  - 3.5 Fashion-MNIST 数据集（10 类 28×28 灰度服装图像、ToTensor 归一化、展平 784 维、DataLoader 并行加载）
  - 3.6 Softmax 从零实现（参数初始化→Softmax 配分函数→高级索引选概率→交叉熵→精度 Accumulator→训练循环复用模板）
  - 3.7 Softmax 简洁实现（nn.Flatten + nn.Linear、LogSumExp 数值稳定技巧、CrossEntropyLoss 吃 logits 不吃概率）

## [2026-07-24] ingest | [1.学习/Robot_perception] 动手学深度学习 Ch04 多层感知机
- 源材料：`Clippings/` 中 4.1~4.3 三份剪藏（MLP 理论 + 从零实现 + 简洁实现）
- 创建概念页（1 篇）：
  - [[Ch04 多层感知机|Ch04 多层感知机]] — 4.1 理论（线性瓶颈/隐藏层/激活函数/通用近似）+ 4.2 三大激活函数对比 + 4.3 从零实现 + 4.4 框架实现
- 补充内容：倒置图像测试说明线性模型局限、仿射函数堆叠=线性的数学证明、"深度>宽度"的通用近似解读、ReLU 梯度消失解决方案的机制解释、激活函数选择指南表
- 更新：[[1.学习/Robot_Learning/Robot_perception/index]] — 新增 Ch04
- 更新：[[Deep Learning Plan|深度学习课程学习计划]] — 阶段 3 状态 ⬜ → 🔄
- 关键决策：激活函数对比独立成节（4.2），而非散落在理论中；强调训练模板不变（train_ch3 跨三章复用）；代码聚焦 PyTorch
- 补充：整合 `4.4. 模型选择、欠拟合和过拟合.md` 为 Ch04 的 4.5 节 — 训练/泛化误差（鲍勃/考试类比）、i.i.d. 假设及其违背、模型复杂三因素、验证集与 K 折交叉验证、欠拟合 vs 过拟合诊断、三阶/一阶/全阶多项式回归演示（正常→欠拟合→过拟合）

## [2026-08-03] ingest | [1.学习/Robot_perception] 动手学深度学习 Ch04_4.5~4.9（后半部分）
- 源材料：`Clippings/` 中 4.5~4.9 五份剪藏（权重衰减 + Dropout + 前向/反向传播 + 数值稳定性/初始化 + 环境/分布偏移）
- 补充到 Ch04：
  - **4.6 权重衰减**（$L_2$ 正则化原理/SGD 更新推导/$L_1$ vs $L_2$ 对比/高维 200 特征 20 样本实验/PyTorch 分组参数用法）
  - **4.7 Dropout**（偏差-方差权衡/随机标签过拟合 90% 泛化差距/无偏设计 $h/(1-p)$/有性繁殖类比/从零实现掩码/框架 `nn.Dropout` /权重衰减 vs Dropout 对比表）
  - **4.8 前向/反向传播与计算图**（前向 6 步公式链/计算图/反向传播链式法则逐步推导（损失+正则化双路径）/OOM 显存原因/自动微分意义）
  - **4.9 数值稳定性与初始化**（梯度消失/爆炸的数学根源/$L-l$ 个矩阵连乘/随机矩阵 100 次连乘演示/Sigmoid 致命伤数学证明/对称性打破/ Xavier 方差推导/$n_\text{in} \sigma^2=1$ 矛盾与调和平均/He 初始化）
  - **4.10 环境与分布偏移**（协变量/标签/概念三种偏移/医学诊断+自动驾驶+坦克检测真实失败案例/重要性加权纠正/协变量偏移纠正的对数几率回归法/批量→在线→老虎机→强化学习光谱/预测性警务反馈循环/伦理）
- 更新：[[1.学习/Robot_Learning/Robot_perception/index]] — Ch04 简介扩展
- 更新：[[Deep Learning Plan|深度学习课程学习计划]] — 阶段 3 状态 🔄 → ✅
- 关键决策：将 4.7 反向传播的公式推导保留为 step-by-step LaTeX（价值密度高）；4.10 强调真实失败案例（比纯理论更有记忆点）；小结重构为四大板块（架构/正则化/机理/稳定性/部署）

## [2026-08-03] ingest | [1.学习/Robot_perception] 动手学深度学习 Ch05 深度学习计算
- 源材料：`Clippings/` 中 5.1~5.6 六份剪藏（层和块 + 参数管理 + 延后初始化 + 自定义层 + 读写文件 + GPU）
- 创建概念页（1 篇）：
  - [[Ch05 深度学习计算|Ch05 深度学习计算]]
    - **5.1 层和块**（nn.Module 块抽象/Sequential/自定义 MLP 从零写/MySequential 揭示 _modules 机制/forward 中的控制流与参数共享）
    - **5.2 参数管理**（state_dict 访问/named_parameters/内置与自定义初始化/nn.Parameter 注册机制/参数绑定与梯度累加）
    - **5.3 延后初始化**（PyTorch vs MXNet 设计哲学分野/LazyLinear 折中方案）
    - **5.4 自定义层**（无参数层 CenteredLayer/带参数层 MyLinear/nn.Parameter 一等公民地位）
    - **5.5 读写文件**（torch.save/load 张量/state_dict 持久化/架构定义留在代码中）
    - **5.6 GPU**（torch.device/try_gpu 模式/数据搬运比计算慢/CPU-GPU 铁律：同一计算所有张量同一设备/net.to(device) 递归移动）
- 补充：PyTorch vs MXNet 延后初始化设计哲学对比；`_modules` 与普通 list 的本质区别；`nn.Parameter` 自动注册机制解析；架构=代码+参数文件的工程实践法则
- 更新：[[1.学习/Robot_Learning/Robot_perception/index]] — 新增 Ch05
- 更新：[[Deep Learning Plan|深度学习课程学习计划]] — 阶段 4 状态 ⬜ → ✅

## [2026-08-04] ingest | [1.学习/Robot_perception] 动手学深度学习 Ch06 卷积神经网络
- 源材料：`Clippings/` 中 6.1~6.6 六份剪藏（从全连接到卷积 + 图像卷积 + 填充步幅 + 多通道 + 汇聚层 + LeNet）
- 创建概念页（1 篇）：
  - [[Ch06 卷积神经网络|Ch06 卷积神经网络]]
    - **6.1 从全连接到卷积**（百万像素→十亿参数灾难/沃尔多游戏引出平移不变性/局部性/四阶张量→两个假设→卷积公式/通道三维扩展/互相关 vs 卷积的术语辨析/归纳偏置概念）
    - **6.2 图像卷积**（corr2d 从零实现/$3 \times 3$→$2 \times 2$ 输出公式/Conv2D 类/边缘检测：`[1,-1]` 垂直边缘检测器/学习卷积核 10 epoch 收敛到 `[1.001, -0.974]`/感受野定义与深层扩大机制）
    - **6.3 填充和步幅**（padding 保持尺寸/padding=$k$−1+奇数核同大/stride 降采样/输出尺寸通用公式）
    - **6.4 多输入多输出通道**（$c_i$ 通道按通道求和/$c_o$ 通道独立卷积核/$1 \times 1$ 卷积=通道间全连接/ResNet bottleneck 伏笔）
    - **6.5 汇聚层**（MaxPool/AvgPool/无参数+平移鲁棒性/步幅默认=窗口大小/多通道独立操作/填充和步幅的池化扩展）
    - **6.6 LeNet**（1989 首个 CNN/卷积编码器+全连接分类器/$28 \times 28 \times 1 → 5 \times 5 \times 16$ 空间-通道转换/$150$ vs $200,704$ 参数对比/train_ch6 GPU 训练至 82% 精度/Sigmoid→ReLU 现代改进提示）
- 补充：全连接 vs CNN 参数对比表；归纳偏置概念的引入与 ViT 对比的伏笔；$1 \times 1$ 卷积在 ResNet/Inception 中的作用预告；LeNet 简化版数据流可视化
- 更新：[[1.学习/Robot_Learning/Robot_perception/index]] — 新增 Ch06
- 更新：[[Deep Learning Plan|深度学习课程学习计划]] — 阶段 5 状态 ⬜ → 🔄

## [2026-08-09] ingest | [1.学习/Robot_perception] 动手学深度学习 Ch07 现代卷积神经网络
- 源材料：`Clippings/` 中 7.1~7.7 六份剪藏（AlexNet + VGG + GoogLeNet + BN + ResNet + DenseNet，缺 7.3 NiN）
- 创建概念页（1 篇）：
  - [[Ch07 现代卷积神经网络|Ch07 现代卷积神经网络]]
    - **7.1 AlexNet**（2012 ImageNet 冠军/手工特征→端到端学习/SIFT-SURF-HOG 时代终结/ImageNet+GPU 双驱动/$11 \times 11 → 5 \times 5 → 3 \times 3$ 架构/第一层滤波器可视化/ReLU+Dropout+数据增强三大支柱/88~89% 精度）
    - **7.2 VGG**（块设计范式/VGG 块=$n \times [3 \times 3\text{ Conv+ReLU}]$+MaxPool/通道翻倍+分辨率减半/两个 $3 \times 3$=$5 \times 5$ 感受野/$25088→4096$ FC 参数爆炸问题/ratio=4 缩减训练/92% 精度）
    - **7.4 GoogLeNet**（Inception 块=四条并行路径/$1 \times 1$ 降维核心技巧/计算量 5× 减少/9 个 Inception 块=5 模块/GlobalAvgPool 取代 FC/盗梦空间命名趣事/90% 精度）
    - **7.5 批量规范化**（BN=标准化+$\gamma$/$\beta$/全连接→特征维度 BN vs 卷积→通道维度 BN/训练用批统计+移动平均/推理用全局统计/学习率 0.1→1.0 十倍放大/$32000$ examples/sec/内部协变量偏移争议/Santurkar 损失曲面平滑解释）
    - **7.6 ResNet**（深层退化非过拟合/函数类嵌套论证/残差块=学习 $f(x)-x$ 而非 $f(x)$/跳跃连接加法门/1×1 conv 匹配形状/ResNet-18=18 层/Conv→BN→ReLU→Conv→BN→+input→ReLU 标准范式/训练 99.7% 测试 92.4%）
    - **7.7 DenseNet**（ResNet 加法→DenseNet 通道拼接/泰勒展开类比/稠密块=每层接收所有之前层输出/增长率 k=32/过渡层=1×1 减半通道+AvgPool 减半空间/通道爆炸→显存杀手/参数利用率极高）
  - 下载图片 11 张（alexnet.svg/filters.png/vgg.svg/inception.svg/inception-full.svg/functionclasses.svg/residual-block.svg/resnet-block.svg/resnet18.svg/densenet-block.svg/densenet.svg）
- 补充：AlexNet vs LeNet 对比表；VGG 两个 $3 \times 3$=$5 \times 5$ 感受野数学推理；Inception $1 \times 1$ 降维计算量分析；BN "内部协变量偏移"学术争议与 Santurkar 替代解释；函数类嵌套的形式化论证；ResNet vs DenseNet 对比表；DenseNet 显存消耗分析
- 更新：[[1.学习/Robot_Learning/Robot_perception/index]] — 新增 Ch07

## [2026-08-09] ingest | [1.学习/Robot_perception] 动手学深度学习 Ch13 计算机视觉
- 源材料：`Clippings/` 中 13.1~13.2 两份剪藏（图像增广 + 微调）
- 创建概念页（1 篇）：
  - [[Ch13 计算机视觉|Ch13 计算机视觉]]
    - **13.1 图像增广**（翻转/随机裁剪+缩放/颜色抖动/组合使用/训练时增广+测试时确定的模式/CIFAR-10 + ResNet-18 实战 → 85~94% 精度/数据增强=免费正则化）
    - **13.2 微调**（迁移学习四步流程：预训练→复制骨干→替换输出层→分组学习率/热狗识别：2800 张图片/微调 93.9% vs 从零 84.3%/三种变体策略：全微调/部分微调/特征提取/冻结操作/10× 学习率差异的动机/与 Ch04 深层特征通用性串联）
  - 下载图片 1 张（finetune.svg）
- 补充：增广与 Dropout/BN 在"训练时随机+推理时确定"模式上的统一性解释；微调三种策略表和适用场景；热狗识别微调 vs 从零训练的量化对比；与 Ch04 深层特征层次的串联
- 更新：[[1.学习/Robot_Learning/Robot_perception/index]] — 新增 Ch13
- 更新：[[Deep Learning Plan|深度学习课程学习计划]] — 阶段 6 状态 ⬜ → ✅

## [2026-08-13] ingest | [1.学习/Robot_perception] 动手学深度学习 Ch08 循环神经网络
- 源材料：`Clippings/` 中 8.1~8.7 七份剪藏（序列模型 + 文本预处理 + 语言模型 + RNN + 从零实现 + 简洁实现 + BPTT）
- 创建概念页（1 篇）：
  - [[Ch08 循环神经网络|Ch08 循环神经网络]]
    - **8.1 序列模型**（序列普遍性/锚定效应-享乐适应-季节性/自回归 vs 隐变量自回归/马尔可夫模型/内插 vs 外推/误差累积导致多步预测崩溃）
    - **8.2 文本预处理**（词元化-单词 vs 字符/词表构建/`<unk>` 处理/索引化）
    - **8.3 语言模型与数据集**（联合概率链式分解/n 元语法与维数灾难/拉普拉斯平滑/齐普夫定律→深度学习动机/随机采样 vs 顺序分区）
    - **8.4 循环神经网络**（隐状态 vs 隐藏层概念辨析/$\mathbf{H}_t = \phi(\mathbf{X}_t \mathbf{W}_{xh} + \mathbf{H}_{t-1} \mathbf{W}_{hh} + \mathbf{b}_h)$/参数跨时间步共享/字符级语言模型/困惑度=调和平均数）
    - **8.5 从零实现**（独热编码/tanh 激活/梯度裁剪 $g \leftarrow \min(1, \theta/\|g\|)g$/预热期/500 epoch 困惑度 1.0 生成文本）
    - **8.6 简洁实现**（nn.RNN/输出层单独定义/速度 67k→400k 词元每秒）
    - **8.7 BPTT**（沿时间展开的反向传播/梯度含 $(\mathbf{W}_{hh}^\top)^k$ 幂次/三种策略：完全计算-常规截断-随机截断/截断=轻量正则化）
  - 下载图片 7 张（ftse100.png/sequence-model.svg/timemachine-5gram.svg/rnn.svg/rnn-train.svg/rnn-bptt.svg/truncated-bptt.svg）
- 补充：隐状态 vs 隐藏层的概念辨析；齐普夫定律作为深度学习语言模型的动机；困惑度的调和平均数直觉；截断 BPTT 的轻量正则化本质；与 Ch04 梯度消失/爆炸的串联
- 更新：[[1.学习/Robot_Learning/Robot_perception/index]] — 新增 Ch08
- 备注：RNN 原属"暂时跳过"清单（集 149-180），现已被学习，后续 GRU/LSTM/Seq2Seq 仍待补充

## [2026-08-13] ingest | [1.学习/Robot_perception] 动手学深度学习 Ch09 现代循环神经网络
- 源材料：`Clippings/` 中 9.1~9.8 八份剪藏（GRU + LSTM + 深度 RNN + 双向 RNN + 机器翻译数据集 + 编码器-解码器 + seq2seq + 束搜索）
- 创建概念页（1 篇）：
  - [[Ch09 现代循环神经网络|Ch09 现代循环神经网络]]
    - **9.1 GRU**（门控动机：选择性记忆/重置门+更新门/候选隐状态/凸组合 $\mathbf{H}_t = \mathbf{Z}_t \odot \mathbf{H}_{t-1} + (1-\mathbf{Z}_t) \odot \tilde{\mathbf{H}}_t$/门控=梯度高速公路类比 ResNet）
    - **9.2 LSTM**（三门+记忆元/记忆元更新 $\mathbf{C}_t = \mathbf{F}_t \odot \mathbf{C}_{t-1} + \mathbf{I}_t \odot \tilde{\mathbf{C}}_t$/仅隐状态传输出/LSTM vs GRU 对比表）
    - **9.3 深度 RNN**（多层堆叠/横向+纵向双向传递/不同时间尺度建模/调参敏感）
    - **9.4 双向 RNN**（下文对理解的重要性/HMM 前向-后向同构/双向 RNN 不能预测下一词元/正确用途：编码、标注）
    - **9.5 机器翻译与数据集**（序列转换问题/单词级词元化/双词表/`<pad>` `<bos>` `<eos>` 特殊词元/截断+填充）
    - **9.6 编码器-解码器**（变长输入→固定状态→变长输出的通用架构/编码器解码器不必同类）
    - **9.7 seq2seq**（RNN 编码器+解码器/嵌入层替代独热/遮蔽损失/强制教学/BLEU 评估/英→法翻译示例）
    - **9.8 束搜索**（贪心搜索的局部最优陷阱/穷举搜索不可行/束搜索折中/长度惩罚/三种策略对比表）
  - 下载图片 17 张（gru-1~3.svg/lstm-0~3.svg/deep-rnn.svg/hmm.svg/birnn.svg/encoder-decoder.svg/seq2seq 三张/s2s-prob 两张/beam-search.svg）
- 补充：GRU 门控与 ResNet 跳跃连接的类比；LSTM vs GRU 对比表；双向 RNN 的"不能预测下一词元"陷阱；束搜索三策略对比表
- 更新：[[1.学习/Robot_Learning/Robot_perception/index]] — 新增 Ch09
- 备注：GRU/LSTM/Seq2Seq 原属"暂时跳过"清单（集 149-180），现已全部学习

## [2026-08-18] ingest | [1.学习/Robot_perception] 动手学深度学习 Ch10 注意力机制
- 源材料：`Clippings/` 中 10.1~10.7 七份剪藏（注意力提示 + Nadaraya-Watson 核回归 + 评分函数 + Bahdanau 注意力 + 多头注意力 + 自注意力/位置编码 + Transformer）
- 创建概念页（1 篇）：
  - [[Ch10 注意力机制|Ch10 注意力机制]]
    - **10.1 注意力提示**（注意力稀缺性/非自主性 vs 自主性提示/查询-键-值三要素映射/注意力 vs 全连接层的本质区别）
    - **10.2 Nadaraya-Watson 核回归**（平均汇聚→注意力汇聚/注意力权重=softmax(高斯距离)/非参数 vs 带参数/注意力本质=查询动态加权平均）
    - **10.3 注意力评分函数**（加性注意力 vs 缩放点积注意力/$\sqrt{d}$ 稳定梯度的数学原因/掩蔽 softmax）
    - **10.4 Bahdanau 注意力**（seq2seq 固定上下文的信息瓶颈/动态上下文变量/查询=解码器隐状态、键值=编码器隐状态/注意力可视化可解释性）
    - **10.5 多头注意力**（多组线性投影并行学习多种关注模式/$p_o/h$ 参数分配/张量变换并行计算）
    - **10.6 自注意力与位置编码**（查询=键=值/CNN vs RNN vs 自注意力三表对比/$O(1)$ 顺序操作+$O(1)$ 路径长度/$O(n^2d)$ 二次代价/正弦位置编码/相对位置线性变换性质）
    - **10.7 Transformer**（纯注意力架构/编码器两子层+解码器三子层/掩蔽自注意力保持自回归/PositionWise FFN/残差连接+层规范化/层规范化 vs 批量规范化/注意力可视化/Transformer 对语言、视觉、多模态的深远影响）
  - 下载图片 8 张（eye-coffee.svg/eye-book.svg/qkv.svg/attention-output.svg/seq2seq-attention-details.svg/multi-head-attention.svg/cnn-rnn-self-attention.svg/transformer.svg）
- 补充：查询键值与生物学的映射表；CNN/RNN/自注意力对比表；$\sqrt{d}$ 缩放的方差推导；位置编码的二进制类比；Transformer 作为"通用关系建模工具"的范式转折意义
- 更新：[[1.学习/Robot_Learning/Robot_perception/index]] — 新增 Ch10
- 更新：[[Deep Learning Plan|深度学习课程学习计划]] — 阶段 8 状态 ⬜ → ✅（全部 8 阶段主线课程完成）

## [2026-08-12] ingest | [1.学习/Robot_perception] 动手学深度学习 Ch13_13.3~13.11
- 源材料：`Clippings/` 中 13.3~13.11 九份剪藏（目标检测 + 语义分割）
- 补充到 Ch13（在原 13.1~13.2 基础上追加）：
  - **13.3 目标检测与边界框**（角点/中心两种表示、坐标转换）
  - **13.4 锚框**（锚框生成/$n+m-1$ 组合/IoU=交集/并集/贪心分配算法/偏移标准化/NMS 去重/Soft-NMS 提及）
  - **13.5 多尺度检测**（浅层小目标+深层大目标/特征图尺寸与感受野对应）
  - **13.6 目标检测数据集**（香蕉检测数据/标签格式 [类别,x1,y1,x2,y2]）
  - **13.7 SSD**（单阶段检测器/基础网络+5 尺度特征块/5444 锚框/图/类别预测层+边界框预测层/$L_1$ 平滑损失/20 epoch 训练收敛）
  - **13.8 R-CNN 系列**（R-CNN→Fast→Faster→Mask 演化表/RoI Pooling vs RoI Align/RPN 锚框提候选区域）
  - **13.9 语义分割与数据集**（像素级分类/语义 vs 实例 vs 图像分割/Pascal VOC2012/21 类/RGB 标签映射）
  - **13.10 转置卷积**（"一到多"广播/$2 \times 2$ in→$3 \times 3$ out 示例/填充作用于输出/步幅拉伸中间结果/矩阵转置命名的由来/形状逆转≠数值逆转）
  - **13.11 全卷积网络 FCN**（ResNet-18 骨干+$1 \times 1$ conv+转置 conv 32× 放大/双线性插值初始化/逐像素交叉熵/84~85% 精度/全卷积范式开创）
  - 下载图片 12 张（iou.svg/anchor-label.svg/ssd.svg/r-cnn.svg/fast-rcnn.svg/roi.svg/faster-rcnn.svg/mask-rcnn.svg/segmentation.svg/trans_conv.svg/trans_conv_stride2.svg/fcn.svg）
- 补充：R-CNN 家族演化表；NMS 贪心算法详解；转置卷积"广播"工作方式；FCN 深远影响（U-Net/DeepLab/特征金字塔）
- 更新：[[1.学习/Robot_Learning/Robot_perception/index]] — Ch13 简介扩展
- 更新：[[Deep Learning Plan|深度学习课程学习计划]] — 阶段 7 状态 ⬜ → ✅
## [2026-08-21] git | [全库] 建立云端备份
- 操作：初始化 Git 仓库 + 创建 GitHub 私有远程仓库 `2X7V14N/obsidian-vault`
- 初始提交：`10d6768 Initial commit: Obsidian vault 全量备份`（251 个文件）
- 分支：`main`，已关联 `origin` 并推送
- 排除项：`.claude/`、`.claudian/`（内部会话数据）、`.obsidian/workspace.json`（机器相关布局）、系统垃圾文件
- 空目录处理：`3.日常/`、`4.炒股/` 放入 `.gitkeep` 保留结构
- 关键决策：私有仓库（个人知识库不公开）；后续建议安装 Obsidian Git 插件实现自动推送

## [2026-08-27] ingest | [Robot_perception] ROS2 入门 21 讲
- 源材料：`Clippings/` 中 9 份剪藏（古月居《ROS2 入门 21 讲》引言 + 1.1~1.5 + 概念总览 + 2.1~2.2）
- 创建：[[concepts/ROS2/Ch01 认识 ROS 与 ROS2|Ch01]]、[[concepts/ROS2/Ch02 ROS2 与 ROS1 的对比|Ch02]]、[[concepts/ROS2/Ch03 ROS2 环境安装与配置|Ch03]]、[[concepts/ROS2/Ch04 ROS2 命令行操作|Ch04]]、[[concepts/ROS2/Ch05 核心概念：工作空间与功能包|Ch05]]
- 下载图片：87 张 → `concepts/ROS2/attachments/`（按来源章节加前缀 r11/r12/r13/r14/r15/r21/r22/intro/c2ov）
- 更新：[[Robot_perception/index]] — 新增「ROS2 课程笔记」章节表；ROS2 基础进度 0% → 15%
- 补充：将 9 份散落剪藏整合为 5 章递进结构（认识→对比→安装→命令行→核心概念）；补充 DDS/RMW 中间层逻辑、colcon/rosdep 工作流说明
- 关键决策：章节命名对齐 Ch0x 规范；图片本地化嵌入（Obsidian wiki-link）；跳过装饰图（footer/交流群）

## [2026-09-12] ingest | [Robot_perception] 项目一：ROS2 + YOLO + RGB-D 桌面目标检测与三维定位
- 源材料：`Robot_Learning/Robot_perception/raw/project_context_package.md`（706 行项目上下文包）
- 创建：[[1.学习/Robot_Learning/Robot_perception/concepts/项目一：基于 ROS2、YOLO 与 RGB-D 的桌面目标检测及三维定位系统]]
- 更新：[[1.学习/Robot_Learning/Robot_perception/index]] — 新增「项目笔记」章节表；updated → 2026-09-12
- 关键决策：按「保留原文 + 补充解释」整合，保留原文的【事实】【推断】【风险】标注；为 5 个核心技术概念（针孔反投影、深度窗口中值、frame_id、ROS2 回调机制、时间戳同步）添加解释性 callout；将原文分散的 mermaid 架构图/时序图/数据流图原样保留；新增第 14 章「能力边界总结（面试口径）」把散落的事实边界汇总成可用的面试口径
- 互链：关联 Ch13 计算机视觉（YOLO）、相机标定 Ch01/Ch02/Ch04（针孔模型/RGB-D）、ROS2 Ch04/Ch05（命令行/功能包）、深度学习 Ch06/Ch07（CNN 骨干）
- 备注：源文件记录了 10 个已知问题（P-01~P-10）和 18 个学习问答（Q-01~Q-18），均已保留为表格

## [2026-09-12] update | [Robot_perception] 项目一笔记新增代码详解章节
- 更新：[[1.学习/Robot_Learning/Robot_perception/concepts/项目一：基于 ROS2、YOLO 与 RGB-D 的桌面目标检测及三维定位系统]] — 新增第 16 章「代码详解：VirtualRgbdPublisher 虚拟 RGB-D 发布节点」
- 内容：__init__ 参数声明（含参数一览表）、publish_frame 发布机制、self 前缀语义、main() 的 try/except/finally 安全退出、16.5 与笔记其他章节的关联索引表
- 更新：[[1.学习/Robot_Learning/Robot_perception/index]] — 项目一描述补充代码详解要点
- 关键决策：将代码拆解作为第 16 章放在正文末尾（附录性质），不改动原有 0~15 章结构；新增「与笔记其他部分的关联」表把代码与原理/问答/问题章节打通

## [2026-09-15] update | [深度学习] Ch10 补充「缩放点积」方差推导
- 更新：[[1.学习/Robot_Learning/Robot_perception/concepts/深度学习/Ch10 注意力机制]] 10.3 节 — 把原来一句话的"除以 √d 的原因"扩展为完整小节
- 新增内容：点积方差 = d 的完整推导（单乘积项均值/方差 → d 项方差相加）；数值失控后果表（d=64/512/4096）；softmax 饱和导致梯度消失的导数解释；除以 √d 修正方差至 1
- 新增两个 callout：「为什么是 √d 而不是 d」（除以 d 会矫枉过正导致注意力失去区分度）；「梯度消失 vs 梯度爆炸」辨析（本节防的是消失，爆炸是另一回事，指向 Ch04 4.9 节）
- 关键决策：保留原句核心结论，把推导放在 10.3 节内部作为独立小节（而非附录），因为它直接支撑 √d 这个设计选择

---
tags: [ros2, command-line, terminal, linux, debugging, robot-learning]
created: 2026-08-27
updated: 2026-08-27
status: budding
domain: robot-learning
---

# Ch04 ROS2 命令行操作

> 来源：《ROS2 入门 21 讲》古月居 · 1.4 ROS2 命令行操作

**命令行（Command Line）是 ROS2 中最重要的一种调试工具。** 在运行小海龟案例时你已经接触过它。本章以"小海龟仿真"为试验场，系统性地掌握 Linux 基础命令和 ROS2 核心命令——运行节点、查看节点/话题、发布话题、调用服务、发送动作目标、录制数据。

## 为什么需要命令行

类似科幻电影中的片段，命令行操作异常炫酷，但上手并不容易。为什么这种看似不便的方式被一直保留至今？

![[attachments/r14_image-20220526083430852.jpg]]

打个比方：去商场买衣服，现成的衣服虽然多，但不一定满足需求。服装定制服务可以根据现有款式结合自己的需求自主定制——操作麻烦一点，但灵活度高。现成的衣服就像可视化软件，被精心设计但不一定能完全解决问题；**定制服务的布料和工具就类似命令行**，可以用这些材料灵活定制各种功能，约束小很多。

![[attachments/r14_image-20220526084719912.jpg]]

## Linux 中的命令行

命令通过字符方式输入，输入工具叫 **Terminal（终端）**。启动方式有多种：

- 在应用列表中打开
- 快捷键 **Ctrl+Alt+T**
- 鼠标右键选择"打开终端"

![[attachments/r14_image-20220524082310466.jpg]]

初次上手会觉得命令行枯燥难记，这很正常。常用命令也就一二十个，其他在需要时搜索即可。

### 常用命令操作

| 命令 | 语法 | 功能 |
|------|------|------|
| **cd** | `cd <目录路径>` | 改变工作目录；无参数则回到主目录 |
| **pwd** | `pwd` | 显示当前工作目录的绝对路径 |
| **mkdir** | `mkdir [选项] <目录名称>` | 创建目录/文件夹 |
| **ls** | `ls [选项] [目录名称…]` | 列出目录中的文件列表 |
| **gedit** | `gedit <文件名称>` | 打开 gedit 编辑器编辑文件，无此文件则新建 |
| **mv** | `mv [选项] <源> <目的>` | 文件/目录改名或移动 |
| **cp** | `cp [选项] <源> <目的>` | 拷贝文件/目录 |
| **rm** | `rm [选项] <文件或目录…>` | 删除文件或目录 |
| **sudo** | `sudo [选项] [指令]` | 以系统管理员权限执行指令 |

![[attachments/r14_image-20220526085342690.jpg]]

![[attachments/r14_image-20220526100604610.jpg]]

![[attachments/r14_image-20220526100617130.jpg]]

![[attachments/r14_image-20220526100626936.jpg]]

![[attachments/r14_image-20220526100643866.jpg]]

![[attachments/r14_image-20220526100724300.jpg]]

这些命令不需要死记硬背，一边用一边查，用多了自然熟悉。

## ROS2 中的命令行

ROS2 命令行的操作机制与 Linux 相同，但所有操作都集成在一个 **ros2 主命令** 中，后边第一个参数表示不同的操作目的（node/topic/service/action/bag…），具体操作干什么，继续跟一系列参数。

![[attachments/r14_image-20220524082444725.jpg]]

接下来以**小海龟仿真**为例，感受 ROS2 命令行的主要功能。先启动小海龟：

```bash
$ ros2 run turtlesim turtlesim_node
$ ros2 run turtlesim turtle_teleop_key
```

### 运行节点：ros2 run

`ros2 run` 用于运行某个功能包中的节点，语法为 `ros2 run <功能包> <可执行文件>`：

```bash
$ ros2 run turtlesim turtlesim_node      # 运行海龟仿真节点
$ ros2 run turtlesim turtle_teleop_key   # 运行键盘控制节点
```

![[attachments/r14_image-20220524082526919.jpg]]

### 查看节点：ros2 node

当前运行的 ROS 系统中有哪些节点？用 `ros2 node list` 查看；对某个节点感兴趣，加 `info` 子命令查看详细信息：

```bash
$ ros2 node list
$ ros2 node info /turtlesim
```

![[attachments/r14_image-20220524082550228.jpg]]

![[attachments/r14_image-20220524082558457.jpg]]

### 查看话题：ros2 topic

查看当前系统的话题用 `ros2 topic list`；想看某个话题中的消息数据，加 `echo` 子命令：

```bash
$ ros2 topic list
$ ros2 topic echo /turtle1/pose
```

![[attachments/r14_image-20220524082639408.jpg]]

![[attachments/r14_image-20220524082647033.jpg]]

### 发布话题消息：ros2 topic pub

想要控制海龟动起来，可以直接通过命令行发布话题指令：

```bash
$ ros2 topic pub --rate 1 /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"
```

`--rate 1` 表示每秒发布一次，向 `/turtle1/cmd_vel` 话题发布 `Twist` 类型消息，设定线速度 2.0、角速度 1.8，海龟就开始画圈。

![[attachments/r14_image-20220524082721919.jpg]]

![[attachments/r14_image-20220524082711671.jpg]]

### 发送服务请求：ros2 service

一只海龟太孤单，仿真器提供"产生海龟"的服务，用 `ros2 service call` 调用：

```bash
$ ros2 service call /spawn turtlesim/srv/Spawn "{x: 2, y: 2, theta: 0.2, name: ''}"
```

![[attachments/r14_image-20220524082737479.jpg]]

![[attachments/r14_image-20220524082745776.jpg]]

### 发送动作目标：ros2 action

想让海龟完成一个具体动作（如转到指定角度），仿真器提供的 action 可以帮忙，用 `ros2 action send_goal` 发送动作目标：

```bash
$ ros2 action send_goal /turtle1/rotate_absolute turtlesim/action/RotateAbsolute "theta: 3"
```

![[attachments/r14_image-20220524082809463.jpg]]

### 录制控制命令：ros2 bag

系统运行中的数据很多，想把某段数据录制下来、回到实验室再复现？`rosbag` 轻松实现数据的录制与播放：

```bash
$ ros2 bag record /turtle1/cmd_vel
$ ros2 bag play rosbag2_2022_04_11-17_35_40/rosbag2_2022_04_11-17_35_40_0.db3
```

![[attachments/r14_image-20220524082824817.jpg]]

## 小结

| 命令 | 作用 |
|------|------|
| `ros2 run <pkg> <exec>` | 运行功能包中的节点 |
| `ros2 node list / info` | 查看节点列表 / 节点详细信息 |
| `ros2 topic list / echo / pub` | 查看话题 / 监听话题数据 / 发布话题消息 |
| `ros2 service call` | 调用服务 |
| `ros2 action send_goal` | 发送动作目标 |
| `ros2 bag record / play` | 录制 / 回放数据 |

每个命令的子命令还有很多，可以自己尝试探索。这套命令是后续开发调试最趁手的工具。

## 相关页面

- [[concepts/ROS2/Ch02 ROS2 与 ROS1 的对比]] — 为什么 ROS2 把所有命令集成到 `ros2` 主命令下
- [[concepts/ROS2/Ch03 ROS2 环境安装与配置]] — 先装好环境，才有命令行可用
- [[concepts/ROS2/Ch05 核心概念：工作空间与功能包]] — 命令行中的 node/topic/service/action 概念，将在工作空间中落地开发

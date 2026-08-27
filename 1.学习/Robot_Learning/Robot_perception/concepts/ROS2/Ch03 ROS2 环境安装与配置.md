---
tags: [ros2, ubuntu, linux, installation, vscode, git, robot-learning]
created: 2026-08-27
updated: 2026-08-27
status: budding
domain: robot-learning
---

# Ch03 ROS2 环境安装与配置

> 来源：《ROS2 入门 21 讲》古月居 · 1.3 ROS2 安装方法 + 1.5 ROS2 开发环境配置

**安装 ROS2 之前，先要理解它的底层依赖——Linux。** ROS2 虽支持 Windows、MacOS，但对 Linux 支持最好，本教程以 **Ubuntu 22.04 + ROS2 Humble** 为例讲解。本章带你走完"装系统 → 装 ROS2 → 配开发环境"的完整链路。

## Linux 系统简介

1991 年，芬兰大学生 **林纳斯（Linus）** 在熟悉操作系统原理和 Unix 系统后，参考已有通用标准重新设计了一套操作系统内核，实现多用户、多任务管理，兼容 Unix 原有应用程序。他把它分享到互联网上，用自己的名字命名——**Linux**。

![[attachments/r13_image-20220523190154911.jpg]]

与 Windows 收费或 MacOS 硬件绑定不同，**Linux 是免费且开放源代码的操作系统**，任何人可以使用或提交反馈。现在每年对 Linux 提交的代码量已成为衡量大公司技术实力的重要指标之一。

这里引入一个重要概念——**发行版（Distribution）**。准确来讲，我们提到的"Linux"应该是**操作系统内核**，并没有可视化界面；发行版就是给内核加上华丽的外衣，把操作界面和各种应用软件打包成安装镜像。

![[attachments/r13_image-20220523190248546.jpg]]

常用发行版各有适用场景：RedHat 适合商业应用、CentOS 适合服务器、Ubuntu/Fedora 适合个人使用。核心都是 Linux，操作方法基本相同。

## Ubuntu 系统简介

本教程使用的 Linux 发行版是 **Ubuntu**。它诞生于 2004 年 10 月，**每六个月发布一个新版本**，用户可以一直免费升级使用。

![[attachments/r13_image-20220523190511789.jpg]]

Ubuntu 版本变动快，如何选择版本很重要。版本编号有规律：如 Ubuntu 22.04，**22 代表 2022 年，04 表示 4 月发布**（还可能出现 10，即 10 月发布）。

![[attachments/r13_image-20220526082937949.jpg]]

为了稳定，Ubuntu 每隔两年的 4 月发布一个**长期支持版**，后缀加 **LTS**，保证 5 年持续维护更新（如 20.04 LTS、22.04 LTS）。其他普通版只维护 18 个月。**选择时优先考虑 LTS 版本**。

## Ubuntu 虚拟机安装

如果第一次接触 Linux，建议在 Windows 上通过**虚拟机**（如 VMware）安装；熟悉后再考虑硬盘安装，发挥硬件最大性能。

![[attachments/r13_image-20220526082404826.jpg]]

安装步骤概要（详见 VMware 官方流程）：

1. **下载系统镜像**：https://ubuntu.com/download/desktop
2. **在虚拟机中创建系统**：新建虚拟机 → 选择 Ubuntu 镜像
3. **设置虚拟机硬盘大小**（建议 60GB 以上）
4. **设置 Ubuntu 镜像路径**
5. **启动虚拟机**
6. **设置用户名和密码**
7. **等待系统安装**
8. **完成安装**

## ROS2 系统安装

Ubuntu 装好后，开始安装 ROS2。四步走：

### 1. 设置编码

```bash
$ sudo apt update && sudo apt install locales
$ sudo locale-gen en_US en_US.UTF-8
$ sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
$ export LANG=en_US.UTF-8
```

### 2. 添加软件源

```bash
$ sudo apt update && sudo apt install curl gnupg lsb-release
$ sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(source /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

> ⚠️ 如遇报错 **"Failed to connect to raw.githubusercontent.com"**（国内网络常见），可参考 https://www.guyuehome.com/37844 的镜像解决方案。

### 3. 安装 ROS2

```bash
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install ros-humble-desktop
```

`ros-humble-desktop` 是 ROS2 Humble 的桌面版安装包，包含完整的开发工具和 GUI 工具。

### 4. 设置环境变量

```bash
$ source /opt/ros/humble/setup.bash
$ echo " source /opt/ros/humble/setup.bash" >> ~/.bashrc
```

第一条命令仅在当前终端生效，第二条写入 `~/.bashrc` 使**每个终端**打开时都自动加载 ROS2 环境。

## ROS2 示例测试

### 示例一：命令行验证 DDS 通信

启动第一个终端，运行发布者节点：

```bash
$ ros2 run demo_nodes_cpp talker
```

![[attachments/r13_image-20220523191236749.jpg]]

启动第二个终端，运行订阅者节点：

```bash
$ ros2 run demo_nodes_py listener
```

![[attachments/r13_image-20220523191243490.jpg]]

如果 "Hello World" 字符串在两个终端中正常传输，说明底层通信系统没有问题。

### 示例二：小海龟仿真

ROS 中的经典示例——小海龟仿真器：

```bash
$ ros2 run turtlesim turtlesim_node      # 启动蓝色背景海龟仿真器
$ ros2 run turtlesim turtle_teleop_key   # 启动键盘控制节点
```

在键盘控制终端中按"上下左右"按键，就可以控制小海龟运动。

![[attachments/r13_image-20220523191347938.jpg]]

小海龟中蕴藏的 ROS 奥义（节点、话题、服务、动作），会在后续章节持续探索。

## 开发环境配置

ROS 开发离不开代码编写。本教程推荐两款重要开发工具——**Git** 和 **VSCode**。

### Git：版本管理软件

Git 是一个**版本管理软件**，也因 Linux 而生。Linux 成千上万人贡献代码，需要高效管理所有提交，让开发者看到每次变更、自动判断冲突、在多个版本间切换。Linux 之父 Linus 因此设计了 Git，之后被广泛应用——开源网站 GitHub 和国内码云 Gitee 都在使用。

```bash
$ sudo apt install git
```

下载《ROS2 入门 21 讲》课程源码：

```bash
$ git clone https://gitee.com/guyuehome/ros2_21_tutorials.git
```

![[attachments/r15_image-20220524083019307.png]]

### VSCode：轻量强大的编辑器

Visual Studio Code，简称 **VSCode**，是微软 2015 年推出的轻量但功能强大的源代码编辑器，支持 Windows、Linux、macOS，扩展组件非常丰富。

下载：https://code.visualstudio.com/Download

### VSCode 插件配置

为便于 ROS2 开发与调试，推荐安装以下插件：

![[attachments/r15_image-20220524083416831.png]]

| 插件 | 用途 |
|------|------|
| **中文语言包** | 中文界面 |
| **Python 插件** | Python 语法高亮、智能提示 |
| **C++ 插件** | C++ 语法高亮、调试 |
| **CMake 插件** | CMake 构建配置支持 |
| **vscode-icons** | 文件图标美化 |
| **ROS 插件** | ROS2 开发专用（编译、运行、调试） |
| **Msg Language Support** | ROS 消息/接口文件语法支持 |
| **Visual Studio IntelliCode** | AI 智能补全 |
| **URDF** | 机器人模型文件高亮 |
| **Markdown All in One** | Markdown 编辑增强 |

VSCode 支持插件众多，以上只作为个人推荐，可以按需配置出自己喜欢的开发环境。

## 相关页面

- [[concepts/ROS2/Ch01 认识 ROS 与 ROS2]] — ROS 的历史与核心目标，理解为什么要装它
- [[concepts/ROS2/Ch02 ROS2 与 ROS1 的对比]] — 为什么选择 ROS2 Humble（ROS2 第一个 LTS）
- [[concepts/ROS2/Ch04 ROS2 命令行操作]] — 装好后的第一个调试工具：命令行
- [[concepts/ROS2/Ch05 核心概念：工作空间与功能包]] — 装好环境后，从工作空间开始搭建工程

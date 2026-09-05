# YOPO Training

YOPO 学习与训练过程记录。

## Phase 1 - Ubuntu 与开发环境基础配置

### 1. Ubuntu 环境

当前使用环境：

- Ubuntu：24.04.4 LTS
- NVIDIA GPU：已识别
- NVIDIA Driver：595.84
- `nvidia-smi`：运行正常

详细环境检测结果：

- `environment/ubuntu-version.txt`
- `environment/nvidia-smi.txt`

> 第一阶段部分 Linux 与 Python 基础练习最初在另一台 Ubuntu 电脑上完成；
> 当前 NVIDIA GPU 环境检查在带 NVIDIA 独立显卡的电脑上补充完成。

### 2. Linux 基础命令练习

Linux 基础操作练习保存在：

`linux_practice/`

目录包括：

- `code/`：Python 测试代码
- `data/`：练习数据
- `results/`：运行结果
- `README.md`：Linux 命令练习记录

练习内容包括：

- `pwd`
- `tree`
- `cd`
- `nano`
- `python3`
- `touch`
- `echo`
- `cat`
- `cp`
- `mv`
- `ls`

### 3. NVIDIA GPU 检查

执行命令：

```bash
nvidia-smi
```


## 第二阶段：YOPO 环境搭建与系统运行

第二阶段完成 YOPO 环境配置、Controller/Simulator 编译、预训练模型运行、RViz 目标点设置以及至少三次自主避障实验。

详细实验记录与提交材料见：[phase2/README.md](phase2/README.md)

## Phase 3：YOPO 论文阅读与代码理解

第三阶段已完成 YOPO 整体结构、网络推理流程、Motion Primitive、轨迹生成以及 YOPO、Controller 和 Simulator 数据传递关系的代码阅读与整理。

### 已完成内容

- 梳理 YOPO 系统整体结构
- 分析 YOPO 网络输入与输出
- 确认深度图网络输入尺寸为 `160 × 96`
- 理解状态信息与深度图进入网络的基本流程
- 阅读 Motion Primitive 的生成方式
- 理解候选轨迹的选择过程
- 阅读五次多项式轨迹 `Poly5Solver` 的生成方式
- 梳理 YOPO → `PositionCommand` → SO3 Controller → `SO3Command` → Simulator 的控制数据链路

### 第三阶段成果

第三阶段材料保存在：

`phase3/`

主要包括：

- `phase3/diagrams/`：系统结构图、数据流图、网络输入输出图等
- `phase3/notes/`：代码阅读及数据流笔记
- `phase3/report/phase3_code_reading_report.md`：第三阶段代码阅读报告
- `phase3/tables/phase3_core_files.md`：核心代码文件说明表
- `phase3/tables/phase3_ros_topics.md`：ROS Topic 整理

第三阶段主要提交 Commit：

`4fbd705 - Complete phase 3 YOPO code reading`

### 当前仍需进一步理解

- `score_pred` 的具体评价指标及训练标签生成方式
- `endstate_pred` 各维度与实际终端状态的具体对应关系
- YOPO、Controller、Simulator 实际运行时的 ROS Topic 实时数据流
- Motion Primitive 参数对轨迹和避障效果的影响

### 下一步计划

进入 Phase 4 基础运行与对比实验，在不同场景和不同飞行速度下运行 YOPO，并记录成功率、碰撞情况、飞行时间和典型轨迹等实验结果。


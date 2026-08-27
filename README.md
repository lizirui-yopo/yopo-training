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


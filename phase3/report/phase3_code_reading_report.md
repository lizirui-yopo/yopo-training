# 第三阶段：YOPO代码阅读报告

姓名：李紫锐

日期：2026-08-31

## 1. YOPO整体思路
YOPO主要解决复杂环境下无人机自主导航规划问题。

传统无人机规划方法通常包含环境感知、路径搜索和轨迹优化等多个步骤，流程复杂，计算开销较大。

YOPO通过学习型规划方法，根据环境观测信息和无人机状态快速生成可执行轨迹，提高无人机自主导航效率。


## 2. YOPO系统主要组成

YOPO系统主要包括：

- 输入数据处理模块
- 神经网络规划模块
- 轨迹生成模块
- 控制执行模块


其中：

输入模块负责获取无人机状态和环境信息；

YOPO网络根据输入信息预测未来轨迹；

控制模块负责执行规划结果。


## 3. 代码阅读记录
### 3.1 ROS 输入

在 test_yopo_ros.py 中，YOPO 节点通过 ROS Subscriber 接收无人机运行信息，主要包括：

- Odometry：无人机位置、姿态和速度等状态信息；
- Depth Image：深度图，用于获取周围障碍物信息；
- Goal：无人机目标位置。

### 3.2 状态与深度图处理

process_odom() 对无人机状态和目标信息进行处理，将速度、加速度以及目标方向等信息转换并组合为网络所需的状态输入。

callback_depth() 对深度图进行预处理，包括数据格式转换、尺寸调整、归一化以及异常深度值处理。

### 3.3 YOPO 网络推理

处理后的深度图和无人机状态共同作为 YOPO 网络输入。

核心推理代码为：

endstate_pred, score_pred = self.policy(depth_input, obs_input)

网络输出预测终端状态 endstate_pred 和对应评分 score_pred。

### 3.4 预测结果处理

网络预测结果经过坐标系转换后用于生成和选择无人机轨迹，并通过 visualize_trajectory() 对预测轨迹进行可视化。

最终轨迹结果进一步用于后续控制消息处理。

## 4. YOPO 网络输入与输出

YOPO 网络的主要输入包括深度图和无人机状态信息。

深度图在配置文件 traj_opt.yaml 中设置为宽 160、高 96，用于提供周围障碍物的空间信息。

状态输入由无人机自身运动状态以及目标相关信息组成。根据 test_yopo_ros.py 中的 process_odom()，程序会处理速度、加速度和目标方向等信息，并组合为网络状态输入。

YOPO 网络的核心推理形式为：

endstate_pred, score_pred = self.policy(depth_input, obs_input)

其中：

- endstate_pred 表示网络预测的候选终端状态或轨迹相关结果；
- score_pred 表示各候选结果对应的评分。

这些预测结果经过后续坐标转换和轨迹处理后，用于选择并生成无人机最终执行的轨迹。

## 5. YOPO 与 Simulator、Controller 的关系

通过阅读 YOPO、Simulator 和 Controller 中的相关代码，我对整个系统的关系有了初步理解。

Simulator 主要负责仿真环境以及传感器相关数据，为系统提供深度图和无人机状态等信息。

YOPO 规划模块接收深度图、无人机状态和目标点等信息，利用网络进行预测，并生成后续轨迹或控制相关信息。

Controller 控制模块接收规划结果，并进一步生成无人机控制命令。

因此，目前理解的整体流程为：

Simulator → YOPO → Controller → 无人机执行 → 状态反馈

具体的数据传递和各 ROS Topic 的作用还需要进一步学习。

### 5.1 ROS 数据传递关系

进一步阅读源码后，目前已经能够明确 YOPO、Simulator 和 Controller 之间的主要 ROS 数据传递关系。

Simulator 向 YOPO 提供无人机状态和深度图信息：

- `/sim/odom`：里程计话题，YOPO 使用 `Odometry` 消息获取无人机的位置、姿态和速度等状态信息。
- `/depth_image`：深度图话题，YOPO 使用 `Image` 消息获取环境深度信息。

YOPO 在接收到状态和深度图后进行网络推理和轨迹生成，并通过：

- `/so3_control/pos_cmd`

发布 `PositionCommand`，其中包含期望位置、速度、加速度以及 yaw 等轨迹控制信息。

Controller 接收 `PositionCommand` 后进一步计算无人机控制量，并生成 `SO3Command`。Simulator 接收控制命令后更新无人机动力学状态，再重新产生 odometry 和传感器数据，从而形成闭环。

因此目前理解的数据流可以表示为：

Simulator → `/sim/odom` + `/depth_image` → YOPO → `PositionCommand` → `/so3_control/pos_cmd` → Controller → `SO3Command` → Simulator → 状态反馈

### 5.2 深度图尺寸关系

YOPO 网络配置中的深度图输入尺寸为：

- `image_width = 160`
- `image_height = 96`

在 `callback_depth()` 中，如果收到的深度图尺寸与网络要求不一致，会通过 `cv2.resize()` 调整到网络输入尺寸。因此仿真器提供的深度图可以在进入 YOPO 网络之前经过尺寸转换，最终形成网络所需的 `160 × 96` 输入。


### 5.3 候选轨迹选择与五次多项式轨迹生成

YOPO 网络推理后得到两个主要结果：`endstate_pred` 和 `score_pred`。其中 `endstate_pred` 表示不同轨迹 primitive 对应的预测终端状态，`score_pred` 表示这些候选轨迹对应的评分。

在 `test_yopo_ros.py` 的 `process_output()` 中，程序首先对网络输出进行重新排列，然后通过：

`action_id = np.argmin(score_pred)`

选择评分最低的候选轨迹。随后根据 `action_id` 得到对应的 lattice primitive，并通过 `pred_to_endstate_cpu()` 将网络预测结果转换为实际的终端状态。

轨迹 primitive 的数量由水平、垂直和径向方向的离散数量共同决定。当前配置中可以看到：

- `horizon_num = 5`
- `vertical_num = 3`
- `radio_num = 1`

因此当前共有 5 × 3 × 1 = 15 个候选 primitive。

网络本身并不是直接输出完整的连续飞行轨迹，而是预测候选 primitive 对应的终端状态。选出最终候选后，程序分别在 x、y、z 三个方向使用 `Poly5Solver` 构造五次多项式轨迹。

`Poly5Solver` 使用起点和终点的位置、速度、加速度以及轨迹持续时间作为边界条件：

`pos0, vel0, acc0, pos1, vel1, acc1, Tf`

并计算五次多项式的系数。轨迹可以表示为：

`p(t) = A0 + A1*t + A2*t^2 + A3*t^3 + A4*t^4 + A5*t^5`

代码同时提供 `get_position()`、`get_velocity()`、`get_acceleration()`、`get_jerk()` 和 `get_snap()` 等函数，因此可以从同一个五次多项式轨迹得到位置、速度、加速度以及更高阶运动信息。

因此，目前对 YOPO 输出到轨迹生成过程的理解可以表示为：

YOPO 网络 → `endstate_pred` + `score_pred` → 选择最低评分候选 → 坐标与尺度转换 → 得到目标终端状态 → x/y/z 三轴 `Poly5Solver` → 连续五次多项式轨迹 → 后续控制执行


### 5.4 YOPO 与 Controller 的控制接口

进一步阅读 Controller 源码后，可以确认 YOPO 生成的轨迹信息通过 `PositionCommand` 传递给 SO3 Controller。

在 `so3_control_nodelet.cpp` 中，Controller 通过 `position_cmd_callback()` 接收 `PositionCommand`。回调函数读取的主要期望状态包括：

- `cmd->position`：期望位置；
- `cmd->velocity`：期望速度；
- `cmd->acceleration`：期望加速度；
- `cmd->yaw`：期望偏航角；
- `cmd->yaw_dot`：期望偏航角速度。

这些信息分别保存为 `des_pos_`、`des_vel_`、`des_acc_`、`des_yaw_` 和 `des_yaw_dot_`。此外，Controller 还可以从消息中的 `kx` 和 `kv` 获取位置和速度相关的控制增益。

Controller 同时通过 `odom_callback()` 从 odometry 中获得无人机当前的位置和速度，并通过 `controller_.setPosition()` 和 `controller_.setVelocity()` 更新实际状态。

在收到新的 `PositionCommand` 后，程序调用：

`publishSO3Command();`

进一步计算并发布 `SO3Command`。Simulator 接收该控制命令后更新无人机动力学状态，并重新产生 odometry 和传感器数据，从而形成闭环。

因此目前确认的规划与控制数据链路可以表示为：

Simulator → odometry / depth image → YOPO → 候选轨迹预测与选择 → 五次多项式轨迹 → `PositionCommand` → SO3 Controller → `SO3Command` → Simulator → 状态反馈


## 6. 当前理解的问题

通过本阶段的代码阅读，目前已经初步理解 YOPO 网络输入输出、候选轨迹选择、五次多项式轨迹生成以及 YOPO 与 Controller 之间的基本数据传递关系。

目前仍需要进一步理解的问题包括：

1. `score_pred` 的具体评价指标以及网络训练过程中评分标签的生成方式。
2. `endstate_pred` 中各维度与实际无人机终端状态之间更具体的对应关系。
3. YOPO、Controller 和 Simulator 在实际 ROS 运行过程中各节点的启动顺序及 Topic 的实时数据流。
4. 候选 primitive 的参数变化对最终飞行轨迹和避障效果的具体影响。

这些问题将在后续实际运行仿真、观察 ROS Topic 和网络输出时进一步分析。

# YOPO 与 Controller 数据传递记录

## 1. YOPO 发送端

YOPO 在 test_yopo_ros.py 中生成规划后的控制/轨迹信息，并使用 PositionCommand 消息进行封装。

主要包括：

- position：期望位置
- velocity：期望速度
- acceleration：期望加速度
- yaw：期望偏航角
- yaw_dot：期望偏航角速度

## 2. ROS Topic

YOPO 通过 ctrl_pub 发布 PositionCommand。

当前配置中对应的 Topic 为：

/so3_control/pos_cmd

因此目前确认的数据链为：

YOPO
→ PositionCommand
→ /so3_control/pos_cmd
→ Controller

## 3. Controller 接收端

Controller 中的 position_cmd_callback() 接收 PositionCommand。

接收后主要保存为：

- des_pos_：期望位置
- des_vel_：期望速度
- des_acc_：期望加速度
- des_yaw_：期望偏航角
- des_yaw_dot_：期望偏航角速度

随后 Controller 调用 publishSO3Command() 进行后续控制处理。

## 4. Controller 输出

Controller 将处理后的控制结果封装为：

quadrotor_msgs::SO3Command

并通过：

so3_command_pub_.publish(so3_command)

进行发布。

代码中可以确认其发布 Topic 为：

so3_cmd

因此目前确认：

Controller
→ SO3Command
→ so3_cmd

## 5. 当前完整理解

目前通过代码阅读确认的数据传递过程为：

YOPO
→ PositionCommand
→ /so3_control/pos_cmd
→ Controller
→ position_cmd_callback()
→ publishSO3Command()
→ SO3Command
→ so3_cmd

这说明 YOPO 主要负责生成规划得到的期望运动状态，而 Controller 接收这些期望状态并进一步生成更底层的 SO3 控制命令。

## 6. 当前仍未完全理解的问题

目前已经确认 Controller 会通过 so3_cmd 发布 SO3Command。

但是在当前搜索到的源码中，还没有找到直接订阅 so3_cmd 的模块，因此 Controller 输出到仿真动力学/无人机执行之间的最后一段数据链仍需进一步学习。

当前暂不假定 Simulator 直接订阅 so3_cmd，后续结合 launch 文件、ROS 节点连接关系或实际运行时 Topic 关系继续确认。

## 7. SO3Command 到四旋翼模拟器

通过查看 so3_quadrotor_simulator 中的 quadrotor_simulator_so3.cpp，可以确认模拟器通过 cmd_callback() 接收 SO3Command。

在回调函数中，主要读取了：

- force：三轴控制力
- orientation：姿态四元数
- kR：姿态控制相关参数
- kOm：角速度控制相关参数
- 其他辅助控制参数

结合 launch 文件中的：

<remap from="~cmd" to="so3_cmd"/>

可以确认 Controller 发布的 so3_cmd 会连接到四旋翼模拟器内部的 cmd 订阅接口。

因此目前理解的控制链为：

YOPO
→ PositionCommand
→ /so3_control/pos_cmd
→ Controller
→ SO3Command
→ so3_cmd
→ Quadrotor Simulator
→ cmd_callback()
→ 四旋翼动力学状态更新

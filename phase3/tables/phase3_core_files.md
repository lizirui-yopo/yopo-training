# 第三阶段核心代码文件说明表

| 文件 | 所属模块 | 主要作用 | 当前理解 |
|---|---|---|---|
| `YOPO/test_yopo_ros.py` | YOPO 在线运行 | 订阅里程计、深度图和目标点，进行网络推理、轨迹生成，并发布 PositionCommand | 已阅读主要数据流 |
| `YOPO/train_yopo.py` | YOPO 训练 | YOPO 网络训练入口 | 尚未深入训练流程 |
| `YOPO/policy/yopo_network.py` | YOPO 网络 | 定义 YOPO 神经网络并完成前向推理 | 已确认输入为 depth_input 和 obs_input |
| `YOPO/policy/primitive.py` | 轨迹/动作原语 | 定义规划过程中使用的轨迹原语 | 初步了解 |
| `YOPO/policy/poly_solver.py` | 轨迹生成 | 根据预测结果生成 x、y、z 方向的多项式轨迹 | 已在 test_yopo_ros.py 中看到调用 |
| `YOPO/policy/state_transform.py` | 状态处理 | 对无人机状态和网络输入进行变换及归一化 | 初步了解 |
| `YOPO/policy/yopo_dataset.py` | 数据集 | 训练阶段的数据读取与预处理 | 尚未深入 |
| `YOPO/policy/yopo_trainer.py` | 网络训练 | 组织 YOPO 网络训练过程 | 尚未深入 |
| `Controller/src/so3_control/src/so3_control_nodelet.cpp` | Controller | 接收 PositionCommand，并保存期望位置、速度、加速度和 yaw 等信息 | 已阅读主要回调函数 |
| `Controller/src/so3_control/src/NetworkControl.cpp` | Controller | 根据期望状态生成并发布 SO3Command | 初步阅读 |
| `Controller/src/so3_quadrotor_simulator/src/quadrotor_simulator_so3.cpp` | 四旋翼仿真 | 接收 SO3Command，并将控制量送入四旋翼动力学仿真 | 已阅读 cmd_callback() |
| `Simulator/src/src/test_simulator_cuda.cpp` | Simulator | 创建深度图 Publisher，并发布传感器深度图 | 已确认 /depth_image 发布链路 |
| `Simulator/src/sim_odom.py` | Simulator | 发布仿真里程计信息 | 已确认 /sim/odom 发布链路 |

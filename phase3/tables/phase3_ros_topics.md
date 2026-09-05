# YOPO ROS Topic 记录

通过阅读 `test_yopo_ros.py`，对 YOPO 运行过程中使用的主要 ROS Topic 进行了整理。

| Topic | YOPO 中的作用 | 数据内容 |
|---|---|---|
| `/sim/odom` | 订阅 | 无人机里程计和运动状态 |
| `/depth_image` | 订阅 | 深度图像 |
| `/move_base_simple/goal` | 订阅 | 无人机目标点 |
| `/so3_control/pos_cmd` | 发布 | 规划后的控制/轨迹相关信息 |

目前理解为：YOPO 通过里程计、深度图和目标点获得规划所需信息，经过网络预测和轨迹处理后，将结果通过控制相关 Topic 发送给后续控制模块。

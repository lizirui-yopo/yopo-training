# Simulator 与 YOPO 数据传递记录

## 1. Simulator 提供的数据

通过查看 Simulator 和 YOPO 的代码，目前确认 Simulator 会向 YOPO 提供里程计信息和深度图。

## 2. 里程计信息

Simulator 中 sim_odom.py 发布：

/sim/odom

消息类型为 Odometry。

YOPO 订阅该 Topic，用于获得无人机的位置、速度等运动状态信息。

因此目前理解为：

Simulator
→ /sim/odom
→ YOPO

## 3. 深度图

Simulator 的配置文件中：

depth_topic: "/depth_image"

在 test_simulator_cuda.cpp 中可以看到 image_pub_ 根据 depth_topic 创建，并通过：

image_pub_.publish(ros_image)

发布生成的深度图。

YOPO 订阅 /depth_image 获得深度图信息。

因此目前理解为：

Simulator
→ 生成深度图
→ /depth_image
→ YOPO

## 4. 当前理解

Simulator 主要为 YOPO 提供仿真环境中的传感器和无人机状态信息。

目前确认的两条主要输入链路为：

Simulator → /sim/odom → YOPO
Simulator → /depth_image → YOPO

YOPO 再结合目标点等信息进行网络预测和轨迹规划。

## 5. 深度图尺寸转换

Simulator 配置中的深度图尺寸为：

- image_width: 160
- image_height: 90

因此 Simulator 生成并通过 /depth_image 发布的原始深度图尺寸为 160×90。

YOPO 在 test_yopo_ros.py 中读取自身配置：

- self.width = cfg['image_width']
- self.height = cfg['image_height']

在接收到深度图后，会检查输入尺寸：

if depth.shape[0] != self.height or depth.shape[1] != self.width:
    depth = cv2.resize(depth, (self.width, self.height), interpolation=cv2.INTER_NEAREST)

结合 YOPO 配置中的 image_width=160、image_height=96，可以确认：

Simulator 160×90
→ /depth_image
→ YOPO 接收
→ cv2.resize()
→ 160×96
→ YOPO 网络输入

因此 Simulator 的深度图输出尺寸与 YOPO 网络输入尺寸不同，尺寸适配是在 YOPO 的输入预处理阶段完成的。

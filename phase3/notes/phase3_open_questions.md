# 第三阶段当前仍未理解的问题

## 1. YOPO 网络内部结构如何产生预测结果？

目前已经知道 YOPO 网络接收 depth_input 和 obs_input，并输出 endstate_pred 和 score_pred。

但是对于 backbone、head 等网络模块内部如何提取特征，以及这些特征如何最终得到候选终端状态和评分，目前还没有完全理解。

后续需要继续阅读：

- policy/yopo_network.py
- policy/models/backbone.py
- policy/models/head.py

## 2. YOPO 的训练流程具体是怎样的？

目前已经找到 train_yopo.py、yopo_trainer.py、yopo_dataset.py 以及 loss 相关文件。

但是训练数据如何组织、不同 loss 如何参与训练、网络参数如何更新，目前还没有系统阅读。

后续需要继续学习训练数据、损失函数和训练循环之间的关系。

## 3. 轨迹原语和五次多项式轨迹生成的数学原理是什么？

目前已经看到 primitive.py 和 poly_solver.py，并在 test_yopo_ros.py 中看到 Poly5Solver 用于生成 x、y、z 方向的轨迹。

但是轨迹原语如何定义候选动作，以及五次多项式如何根据起点、终点、速度和加速度约束生成最终轨迹，目前还没有完全理解。

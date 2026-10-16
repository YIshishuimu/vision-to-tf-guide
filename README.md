# 新生视觉入门：从摄像头到 ROS 2 TF

参考项目：[ARES-Algorithm/kfs_alignment](https://github.com/ARES-Algorithm/kfs_alignment)。本仓库概述一项循序渐进的视觉学习任务：从相机图像中识别已知目标，估计三维位姿，并将目标坐标系发布到 ROS 2 TF。

> 参考项目是私有仓库，访问源码和模型需要单独获得该仓库的读取权限。

## 学习目标

完成后，能够解释并实现这条基本工作流：

```mermaid
flowchart LR
    A[选择摄像头并获取图像] --> B[标定相机]
    B --> C[识别目标并定位特征点]
    C --> D[结合目标尺寸求解三维位姿]
    D --> E[转换坐标系并发布 ROS 位姿与 TF]
    E --> F[在 RViz 中检查结果]
```

学习重点是理解每一步的输入、输出和误差来源，并能把结果连成一条可解释的 ROS 视觉链路。目标范围到 TF 发布与验证，不涉及底盘运动控制、PID、抬升机构或 IMU 融合。

## 大致工作流

1. **选择图像来源。** 根据手头设备选择 USB 相机、ROS 图像话题或厂商相机 SDK。重点是稳定获得图像，并确认分辨率、帧率和图像编码。
2. **标定相机。** 采集覆盖不同位置和角度的标定图像，求出相机内参及畸变参数。标定结果必须对应实际送入算法的图像尺寸和处理方式。
3. **识别目标与特征点。** 选择适合目标的检测方法，输出目标区域及有固定顺序的图像特征点。仓库提供的 YOLO 权重可作为参考；换相机或换目标后，需要先用实际图像评估其适用性。
4. **求解三维位姿。** 将图像特征点与目标的已知几何尺寸、相机标定数据对应起来，通过 PnP 等方法求出位置和姿态。用重投影和实际测量检查结果，并关注点序、尺度和误差。
5. **发布并检查 TF。** 明确父子坐标系、坐标轴方向、单位和时间戳，再发布目标位姿和动态 TF。通过 RViz 与 TF 数据检查坐标方向及更新是否合理。

每个阶段都应留下可检查的中间结果，例如原始图像、标定结果、带编号的特征点图、重投影结果和 TF 可视化。具体实现可以根据所选相机与目标调整。

## 需要掌握的概念

- 相机内参、畸变参数，以及图像分辨率变化对标定的影响。
- 图像坐标、相机坐标、目标坐标之间的对应关系。
- 特征点顺序、目标尺寸、PnP 解和重投影误差。
- ROS 位姿消息与 TF 中父子坐标系、单位、时间戳的含义。
- 相机光学坐标常用的右、下、前轴向，与机器人常用坐标约定之间的转换。

## 参考项目阅读顺序

获得原仓库访问权限后，可以按数据流阅读：

1. [`yolo_detector.py`](https://github.com/ARES-Algorithm/kfs_alignment/blob/main/src/kfs_alignment/kfs_alignment/yolo_detector.py)：目标与特征点检测。
2. [`geometry_utils.py`](https://github.com/ARES-Algorithm/kfs_alignment/blob/main/src/kfs_alignment/kfs_alignment/geometry_utils.py)：图像区域变换及点坐标映射。
3. [`pnp_solver.py`](https://github.com/ARES-Algorithm/kfs_alignment/blob/main/src/kfs_alignment/kfs_alignment/pnp_solver.py)：几何关系与位姿求解。
4. [`cube_pose_node.py`](https://github.com/ARES-Algorithm/kfs_alignment/blob/main/src/kfs_alignment/kfs_alignment/cube_pose_node.py)：图像输入、坐标转换、位姿和 TF 发布。
5. [`cube_pose_estimator.yaml`](https://github.com/ARES-Algorithm/kfs_alignment/blob/main/src/kfs_alignment/config/cube_pose_estimator.yaml)：相机、模型与坐标系参数。

## 延伸资料

- [ROS 2 Jazzy 教程](https://docs.ros.org/en/jazzy/Tutorials.html)
- [tf2 入门](https://docs.ros.org/en/jazzy/Tutorials/Intermediate/Tf2/Introduction-To-Tf2.html)
- [Python 动态 TF 广播器](https://docs.ros.org/en/jazzy/Tutorials/Intermediate/Tf2/Writing-A-Tf2-Broadcaster-Py.html)
- [OpenCV 相机标定](https://docs.opencv.org/4.x/dc/dbb/tutorial_py_calibration.html)
- [OpenCV PnP](https://docs.opencv.org/4.x/d5/d1f/calib3d_solvePnP.html)

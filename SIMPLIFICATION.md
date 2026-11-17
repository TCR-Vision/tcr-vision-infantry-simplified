# MPC 精简说明

修改仅在 `tcr-vision-infantry简化` 副本中进行，原工程不变。

保留三个主程序：

```text
src/auto_aim_debug_mpc.cpp    自瞄 MPC 调试
src/standard_mpc.cpp          自瞄 MPC 与大小符模式切换
src/auto_buff_debug_mpc.cpp   打符 MPC 调试（当前固定 SmallTarget）
```

删除 `src/` 中其他十个旧入口；删除 `tasks/auto_aim/` 中的
`aimer.hpp/cpp`、`shooter.hpp/cpp`、`multithread/commandgener.hpp/cpp`。
根 CMake 移除旧入口目标，auto_aim 库移除旧决策源文件。

检测、Solver、Tracker、Target、EKF、Planner/TinyMPC、打符算法、相机与通信协议没有更换。
保留所有 21 个测试源文件与标定、模型、YAML 配置。
`auto_aim_test` 和 `minimum_vision_system` 的旧决策调用改为 Planner；前者按录制时间和配置延迟预测目标，避免回放速度影响预测。`camera_thread_test` 与 `standard_mpc` 清理删除文件的 include。

## ROS 保留范围

保留 `io/ros2/`、全向感知模块、ROS 依赖检测和三个 ROS 收发测试。
按当前实际部署使用 MPC 入口的要求，不再保留旧 sentry 入口。
现有三个 MPC 入口原本没有调用 ROS，此次也没有新增 ROS 调用。
因此旧 sentry 入口提供的视觉目标发布、导航指定目标过滤和辅助相机搜索流程不再有独立应用入口；这些不是独立导航程序本身。
若实际导航依赖这些视觉消息，仍需接入 MPC 入口，不能只靠保留 ROS 库自动获得通信。

## 构建与运行

依赖沿用原工程，包括 OpenCV、fmt、Eigen、spdlog、yaml-cpp、nlohmann_json、OpenVINO、Ceres 和相机 SDK。
OpenVINO 的 CMake 配置沿用原路径 `/opt/intel/openvino_2024.6.0/runtime/cmake`。
ROS 测试需要已加载完整 ROS2 和外部 `sp_msgs` 环境。

```bash
cmake -S . -B build
cmake --build build -j
./build/auto_aim_debug_mpc configs/standard3.yaml
# 综合自瞄与大小符：
./build/standard_mpc configs/standard3.yaml
# 独立打符调试：
./build/auto_buff_debug_mpc configs/standard3.yaml
```

从项目根目录选择一个入口运行，避免同时争用相机和串口。
启动配置若仍调用旧 standard/sentry/uav 等程序，需要切换到现有实际部署入口；现有 autostart.sh 指向的 watchdog.sh 不在仓库中，本次没有臆改实车启动配置。

## 验证记录

- 已检查 CMake 源文件及项目 include 路径：没有删除文件的残留引用。
- 已确认删除仅发生在 src/、tasks/，所有测试文件保留。
- 两个 MPC 调试入口、Planner、打符模块和 ROS 接口与仓库原版保持一致；综合入口只清理旧 include。
- 独立编译 planner_test_offline，并用 configs/standard3.yaml 连续运行 3 秒后停止。
- 初次验证时环境缺少开发依赖，未完成全工程构建；2026-11-17 的修复与最新验证见 REVIEW_20261117.md。
- 三个 MPC 主程序、auto_aim_test、minimum_vision_system、camera_thread_test 均通过 g++ C++17 语法检查；检查使用 /tmp 中官方 JSON/OpenVINO 头文件，不修改系统依赖。
- 未进行相机、串口、大小符及 ROS/导航的实车验证。

2026-11-17：根据用户后续删除的 CAN、达妙、迈德威视及串口枚举模块完成引用修复；本轮已恢复仍被打符和全向接口使用的 io/command.hpp 数据结构，未恢复 CAN 驱动。详见 REVIEW_20261117.md。

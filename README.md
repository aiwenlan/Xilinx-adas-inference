# Xilinx ADAS Inference Demo

基于 Xilinx DPU 的 ADAS（高级驾驶辅助）相关模型推理示例：车与人检测、车道线检测、交通标志（限速牌）检测与分类。本仓库侧重推理与展示链路，**未包含完整的后处理模块**（如业务级融合、跟踪、报警策略等）。

## 功能概览

| 模块 | 目录 | 可执行文件 | 说明 |
|------|------|------------|------|
| 车人检测 | `vehicle_detection/` | `aeb_od.elf` | YOLOv5n 目标检测（`yolov5n_od`），输入分辨率 512×288 |
| 车道线检测 | `lane_detection/` | `aeb_ldw` | LaneNet 变体（`lanenet_cd`） |
| 限速牌检测+分类 | `tsr_detection_cls/` | `aeb_tsr.elf` | YOLOv5n 检测 + 分类网络（`tsr_cls`） |

推理结果以窗口实时显示；各 demo 从视频文件读入帧并送入 DPU。

## 演示

系统工程中车人检测、车道线检测、限速牌检测的功能测试示意

![功能测试](https://github.com/allrivertosea/Xilinx-adas-inference-demo/blob/main/test.gif)

## 硬件与软件环境

- **硬件**：Zynq UltraScale+ MPSoC 系列开发板；已在 **XAZU3EG** 等型号上验证。
- **推理**：Xilinx **DPU**（DNNDK / Vitis AI 运行时）。
- **推荐版本**（与工程一致）：
  - Vitis AI **1.2.82**
  - OpenCV **3.4.13**
  - CMake **3.19.3**（Linux aarch64 交叉或板端环境）

> 本工程为 **Linux 板端 / aarch64** 场景设计，需在已部署 Vitis AI 与 DPU 的环境中编译运行，**不能**在 Windows 上直接 `make` 得到可运行二进制。

## 仓库结构

```
Xilinx-adas-inference/
├── vehicle_detection/    # 车人检测（aeb_od.elf）
├── lane_detection/       # 车道线（aeb_ldw）
├── tsr_detection_cls/    # 限速牌检测+分类（aeb_tsr.elf）
└── README.md
```

各子目录内需包含 `model/`，放置对应 `.elf` 编译后的 DPU 模型文件（名称与 `Makefile` 中一致，如 `dpu_yolov5n_od.elf` 等）。模型需通过 **Vitis AI** 流程从自身或官方示例网络编译得到；若缺少模型文件，`make` 阶段拷贝步骤会失败。

## 编译

在**目标板或已配置交叉工具链的环境**中，进入对应子目录执行：

```bash
cd vehicle_detection    # 或 lane_detection、tsr_detection_cls
make
```

清理：

```bash
make clean
```

依赖：OpenCV（含 `pkg-config`）、DNNDK（`-lhineon -ln2cube` 等）、`glog` 等，与板端 Vitis AI 安装一致。`lane_detection` 另依赖其 `src` 下第三方库路径（见该目录 `Makefile`）。

## 运行

各程序均需 **1 个参数**：输入视频文件路径。

```bash
# 车人检测
./aeb_od.elf /path/to/video.mp4

# 车道线
./aeb_ldw /path/to/video.mp4

# 限速牌检测 + 分类
./aeb_tsr.elf /path/to/video.mp4
```

请确保设备可访问显示（OpenCV `imshow` 等），且视频编码为系统 GStreamer/FFmpeg 与 OpenCV 所支持格式。

## 已知说明

- 后处理以 demo 为主，生产环境需自行补全 NMS、标定、多传感器融合等逻辑。

## 许可与致谢

若本工程基于 Xilinx / 社区示例衍生，请遵循相应开源协议与 Xilinx 软件许可。模型与工具链版本升级时，请以 [Xilinx Vitis AI](https://github.com/Xilinx/Vitis-AI) 官方文档为准。

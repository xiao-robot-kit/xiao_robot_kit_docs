<!-- SPDX-License-Identifier: MIT -->
# XIAO Robot Kit Docs

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

XIAO Robot Kit 的**使用方法文档**：这块板子是什么、三个库怎么用、每个示例
拿去怎么跑、跑起来看什么、换参数怎么改。**教学导向**——不是文件清单,
是「拿到手 30 分钟内跑通第一个示例」的路。

## 阅读地图（按你的角色选路）

| 你是谁 | 走这条路 | 预计时间 |
| --- | --- | --- |
| 第一次拿到套件 | [硬件介绍](hardware/board.md) → [跑通第一个示例](examples/first-run.md) | 15 min |
| ESP-IDF 开发者 | [IDF 库教学](tutorials/idf.md) → [imu_quickstart 拆解](examples/idf-imu-quickstart.md) | 30 min |
| Arduino 用户 | [Arduino 库教学](tutorials/arduino.md) → [imu_raw 拆解](examples/arduino-imu-raw.md) | 30 min |
| 没有实物板 | [QEMU 仿真](https://github.com/xiao-robot-kit/xiao_robot_kit_qemu) → [串口协议](https://github.com/xiao-robot-kit/xiao_robot_kit_qemu/blob/main/docs/protocol-spec.md) | 20 min |
| 想用 AI 代写 | [Vibe 工作区教学](tutorials/vibe.md) → [大小库切换](https://github.com/xiao-robot-kit/xiao_robot_kit_vibe) | 10 min |
| 想可视化调试 | [上位机指南](https://github.com/xiao-robot-kit/xiao_robot_kit_host) | 10 min |

## 六仓总览

```
xiao-robot-kit（组织）
├── xiao_robot_kit_idf     手工档 IDF 库——四组件 C 核心 + 已验证 IDF 示例
├── xiao_robot_kit_arduino 手工档 Arduino 库——同一颗 C 核心 + 三 sketch
├── xiao_robot_kit_vibe    vibecoding 工作区——五配方，AI 代写，大小库自由切换
├── xiao_robot_kit_docs    ★ 本仓：硬件介绍 + 教程 + 示例用法
├── xiao_robot_kit_qemu    QEMU 虚拟板——无实物烧录调试
└── xiao_robot_kit_host    上位机——已支持示例的可视化交互调试
```

两个手工档库是**同一颗 C 核心**（MIT），API 一致，只有构建形态不同。切换成本为零。

## 目录

```
docs/
├── README.md               ★ 本文件（教学地图）
├── hardware/
│   └── board.md            硬件介绍：XIAO ESP32-S3 + 引脚 + 板载器件
├── tutorials/
│   ├── idf.md              ESP-IDF 手工档库教学（挂依赖 → 最小工程 → 跑示例）
│   ├── arduino.md          Arduino 手工档库教学（装库 → 选板 → 跑 sketch）
│   └── vibe.md             vibecoding 工作区教学（让 AI 读配方 → 派活 → 验收）
└── examples/
    ├── first-run.md        第一次跑通：imu_raw 或 imu_quickstart（选一个）
    ├── idf-imu-quickstart.md   imu_quickstart 逐片拆解（IDF）
    ├── arduino-imu-raw.md      imu_raw 逐片拆解（Arduino）
    ├── arduino-motor-sweep.md  motor_sweep 逐片拆解（Arduino 电机三形态）
    └── arduino-attitude-print.md  attitude_print 逐片拆解（姿态融合看什么）
```

## 通用知识（所有教程共用）

- **引脚分配**：IMU I2C = D4/D5（GPIO5/6）；电机 D0/D1/D8/D9（GPIO1/2/7/8）；
  串口 = USB-Serial-JTAG（插上就有，不用接杜邦线）。详见
  [硬件介绍](hardware/board.md)。
- **串口帧协议** :IMU/姿态/电机遥测都走同一帧打包（SOF 0xA5 | LEN | TYPE |
  SEQ | PAYLOAD | CRC8）。上位机与 QEMU 都认这套帧。完整规范见
  [protocol-spec.md](https://github.com/xiao-robot-kit/xiao_robot_kit_qemu/blob/main/docs/protocol-spec.md)。
- **零硬编码参数**：两个库的一切可调项都来自命名预设表（`imu.raw.104`、
  `motor.servo.sg90`…）。改行为 = 换预设，不是改代码里的魔法数字。

## 内容守则（本仓自我约束）

- 每个教程都以「**能跑通的最小闭环**」收尾——跑完有可见现象、可验证。
- 不罗列文件清单；文件树只作为帮你定位的索引。
- 数字写实测值，截图占位不会冒充已测。

## License

[MIT](LICENSE) © XIAO Robot Kit Contributors
<!-- SPDX-License-Identifier: MIT -->
# 硬件介绍：XIAO Robot Board V1.0（XIAO ESP32-S3）

> 这块板子是 XIAO ESP32-S3（Seeed）配上 XIAO Robot Board V1.0 扩展板：
> 扩展板把 IMU、电机驱动接口、电源都接到了 XIAO 的标准引脚上。
> 引脚：以下数字是 **GPIO 号**（Arduino 与 IDF 都用 GPIO 号编址）。

## 核心板：XIAO ESP32-S3

| 特性 | 值 |
| --- | --- |
| 主控 | ESP32-S3（双核 Xtensa LX7, 240 MHz） |
| 内存 | 8 MB PSRAM / 8 MB Flash |
| 板载串口 | USB-Serial-JTAG（插 USB-C 即出现串口，默认 115200,8N1） |
| 无线 | Wi-Fi + BLE（本教程用不到） |
| GPIO 引出 | D0~D10（A0~A2 与 D0~D2 复用） |

## 板载 / 扩展板器件

| 器件 | 位置 | 与 XIAO 的接口 | 库驱动 |
| --- | --- | --- | --- |
| LSM6DS3TR-C 六轴 IMU | 扩展板，贴片 | I2C（D4=SCL? 见下文） | `xiao_imu` |
| 四路电机接口 | 扩展板排针 | GPIO PWM / 舵机 / H 桥 | `xiao_motor` |
| 电源 | USB-C / 3.7V 锂电池 | 5V/3.3V 输出 | — |

## 引脚分配表（XIAO ESP32-S3 GPIO 号）

来自示例实测（`examples/idf/imu_quickstart/main/bus_i2c_idf.c`、
`examples/motor_sweep/motor_sweep.ino`）：

| XIAO 引脚 | GPIO | 用途 | 备注 |
| --- | --- | --- | --- |
| D4 | GPIO5 | **IMU I2C SDA** | 例程 I2C_NUM_0 |
| D5 | GPIO6 | **IMU I2C SCL** | 同上 |
| D0 | GPIO1 | 电机通道 A | 开环 PWM / 舵机信号 |
| D1 | GPIO2 | 电机通道 B | 同上 |
| D8 | GPIO7 | H 桥方向 A（一组） | 直流电机/舵机 |
| D9 | GPIO8 | H 桥方向 B（一组） | 直流电机正反 |
| D6/D7 | — | 串口 / 避开 | USB-Serial-JTAG 已够用 |

> ⚠️ 以上为示例实测编号（motor_sweep.ino 头部注释：`D0=1 D1=2 D8=7 D9=8`；
> `D4/D5=I2C，D6/D7=串口，避开`；imu_quickstart：`SDA=GPIO5 SCL=GPIO6`）。
> 换板改 `pin_a/pin_b` 与 `BUS_SDA_GPIO/BUS_SCL_GPIO` 即可。

## I2C 总线约定

- 总线 0（I2C_NUM_0），SDA=GPIO5、SCL=GPIO6，速率来自预设
  `cfg->i2c_speed_hz`（如 `imu.raw.104` 预设）。

## 遇到问题先看这里

- 板插上串口没出现 → 换线 / 按住 BOOT 再插（进入下载模式）。
- IMU 读零 → 确认总线是 0、引脚没接反。
- 电机不动 → 先试开环 PWM 槽（最简），再看 H 桥两路时序。

> 实物图与接线照片：待补（占位——见 README 内容守则，不冒充已测）。
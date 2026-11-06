<!-- SPDX-License-Identifier: MIT -->
# ESP-IDF 手工档库教学（xiao_robot_kit_idf）

> 目标：在 30 分钟内，用 `xiao_robot_kit_idf` 库跑通第一个 IMU 流式示例。
> 同一颗 C 核心的 Arduino 形态见 [Arduino 教学](arduino.md)。

## 0. 你在用什么

`xiao_robot_kit_idf` 是一个 **ESP-IDF 组件库**（esp32s3），四个独立组件：

| 组件 | 头文件目录 | 干的事 |
| --- | --- | --- |
| `xiao_imu` | `src/imu/` | LSM6DS3TR-C 驱动：探测 → 按预设配置 → 物理量采样 |
| `xiao_attitude` | `src/attitude/` | 四元数 + Mahony 互补滤波 → roll/pitch/yaw |
| `xiao_motor` | `src/motor/` | 电机实例（PWM 定时器 / PID / S 梯度轨迹） |
| `xiao_pstore` | `src/pstore/` | 参数持久化（项注册表 + 帧序列化 + CRC32） |

零硬编码参数：所有可调项来自预设表（`imu.raw.104` 等），见各组件 `*_presets`。

## 1. 建立工程（两条路选一）

**路 A：直接抄已验证工程（推荐第一次用）**

```bash
git clone https://github.com/xiao-robot-kit/xiao_robot_kit_idf
cd xiao_robot_kit_idf/examples/idf/imu_quickstart
idf.py set-target esp32s3
idf.py build flash monitor      # 串口 115200,8N1
```

打开串口监视器应看到 CSV 流（`az≈±1g, gz≈0`）：

```
az=+0.976g gz=+0.2dps
az=+0.981g gz=-0.4dps
...
```

**路 B：挂进你自己的工程**

在项目 `CMakeLists.txt` 加一行（`EXTRA_COMPONENT_DIRS` 或 `add_subdirectory`）：

```cmake
set(EXTRA_COMPONENT_DIRS ${CMAKE_CURRENT_LIST_DIR}/xiao_robot_kit_idf)
```

然后 `main/CMakeLists.txt` 里 `REQUIRES` 全列你要用的组件
（库 CMake 约定：`REQUIRES` 无条件全列，不要只写一个）：

```cmake
idf_component_register(
    SRCS "main.c"
    INCLUDE_DIRS "."
    REQUIRES xiao_imu xiao_attitude xiao_motor xiao_pstore)
```

## 2. 最小代码（抄 imu_quickstart 主循环）

```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "lsm6ds3.h"        /* xiao_imu */
#include "imu_cfg.h"
#include "bus_i2c_idf.h"    /* 你自己的一层：填 imu_i2c_bus_t 函数指针表 */

void app_main(void)
{
    const imu_cfg_t *cfg = imu_cfg_find("imu.raw.104");  /* 预设，非魔法数字 */
    bus_i2c_init(cfg->i2c_speed_hz);

    lsm6ds3_t imu;
    lsm6ds3_init(&imu, &bus_i2c);
    while (lsm6ds3_probe(&imu) != 0) vTaskDelay(pdMS_TO_TICKS(1000));
    lsm6ds3_configure(&imu, cfg);

    for (;;) {
        imu_sample_t s;                    /* 单位：g / dps */
        if (lsm6ds3_read(&imu, &s) == 0)
            printf("az=%+.3fg gz=%+.1fdps\n", s.az, s.gz);
        vTaskDelay(pdMS_TO_TICKS(cfg->sample_period_ms));
    }
}
```

## 3. 改行为 = 换预设（不是改代码）

| 想要的效果 | 换哪行 |
| --- | --- |
| 采样率更快/更慢 | `imu_cfg_find("imu.raw.104")` → 另一个预设（`.raw.208` 等，看 `imu_cfg_presets.c`） |
| 换 I2C 引脚 | `bus_i2c_init` 里的 SDA/SCL GPIO 常量 |
| 调 PI 增益 | 用 `xiao_motor` 时找 `motor_cfg_presets` 的 PID 预设 |

## 4. 单元测试（改动库本身时跑）

```bash
cd xiao_robot_kit_idf
cmake -S . -B build && cmake --build build
ctest --test-dir build --output-on-failure   # 7 套，1,311 断言
```

## 5. 常见坑

- `idf.py build` 首次会拉 IDF 依赖，慢是正常的。
- 烧不进 → 按住 BOOT 键再插 USB。
- 改了组件但没生效 → `idf.py fullclean` 再 build（增量缓存会骗你）。

下一步：逐片拆解这个工程 →
[imu_quickstart 拆解](../examples/idf-imu-quickstart.md)。
# WS63E 蜂鸣器 / LED 外设示例

基于 HiSpark `fbb_ws63` SDK（WS63E / LiteOS）的两个外设示例，已在本机完成代码集成与编译验证。

## 目录说明

| 目录 | 内容 |
| --- | --- |
| `beep/` | 蜂鸣器示例：PWM 控制 GPIO 9 输出，驱动蜂鸣器鸣响。源码 `pwm_example.c` |
| `led/` | LED 示例：GPIO 7 输出，每 50ms 翻转一次，实现红灯闪烁。源码 `led_example.c` |
| `integration/` | 集成时修改过的 `peripheral/CMakeLists.txt` 和 `peripheral/Kconfig`，供对照参考 |

## 关键改动

1. 将 `vendor/HiHope_NearLink_DK_WS63E_V03/demo/beep` 的示例拷贝到
   `src/application/samples/peripheral/beep/`，并补齐 `Kconfig`。
2. 在 `src/application/samples/peripheral/CMakeLists.txt` 末尾增加：
   ```cmake
   if(DEFINED CONFIG_SAMPLE_SUPPORT_LED)
       add_subdirectory_if_exist(led)
   endif()

   if(DEFINED CONFIG_SAMPLE_SUPPORT_BEEP)
       add_subdirectory_if_exist(beep)
   endif()
   ```
3. 在 `src/application/samples/peripheral/Kconfig` 增加 `SAMPLE_SUPPORT_LED` 与
   `SAMPLE_SUPPORT_BEEP` 配置项的声明及菜单入口。
4. 在 `src/build/config/target_config/ws63/menuconfig/acore/ws63_liteos_app.config` 中使能：
   ```text
   CONFIG_ENABLE_PERIPHERAL_SAMPLE=y
   CONFIG_SAMPLE_SUPPORT_BEEP=y
   CONFIG_SAMPLE_SUPPORT_LED=y
   CONFIG_SAMPLE_SUPPORT_HELLOWORLD_OLED=y
   ```

注意：教程 PDF 里的 `uapi_pwm_set_group` 是旧签名，实际 SDK 是
`(group, channel_set, len)` 三个参数，本仓库代码已按新版 SDK 写法修正，可正常编译。

## 编译与烧录

编译命令（需 HiSpark 工具链在 PATH 中）：

```bash
export PATH="/d/HiSPark/tools/python:/d/HiSPark/tools/python/Scripts:/d/HiSPark/tools/python/Lib/site-packages/cmake/data/bin:/d/HiSPark/tools/Windows/ninja:/d/HiSPark/tools/Windows/cc_riscv32_win_env:$PATH"
python build.py -c ws63-liteos-app
```

编译产物：`src/output/ws63/fwpkg/ws63-liteos-app/ws63-liteos-app_all.fwpkg`

烧录：HiSpark Studio → 工程配置 → 程序加载，传输方式选 serial，选择对应 COM 口；
出现 "Connecting, please reset device..." 时复位开发板。烧录完成后蜂鸣器鸣响 / LED 闪烁。

## 验证状态

- 已编译成功，ELF 含 `pwm_entry` / `pwm_task` 符号。
- 硬件烧录需要开发板连接到串口（当前环境未接板，尚未烧录验证）。

## 许可

源码沿用 HiSilicon 原示例的 Apache-2.0 许可声明。

# OnePlus 13T 设备树比较

最后核对：2026-08-25

本页比较已经收集或被构建指南明确引用的 OnePlus 13T（`PKX110` / `pagani`）设备树。比较只记录冻结提交、Git 历史、文件结构和配置差异，不判断哪套实现正确，也不把未测试来源提升为实机结论。

## 范围与方法

当前能够递归读取并比较的源码包括：

- 三棵进入核心逐文件统计的 `device/oneplus/pagani` 机型树；
- 三棵进入核心逐文件统计的 `device/oneplus/sm8750-common` 公共树；
- 通过 Evolution X XDA 源码入口定位的一组 `renhiyama` 机型树／公共树和相机 vendor 树，以及用户筛选保留的 `Neveark` common 树；
- 当前已构建基线与 `RandomLemon` 候选各一棵 `hardware/oplus`，用于定向比较 UDFPS shim；
- 一棵面向多款 OPLUS SM87xx 设备的 TWRP recovery 树。

另保留两棵 `Oneplus-13T-AOSP` 历史冻结树。它们在 2026-08-24 已不能通过原公开 URL 读取，因此只引用 2026-08-16 留存的 commit 和审计记录，不用当前可访问性改写历史快照。

文件数量按冻结 commit 的 Git blob 统计；“内容相同”按 blob ID 判断。proprietary 数量排除了空行和注释行。

## 来源组合

| 组合 | `pagani` 机型层 | `sm8750-common` 公共层 | 分支／用途 |
|---|---|---|---|
| 已构建并实机验证的冻结基线 | [`OnePlus-13T-Development@dfe9aa4`](https://github.com/OnePlus-13T-Development/android_device_oneplus_pagani/tree/dfe9aa41e8154fc89dc217efae564bac6c376216) | [`LineageOS@44ad18f`](https://github.com/LineageOS/android_device_oneplus_sm8750-common/tree/44ad18fa12b51983a36fb7ec67c54a6b4c032859) | 均为 `lineage-23.2`；对应本仓库 2026-08-14 构建基线 |
| zzkeier 待参考组合 | [`pagani@f935f38`](https://github.com/zzkeier/android_device_oneplus_pagani/tree/f935f38673bc581a09944ec342984816c7db78ca) | [`common@4247c19`](https://github.com/zzkeier/android_device_oneplus_sm8750-common/tree/4247c19bb791f9ff293e0e169380c83a46c10bd2) | 机型层为 `lineage-23.0`，公共层为 `lineage-23.2` |
| ABNOTF 构建指南指定的实际源码输入 | [`pagani@560f47e`](https://github.com/ABNOTF/android_device_oneplus_pagani/tree/560f47ecd11bb520db4319bde01c0fa03c36fa50) | [`common@5018caf`](https://github.com/ABNOTF/android_device_oneplus_sm8750-common/tree/5018cafefeb84ddc7ad1beda21f6d70ed6f28163) | 均为 `lineage-23.2`；由 [`patches_for_pagani@1818541`](https://github.com/ABNOTF/patches_for_pagani/tree/1818541189e505057f84a34ad4ba86f86e571fb5) 的构建命令直接指定 |
| Evolution X 帖公开的源码组合 | [`renhiyama pagani@04be737`](https://github.com/renhiyama/android_device_oneplus_pagani/tree/04be737a8349ffe940addaa75f3ea8b7140c1fc6) | [`renhiyama common@b83096a`](https://github.com/renhiyama/android_device_oneplus_sm8750-common/tree/b83096a7331d33a5732f38eb4bbfc4b781f54d87) | 分别为 `op13s`／`lineage-23.2`；来自 [Evolution X XDA 帖的源码入口](https://xdaforums.com/t/rom-unofficial-a16-evolution-x-for-oneplus-13s-pagani-in.4787408/post-90582155)，这里只收源码，不采用该 ROM 作为当前构建评价基准 |
| 用户筛选保留的 Neveark common 候选 | — | [`Neveark common@86ca73e`](https://github.com/Neveark/android_device_oneplus_sm8750-common/tree/86ca73eaa40b6d06e2f9fd2a4336fd562aa101c9) | `lineage-23.2`；当前只取得公共树，不推定与任意 `pagani` 机型树组成可构建组合 |
| 历史 AOSP 冻结快照 | `Oneplus-13T-AOSP@438f9c9` | `Oneplus-13T-AOSP@faca90d` | 历史 `16.0` 快照；原远端当前不可公开读取 |

ABNOTF 组合在本页称为“实际构建树”，仅表示它们是该构建指南给出的真实 clone 目标，不表示本仓库已经使用这套组合完成构建或设备测试。

## 依赖拓扑

三棵 `pagani` 树的 `lineage.dependencies` 内容完全相同：

```text
device/oneplus/pagani
└── android_device_oneplus_sm8750-common
    └── device/oneplus/sm8750-common
```

三棵 common 树的 `lineage.dependencies` 也完全相同：

```text
device/oneplus/sm8750-common
├── hardware/oplus
├── kernel/oneplus/sm8750
├── kernel/oneplus/sm8750-devicetrees
└── kernel/oneplus/sm8750-modules
```

这些依赖文件只写仓库名和目标路径，没有固定 GitHub 所有者、分支或 commit。因此，表中的“组合”依据已验证 manifest、已收集链接或构建指南命令整理，不是由 `lineage.dependencies` 自动锁定。

## `pagani` 机型树

### 总览

| 项目 | Development | zzkeier | ABNOTF |
|---|---|---|---|
| 冻结 HEAD | `dfe9aa41e8154fc89dc217efae564bac6c376216` | `f935f38673bc581a09944ec342984816c7db78ca` | `560f47ecd11bb520db4319bde01c0fa03c36fa50` |
| HEAD 日期 | 2026-03-01 | 2026-01-08 | 2026-08-22 |
| Git blob 数 | 39 | 47 | 45 |
| 显示尺寸／密度 | 1216×2640／480 dpi | 1416×2460／560 dpi | 1216×2640／480 dpi |
| UDFPS 接入 | `hardware/oplus` 的共享 `libudfps_extension.oplus` | 本地 `fingerprint/` 扩展及三个 shim | 设置 `qtidisplay.oplus_udfps=true` |
| 本地 `mixer_paths.xml` | 无 | 有 | 有 |
| PlusKey keylayout | 有 `gpio-keys.kl` | 无该文件 | 有 `gpio-keys.kl` |
| 电源扩展 | `qtipower/power-ext-oplus` | `qtipower/power-ext-oplus` | 本地 `libperfmgr-ext-pagani` |
| eSIM 相关 | 无机型专用 overlay | 构建 `OplusEsimSwitcher`、`OplusEuicc` | 构建 `FrameworksResTargetEuicc`；两个 Oplus 包在 `device.mk` 中被注释 |
| proprietary 条目 | 1,388 | 1,431 | 1,420 |
| firmware 条目 | 30 | 30 | 30 |

三份 `proprietary-firmware.txt` 的 30 个非注释条目相同，文件头的来源说明不同：

- Development：`PKX110_16.0.3.501(CN01)`；
- zzkeier：`CPH2653_16.0.1.304(EX01)`；
- ABNOTF：`PKX110_16.0.9.400(CN01)`。

### Git 历史关系

- Development 与 zzkeier 的 merge base 是 `a3c98b8b86a484714b76b0165f91c7e04b034047`，两侧各有 25 个独有提交。
- ABNOTF 的冻结 HEAD 是 Development `dfe9aa4` 的后代，在该点之后有 35 个提交。
- zzkeier 与 ABNOTF 仍可追溯到 `a3c98b8`；相对该点分别有 25 和 60 个提交。

### 文件级结果

| 对比 | 左／右 blob 数 | 相同路径且内容相同 | 相同路径但内容不同 | 仅左侧路径 | 仅右侧路径 |
|---|---:|---:|---:|---:|---:|
| Development ↔ zzkeier | 39／47 | 10 | 23 | 6 | 14 |
| Development ↔ ABNOTF | 39／45 | 22 | 17 | 0 | 6 |
| zzkeier ↔ ABNOTF | 47／45 | 8 | 26 | 13 | 11 |

Development 相对 zzkeier 独有的路径包括 PlusKey keylayout、Lineage SystemUI overlay，以及四套 `sw328dp`、`sw363dp`、`sw423dp`、`sw427dp` 尺寸资源。

zzkeier 相对 Development 独有：

- `configs/audio/mixer_paths.xml`；
- `fingerprint/` 下的 UDFPS extension、Biometrics/Fppress/SensorProps shim；
- `sepolicy/private/` 下四个文件；
- project `24821` 的 recovery `build.default.prop`；
- `sw320dp`、`sw360dp`、`sw411dp` 尺寸资源。

ABNOTF 相对 Development 没有删除路径，新增六个路径：

- `configs/audio/mixer_paths.xml`；
- 三个 `FrameworksResTargetEuicc` overlay 文件；
- `power/Android.bp`；
- `power/power-mode.cpp`。

### 显示配置

- Development 与 ABNOTF 均声明 1216×2640、480 dpi；zzkeier 声明 1416×2460、560 dpi。
- Development 与 ABNOTF 都使用多点 SDR→HDR ratio map，但各点数值不同。
- zzkeier 另带完整 brightness value→nits 映射，并使用两点 SDR→HDR ratio map。
- 三者的高亮模式、亮度 ramp 和部分 overlay 尺寸值存在差异。本页不把这些数值解释为实机正确性结论。

### proprietary 列表

| 对比 | 完全相同条目 | 仅左侧 | 仅右侧 |
|---|---:|---:|---:|
| Development ↔ zzkeier | 1,352 | 36 | 79 |
| Development ↔ ABNOTF | 1,381 | 7 | 35 |

zzkeier 的右侧独有项包含部分面板色彩文件、音频算法版本文件、相机校准文件和音频／显示组件。ABNOTF 的右侧独有项包含 eUICC、Dolby Vision、显示 color feature、部分 secure TA 与 UFS firmware 文件路径。这里只记录清单组成，不代表这些文件已被重新分发或测试。

## `sm8750-common` 公共树

### 总览

| 项目 | LineageOS | zzkeier | ABNOTF |
|---|---|---|---|
| 冻结 HEAD | `44ad18fa12b51983a36fb7ec67c54a6b4c032859` | `4247c19bb791f9ff293e0e169380c83a46c10bd2` | `5018cafefeb84ddc7ad1beda21f6d70ed6f28163` |
| HEAD 日期 | 2026-08-01 | 2026-06-19 | 2026-07-26 |
| Git blob 数 | 82 | 83 | 88 |
| `BOOT_SECURITY_PATCH` | `2026-07-01` | `2026-03-01` | `2026-04-01` |
| `ro.build.version.svn` | 91 | 未设置 | 88 |
| `oplus/kernel/graphics:kbuild` | 无 | 有 | 有 |
| 电源服务 | `android.hardware.power-service-qti` | 同左 | `android.hardware.power-service.lineage-libperfmgr` |
| 本地 OMAPI UUID 配置文件 | 无 | 有 | 无；phone blob 条目创建相关 symlink |
| DeviceAsWebcam overlay | 无 | 无 | 有 |
| `oplus_sync_fence.ko` | 未列入两个模块表 | 正常／recovery 模块表均列入 | 正常／recovery 模块表均列入 |
| common proprietary 条目 | 928 | 943 | 868 |
| phone proprietary 条目 | 484 | 487 | 485 |

proprietary 来源注释分别是：

- LineageOS：`CPH2653_16.0.9.401(EX01)`；
- zzkeier：`PKX110_16.0.3.501(CN01)`；
- ABNOTF：`CPH2653_16.0.5.703(EX01)`。

### Git 历史关系

- LineageOS 与 zzkeier 的 merge base 是 `03e510cc52b554f43d1ffb395d742446c3a12627`；LineageOS 侧 15 个独有提交，zzkeier 侧 5 个。
- LineageOS 与 ABNOTF 的 merge base 是 `e0721a9406d4262bf10138c6f351bcaaac5c0347`；LineageOS 侧 13 个独有提交，ABNOTF 侧 41 个。
- zzkeier 与 ABNOTF 从 `03e510c` 分开；zzkeier 侧 5 个独有提交，ABNOTF 侧 43 个。

### 文件级结果

| 对比 | 左／右 blob 数 | 相同路径且内容相同 | 相同路径但内容不同 | 仅左侧路径 | 仅右侧路径 |
|---|---:|---:|---:|---:|---:|
| LineageOS ↔ zzkeier | 82／83 | 71 | 11 | 0 | 1 |
| LineageOS ↔ ABNOTF | 82／88 | 62 | 20 | 0 | 6 |
| zzkeier ↔ ABNOTF | 83／88 | 65 | 17 | 1 | 6 |

LineageOS 与 zzkeier 的 11 个同路径差异集中在：

- `BoardConfigCommon.mk`、`common.mk`；
- audio policy；
- `extract-files.py`；
- `init/ueventd.oplus.rc`、`init/ueventd.qcom.rc`；
- `modules.load`、`modules.load.recovery`；
- `odm.prop`；
- 两份 proprietary 列表。

zzkeier 相对 LineageOS 的唯一新增路径是 `configs/omapi/hal_uuid_map_config.xml`。

ABNOTF 相对 LineageOS 的六个新增路径是 `configs/power/powerhint.json`、`configs/sensors/sns_aont.json` 和四个 `DeviceAsWebcamResTarget` overlay 文件；另有 20 个同路径文件内容不同。

### 2.4 GHz Wi-Fi 相关差异

已单独收录的候选提交是 [`zzkeier@9280681`](https://github.com/zzkeier/android_device_oneplus_sm8750-common/commit/92806812c82f10d42ea663c1b6348c2a97294d7b)。三个冻结 HEAD 的状态是：

- LineageOS：`ueventd.qcom.rc` 的 firmware directory 只有 `/vendor/firmware_mnt/image/`；扩展目录另写在 `ueventd.oplus.rc`；proprietary 列表没有 `WCNSS_qcom_cfg_roam.ini` 与 `WCNSS_qcom_cfg_cmcc.ini`。
- zzkeier：`ueventd.qcom.rc` 同时列出 persist、ODM Wi-Fi 和 vendor firmware 路径；proprietary 列表保留上述两个 WCNSS 配置。
- ABNOTF：历史中包含对应 Wi-Fi 路径提交 `cb33454`；当前 HEAD 保留扩展 firmware directory，但两个 WCNSS 配置条目已在后续提交中移除。

这些是源码状态，不代替 2.4 GHz 与 5 GHz 的分频段实机测试。

### UDFPS、模块与公共服务差异

- zzkeier 的 `ueventd.oplus.rc` 为 `fp_touch_state` 和 `notify_fppress` 设置节点权限；LineageOS 与 ABNOTF 冻结 HEAD 没有这两行。
- zzkeier 和 ABNOTF 都在正常与 recovery 模块表中列入 `oplus_sync_fence.ko`；LineageOS 没有列入。
- ABNOTF 切换到 Lineage libperfmgr，并增加本地 `powerhint.json`；另增加 DeviceAsWebcam overlay。
- LineageOS 的 audio 配置包含较新的 AIDL interface 组合和 stub audio policy include；另两棵树的 audio 文件、fixup 与 proprietary 组合不同。

## `renhiyama` 与 `Neveark` 的定向源码比较

这部分按用户筛选结果只保留可复用源码，不评价论坛 ROM 本身。用户当前实机反馈是相机等功能已经可用，明显问题集中在指纹；因此 Evolution X 帖中的旧相机故障反馈不外推到当前构建。

### `renhiyama` 源码组

Evolution X XDA 帖的公开源码入口包含：

- [`android_device_oneplus_pagani@04be737`](https://github.com/renhiyama/android_device_oneplus_pagani/tree/04be737a8349ffe940addaa75f3ea8b7140c1fc6)，分支 `op13s`；
- [`android_device_oneplus_sm8750-common@b83096a`](https://github.com/renhiyama/android_device_oneplus_sm8750-common/tree/b83096a7331d33a5732f38eb4bbfc4b781f54d87)，分支 `lineage-23.2`；
- [`vendor_oplus_camera@563c432`](https://github.com/renhiyama/vendor_oplus_camera/tree/563c432ecd67c645cbc9ed20de7959d6173f5d34)，分支 `main`。该树包含 OPlus Camera APK、库和其他 proprietary 内容，本仓库只记录链接，不复制其中二进制，也不把它列为当前指纹问题的优先输入。

机型树是本仓库已验证构建所用 Development `dfe9aa4` 的直接后代，之后有 58 个提交。其 `device.mk` 接入 `hardware/oplus` 的 `libudfps_extension.oplus`，设置 `qtidisplay.oplus_udfps=true`；OPlus framework overlay 声明 `org.lineageos.sensor.udfps` 长按传感器，但屏幕关闭 UDFPS 的两个布尔项仍被注释。

common 树相对冻结 LineageOS `44ad18f` 的 merge base 为 `e0721a9406d4262bf10138c6f351bcaaac5c0347`；LineageOS 侧 13 个独有提交，renhiyama 侧 27 个。与指纹直接相关的源码自述包括：

- 在指纹服务上同时链接 `libshims_aidl_fingerprint_v3.oplus.so` 与 `libshims_fingerprint.oplus.so`；注释称前者提供运行时传感器参数，后者处理解锁 bootloader 下的新指纹录入限制；
- 新增 `vendor.oplus.biometrics.fingerprint.*` property context，并允许 `touchDaemon` 读取 `vendor_fingerprint_prop`；注释把它与 HBP/LHBM 协调、录入帧质量和 enrollment 停滞联系起来。

以上均是冻结源码中的实现与注释，不是本仓库的设备验证结果。

### `Neveark` common 候选

[`Neveark/android_device_oneplus_sm8750-common@86ca73e`](https://github.com/Neveark/android_device_oneplus_sm8750-common/tree/86ca73eaa40b6d06e2f9fd2a4336fd562aa101c9) 的冻结 HEAD 有 88 个 blob。它与 renhiyama common 的 merge base 同为 `e0721a9`；从该点起，renhiyama 侧 27 个独有提交，Neveark 侧 40 个，两个 HEAD 之间有 24 个路径发生新增、删除或修改。

该 HEAD 会给 OPlus 指纹服务加入 `libshims_aidl_fingerprint_v3.oplus.so`，可作为传感器类型、坐标和尺寸参数路径的比较对象；但在此次冻结点没有 renhiyama 的第二个 enrollment shim，也没有其 `sepolicy/vendor/property_contexts` 与 `oplus_touchdaemon.te`。这里只记录差异，不判断哪一套组合正确，也不把仓库名称或 commit 标题当成指纹已修复证据。

## `hardware/oplus` 与 UDFPS 事件桥

本仓库成功构建时冻结的是 [`LineageOS/android_hardware_oplus@ec3b821`](https://github.com/LineageOS/android_hardware_oplus/tree/ec3b8211676f55ed09905c4c336356acedc040d3)。社区截图指向的候选现已固定到 [`RandomLemon/android_hardware_oplus@5e5ff9a`](https://github.com/RandomLemon/android_hardware_oplus/tree/5e5ff9a5bfce2183e538c4a00b3fd51d4e8719c8)：

| 项目 | 已构建 LineageOS | RandomLemon 候选 |
|---|---|---|
| 冻结 HEAD | `ec3b8211676f55ed09905c4c336356acedc040d3` | `5e5ff9a5bfce2183e538c4a00b3fd51d4e8719c8` |
| HEAD 日期 | 2026-07-16 | 2026-07-27 |
| Git blob 数 | 2,500 | 730 |
| 从 merge base 的独有提交 | 30 | 9 |
| 指纹 shim | 当前 `SensorPropsShim`，支持 `iconlocation` 和 DRM 模式推导 | `SensorPropsShim` 加 `OplusFodShim` |
| 状态 | 已进入成功构建；指纹录入仍失败 | `NOT_APPLIED + NOT_TESTED` |

两棵树的 merge base 是 `dad4fa2064230191bb616f17cf660423dfd31d94`，HEAD 间共有 1,786 个路径差异，所以整棵替换会同时倒退或改变大量与 pagani 指纹无关的接口。限定到 `fingerprint/` 后，只有三个路径不同：`Android.bp`、新增的 `OplusFodShim.cpp`，以及 `SensorPropsShim.cpp`。当前 LineageOS 一侧的 SensorProps 还带有较新的 [`7d36841`](https://github.com/LineageOS/android_hardware_oplus/commit/7d3684150bc3ca73e21507f11e22b87f97d24ba3) `iconlocation`／DRM 分辨率逻辑，不能随候选一起回退。

`OplusFodShim` 的冻结实现会：

- 监测 `/sys/kernel/oplus_display/fp_state`；
- 在状态变化、AIDL pointer down/up，或 HAL 回调 vendor acquired code `22`／`23` 时写入 `/sys/kernel/oplus_display/notify_fppress`；
- 在认证成功、错误或 session close 时写 `0` 并重置内部状态；
- 作为源码被编进现有 `libshims_aidl_fingerprint_v2/v3/v4.oplus`，而已构建 common 树本来就会把 v3 shim 注入 UFF 指纹服务。

因此它是当前 `waitUiready` 超时问题最接近的源码候选，但还不是已验证修复。`.501` 的 `init.oplus.display.rc` 会把 `notify_fppress` 设为 `system:system 0666`；冻结 LineageOS common 的 proprietary list 包含该文件，`init.oplus.rc` 也会导入它。冻结 `hardware/oplus` 还把 `/kernel/oplus_display` 标为 `vendor_sysfs_graphics`，并允许 fingerprint HAL 读写该类。也就是说，缺少专用 ueventd 行不等于访问链缺失。后续最小候选应只移植 `OplusFodShim` 与构建依赖，先验证现有节点模式、上下文、HAL 注入和 HBM 清理；只有实机出现 AVC／权限失败时才追加最小规则。完整文件证据见 [`.501` 交叉审计](stock-501-cross-audit.md)。

另有两个容易误判的源码边界：

- zzkeier pagani 树确实含本地 `FppressShim.cpp` 和 `.pagani` shim 定义，common 树也有 `notify_fppress` 的 ueventd 权限，pagani 策略允许指纹 HAL 访问该节点；但检查到的 `device.mk` 没有构建这些 `.pagani` shim，common `extract-files.py` 实际仍给 OEM 服务注入 `.oplus` v3 shim。因此“文件存在”不等于该实现已进入产物。
- ABNOTF 的 SystemUI 补丁会直接在 finger down/up 写 `notify_fppress`，但冻结构建指南只列出并应用相机补丁；其 writer 是 SystemUI，而不是已有通用访问规则覆盖的 fingerprint HAL，因此当前源码也没有证明 SystemUI domain 可以写该节点。该补丁同样不能单独作为可落地修复。

### 2.4 GHz 提交排重

Neveark 历史中的 [`cb33454`](https://github.com/Neveark/android_device_oneplus_sm8750-common/commit/cb33454925ab2bec9eb4077a1cfeea4b9b86f3fa) 与已收录的 [`zzkeier@9280681`](https://github.com/zzkeier/android_device_oneplus_sm8750-common/commit/92806812c82f10d42ea663c1b6348c2a97294d7b) 具有相同作者日期、标题和两项语义改动：扩展 firmware 搜索目录，并加入两个 WCNSS 配置条目。两者因应用基线／上下文不同而具有不同 commit 与 patch ID，视为同一候选实现的移植，不重复登记为第二条修复。

Neveark 随后的 [`4106c3d`](https://github.com/Neveark/android_device_oneplus_sm8750-common/commit/4106c3dba07896445fcbe251879e3143c6964847) 又移除了 `WCNSS_qcom_cfg_roam.ini` 与 `WCNSS_qcom_cfg_cmcc.ini` 两项 proprietary 条目，冻结 HEAD 仍保留扩展 firmware directory。后续验证仍以原始 zzkeier 链接为单一入口，同时记录 Neveark HEAD 的实际状态。

`.501` 审计确认两个额外配置都存在，但没有闭合候选的运行机制：普通原厂 init 路径仍选基础配置，cmcc 选择 helper 不在所查设备树清单中，而且新增 firmware 搜索目录与配置实际目录不同。因此“提交差异已排重”和“原厂文件已确认”都不等于“2.4 GHz 修复已验证”；采用前仍须追踪 persist 配置来源并做分频段实测。详见 [`.501` 交叉审计](stock-501-cross-audit.md#24-ghz-wi-fi)。

## TWRP recovery 树

[`kmiit/twrp_device_oplus_sm87xx@74d0756`](https://github.com/kmiit/twrp_device_oplus_sm87xx/tree/74d075623b35f685fcde82efb5a43548697d1a6e) 是 recovery 树，不是 `lineage_pagani` 系统产品树：

- 产品名为 `twrp_sm87xx`，同时覆盖多款 OPLUS SM87xx 设备；
- `libinit` 按 project ID 识别机型，其中 `24821` 映射到 `PKX110` / `OP60F5L1`；
- 配置 A/B、动态分区、FBE、recovery fstab、显示、触摸和 OPLUS `my_*` 分区；
- 冻结 HEAD 共 336 个 Git blob，约 31.5 MB。

因此它只在 recovery 启动、分区与设备识别维度与系统设备树交叉，不参与本页对系统相机、UDFPS、音频或产品 overlay 的一对一统计。

## 补丁仓库边界

[`ABNOTF/patches_for_pagani@1818541`](https://github.com/ABNOTF/patches_for_pagani/tree/1818541189e505057f84a34ad4ba86f86e571fb5) 本身不是设备树。冻结 HEAD 只有三个文件：

- `0001-tmp-add-trigger-for-udfps.patch`：修改 SystemUI `UdfpsController`，在 finger-down/up 时写入 `notify_fppress`；
- `0002-tmp-hide-cameraid0-for-aperture.patch`：修改 Aperture 的相机 ID 忽略逻辑；
- `README.md`：构建步骤，以及本页收录的 ABNOTF `pagani`／common clone 目标。

README 中写出的第二个补丁名是 `0002-tmp-hide-cameraid1-for-aperture.patch`，仓库实际文件名是 `0002-tmp-hide-cameraid0-for-aperture.patch`。这里只保留名称差异，不推导补丁正确性。

## 历史 AOSP 快照

2026-08-16 的审计冻结了：

- `Oneplus-13T-AOSP/android_device_oneplus_pagani@438f9c9e68215bd690a4a79285d43b2ba26c6f7c`；
- `Oneplus-13T-AOSP/android_device_oneplus_sm8750-common@faca90d7fa295b15be8daca0f9179394c907ffa3`。

两者当时位于 `16.0` 分支，最新提交说明均为从 COS 11 C.58 更新。当前原 URL 已不能作为公开仓库读取，因此本页没有把它们加入当前递归文件数量或三方 HEAD 统计；历史差异继续以 [AOSP 源码差异审计](aosp-gap-audit.md) 为准。

## 本页不作出的结论

- 不根据文件数量、提交数量或日期判断实现优先级。
- 不把构建指南指定的源码输入写成成功构建或实机验证。
- 不把 proprietary 列表中的路径当成可公开分发文件。
- 不把 commit 标题当成 Wi-Fi、指纹、相机、HDR 或其他功能已经修复的设备证据。
- 后续若采用任何一棵树，仍需在单独记录中固定完整 manifest、stock 来源、构建结果和设备测试。

## 2026-08-25 本地构建树快照

本次本地审计实际读取的是 [本地构建审计](local-build-audit-2026-08-25.md) 中的 `<LINEAGE_ROOT>`。六个冻结成功构建基线 commit 均重新确认；本地 dirty 变化不属于新的公开设备树组合，也没有覆盖既有候选比较。

| 层 | 本地 HEAD | dirty 状态 | 与冻结基线的实际变化 |
|---|---|---|---|
| `device/oneplus/pagani` | `dfe9aa41e8154fc89dc217efae564bac6c376216` | 是 | pagani sepolicy、`qtidisplay.oplus_udfps`、camera AIDL prebuilt 冲突处理 |
| `device/oneplus/sm8750-common` | `44ad18fa12b51983a36fb7ec67c54a6b4c032859` | 是 | `notify_fppress` ueventd 节点规则 |
| `hardware/oplus` | `ec3b8211676f55ed09905c4c336356acedc040d3` | 否 | 无 |
| `frameworks/base` | `aaa4284f7e061771afb58c789924397111487e62` | 是 | SystemUI `UdfpsController` 直接写入 `notify_fppress` |
| `hardware/qcom-caf/sm8750/display/core` | `20cf597e21bdd31af4e3a55660e991e22f69bf8b` | 是 | fingerprint mask connector property |
| `hardware/qcom-caf/sm8750/display/hal` | `4b74f47925c54e95c805275832a65830af0431b6` | 是 | `oplus_udfps` 编译宏和 include dirs |
| kernel / DT / modules | `999b95d4` / `ebb25e35` / `eab0a34a` | 否 | 无 |

`SOURCE_VERIFIED`：当前 `hardware/oplus` 仍只有共享 `SensorPropsShim` 和 `libudfps_extension.oplus`，没有 RandomLemon 候选的 `OplusFodShim.cpp`。因此本地 dirty SystemUI writer 与既有硬件 shim 候选是两条不同实现路径，不能合并描述为同一个已验证修复。

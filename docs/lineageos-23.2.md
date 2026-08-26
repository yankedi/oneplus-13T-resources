# LineageOS 23.2：构建、安装与硬件状态

更新时间：2026-08-26

2026-08-25 的本地源码、dirty 状态、active 输出混合快照和重新验证结果见[本地构建审计](local-build-audit-2026-08-25.md)。本页的 2026-08-14 完整 OTA 和 2026-08-16 设备状态结论保持原时间截面，不被未重新打包的增量输出覆盖。

2026-08-26 新增 [ARB1 Linux EDL 实机报告](edl-arb1-device-validation-2026-08-26.md)：从连贯的 2026-08-14 target-files 取出 Lineage recovery，通过 EDL 写入 `recovery_b`，首次失败经诊断修正后重试通过完整回读。随后普通 Android B 启动、Lineage Recovery `23.2-20260814-UNOFFICIAL-pagani` 启动身份及 root ADB 已确认；Recovery 日志存在显示提交和 `/data` 挂载警告，UI、触控、解密和备份恢复尚未完成验收。本次没有重装 ROM、刷写其他分区或改动 A 槽。

## 构建产物

2026-08-14 的 `lineage_pagani-userdebug` 构建完成，产物为非官方、`userdebug/test-keys`：

| 字段 | 值 |
|---|---|
| 文件 | `lineage-23.2-20260814-UNOFFICIAL-pagani.zip` |
| 大小 | 2,503,659,880 bytes |
| SHA256 | `bcdec8cbcf5b42f42ff2e998ecb8f49bab6316fdc5d36d426725f0ee12bdde8e` |
| Android | 16 / SDK 36 |
| Kernel | 6.6.142 |
| OTA 类型 | A/B payload OTA |

主机侧检查：

- whole-package signature：PASS；
- payload signature、hash 和 size：PASS；
- AVB chain 与 hashtree：PASS（test key，root vbmeta flags `3`）；
- target-files VINTF：`compatible`；
- dynamic partition image-fit：`6,334,275,584 <= 13,329,498,112`。

这些结果是 `HOST_VERIFIED`，不能单独证明真机可启动或 userdata 安全。

## 冻结源码版本

首次成功构建/安装所用关键源码：

| Project | Commit |
|---|---|
| `OnePlus-13T-Development/android_device_oneplus_pagani` | `dfe9aa41e8154fc89dc217efae564bac6c376216` |
| `LineageOS/android_device_oneplus_sm8750-common` | `44ad18fa12b51983a36fb7ec67c54a6b4c032859` |
| `LineageOS/android_hardware_oplus` | `ec3b8211676f55ed09905c4c336356acedc040d3` |
| `LineageOS/android_kernel_oneplus_sm8750` | `999b95d4792dd41fba72a7914abe897cc3ee2ecd` |
| `LineageOS/android_kernel_oneplus_sm8750-devicetrees` | `ebb25e3526ad84cc5a1090a5a9242f33ff087bf2` |
| `LineageOS/android_kernel_oneplus_sm8750-modules` | `eab0a34a8b657ee22839c9b3878c3b6e687675f9` |

构建中的 pagani `proprietary-files.txt` 有最小本地修正，用于解决 AON/camera 重复 prebuilt 和跨分区模块冲突。由 `.501/.401` 生成的 proprietary vendor tree 含受限制的专有文件，不应公开分发。

## 安装时间线与关键教训

### 第一次写入 slot A

45 个 payload partitions（7 dynamic、8 boot-critical、30 firmware）成功写入目标 slot A，update_engine 返回成功。首次启动进入 Lineage Recovery，并报告 `init_user0_failed`。

决定性日志是 userdata 的 F2FS superblock 读取为无效数据。旧 ColorOS 的 metadata-encryption 链不能由该 Lineage 构建直接解开，因此不存在保留原 `/data` 的无损启动路径。此时没有擅自 wipe。

### 第二轮写入 slot B

在用户授权清除 userdata 后，第一次从 Lineage Recovery factory-reset 重启携带 `reboot,factory_reset`，OPlus ABL 回到健康的 stock slot A，并把 B 标成 unbootable；snapshot 被取消。该行为最初只是一项机制假设。

重新 sideload 后，直接尝试启动 B 仍因 TWRP 创建的 userdata 布局触发 `init_user0_failed`。随后在 **Lineage Recovery B 内执行 factory reset**，再经 bootloader 重启，Lineage 成功进入完整 userspace，并把 slot B 标记为 successful。

结论：

```text
当前 Lineage 构建的 /data 必须由 Lineage Recovery 创建。
不要使用 TWRP Format Data 为它准备 userdata。
```

## 最终成功状态（2026-08-16）

| 字段 | 值 |
|---|---|
| Booted slot | `_b` |
| Lineage | `23.2-20260814-UNOFFICIAL-pagani` |
| Android | 16 / SDK 36 |
| Kernel | `6.6.142-4k-g999b95d4792d` |
| `/data` | Lineage Recovery 新建，成功挂载 |
| Slot A | ColorOS `.501`，successful=yes |
| Slot B | Lineage，successful=yes |
| Snapshot state | none（已完成/合并） |

## 硬件冒烟测试

| 子系统 | 结果 | 边界 |
|---|---|---|
| Cellular RF / Mobile Data | PASS | NR SA 与数据连接稳定 |
| IMS / VoLTE / VoNR | PARTIAL→通话记录为正常 | 仍建议补充标准化长通话测试 |
| Wi-Fi | PASS | 当时 L2 双向约 500 Mbps |
| Bluetooth | PARTIAL | HAL/service 在；配对未按测试表复测 |
| NFC | PASS | enabled |
| GNSS | PARTIAL | service 注册；实际 fix 仍需标准化复测 |
| Camera | PASS | provider 工作，枚举 4 个设备；仍需补实拍矩阵 |
| Audio | PASS | QTI HAL、speaker patch、低延迟流工作 |
| Sensors | PASS | accel/gyro、mag、light/prox 注册 |
| USB / ADB | PASS | 稳定 |
| Charging | PASS | 基础充电状态正常 |
| Display | PASS | 亮度与唤醒正常，UDFPS overlay 可显示 |
| Fingerprint enrollment | FAIL | `waitUiready` 超时，vendor error `15001` |

这里的 Camera `PASS` 只表示当前 AOSP/Lineage Camera provider、设备枚举和基础拍摄链工作，不表示 OPlus/ColorOS 原厂相机已经完成移植。关于 ColorOS ODM 文件和 OPlus framework 依赖的新增线索见 [OPlus / ColorOS 相机移植线索](camera-porting-clue.md)。

### 2.4 GHz Wi-Fi 的验证边界

上表中的 Wi-Fi `PASS` 没有单独记录频段，因此不能外推为 2.4 GHz 已完成专项验证。[`zzkeier/android_device_oneplus_sm8750-common@9280681`](https://github.com/zzkeier/android_device_oneplus_sm8750-common/commit/92806812c82f10d42ea663c1b6348c2a97294d7b) 是一项待参考的 pagani 2.4 GHz 候选修复：它扩展 ueventd firmware 搜索路径，并把两个额外的 ODM Wi-Fi 配置文件加入 proprietary 列表。

该差异不在本次成功构建冻结的 `LineageOS/android_device_oneplus_sm8750-common@44ad18f` 中，尚未合入、构建或实机测试。具体文件变化和采用前检查项见[候选参考目录](pending-references.md#24-ghz-wi-fi--common-tree-候选修复)。

## UDFPS 根因边界

已经确认：sensor、TA、vendor HAL、VINTF、framework provider 和 overlay 显示链存在；失败发生在 OPlus HAL 的 `waitUiready` 门。Settings enrollment 场景没有在约 502 ms 窗口内向 HAL 送达 `session.onUiReady()`。

`.501` 文件级审计进一步确认：原厂 `init.oplus.display.rc` 会把 `notify_fppress` 设为 `system:system 0666`；冻结 common 会导入该 RC，冻结 `hardware/oplus` 也已有 fingerprint HAL 对 `/kernel/oplus_display` 通用类型的读写规则。因此第一优先级不是更换指纹 APK、vendor blob、firmware、预先堆叠 SELinux 规则或盲目忽略错误，而是补齐并验证：

1. kernel 是否提供 `/sys/kernel/oplus_display/notify_fppress`；
2. 现有 init ownership/mode 与通用 SELinux 链在实机是否生效，有无 AVC；
3. 当前 v3 shim 中加入最小 `OplusFodShim` 事件桥后是否真实加载；
4. finger-down/up、`notify_fppress`、`waitUiready`／`postUiready` 的顺序与取消/休眠清理；
5. HBM、触控、手势、AOD 与解锁的回归。

## 蜂窝信号格

当时 60 秒采样为 NR SA n1，ssRSRP 中位数约 `-88 dBm`、ssRSRQ `-12 dB`，数据正常；UI 报告 level 2。运行时采用 AOSP 默认 NR SSRSRP 阈值 `[-110, -90, -80, -65]`，且没有有效的 OPlus product CarrierSettings。

`.501` 的进一步结果是：官方 common overlay 的中国运营商 LTE 门限与当前 Lineage 对应块一致；OPlus overlay 中发现的 NR SSRSRP/SSSINR 门限属于日本运营商块，CMCC、CU、CT、CBN 块没有同类表。`ro.oplus.radio.lte_rsrp_thresholds` 是 ColorOS LTE 属性，而 `oplus_carrier_ims_rtp_redun_*` 是 IMS 媒体质量门限，两者都不能直接当成中国 NR 状态栏门限。

因此结论仍是 **RF 正常，bar mapping/config 有缺口**，但 `.501` 没有给出一个可直接复制的中国 NR 阈值答案。修复应继续检查运行时 CarrierConfig 选择和当前 framework 行为，不能通过替换 modem/RIL/早期 firmware 解决。文件级证据见 [`.501` 交叉审计](stock-501-cross-audit.md#运营商与信号格)。

## 不应直接移植的内容

- `.500` generation 的 ABL/XBL/TZ/HYP/AOP/modem/DSP；
- 完整 vendor tree 或 camera blobs；
- 旧的 Fppress/SensorProps 重复实现；
- 未复现的 vendor error `10108` 抑制；
- aggressive power hints、LTPO magic write 或整批 framework patch。

详细差异见 [AOSP 源码差异审计摘要](aosp-gap-audit.md)。
